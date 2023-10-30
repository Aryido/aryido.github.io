---
title: "Kubernetes - ConfigMap"

author: Aryido

date: 2023-10-30T22:42:57+08:00

thumbnailImage: "/images/kubernetes/configmap-logo.jpg"

categories:
- kubernetes

comment: false

reward: false

---

<!--BODY-->
> ConfigMap 是一種**資源配置管理**的抽象，可讓不同的微服務間共享配置，提供了一種將**配置設置與應用分離的方法**，讓我們可以只更新 Config 設定檔 ，而無需修改應用的 Code 或其 Image ，解決服務間配置的冗余與維護問題。基礎的 ConfigMap 用法，通常用於存儲**鍵值對**，來作爲容器化應用中的**環境變量**。
>
> <!--more-->

---
{{< image classes="fancybox fig-100" src="/images/kubernetes/configmap-secret.jpg" >}}

# 簡介
ConfigMap 是一種 Kubernetes API 對象，用來將{{< hl-text red >}}非機密性的資料
{{< /hl-text >}}，保存到鍵值對中，且資料大小必須小於 ```1MB```。 使用 ConfigMap 是希望讓**應用程式**與**設定**解耦 (Decoupled)。舉一個實際會遇到問題，例如:
> Question: 當我們有多個不同服務用 Deployment 配置，且不同服務都需要跟 Database 連接。這樣然到要在每個 Deployment YAML 內，重複寫上同樣配置連接的設定嗎 ?

在每個 Deployment 文件中，若都寫上相同 Database 連接設定，就會造成冗余。想像後續如果要更改 Database 連接設定，就會需要對多個 Deployment 都做修改，這會造成維護問題，而通過使用 ConfigMap ，可以集中配置方便管理。除了 Database 連接地址之外，API 端點 URL、應用的環境變數等等，都可以設定在 ConfigMap 內。

{{< alert success >}}
ConfigMap 可以單獨存在於 Kubernetes 中。當 Pod 需要使用 ConfigMap 時，才將 ConfigMap 給 Pod 使用。 Pod 可以將其用作
- 環境變數 Env
- 命令列參數 CLI
- 在 readonly Volume內，新增一個文件，讓應用程式來讀取
- 編寫程式在 Pod 運行，使用 Kubernetes API 來讀取 ConfigMap
{{< /alert >}}

---

# [ConfigMap YAML](https://kubernetes.io/zh-cn/docs/concepts/configuration/configmap/)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # 簡單型
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # 複雜型
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```
下面是一個簡單 Pod 的範例，會使用 上述 ConfigMap，設定給 Pod：
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: [ "/bin/sh", "-c", "echo $(PLAYER_INITIAL_LIVES)" ] # 可以透過 $(VAR_NAME) 使用參數
      env:
        - name: PLAYER_INITIAL_LIVES # 注意，這裡的名稱是環境變數的 KEY 名稱
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 指定對應的 指定對應的 ConfigMap，要用 metadata 名稱
              key: player_initial_lives # ConfigMap 中對應的 KEY，故名稱要完全一樣
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  # 可以在 Pod 設定 volume，然後將 configMap 掛載到 Pod 內的容器中
  - name: config
    configMap:
      # 指定對應的 指定對應的 ConfigMap，要用 metadata 名稱
      name: game-demo
      # 來自 ConfigMap 的鍵，將會建立為文件
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
```
上面的範例定義了一個 volume ，將它作為 ```/config``` 資料夾掛載到 demo 容器內， 並建立兩個文件 ```/config/game.properties``` 和 ```/config/user-interface.properties```。

## [envFrom](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)
使用 envFrom 可將所有 ConfigMap 的資料，定義為容器環境變量，ConfigMap 中的 Key 會直接成為 Pod 中的環境變數名稱。
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
      envFrom:
      - configMapRef:
          name: game-demo
```

{{< alert danger >}}
以環境變數方式使用的 ConfigMap 資料不會被自動更新。 需要重新啟動 Pod，才會讀到最新的 ConfigMap 資料。
{{< /alert >}}

{{< alert warning >}}
可使用**更新 configMap 的 name** ，方式，來啟動 Pod 更新機制。
{{< /alert >}}

---
### 參考資料

- [ConfigMap：動態更新應用程序配置](https://www.readfog.com/a/1705835985067151360)

- [[Kubernetes / K8s] ConfigMap 用於讓不同的微服務共享配置| Configure a Pod to Use a ConfigMap](https://medium.com/k8s%E7%AD%86%E8%A8%98/kubernetes-k8s-configmap-%E7%94%A8%E6%96%BC%E8%AE%93%E4%B8%8D%E5%90%8C%E7%9A%84%E5%BE%AE%E6%9C%8D%E5%8B%99%E5%85%B1%E4%BA%AB%E9%85%8D%E7%BD%AE-configure-a-pod-to-use-a-configmap-b2570b58fd07)

- [Day 16 - 系統設定就交給它吧：ConfigMap](https://ithelp.ithome.com.tw/articles/10193935)

- [Kubernetes 應用配置管理](https://www.readfog.com/a/1653238144516067328)

- [Kubernetes学习(configmap及secret更新后自动重启POD)](https://izsk.me/2020/05/10/Kubernetes-deploy-hot-reload-when-configmap-update/)

- [Kubernetes的ConfigMap说明](https://www.cnblogs.com/breezey/p/6582082.html)
