---
title: Datadog V.S Grafana

author: Aryido

date: 2022-10-31T23:35:14+08:00

thumbnailImage: "/images/grafana/datadog.jpg"

categories:
- Grafana

tags:
- datadog

comment: false
reward: false
---
> Datadog 是 2010 年成立，以 infrastructure monitoring 起家。主要是整合多個雲平台如 GCP, AWS, Azure，讓工程師方便監測以進行 debug 。另外可用機器學習方式，將預先對可能發生異常狀況發出警示，是企業級的解決方案。

<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/grafana/datadog-introduction.jpg" >}}

---

# Datadog 簡易架構
使用者必須在 Datadog 雲中**註冊**才能開始使用。 Datadog 可以要監測的服務端上安裝 **Agent**，然後 Agent 會把相關的 Logs，Metrics 和 Traces 送到雲端 Datadog server上。
{{< image classes="fancybox fig-100" src="/images/grafana/agent.jpg" >}}

# Datadog 簡易使用情境
試想現在有許多服務運行在 AWS 上，且都是不同帳號。 雖然 AWS 有提供 CloudWatch 和  CloudTrail，可幫助我們檢視  Logs，Metrics 和 Traces 。但可能比較麻煩，會需要登入每個帳號個別查看。更進一步若是還有其他雲環境例如 GCP，要共同監控其實是有點麻煩的。

這時 Datadog 其實就提供了一個統一平台，方便我們在同一個 console 管理各式雲平台的監控畫面。Datadog 可快速建立監控儀表板。

---

# 其他相關服務系統簡介
Datadog 的主要功能就是 infrastructure monitoring ，現今關於軟體服務監控平台的選擇其實也有不少。以下探討開源的有:

{{< alert info >}}
## ELK or EFK
- Elastic Search: 儲存資料、資料搜尋、索引。
- Logstash or Fluentd: 多管道蒐集資料並送到指定位置
- Kibana: 可視化平台
{{< /alert >}}

{{< alert info >}}
## LFG、LPG

- Loki
- Fluentd or Promtail
- Grafana

{{< /alert >}}



---

## [Datadog V.S Grafana 系列](https://signoz.io/blog/datadog-vs-grafana/)

- Datadog 要在雲上註冊才能開始使用，grafana server 本地安裝完後啟動就可以使用了

- 隱私相關： Grafana 可以在自己的infra中管理，隱私可能比較好。DataDog 是第三方 SaaS 供應商，data會被發送到 DataDog 雲進行分析和可視化，但目前沒看到datadog相關的 [FedRAMP](https://www.arubanetworks.com/zh-hant/faq/what-is-fedramp/) 標準，且發到雲端上就要注意 [GDPR](https://zh.m.wikipedia.org/zh-tw/%E6%AD%90%E7%9B%9F%E4%B8%80%E8%88%AC%E8%B3%87%E6%96%99%E4%BF%9D%E8%AD%B7%E8%A6%8F%E7%AF%84) 問題

- DataDog 是企業監控工具，錢比較貴:
    - infrastructure enterprise monitoring starts at $23 per host per month
    - APM sand continuous profiler starts at $40 per host per month.
- Grafana 是 open source，但需要考慮 data 存儲和 network 的成本。但 GrafanaLabs 也提供每月 49 美元起的 grafanacloud

- Datadog在資料的持久化、索引index、儲存，都幫我們管理好了；grafana 則可能需要自建 Elastic Search 來管理。

- Grafana 在畫面客製化表現的會比 Datadog 還要方便

---

# Structured data 和 Unstructured data
- Schema on write 就是 Structured data化：
{{< alert info >}}
將 log 存入 database 時就進行結構化。優點是方便未來每一次 query 都可以快速得出結果。但存入效率就會變慢。
{{< /alert >}}


- Schema on read 則是 Unstructured data 化：
data化：
{{< alert info >}}
將 log高速寫入 database ，但是只有在每次調用時才進行結構化。但是 query 大數據效率低下。
{{< /alert >}}

{{< image classes="fancybox fig-100" src="/images/grafana/data-management.jpg" >}}

---

# Reference

- [Datadog VS Grafana](https://sa123.cc/budedtd7jr4y56vywlab.html)
- [what is datadog](https://cloud.ofweek.com/news/2021-12/ART-178803-8110-30540222.html)
