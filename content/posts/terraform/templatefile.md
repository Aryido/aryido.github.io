---
title: "Terraform: templatefile
範例"

author: Aryido

date: 2023-04-23T17:45:12+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->
> templatefile 是 Terraform 的一個**內置函數**，可讓 Terraform 在運行時，根據變量動態生成文件內容 。 templatefile 函數的基本用法是：
> - 定義一個模板文件，其中包含要插入變量的佔位符。
> - 在 Terraform code 中，您可以使用 templatefile 函數來讀取模板文件，並提供要插入的變量。
>
> 最後 Terraform 會動態生成文件內容，並將其輸出。
<!--more-->

注意 ! 有許多比較舊版本的教學，會要我們使用 [hashicorp template provider](https://registry.terraform.io/providers/hashicorp/template/latest/docs/data-sources/file) 提供的 template resource，但**現在完全不推薦使用**，因為在 Terraform 0.12 版本以上，templatefile 已經是**內置函數**了。


## 基本語法
[templatefile Function](https://developer.hashicorp.com/terraform/language/functions/templatefile) 官網上有詳細的使用說明。

```json
templatefile( filename, vars )
```
兩個參數都是必須的，其中：

- filename：表示要使用的模板文件的路徑。
- vars：表示要插入模板文件的變量，是key-value pair。

{{< alert warning >}}
*.tftpl 是**建議**用於範本檔的命名模式，並不是強制的。
{{< /alert >}}

---

## 範例
templatefile 還算蠻常使用到的，例如要啟動 EC2 實例，並且需要要在 EC2 上安裝或執行一些預設腳本，我們會用到 **userdata**；或是相對應啟動 GCP VM安裝或執行一些預設腳本，要使用 **startup_script**，都會有一個 bash script檔，例如參考 google 啟動 XWIKI 範例，在啟動 GCP VM 時，要使用 **startup_script** 來安裝一些軟體和設定一些變數 :

- [startup_script.tftpl](https://github.com/Aryido/terraform-example-deploy-java-multizone/blob/main/infra/templates/startup_script.tftpl) 中有些變數要設定，例如 DB_IP 等等...

- [terraform main vm module](https://github.com/Aryido/terraform-example-deploy-java-multizone/blob/main/infra/main.tf) 內，就有 **startup_script** 對應要傳的變數值。

{{< alert success >}}
Terraform 也有更簡單的一個內置函數**file**，可單純直接讀取檔案。
{{< /alert >}}

---

## GCP Terraform Best practices

[Best practices](https://cloud.google.com/docs/terraform/best-practices-for-terraform#module-structure) 中有特別說明這些 file 或 .tftpl 檔案要怎麼放和一些規範:

- 使用 Terraform templatefile 的話，其 file extension 要求使用 ```.tftpl```

- Terraform templatefile 的檔案都放在 ```templates/``` 這個資料夾

- Terraform 引用的一般靜態文件，就放到 ```files/``` 這個資料夾