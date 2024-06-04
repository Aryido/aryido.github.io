---
title: Kubernetes Job 簡介

author: Aryido

date: 2023-03-02T23:10:15+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- containerization
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 我們知道 kubernetes 的 deployment 可以生成並管理 Pod ，且盡量維持其狀態為 Running 。但有的時候我們會有**只運行一次性任務的需求**，這時候就可以使用  [kubernetes Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/)。 Kubernetes Job 主要是針對短時和批量的 workload ，用於處理一次性工作，會創建一個或多個 Pod，並在該工作完成後終止這些 Pod，而不是像 deployment、DaemonSets 那樣持續運行。
<!--more-->

Job 及 CronJob 是一組的概念 :
- Job 會保證在它所定義的 Pod 運行一次性任務
- CronJob 有**排程**的概念，我們可以透過 CronJob 把 Job 中所定義的工作放到排程中

# Job 簡介

當 Job 完成時，Kubernetes 會將其狀態更新為 :

- 若正常運行結束後不會重啟，而會將 Pod 狀態改成 Completed。

- 若失敗並且已達到指定的**重啟配置**次數，則 Job 的狀態為 Failed

{{< alert info >}}
Kubernetes Job 很適合用於需要處理*一次性*或*排程*的情況。
例如 :
- 批次處理
- **db data import**

以實現更高的可用性和可靠性。
{{< /alert >}}

---

# Job YAML 簡單範例

Job YAML 的 .spec 中只有 .spec.template 是必需的。.spec.template 的值是一個 Pod 模板。其定義規範與 Pod 完全相同。以下是一個 Kubernetes Job 的簡單範例 YAML 文件。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: my-container
        image: busybox
        command: ['sh','-c','echo Hello']
      restartPolicy: Never
```
{{< alert success >}}
其實可以把 Kubernetes Job 看成一個特殊的 deployment， 寫法上並沒有差太多。Job 最終的追求目的是為了結束而運行的，跟 Deployment 追求的是持續運行有著很大的不同。
{{< /alert >}}

---

# Job workflow

Kubernetes 中，**並沒有自然的支持 Job workflow**，會需要其他 CICD 工具輔助。Job 的默認是 **parallel 執行 Job** ，會同時啟動多個 Pod 來處理 Job 。

如果有**按順序執行的任務**的需求，有一個不是很正規技巧實現，是使用 **Init Containers** 來實現任務的**順序執行**。以下範例：

```
apiVersion: batch/v1
kind: Job
metadata:
  name: sequential-job
spec:
  template:
    spec:
      initContainers:
      - name: task-1
        image: busybox
        command: ['sh', '-c', 'echo job-1']
      - name: task-2
        image: busybox
         command: ['sh', '-c', 'echo job-2']
      containers:
      - name: job-done
        image: busybox
        command: ['sh', '-c', 'echo "job-1 and job-2 completed"']
```

Init Containers 會依次執行，如果某個步驟 Init Container 失敗，整個 Job 就會被終止並且顯示失敗。

{{< alert warning >}}
使用 **Init Containers**來實現任務順序執行，我覺得有點不太正規的...
{{< /alert >}}

---
### 參考資料

- [入門教程：5 步創建 K8S Job，搞定批處理](https://www.readfog.com/a/1631579911044042752)

- [K8S Cron Job 記憶體不足](https://medium.com/@robhamk/k8s-cron-job-%E8%A8%98%E6%86%B6%E9%AB%94%E4%B8%8D%E8%B6%B3-b2cb8e222ab)