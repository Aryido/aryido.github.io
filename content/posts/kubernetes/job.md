---
title: Kubernetes Job 簡介

author: Aryido

date: 2023-03-02T23:10:15+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> 我們知道 kubernetes 的 deployment 可以生成並管理 Pod ，且盡量維持其狀態為 Running 。但有的時候我們會有**只運行一次性任務的需求**，這時候就可以使用  [kubernetes Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/)。 Kubernetes Job 用於處理一次性工作，會創建一個或多個 Pod，並在該工作完成後終止這些 Pod。
<!--more-->

Job 及 CronJob 是一組的概念 :
- Job 會保證在它所定義的 Pod 運行一次性任務
- CronJob 有排程的概念，我們可以透過 CronJob 把 Job 中所定義的工作放到排程中

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
        command: [
          "sh",
          "-c",
          "echo Hello; sleep 5"]
      restartPolicy: Never
```
{{< alert success >}}
其實可以把 Kubernetes Job 看成一個特殊的 deployment， 寫法上並沒有差太多。Job 最終的追求目的是為了結束而運行的，跟 Deployment 追求的是持續運行有著很大的不同。
{{< /alert >}}

{{< alert warning >}}
因為 Job 也是會把 Pod 創建出來，故 Job 的 .metadata.name 的命名合法的，即 :
- 不能超過 253 個字符
- 只能包含小寫字母、數字，以及 - 和 .
- 必須以字母數字開頭
- 必須以字母數字結尾
{{< /alert >}}

---