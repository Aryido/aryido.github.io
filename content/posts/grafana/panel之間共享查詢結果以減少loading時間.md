---
title: "Grafana: panel之間共享查詢結果以減少loading時間"

author: Aryido

date: 2022-09-08T22:18:02+08:00

thumbnailImage: "/images/grafana/grafana-icon.jpg"

categories:
- Grafana

tags:
- Grafana

comment: false
reward: false
---
> Grafana 的 panel 會連接 datasource 並發出請求。 故當我們向 dashboard 中添加多個 panel 時，會發出更多的請求，這可能會導致需要更長的時間來loading資料。

<!--more-->

但我們真的需要這麼多的請求嗎 ?? 是不是有些 panel 的 datasource 和其 Query 語法根本都一樣，只是視覺化部分不同罷了。

# Share query results with another panel
{{< alert success >}}
Grafana 提供對不同 panel ，但可以共用同一查詢結果。即 Grafana 允許將一個 panel 的查詢結果用於同一 dashboard 中的任何其他 panel。

跨 panel 共享查詢結果可減少對 datasource 的查詢次數，從而提高 dashboard 的性能。
{{< /alert >}}
這樣設定後， Grafana 只會在一個 panel 內發送查詢，而其他 panel 是使用查詢結果呈現。

{{< image classes="fancybox fig-100" src="/images/grafana/share-query-results.jpg" >}}

查查看自己現有的 dashboard 吧。如果有查詢相同的 panel ，可以使用來增進效能!

---
# Reference
- [Learn Grafana: Share query results between panels to reduce load time](
https://grafana.com/blog/2020/10/14/learn-grafana-share-query-results-between-panels-to-reduce-load-time/)

- [Document](https://grafana.com/docs/grafana/v9.0/panels/query-a-data-source/share-query/)