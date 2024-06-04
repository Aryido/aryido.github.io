---
title: cloud init 簡介

author: Aryido

date: 2022-11-29T20:23:44+08:00

thumbnailImage: "/images/others/cloud-init.jpg"

categories:
- cloud
- _common

tags:
- virtual-machine 

comment: false

reward: false
---
<!--BODY-->
> cloud-init 是一個 package，可以藉由配置 cloud-init 來執行各種任務，自動初始化 cloud instance。在初次開機時就將想要的檔案或設定與系統一併弄好而不用手動處理。大部分雲平台都支持 cloud-init，目前是 **industry standard (行業標準)**。

<!--more-->

---

# cloud-init 解決了什麼問題
因為大部分的 cloud 都支援 cloud-init 作為啟動標準，cloud init 標準化了設定，可以不用在機器啟動時，還要自己做一些
- package 管理
- 帳號管理
- 儲存空間初始化
- ssh key 管理

其實直接包成一個 image 也是一種方法，但rebuild image 會比較花時間，正確的配置 cloud-init 可以有和 image-base 一樣的優點。

---
# cloud-init architecture
{{< image classes="fancybox fig-100" src="/images/others/cloud-init-arch.jpg" >}}

原理上來說，Cloud-Init 就是在開機時，掃描 CD 或 磁碟機上的設定文件，並跑一些 Script。

## meta-data
關於開機訊息、主機的訊息都是 metadata 相關。例如:
- local-ipv4
- public-ipv4
- hostname

{{< alert info >}}
為甚麼 vm 啟動後會分配到 ip 呢 ? 這邊就是 metadata 幫我們透過 dhcp 分配網路設定的。
{{< /alert >}}

## user-data
user-data 由來是設想 user 想對 vm 的做的設定，為客製化的起點，通常使用 shell script 來撰寫設定。只會在 launch instance 時執行一次，
注意 stop instance 再 start instance 之後，不會執行第二次，故
很適合用來安裝 package 和做初始化

{{< alert info >}}
AWS EC2 要設定 VM 開機之後的行為，也是直接沿用這名稱。連 launch instance 時只執行一次也一樣。
{{< /alert >}}

{{< alert warning >}}
GCP VM instance 也有類似的功能，較startup-script。但 startup-script 不只 launch instance 時會執行，連 stop instance 再 restart instance 也會執行。
{{< /alert >}}

## vendor data
基本上 vendor-data 跟 user-data 差不多，但 user-data 會覆蓋 vendor-data，想成是 user-data 級別高一點吧。

---
# 工作原理
cloud-init 對系統的配置分為四個階段， 稱 stage。分別是
- local
- network
- config
- final

詳細工作階段在這裡就不詳述了，cloud-init 所有 debug 日誌全部默認輸出到 ```/var/log/cloud-init.log```，可以用來 debug。


---
# Reference
- [cloud init 介紹](https://hackmd.io/@txLtb1_dT1eziDq4utYbqA/ryUSQD_wu#%E4%B8%80%E5%88%87%E8%B3%87%E6%96%99%E7%9A%84%E8%B5%B7%E9%BB%9E--metadata-)