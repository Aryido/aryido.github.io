---
title: "淺談影音後端知識"

author: Aryido

date: 2025-07-11T15:48:45+08:00

thumbnailImage: "/images/video-streaming/streaming-logo.jpg"

categories:
  - product

tags:
  - video

comment: false

reward: false
---

<!--BODY-->

> 雖然影音串流已經融入生活中，但自己對於這一技術部份的專有名詞還有蠻多不了解的。例如說「 MOV vs MP4」、「 1080P」、「 H.264 」 等等，這些名詞基本上好像都看過，但實際上對於其定義是蠻模糊的。而更進階的 Streaming Protocol (串流協議)，其從定義上來說是指 Client 透過 Streaming Protocol 向 Server 請求檔案，而這些檔案是 Server 把經過壓縮處理切割的影片資料，以「小塊 Chunks」、「小封包 Packets」或稱「資料流 Stream」的格式傳給 Client 端，Client 在接收到檔案片段後就可以開始播放了，無需等待完整影片下載完，那這是怎麼達成的呢 ？以下就把我看的資料簡單整理一下。 
 
<!--more-->

---

隨機點開了自己 MAC 中的一個影片，稍微看一下檔案的 info，會有幾個和該影片相關的 metadata :
- Kind: MPEG-4 movie
- Codecs: H.264
- Dimensions: 3840 * 2160

會選這幾個關鍵字，是因為 backend 在處理影片壓縮時，經常會出現幾個 keyword ，接下來就來調查一下各個名詞術語的意思吧 ！

# 簡談 MPEG-4

