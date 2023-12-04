---
title: "Kubernetes - Secret"

author: Aryido

date: 2023-10-31T21:14:57+08:00

thumbnailImage: "/images/kubernetes/secret-logo.jpg"

categories:
- kubernetes

comment: false

reward: false

---

<!--BODY-->
> Kubernetes Secret 是一種將**配置設置與應用分離的抽象**，解決服務間配置的冗余與維護問題。主要可以用來保存敏感訊息，將這些 data 放在 Secret 中，比放在 Pod 的定義中或者 Docker Image中，來說更加安全和靈活。
>
> <!--more-->

---
{{< image classes="fancybox fig-100" src="/images/kubernetes/configmap-secret.jpg" >}}

# 簡介
Kubernetes 資源對象 Secret 概念上和 ConfigMap 一樣，也是希望將**資料與應用程式**解耦，但最重要的部分是可以用來保存{{< hl-text red >}}敏感的資料{{< /hl-text >}}，而不隨便曝露。一般情況下 ConfigMap 是用來儲存一些非敏感的設定 data ，如果涉及到一些安全相關的資料的話，用 ConfigMap 就非常不妥，因為 ConfigMap 是明文儲存的。而 Secret 是將一些敏感資訊，像是 :
- 資料庫帳密
- 訪問其他台 server 的 Access Token
- SSH Key

用 **非明碼的方式(opaque)** 存放在 Kubernetes 中。
{{< alert success >}}
Secret 有以下幾種常見的使用方式：
- 當成**environment variables**使用
- 將 Secrets File mount 在 Pod 某個檔案路徑底下使用
{{< /alert >}}

---

# [Secret YAML](https://kubernetes.io/zh-cn/docs/tasks/configmap-secret/managing-secret-using-config-file/)
Secret 有分 data 和 stringData。
- data: 用來儲存 base64 編碼的任意資料
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysecret
  type: Opaque
  data:
    username: YWRtaW4= # echo -n 'admin' | base64
    password: MWYyZDFlMmU2N2Rm # echo -n '1f2d1e2e67df' | base64
  ```
  {{< alert info >}}
  檢查是否正確編碼:
  ```shell
  echo "YWRtaW4=" | base64 -d
  admin

  echo "MWYyZDFlMmU2N2Rm " | base64 -d
  1f2d1e2e67df
  ```
  {{< /alert >}}

- stringData: 為了方便，它允許 Secret 使用未編碼的字串。
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysecret
  type: Opaque
  data:
    username: admin
    password: 1f2d1e2e67df
  ```

在 Pod 內，可以用環境變數的方式，使用 Secret 中的資料。

```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
      - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
```

## [envFrom](https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-env-var)
此功能在 Kubernetes 1.6 版本之后可用。使用 envFrom 可將所有 Secret  Secret  中的 Key 會直接成為 Pod 中的環境變數名稱。
```
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    envFrom:
    - secretRef:
        name: mysecret
```


{{< alert danger >}}
除非容器重新啟動，否則容器將無法感知到 Secret 更新。但有第三方解決方案可以在 Secret 改變時觸發容器重新啟動。
{{< /alert >}}

---
### 參考資料

- [Day 17 - 藏好你的秘密：Secret](https://ithelp.ithome.com.tw/articles/10193940)

- [Kubernetes 應用配置管理](https://www.readfog.com/a/1653238144516067328)

- [Secret 的使用](https://www.qikqiak.com/post/use-secret-in-k8s/)

- [Kubernetes — Secret](https://medium.com/learn-or-die/kubernetes-secret-9d4733d10ff2)

- [[Day 12] 敏感的資料怎麼存在k8s?! - Secrets](https://ithelp.ithome.com.tw/articles/10195094)

- [Can't create Secret in Kubernetes: illegal base64 data at input](https://stackoverflow.com/questions/53394973/cant-create-secret-in-kubernetes-illegal-base64-data-at-input)