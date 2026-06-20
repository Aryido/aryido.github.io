---
title: "System Design: Estimation"

author: Aryido

date: 2026-06-13T08:44:38+08:00

thumbnailImage: "/images/backend-system/system-design/logo.jpg"

categories:
  - backend-system

comment: false

reward: false
---

<!--BODY-->
> back-of-the-envelope calculations 中文翻譯是「**粗略估算**」，英文字面上的意思就是在一張信封後面就能隨手計算這樣，雖然簡單但卻**一定要有一些事實根據來判斷和計算數字**。真的開始在系統設計之前，會跟面試官或者是 PM 或者是自己調查，得到一些參數比如說 Daily active user 有多少等等的，再來會需要由這些假設的參數來「估算一下」系統性能會需要承擔多少，例如有以下常見的指標: 
> - **QPS(Queries Per Second)**
> - **Latency**
> - **Storage Size**
> - **Network bandwidth**
> 
> 這些不僅幫助我們了解系統設計的邊界，也讓我們掌握系統在不同併發情境下的 performance。值得注意的是「**計算上其實都有一些小技巧**」和「**既有數值推估**」可以去留意。

<!--more-->

---

# 每個人都應該知道的一些數字
蠻多數值是取自 [Google Pro Tip: Use Back-of-the-envelope-calculations to Choose the Best Design](https://highscalability.com/google-pro-tip-use-back-of-the-envelope-calculations-to-choo/)給的數值，以下列出一些：
- `L1 Cache 0.5 ns`：如果換算成日常尺度，大約是 0.5 秒
- `L2 Cache 7 ns`：如果換算成日常尺度，大約是 7 秒
- `DRAM 100 ns`： 如果換算成日常尺度，大約是 100 秒
- `SSD 150,000 ns`：如果換算成日常尺度，大約是 1.7 天
- `HDD 10,000,000 ns`： 如果換算成日常尺度，大約是 16.5 週
- `Network Storage 約 30,000,000 ns`：如果換算成日常尺度，大約是 11.4 個月

以上大概給出了資料從 「**memory 直接取得**」和「**跨集群調度**」的時間耗費差異。
{{< alert success >}}
1 奈秒（nanosecond, ns） = `10^-9 sec`
也就是：
- 0.000000001 秒
- 10 億奈秒 = 1 秒
- 1 秒 = 1,000,000,000 ns
{{< /alert >}}

- 雖然一天是 `86400s`，但經常為了實際計算方便會約等於 `100,000 = 100K = 10萬秒` 
- 峰值 peak 經常假設為一般數值的 `3～5 倍`
- 大部分業務可以假設是 `80％ 讀取 20 % 寫入`
- 熱門與非熱門的比率，也經常使用 `80/20 法則`
- 寫入操作的成本，大概是讀取操作的 `40 倍`
- 單機 MySQL database 對於讀寫的極限可以記憶一下 ：
    - 極限讀取 QPS 大概是在 `1K~5K` 左右
    - 極限寫入 QPS 大概不到 `1K`
    {{< alert warning >}}
database 對於寫入，因為要建立索引所以會比讀取的時候慢，大約會低上一個數量級
{{< /alert >}}

-  單機 MySQL database 在儲存 `100 萬` row 以內的資料，其實都沒什麼問題，但差不多到 100 萬這個數字後，就可以開始考慮分庫分表了
- 對於參數的假設，如果有真實產品支持的資料，提出來會蠻有說服力的，例如說 twitter：
  - 每則推文大小為 `140 個字元`，故有 `280 bytes`
  - metadata 為 `30 bytes`

- 約 20% 的文章文包含圖片，每張照片：`200 KB`
- 約 10% 的文章包含影片，每支影片：`2 MB`，其中只有 30% 的影片會被點開觀看

- 作為 in-memory storage 服務器 memory 基本都 `64 GB` 起跳，單台在 `100 GB` 內都算正常，並且使用率達到 `90%` 也都算可以接受的


# Estimation 範例

雖然很推薦在系統設計中，腦中都要都要做估算的動作，但估算數字要對設計的選擇有判斷有幫助才是重要的。以下給出一些估算範例和執行操作筆記：

{{< alert warning >}}
在系統設計面試中，如果估算的東西，對關鍵設計的判斷一點幫助都沒有，這反而會是負面訊號。例如花了五分鐘估算，然後估算出來的數字在接下來的設計都沒有提到怎麼用，那等於白白浪費那五分鐘在做沒幫助的事情
{{< /alert >}}


### 假設一個社群平台 APP ，是一個擁有 1 億的活躍 user 的社群平台 
{{< alert success >}}
「1 億」 也就是 「100 million」，可以使用 100M 來記錄，這種換成 M 的方式可以記憶一下，尤其在之後要算容量 size 非常管用
{{< /alert >}}

##### 若每個 user 每天會刷 10 次社群平台，然後發兩次 Post

- 首先是 「**Read QPS**」: 得到計算公式為 : `100M * 10 times / day`。

  > `100M * 10 times/day = 100000K * 10 times/ 100K sec = 10000 times/sec = 10k QPS`

  峰值算 3～5 倍，那峰值 peak read QPS 的話，大約就是 `50 K`，承上知道單一的 MySQL database 極限讀取 QPS 大概是在 `1000~5000` 左右，明顯承載量是撐不住 peak read QPS `50 K` 的。
  {{< alert success >}}
  因為會除掉 `100K`，故也會把 `100M` 換成 `100,000K` ，這種「增加或減少三個零」、 「K 和 K 之間互相抵銷」的類似操作，經常會在計算中出現
  {{< /alert >}}


- 再來是 「**Write QPS**」: `100M * 2 times / day = 100000K * 2 / 100K sec = 10000 = 2K QPS`
  
  峰值算 3～5 倍，所以 peak write QPS 的話，大約就是 `10 K`，承上知道單一的 MySQL database 極限寫入 QPS 大概不到 `1000` ，同理明顯承載量是撐不住寫入 peak write QPS `10 K` 的。 


從上面分析的結果，就知道系統設計要想一些辦法來 scaling database ，不能簡單指用單機 database 來執行任務了。


### 想引入 Caching 來提高讀取的效能，打算緩存最近三天的熱門的 post

承之前假設，因為 1 億的活躍 user 每天發兩次 Post，打算紀錄前三天。那先推估一個 Post 文章大概有多大呢？ 首先 Post 表裡面可能包含:
- content 估算，假設平均每篇 30 個中文字，UTF-8 裡大部分中文單字通常是 3 bytes，所以大約是 `90 bytes`

- post_id 表示文章的 id ; user_id 表示本篇文章作者 ; timestamp 表示文章發佈時間，因為可能需要排序。如果以上 field 都用 int 來存，那一個 int 是 32 個 bit，也就是 4 個 bytes
  
所以一整個 Post 儲存在 DB 內其 row 的佔用空間大概先估算是：
> `content + post_id + user_id + timestamp = 3*30 + 4 + 4 + 4 =~ 100 bytes`
       
故 Caching 要儲存的文字部分大小約略是： `100M * (2 / day) * 3 day * 100 bytes =  60000 MB = 60 GB`，但是如果特別只存熱門的文章，假設只有 20% 的文章可以被稱為熱門，這樣的話 caching storage 只需要 `12 GB`。

這樣算起來其實不算大，因為電腦 memory 基本上 `16 GB` 起跳了，若服務器還特別作為 in-memory storage 的話，單機有 `64 GB` 是非常常見的，更別說使用了集群架構。目前計算的 caching storage 值，單機都可以撐下來，緩存空間完全夠用。

### 引入 message queue 異步處理來 Pre-Built Timeline 並放到 Cache 中，會佔用多少 size

可能不需要存入整個 Post 完整資訊，因為只要拿到 post_id ，就可以透過 id 去讀表。假設每個人大約每天都刷 100 左右的貼文，故 Pre-Built 大約 100 則文章應該也就夠了，不會需要到 1000 則。那麼這個 caching storage 總共:

> `100M 活躍用戶 * 4 bytes * 100 則 = 40000MB = 40 GB`

承前面分析和假設，這個 timeline cache 佔的空間也不算大，儲存也是沒有問題，可能不需要特別使用 caching 集群架構設計。

### 每天網路 Bandwidth Estimate 估算
前面在計算 storage、cache 時，都沒有特別算圖片影片，是因為這些高機率都是用到外面的 CDN 和 Object storage 儲存，蠻多場合不會是自家公司自己搭建 CDN 和物件存儲服務器，所以暫時沒有考慮進去。但使用外面服務是會依據流量算花費的，這題就一併考慮一下。假設：
- 貼文一篇儲存在 db 的資料大小大概是 `100 bytes`
- 約 20% 的貼文文包含圖片，每張照片：`200 KB`
- 約 10% 的貼文文包含影片，每支影片：`2 MB`，10% ，其中只有 30% 的影片會被點開觀看
- 1 億個用戶，每個 user 每天會看 100 篇文章

計算結果爲 :

> - text: `100000k * 100 posts * (100 bytes / post) / 86400 = 100000k * 100 * 100 bytes / 100000 = 10000KB = 10MB`
> - image: `100,000k * 20 posts * (200 KB / post) / 86400 =  100000k * 20 * 200000 bytes / 100000 = 4000000KB = 4GB`
> - video: `100,000k * 10 posts * 0.3 * (2 MB / post) / 86400 =  100000k * 3 * 2000000 bytes / 100000 = 600000KB = 6GB`

承上得到需要的 total bandwith 約是 `10 GB/s`

{{< alert warning >}}
- 小型流量服務： `< 10 MB/s`
  
  通常自己主站就能扛，前提是使用者集中在同一地區、靜態資源不大

- 中型流量服務：`10 ~ 100 MB/s`
  
  開始要看情況，如果有很多圖片、前端靜態檔、下載內容，CDN 會很有價值

- 大型流量服務：`> 100 MB/s`
  多半應該上 CDN，因為這時候你考慮的不只是頻寬，還有延遲、尖峰、DDoS、跨區分發

- `1 GB/s 以上`： 通常幾乎一定要用 CDN
{{< /alert >}}

由上面分析，是會建議要加開 CDN 來減少 latency。

---

### 參考資料

- [系統設計基礎教學 - 導覽](https://www.explainthis.io/zh-hant/swe/system-design-tutorial)

- [系统设计面试 How to Design Twitter - System Design EP1 花花酱](https://www.youtube.com/watch?v=wqSnp49gGbg&list=PLS1xNNEa7rdzbw4bbuU0jTSFsgvRCfpqu&index=8)

- [system design 02 - scaling database](https://www.youtube.com/watch?v=odPx__SxLgI&t=64s)
