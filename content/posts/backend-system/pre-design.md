---
title: "System Design: 前導"

author: Aryido

date: 2026-06-13T08:44:38+08:00

thumbnailImage: "/images/backend-system/system-design/logo.jpg"

categories:
  - backend-system

comment: false

reward: false
---

<!--BODY-->
> 在做 System Design 時，可以有幾個階段來協助思考，分別是：
> - 釐清探索需求 **Requirement**，而對於產品有所謂的 :
>    - **Functional** 功能需求 : 是從使用者的角度來看，以「User 要能夠...」來思考產品本身要達到的功能
>    - **Non-Functional** 非功能需求 : 從系統的角度來看，以「系統要能夠...」來思考，可以把這類需求理解成「品質」要求，例如 : Scalability、Reliability 、高 Performance 低 latency、可承受高 QPS 、成本效益等等
> - 架構 **High-Level Architecture Design** : 給初步的系統架構，依照**資料流**把會用到的相關元件展示出來，思考每一個部件優缺點，並概略把流程串起來
> - **Detail Deep dive** : 更深入的說明要用的工具、算法、資料結構、框架，流程上資料怎麼進入、儲存、處理、輸出格式等等
> 
> 除了以上事情之外，其實定義出「**什麼不在討論範圍內 (out of scope)**」也是非常重要的，可以展示出「排定優先順序的能力」。雖然 System Design 範圍很大，但其實也是有不少**模板和套路**可以練習。可以多看看各式各樣系統的 high-level 設計，順便把其中自己不熟悉的知識補上。

<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/backend-system/system-design/interview.jpg" >}}

系統設計從職能分工上也會分成：
- 前端系統設計
- 後端系統設計

