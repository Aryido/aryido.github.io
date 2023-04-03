---
title: "Terraform : google_firestore_document 範例"

author: Aryido

date: 2023-03-28T22:22:08+08:00

thumbnailImage: "/images/terraform/logo.jpg"

categories:
- terraform

tag:
- gcp

comment: false

reward: false
---
<!--BODY-->
> 在 Cloud Firestore 中，存儲單位是 document 。document 是一個  lightweight record ，可包含 data 欄位 (稱: fields)，也可以嵌套另一個 collection。
> 因為 terraform 的 google_firestore_document，要求 fields 的 format 要是 json string ，比想像中的難寫，在這邊簡單紀錄一下範例。

<!--more-->

Firestore 的 fields ，會需要指定值的類型，可以參考官方提供的[文件](https://cloud.google.com/firestore/docs/reference/rest/v1/Value)。

因為每個 project ，只允許一個 Firestore ，所以**沒有刪掉整個庫來代替刪除資料**的操作；即使關閉 api ，下次 enable api 後，資料仍然會在 Firestore 裡面。

{{< alert success >}}
在 terraform 中定義 data 的好處是使用 terraform destroy 後，可以清除 data。(也**只能**刪除自己定義且由 terraform 建立的 data)
{{< /alert >}}

**注意 ! 雖然資源名稱是 google_firestore_document ，但其實更像是一個 data** 資源。 這邊有測試過，若使用 google_firestore_document 建立資料後，再手動或者使用 app 加資料進同一個 document，然後使用 terraform destroy，會發現只有 google_firestore_document 建立的 field 被刪除了，而不是整個 document 刪掉，其他手動或 app 家的資料都還是會留在 Firestore 。

---

## 範例

```yaml
locals {
  defaultJson = {
    name = {
      "stringValue" = "Tom"
    }
    age = {
      "integerValue" = 23
    }
    tags = {
      "arrayValue" = {
        "values" = [
          {
            "stringValue" : "test"
          },
        ]
      }
    }
  }
}

resource "random_id" "code" {
  byte_length = 4
}

resource "google_firestore_document" "main" {
  collection  = "users"
  document_id = "user-${random_id.code.hex}"
  fields      = jsonencode(local.defaultJson)
}

```

---
