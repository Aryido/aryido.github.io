---
title: "apt、yum、apk 介绍"

author: Aryido

date: 2023-03-22T22:58:26+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
- linux

comment: false

reward: false
---
<!--BODY-->
> Linux 有多種流通版，例如常見的 Ubuntu、Fedora、CentOS、Debian、Red Hat 等等，其中裡面預設的**包管理系統**也不太一樣。包管理系統可以**安裝 package** 、**更新 package** 、確保使用的 **package 是經過審查的**。 接下來淺淺的分析**包管理系統** apt 、 yum 、 apk 之間的差別。
>
<!--more-->

包管理系統的**打包格式**和**工具**有平台相異性 :


| 操作系统 | 格式 | 工具 | 個人常用順序 |
| :---: | :---: | :---: | :---: |
| Ubuntu | .deb | apt, apt-cache, apt-get, dpkg | 1 |
| Debian | .deb | apt, apt-cache, apt-get, dpkg | 2 |
| CentOS | .rpm | yum | 3 |
| Fedora | .rpm | **dnf** | 4 |

- Ubuntu 最多人說它有強大視窗特效和親合力很高的圖型，是 Linux 初學者相當推薦的一套入門 Linux 流通版。

- CentOS 是由 Red Hat Enterprise Linux 改造而來、但卻不用收費的 Linux 作業系統。，合想嘗試 Red Hat Enterprise Linux，卻無力負擔花錢購買該 Linux 的人使用。

- Fedora 版本中，yum 已经被 dnf 取代，dnf 是它的一个现代化的分支，它保留了大部分 yum 的接口。專注於快速發布新功能，可以最快嘗鮮但也不太穩定。

{{< alert success >}}
Debian 、 Ubuntu 屬於一家族
{{< /alert >}}

{{< alert success >}}
CentOS、Fedora、Red Hat 屬於一家族
{{< /alert >}}

---
## apt
全名是 [**Advanced Packaging Tool**]， 是 Debian 和 Ubuntu 中的包管理器。
apt 常用命令：　　
```shell
apt update  #更新 package 清單

apt upgrade  #升級 package

apt install <package_name>  #安裝指定的 package

apt list --installed  #列出所有已安裝的 package

```

{{< alert info >}}
dpkg 和 apt-get 都是 Linux 下的套件管理工具。 dpkg 命令可以進行安裝或卸載，但是 dpkg 需要用戶手動解決依賴項。而 apt-get 則提供了更高級的功能，如解決了 package 依賴。故**建議使用 apt-get 進行套件管理**。
{{< /alert >}}

---

## yum
全名是 [**Yellow dog Updater, Modified**]，是 Fedora 和 RedHat 包管理器。

yum 常用命令 :
```shell
yum update  #列出所有可更新的 package

yum upgrade  #升級 package

yum install <package_name>  #安裝指定的 package

yum list #列出所有已安裝的 package

```

---

接下來介紹 Alpine Linux，它是一個輕量級的 Linux 發行版。它採用了musl libc 和 busybox 以減小系統的體積和運行時資源消耗。而 apk是 Alpine Linux 系統上使用的套件管理器，它是一個輕量級、快速、安全的套件管理器，旨在提供一個簡單、易於使用的方式來管理和安裝軟件包。

## apk
全名是 [**Alpine Package Keeper**]，使用 .apk 格式的軟件包，apk 包管理器非常輕量級不會占用過多的系統資源。

apk 常用命令 :
```shell
apk update  #更新 package 列表

apk upgrade  #升級 package

apk add <package_name>  #安裝指定的 package

```

{{< alert warning >}}
- update：
  用於更新軟件源列表，即更新本地套件緩存，以便系統知道哪些軟件包是可用的、哪些是最新的。它只會更新套件列表，不會安裝或升級套件。

- upgrade
  用於升級已安裝的軟件包，它會檢查所有已安裝的軟件包，並將可用的更新版本下載和安裝到系統中。
{{< /alert >}}



---

## 整理

選擇 apt 還是 yum ? 查看當前系統是什麼系統，如果是 :
- Debian 或 Ubuntu 則**使用 apt** 即可
- CentOS 則**使用 yum** 即可