兩者會有重疊的部分，但又有其專門的地方。例如有一些前端才會要特別側重的點 : UI呈現、使用體驗、無障礙化 (a11y)、國際化 (i18n)等等，還可以參考 [The Elements of UI Engineering](https://overreacted.io/the-elements-of-ui-engineering/) ，這些主題對前端工程師重要，但一般通用系統設計就不會問到這類問題。對於後端系統設計則主要包括 Horizontal scaling、Caching、Load balancing、Database replication/sharding 、非同步、避免單點故障、監控等等，甚至是「節約架構 (The Frugal Architect)」等等，還需要分析 high-level trade-offs ，例如 : 

- 效能與可擴展性 Performance vs Scalability
  > 如果 service 能夠以資源成比例增加的方式提升 Performance ，則稱該服務具有 Scalability。
  > - Performance Problem : 表示系統對於單一使用者來說運作緩慢
  > - Scalability Problem : 在單一使用者使用時速度很快，但在高負載下速度會變慢

- 可用性與一致性 Availability vs Consistency
  > 可以參考 [CAP theorem](https://www.youtube.com/watch?v=vVDhlwXdVb8)，在分散式電腦系統中，網路保證不穩定，因此一定會有分區容錯 partition tolerance，故需要在軟體層面上權衡consistency 和 Availability。

---

# 系統設計模板類

系統設計常見的型態，整理之後大概可以分成以下類型，下面的分類是以「產品型態」或「功能 module 」做基本切分。另外在一個 App 中，功能是可以有以下的許多個 module 綜合的，且因為許多 module 是完全不同的性質，故「Non-Functional Requirement」的側重點也會很不一樣。

### 社交互動
使用者生成內容、社群關係連結、即時互動，這一類包含：
- 社群網站平台：登入、發文、追蹤他人
- 長短影音平台：上傳影音、觀看影音
  {{< alert success >}}
影片相關的話，可以去了解 ffmpeg 之類的 video 編解碼相關軟體
{{< /alert >}}

這一類的系統，基本上 function requirement 就是處理 Data 「Flow In」/「Flow Out」這兩件事情，具體來說就是使用者 upload 自己的東西，其他人可以 read 這種需求，而這種類型最常見的問題是：「**可能會在極短時間湧入極大量的流量**」，且系統都偏向 read-heavy system，經常會使用 cache 和 pre-build 優化檢索，需專注如何處理尖峰時段的吞吐量問題。

{{< alert info >}}
網頁相關服務，壓縮到 latency 壓到 200ms 以下是比較好的體驗
{{< /alert >}}

### 即時互動
- Google doc: 實時同步與衝突解決
  {{< alert success >}}
可去了解 OT (Operational Transformation) 和 CRDT(Conflict-free Replicated Data Type)。業界中用 OT 比較有名的例子是 Google Doc ; Notion 則是使用 CRDT。
{{< /alert >}}
- 聊天室: 即時雙向溝通
- 直播平台: 實時影音傳輸

這種類型概念上是「即時資料雙向傳輸」應用場景，這時候就會 WebSocket 這種傳輸協定 protocol。最好可以有實作或應用 Polling、Long-Polling、WebSocket 的經驗。

### 搜尋、發現，地理地圖位置相關
讓使用者快速找到「附近的、適合的、現在可用的」目標，這一類的本質通常是搜尋、排序、即時狀態更新 :
- 叫車服務
- 訂房服務
- 搜尋系統

共通重點有 Search index ; Geo index 範圍查詢 ; Ranking 依距離、價格、可用性、相關性排序。想像一下 Uber ，用戶請求乘車，平台必須盡快找到附近的司機，這就是使用地理空間索引和記憶體位置儲存的好時機：
- 司機會按地理區域不斷分組，因此可以在幾毫秒內找到附近的匹配項
- 司機位置資訊高度動態，且會被持續查詢，故將即時位置資料保存在記憶體中，以最大限度地減少查找延遲
  {{< alert info >}}
經常變化、經常存取的資料通常應該保存在記憶體中而不是磁碟上
{{< /alert >}}

### 交易、訂單與金流
核心問題如: 錢、庫存、訂單、在分散式的狀況下，如何確保不同節點之間能維持強一致(Strong consistency)，這類型的設計，狀態一致性是非常要注重的：
- 支付系統
- 訂單系統 
- 庫存管理系統

典型的付款請求可能涉及多個操作如：詐欺偵測(Fraud detection) 餘額核實(Balance verification)、銀行授權(Bank authorization)、資料庫事務提交(Database transaction commit)、審計日誌(Audit logging) 等等，這類型要優先思考：重複扣款怎麼防、狀態機怎麼設計、失敗怎麼補償、哪裡一定要強一致(Strong consistency) 等等，通常會選擇更高的延遲，以換取更強的穩定性、可靠性和正確性。

{{< alert warning >}}
例如電商平台，在訂單系統部分為了避免重複下單，會需要比較高的一致性 ; 庫存管理多半是 2B 的系統用的人不多，但是對庫存精確度的掌握要高，前面範例，如果沒花時間討論一致性，而是專注流量優化，方向可能就是錯誤的。

但在例如商品評價留言的部分，大概只會需要高可用，一致性要求可能還好。
{{< /alert >}}
  
### 即時事件與非同步
大量事件怎麼可靠地處理、推送、重試、排程，這一類包含：
- 任務調度系統
- 通知系統

本質上是：事件進來 -> 非同步處理 -> 狀態更新 / 外部通知，所以有 Queue / stream 、 Scheduler、Retry / DLQ、Notification delivery、Event-driven 等等知識點。從事批次系統 、資料分析或 ETL 管道方面的工作，可能首要可以優先考慮 Throughput 吞吐量。

任務調度在 High Throughput 的情況下，因為有這種搶資源的衝突，拆分為不同處理任務 : 「事務型 Online Transaction Processing (簡稱 OLTP)」或「分析型 Online Analytical Processing (簡稱 OLAP)」就非常必要。

{{< alert warning >}}
在 OLAP 這個背景下，「資料倉儲(Data Warehouse)」與「資料湖 (Data Lake)」的概念被提出，透過 ETL 的過程 (Extract-Transform-Load) 的方式，存到 Data Warehouse/Lake 中，而資料科學家與分析師，可以直接對 Data Warehouse/Lake 做查詢而不直接對後端 DB 做查詢，這樣就不會影響到後端系統。
{{< /alert >}}

{{< alert success >}}
- 資料倉儲 Data Warehouse: 使用 ETL 過程會先做資料清洗、格式統一化等，預先定義好資料結構再載入，雖然能讓查詢時能用輕易用最佳化的 SQL 語句來查，但同時也降低了彈性
- 資料湖 Data Lake:  使用 ELT 擷取後就載入，載入後的資料沒有強制特定的格式，僅是作為集合，在要用的時候，再轉換成當下需要的格式
{{< /alert >}}


<!-- ### AI / LLM 
模型推論、上下文管理、工具調用、成本與延遲控制，這一類包含：
- ChatGPT 系統
- AI Support Agent
- LLM Inference API

現在最新型態系統設計，額外知識點蠻多的，需要花點時間去了解以下新的 keyword :
- RAG：檢索 + 知識庫
- context management
- Tool calling / agent workflow
- Model routing
- GPU batching / throughput optimization -->

目前眾多系統設計架構中，比較公認入門系統設計範例，大概就是社群平台系統設計，之後會先以此為基礎來討論 :

> {{< image classes="fancybox fig-100" src="/images/backend-system/system-design/social-web.jpg" >}}

---

### 參考資料

- [系統設計基礎教學 - 導覽](https://www.explainthis.io/zh-hant/swe/system-design-tutorial)

- [九分鐘略懂系統設計面試](https://www.youtube.com/watch?v=Y93BGebBwEE)

- [系統設計入門](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-TW.md)
