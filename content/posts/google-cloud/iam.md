---
title: "GCP - Identity and Access Management 概述"

author: Aryido

date: 2024-09-16T19:23:44+08:00

thumbnailImage: "/images/google-cloud/iam/iam-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> GCP 雲端資源的權限管理服務是 IAM，其全稱為 Identity and Access Management，可讓管理員(Administrator)去授權(authorize)一個 Identity，使它可以操作特定的 Resources。對應到其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : **AWS Identity and Access Managemen**
> - Microsoft Azure : **Microsoft Entra ID (舊稱 Azure Active Directory)**
>
> 簡單從英文上來說 IAM 是做什麼的 : 「manage **who** can do **what** on **which resources**」，此概念會貫穿整個 IAM 的使用，用於保護 GCP 資源，提供了非常細粒度的控制，可根據具體需求定制安全性策略。
> {{< image classes="fancybox fig-100" src="/images/google-cloud/iam/iam-overview.jpg" >}}

<!--more-->

---

# IAM 的工作原理

透過 IAM 來定義 :

```text
「誰(Principal)」對「哪個資源(Resource)」具有什麼「訪問許可權(Role)」
```

其中最簡單直觀可以了解的，就是 Resource 是什麼 :

- 例如說 Compute Engine、GKE、Cloud Storage 這些我們熟知的 GCP 服務
- 甚至比較抽象的管理資源如 Organization、Folder、Project

以上這些都被定義為 Resource 。接下來用下圖來說明 Role 和 Principal :

{{< image classes="fancybox fig-100" src="/images/google-cloud/iam/iam-examples.jpg" >}}

### Role

在開始 Role 介紹之前先提一下 Permission ，它是用來決定對特定一個 Resource 來說「**哪些操作是允許執行的**」，在 GCP IAM 中的 Permission 形式格式表示為 :

```
<service>.<resource>.<verb>
```

實際舉幾個 Permission 例子如 `pubsub.subscriptions.consume`、`storage.buckets.create` 等等。現在大概知道 Permission 的樣子了，那 Role 是什麼呢？

> 「 **Role 是 Permissions 的集合** 」

從官方文件的介紹知道 Role 的形式格式表示為 :

```
roles/<service>.<roleName>
```

