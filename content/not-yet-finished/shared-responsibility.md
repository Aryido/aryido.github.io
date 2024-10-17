---
title: "GCP -  概述"

author: Aryido

date: 2024-10-14T00:23:58+08:00

thumbnailImage: "/images/google-cloud//-logo.jpg"

categories:
  - cloud
  - gcp

comment: false

reward: false
---

<!--BODY-->

> 對應到其他的雲端服務是 :
>
> - Amazon Web Services (AWS) : \*\*\*\*
> - Microsoft Azure : \*\*\*\*

<!--more-->

---

gcp manages the lower layers of the security stack, such as physical security, and give customers tools for manageing the gigher layers

GCP 管理安全堆棧的較低層，例如物理安全，並為客戶提供工具來管理較高層。

在 Google Cloud Platform (GCP) 中，安全性是通過**責任共擔模型（Shared Responsibility Model）**來管理的，這意味著 Google 和客戶都對安全性負有責任，但每一方負責不同的層級。

GCP 管理較低層的安全性：Google 負責管理其基礎設施的較低層級，包括數據中心的物理安全、網絡安全、硬件、操作系統和虛擬化層等。Google 確保其數據中心有物理保護措施，如訪問控制、監控和防止自然災害的措施等。Google 也負責維護基礎設施的可用性、可靠性和操作系統的安全補丁。

客戶負責較高層的安全性：客戶則負責管理應用層和數據層的安全性，例如配置防火牆規則、管理虛擬機的操作系統安全補丁、加密數據、控制訪問權限以及使用 GCP 提供的安全工具來管理和保護其應用程序和數據。GCP 提供了多種工具，如 Identity and Access Management (IAM)、Cloud Armor、Cloud Identity 和 VPC 防火牆，幫助客戶更有效地管理和保護其資源。

結論：
GCP 確實管理了安全堆棧的較低層（如物理安全），並且提供了工具讓客戶可以管理其負責的較高層安全。因此，這個敘述是正確的。

{{< image classes="fancybox fig-100" src="/images/google-cloud//.jpg" >}}

{{< alert warning >}}

{{< /alert >}}

共同責任模型
這一塊我很喜歡講 其實這一段是要提醒大家 就是我們今天用雲端之後
並不能把責任賴給雲端 因為我們使用者自己也有責任
我們可以看到 我們今天如果通通都用地端的話 全部都我們自己負責 沒話說
但是我們用了雲端之後 雖然可能有一部分是由雲端負責
就是 Google 來負責 但是我們也必須要 負部分責任
比如說權限還是我們自己管的 內容還是我們自己擁有的 那防火牆可能也是我們自己管的
我們的防火牆開放太大 或者是我們的密碼沒有設得很好 這些都還是我們自己要負的責任
今天雲端發生了什麼事 或者是被駭客入侵 或者是機器掛掉等等
Google 它會依照 SLA 條款上面的規定去做賠償
那它賠償的是什麼 它賠償是主機運行時間所產生的費用
比如說三個小時不能跑 它就賠三個小時的機器的錢給你 但是你的資料不見
你沒辦法向它求償什麼資料損失的金額 這沒辦法 所以這一點要注意
萬一真的資料不見 就是雲端廠商是沒有辦法做任何事情的
也沒有辦法跟它求償這部分的金額 我們自己還是要把資訊安全做好

---

### 參考資料

- []https://cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate?hl=zh-cn)

- [[GCP 教學] 043 2 小時學完 GCP 重點服務 VM, LB, Kubernetes, DevOps, 混合雲, 資料庫, 大數據, 機器學習, AI, 網路防禦, 權限, 資訊安全等](https://www.youtube.com/watch?v=hQE14DX4LHQ&t=134s)
