---
title: Terraform 實用技巧

author: Aryido

date: 2022-11-24T23:07:42+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->
> 最近很常寫 Terraform ，知道一些 terraform cli 指令可以幫助自己寫的更好，也在學習 Terraform 的過程中，把覺得值得記錄的一些注意事項 & 小技巧留在這裡。

<!--more-->

---

# 基礎知識- mac 增加和刪除環境變數
```shell
# 查看環境變數
env

# 新增環境變數，例如設定環境變數 TF_LOG ，值為 INFO
export TF_LOG=INFO

# 刪除環境變數
unset TF_LOG
```


---
# [Terraform 排版](https://developer.hashicorp.com/terraform/cli/commands/fmt)
自動格式化程式碼工具 ```terraform fmt``` 會自動處理排版風格，但只會 format 當前層 folder 內的 .tf 檔。

```shell
terraform fmt -recursive
```

{{< alert success >}}
可以遞迴排版所有 folder 的 .tf 檔，強烈建議在 git commit 前可以做一次。
{{< /alert >}}

---
# [Terraform 輸出  state file 裡的值](https://developer.hashicorp.com/terraform/cli/commands/output)
Terraform 有 output.tf file，我們可以用
```terraform output``` 檢查是不是我們要的值。

```shell
terraform output public_ip
echo $(terraform output public_ip) # ex: "18.118.18.111"

terraform output -raw public_ip
echo $(terraform output public_ip) # ex: 18.118.18.111 ， 沒有刮號

```

---

# [Terraform log](https://support.hashicorp.com/hc/en-us/articles/360001113727-Enabling-trace-level-logs-in-Terraform-CLI-Cloud-or-Enterprise)
若要 Terraform 顯示 log ，要設置環境變數為:
```shell
export TF_LOG="TRACE"
terraform apply -no-color 2>&1 | tee apply.txt
```

也可以使用環境變數預設 log 文件位置
```shell
export TF_LOG_PATH="./terraform.log"
```
---