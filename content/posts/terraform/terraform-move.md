---
title: Terraform Move

author: Aryido

date: 2022-12-04T19:31:26+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->
> 隨著系統的擴充， Terraform 配置也會變得越來越複雜，這時可能會需要做一些 Refactor，例如 :
> - 將某些 terraform resource 移動到其他 module
> - change resource ID
>
> 這時候用 terraform plan 檢查一下，會發現 terraform 打算把原本的 resource 移除，然後重新建立一個新的 resource。但 resource 中間被刪掉，之後再造回來是會影響服務的。我們必須讓 Terraform 知道我只是重新命名，這就是 Terraform move 想做的事情。

<!--more-->

---
# 具體問題敘述
若 Terraform 資源已經在運行了，且我們對 resource 進行了一些修正，則可能會發生甚麼事情呢 ? 具體例如:
```
# 將名稱 instance 改為 cluster_instance

resource "aws_instance" "instance" {
.....
}
```
terraform 會怎樣判斷呢？

terraform 和 state 比對後發現 .tf 中，名為 instance 的 aws_instance 已經不存在；然後也發現 terraform state 中並不存在一個新的名為 cluster_instance 的 aws_instance ，但 .tf 中有。所以 terraform 認為需要 :
- **destroy 這個 aws_instance 資源**
- **create 這個 cluster_instance 資源**

{{< alert warning >}}
以上符合 terraform 運作的邏輯，最終的狀態也會是對的，但這會過程發生些問題。因為 aws_instance 正在使用，就算會再造出 cluster_instance 這個資源，destroy 也會影響服務。
{{< /alert >}}

---

# 語法

Terraform Move 就是來處理上述的狀況。因為 Terraform state 的主要功能是綁定 local code 的資源和 remote cloud 的資源，是保持一致性的，有兩種方式可以強制改變 state 的狀態 :
- 使用 terraform state mv
- 使用 moved block

承前範例，我們 :

## 使用 terraform state mv
```
terraform state mv aws_instance.instance aws_instance.cluster_instance
```
這樣子做完後，terraform state 在 aws_instance 的名稱就會改為 cluster_instance 。所以執行 terraform plan 時，因為 .tf 檔的資源名稱也是 cluster_instance ，而不會有特別變動。

{{< alert info >}}
在 Terraform 1.1 之前，更改資源名稱而不重新創建資源的唯一方法是使用 terraform state mv
{{< /alert >}}

## 使用 moved block
```
moved {
  from = aws_instance.instance
  to   = aws_instance.cluster_instance
}
```
同樣地，這樣子做完後，因為 terraform state 和 .tf 檔的資源名稱都是 cluster_instance，故 terraform apply 時並不會刪除又新增資源。

{{< alert info >}}
Terraform 1.1 才引入了*moved blocks*
{{< /alert >}}


---
# 結論

目前比較推薦使用 *moved blocks* 來 refactor 現有 Terraform code ， 但還是有某些情況只能用 *terraform state mv* ，例如：
- Removing Resources(s) from State
在極少數情況下，Terraform 的狀態文件中會存在一些cloud上已經不存在的資源。在這種情況下，不能用一個空字符或 null 來從 moved blocks 刪除資源
```
moved { // not work, it will error
  from = aws_security_group.instance
  to   = ""
}

moved { // not work, it will error
  from = aws_security_group.instance
  to   = null
}

```
- Moving Resource(s) Between State Files
moved blocks 不可能將資源從一個狀態文件移動到另一個狀態文件。

{{< alert success >}}
moved block 不會影響 state，故使用完可以安全地移除。
但根據項目的類型和大小，公共 module HashiCorp 建議不要刪除 moved block 。因為並非每個使用該模塊的人都可能已經獲取了最新版本。除了避免給用戶帶來麻煩之外，記錄項目結構的任何重大更改對以後審查都會有所幫助。
{{< /alert >}}

---
# Reference
- [Use Configuration to Move Resources](
https://developer.hashicorp.com/terraform/tutorials/configuration-language/move-config)

- [Command: state mv](https://developer.hashicorp.com/terraform/cli/commands/state/mv)

- https://content.fme.de/en/blog/terraform-moved-blocks

- https://ithelp.ithome.com.tw/articles/10268961