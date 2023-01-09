---
title: Apple M1 作業系統坑 - cloud run 出現錯誤

author: Aryido

date: 2023-01-08T21:54:38+08:00

thumbnailImage: "/images/google-cloud/cloud-run.jpg"

categories:
- os

tags:
- gcp

comment: false

reward: false
---
<!--BODY-->
> Cloud Run 是 Google 的 Serverless 產品，讓我們不用管理基礎 infra 也能建置容器，並會根據流量自動調整資源，且只依據實際使用的資源收費。 這邊特別注意一下，目前 Cloud Run 似乎還沒支持 ARM 格式的 image，故有使用 M1 筆電包 docker image 要特別注意一下，這會出現不可預期的 bug !

<!--more-->

---
# Cloud Run 簡介
Google Cloud 上主要有三種 Serverless 服務，分別為
- Cloud Functions
- App Engine
- Cloud Run

若服務需要更多的靈活性，且希望能並行運作 Containers 來處理大量的服務或作業，則建議使用 Cloud Run。 Cloud Run 即是一套基於 Knative 的全代管 Serverless 服務，不像 GKE 需管理 worker node 及複雜的 yaml 檔案部署設定，Cloud Run 更加簡單。

{{< alert info >}}
Cloud Run 有人把它當作是一個快速驗證的服務。將 build 好的 artifact 透過 Cloud Run 執行，就可初步確定 Container 的行為狀態，而無須考慮 Kubernetes cluster 及 yaml 檔。
{{< /alert >}}

---

# Cloud Run 的功用
1. Cloud Run 支援的 Container images，**請特別注意， image 必須針對 Linux 64 位編譯件。** Cloud Run  在官方文件中，有明確支持 Linux x86_64 ABI 格式。

{{< alert warning >}}
[Supported languages and images](https://cloud.google.com/run/docs/container-contract#languages)

Cloud Run 接受 Docker Image Manifest V2、Schema 1、Schema 2、 and OCI image，目前不支持用於多架構映像的 Manifest 列表。
{{< /alert >}}

2. 可直接與 Cloud Monitoring、Cloud Logging、Cloud Trace 和 Error Reporting 整合。

3. 只需要支付程式運作期間的費用，**計費單位為 100 毫秒**。

4. 會依據流量自動調整容器主機的資源配置，從零擴展至多個資源或縮小至零（Scale up or down from zero to mulitples）

Serverless 是一種 NoOps 的解決方式，可以讓只有 developer 的公司也能方便進行部屬，缺點就是少了建置基礎環境的彈性。

---

# cloud run 出現錯誤
自己在本地電腦 build docker image 後，推到 contianer registry ， 但是部屬至 cloud run 時出現錯誤資訊 :
```
Cloud Run error: Container failed to start.

Failed to start and then listen on the port defined by the PORT environment variable.

Logs for this revision might contain more information.
```
繼續往 cloud run 的 log 錯誤訊息搜尋，會看到 :
```
Failed to create init process: failed to load startup_script.sh: exec format error
```
咦 ? 這個 ```startup_script.sh``` 是我上傳的，但 cloud run 無法執行 !?  上網調查後，發現是 **Apple M1 作業系統坑** ，也就是往上面一直強調的 **Cloud Run 目前不支援 ARM 類型的 Container images**。

因為自己電腦是 M1 ，自己測試是本地包 docker ，所以發生這樣的錯誤...

{{< alert info >}}
不同 CPU 架構的電腦，會需要使用不同的 Docker Image。 Docker 是跑在 OS 之上的，不同的 OS 跟 CPU 架構，當然就要有不同的 Image，
{{< /alert >}}

{{< alert warning >}}
既然不同架構的 Image 不能混用，那就表示我們執行 docker pull 指令時，docker 會自動辨識並抓取對應架構的 Image 來使用，是怎麼做到的呢 ?

**docker 透過 manifest 這個清單，來決定要使用哪一個 image !**

可用用看 ```docker manifest inspect``` 指令看看 Image 不同的架構清單 !
{{< /alert >}}

---
# Reference
[M1 使用本地 docker push 到 cloud run 出現錯誤](https://penueling.com/%e7%b7%9a%e4%b8%8a%e5%ad%b8%e7%bf%92/m1-%e4%bd%bf%e7%94%a8%e6%9c%ac%e5%9c%b0-docker-push-%e5%88%b0-cloud-run-%e5%87%ba%e7%8f%be%e9%8c%af%e8%aa%a4/)