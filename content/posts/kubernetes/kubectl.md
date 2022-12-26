---
title: kubectl 簡單筆記

author: Aryido

date: 2022-12-25T21:38:20+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> kubectl 是針對 k8s cluster 的 API Server 發送命令的工具，有些下達的指令會改變 K8s cluster 的 state 和任何對應到的環境變量。默認情況下，kubectl 在 $HOME/.kube 目錄下查找名為 config 的文件，kubectl 使用該 config 文件來查找要通訊的 K8s cluster 資料。

<!--more-->

---
安裝任何 CLI 後都有個起手式 :
```
kubectl version
```
檢查安裝是否成功。

工作上會有多個 k8s cluster 要管理，例如有 GKE 和 EKS，會有切換 k8s cluster 的需求，故可使用 kubeconfig 文件，便在集群之間快速輕鬆地切換。若要看整個 kubeconfig 配置 :
```
# 取得設定檔
kubectl config view
```
# Context
若有**新建** k8s cluster 在對應雲端上，可使用以下指令，連結到對應新建雲端 k8s cluster 資訊且**自動更新 kubectl 配置**:

```
# GKE
gcloud container clusters get-credentials <CLUSTER_NAME> --zone <ZONE_NAME>

# EKS
aws eks update-kubeconfig --region <REGION-CODE> --name <CLUSTER_NAME>
```

{{< alert warning >}}
使用 CLI 來連接 GKE 中的集群時，會需要安裝 [gke-gcloud-auth-plugin](https://stackoverflow.com/questions/74233349/how-do-i-install-gke-gcloud-auth-plugin-on-a-mac-m1-with-zsh)，可能會遇到一些問題，這邊附上連結。

{{< /alert >}}

要看 Context 配置，輸入以下命令
```
kubectl config get-contexts

# 查看當前 context
kubectl config current-context

# 切換 k8s cluster
kubectl config use-context <CONTEXT_NAME>

# 刪除指定 context
kubectl config delete-context <CONTEXT_NAME>

# 刪除 contexts and users
kubectl config unset contexts
kubectl config unset users
```
對 Pod 中的容器执行命令

```
kubectl exec -it [pod-name] bash
```

---
# 其他 Tool 推薦

## k9s
在使用 k8s 時，要查看 pod 和 service 狀態，都要打一長串指令，初學時建議熟悉下，但也在這邊介紹一個工具叫 k9s。它提供一個 terminal 介面，來跟 k8s cluster互動。
{{< image classes="fancybox fig-100" src="/images/kubernetes/k9s.jpg" >}}

## [zsh auto-completion](https://kubernetes.io/zh-cn/docs/tasks/tools/included/optional-kubectl-configs-zsh/)
指令打久了也會希望能快速 auto-completion ，故 kubectl 也有提供相關腳本。

---