首先一開始遇到的 MPEG-4 這個詞，[其發音是 m-peg](https://www.youtube.com/watch?v=V0DJLoi-2yM)，全稱 Moving Picture Experts Group 簡稱 MPEG。它其實就和 MP4 蠻難區分的，那到底 [MPEG-4 和 MP4 有什麼區別呢？](https://www.cloudflare.com/learning/video/what-is-mp4/) 在看完許多文章所述后，整理這兩個的關係是：

[**MPEG-4** 是影片的 **Encoding standard(編碼標準)**](https://www.mpeg.org/standards/MPEG-4/)，有分為多個 part ，例如說：
- MPEG-4 第 3 部分: **AAC  audio codec 聲音編解碼**
- MPEG-4 第 10 部分: **H.264 video codec 影片的邊解碼**
- MPEG-4 第 14 部分: **MP4 container 容器**
    - 比較專業的術語是稱為 Digital Container File(簡稱 Container)
    - 或者說是直接說 MP4 是一種 **Digital Container Format(簡稱 Format， 檔案格式)**

目前**影音檔案的 file extensions 經常使用其 Container 來命名**，也就是我們常看到檔名後面的 `.mp4`，那雖然 `.mp4` 是官方文件認定的 MP4 擴展名，但也有其他擴展名如 `.m4a` 等等，另外 `.mov` 也是其他有名的 Container。 

雖然 MP4 基本上都會採用 MPEG-4 標準進行 Encoding 編碼，但要注意 Container 只是一個外殼的概念，只要裡面的編解碼器(codec)**不同**，那影音播放軟體可能就會無法正確播放，所以只用副檔名來判斷影音的格式會不太精確。

{{< alert warning >}}
承上可以知道「MP4」和「MPEG-4」這兩個術語並不是指相同的東西，但令人頭痛的是，其實還是蠻多人或文章把兩個一律都稱為 MP4 混用在一起... 
{{< /alert >}}


接下來進階說明一下在上面描述中的一些術語例如 **Encoding Standard** 和 **Digital Container File** 是什麼？以下繼續介紹 : 

### Digital Container File

[Digital Container File](https://www.cloudflare.com/en-ca/learning/video/mov-vs-mp4/) (有時也會被簡稱為 Container)，它是一個把 Metadata 和 Data 封裝在一起的 Wrapper ，不同的檔案格式代表不同的 [Digital Container File Format（有時也會被簡稱為 Digital Container Format 、 Container Format 甚至單純 Format）](https://www.cloudflare.com/learning/video/video-encoding-formats/)，都會有不同的方式來組合影像和聲音資料。 Container 架構主要包含兩部分：

- **Data**: 內容可能有 Audio Codec (例如 AAC 碼格式) 或者 Video Codec (例如 H.264 編碼格式) 這兩種資料 ; 若是單純音樂檔就只會有 Audio Codec
- **Metadata**: 這部分能提供支援哪些「影片播放軟體」; 也告訴軟體該如何組織 Audio Codec 和 Video Codec 並呈現出來，此外還可以提供其他資訊，例如字幕等等

Container 是將 「Metadata」、「編碼的音訊串流(Audio Codec)」、「編碼的影片串流(Video Codec)」 組合 Wrapper 到一起，只有在 Codec 和 Container 都與影片播放程式相容時才能正常播放該 Video File。常見的 Container 有 : 
- {{< alert success >}}
**MP4** 是算是最廣泛使用的 Digital Container ，是國際標準(International Standard)的，所以相容於更多串流媒體協定，而 MP4 檔案通常壓縮程度也叫高，因此比其他類型的檔案 Size 更小，其採用常見的編碼標準 MPEG-4 進行 Encoding 。
{{< /alert >}}

- {{< alert success >}}
**MOV** 是 Apple 開發的，專門為 QuickTime Player 搭配使用，也採用常見的編碼標準 MPEG-4 進行 Encoding ，但檔案壓縮率不如 MP4
{{< /alert >}}

- {{< alert success >}}
**WebM** 是 Google 開發的開源 digital container file ，受 Android 裝置的支持
{{< /alert >}}

# 簡談 Codec
在 Digital Container File 敘述中，也有提到一個術語叫[ codec (coder/decoder)，發音可以記憶一下 /ˈkoʊˌdɛk/](https://www.youtube.com/watch?v=OQlK_oaSrUo&t=2s)，翻譯是「編解碼器」，代表壓縮和解壓縮資料方式。MP4、MOV 這些常見的 Container ，其採用的 Video Encoding 經常是 H.264，為目前最常使用的影片壓縮標準。

{{< alert warning >}}
在蠻多文章中，Video compression(影片壓縮)，也稱為 Video encoding(影片編碼)，兩個是指同一件事情...
{{< /alert >}}

在串流播放(Streaming)中，最重要的是 encoding 要與盡可能的，**相容於**不同的播放器，以便所有用戶都能觀看。例如 「Cloudflare Stream」這個產品是使用 **H.264** 進行編碼，該編碼相容非常多串流協議(Streaming Protocol)，也支援 adaptive bitrate streaming（從 360p 到 1080p 可選畫質）。 

接下來介紹一下一直都有提到的 `H.264`，其全名是[ Advanced Video Coding(高級視訊編碼) 簡稱 AVC ](https://www.cloudflare.com/learning/video/what-is-h264-avc/)，是由官方組織所共同制定提出的，故非常廣泛使用。其進階 **H.265** 全名是 High Efficiency Video Coding(高效能視訊編碼) 簡稱 HEVC，雖然提供比 H.264 更佳的畫質，此編解碼器對於 8K 影片很受歡迎，但支援的瀏覽器仍有限，所以目前主流還是 H.264。

# 簡談影片解析度
解析度、畫質常看到的英文翻譯有 「Resolution」與「Dimensions」，這兩個經查詢過後的結論，應該可以說基本上是一樣的，但 Dimensions 會比較接近說實際大小如「寬×高」 為 3840 × 2160 而 Resolution 是說 2160p。接下來說明一下我們常看到的如 720p、1080p 是什麼意思：

{{< image classes="fancybox fig-100" src="/images/video-streaming/1080p.jpg" >}}

首先舉例 1080p ，其中的「 p 」是來自 progressive scan，原意思為「逐 row 掃描」，是指垂直方向有 1080 條水平線，但水平像素並無嚴格規範。 通常 1080p 的畫面會搭配 16:9 的寬高比，故解析度為 1920×1080 ，以下附上簡單表格：


| 解析度 | 寬×高 | 總像素 | 顯示面積倍率 (與前一格式相比) | 解析度規格名稱 |
|:------:|:----------------------------:|:-------:|:--------------------------------:|:-------------------------:|
| 🟢 **480p** | 854 × 480 (16:9)  | 409,920 |  | **ED** (Enhanced-Definition) |
|  | 640 × 480 (4:3) | 307,200 |  |  |
| 🔵 **720p** | 1280 × 720 | 921,600 | **2.25X** | **HD** (High Definition) |
| 🟣 **1080p** | 1920 × 1080 | 2,073,600 | **2.25X** | **FHD** (Full High-Definition) |
| 🔴 **4K** | ***3840 × 2160*** | 8,294,400 | **4X** | **UHD** (Ultra High-Definition) |


{{< alert danger >}}
比較特別要注意的是所謂的 4k，是指**水平橫向顯示大約 4096 像素**左右的解析度。由於是「水平橫向顯示」不是原始垂直方向的定義，所以 4k 並不是指 4000p。

更進一步來說，其實主流的 4K 標準是 「3840*2160」，所以其實是 2160p 才是正確的，電影行業才使用 4096×2160（DCI 4K）。

~~那就暫且不討論為什麼突然改看橫向有詐騙嫌疑... ，且 3840 沒超過 4000 好個大約，所以叫 4K ...~~
{{< /alert >}}

---


# Streaming Protocol (串流協議)

**H.264** 編碼相容非常多串流協議(Streaming Protocol)，那 串流協議 是什麼呢？首先說明 Streaming 意思，這是一種無需實際下載即可觀看影片或收聽音訊內容的方法，而 Protocol 就算是針對這需求的實現，以下舉例一些串流協議：
- Real Time Streaming Protocol (RTSP)
- **HTTP live streaming (HLS)**
- HTTP dynamic streaming (HDS)
- dynamic adaptive streaming over HTTP (MPEG-DASH)

比較有名的 AWS 的產品 MediaConvert ，其官方範例使用的 Protocol 是 HLS，也可以關注一下：

### [HLS （HTTP Live Streaming）](https://www.cloudflare.com/learning/video/what-is-http-live-streaming/)

HLS 為 Apple 於 2009 年時提出的基於 HTTP(S) 的 Streaming Protocol，算是最廣泛使用的，雖然稱為 "live"「即時」串流，但也可用於 on-demand 點播串流。原理如下：

- 將檔案分割成多個片段，例如每 10 秒一個片段，其產生的 file 副檔名為 「 .ts 」

- 有一個 index file 其副檔名是「 .m3u 」，其作用是記錄分割小檔案的播放順序與位址

  {{< alert info >}}
  常見的 .m3u file 的類型有 「.m3u（Windows-1252）」和 「.m3u8（UTF-8）」
  {{< /alert >}}

- Client 向 Server 請求時， Server 會回傳對應的 `.m3u file`

- Client 拿到 `.m3u file` 後，會解析檔案，然後按照播放列表逐一向 Server 請求片段 `.ts file`

- 當 `.m3u file` 內記載的片段下載完後，Client 會再請求下一個 `.m3u file` ，直至影片播放結束

{{< alert warning >}}
HLS 必須使用 H.264 或 H.265 編碼
{{< /alert >}}

HLS 相較於其他 Streaming Protocol 的優勢之一是 adaptive bitrate streaming(自適應位元率串流)，代表能夠根據網路狀況的變化調整影片品質，若網路狀況變差，影片播放器會偵測到這種情況，就會改拉取較低影片解析度影片，確保影片繼續播放。


### DASH (Dynamic Adaptive Streaming over HTTP)

DASH 會把一個影片會被編碼成多個不同的版本，每一個版本都有不同的 Bitrate 對應到不同的畫質， Client 可以根據自己的頻寬來**動態選擇**畫質 ; 當然如果 Client 當前已經緩衝了很長一部份的影片，而且頻寬也很高時，DASH 也能自動選擇高 Bitrate 的版本

還有很多[其他串流協議](https://blendvision.com/zh-tw/blog-zh/live-streaming-protocols-comparison)，這邊就不列出來了。


---

### 參考資料

- [什麼是串流與 HLS 串流協議？](https://zoejoyuliao.medium.com/%E7%94%A8-aws-lambda-aws-mediaconvert-%E5%AF%A6%E7%8F%BE%E5%BD%B1%E7%89%87%E8%BD%89%E6%AA%94%E8%88%87%E4%B8%B2%E6%B5%81-%E4%B8%80-%E4%BB%80%E9%BA%BC%E6%98%AF%E4%B8%B2%E6%B5%81%E8%88%87-hls-72c8a7b9201)

- [串流技術是什麼？一次搞懂 8 個影音串流常見名詞，順利接軌影音時代](https://blendvision.com/zh-tw/blog-zh/streaming-technology-explained)

- [Computer Networking — 2.6 Video Streaming and Content Distribution Networks](https://hackmd.io/@kaeteyaruyo/computer-networking-2-6)

- [影片解析度的比較：1080p、720p、480p 和 4K【如何選擇最適合你的需求】](https://www.movavi.com/zh/learning-portal/1080p-720p-4k-resolution-comparison.html)

- [影片格式選擇障礙? 先來了解一下各種常見的影片編碼格式](https://jacksonlin.net/20221230-how-to-choose-format/?srsltid=AfmBOorJGhkrnpbVAzN3b22qzbQLMq55wHnT0vFLwaAwl_Sf-tzAGHQs)


