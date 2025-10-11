---
title: "Cron "

author: Aryido

date: 2025-07-22T23:47:43+08:00

thumbnailImage: "/images/linux/logo.jpg"

categories:
  - language
  - shell-script

comment: false

reward: false
---
<!--BODY-->
> 在軟體工程中，有蠻多工作都會需要排程的，而 Linux 排程是透過 crontab 與 at 這兩個，這兩個有啥異同呢？ 我們可以發現工作排程的方式基本上分成：
> - 例行性的 : 每隔一定的週期就會需要處理，那就可以使用 `rontab` 這個指令設定的任務循環
> - 突發性的 : 會訂一個時間點執行，但做完以後就沒有了，那就可以使用 `at` 指令處理僅執行一次就結束的任務
>
> Cron 是 Linux 系統下的一個定時任務管理服務，為了要精確表示「何時」執行， 就發明了一個抽象的 Cron 表達式，相信大家都有看過類似 `0 12 * * *` 這種在定時任務中常出現的寫法，像 「 GCP Cloud Scheduler」、「 GitHub Actions」、「 SpringBoot 的 @Scheduled」，都以 Cron 表達式來定義任務觸發時間點。

<!--more-->

---

循環執行例行性工作，是由 cron (crond) 這個系統服務來控制的，想要建立循環型工作排程時，使用的是 crontab 這個指令。排程依照層級設定可分成:

- **系統層級** : 要寫在 `/etc/crontab` 裡面，可以**指定哪個使用者**去執行工作

- **使用者層級** : 每個使用者自己管理的排程，只能用自己的身份去跑


# 基本 CLI

crontab 的常用指令非常簡單：

```bash
# 編輯 edit，修改後的 crontab 將會自動安裝
crontab -e
# 列出 list
crontab -l
# 刪除
crontab -r

# 查看系統層級 crontab
sudo cat /etc/crontab

# 編輯其他使用者的排程
sudo crontab -u <user> -e

```

比較重要的是 `-u` 參數， `−u` 指定要調整 crontab 的使用者。如果沒有此選項，crontab 會以**當前使用者**執行crontab。那特別注意，有時候可能有使用 `sudo su - aryido`，這代表：

- `sudo` : 以 root 身分執行指令
- `su` : 代表 switch user，切換使用者
- `-` : 表示會載入目標使用者的登入環境（例如 .bash_profile, .bashrc）
- `aryido` : 代表目標使用者帳號

由於切換環境，故會混淆 crontab，因此如果 `su` 之後，建議都要使用 -u 選項。

---

# 排程時間語法

例如說在 github action 的 yaml 會看到 : 
```
on:
  schedule: 
    - cron: "0 0 */2 * *" # every 2 days
```

這就是 crontab 比較需要額外記憶的時間語法，可用於指定分鐘、小時、月份中的日期、月份和星期幾，然後指定按該間隔運行命令，含義如下圖所示：

{{< image classes="fancybox fig-100" src="/images/linux/crontab.jpg" >}}


| 符號                                           | 含意                              |
| --------------------------------------------- | --------------------------------- |
| `*`                                           | 任意值（每個時間單位都執行）          |
| `,`                                           | 多個值，例如 `1,15` 代表第 1 和第 15 |
| `-`                                           | 範圍，例如 `1-5` 代表 1 到 5       |
| `/`                                           | 間隔，例如 `*/10` 表示每 10 一次  |


雖然看起來大概知道規則了，但偶爾還是會有一些小錯誤發生，以下筆記一些注意事項。最後都會建議寫完表達式之後，可以使用一些檢查網站來確認自己的排程時間有沒有符合當初的定義，例如[測試網站](https://tool.lu/crontab/) : 


- ### 注意 0 是週日，1 - 6 是週一到週六

這個初期最容易踩的坑，例如說想要在 `每週日的早上 7 點執行任務`，要寫成
```
0 7 * * 0
```


- ### 儘量避免寫 0 點 0 分這種排程數字

因為 0 點 0 分可以代表一天的開始或是一天的結束，這個實務上含義差蠻多的，所以如果是要進行「當天結算的任務」，可能改成 `23:59` 會比較好，使用
```bash
# 在每週五的 0 點 0 分執行任務
0 0 * * 5
```
這種會不確定真實含義，可以避開這種表達方式。

- ### 最常見的表達式為「5 個 *」，不過 Spring Boot  @Scheduled 採用「6 個 *」

[spring boot cron 有到秒](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#cron())，雖然更加細膩，但由於和其他傳統寫法不一樣，不小心會寫錯。


- ### 每個偶數月份都要執行的情況

在小時及分鐘的部分 `*/2` 是代表 `2,4,6,8,10..` ，這部分是經常用的。但是由於月份是 `1-12`，故**在月份的部分 */2**其實是代表 `1,3,5,7,9,11` 月的。假設若有個需求是

> 在每年的 `2、4、6、8、10、12` 月 `25～31` 號之間，每天凌晨 `2:17` 執行一次指定的指令

則應該要寫成 ：
```bash
17 2 25-31 2-12/2 *
# or
17 2 25-31 2,4,6,8,10,12 *
```

若寫成 `17 2 25-31 */2 *` 換變成是**每年的奇數月**。

---

### 參考資料

- [第十五章、例行性工作排程(crontab)](https://linux.vbird.org/linux_basic/centos7/0430cron.php)

- [Cron 是什麼？定時任務的語法怎麼寫？](https://kucw.io/blog/cron/)

- [Linux-系統排程,cron運行與錯誤日誌檢查](https://www.youtube.com/watch?v=VKzsCbR2-AU)

- [Anatomy of crontab!](https://ahmadawais.com/setup-cron-in-unix-basic-understanding/)
