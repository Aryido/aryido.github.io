---
title: "CVE 簡介"

author: Aryido

date: 2026-11-24T11:03:34+08:00

thumbnailImage: "/images/others/cve/logo.jpg"

categories:
  - develop

tags:

comment: false

reward: false
---

<!--BODY-->

> CVE 全稱為 Common Vulnerabilities and Exposures，是**美國非營利組織 MITRE(/ˈmaɪtə/) Corporation** 於 1999 年提出的，其負責制定審核規則，並協調全球的 CVE ID發放。近年因為 AI 大模型的能力大躍進，「發現漏洞」和「利用漏洞的攻擊」的數量和速度都大幅增加了，當資安掃描報告出爐或者在網路上也常常都會看到這個關鍵字，也經常說 **CVSS 幾分**，或者 **KEV 已確認遭實際攻擊利用**等等...，專有名詞還不少，本篇就來整理一下這些名詞。

<!--more-->

---

# [CVE (Common Vulnerabilities and Exposures)](https://nvd.nist.gov/developers/vulnerabilities)

建立全球統一命名方式和識別標準後，當某個漏洞被 Exposures 公開曝光後，各種安全公告、修補程式與防護建議都會以該 CVE ID 來溝通定位，讓漏洞資訊能在全球快速共享與追蹤。
舉例來說 2026/10 月發佈的著名的 PHP 大漏洞 `CVE-2026-109619`， 就是一筆被官方記錄的 Vulnerability ID ，其中：

- **CVE** 為通用前綴
- **2026 代表分配 CVE ID 的年份，即漏洞首次官方公開的年份**(不能看到編號中的年份，就直接當成漏洞的從這年份才有，可能是已經存在一段時間，之後才被發現或公開)
- **109619 代表序號**

為了能更持久的發展下去，**美國網路安全暨基礎設施安全局(Cybersecurity and Infrastructure Security Agency，簡稱 CISA)** 主要出資贊助 MITRE ，讓他們維護 CVE 計畫。
{{<alert info>}}
CVE 計畫長期由**美國國土安全部(United States Department of Homeland Security，縮寫 DHS)** 旗下的 **CISA** 提供資金贊助與政府層級的支持
{{</alert>}}

