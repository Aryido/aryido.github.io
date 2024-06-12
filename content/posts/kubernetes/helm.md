---
title: Helm 簡介

author: Aryido

date: 2023-03-21T22:02:20+08:00

thumbnailImage: "/images/kubernetes/helm.jpg"

categories:
- containerization
- kubernetes

comment: false

reward: false
---
<!--BODY-->
> Helm 是 kubernetes 的包管理工具。 Helm 有一個公共 Repository ，裏面主要都是配置文件，會把 Kubernetes 服務中各種元件 yaml ，統一打包成一個叫做 Chart 的模組，然後透過 value.yaml，可用來**統一**管理與設定 Kubernetes ，幫助 developer 和系統管理員，更輕鬆地部署、管理和升級 Kubernetes 中的應用程式。
>
<!--more-->

Helm Repository 也稱 Chart 庫，這些 Chart 包含了許多常見的 Kubernetes 應用程式，例如 MySQL、Redis、Nginx 等，這些 Chart 可以通過**自定義 value.yaml** 來進行個性化配置，以滿足不同的需求。

## Helm architecture

{{< image classes="fancybox fig-100" src="/images/kubernetes/helm-architecture.jpg" >}}

---

## Helm folder 介紹

來使用 helm create 指令創建一個基本的 Helm Chart，並從中瞭解其中的架構吧 !

``` bash
helm create nginx
tree nginx
```
把這個 Chart 的檔案結構化簡後就如下所見。
```
nginx/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

- Chart.yaml

  該 Chart.yaml 描述文件，定義了 Chart 的 Metadata，包括 Chart 名稱、版本號、description 等等...

- charts 目錄

  在這個資料夾裡可以放其他依賴的 Chart，稱作 SubCharts，可以在父 Chart 中進行配置和管理 。

- **templates 目錄**
  {{< alert success >}}
為存放 k8s 模板文件目錄地方，例如 Deployment、Service、ConfigMap、Secret 等模板文件，這些文件定義了 Chart 服務準備的 Kubernetes 元件。這些模板文件將根據 **values.yaml** 中的值進行渲染，生成實際的 Kubernetes 資源清單。
  {{< /alert >}}
- **values.yaml**
  {{< alert success >}}
Chart 的**預設配置文件**，內容均為給模板文件使用的變量，會定義這個 Chart 的可更動的所有參數。這些參數都會被代入到 templates 中的元件，可以通過修改此文件中的值，來進行個性化配置。
  {{< /alert >}}

從上面的介紹可以知道，我們可透過編輯 values.yaml，就可以對所有的 yaml 設定檔做到版本控制與管理。


{{< alert info >}}
Helm Chart 模板是按照 Go 模板語言寫成的。所有模板文件存儲在 chart 的 templates 目錄中。當 Helm 渲染 chart 時，它會通過模板引擎遍歷目錄中的每個文件，所以每個都會被渲染到。
{{< /alert >}}

{{< alert info >}}
Chart 開發者可以在 values.yaml 中提供默認值 ; 也可以在命令行使用 ```helm install``` 命令時通過 -f 指定 values.yaml 文件。
{{< /alert >}}

---

# Helm value 簡單範例

templates 目錄中的模板內，變量表示都是在 **{{ }}** 中，比如：

- values.yaml
  ```yaml
  Name: demo-deploy-nginx
  replicas: 1
  image: nginx:latest
  ```

- templates/deployment.yaml
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: deployment
    name: {{ .Values.Name }}
  spec:
    replicas: {{ .Values.replicas }}
    selector:
      matchLabels:
        app: deployment
    template:
      metadata:
        labels:
          app: deployment
      spec:
        containers:
        - image: {{ .Values.image }}
          name: nginx
  ```

承上範例，經過 Helm 渲染過後，會變成

- {{ .Values.Name }} 會變成 :  **demo-deploy-nginx**

- {{ .Values.replicas }} 會變成 :  **1**

- {{ .Values.images }} 會變成 :  **nginx:latest**


Helm 可以在 values.yaml 和 template 內，操作與設定非常多的東西，上面的設定只是一個非常簡單的範例，實務上有非常多的事情要處理。

{{< alert warning >}}
Values 可以來源於
- values.yaml文件
-  -f 指定的 yaml 文件
-  --set設置的變量
{{< /alert >}}

---

## 補充

```helm lint PATH [flags]```

是一個用於檢查 Helm chart 的命令，它可以幫助開發者在部署 Helm chart 之前找出可能的錯誤和問題。因爲渲染後的模板會發送給了 Kubernetes API server，可能會以格式的原因拒絕 YAML 文件。

---
