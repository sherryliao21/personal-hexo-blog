---
title: 解決問題紀錄
date: 2022-03-03 14:58:28
tags: [work-related, backend]
categories:
- Work
---

### Outline

1. 前言，說明為何要寫這篇（首圖）
2. 說明遇到的問題與情境
    1. 問題：用 monthly 條件去找一個 mainNode 下的所有 subNode 的時候很慢，但已經有 down sampling 了還是一樣
    2. 情境解釋：
        1. mainNode 與 subNode、meter 之間的關係（示意圖）
        2. down sampling 大致的解說
        3. 總共有七條產線（subNode）、八個 meter
        4. 每次都會一次打兩個 DB （歷史包袱、產品初期架構）
3. 說明如何去測試、收集資訊的過程
    1. 前端發現 UI 好慢，跟後端講
    2. 用 devTools 看 request 的 Timing，發現 Waiting(TTFB) 超級久 （圖 04）
    3. 先用 postman 找出 API response time（圖 01）
    4. 到這支 API 的 controller 去找，到底是哪個 function 卡了
    5. 因為 controller 已經包裝很精簡，所以可以快速測花費時間
        1. 在該 function 那行的前後，加上 console.log(new Date().getTime())，來看這個 function 花了多久 （pseudo code 圖/embedded code）
    6. 發現是有去 DB 撈資料的 function 在拖，而且兩個 function 差不多時間
    7. 選一個進去追，從 model 層（組裝/加總）到 helper 層（純撈），看到底是哪裡拖 → 原來是 helper 層（純撈）在拖
    8. 印出 SQL logging，把 raw SQL 貼去 workbench 觀察一下 performance （圖 02, 03）
    9. 觀察到一個產線一次的 query 時間就是花這麼久
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