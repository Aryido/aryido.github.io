---
title: 建立自己的 grafana dashboard plugin - 2

description: ""

author: Aryido

date: 2022-09-04T11:17:52+08:00

thumbnailImage: "/images/grafana/grafana-icon.jpg"

categories:
- monitoring

comment: false

reward: false

---

> 參考[官方教學](https://grafana.com/tutorials/build-a-panel-plugin/)遇到的一些小問題...

<!--more-->

# 依照官方教學步驟，會遇到的問題

    1. docker找不到自己掛載的plugin...

- 打包完成的plugin後，會產生一個dist資料夾，這個資料夾的內容才是要掛載到/var/lib/grafana/plugins的主體，如果多放一層目錄，grafana server會讀取不到。
解決方式:
    ![](https://i.imgur.com/5KtvIq1.png)

---
    1. grafana現在所有的plugin都必須簽章才會讀到

自製的plugin預設都是未簽章的。故可以照官方教學簽章自己的plugin([簽章教學](https://grafana.com/docs/grafana/latest/developers/plugins/sign-a-plugin/))，或者需要再啟動grafana server時，環境變數要加上Allow unsigned plugins。

![](https://i.imgur.com/YhSMCtv.png)

(開發環境測試才建議直接Allow unsigned plugins)
![](https://i.imgur.com/OctrLLF.png)

![](https://i.imgur.com/t34XzPh.png)

---

# grafana-jsontext-panel

## 上傳自製panel-plugin步驟(用在私人專案暫時的方法)

### step1.
執行
```bash
   yarn build
```
當前專案目錄會產生一個dist目錄，內部都是編譯好的檔案，然後把這些檔案打包成zip
(注意! 不要多一層目錄)

### step2.
把zip上傳到github release 提供下載

### step3.
在docker-compose加上github release路徑
例如:
```
GF_INSTALL_PLUGINS:"https://github.com/xxxx/XXXX.zip;jsontext"
```

### step4.
因為plugin還沒簽章，故在docker-compose加上認證未簽章認證，使得grafana server可以讀取該plugin
```
GF_PLUGINS_ALLOW_LOADING_UNSIGNED_PLUGINS: jsontext
```

### step5.
檢查grafana server啟動訊息
至grafana網頁內看plugin

# Panel 製作須知

從 Grafana 6.x 開始，panel是 [ReactJS components](https://reactjs.org/docs/components-and-props.html)。在 Grafana 6.0 之前，plugin是用 AngularJS 編寫的。 雖然還是可以用AngularJS，但現在強烈建議是使用 ReactJS 編寫。
---