例如說 GCP 已經有一個定義 Role 名稱叫 [`Compute Instance Admin`](https://cloud.google.com/iam/docs/understanding-roles?hl=en&_gl=1*ipsrgi*_ga*Mjk3MDYwNi4xNzE4MjU5OTM1*_ga_WH2QY8WWF5*MTcyNjY0MTkzMi4xNjYuMS4xNzI2NjQyMTQ2LjM2LjAuMA..#compute.instanceAdmin)，從形式格式上呈現是 `roles/compute.instanceAdmin` ，並且其擁有的 Permissions 從官網上查尋到如下圖 :

{{< image classes="fancybox fig-100" src="/images/google-cloud/iam/compute-instanceAdmin.jpg" >}}

上圖的 `Compute Instance Admin` 其還有很多 Permissions ，但因為圖片大小而沒有辦法列出來，基本上要分配「**principle of least privilege (最小權限原則)**」是要花時間精力的，只能花時間去看每個 Role 的定義，然後去對應實際應用場景來分配合適的 Role。

**Role 可當成是用來管理許多 Permissions 的載體**，因為 GCP 上面 Permissions 大概近萬個，為了管理這些權限 GCP 已經有先幫忙把一些 Permissions ，組合成一些更高階的形容來描述了：

- ##### **Basic roles**

  有 Owner、Editor、Viewer、Browser，但因為這些 Role 太廣義，故不建議在 prod 上使用

- ##### **Predefined roles**

  像上述 `Compute Instance Admin` 就是一個範例，比 Basic Role 更精細，為特定服務提供 granular access ，這些 Role 會由 Google 來創建及維護，當有新的服務或功能加入時，這些 Role 的權限也會跟著擴充

- ##### **Custom role**:

  也可以選擇來完全自己客製化 Role ，自己取名稱以及加上適用的 Permissions

{{< alert success >}}
「Access」 和「Permission」也十分相像，**個人自己看完文件後，我個人的理解是**:

- Permission 比較是指「對某個資源的一個具體的操作」如讀取、寫入或刪除，是比較官方且具體的說法，文件中也有對 Permission 的定義
- Access 比較是指「對資源進行操作的能力」，並沒有特別指說是哪種特定的動作，相比 Permission 來說比較抽象描述，通常只是夾雜在句子的描述用詞
  {{< /alert >}}

### Principal

首先來看官網的簡單介紹，會知道每個 Principal 都有一個 identifier 唯一標示碼，然後定義是:

> 「 **Principal 是有能力 access 資源的 Identity** 」

故看起來: `Principal = Identity + Role`，因此 Principal 也可以詳細稱為 「Authenticated principal」，另外官網中其實也有說**在過去 Principal 的舊稱是 Member**，這個稱呼在一些 API 上也還是有沿用(例如 Terraform)。雖然上述這樣分析，但其實在很多文章中或者講解中，敘述的 Principal 是不是真的有付加上 Role 而有權限了這件事情，其實都沒有很明確，故其實基本上還是可以當成：

> `Principal = Identity = Member`

再來要知道，實際上 Resource 的 Permission 並不會直接賦予給 Principal，而是先把 Permissions 集中起來成一個 Role ，再來 granted Role 給 Principal ，當然一個 Principal 可以有多個不同的 Roles。

Principal 在官網上有[很多不同的類型](https://cloud.google.com/iam/docs/overview#concepts_related_identity)，舉簡單常見的例如 :

- ##### Google Account

  Google Account 基本可代表成 End-User 終端用戶，可以是 developer 或者 administrator，是使用 [Google Account signup page](https://accounts.google.com/lifecycle/steps/signup/name?ddm=0&dsh=S-2128785099:1726630792338814&flowEntry=SignUp&flowName=GlifWebSignIn&TL=APps6eawczPxyJ3RscxI5EZ03JQDdNwO0VqYkUcYUgPv4E_-a9GH69oqT77aHKJ9) 註冊的帳戶，其會有像是 `gmail.com`代表個人的 email address 或者是公司行號的 `XXX@acompany.com` 等等作為 identifier

- ##### [Service Account](https://cloud.google.com/docs/authentication#service-accounts)

  Service Account 代表了**非人類的使用者**，是給 Application 使用的 Identity 。例如說有一段程式運行在 GCP 環境內且會呼叫其他 GCP 雲端服務，那會需要給那段程式一個 Identity ，這就會是其用 Service Account

{{< alert success >}}
這裡也大概可以猜測為什麼舊稱 Member 會被換掉，因為 Member 太有「人」的感覺了，這和 Service Account 的概念差比較多，所以 GCP 官方換成了 Principal 這個更中性的詞
{{< /alert >}}

雖然 「Google Account」 和 「Service Account」 是兩個最常見的 Principal，但其他還有像是 :

- **Google group** 是 「 Google Accounts 和 Service Accounts 的集合」，會有一個代表 Group 的 unique email，其可在 GCP IAM 上設定，需要 organization 層級的權限

- **Google Workspace account** 通常會代表一個公司組織的 internet domain name，例如說公司有申請 domain name 叫做 `mysupercompany.com`，那當創建 Google Account 是 `username@mysupercompany.com`的話，就會自動加入 Workspace 來管理

- **allAuthenticatedUsers** & **allUsers** : 這兩個都是特別的 Principal 類型，例如在 Terraform 內設定 iam-binding 會看到

### Policy

當我們有 Role 以後，要如何將它授權給 Principal 呢？ 這就需要使用 IAM Policy ，它是定義「**哪些角色 Role(s)要 granted 給哪些 Principal(s)**」，當 Principal 嘗試訪問某個 Resource 時，IAM 會檢查該資源的 IAM Policy 以確定 Principal 是否允許該操作。

{{< alert info >}}
Policy 除了常見的 Allow policy ，其實也有 Deny policy，目前先介紹 Allow policy
{{< /alert >}}

例如在 GCP IAM 的 Console 畫面:

{{< image classes="fancybox fig-100" src="/images/google-cloud/iam/iam-ui.jpg" >}}

當我們把某些 `XXX@gmail.com`、`YYY@gmail.com` 輸入到 Principal 格子內，然後再加上各種 Roles 然後按下創建，這就是一個 IAM Policy **Binding** 的過程。

{{< alert success >}}
會發現上圖中的 UI 邏輯算是把 Principal 和 Identity 當成**一樣的**事情了，所以**個人自己看完文件後，我個人的理解是**:

- Principal(主體) : 是指一個擁有 Role 的實體，也是比較官方且具體的說法，文件中也有對其的詳細定義
- Identity(身份) : 相比 Principal 來說也比較抽象描述，通常只是夾雜在句子的描述用詞，多數情況下可把 Identity 等同於 Principal
{{< /alert >}}

另外 Google Cloud Resources 有層次結構的例如:

```
organization >> Folders >> Project >> Resource
```

我們**可以在資源層次中的任何 level 設定 Policy**，子層級會繼承其所有父層級的 Policy。

{{< alert warning >}}

##### Consistency model for the IAM API

IAM API 是 eventually consistent，所以如果更改了 IAM 權限，然後立即讀取該 IAM 拿取其設定資料，讀取操作可能會返回舊版本的 data ，故有時候可能可以等待一些時間讓更改生效，但通常幾秒鐘後就完成了
{{< /alert >}}

---

# Practice

> 如果有一個 GCP project owner 想把管理 Cloud Storage buckets 和 files 的權限 delegate 委託給同事 colleague 。若要依照 Google-recommended practices 來做的話，應該給 colleague 授予哪些 IAM Role ?
>
> - A. `Project Editor`
> - B. `Storage Admin` **(O)**
> - C. `Storage Object Admin`
> - D. `Storage Object Creator`

`Project Editor` 會有幾乎所有 GCP Resource 的編輯權限，賦予的權限太大，違反了**最小權限原則**。 `Storage Admin` 角色是針對 Cloud Storage 的最廣泛的 Role ，可以存取 buckets 和其內部的 objs 內容，這是正確答案 ; `Storage Object Admin` 就只有 obj 的權限，如果需要的是對 buckets 及其內容的全面管理，這個角色的權限不夠。

# Practice

> 有一個已定義好 IAM Role 的 dev-project，若要創建一個 prod-project 並在此 prod-project 中擁有相同的 IAM Role，怎麼做最快速？

有 `gcloud iam roles copy` 可以使用，舉例如 :

```bash
gcloud iam roles copy --source="roles/myCustomAdmin" --destination=myCustomAdmin --dest-project=prod-project
```

---

### 參考資料

- [IAM overview](https://cloud.google.com/iam/docs/overview)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)

- [IAM 是什麼？GCP Cloud IAM 介紹](https://blog.cloud-ace.tw/identity-security/what-is-cloud-iam/)

- [【Explanation】GCP 新手村 — IAM](https://medium.com/@kellenjohn175/explanation-gcp-%E6%96%B0%E6%89%8B%E6%9D%91-iam-e0bb19952413)

- [快速了解 Google Cloud Platform IAM](https://hackmd.io/@kevinhuangtw/SJ2KjoRKc)
