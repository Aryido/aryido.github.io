---
title: "linux 指令範例: sed "

author: Aryido

date: 2023-03-06T22:56:28+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
- language
- shell-script

comment: false

reward: false
---
<!--BODY-->
> *sed* 全名為 *Stream EDitor* ，取了前面的 *S* 和後面的 *ED* 來命名。*sed* 對正規表示法有良好的支援，主要功能為自動化的修改文字檔，是在 Linux 和 Unix 系統中使用的文本處理工具，可在 pipe 中間進行文字的取代、刪除、插入等等。

<!--more-->
*sed* 指令要詳細說明的話會很雜，故這篇主要會以範例為主。並盡量會在下面補充分析指令。

- 基礎 sed 範例：
    ```bash
    echo 'This is a book' | sed 's/is/IS/g'
    # 將字串〝is〞改為〝IS〞
    # 會輸出 ThIS IS a book

    echo 'This is a book' | sed 's/ is/ IS/g'
    # is 和 IS ，兩個前加一空隔，這樣就不會把 This 改成 ThIS
    # 會輸出 This IS a book

    sed 's/\<is\>/IS/g' fileA fileB fileC
    # 檔案〝fileA〞用正規表示法的單字配匹把〝is〞改為〝IS〞
    # 可一次輸入好幾個檔案，也可以只有一個 fileA

    ```
sed 最基本的使用為搜尋內容並取代。以下為基本 template :
```s/pattern/replacement/g``` ，作用是將文本中的 pattern 替換為 replacement。
- *s*：替換指令
- *g*：全局指令。使用 g 命令可以在每個匹配上進行替換，而不僅僅是在每行的第一個匹配上進行替換。

# [safe_sed 範例](https://github.com/xwiki/xwiki-docker/blob/master/14/mysql-tomcat/xwiki/docker-entrypoint.sh)

這個是在 xwiki 的 docker-entrypoint.sh 看到的範例，有完整使用介紹，在這裡筆記一下。

```bash
# Allows to use sed but with user input which can contain special sed characters such as \, / or &.
# $1 - the text to search for
# $2 - the replacement text
# $3 - the file in which to do the search/replace
function safesed {
  sed -i "s/$(echo $1 | sed -e 's/\([[\/.*]\|\]\)/\\&/g')/$(echo $2 | sed -e 's/[\/&]/\\&/g')/g" $3
}

```
以下簡單分析 :

- *-i* : 表示直接修改讀取的檔案內容，而不是由螢幕輸出。

- 承前面模板，可把 **pattern** 部分看成 :
  {{< alert info >}}
  ```$(echo $1 | sed -e 's/\([[\/.*]\|\]\)/\\&/g')```
  {{< /alert >}}
  其意義是會把 echo 的 ```$1```變數，經過 pipe，丟給 ```sed -e 's\/([[\/.*]\|\]\)/\\&/g'``` 命令

{{< alert warning >}}
目前測試的結論，```sed -e 's/\([[\/.*]\|\]\)/\\&/g'```是把 echo 的變數
- 左方括號 **[**
- 右方括號 **]**
- 正斜線 **/**
- 句點 **.**

前面都加個 反斜線 \\

範例:
```
 echo "][/." | sed -e 's/\([[\/.*]\|\]\)/\\&/g'

# 輸出: \]\[\/\.

```
{{< /alert >}}

- 承前面模板，可把 **replacement** 部分看成 :
  {{< alert info >}}
  ```$(echo $2 | sed -e 's/[\/&]/\\&/g')/g```
  {{< /alert >}}
  其意義是會將 echo 的 ```$2```變數，經過 pipe，丟給 ```sed -e 's/[\/&]/\\&/g')/g```命令

{{< alert warning >}}
目前測試的結論，```sed -e 's/[\/&]/\\&/g')/g```是把 echo 的變數
- 反斜線 \\ 拿掉，**而且只針對跳脫字元的拿掉**

範例:
```
echo \]. | sed -e 's/[\/&]/\\&/g'
# 輸出: ].

 echo \]]\.\\ | sed -e 's/[\/&]/\\&/g'
# 輸出: ]].\\
# 可發現連續反斜線不會被拿掉

{{< /alert >}}
