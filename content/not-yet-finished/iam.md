---
title: "GCP - Identity and Access Management "

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
> 

<!--more-->

---

You have one project called proj-sa where you manage all your service accounts. You want to be able to use a service account from this project to take snapshots of VMs running in another project called proj-vm. What should you do?
A. Download the private key from the service account, and add it to each VMs custom metadata.
B. Download the private key from the service account, and add the private key to each VM's SSH keys.
C. Grant the service account the IAM Role of Compute Storage Admin in the project called proj-vm. Most Voted
D. When creating the VMs, set the service account's API scope for Compute Engine to read/write.

https://gtseres.medium.com/using-service-accounts-across-projects-in-gcp-cf9473fef8f0



{{< image classes="fancybox fig-100" src="/images/google-cloud/iam/compute-instanceAdmin.jpg" >}}


----


Every employee of your company has a Google account. Your operational team needs to manage a large number of instances on Compute Engine. Each member of this team needs only administrative access to the servers. Your security team wants to ensure that the deployment of credentials is operationally efficient and must be able to determine who accessed a given instance. What should you do?


A. Generate a new SSH key pair. Give the private key to each member of your team. Configure the public key in the metadata of each instance.
B. Ask each member of the team to generate a new SSH key pair and to send you their public key. Use a configuration management tool to deploy those keys on each instance.
C. Ask each member of the team to generate a new SSH key pair and to add the public key to their Google account. Grant the ג€compute.osAdminLoginג€ role to the Google group corresponding to this team.
D. Generate a new SSH key pair. Give the private key to each member of your team. Configure the public key as a project-wide public SSH key in your Cloud Platform project and allow project-wide public SSH keys on each instance.


B. 手動部署每個成員的 SSH 公鑰
這個選項要求每個團隊成員生成 SSH 密鑰對，然後將公鑰發送給管理者。管理者需要使用配置管理工具來將這些密鑰部署到每個實例上。這種方法在操作上比較繁瑣，且需要維護大量 SSH 公鑰。如果有新成員加入或離職，管理維護的負擔將非常大。此外，這並不是 Google 推薦的最佳實踐。

不推薦：操作效率低，且管理 SSH 密鑰對繁瑣，易於出錯。

C. 讓每個團隊成員使用 Google 帳戶並授予適當角色
這是 Google 建議的最佳實踐之一。要求團隊成員將 SSH 公鑰添加到他們的 Google 帳戶中，然後將 compute.osAdminLogin 角色授予對應的 Google 群組。這樣，團隊成員可以通過其 Google 帳戶直接登錄到 Compute Engine 實例，無需手動管理 SSH 密鑰。Google Cloud 日誌還可以追踪每個團隊成員的活動，滿足了安全團隊的審計要求。這種方法運維高效且安全。

推薦：使用 Google 帳戶和 IAM 角色來進行身份管理和權限控制，符合 Google 的最佳實踐，且便於訪問審計。

---

### 參考資料

- [IAM overview](https://cloud.google.com/iam/docs/overview)

