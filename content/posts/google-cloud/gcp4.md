---
title: AWS 與 GCP reliability 不同的地方比較 - 2

author: Aryido

date: 2022-10-04T22:23:28+08:00

thumbnailImage: "/images/google-cloud/logo.jpg"

categories:
- cloud

tags:
- gcp
- aws
- english

comment: false

reward: false
---
<!--BODY-->
> 前段時間社群上有人 po 出 GCP 和 AWS 的比較，然後測出 GCP 慘烈的 VM 生成時間和一堆 409 錯誤，聽說有驚動 Google 高層(~~怕.jpg~~)。那現在我們來針對該作者開源的測試程式來看看吧!

<!--more-->

---

以下分析由[cloud-gpu-reliability](https://github.com/piercefreeman/cloud-gpu-reliability)，測試 AWS 與 GCP 的可用性。

# Benchmarking Harness
## machine type
首先先來看下 [machine types](https://github.com/piercefreeman/cloud-gpu-reliability/blob/main/gpu_reliability/cli.py):
- GCP: n1-standard-1
- AWS: g4dn.xlarge

關於機器部分，PR有人有提到，一開始 Benchmarking 其實不算太精準。因為 GCP 會因為 vCPUs 數量限制**上傳頻寬(Egress Bandwidth)**，故把 GCP machine type 更改為 **n1-standard-4**，會比較接近 **g4dn.xlarge**。

---

## Zone 的差別
該[測試](https://github.com/piercefreeman/cloud-gpu-reliability/blob/main/gpu_reliability/cli.py)的 region ， GCP 是指定 us-central1-b 這一個 Zone 而已，但 AWS 是整個 us-east-1 region 都可使用。若可以讓 GCP Zone 隨機化，可能會更好。

---

# 409 錯誤分析
目前調查知道 Google cloud engine API 會返回 409 錯誤的原因是有**重複的 instance 名稱**

最後檢查，發現是測試 code 有 [bug](https://github.com/piercefreeman/cloud-gpu-reliability/pull/5/commits/8539717d586a5888210f87bc1da2f7c694f0689b)，因為有可能有某些 instances 在進入到 RUNNING 前，收到刪掉的request，故返回 409 CONFLICT 錯誤，這個返回是 sensible 的，因為不知道到底要創建還是刪除，可歸類為有衝突。

---

# operation timing
要測量創建時間，其實初始 HTTP 請求應該也要算到創建時間的一部分，[後續有更正了](https://github.com/piercefreeman/cloud-gpu-reliability/pull/8)。
但最重要的是後面一段，內部工作人員發現他們的 GCP 的 library 有一些 Bug 導致 VM 創建完成時沒法快速發出通知，已反映給 Google 內部了。

---

# 結論
{{< alert warning >}}
前面的 machine types 和 Zone 的差別，其實沒有太大。 **GCP 是真的比較慢!!!**
{{< /alert >}}

{{< alert warning >}}
409 錯誤是測試框架的 bug，和 GCP 沒關係。
{{< /alert >}}

{{< alert warning >}}
**GCP 是真的比較慢!!!**，但是原因有一部份是他們官方自己提供的 library 有 bug ，回覆完成時間比較久。但也不確定 library 修好後時間會進步多少，但應該不會像測試那樣差 AWS 那麼多。
{{< /alert >}}

---

# Vocabulary
{{< alert info >}}
**disclosure**[dɪsˋkloʒɚ] :

n.揭發；透露；公開

**disclose**[dɪsˋkloz] :

vt.使露出，使顯露

{{< /alert >}}

{{< alert info >}}
**severely**[səˋvɪrlɪ] :

adv.嚴格地；嚴厲地；

{{< /alert >}}

{{< alert info >}}
**hinder**[ˋhɪndɚ] :

vt.妨礙；阻礙（+from）

{{< /alert >}}

---