---
title: GCP Address 介紹

author: Aryido

date: 2022-11-26T23:20:50+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- gcp


comment: false

reward: false
---
<!--BODY-->
> GCP 每個 vm instance 創建時，都會分配一個 internal IP，並且也可以選擇是否附加 external IP。 不管是 internal IP 或者是 external IP 其實都可以 static 化，把它固定下來使之不會變動。

<!--more-->

---

# Internal IP addresses
一般來說建立 vm instance ，Internal IP 預設是會從同樣的 subnet 隨機賦予一個給 vm instance，Internal IP 在關機或重啟都還會留在該台 VM instance 上，把機器刪除才會釋出內部 IP 位址。如果 vm instance 被刪除並重新創建，可能會分配一個新的 Internal IP。如果希望 Internal IP 固定的話，可以:

- 建立一個並 Reserve 一個 static internal IP，然後分配給建立的vm instance

- 把一個正在使用的**臨時 internal IP**( ephemeral internal IP)，升級成 static internal IP

自己寫指定 IP 時，範圍當然要在 subnet 範圍內，只要 static internal IP 之後，該 address 便會從 dynamic allocation pool 中剔除，不會再被隨機分配使用直到重新釋放。

{{< alert warning >}}
刪除 vm 不會自動釋放 static internal IP 位址。必須手動刪除
{{< /alert >}}

---
# external IP address
預設賦予 VM instance 的外部 IP 為暫時的，當 VM instance 重啟之後，GCP 會重新賦予該台機器一個新的 IP 位址。同樣如果希望 external IP 固定的話，也可以把它 static 化。

|     功能     |     花費        |
| :----------: |:----------:|
| 外部IP – Standard VM     | $0.004/小時 |
|Unused External Static IP |     $0.01/小時  |
{{< alert danger >}}

注意，若有保留 static external IP ，卻沒有使用的話，價格還會更高喔!
{{< /alert >}}


---
# Vocabulary
{{< alert info >}}
**ephemeral**[ɪˋfɛmərəl] :  ephemeral IP 是臨時 ip 的意思

adj.短暫的

{{< /alert >}}