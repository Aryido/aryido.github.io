---
title: GCP - Google Kubernetes Engine 概述

author: Aryido

date: 2024-07-15T20:06:13+08:00

thumbnailImage: "/images/google-cloud/gke/gke-logo.jpg"

categories:
  - cloud
  - gcp

tags:
  - gcp-compute-service
  - kubernetes-service

comment: false

reward: false
---

<!--BODY-->

> Google Kubernetes Engine 簡稱是 GKE ，是一個由 Google 管理的 Kubernetes 開源容器編排平台的實現。因為 Kubernetes 的前身 Borg 本來就是 Google 內部的產品，憑藉著這樣的背景 GKE 號稱對於 Kubernetes 的支援跟擴展性跟其它雲端比起來會是最好的。對應其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **EKS**
> - Microsoft Azure : **AKS**
>
> 雲端化的 Kubernetes 簡單的說就是**把可在地端原生的 Kubernetes 放到雲端上運行**，由雲供應商幫助我們大幅簡化集群的設置管理與維運。由於只是讓雲供應商託管 Kubernetes ，最終差異也只是看雲供應商如何「預設」和「整合自己平台其他服務」至 Kubernetes 罷了，本質上 EKS、AKS、GKE 差異並不大，故基本上不推薦隨意更換 Kubernetes 服務的雲供應商或使用多雲，會考慮 GKE 的公司大多都是本來就在使用 GCP 雲端，更方便整合 Kubernetes 和 GCP 的各種服務。

<!--more-->

---

# GKE Architecture

