---
title: 建立自己的 grafana dashboard plugin - 1

description: ""

author: Aryido

date: 2022-09-04T11:17:02+08:00

thumbnailImage: "/images/grafana/grafana-icon.jpg"

categories:
- Grafana

tags:
- Grafana

comment: false
reward: false
---

 > 雖然 Grafana 已經內置了多種類型的dashboard，但有時候可能會覺得官方或其他免費開源plugin，提供的功能不太夠。這時就需要建立自己的dashboard。

<!--more-->

## Prerequisites
### Grafana >=7.0
grafana 於 7.0 版，整合了`Time series` 和 `Table structures` 為一個更廣義的data structure，稱[Data frames](https://grafana.com/docs/grafana/latest/developers/plugins/data-frames/)

### NodeJS >=14
panel 部分都由typescript撰寫，需要編譯成js
### yarn
前端打包工具(類似maven)


## 環境架設
grafana有提供一個CLI application，工具叫做grafana-toolkit。運用該工具，在create一個新的plugin後，會把環境和基礎模板設定好，可以更方便我們打包自製plugin和建立環境
![](https://i.imgur.com/ChJrAQw.png)


## plugin 模板分析
grafana在讀取plugin時，會參考兩個重要檔案。

    1. plugin.json
    2. module.ts(js)

### plugin.json
當 Grafana server啟動時，它會掃描Grafana server內的plugin目錄(預設/var/lib/grafana/plugins)下的一層子目錄，查找包含 plugin.json的目錄。

舉例一些plugin.json強制需要的屬性：

- type
告訴Grafana是什麼類型的plugin。只有三種。app, datasource, panel.

- name
在登入grafana後，製作panel時右邊視覺化選單列表中，看到的名稱。

- id
Unique name of the plugin. If the plugin is published on grafana.com, then the plugin id has to follow the naming conventions.。

[plugin.json 其他詳細介紹](https://grafana.com/docs/grafana/v7.5/developers/plugins/metadata/)

### module.ts(module.js)
藉由plugin.json發現plugin後，Grafana 會加載 module.ts(js) 文件，是plugin的入口點。 module.ts(js)會expose plugin 的 implementation。

module.ts(js) 需要extends GrafanaPlugin 的object，並且該object可以是：
- PanelPlugin
- DataSourcePlugin
- AppPlugin

之後就可以開始自製plugin了
---