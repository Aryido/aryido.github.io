---
title: "Opentelemetry 緣起簡介"

author: Aryido

date: 2024-03-17T11:17:16+08:00

thumbnailImage: "/images/monitoring/opentelemetry-logo.jpg"

categories:
- develop

comment: false

reward: false

---
<!--BODY-->
> OpenTelemetry ，也稱為 OTel，是 Cloud Native Computing Foundation(CNCF) 下的一個開源專案，為一供應商中立(vendor-neutral)的**可觀測性框架**，目的是提供一個**標準化**Observability 的 pluggable 框架，解決 telemetry data 的模型定義、檢測、採集、處理、輸出等一致性問題。
> {{< image classes="fancybox fig-100" src="/images/monitoring/opentelemetry-architecture.jpg" >}}
> 借助 OpenTelemetry ，可以從各種來源收集 telemetry data ，但對於資料的存儲和可視化是留給其他工具的，本身並不提供儲存，只能將其發送到其他的服務來存儲，如 Prometheus、或其他雲端廠商服務。
<!--more-->

---

# 前言
伴隨著 Distributed system 的普及，一個 request 會經過的 service 或 host 也越來越多，而其中任何一個的環節的失敗、效能問題，都會對 user 造成影響。但針對這種**長請求鏈**的追蹤監控也是有難度的，因為要讓系統支援分散式追蹤，會需要以下實現如 :
- 可能要在 **process 內**或**不同 process 間**，甚至**不同主機**，進行追蹤資料的傳遞
- 也需要對其他額外 component 例如 NGINX、Redis 等開源服務，進行分布式追蹤的支持
- 可能須在服務內引入 gRPC 等額外功能來作通訊

為了解決這些問題，相關產品也開始出現，比較早的如 : **google 內部自行研發的 Dapper**，來自 2010 年發佈的論文，為**分佈式鏈路追蹤**的開端；再來是後續基於谷歌 Dapper 協議發展的開源產品如 : **2015 Uber 的 Jaeger**、**Twitter 的 Zipkin**。 但每個產品都各自有一套資料擷取標準和 SDK，其**每一個實作都不太一樣**。 為了解決這個問題， OpenTracing 和 OpenCensus 等規範分別被提倡了出來。

{{< image classes="fancybox fig-100" src="/images/monitoring/opentelemetry-history.jpg" >}}

---

# OpenTracing
OpenTracing 制定了一套無關廠商、平台的協議標準，使得開發人員能夠方便的添加或更換底層的實現。故 2016 年 CNCF 正式接納 OpenTracing ，規範主要是 :
- API 接口標準化：只需要呼叫相關 API 接口，就可被任何實作這套接口的追蹤後台支援。

- 對追蹤最小單位定義標準化 **Span**
- 進程間追蹤資料傳遞方式標準化
- 多語言應用支援的標準化

可以發現 OpenTracing 規範中，主要是釐清了 **Trace**、**Span** 的關係和定義。

{{< image classes="fancybox fig-100" src="/images/monitoring/opentracing-model.jpg" >}}

---

# OpenCensus
OpenTracing 本身出現的更早且更流行，那爲什麼 google 還會 2017 再提出個 OpenCensus 呢？

> OpenCensus 的最初目標只是爲了把 Metrics collector ，和一些 GO 工具整合。但最後也想把其它各種語言的相關 collector 都統一完成，因此不僅做了 Metrics 基礎指標監控，也做了一些分佈式追蹤的實現。

OpenCensus 主要提供了 Agent ，Agent 內部有 Collector 和 Exporters，實現了**跨服務擷取 Span**、和 Metrics 監測，並規範 :
- 標準化通訊協定和 API，用於處理 Metrics 和 Trace；而 OpenTracing 只規範 Trace
- 與 RPC 框架的整合，並自己實現 Agent 來支援 OpenCensus
- 整合 Collector ，還有開源並支援三方儲存和分析工具

{{< image classes="fancybox fig-100" src="/images/monitoring/opencensus-agent.jpg" >}}

---

# OpenTelemetry
OpenTracing 和 OpenCensus 都是為解決一些監控問題，而獨立開發的分散式追蹤 project，所以二者的職責範圍有所重合，且由於兩者各有優點，例如：

- OpenTracing 支援語言更多、對系統耦合性更低；且之前對於 trace 的定義非常完整
- OpenCensus 補強 Metrics 部分、且分散式 tracing 同時到基礎設施層都進行了支援

為此大家開始思考是否將 OpenTracing 和 OpenCensus 結合 ? 並再加上能夠支援 Log 日誌相關功能，形成新的可觀測資料的專案，結合每個 project 的優點應該很不錯的。

{{< image classes="fancybox fig-100" src="/images/monitoring/opencensus+opentracing.jpg" >}}

為了更好的將 Traces、Metrics 和 Logs 融合在一起，2019 年 OpenTelemetry 誕生了，也是作為 CNCF 項目，旨在爲所有類型的**可觀測數據** Metrics、Tracing、Logs 定義一個**標準統一**。

{{< alert info >}}
OpenTelemetry 在 AWS、Google、Microsoft 等大廠的貢獻下得到了極大的發展，以至於它成爲按活躍度和貢獻者排名第二的 CNCF 項目，僅次於 Kubernetes 。
{{< /alert >}}


---
### 參考資料

- [初探 OpenTelemetry 工具組：蒐集遙測數據的新標準](https://www.youtube.com/watch?v=PT-Bjs6iCug)

- [opentelemetry doc](https://opentelemetry.io/)

- [從Opentracing、OpenCensus 到 OpenTelemetry，看可觀測資料標準演進史](https://www.cnblogs.com/alisystemsoftware/p/16143318.html)

- [OpenTelemetry - 結束分佈式追蹤的江湖之亂](https://www.readfog.com/a/1642431511708930048)