{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/gke-architecture.jpg" >}}
GKE Cluster 由 「Control Plane」 和 「Nodes(也常稱為 Workers)」組成，雲端代管的優點是部分 GKE 的整個 Control Plane 組件會由雲端供應商來維護，因此我們只需關注 Node 的管理和設置即可。

而對於 Node 的管理， GKE 還分成兩種不同的 Mode，為「Autopilot Mode」、「Standard Mode」：

### Autopilot （recommended）

Autopilot 模式是一種**全託管**集群模式，是連 Node 設定都基本上託管給 GCP 管理， GCP 會根據 Pod 的數量自動擴展 Node 或優化 Node 利用率，從而降低成本，讓用戶專注於應用開發而無需手動管理集群。 Node 能請求的硬體種類也大致上被固定了，可參考 [Request compute classes](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-compute-classes)。
{{< alert warning >}}
Autopilot 模式下若要查看 Node 資訊只能使用 kubectl，並沒辦法 SSH 直接連線到 Node 裡面
{{< /alert >}}
{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/autopilot-standard.jpg" >}}
從 GCP Console 介面預設就是 Autopilot ，上圖為 GCP console 近期的新畫面，一開始想換成 Standard 時，我還花了點時間找畫面按鈕在哪裡...

### Standard

可以自己掌控 Node 的設置但比較繁瑣，藉由配製 [node pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools)來管理 nodes ，在 pool 中的 nodes 都是相同的 configuration，再進階還可以透過 `nodeSelector` 來指定 Pod 要部署到哪個 node pool 中。 在過去 node pool 一旦建立是無法修改的，但現在可以嘗試使用

```bash
gcloud container node-pools update POOL_NAME \
    --cluster CLUSTER_NAME \
    --machine-type MACHINE_TYPE \
    --disk-type DISK_TYPE \
    --disk-size DISK_SIZE
```

來修改 node pool 已配置的 machine-type 、 disk-type 和 disk-size。修改時 GKE 會使用 node pool 配置的升級策略，如果有配置 [blue-green upgrade policy](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pool-upgrade-strategies?hl=zh-cn#blue-green-upgrade-strategy)，在遷移失敗時， Pod 也能會回溯回原始節點。



{{< alert success >}}
Standard 模式下要查看 Node 資訊，可以使用 「kubectl」或者 「gcloud CLI」。由於 gcloud CLI 可以看到 Node 資訊，這其實也代表 Node 屬於在 GCP 雲端的管理範圍，那是管理在哪裡呢？

> 在 GCP VM 頁面可以看到 Node 的資訊，也有機會進一步 SSH 到 Node 內
{{< /alert >}}

##### Standard Mode 下 Zonal 與 Regional

Zonal 與 Regional 的設定，用於決定 Control Plane 和 Node 的 Availability：

- **Zonal** :

  Control Plane 僅會部署在一個 Zone 當中，當 Control Plane 所在的 Zone 發生 Outage 或是 Control Plane 更新，雖然 Workload 仍然會繼續運行，但會無法對 Node、 Workload 做設定，因為那時無法藉由 Control Plane 來溝通。

- **Regional** :
  - 由於 GKE 會在多個 Zone 部署 Control Plane，即便有單一 Zone Outage 又或是 Control Plane 更新的情境，還是可以正常訪問 Node 或 Workload
  - 這邊要注意的是 「node pool」 ，在 Regional cluster 下會自動進行 Replica 到每個區域下的 Zone ，故 node 數量會以**乘法**方式計算，**要注意專案的 quota**

{{< alert info >}}
費用計算方面：

- Autopilot 只針對正在執行的 Pod 的 compute resources 付費
- Standard 要為 Node 上的所有資源付費

由於 Autopilot 模式， GCP 會給出成本優化的建議，故可以先嘗試用一陣子 Autopilot 看看費用變化情形。
{{< /alert >}}

# Benefits and Drawbacks of GKE

Kubernetes 功能雖然強大，但背後最大的成本便是**維運複雜的架構**。傳統上在地端要架設好 Kubernetes，要準備一些硬體資源機器，然後組合成 Cluster，再來去做各式各樣的網路設定 ; 若中間遇到問題時還要 Troubleshooting ，環境要弄起來蠻花時間的，不僅需要具備 Container 相關的背景知識，同時也需具備網路架構知識。

> 這時雲端化 Kubernetes 有機會顯現一定的優勢...嗎？

「 K8S上雲能簡化運維，更節省人力」其實也找到蠻多文章直接否定了，簡單來說，似乎還沒有真的有研究直接表明，僅僅因爲遷移到雲端，就能實質性地大幅縮減運維團隊。

### 軟硬體設置

軟體方面在 GKE 中的 Kubernetes 版本會自動維護升級，並滿足安全性、可靠性和合規性，進一步還可以選擇維護時段和配置升級類型和範圍 ; 硬體方面由 google 平臺管理的 kubernetes ，可以自動修復 Node 以保持運行狀況正常和可用性，還有內建啟動日誌記錄和監控功能，以及 `>99%` [每月正常運行時間 SLO](https://cloud.google.com/kubernetes-engine/sla)。

{{< alert warning >}}
GKE 包含大多數 beta 版和穩定的 Kubernetes 功能，但如果想要嘗試還不太穩定的 alpha 階段 Kubernetes ，可以使用 alpha Standard cluster。
{{< /alert >}}

雖然風險是真的需要注意的，但爲某些微小的可能性而付出過高的溢價，雖然不能全盤否定，但總體上到底有沒有浪費，得就需要思考一下了。文章還有很貼切的比喻：「It's like paying a quarter of your house's value for earthquake insurance when you don't live anywhere near a fault line.」（沒有住在地震帶的附近卻花了四分之一的房價去買了地震保險）


### 雲端整合及安全性

GKE 在雲端整合 GCP 方面最常用的: 
- 透過 K8S-Ingress 或 K8S-Service 建立 Google Cloud Load Balancer
- 至此還可以進階設定整合 Load Balancer 的 Cloud CDN。

網路安全方面有 
- project 層級的 VPC Firewall
- Cluster 層級的 Private Cluster Mode 與 Network Policy
- 對於 IAM 可以配合 GCP 的 IAM 或者使用 Cluster 層級的 Role-Based Access Control(RABC) 來限制使用者的權限

### 提升服務可用性

GKE Cluster Autoscaler 是 GKE 對運算資源擴充的一種解決方案，會根據用戶提出的 Workload 需求自動調整機器的數量，以 Node Pool 為單位進行開啟，用戶僅需要設定 Autoscaling 的最高與最低機器數量。 一個 Cluster 聽說有超過一萬個 Node 可以配置都沒有問題，故一般來說中小型公司如果對自己業務成長真的不太確定時，很難規劃硬體需要購買的量，這時雲端的彈性就有一些用武之地。

{{< image classes="fancybox fig-100" src="/images/google-cloud/gke/gke-features.jpg" >}}

上面詳細地說出了上雲的好處，**但請注意**是在某些條件下上雲才會是適宜的選擇，例如：

- 當應用程序非常簡單且流量很低時，可能可以使用完全託管的服務，來減少基礎建設的維運

- 當負載不確定時，無法預判到底是需要 10 臺服務器還是 100 臺服務器的情況下，上雲可能會是好的選擇
  
- 當負載波動很大，例如一年中只有幾天要用到超多台服務器，那麼把購買許多服務器並閑置著就是沒有意義的，這種情況上雲也是個選擇

如果是增長穩定的中型公司，租用 IT 基礎設施可能就沒什麼必要。

---

# Practice

> 有一個 APP 在 GKE 上運行且有多個 replicas ，其 APP expose 了一個 TCP endpoint ; 且還有一個位於另一個 VPC 中的 VM ，並且兩個 VPC 之間 no overlapping IP ranges。若該 VM 需要連接到 GKE 上的 APP， Best-Practice 應該怎麼做？

在 GKE 中，創建一個[K8S-Service LoadBalancer 且 backend 是 Network Endpoint Group (NEG)](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balancing#create)，會需要在 K8S-Service YAML 的範例大概會是這樣:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ilb-svc
  annotations:
    networking.gke.io/load-balancer-type: "Internal"
spec:
  type: LoadBalancer
  externalTrafficPolicy: Cluster
  selector:
    app: ilb-deployment
  ports:
  - name: tcp-port
    protocol: TCP
    port: 80
    targetPort: 8080
```

由於題目中說兩個 VPC 沒有 overlapping Subnet，所以可以 **Peering** 在一起，故 VM 可以在另一個 VPC 中**直接使用**創建的 TCP Internal LoadBalancer 的 IP 和 GKE 內的 APP 做溝通。

以上創建了一個僅在內部網絡中可訪問的 LoadBalancer ，通過 VPC-Peering 可以實現不同 VPC 之間的內部通訊，以上算符合最小化配置工作的要求。

# Practice

> 在 GKE 上創建了 backend 和 frontend 兩個 App，希望確保 backend 的 Pod 被移動或重啟時 frontend 不會連接不到 backend，那應該怎麼做？
> - A. Create a service that groups your pods in the backend service, and tell your frontend pods to communicate through that service. **(O)**
> - B. Create a DNS entry with a fixed IP address that the frontend service can use to reach the backend service.
> - C. Assign static internal IP addresses that the frontend service can use to reach the backend pods.
> - D. Assign static external IP addresses that the frontend service can use to reach the backend pods.

A. 創建一個 Kubernetes Service 將後端服務的 Pod 管理起來讓前端 Pod 通過 Kubernetes Service 進行通訊，這是正確的，因為 Kubernetes Service 會提供穩定的網絡端點，即使後端 Pod 被移動或重啟，Kubernetes Service的 IP 地址仍然不會變，這確保了前端可無需關注後端 Pod 的變動。

B. DNS entry with a fixed IP address 不可行，因為後端 Pod 在重啟後 IP 可能會變動，這還是不能保證穩定的通訊。而 C.D. Pod 的生命周期可能很短，經常會被移動或重啟，故使用  static internal/external IP 都不符合 Kubernetes **動態調度**的設計初衷。

---

### 參考資料

- [Google Kubernetes Engine (GKE)
  Doc](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [淺談 Google Kubernetes Engine 架構與五大面向介紹](https://mile.cloud/zh/resources/blog/gke-k8s-introduction-Node-Pool-autoscaling_522)

- [GCP 百科 04：全面探索 Google Kubernetes Engine（GKE）：功能、特性和服務教學](https://tw.cocloud.com/zh-tw/blog/google-kubernetes-Engine)

- [Why we're leaving the cloud](https://world.hey.com/dhh/why-we-re-leaving-the-cloud-654b47e0)

- [Our cloud exit has already yielded $1m/year in savings](https://world.hey.com/dhh/our-cloud-exit-has-already-yielded-1m-year-in-savings-db358dea)