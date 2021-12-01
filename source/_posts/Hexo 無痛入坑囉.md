---
title: Hexo 無痛入坑囉
date: 2021-12-01 17:14:32
tags: [Hexo]
categories:
  - Tutorials
---

![](https://i.imgur.com/R9hqhuM.png)

先附上 [我的 Hexo](https://icaughtacode-hexo-blog.netlify.app/)，之後裡面的內容會跟 Medium 做出區別～

開始工作後經歷了一段手忙腳亂的適應期，最近總算是能夠重啟自學習慣，但對於一些在工作上學到的、片段式的筆記、或刷題紀錄，很難更新在 Medium 上，因為它給我的感覺比較正式，不寫完整點、不潤稿的話，沒辦法隨意發佈。

所以我開始搜尋替代方案：希望能夠快速撰寫、推出文章，好管理，可自訂 domain name、對於內容樣式有更大的掌控權...，看來看去 Hexo-blog 滿符合我的需求：
- 使用 Node.js
- 可以快速建立
- 使用 Markdown 語法
- 檔案好管理
- Build 和 deploy 也很容易且自動化

雖然網路上很多教學的資源，但我使用的這個佈景主題目前還沒什麼人在討論，在建立的過程中踩了不少雷，所以乾脆整理成一份分享，希望透過這篇讓還在對 Hexo 猶豫的朋友們無痛入坑。
<!-- more -->

---

首先你需要有 Node.js & Git。Hexo 本身是用 Node.js 打造的部落格框架，過程中我們也會使用 npm install 相關套件庫，十分便利快速。Git 則做為版本控制及後續部屬靜態網站服務重要的工具。

## 下載 Node.js

以下是官網建議的 Hexo - Node.js 版本對應表：

| Hexo version | Minimum Node.js version |
| --------  | -------- |
| 5.0+      |	10.13.0|
| 4.1 - 4.2 |	8.10   |
| 4.0       |	   8.6 |
| 3.3 - 3.9 |       6.9|
| 3.2 - 3.3 |	   0.12|
| 3.0 - 3.1 | 0.10 or iojs|
| 0.0.1 - 2.8|	0.10   |

建議不管 Hexo 或 NodeJS 都能夠下載最新（至少 LTS）的版本，才能得到官方修補漏洞或優化後的最佳體驗。

跟隨官方教學先在 terminal 用 npm 下載 hexo-cli 工具 `$ npm install -g hexo-cli`，下載好之後，利用 hexo-cli 的指令來產生我們的目標資料夾 `$ hexo init <folder_name>`

進入資料夾 `$ cd <folder_name>` 後， `$ npm install` 來下載所有必須的套件。之後你的資料夾結構大致會長這樣：

```
.
├── _config.yml
├── node_modules
├── package.json
├── package-lock.json
├── scaffolds
|   ├── draft.md
|   ├── page.md
|   └── post.md
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
## **檔案介紹**
這邊有幾個比較重要的檔案/資料夾，稍微介紹一下：

### **package.json**
Hexo 的文章內容是使用 ejs 等模板語言來撰寫，經解析後渲染成靜態的 HTML，所以可以在 `package.json` 的 `dependencies` 裡面看到 ejs, stylus, markdown 的 renderer-package

```
  "dependencies": {
    "hexo": "^5.0.0",
    "hexo-generator-archive": "^1.0.0",
    "hexo-generator-category": "^1.0.0",
    "hexo-generator-index": "^2.0.0",
    "hexo-generator-tag": "^1.0.0",
    "hexo-renderer-ejs": "^2.0.0",
    "hexo-renderer-marked": "^4.0.0",
    "hexo-renderer-stylus": "^2.0.0",
    "hexo-server": "^2.0.0",
    "hexo-theme-landscape": "^0.0.3"
  }
```
### **scaffolds/**
scaffolds 資料夾中有三個檔案： `draft.md`, `page.md`, `post.md`

這些是檔案模板，每使用 `$ hexo new <type> <name>` 創造一個新的貼文或頁面，Hexo 就會使用 `scaffolds` 中的模板為你建立檔案雛型。

### **source/**
`source` 資料夾放著這個網站所有的資源。前綴詞帶有底線的資料夾會被 Hexo 忽略，`_post` 除外，因為這裡面放的是我們要上架的文章。這些靜態檔案（`Markdown`, `HTML` 等）在 build 完後會被放進 public 資料夾，而其他的檔案則是用複製的方式。

### **theme/**
theme 資料夾用來放佈景主題相關的各種資料，官方預設的佈景主題是 `landscape`，如果想要換佈景，也可以到[官方的 theme shop](https://hexo.io/themes/) 去找你喜歡的主題，下載後整包放到 `themes/` 下面來使用

(這邊建議佈景主題盡量挑選近期很多人穩定在維護的專案，不然出問題不知道要到哪裡反映 😂)

### **_config.yml**
`_config.yml` 是我們最重要的設定檔，這個檔案中可以針對我們的全站呈現方式做設定

## **主題設定教學**
由於這次使用的主題 [aomori](https://github.com/lh1me/hexo-theme-aomori) 網路上沒有太多人介紹，踩雷無數後提供我的簡易設定：

### **_config.yml**
注意這裡的 `_config.yml` 指的是根目錄下的，而不是 `theme/`底下的那一個喔。

#### **全站設定**
```
# Site
title: I Caught A Code
subtitle: 
description: 
keywords: web dev, self-taught programmer, NodeJS, JavaScript
author: Sherry L
language: en
timezone: 
```
#### **網站 url 設定 (僅改動 `url`)**

```
# URL
url: https://icaughtacode-hexo-blog.netlify.app/
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

#### **存放檔案的資料夾設定 (預設)**
這個關乎到 build 產生靜態檔案之後他們的去處 
```
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```

#### **文章寫作相關設定**
這邊只有將 `highlight`: `enable` 設為 `false`，是我使用的 aomori 主題的設置要求
```
# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
filename_case: 0
render_drafts: false
post_asset_folder: true
relative_link: false
future: true
highlight:
  enable: false
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```
#### **首頁顯示文章筆數、順序等設定**
設置一頁顯示最多 5 篇文章
```
# Home page setting
index_generator:
  path: ''
  per_page: 5
  order_by: -date
```

#### **分頁器設定**
如果將 per_page 設置為 0，就不會有分頁器
```
# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page
```
#### **主題設定**
把我們放在 `theme/` 底下的佈景主題資料夾名稱填上來
```
# Extensions
theme: hexo-theme-aomori
```
#### **佈署設定**
如果需要用到 Github Pages 或 Heroku 等服務來 deploy，這邊記得填入。不過這次我使用的是 Gitlab + Netlify，所以這邊用不到：
```
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: ''
```

#### **加入 aomori 這個主題特製的設定**
```
# aomori_customized settings
aomori_logo: /images/code.png
aomori_logo_typed_animated: true
aomori_menu:
  Home: /
  About: /about
  Archives: /archives
aomori_widgets:
  - toc # Article navigation
  - category  # Article classification
  - tag # Article tags
  - recent_posts  # latest articles
  - archive # Article Archive
aomori_copyright: false
aomori_social:
  -
    icon: github # Boxicons name
    type: logo # Boxicons type
    url: https://github.com/sherryliao21 # Your social link
  -
    icon: linkedin-square
    type: logo
    url: https://www.linkedin.com/in/sherrycliao/
  -
    icon: medium
    type: logo
    url: https://icaughtacode.medium.com/
aomori_google_analytics: 'UA-XXXXXXXXX-X'
aomori_favicon: 📓
```
- `aomori_logo` 裡的檔案路徑，是在 `themes/hexo-theme-aomori/source/images` 之下，可以把你想要的 logo（畫面左上角，部落格名稱左邊）圖案放到這個資料夾中，用相對路徑去引用它
- `aomori_menu` 是網站的分頁，類似 RESTful API 將不同類型的資源 group 起來，這上面有什麼，就必須在根目錄 source 資料夾中有名字一樣的資料夾，才能打到這支 API、這個畫面才會有內容。例如：

    ```
    aomori_menu:
      About: /about
      
    在 source 中就需要有 about 資料夾、 about/index.md 檔案
    ```
    這邊可以使用 `hexo-cli` 的指令幫我們產生 `$ hexo new page <page_name>`，Hexo 會自動幫你在根目錄下的 source 資料夾中產出一個 <page_name> 資料夾，包含一份 `index.md` 檔案，像這樣：

    ![](https://i.imgur.com/zElROb3.png)

- `aomori_widgets` 是 sidebar 中的分類項目
- `aomori_copyright` 可以寫一些版權相關的聲明，預設是中國地區的版本，可以自己到 `themes/hexo-theme-aomori/layout/partial/post/copyright.ejs` 中更改，我覺得太麻煩就設定 `false` 關掉了
- `aomori_social` 裡面可以放你的社群媒體帳號，logo 的部份要到 [Boxicons](https://boxicons.com/) 找，把他的名字和 HTML tag 中的 type 填到裡面，例如這個 LinkedIn logo：
![](https://i.imgur.com/Kd9RUjE.png)
- `aomori_favicon` 可以套用 emoji ，當作自己網站的 favicon
- `aomori_google_analytics` 就填入你的 Google Analytics 追蹤碼

## **開始寫文章**
使用 `hexo-cli` 指令產生新貼文模板 `$ hexo new post <post_name>`

### **設定 front-matter**
front-matter 是 Hexo 對於文章模板的設定，預設的 post 模板會長這樣：

```
---
title: {{ title }}
date: {{ date }}
tags:
---
```
- `title` 輸入你的文章標題
- `date` 如果用 `hexo-cli` 產生文章模板，會自動帶入
- `tags` 及 `categories` 可以自訂標籤及分類

以 leetcode 解題紀錄這篇為例，注意撰寫格式不對的話就無法正常顯示：
```
---
title: "[Leetcode] #206_Reverse Linked List"
date: 2021-08-06 15:21:16
updated: 2021-11-30 15:21:16
tags: [Leetcode, easy]
categories: 
    - leetcode
    - data structure
---
```

定義好 front-matter 就可以開始寫文章了，基本上使用 Markdown Language，也可用 HTML 插入圖片，例如我的 about 頁面：

```
<p align="center">
  <img src="sherry.png" alt="avatar" width="200" style="margin-bottom: 6px; border-radius: 50%; 
  border: 4px #ffffff solid; 
  box-shadow: 6px 6px 10px -4px rgba(189,189,189,0.75);
  -webkit-box-shadow: 6px 6px 10px -4px rgba(189,189,189,0.75);
  -moz-box-shadow: 6px 6px 10px -4px rgba(189,189,189,0.75);"/>
</p>

# **I am Sherry**
A junior web backend engineer who likes to learn about new technologies and cats. 😸
```

如果你希望把這個文章中會用到的靜態資源都集中管理，可以到 `_config.yml` 中把 `post_asset_folder: true` 打開，Hexo 就會在執行 `$ hexo new post <post_name>` 指令時，一併產生一個 `<post_name>` 的資料夾給你放靜態資源，在文章中可以用相對路徑的方式引用

## **分頁管理**
分頁中的資源一樣用資料夾來管理，如果 about 下的 `index.md` 會用到圖片等檔案，也可以一併放在這個 source/about 資料夾中，用相對路徑的方式來引用

## **Live server**
此時可以用 `$ hexo server` 指令來用瀏覽器查看目前本地開發的情況，預設位址是 `localhost:4000`，通常主題作者沒提供模板的頁面，除了背景和一些常設的 widget 外，都會是空白的，你可以自己設計該頁 HTML 或是 利用 Markdown 去填入你想要的資訊。

## **佈署服務**
我們可以把 Hexo 佈署在 [Github Pages]() 上，網路上相關教學很多，這邊介紹我使用的另一種方式： [Gitlab]() + [Netlify]()

Netlify 是一個很強大的靜態網站 hosting 服務，不只免費又快、一切繁雜的手續都幫你打理好了，同時它也有免費的 HTTPS(SSL)、可自訂 DNS 設定、serverless APIs，以及其他強大的包含 CI/CD 任務等功能。

### **Gitlab project repository**
首先在 Gitlab 創一個 project (repo)，並把自己本地的專案推上去

### **Netlify hosting**
到 Netlify 申請一個帳號，然後到你的 team dashboard 中，按 New site from Git 開一個 project

![](https://i.imgur.com/FI77Fdj.png)

接著在 Connect to Git provider 選擇 Gitlab 

![](https://i.imgur.com/Q4AAb7b.png)

會被要求驗證

![](https://i.imgur.com/8ubGFxG.png)

驗證後，從你的 Gitlab 中選擇你要部屬的 project

![](https://i.imgur.com/EfYYg9l.png)

填入 Basic build settings

![](https://i.imgur.com/s5bHAT0.png)

基本上只要填入 Build command 與 Publish directory：
1. Publish directory: 前面有提到 build 後生成的靜態檔案都會放在 public 資料夾下，因此填 public。
2. Build command: 我們專案的 `package.json` 裡的 script 也有寫到 build command 為 `$ hexo generate`，這邊我們只要填 `npm run build` 即可，Netlify 會自動幫我們執行。

接著就是每次你在本地 commit 完 push 到遠端 Gitlab 上，Netlify 就會偵測並啟動 CI/CD 幫你 build 跟自動部屬囉。而且 build 還滿快的，大概 20 秒內就會完成，可以直接在你的 Netlify 網站上看到成品囉。


# **結語**
這就是整個架站的過程，很簡單吧～ 當初不知為何搞很久，但也趁這個機會好好研究了一下官方的 documentation，發現 Hexo 還有很多強大的功能，雖然目前的配置已經堪用，但希望將來有時間可以擴充。

Hexo 的彈性算滿大的，如果想要改畫面或加入一些客製化的東西，可以自己到對應的 ejs 模板去改(前提是你會 ejs)，或是也可以自己開發主題模板！