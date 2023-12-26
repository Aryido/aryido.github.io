---
title: Terratag

author: Aryido

date: 2022-09-21T23:11:25+08:00

thumbnailImage: "/images/terraform/logo.jpg"

comment: false

reward: false
---
<!--BODY-->

> Terratag 是個 CLI 工具，可簡化 resource tag 的方式，允許將標籤應用於整個 Terraform 或 Terragrunt，對於 Terraform 社群來說，他們希望**集中化**來標註 resource 而不是分別寫在每個resource內，以更方便的追蹤和管理...

<!--more-->

---
Terratag 可將標籤或標籤應用於任何 AWS、GCP 和 Azure 資源。
{{< alert info >}}
補充一下，在 aws & azure ，我們會說標註是 tags 。但 gcp 因為已經有一個名為 network tag 用於連接網路防火牆的配置，故 gcp 中標註稱為 labels

以下文章統稱 tags
{{< /alert >}}

---

# 功能
### Terratag 可以統一把要部署的 resources **全部**一次性都加上 tags

不是每個 resource 都能被標註，但 Terratag 可以自動分辨，把可以標注的資源，加上標注。

---

### Terratag 能針對 .tf 內的 resource type 進行標注？

可以使用 -filter=<resource-type> 來只讓需要的 resource 加上 tags
``` shell
    terratag -tags="flag=vm1,author=henrylee" -filter=google_compute_instance
```
{{< alert warning >}}
目前只能針對resource-type 來 filter tag , 無法精細到 resource-name
{{< /alert  >}}

{{< alert danger >}}
使用 -tags 時，如果是使用 a comma seperated list of key=value。 各個 kvp 間**不要留空白**，會讀不到。

-tags="flag=vm1,author=henrylee" （Ｏ）

-tags="flag=vm1, author=henrylee" （Ｘ 不會報錯但沒辦法寫入tag）
{{< /alert >}}

---

### Terratag 針對整個 .tf 檔， 使用 Terratag 會不會override？ 還是可以繼續加上去？

Terratag 指令下完後，他會把原本的 <<base-file>>.tf 檔，後面加上 .bak 變成
<<base-file>>.tf.bak。 然後新創一個檔名為 <<base-file>>.terratag.tf 的檔案。 此檔案內容會添加 tags。

預設情況下， Terratag 指令不會對有.terratag 的檔案下標註。 所以 Terratag 指令下完後再次下 Terratag 指令，不會有任何改變，因為原本tf檔名改變了變成有 terratag 關鍵字出現在檔名，所以skip了。

但配合-skipTerratagFiles；-rename 等等參數測試後，Terratag 可以重複呼叫，tag 部分也會保留原始資料，並持續擴展下去。

#### -skipTerratagFiles
沒有指定 skipTerratagFiles 的話，因為預設是true，顧會跳過。
``` shell
    terratag -tags="flag=vm1,author=henrylee" -filter=google_compute_instance
    # 執行完後原檔會加上tags
    #更名為<<base-file>>.terratag.tf

    terratag -tags="author=sting,id=ps145" -filter=google_compute_instance -skipTerratagFiles=false
    # 因為有 -skipTerratagFiles=false ，執行完後原檔會加上新tags
    # 因為有key相同，會override舊的
    # 會產生 <<base-file>>.terratag.tf.bak
    # 檔名變為<<base-file>>.terratag.terratag.tf，越來越長

    terratag -tags="author=paul" -filter=google_compute_instance -skipTerratagFiles=false
    # 因為有 -skipTerratagFiles=false ，執行完後原檔會加上新tags
    # 因為有key相同，會override舊的
    # 會產生 <<base-file>>.terratag.terratag.tf.bak
    # 檔名變為<<base-file>>.terratag.terratag.terratag.tf，越來越長

```
會用 terraform 的內置函數merge起來。 若 tag 的 key 值一樣，則會覆蓋成新的。

#### -rename
``` shell
    terratag -rename=false -tags="flag=vm1,author=henrylee" -filter=google_compute_instance

    terratag -rename=false -tags="author=sting,id=ps145" -filter=google_compute_instance

```
如果是使用-rename，執行多次terratag,只會保存最新和前一次的狀態。

---
