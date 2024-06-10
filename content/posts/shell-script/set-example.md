---
title: "linux 指令範例: set "

author: Aryido

date: 2023-03-15T22:25:50+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
- language
- shell-script

comment: false

reward: false
---
<!--BODY-->
> set 是 shell 內建的命令，適當的使用可以增加腳本的安全性和可維護性，幫助腳本執行時可盡快發現錯誤，從而減少不必要的問題。因此很多 ```script.sh``` 檔，第一行都會加
>
> - set -euo pipefail
>
> 這篇文章簡單解釋並記錄一下，可以參考使用。

<!--more-->

使用任何 CLI 時，都要先知道查看說明語法，以便後續查詢學習
```bash
sh -c "help set"

```

Bash 在執行 script 的時候，會創建一個新的 Shell，這個 Shell 就是 script 的執行環境，Bash 默認給定了這個環境的各種參數。而 set 命令可用來修改 Shell 環境的運行參數，也就是可以定制環境。

```bash
# 查看各個參數的默認狀態
set -o
```

{{< alert info >}}
用 set 命令可以設置各種 shell 選項

- -[參數] : 將某些特性打開

- +[參數] : 將關閉某些特性

**不帶任何參數的 set 命令將顯示 shell 的全部變量**

{{< /alert >}}

## set 範例
```bash
#!/bin/bash

set -euo pipefail
```
以下簡單分析 :

- **-e (-o errexit)**：如果任何一行命令發生錯誤，會立即退出shell，不會再往下執行。

- **-u (-o nounset)**：當嘗試使用未定義的變量時，會引發錯誤。
  {{< alert warning >}}
```bash
# 若前面沒有設定過變數TEST
echo $TEST
bash: TEST: unbound variable
```
  {{< /alert >}}

  {{< alert danger >}}
對於一些變量，若忘記設置值，可能會導致 非常危險的操作。例如 :

```bash
APP=
rm -rf /${APP}
# 變為 rm -rf /. 非常危險
```

  {{< /alert >}}
- **-o pipefail**：如果管道中的任何命令失敗，則整個管道失敗。。

  以下是一個例子，表示使用 -o pipefail 的效果：
  ```bash
  set -o pipefail
  true | false | true
  echo $?
  ```
  上例中，我們執行了一個 pipe 命令，其中 :
  - 有命令是 false 返回了非零的退出狀態碼，表示命令失敗了

  若沒加 ```set -o pipefail``` ，在預設情況下 pipe 會繼續進行，且**只要最後一個 comment 是完成返回了 0**，則整個 pipe 就會返回了 0 正常狀態碼，但不是正確的。如果啟用了 ```set -o pipefail``` 模式，**pipe 要全正確才會返回 0 正常狀態碼**。

  {{< alert info >}}
  -o 這個參數無法獨自存在，它一定要後面在加一個值。
  {{< /alert >}}

---

所以一個好的做法是在 每個 shell 中添加
```bash
#!/bin/bash

set -euo pipefail
```




