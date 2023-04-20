---
title: "linux 指令範例: tar & gz "

author: Aryido

date: 2023-04-16T16:44:04+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
- linux

comment: false

reward: false
---
<!--BODY-->
> tar是 Unix 和類 Unix 系統上常用的壓縮工具，名字來自於 tape archive 的縮寫， tar 可以將多個文件或目錄打包成一個檔案。單純 .tar 檔案是沒有壓縮資料的，只是把好多目錄與資料夾打包起來變成一個大檔案而已；如果要有壓縮資料的功能，要使用 .tar.gz 壓縮檔案，是最常見的壓縮檔案格式。

<!--more-->

在 Linux 之中如果需要將一堆檔案壓縮或打包成單一個檔案時，使用的並不是我們在Windows下常用的 .zip、.rar、7z 這種壓縮格式。在網路上下載的大多數 Linux 資源不外乎都是使用 .tar.gz 這個格式來壓縮打包的

## 基礎指令

使用 gzip 壓縮算法進行壓縮和解壓縮操作。

```
# 將一個 folder 內，全部文件夾打包成 tar.gz 格式的壓縮檔
tar -czvpf archive.tar.gz .

```

{{< alert info >}}
```.``` ，代表該目錄下所有 files 和 folders，都會被打包
{{< /alert >}}


```
# 解壓縮 .tar.gz 格式的壓縮檔
tar -xzvpf archive.tar.gz

```

{{< alert warning >}}
解壓縮時，注意解壓縮的位置和權限，以確保解壓縮後的檔案可以正確地使用。
{{< /alert >}}

其中命令參數 :
- **-c : 打包檔案**
- **-x : 解開壓縮檔**
- **-z : 啟用gzip壓縮**
- -v : 啟用顯示詳細輸出
- -p：表示保留文件的權限和屬性信息。
- -f : 要創建的壓縮文件的名稱

{{< alert warning >}}
如果使用 tar 工具解壓縮文件時，發現在解壓縮後的目錄下出現了，.DS_Store 等等以 . 開頭的文件。這是因為使用了 Mac OS X 的 tar 命令，並且該命令會創建一些隱藏的系統文件。
{{< /alert >}}


在 Mac OS X 中，文件系統使用了一些額外的文件屬性，這些屬性通常被存儲在名為 .DS_Store 的隱藏文件中。故使用 Mac OS X 的 tar 命令壓縮文件時，會包含這些隱藏文件。要避免這個問題，可以使用 ```--exclude``` 選項來排除 :

```
tar -czvf archive.tar.gz --exclude='.DS_Store' .

# 支持正規表達式
tar -czvf archive.tar.gz --exclude='._*' .
```

---

## 範例

```
# 做個新的 folder，其名稱和 .tar.gz 文件前綴名稱一樣
name=$(basename archive.tar.gz .tar.gz)
mkdir $name

# 解壓縮檔，到指定的 folder
tar -zxvpf archive.tar.gz -C $name

# 做一個 folder name list
folders=($(find $name -type d | awk -F/ '{print $NF}'))

for folder in "${folders[@]}"
do
  echo $folder
done

```