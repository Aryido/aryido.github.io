---
title: "Kubernetes: Container Network Interface"

author: Aryido

date: 2023-06-25T21:12:29+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> Container Network Interface（CNI）的基本思想，是先創建好網路命名空間 （netns） ，然後調用外掛程式，替這個網路命名空間配置網路，之後再啟動容器內的進程。 CNI 通過 Json Schema 描述當前容器網路的配置，實現標準化。專注於
> - 創建容器時分配 IP、網段等等
> - 容器被回收時刪除網路資源
>
> CNI 作為 Kubernetes 和底層網路之間的一個抽象存在，遮罩了底層網路實現的細節、實現了 Kubernetes 和具體網路實現方案的解耦。

<!--more-->

---

Kubernetes 中有運行著 Pod 的時後，可以通過 ssh 登錄到 cluster 中的**任何一個節點**上，然後進行如下做，都是可以做到的 !
- 使用 ```kubectl get pods``` ，都能夠拿到 Pod 的資訊，不論這個 Pod 是被分配到哪個節點上
- 另外也可以使用諸如 curl 之類的工具，向任意 Pod IP 地址發出請求，不論這個 Pod 是被分配到哪個節點上，都可以成功通信

以上是整個 k8s 預設的網路架構就能達到的事情。

## 網路模型
Kubernetes 網路模型遵循的一些核心規範原則，稱 **IP-Per-Pod 模型** :

- 每個 Pod 都擁有一個唯一且獨立的 IP 位址，且不論在 Pod 內部還是外部，IP 和埠資訊都是一致的
- 只要 Pod 在同一個 cluster 內，即使 Pod 不在同一個 node 節點上，也可以通過單純 Pod IP 的方式直接訪問
- 任意節點上的 Pod 可以在不藉助 NAT 的情況下與任意節點上的任意 Pod 進行通信
- 節點上的代理（諸如系統守護進程、kubelet等）可以在不藉助 NAT的 情況下與該節點上的任意Pod進行通信

這個模型可以很好的相容現有的應用架構，類似像有虛擬機，且每個有自己獨立的 IP ，彼此之間可以還可以相互通信。

{{< alert info >}}
因為和原始 VM 網路架構類似，可以比較好的相容遷移到Kubernetes集群上，降低成本和風險。
{{< /alert >}}

---
## Container Network Interface 歷史簡介
因為並非所有的 Kubernetes 集群都部署在 GCE、AWS、Azure 等有良好的 VPC 的雲端架構上，且 Kubernetes 僅關注**容器編排**的部分，網路管理並不是它原本分內的。故在私有雲的部署方案也日漸增多情況下，如何保證集群網路可以滿足 **IP-Per-Pod 模型**要求就成為了一個首要問題。

起初 Kubernetes 通過開發 Kubenet 來實現網路管理功能。 Kubenet 是一個基礎的網路外掛程式實現，本身並不支援任何跨節點之間的網路通信和網路策略功能，且僅適用於 Linux 系統，所以為了解決這個問題一些公司做出一個更優秀的方案 :
- CoreOs 公司的 CNI（Container Network Interface）
- Docker 推出 CNM （Container Network Model）

最後 CoreOs 公司的 CNI 擊敗了  Docker 推出 CNM ，並成為了 Kubernetes 首選的網路外掛程式介面規範，現已成為 CNCF 主推的網絡模型。思想为：

- {{< alert info >}}
container Runtime 在創建容器時，先創建好 network namespace ，然後調用 CNI 插件，為這個 netns 配置網路，之後再啟動容器內的進程。
    {{< /alert >}}


---
## Container Network Interface（CNI）
{{< image classes="fancybox fig-100" src="/images/kubernetes/cni.jpg" >}}

CNI規範只是一個規範，規定了如何連接容器編排系統和網路外掛程式以完成 Pod 網路管理，例如 Flannel、Calico、Weave 等，都是 CNI 規範的實現方案。 Kubernetes 藉助 CNI 外掛程式體系組合，滿足網路功能，在創建 Pod 的過程中:
- Scheduler 選定了一個 Node 節點來運行新創建的 Pod
- 該節點上的 kubelet 收到消息開始創建 Pod ，當涉及到網路部分時，首先會讀取```/etc/cni/net.d```下 CNI JSON 配置檔，獲得要使用的外掛程式
- 進入外掛程式的目錄來執行二進位檔，之後 CNI 外掛程式會進入 Pod 的網路空間去設定 Pod 的網路。

---

### 參考資料
- [K8s network之一：K8s網路模型與網路策略](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-one.html)