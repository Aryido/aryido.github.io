---
title: "Kubernetes Deployments: Rolling vs Canary vs Blue-Green"

author: Aryido

date: 2023-05-17T00:26:34+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
>

---

### Canary Deployment 實施流程範例

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:v1
        ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
      livenessProbe:
        httpGet:
          path: /
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 10
```
在範例 yaml 中，使用的 image 是 ```myapp：v1```，其中 ```replicas``` 先設 3 個來模擬狀況；比較重要的是要加個 health check 來檢查運行狀況，如果這些探測中的任何一個失敗，Pod 將重新啟動 :
- readyinessProbe 檢查容器是否已準備好接收流量
- livenessProbe 檢查容器是否仍在運行。

若要執行 Canary Deployment，要創建第二個 deployment yaml 檔，**並且使用的 image 是 ```myapp：v2``` 新版**。然後 :

- 將更新 deployment yaml 的 ```replicas``` 從 3 >> 2

- 部屬新的 deployment yaml 檔，```replicas``` 設 1 就可以了，代表一小部分的 pod 也在系統上

- 最後在配置 load balancer 來讓流量可以進到 v1 和 v2 的 pod

v2 新版本運行后，可以對其進行測試以確保其正常運行。如果一切正常，就可以更新擴展 v2 新版本的 ```replicas``` 數量，並逐步縮減 v1 舊版本的```replicas```數量。如果 v2 版本出現任何問題，我們可以通過反轉該過程快速回滾到 v1。

---


{{< alert success >}}
Deployment >> ReplicaSet >> pod
{{< /alert >}}




