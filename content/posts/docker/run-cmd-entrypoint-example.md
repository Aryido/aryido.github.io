---
title: "Dockerfile - RUN、CMD、ENTRYPOINT 範例及比較"

author: Aryido

date: 2023-04-09T14:44:54+08:00

thumbnailImage: "/images/docker/docker.jpg"

categories:
- docker

comment: false

reward: false
---
<!--BODY-->
> RUN、CMD 和 ENTRYPOINT 指令都可以用來執行具體的命令。RUN 指令是在 Docker **鏡像構建**時把執行結果會記錄到鏡像中；而 CMD 和 ENTYPOINT 指令是在**容器啟動**時自動執行。
>
> ENTRYPOINT 和 CMD 的區別在於使用 ENTRYPOINT 時， CMD 指令會被作為其**默認參數**，也可以在啟動容器時通過覆蓋 CMD 指令來輸入參數。
>
<!--more-->

---

CMD 指令最特別的是，**可以作為 ENTRYPOINT 的參數**，以下用一些範例來敘述整個 RUN、CMD 和 ENTRYPOINT 的觀念。

## scenario

首先建立一個很簡單的範例 Dockerfile :
```docker
FROM ubuntu
CMD ["hostnmae"]
```

當我們運行 ```docker build -t=test-image .``` ，會把上面 Dockerfile 建構成，一個名為 test-image 的 image 。接下來把 image 啟動成一個 container 實例 :

```
docker run --name=test --hostname=my-test -it test-image

# 運行後看到我們設的 hostname ，也就是 my-test
```
運行容器時自動執行了 ```hostname``` 指令，為甚麼呢 ? 之前有提過，**Docker 在啟動容器時，會先使用預設的 ENTRYPOINT 指令，而 linux 其預設的指令就是 ```/bin/sh -c```** ! 所以其實我們的 Dockerfile 實際內容可以看成是 :
```docker
FROM ubuntu
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["hostnmae"]
```
這裡 CMD **就作為 ENTRYPOINT 的參數**，相當於在 linux 環境，下一個 hostname 指令，故理所當然會印出本機名稱。

接下來把 container 刪掉，再使用 :

```
docker run --name=test --hostname=my-test -it test-image ls

# 運行後可以看到 ls 命令的輸出
```
{{< alert warning >}}
特別注意一下，上述 cli ，多加了個 ls 參數
{{< /alert >}}

因為 CMD 指令可以可以被 ```docker run [OPTIONS] IMAGE [COMMAND] [ARG...]``` 的 ```[COMMAND]``` 覆蓋，上述範例 Dockerfile 中， CMD 指令指定的 hostname 命令便被 ls 覆蓋了，所以運行後顯示 ls 命令的輸出。

---

## 比較 CMD & RUN

- ```docker
    FROM ubuntu
    CMD ["apt-get", "install" "golang" "-y"]
    ```

若 Dockerfile 中有 ```CMD ["apt-get", "install" "golang" "-y"]```， 則會在**每次啟動容器**時都會嘗試下載安裝 golang，而若以該 Dockerfile 生成的鏡像作為基礎鏡像構建新的鏡像，則會發現新鏡像中**沒有安裝 golang**。

- ```docker
    FROM ubuntu
    RUN apt-get install golang -y
    ```

若 Dockerfile 中有 ```RUN apt-get install golang -y``` 命令，則它會在構建鏡像時，就將 golang 下載安裝好， 且以此為基礎鏡像就是**有 golang 的狀態**。

---

##  使用 RUN 注意事項

- apt-get update 和 apt-get install 建議放在一個 RUN 指令中執行

因為 RUN 有緩存機制。如果 apt-get install 在單獨的 RUN 中執行，則可能會使用 apt-get update 創建的鏡像層，而這一層可能是很久以前緩存的，放在一起能夠保證每次安裝的是最新的包。