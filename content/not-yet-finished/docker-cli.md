---
title: Docker - 基本操作

author: Aryido

date: 2023-04-06T20:53:33+08:00

thumbnailImage: "/images/docker/docker.jpg"

categories:
- docker

comment: false

reward: false
---
<!--BODY-->
> 虛擬化技術 :
>- 系統層級虛擬化（Virtual machine ) ，例如 Virtual Box
>- 作業系統層級虛擬化（Container），立如 Docker
>
> Docker 是一個 software platform ，可讓我們快速構建、測試和部署應用程式。Docker 將軟體打包到標準化單元中，這些單元包含軟體運行所需的一切，包括 libraries 、 system tools 、code 等等。Docker 的基本哲學 **Build and Ship any Application Anywhere** 。
>
<!--more-->

---

使用一個新的 CLI 的起頭，都要先知道如何查看常用的命令。

```
docker --help

# version 基本上只是用來看自己 Docker 有沒有正確安裝。只要安裝成功便會顯示資訊安裝版本資訊。
docker version

# info 偏向顯示 Docker 內部的訊息，例如有幾個 image、幾個 container、記憶體多少等等。
docker info
```

---

Docker Repository 是集中存放 image 的場所。最大的公開倉庫註冊伺服器是 Docker Hub ，透過 push、pull 的方式上傳、存取。Registry 可以是公開或者私有，官方有提供公開 **Docker Hub** 可用來下載 image。

## registry 命令
```
# search 指令是在 Docker Hub 找中尋找映像檔用的
docker search <軟體名稱>

# pull 指令從 Docker Hub 下載映像檔。
docker pull <image名稱:TAG>
```
{{< alert info >}}
seatch 可加上 -f (--filter) : 例如可加 ```starts=10```，代表結果只要星星數 10 以上的 images
{{< /alert >}}

{{< alert info >}}
沒有加任何 Registry 的位址時，就預設從官方的 Docker Hub Registry 下載
{{< /alert >}}

{{< alert info >}}
若沒有指定 TAG ，會找最新的版本 ( latest )
{{< /alert >}}

```shell=
docker rmi [image名稱]

docker rmi -f $(docker images -aq)
```
這個指令可刪除本機中存放的映像檔，也可以配合批次指令來一次清乾淨所有的映像檔。
{{< alert warning >}}
如果有容器還在使用這個映像檔，則無法刪除。加上 -f 參數可強迫刪除。
{{< /alert >}}

---

Image 檔用 ```docker run``` 指令執行後，執行實例就稱為 Container，每個 Image 檔可以拿來啟動成數個 Container。

## Image命令

```
# 可列出本機存在的所有image檔
docker images
```

{{< alert info >}}

option:
- -a (--all) : 可把Image的中間層也列出來
- -q (--quiet) : 只顯示Id
- -digest : 可以再多顯示摘要訊息
- -no-trunc : 不截斷訊息，代表顯示完整訊息
{{< /alert >}}

---
