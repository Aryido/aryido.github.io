---
title: "Kubernetes: command & arguments"

author: Aryido

date: 2023-05-07T20:22:59+08:00

thumbnailImage: "/images/kubernetes/logo.jpg"

categories:
- containerization
- kubernetes

tags:
- docker

comment: false

reward: false
---
<!--BODY-->
> 當我們在編寫 Kubernetes Pod 相關的 yaml spec 時，有時會針對 spec.containers ，設置啟動時要執行的命令及其參數，而 Kubernetes 提供 ```command ``` 和 ```args```，兩種方式可以選擇。但這時候就會出現一些疑問 :
> - 這兩個差異是甚麼 ?
> - Docker Image 中如果自帶 ENTRYPOINT 和 CMD ，若 Kubernetes 再設置 ```command``` 和 ```args``` 會發生甚麼事情呢 ?
>
> 以下就來簡單說明一下。
>
<!--more-->

---

## 回顧 Docker ENTRYPOINT & CMD
前面有幾篇文章已經有介紹了 Docker ENTRYPOINT & CMD & RUN 了，以下再次簡單回顧一下 :

- Docker CMD 指令，可以指定容器啟動時，要執行的命令，但是要注意它很容易被 ```docker run``` 命令的參數**覆蓋掉**。

- Docker ENTRYPOINT 也是指定容器啟動時要執行的命令，但如果 dockerfile 中有 ENTRYPOINT 且也有 CMD 的話，**CMD 就會被附加到 ENTRYPOINT 指令的後面**。

{{< alert success >}}
可以看出來比起 CMD ， 相對來說 ENTRYPOINT 指令優先級更高。
{{< /alert >}}

---

## command & args

注意不要把 ```Kubernetes command``` 和 ```Docker CMD``` 混淆了 :
- ```Kubernetes command``` <<>> ```Docker ENTRYPOINT```
- ```Kubernetes args``` <<>> ```Docker CMD```

下表直接總結了 Docker 和 Kubernetes 這次主題探討的對應關係 :


|Docker       | Kubernetes   |
|-------------|--------------|
| ENTRYPOINT  | command      |
|  CMD        | args         |


**Kubernetes 設定是會覆蓋 Docker 映像中的預設 ENTRYPOINT 和 CMD 的**，覆蓋的規則如下：

### 如果 Kubernetes **沒有寫 command ，也沒有寫 args**
{{< alert info >}}
兩個都沒有寫的話，會使用 Docker 映像中定義的預設值。
{{< /alert >}}


### 如果 Kubernetes **寫了 command 和 args**
{{< alert info >}}
兩個都有的話，則會忽略 Docker 映像中定義的預設 ENTRYPOINT 和預設 CMD 。使用 Kubernetes 提供的 command 和 args。。
{{< /alert >}}


上面兩個情況是比較好想像的，但接下來就有點特別了 :

### 如果 Kubernetes **寫了 command 但沒有 args**
{{< alert danger >}}
則只會使用 Kubernetes 提供的 command。**Docker 中定義的預設 ENTRYPOINT 、 CMD 、 ARGS 都會被忽略。**

k8s 如果提供了 command，則 Dockerfile 默認的配置會被忽略，只會使用 k8s yaml 設定的 command ! (這個第一次最容易踩到坑)
{{< /alert >}}


### 如果 Kubernetes 只寫了 args 但沒有 command
{{< alert warning >}}
則會使用 Docker 映像中定義的預設 EntryPoint 並運行 Kubernetes 提供的 args。
{{< /alert >}}

---

## 總結

k8s 不管是 command 還是 args ，都會把 Docker CMD 蓋掉 !

---