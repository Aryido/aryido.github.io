---
title: GCP - Google Kubernetes Engine 概述

author: Aryido

date: 2024-07-15T20:06:13+08:00

thumbnailImage: "/images/google-cloud/gke/gke-logo.jpg"

categories:
  - cloud
  - gcp

tags:

comment: false

reward: false
---

<!--BODY-->

> Google Kubernetes Engine 簡稱是 GKE ，是一個由 Google 管理的 Kubernetes 開源容器編排平台的實現。因為 Kubernetes 的前身 Borg 本來就是 Google 內部的產品，憑藉著這樣的背景，GKE 號稱對於 Kubernetes 的支援跟擴展性跟其它雲端比起來會是最好的。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **EKS**
> - Microsoft Azure : **AKS**
>
> 雲端化的 Kubernetes 簡單的說就是**把可在地端原生的 Kubernetes 放到雲端上運行**，由雲供應商幫助我們大幅簡化集群的設置管理與維運。由於只是讓雲供應商託管 Kubernetes ，最終差異也只是看雲供應商如何「預設」和「整合自己平台其他服務」至 k8s 罷了，本質上 EKS、AKS、GKE 差異並不大，故基本上不推薦更換 Kubernetes 服務的雲供應商或使用多雲，會考慮 GKE 的公司，大多都是只是思考怎樣更方便的整合 Kubernetes 和 GCP 的各種服務而已。

<!--more-->

---

# GKE Architecture

{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/gke-architecture.jpg" >}}
GKE Cluster 由 Control Plane 和 Nodes（也常稱為 Workers）組成，雲端代管的優點是部分 GKE 的組件例如 Control Plane會由雲端供應商來維護，因此我們只需關注 Node 的管理和設置即可。而對於 Node 的管理， GKE 還分成兩種不同的 Mode，為「Autopilot Mode」、「Standard Mode」：

- ### Autopilot （recommended）
  連 Node 都完全託管給 GCP 管理，會根據 Pod 中的 Pod 數量自動擴展 Node，能請求的硬體種類也大致上被固定，參考 [Request compute classes](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-compute-classes)，要查看 Node 資訊只能使用 kubectl，並沒辦法 SSH 直接連線到 Node 裡面。

{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/autopilot-standard.jpg" >}}
從 GCP Console 介面預設就是 Autopilot ，上圖為 gcp console 近期的新畫面，想換成 Standard 我還花了點時間找畫面按鈕在哪裡...

- ### Standard

  可以自己掌控 Node 的設置但比較繁瑣，藉由配製 [node pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools)來管理 nodes ，在 pool 中的 nodes 都是相同的 configuration，再進階還可以透過 `nodeSelector` 來指定 Pod 要部署到哪個 node pool 中。
  
  Standard mode 下要查看 Node 資訊可以使用 kubectl 和 gcloud CLI，由於 gcloud CLI 可以看到，其實也代表在 Google Cloud VM console 畫面都可以看到 Node 的資訊，進一步也有能力 SSH 到 Node 內。

  ##### Standard Mode 下 Zonal 與 Regional

  Zonal 與 Regional 之設定用於決定 Control Plane 和 Node 的 Availability：

  - **Zonal** : 
    - Control Plane 僅會部署在一個 Zone 當中，當 Control Plane 所在的 Zone 發生 Outage 或是 Control Plane 更新，雖然 Workload 仍然會繼續運行，但會無法對 Node、 Workload 做設定，因為那時無法藉由 Control Plane 來溝通。

  - **Regional** : 
    - 由於 GKE 會在多個 Zone 部署 Control Plane，即便有單一 Zone Outage 又或是 Control Plane 更新的情境，還是可以正常訪問 Node 或 Workload 。
    - 這邊要注意的是 node pool ，在 Regional cluster 下會自動 replica 到每個區域下的 Zone ，故 node 數量會以**乘法**方式計算，要注意專案的 quota。

{{< alert info >}}
費用計算方面：

- Autopilot 只針對正在執行的 Pod 的 compute resources 付費
- Standard 要為 Node 上的所有資源付費

由於 Autopilot 模式， GCP 會給出成本優化的建議，故可以先嘗試用一陣子 Autopilot 看看費用變化情形。
{{< /alert >}}


# Benefits of GKE

Kubernetes 功能雖然強大，但背後最大的成本便是**維運複雜的架構**。傳統上在地端要架設好 Kubernetes，要準備一些硬體資源機器，然後組合成 Cluster，再來去做各式各樣的網路設定，若中間遇到問題時還要 Troubleshooting ，環境要弄起來蠻花時間的，不僅需要具備 Container, Kubernetes 相關的背景知識，同時也需在具備企業級網路架構知識 ; 另外怎麼樣配置合適的 respurce 用量也是難題，這時雲端化 Kubernetes 就顯現一定的優勢：

### 軟硬體設置

軟體方面在 GKE 中的 Kubernetes 版本會自動維護升級，並滿足安全性、可靠性和合規性，進一步還可以選擇維護時段和配置升級類型和範圍 ; 硬體方面由 google 平臺管理的 kubernetes ，可以自動修復 Node 以保持運行狀況正常和可用性，還有內建啟動日誌記錄和監控功能，以及 `>99%` [每月正常運行時間 SLO](https://cloud.google.com/kubernetes-engine/sla)。

{{< alert info >}}
GKE 包含大多數 beta 版和穩定的 Kubernetes 功能，但如果想要嘗試還不太穩定的 alpha 階段 Kubernetes ，可以使用 alpha Standard cluster。
{{< /alert >}}

### 雲端整合及安全性

GKE 在雲端整合 GCP 方面最常用的: 是透過 K8S-Ingress 或 K8S-Service 建立 Google Cloud Load Balancer，至此還可以進階設定整合 Load Balancer 的 Cloud CDN。

網路安全方面有 project 層級的 VPC Firewall 和 Cluster 層級的 Private Cluster Mode 與 Network Policy ; 對於 IAM 可以配合 GCP 的 IAM 或者使用 Cluster 層級的 Role-Based Access Control（RABC） 來限制使用者的權限。

### 彈性架構提升服務可用性

GKE Cluster Autoscaler 是 GKE 對運算資源擴充的一種解決方案，會根據用戶提出的 Workload 需求自動調整機器的數量，以 Node Pool 為單位進行開啟，用戶僅需要設定 Autoscaling 的最高與最低機器數量。
最高一個 Cluster 可以有一萬五千個 Node，一般來說中小型公司如果對自己業務成長不太確定時，也很難規劃硬體需要購買的量，這時雲端的彈性就有一些用武之地。

{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/gke-features.jpg" >}}

---

### 參考資料

- [Google Kubernetes Engine (GKE)
  Doc](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [淺談 Google Kubernetes Engine 架構與五大面向介紹](https://mile.cloud/zh/resources/blog/gke-k8s-introduction-Node-Pool-autoscaling_522)

- [GCP 百科 04：全面探索 Google Kubernetes Engine（GKE）：功能、特性和服務教學](https://tw.cocloud.com/zh-tw/blog/google-kubernetes-Engine)
