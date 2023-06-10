---
title: "Terraform: Input Variables"

author: Aryido

date: 2023-06-06T23:21:58+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

comment: false

reward: false
---
<!--BODY-->
> 要讓 Terraform 能有複用性，我們需要加入一些**參數化**變數來增加彈性，也就是為 module 的 parameter，而且在 root module 中定義好的 variable，可以根據需求來覆蓋。 Terraform Input Variables 的功能，讓我們可以宣告變數，並有幾種方式可以傳入自訂義 variable values:
> - command line
> - ```.tfvars```
> - Environment 讀取
>
> 以下會簡單介紹這些方式和優缺點。
<!--more-->

---

雖然對於 Terraform 的設計，在任何 ```.tf``` 檔案內的任何位置，我們都可以宣告  Variable ，但這不太符合 best practice ...，一般會建議把所有變數相關的宣告，都放在 ```variables.tf``` 內。在 plan 或是 apply 時，若 terraform 抓不到任何 Variable 資訊的話，會要求你在 console 輸入資料。

---

## command line
想要在輸入指令時帶入資料，需要加上 ```-var``` ，範例如下:
```bash
terraform apply -var="project_id=my-test-project"
```
比較需要注意的是**複雜型態**的變數

```bash
terraform apply  -var="project_id=my-test-project" -var='zones=["us-central1-a", "us-central1-b"]'

terraform apply -var='img_info={"image_project":"my-img-project","image_name":""my-test-img"}'
```

{{< alert success >}}
```-var``` 可以在 cli 中使用多次，來傳入多個 variable value 。
{{< /alert >}}

{{< alert warning >}}
個人覺得在 cli 中傳入太多變數的寫法其實不太好，而且在面對**複雜型態**的變數，其實也蠻容易寫錯的。
{{< /alert >}}

---

## .tfvars
建立 ```.tfvars``` 檔案，這個副檔名就是變數定義檔。例如定義一個 ```testing.tfvars``` 檔案內容如下 :

```bash
project_id="my-test-project"

zones=[
    "us-central1-a",
    "us-central1-b"
]
```

在輸入指令時，加上 ```-var-file``` ，可以用來指定變數定義檔，範例如下:

```bash
terraform apply -var-file="testing.tfvars"
```
但還需要指定 ```.tfvars``` 檔案實在是有點麻煩...，為解決這問題，有一些預設會抓取的名稱。如果 ```.tfvars``` 檔案，符合以下檔名規則，Terraform 則會自動用來當作```-var-file``` :
- **terraform.tfvars**(最常用)
- *.auto.tfvars

{{< alert info >}}
自動當作 ```-var-file``` 的檔案，必須放在執行 terraform apply or terraform plan 時所在的目錄中喔。
{{< /alert >}}

{{< alert warning >}}
個人最常使用 **terraform.tfvars** ，並且建議不要把它上傳到 GIT 內管理。
{{< /alert >}}

---

## Environment
Terraform 會搜尋以 ```TF_VAR_``` 開頭宣告的環境變數來使用，，範例如下:

```bash
export TF_VAR_project_id=my-test-project

terraform plan
```
{{< alert warning >}}
個人基本上不會採取這種做法，在 CICD 流程中，我會選擇造出 **terraform.tfvars** 檔案並放到對應位置，而不會使用寫到環境變數的方式。
{{< /alert >}}


---

## 變數的使用優先順序
上面提到幾個方法可以傳入 variables ，那如果同時有多個方法被使用，優先級如何排序呢 ? Terraform 載入變數的順序如下，**較晚載入的會覆蓋前面的**:
- 環境變數
- terraform.tfvars
- *.auto.tfvars
- 指令加上 -var 或是 -var-file 參數

{{< alert info >}}
基本上以 cli 指令代的值為主，再來是 ```.tfvars```，最後才是環境變數。
{{< /alert >}}

---
