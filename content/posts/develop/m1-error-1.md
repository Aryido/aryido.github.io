---
title: Apple M1 作業系統坑 - CPU 簡介

author: Aryido

date: 2023-01-04T23:02:30+08:00

thumbnailImage: "/images/others/m1.jpg"

categories:
- develop

comment: false

reward: false
---
<!--BODY-->
> 現在公司很多都會給新進員工配上 Apple M1 筆電，整體筆電用起來都還不錯的。但因為 Apple M1 底層處理器架構大改變，對於軟體開發在本地端測試時候，常發生一些不可預期的狀況。這邊就來記錄一下有遇到的 BUG。

<!--more-->

---

隨著 Apple 推出最新的 ARM 架構晶片，那 **ARM架構** 和 **x86架構** 有什麼不同呢 ? 以下簡單介紹兩者之間的差異。

## x86架構 (基於 Intel 8086 且向下相容的 CPU 指令集架構)
x86 屬於 **複雜指令集計算機(CISC)** 架構。(Complex Instruction Set Computer)
- **Pros :**

    因為是讓新系統使用一個包含舊系統的指令集合，故好處是可以兼容舊系統。x86平台還有大量相容的軟體生態

- **Cons :**

    缺點像是耗電、高發熱量。另外 x86 也會先用解碼器先將複雜指令變成類似於RISC的指令，再給核心執行，整體上執行工作效率會變差，處理資料速度也會比較慢。

那 AMD64 又是甚麼呢 ? 歷史故事是說 Intel 先有做出 64 位，但 Intel 64 位不兼容 32 位，結果市場反響不好。AMD 公司看到機會，開發出向下兼容 x86 架構的 64 位架構。而 x86-64 是基於 x86 架構的 64 位元並向後相容於 16 位元及 32 位元的x86架構，但因為使用 AMD 的設計，故也稱為「AMD64」。

{{< alert info >}}
x86-64 或 x64 或 AMD64 其實都是差不多一樣的東西，稱呼不同是政治的原因...
{{< /alert >}}

## ARM架構 ( Advanced RISC Machine )
ARM 屬於 **精簡指令集計算機(RISC)** 架構。(Reduced Instruction Set Computing)
- **Pros :**

    RISC 在相同晶片條件下的執行速度是CISC 的 2-4 倍。且晶片體積縮小、降低功耗。
- **Cons :**

    因為晶片上只有基本且簡單的指令，工程師在寫程式碼執行相同任務時，RISC架構相較 CISC 架構就需要寫更多程式。另外 ARM 平台軟體生態還沒有很齊全。

{{< alert danger >}}
因為底層晶片架構不一樣，所以 ARM 編譯的程式 x86 / x64 上不行跑，反過來說 x86 / x64 編好的程式也不能拿去 ARM 上面跑。
應用程式必須針對它們運行的 CPU 架構進行編譯。
{{< /alert >}}

---

## 筆記

- MAC M1 是 ```ARM 架構```
- 其他筆電常用 ```x86 、 AMD64```
