---
title: Packer build machine image 偶爾會出錯誤

author: Aryido

date: 2022-11-28T22:04:39+08:00

thumbnailImage: "/images/others/packer.jpg"

categories:
- cloud

tags:
- aws
- gcp

comment: false

reward: false
---
<!--BODY-->
> 使用 Packer 建立 AWS EC2 AMI 或者是 GCP Machine Image，兩個都會有機率發生一些問題， 有時候是 image 內一些應用程式安裝出現問題；有時候是再最後啟動 AWS 或 GCP 虛擬機時，使用 user-data 或 startup-script 時會出現問題，共同的錯誤訊息是 **no installation candidate.** 。 AWS 機率發生體感機率比 GCP 高不少。那問題的根源是什麼呢，來看看吧!

<!--more-->

---
# 錯誤訊息原因分析
當我們在 script 內使用```sudo apt install XXX```時，如果:
- 輸入錯誤的安裝應用程式名稱
- 嘗試安裝不在預設存儲庫中的應用程式

或者是:
- **Apt 無法在存儲庫中找到要安裝的應用程式，但知道應用程式的存在(可能是預設存儲庫中還有許多其他軟體包引用到該應用程式的引用。**

以上問題發生，就會出現 *no installation candidate.* 錯誤訊息。

如果是 Apt 無法在存儲庫中找到要安裝的應用程式的話，解決方法是先把 Apt 的資料庫更新:
{{< alert success >}}

```shell
sudo apt update && sudo apt upgrade
```
{{< /alert >}}

出現此問題的原因是 Apt 不會自動檢查本地儲存庫是否與遠端儲存庫同步。故當本地落後於更新時，會因為和遠端連結不同步，而斷開遠端連結，就會產生
*no installation candidate.* 的錯誤訊息。

---

# Packer build machine image 機率性錯誤分析

承上分析知道，正常的情況下，在 Packer build image 的 script 內，有寫 apt update ，故 Apt 本地儲存庫狀態應該會是完好最新的，應該都可以正常獲取 package，但是卻發現機率性 Apt 本地儲存庫狀態會有問題，到底為甚麼呢?

最後找到原因:

{{< alert info >}}
**cloud-init 執行時會將 apt default source 改掉**。
{{< /alert >}}

例如 AWS:
- **apt default source**：

    http://archive.ubuntu.com/ubuntu bionic InRelease

- Cloud-init 改過後變成 **apt cloud source**：

    http://us-west-2.ec2.archive.ubuntu.com/ubuntu bionic InRelease

{{< alert danger >}}
看起來是有發生 race condition。Packer 有時會在 cloud-init 仍在運行時就啟動 build script。
{{< /alert >}}

這個狀況並非每次都發生的原因是在於  cloud-init 和 Packer 哪一個先跑完

##  cloud-init 先跑完：
Apt 本地儲存庫狀態，會改成 **apt cloud source** 。 Packer 再執行的 apt update 就會是 cloud source 最新狀態。在最後啟動虛擬機時用 user-data 或startup-script 時套件正常安裝。

## Packer 先跑完：
Packer 先執行 apt update ，故 **apt default source** 狀態為最新狀態。 接下來 cloud-init 才完成，**apt default source** 會改成 **apt cloud source** ，且狀態會是舊的。故可能在最後啟動虛擬機時用 user-data 或 startup-script 時，發現 **apt cloud source** 狀態並非最新，而發生 *no installation candidate.*

---
# Solution

這個雷常出現在 Packer 中，網路上有些解決方法:

- {{< alert warning >}}
強制 sleep，等待 cloud-init 完成。這方法雖然不太好，但還是有人用...
```shell
while [ ! -f /var/lib/cloud/instance/boot-finished ]; do echo 'Waiting for cloud-init...'; sleep 1; done
```
{{< /alert >}}

- {{< alert success >}}
推薦解法
```
cloud-init status –wait
```
{{< /alert >}}


---
# Reference
[How to Fix the “No Installation Candidate” Problem in Ubuntu](https://www.maketecheasier.com/fix-no-installation-candidate-problem-ubuntu/)

[Random failings of apt installing packages during packer build](https://github.com/dsaidgovsg/terraform-modules/issues/169)

[Option for builder to wait on cloud-init to complete](https://github.com/hashicorp/packer/issues/2639)

[Packer build EC2 AMI 時等待 cloud-init 先完成](https://shazi.info/packer-build-ec2-ami-%E6%99%82%E7%AD%89%E5%BE%85-cloud-init-%E5%85%88%E5%AE%8C%E6%88%90/)