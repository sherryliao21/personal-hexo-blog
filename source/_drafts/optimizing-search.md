---
title: 解決問題的過程
date: 2022-03-03 14:58:28
tags: [work-related, backend]
categories:
- Work
---

### Outline

<p align="center">
  <img src="../optimizing-search/01.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

# **前言**
「解決問題」可以說是工程師的日常，例如：
1. 需求下來了，準備實作功能時，如何去規劃
2. 知道程式碼有 bug，要找出它並修好，使服務正常上線
3. 服務 performance 不佳，想找出徵節點並加以改善

但我卻很少好好檢視自己「面對問題並找出原因」的過程，我想藉由最近一次解決問題的過程，來剖析自己面對問題的思考、與後續採取動作的流程，做一次詳細的紀錄。

# **發現問題**
最近在做一個以「計測週期」為條件撈取歷史資料，在前端利用線圖呈現的功能。計測週期從 每 10 秒、每小時、每天、到每月都有，而撈取資料的筆數固定為 16筆（往前推16個區間，包含當下），而資料組數則以「撈取多少個子節點」的資料決定。

使用 每 10 秒、每小時、每天 的計測週期撈取資料時，回應都在一秒以內。**唯獨使用「月」的計測週期來撈取時，回應超過 7 秒。**

## **情境補充**
1. mainNode 與 subNode、meter 之間的關係
<p align="center">
  <img src="../split-stack-coop/nodes.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

該功能為取得所有 subNode 下之 meter 資料加總，並以 subNode 分類呈現。而這次總共有 7 條產線（subNode），8 個 meter。

2. 不只撈取資料，也做 down sampling
由於連接 PLC 儲存資料的週期為每 10 秒一筆，因此 24 小時不間斷的儲存，每天都有 86400 筆。為了使撈取筆數和所需時間在不同計測週期時盡量不要差太多，固做了取樣。

3. 為了前端線圖函式庫所需，必須幫忙補 timestamp 與 null 值
由於有些機台會在某幾日或某些時段停機，這段時間內，PLC 並不會回傳資料，因此資料庫中的資料會有時間上的斷層。為了能夠讓前端繪製線圖，做了補 null 值與 timestamp 的動作。

4. 歷史包袱：每次都要打兩個 DB 撈取資料
目前資料庫由別的團隊管理，因資料量龐大，且容量有限，需搬移儲存或定期清理，當初設計時便決定將資料拆分為「今天」以及「今天以前」的資料，分別除存在兩個不同的 DB。若遇到撈取資料起始時間有跨今天及過去，會需要在兩個資料庫撈取資料並做合併。

# **收集資訊**
發現問題之後，必須先收集能夠「證明這的確造成問題」、以及「問題到底發生在哪」的資訊。這次會發現問題主要是因為**前端在串接資料時發現線圖畫的好慢，跟後端反映要求資料時間過長**。

收到前端的 feedback 後，我便著手尋找問題出處。首先想再次確認究竟是前端還是後端的問題，我使用 devTools 查看瀏覽器 request 的 Timing，發現**Waiting(TTFB) 超級久**，看來是後端 API response 拖時間。

<p align="center">
  <img src="../optimizing-search/04.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

再來我使用 postman 測試這支 API，輸入一樣的參數，觀察 response time：

<p align="center">
  <img src="../optimizing-search/01.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

確定是後端問題後，回到這支 API 的 controller 檢視到底是哪個 function 拖最久時間：

之前整頓過架構，將 controller 中的 function 再拆開，依據職責封裝到 model 或 helper 中。目前已經包裝很精簡，函式粒度適中、一個函式只做一件事，所以可以快速測試花費時間。我在呼叫該 function 那行的前後，加上 `console.log(new Date().getTime())`，來看這個 function 花了多久才回傳值

pseudo code:
```
function getData(source) {
    // 一些非同步程式碼
}

console.log(`START: ${new Date().getTime()}`)
const data = await getData('database1')
console.log(`END: ${new Date().getTime()}`)
```

到這邊發現是有去 DB 撈資料的 function 在拖，而且兩個 function 差不多時間。於是選了一個進去追，從 model 層（組裝/加總）到 helper 層（純撈），看到底是哪裡拖。

原來是 helper 層（純撈）在拖阿！

於是我印出 SQL logging，把 raw SQL 貼去 workbench 執行，觀察一下 performance：

<p align="center">
  <img src="../optimizing-search/02.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

<p align="center">
  <img src="../optimizing-search/03.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

原來一條產線一次的 query 時間就是花這麼久。回頭看程式碼，有幾個產線迴圈就跑幾次，算算產線數、比對回應時間，這個結果是合理的，看來拖慢時間的部份是這裡。

# **根據資訊去分析原因的過程、得出結論（問題點）**
看了 SQL performance，`Using where; Using temporary; Using filesort` 可以理解它就是會花這麼多時間（`temporary, filesort` 都很耗時）

<p align="center">
  <img src="../optimizing-search/02-1.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

<p align="center">
  <img src="../optimizing-search/03-1.png" style="margin: 20px; height: 300px; width: 600px"/>
</p>

關於 filesort, temporary，我查到一些[資料一](https://www.redoc.top/article/65/MySQL%E8%A7%A3%E5%86%B3using%20filesort%E5%AF%BC%E8%87%B4%E7%9A%84%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98)、[資料二](https://www.gushiciku.cn/pl/pjuz/zh-tw)，之後有空來研究看看。

於是得到結論：這些**非同步**的程式碼必須被優化（廢話）。
# **討論可行方法**



4. 說明根據這些資訊去分析原因的過程、得出結論（問題點）
    1. 回頭看程式碼，有幾個產線就有幾個迴圈，算算時間跟產線數，這個結果是合理的，看來拖慢時間的部份是這裡
    2. 看了 SQL performance，Using where; Using temporary; Using filesort 可以理解它就是會花這麼多時間（filesort 會耗）（圖 02-1, 03-1）
    3. 得到結論：要怎麼把這些 async code 優化呢。
5. 討論可行方法
    1. 優化 raw query
    2. 使用 Promise.all() ，因為目前都是用 await 寫成同步程式碼，因此一個等一個，都在排隊，當然比較慢
        1. 如果可以同時出發去做，時間就可以不用累加，而是縮短成一次所需的時間
        2. 例子說明：原本 300毫秒 * 7 條產線 = 2100 毫秒，使用 Promise.all() 之後照理來說，不會差 300 毫秒的 1~2 倍太遠，至少不會是 2100 毫秒那麼久
6. 決定可行方法並開始動手改
    1. 根據之前計畫實作 down sampling + 現階段能力不足，已經無法再優化 raw SQL，固決定使用 Promise.all() 對這些迴圈做包裝、甚至今天昨天DB 做包裝。
7. 修正過程與實做結果測試
    1. 包裝大致的模式（pseudo code）
    2. 結果展示：只改一個 function 就加速了兩秒（圖 05），5.41 s（共兩個 function）（圖 06），兩個都改完變成 2.57s
    3. 測試後，前端工程師表示：（圖 07）
8. 後記，心得
    1. 拆解問題的過程
    2. 尋找優化方案
    3. 重新認識 Promise
    4. 重構心得、MVP 仍需不斷改進，產品迭代這樣的過程很正常