雖然制度是 MITRE 定的，只靠一個組織去維護 CVE 當然是不夠的，現在每天新增的大量 CVE ID，並不是全部都由 MITRE 親手編寫，而是 MITRE 授權給全球多家認證機構組織來核發，使其有權限**可以在自己負責的 scope 內建立 CVE 資料**(例如 Microsoft 只替 Microsoft 產品的漏洞發號)，而參與的角色就稱 **CNA（CVE Numbering Authority）**，[全球蠻多公司和國家都有的，例如說： **台灣電腦網路危機處理暨協調中心(TWCERT/CC)**](https://www.cve.org/PartnerInformation/ListofPartners) 等等，都可以上網查詢到。

### NVD(National Vulnerability Database)

CVE 本身只負責編號和簡短描述，但還需要更詳細的描述補充說明才能更了解漏洞影響範圍，而 NVD 是**美國國家標準暨技術研究院(National Institute of Standards and Technology，簡稱 NIST)** 旗下運作的「漏洞資料儲存庫」，它會把某些 CVE 再做 enrichment 補充說明，主要會加上: CVSS、CWE、CPE，這些就是評估資安風險時最核心的三大指標。

- ##### **CVSS(Common Vulnerability Scoring System )** : 評估漏洞嚴重度分數

  漏洞嚴重程度量化為 0～10 分，分數越高，代表破壞力與風險越大。
  | CVSS 分數 | 嚴重程度 |
  |-----------|----------|
  | 0.1 – 3.9 | 低（Low） |
  | 4.0 – 6.9 | 中（Medium） |
  | 7.0 – 8.9 | 高（High） |
  | 9.0 – 10.0 | 嚴重（Critical） |

- ##### **CWE(Common Weakness Enumeration)** : 弱點類型

  描述涵蓋軟體、硬體、程式語言本身，而每一種弱點都有相應編號，格式為 `CWE-數字`。值得一提的是每年都會發布 [**CWE Top 25 Most Dangerous Software Weaknesses**](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html)，常年排在前面的都是常聽到的資安問題：
  - XSS(CWE-79)
  - Out-of-bounds Write(CWE-787)
  - SQL Injection(CWE-89)

- ##### **CPE(Common Platform Enumeration)** : 受影響的產品和版本

  工作上提到的資安掃描，主藥就是會對上 CPE 來判斷自己的系統有沒有受影響。CPE 名稱是由 **13 個以「冒號分隔」的值組成的字串**，用於描述產品。實際舉例如下：

  > `cpe:2.3:a:apache:log4j:2.14.1:*:*:*:*:*:*:*`
  > `cpe:2.3:a:vmware:vmware_workstation:6.0.2:*:*:*:*:*:*:*`
  > `cpe:2.3:o:microsoft:windows_server_2008:r2:sp1:x64:*:*:*:*:*`
  > `cpe:2.3:h:intel:core_i7:4760hq:*:*:*:*:*:*:*`

  在 `CPEv2.3` 中，前兩個值始終為「cpe」和「2.3」
  ，而後面資訊依序是：
  識別類型的代號(應用程式 (a)、作業系統 (o) 及硬體上 (h))、廠商、產品、版本、...。用 CPE 可以知道某個 CVE 影響了哪些軟硬體服務。

{{< image classes="fancybox fig-100" src="/images/others/cve/cve-nvd.jpg" >}}

CVE 資訊可以用[美國國家標準暨技術研究院（NIST）NVD CVE API 2.0](https://nvd.nist.gov/developers/vulnerabilities) 查到，是由美國政府維護的官方漏洞資料庫，想了解一個漏洞有多嚴重，NVD 是最簡便的資料參考來源。

---

# [KEV（Known Exploited Vulnerabilities）](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

KEV 是由美國網路安全暨基礎設施安全局（CISA）維護的「**已經確認在真實環境被利用的漏洞**」的清單，故只要列在 KEV 中，基本代表駭客的攻擊武器已經準備好了，相關漏洞會是最高優先級需要立即修補。現在評估漏洞風險的最新標準，基本都一定會參考 CISA KEV 來決定修補優先順序。

在數十萬筆公開 CVE 中，僅有極少數會進入 KEV，目前收錄在 KEV 裡面的 CVE 大概幾千個而已，以下條件都滿足才會被記錄進來：

- 已經有 CVE ID
- 有被實際攻擊利用的可靠證據(攻擊嘗試也算，例如對蜜罐（honeypot）發動的攻擊)
- **要有明確的修補方式**如：套用廠商更新、只是過渡的官方 patch

目前官方的 KEV 更新推播管道是 Email。

{{<alert info>}}
由於 CVE 上實在有太多的漏洞了，目前是把重點放在「已經構成實際威脅的那些」要優先修理。在[美國國家標準暨技術研究院（NIST）NVD CVE API 2.0](https://nvd.nist.gov/developers/vulnerabilities)文檔中，介紹回傳參數 `cisaActionDue` 那一段落，也有提到可以參考 KEV 目錄中列出的漏洞優先修復
{{</alert>}}

最後是關於資安修補相關的法律，可以參考 BOD 系列：[BOD 26-04](https://www.cisa.gov/news-events/directives/bod-26-04-prioritizing-security-updates-based-risk)，這裡不贅述。

# [EPSS(The Exploit Prediction Scoring System)](https://www.first.org/epss/)

EPSS 是一種數據驅動的機器學習模型，用於估計已發布的 CVE 在未來 30 天內實際利用的機率。

前面提到的 CVSS 分數，是一套評分機制，可幫助快速判斷漏洞的嚴重性，但只靠這個分數決定修補的順序似乎不太足夠，例如說: 有一個漏洞的 CVSS 9.5 分 和EPSS 預測機率不到 1 % ; 另一個漏洞 6 分但 EPSS 預測機率 90 % ，這兩個修復的順序性，就有待討論了。現代評估漏洞風險的最新標準，基本會結合

- CVSS(Common Vulnerability Scoring System)
- KEV(Known Exploited Vulnerabilities)
- EPSS(The Exploit Prediction Scoring System)

這三項來決定真實的修補優先順序。這邊就可以有簡單到複雜的定義，例如簡單一些可以定義：

- 已列 CISA KEV ➔ 無論 CVSS 幾分，代表已經有實際災情，必須立刻處理
- 高 CVSS 分數 ＋ 高 EPSS 機率 ➔ 破壞力強且駭客極可能發動攻擊，盡快修補
- 高 CVSS 分數 ＋ 極低 EPSS ＋ 未上榜 KEV ➔ 雷聲大雨點小，排入每月的例行性更新即可



---

# API 使用

按照 NVD CVE API 2.0 官網 api 介紹，查一下上面提到的 php 漏洞 `CVE-2026-109619` 吧！ api 給出的訊息更加完整，但也可以簡單參考 NVD 網頁上的整理內容：[CVE-2026-109619](https://nvd.nist.gov/vuln/detail/cve-2026-109619)。按照官方文件寫出的 query url 如下：

> https://services.nvd.nist.gov/rest/json/cves/2.0?cveIds=CVE-2026-109619

以下一些 take ways：

- 一開始直接使用 AI 來得出 query url，但後續看文件之後發現問題。從文件知道 `cveId optional (deprecated, see cveIds)`，但是 AI 還是都使用 `cveId` ，若我改成 `cveIds` 會我說用法錯誤

- 既然是使用別人家的 api ，就要[注意 Rate Limits 和 API Key](https://nvd.nist.gov/developers/start-here) 相關事情：
  - 速率限制（不使用 API 金鑰）: `5` requests in a rolling 30 second window
  - 使用 API 金鑰後: `50` requests in a rolling 30 second window
  - 建議使用者在兩次請求之間讓腳本 `6s` (Best practice 的建議)
  - parameter 參數那邊的 `resultsPerPage` 預設值和上限都是 `2000`

- 由於 CVSS 這套評分標準本身改版好幾次，因此查出資料的 metrics 有多個版本，主流是 `cvssMetricV31`，但新版 `cvssMetricV40` 也越來越多 CNA 採用 ; 同一版本也可能有多個 source/type 各自評分

- 實務上，`cisaActionDue`、和 `cisaRequiredAction` 還蠻重要的。如果產品和**聯邦民事行政部門 (FCEB) 機構相關**，就要按照 `cisaRequiredAction` 的指示，在 `cisaActionDue` 期限內修好

- query parameter 一些和 KEV 相關的參數如：`hasKev`、`kevStartDate`、`kevEndDate`

從 KVE 的 Json 檔中找出目標資訊也蠻簡單的：
```json
{
    "cveID": "CVE-2026-109619",
    "vendorProject": "PHP Group",
    "product": "PHP",
    "vulnerabilityName": "PHP Stale Internal Request-State Vulnerability",
    "dateAdded": "2026-10-26",
    "shortDescription": "PHP contains a stale internal request-state vulnerability that allows for unauthenticated remote code execution.",
    "requiredAction": "Apply mitigations in accordance with vendor instructions, ensuring compliance with CISA\u2019s BOD 26-04 Prioritizing Security Updates Based on Risk (see URL in Notes) guidance and CISA\u2019s \u201cForensics Triage Requirements\u201d (see URL in Notes). Follow applicable BOD 26-04 guidance for cloud services or discontinue use of the product if mitigations are unavailable. Stakeholders are responsible for evaluating each asset's internet exposure and ensuring adherence to BOD 26-04 patching guidelines.",
    "dueDate": "2026-10-29",
    "knownRansomwareCampaignUse": "Unknown",
    "forensicTriage": "Yes",
    "notes": "This vulnerability affects an open-source component, third-party library, protocol, or proprietary implementation that could be used by different products. For more information, please see: https:\/\/www.php.net\/ ; https:\/\/www.cisa.gov\/news-events\/alerts\/2026\/10\/21\/critical-vulnerabilities-php-require-immediate-patching ; BOD 26-04: https:\/\/www.cisa.gov\/news-events\/directives\/bod-26-04-prioritizing-security-updates-based-risk ; Forensics Triage Requirements: https:\/\/www.cisa.gov\/news-events\/directives\/bod-26-04-implementation-guidance-prioritizing-security-updates-based-risk ;  https:\/\/nvd.nist.gov\/vuln\/detail\/CVE-2026-109619",
    "cwes": [
        "CWE-416",
        "CWE-755",
        "CWE-824",
        "CWE-1325",
        "CWE-1419"
    ]
}
```

另外 EPSS api 的使用也是蠻簡單的，官方文件有說，一般是**每分鐘請求數限制為 1000 次**，濫用會被暫時封鎖，且不通知。如果您收到 429 Too many requests 回應，則表示您已超出此限制並已暫時封鎖：
> https://api.first.org/data/v1/epss?cve=CVE-2026-109619

```json
{
  "status": "OK",
  "status-code": 200,
  "version": "1.0",
  "access": "public",
  "total": 1,
  "offset": 0,
  "limit": 100,
  "data": [
    {
      "cve": "CVE-2026-109619",
      "epss": "0.999990000",
      "percentile": "1.000000000",
      "date": "2026-11-24"
    }
  ]
}
```

---

### 參考資料

- [VE 是什麼？CVE 漏洞修補與 CVSS 分數完整指南](https://www.hiyun.com.tw/news/blog/what-is-cve)

- [公共漏洞和暴露 CVE 簡介](https://medium.com/@jieshiun/%E5%85%AC%E5%85%B1%E6%BC%8F%E6%B4%9E%E5%92%8C%E6%9A%B4%E9%9C%B2-cve-%E7%B0%A1%E4%BB%8B-f362626c9fc7)
