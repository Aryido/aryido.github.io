---
title: "cloc: code line 統計工具 "

author: Aryido

date: 2023-04-22T13:25:32+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
- linux

comment: false

reward: false
---
<!--BODY-->
> cloc 全名是 **Count Lines of Code** ， 為一個計算 code 和設定檔**行數**的 CLI 工具，是使用 Perl 語言開發的開源統計工具。cloc 支援非常多程式語言、平台、格式的統計，可以快速地計算一個 project 中所有文件的行數、空行、註釋行等等。有時候在寫一些報告會用到，可以幫助整理資料。
>

<!--more-->

在看一個新 project 的時候，有時候 BOSS 會想了解下 code 量。最古老方便的方法是單純使用 linux 預設命令，例如統計我自己這個 blog 的 java category 文章行數:
```shell
find ./content/posts/java -type f  | xargs wc -l
```

但是 wc 使用上還是蠻不方便的，例如 code 裡面的註釋、空白行等就無法統計了。這時候就可以用更高級的工具 cloc

---

## 安裝
cloc 可以在大多數 Unix 和 Linux 系統上使用，至於要使用那種包管裡工具，可以參考之前寫的 **[apt、yum、apk 介绍]**。命令安裝：
```bash
# Debian / Ubuntu
sudo apt-get install cloc

# Fedora / Red Hat / CentOS
sudo yum install cloc

# macOS
brew install cloc
```

---

## 格式及語法
```bash
cloc [options] [file(s)/dir(s)/archive(s)]

```
其中 options 可以使用：

- -h: 顯示幫助信息
- ```--exclude-dir=<dir>```: 排除指定的目錄
- ```--exclude-lang=<language>```: 排除指定的語言
- --csv: 結果輸出為 CSV 格式
- --json: 結果輸出為 JSON 格式

---

## 範例

```bash
cloc ./content/posts/java

## 以下為輸出
  9 text files.
  9 unique files.
  1 file ignored.

github.com/AlDanial/cloc v 1.82  T=0.03 s (317.8 files/s, 33505.0 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Markdown                         9            256              0            693
-------------------------------------------------------------------------------
SUM:                             9            256              0            693
-------------------------------------------------------------------------------
```

{{< alert info >}}
cloc 甚至可以在 .tar.gz 上使用。
{{< /alert >}}

---





