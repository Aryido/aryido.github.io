---
title: Kubernetes Pod 實現解析

author: Aryido

date: 2023-04-01T17:34:53+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> Pod 是 Kubernetes 中最小的運行單位，它可以包含一個或多個 container。Pod 的實現原理主要涉及以下幾個方面：
> - 容器技術 : 實現隔離和獨立運行
> - 共享網絡和存儲 : Pod 中的所有容器共享相同的網絡和存儲空間
> - Pod 調度 : 調度器會監測節點的資源利用率，將 Pod 調度到適合的節點上運行。
> - 生命週期管理 : 當 Pod 發生故障或需要擴展時，控制器會自動創建、刪除或調整 Pod 的數量。
>
> 在 Kubernetes 中，Pod 是容器組的概念，爲應用程序提供了一個更加靈活的運行環境，負責管理容器的生命周期和資源。
<!--more-->
---

## 簡介
Pod 通過 Linux Namespace、cgroups 和容器化技術實現，容器的本質是一個特殊的 **process** ，其創建了 NameSpace 隔離不同 Pod 運行環境；並用 Cgroups 控制資源開銷，還借助了一些 Linux 網路虛擬化技術解決了網路通信的問題，**Pod 所做的則是讓多個容器加入同一個 NameSpace 以實現資源共享**。

{{< alert success >}}
cgroups（Control Groups）是 Linux 內核提供的一個機制，它可以限制、監視和分配系統資源，也就是**可以用來限制，控制與隔離 Process 所使用到的系統資源**。
{{< /alert >}}

---

## 運用 Docker 來模擬 Pod 的實現原理

目標是部屬兩個容器，其中可以彼此共享資源。

### 首先啓動 nginx 容器：
```shell
docker run -d --name nginx --ipc="shareable" nginx
```

默認情況下，Docker 的 IPC Namespace 是私有的，我們可以使用 ```--ipc="shareable"``` 來指定允許共享。

### 接下來啓動 busybox 容器:
```shell
docker run -d --name busybox --net=container:nginx --ipc=container:nginx --pid=container:nginx  yauritux/busybox-curl /bin/sh -c 'while true; do sleep 1h; done;'
```
Docker 在 run container 的時候，會創建一個 docker 模式的網路架構，再來通過 ```container：NAME_or_ID```來把新創建的 container ，使它共用 ```NAME_or_ID`` 容器的
- 網路命名空間
- 同一個網路棧
- 對外使用同一個 IP 位址或者 MAC 位址進行通信

故上述為啓動 busybox 容器，並且把 busybox 容器加入到 nginx 容器內，其中 NET、IPC、PID 全部共享 NameSpace 。兩個容器都啓動後，可以使用
```shell
docker exec -it busybox ps
```
可進入 busybox 容器中並看到 process 列表。這時會發現**可以看到 nginx 容器的進程 !** 這代表現在兩個容器處於共享資源的狀態。

{{< alert danger >}}
因為 Namespace 是由 nginx 容器創建的，若 nginx 停止運行，那麼 busybox 容器也會終止。
{{< /alert >}}

---
## kubernetes pod 實現解析
承上操作，成功的讓兩個 container 共享資源，但發現容器之間有 dependency。要保證每個容器都是**等價關係**，就**不可以讓其中的業務容器充當共享的基礎容器**。Kubernetes 考慮到了這一點，故產生了一個 **Pause 容器**來當作 Infra container 。

### Pause container
Pause container ，又稱 Infra container 。其功用就是在 Pod 創建時首先啓動，並創建**基礎 Namespace**。 等 Pause 容器啓動完成後，其它容器才接着啓動，並加入以 Pause 容器為基礎的 Namespace。這樣每個 container 就都在同一個 Namespace 下，可以訪問同一 Pod 中其他容器的資源了。

pause 容器的創建用的是 **docker 的 none 網路模式**。 CRI 只負責給 pause 容器生成一個 network namespace ， CNI 會完成 pause 容器的網路配置。

{{< alert success >}}
因此 Pause container 的生命週期就相當於是整個 Pod 的生命週期。
{{< /alert >}}

創建過程可以查看 [kubelet source code](https://github.com/kubernetes/kubernetes/blob/v1.26.1/pkg/kubelet/kuberuntime/kuberuntime_manager.go)，其中第 4 步的創建 Sandbox，實際就是創建 Pause 容器。

{{< alert info >}}
[鏡像非常小，才幾百 KB 而已。](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/pause-amd64)，性能開銷幾乎可以忽略的。
{{< /alert >}}

{{< alert info >}}
pause container 啟動後，就永遠處於暫停狀態，這也是稱為 pause container 的原因。
{{< /alert >}}

---

### 參考資料

- [K8s network之四：Kubernetes集群通信的實現原理](https://marcuseddie.github.io/2021/K8s-Network-Architecture-section-four.html)


- [Kubernetes Pod 實現原理](https://www.readfog.com/a/1697874061307252736)