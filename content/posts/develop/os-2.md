---
title: Parallel、Concurrent 介紹

author: Aryido

date: 2022-09-26T22:02:41+08:00

thumbnailImage: "/images/others/os.jpg"

categories:
- develop

comment: false

reward: false
---
<!--BODY-->
> 前面介紹了 Program、Process、Thread 差異。再來淺談Parallel、Concurrent 吧! 其實主要是要注意中英文對照...

<!--more-->

---

## Process(進程)：
- 指在系統中正在運行的一個應用程序；Program 一旦運行就是 Process
- Process: 資源分配的最小單位。

## 多進程（multi-processing）
指的是在單一作業系統中使用兩個以上的 CPU 處理器（processor）來在同一時間執行多個 Process

---

## Thread(線程)：
- 系統分配處理器時間資源的基本單元，Process之內獨立執行的一個單元執行流。
- Thread: cpu 的最小執行單位，且它包含在 process 中。

在執行程式時，thread 會將變數保存在 stack 中。 stack 會在程式 runtime 執行，只有它自己可以使用，並不能含其它 thread 共享。 heap 則是 process 中的另一個屬性，它可以被該 process 中的任何 thread 取用，也就是 heap 是共享的記憶體空間

## 多執行緒（multi-threading）
在一個 process 中可以有多個 thread 以 concurrently 或 parallelism 的方式運作


{{<alert warning>}}
當一個 thread 有 memory leak 的情況產生時，它可能會耗盡其它 thread 的資源，進而導致整個 process 停滯
{{</alert>}}

---

# Parallel &  Concurrent

concurrently 並不是真的「同時」運作，而是透過在 context 間快速切換來看起來同時，但實際上一次還是只做一個；parallel 才是真的「同時」處理多個事務。

## Concurrent(併發):
並發指能夠讓多個任務在邏輯上交織執行的程序設計。

多個任務在同一個 CPU 核上按細分的時間片輪流(交替)執行。在 Java 5 中新加的並發 API 的包名是 java.uti.concurrent。

## Parallel(並行):
同一時刻是否有超過一個 job 在運行，但 Parallel 必須有多核才能實現, 否則只能實現併發。所以，單線程永遠無法達到 Parallel 狀態。要達到並行狀態，最簡單的就是利用多線程和多進程。

{{<alert info>}}
注意到 Java 8 的  Collection 除了 stream() 方法外，還有個方法名字叫做  parallelStream(), 為什麼它是 parallel 呢？

因為它使用的線程池是 ForkJoinPool.commonPool, 而這個池的大小是 CPU 內核數減一，主線程已經佔了一個核，希望 的是 parallelStream() 中每個任務都能理想的分配到不同的 CPU 內核上去並行執行。
{{</alert>}}

## scheduling

在同一時間點只能有一個 thread 去存取某個資料，以特定順序的方式執行 multiple threads 稱作「scheduling」

---