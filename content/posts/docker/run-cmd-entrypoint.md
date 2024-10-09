---
title: "Dockerfile - RUN、CMD、ENTRYPOINT 介紹"

author: Aryido

date: 2023-04-08T12:50:16+08:00

thumbnailImage: "/images/docker/docker.jpg"

categories:
- containerization

tags:
- docker

comment: false

reward: false
---
<!--BODY-->
> Dockerfile 讓我們可以透過設定**指令**的方式，快速地更新和建構 Image 。由於 Dockerfile 中可以清楚的知道 Image 建構的過程和引用的 package 組成，因此在安全性上會有所提升；也因為是純文字檔，所以 size 很小、易於分享。Dockerfile 裡面有一些指令蠻容易混淆的，這次重點介紹:
> - RUN
> - CMD
> - ENTRYPOINT
>
> 這三個指令都可以用來執行具體的命令，**但其中又有些差異**，以下做一些說明和整理。
>
<!--more-->

---

## 形式簡介

在 Docker 中，可以使用 RUN，CMD 和 ENTRYPOINT 指令運行命令。這些指令都支持兩種形式：
- **exec form**
- **shell form**

### Shell Form
在 shell 中運行命令，比較容理解和使用。特點是可以使用 shell 的一些特性，例如 *pipe* 。但是額外啟動 shell可能會**增加容器啟動時間和佔用空間**。

{{< alert info >}}
以 shell 的形式執行:
- Linux 的預設是```/bin/sh -c```

- Windows上的預設環境則是```cmd /S /C```
{{< /alert >}}

### Exec Form
直接運行命令，格式為 ```["executable", "param1", "param2"]```，運行速度較快，佔用空間也較少，但是可能會因為 executable 的選擇，**可能沒有 shell 帶來的功能**。

{{< alert info >}}
如果 Linux 上不想用預設的 shell 執行指令，那麼就可以透過 Exec Form 去改變。例如 :

```RUN ["/bin/bash", "-c", "echo hello"]```

指定想要的 shell
{{< /alert >}}

總體來說，如果需要使用 shell 的特性，則應該使用 shell form。否則建議使用 exec form，以減少容器啟動時間和佔用空間。

---

## RUN
RUN 步驟是**在 image 建構時執行的，不是在容器運行時執行的**。每加一個 RUN ，就會在基底映像層，再加上一層新資料層，執行完後會建立新的 image 。一個 Dockerfile 可以有多個 RUN 命令並能依次執行。通常用於安裝軟體、建立檔案和目錄，以及建立環境設定等項目。

格式為

```Docker
# exec form
RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form
RUN <command>
```

每條 RUN 指令，都會將**在當前映像檔基底上產生新的映像檔**，所以有以下注意要點：
- 當命令較長時可以使用 ```\``` 來換行比較容易閱讀。
- 使用 exec 形式執行時，必需使用 JSON array 的格式，因此要使用**雙引號**
- **每一個 RUN 就會新增一層資料層，為了減少不必要的資料層，可以利用 && 來串連多個命令**

{{< alert warning >}}
如果 Dockerfile 不變，則 Docker 會緩存 run 指令的結果。如果要使緩存無效，則需要將 –no-cache 選項傳遞給 build 命令。
{{< /alert >}}

---

## CMD
CMD 指令是**可以被複寫**的預設命令，是 Container 啟動時，預設要執行的指令。支援三種格式 :

```Docker
# exec form
# 使用 exec 執行，官方推薦使用
CMD ["executable","param1","param2"]

# shell form
# 預設是在 /bin/sh -c 中執行，使用在給需要互動的指令
CMD command param1 param2

# 適用於有定義 ENTRYPOINT 指令的時候，提供給 ENTRYPOINT 的預設參數
CMD ["param1","param2"]

```

如果 Dockerfile 指定了多條 CMD 命令，**只有最後一條會被執行**。且如果使用者在建立 Container 時有帶運行的命令，則會**覆蓋掉 CMD 指定的命令**。

{{< alert warning >}}
```docker run [OPTIONS] IMAGE [COMMAND] [ARG...]``` 的 ```[COMMAND]``` 會覆蓋掉 Dockerfile CMD 命令 。
{{< /alert >}}

{{< alert info >}}
CMD 最特別的地方，是可以當成傳給 ENTRYPOINT 的預設參數
{{< /alert >}}

---

## ENTRYPOINT
和 CMD 一樣，是 container 被建立後所執行的指令，**但 ENTRYPOINT 基本上一定會被執行，而不會有像 CMD 覆蓋的情況發生**。有兩種格式：
```Docker
# exec form
ENTRYPOINT ["executable", "param1", "param2"]

# shell form
ENTRYPOINT command param1 param2
```

指定容器啟動後，ENTRYPOINT **基本上**不會被覆蓋。每個 Dockerfile 中只能有一個，**當 ENTRYPOINT 指定多個時，只有最後一個會生效**。

{{< alert info >}}
Docker 在啟動容器時會先使用預設的 ENTRYPOINT 指令，而預設的指令為 /bin/sh -c 。
{{< /alert >}}

{{< alert danger >}}
如果真的想要覆蓋 ENTRYPOINT 的預設值，則在啟動 Container 時，可以加上「--entrypoint」的參數。但基本上不建議使用
{{< /alert >}}

---

## RUN、CMD、ENTRYPOINT 總整理
推薦形式:
- ```RUN``` : shell form
- ```ENTRYPOINT```: exec form
- ```CMD```: exec form

---
