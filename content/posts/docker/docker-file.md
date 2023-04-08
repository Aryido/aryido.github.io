---
title: Docker File

author: Aryido

date: 2023-04-08T12:50:16+08:00

thumbnailImage: "/images/docker/docker.jpg"

categories:
- docker

comment: false

reward: false
---
<!--BODY-->
> Dockerfile 是一個設定檔，讓我們可以透過指令的設定，快速地更新或建構 Container 的環境。由於 Dockerfile 中可以清楚的知道映像檔的組成，因此，在安全性上會有所提升，也因為是純文字檔，所以檔案很小、很容易分享。Dockerfile 可以看成是一個腳本，他用來明確告訴 Docker build 指令產生新的 Image 時候，所需的資訊和步驟。Dockerfile 會提供 Docker 引擎建立容器映射所需的指示。這些指令會依序逐一執行。
>
<!--more-->

---

Dockerfile 在每次要產生一個新的映像檔前都會先看 Dockerfile 中有沒有哪個是被修改過的，如果發現該行的結果其實是不需要修改的話，Docker 就會很聰明的直接利用上一次產生的映像檔當成是 cache 來當作是該行的結果。以下介紹指令 :

## Dockerfile 指令

### FROM
FROM 指令會 Pull 指定的  Image ，做為新映像建立期間，所使用的基礎 Image 。格式為 :
```Docker
FROM <image>

FROM <image>:<tag>
```

如果在同一個 Dockerfile 中，會建立多個映像檔時，可以使用多個 FROM 指令（每個映像檔一次）。

---

在 Docker 中，可以使用 RUN，CMD 和 ENTRYPOINT 指令運行命令。這些指令支持兩種形式：
- exec form
- shell form。

#### Shell Form
在 shell 中運行命令，較為簡單，易於理解和使用。可以使用 shell 的一些特性，例如管道和重定向。但是需要額外啟動 shell，可能會增加容器啟動時間和佔用空間。

{{< alert info >}}
以 shell 的形式執行:
- Linux 的預設是```/bin/sh -c```

- Windows上的預設環境則是```cmd /S /C```
{{< /alert >}}

#### Exec Form
直接運行命令，格式為 ```["executable", "param1", "param2"]```，直接運行指定命令，運行速度較快，佔用空間也較少。但是不支持 shell 的特性，例如 pipe。

{{< alert info >}}
如果 Linux 上不想用預設的 shell 執行指令，那麼就可以透過 Exec Form 去改變。例如 :

```RUN ["/bin/bash", "-c", "echo hello"]```

指定想要的 shell
{{< /alert >}}

總體來說，如果你需要使用 shell 的特性，則應該使用 shell form。否則，建議使用 exec form，以減少容器啟動時間和佔用空間。

### RUN
執行某些命令，每加一個RUN，就會在基底映像層加上一層資料層，執行完後會建立新的 image。 通常用於安裝軟體、建立檔案和目錄，以及建立環境設定等項目。

格式為

```Docker
# exec form
RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form
RUN <command>
```

每條 RUN 指令將在當前映像檔基底上執行指定命令，並產生新的映像檔，所以有以下注意要點：
- 當命令較長時可以使用 \ 來換行比較容易閱讀。
- 使用exec形式執行時，必需使用JSON array的格式，因此，請使用雙引號
- 每一個RUN就會新增一層資料層，為了減少不必要的資料層，可以利用&&來串連多個命令

### CMD
指令可以被複寫的預設命令，設定映像檔啟動為Container時預設要執行的指令。支援三種格式:

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

如果 Dockerfile 指定了多條 CMD 命令，**只有最後一條會被執行**。且如果使用者在建立Container時有帶運行的命令，則會**覆蓋掉 CMD 指定的命令**。

### ENTRYPOINT
和 CMD 一樣，是 container 被建立後所執行的指令，**但 ENTRYPOINT 一定會被執行，而不會有像 CMD 覆蓋的情況發生**。有兩種格式：
```Docker
# exec form
ENTRYPOINT ["executable", "param1", "param2"]

# shell form
ENTRYPOINT command param1 param2
```

指定容器啟動後，ENTRYPOINT **基本上**不會被覆蓋。每個 Dockerfile 中只能有一個 **當 ENTRYPOINT 指定多個時，只有最後一個會生效**。

{{< alert info >}}
Docker 在啟動容器時會先使用預設的 ENTRYPOINT 指令，而預設的指令為 /bin/sh -c 。
{{< /alert >}}

{{< alert danger >}}
如果真的想要覆蓋 ENTRYPOINT 的預設值，則在啟動 Container 時，可以加上「--entrypoint」的參數，例如：
```docker run --entrypoint```
{{< /alert >}}

### RUN、CMD、ENTRYPOINT 整理
推薦形式:
- RUN : shell form
- ENTRYPOINT: exec form
- CMD: exec form

---

### ENV
設定環境變數，格式為
```Docker
ENV <key> <value>
```
指定一個環境變數，**會被後續 RUN 指令使用，並在容器運行時保持**。

### EXPOSE
設定 Docker 容器對外的埠號，供外界使用。格式為
```Docker
EXPOSE <port> [<port>/<protocol>...]
```
在啟動容器時需要透過 -P，Docker 會**自動分配**一個埠號轉。


### WORKDIR
指定 container 裡面的路徑

格式為 ```WORKDIR /path/to/workdir```。
為後續的 RUN、CMD、ENTRYPOINT 指令指定工作目錄。
可以使用多個 WORKDIR 指令，後續命令如果參數是相對路徑，則會基於之前命令指定的路徑。

---

### COPY
複製本地端的檔案到 container 裡面，
格式為
```Docker
COPY <src> <dest>
```

複製本地端的 <src>（為 Dockerfile 所在目錄的相對路徑）到容器中的 <dest>。
當使用本地目錄為根目錄時，推薦使用 COPY。

### ADD
ADD 指令就像 COPY 指令，但還有更多功能。 除了將檔案從主機複製到容器映像，ADD 指令也可以從具有 URL 規格的遠端位置複製檔案。格式為
```Docker
ADD <src> <dest>
```
該命令將複製指定的 <src> 到容器中的 <dest>。 其中 <src> 可以是 Dockerfile 所在目錄的相對路徑；也可以是一個 URL；**還可以是一個 tar 檔案（其複製後會自動解壓縮）**。

{{< alert info >}}
使用 ADD 或 COPY 時，如果 src 或 dest 包含空白字元，可將路徑括在雙引號中
{{< /alert >}}
