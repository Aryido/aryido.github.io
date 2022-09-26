---
title: Parallel、Concurrent 介紹

author: Aryido

date: 2022-09-26T22:02:41+08:00

thumbnailImage: "/images/others/os.jpg"

categories:
- os

comment: false

reward: false
---
<!--BODY-->
> 前面介紹了 Program、Process、Thread 差異。再來淺談Parallel、Concurrent 吧! 其實主要是要注意中英文對照...

<!--more-->

# 前提整理
## Process(進程)：
- 指在系統中正在運行的一個應用程序；Program 一旦運行就是 Process
- Process: 資源分配的最小單位。

## Thread(線程)：
- 系統分配處理器時間資源的基本單元，或者說Process之內獨立執行的一個單元執行流。
- Thread: 程序執行的最小單位。

Process 要分配一大部分的內存，而 Thread 只需要分配一部分棧就可以了。 一個程序至少有一個 Process , 一個 Process 至少有一個 Thread。

---

## Concurrent(併發):
並發指能夠讓多個任務在邏輯上交織執行的程序設計。

多個任務在同一個 CPU 核上按細分的時間片輪流(交替)執行，從邏輯上來看那些任務是同時執行。針對 CPU 內核來說，任務仍然是按細粒度的串行執行。也難怪在 Java 5 中新加的並發 API 的包名是 java.uti.concurrent。

## Parallel(並行):
同一時刻是否有超過一個 job 在運行，但 Parallel 必須有多核才能實現, 否則只能實現併發。所以，單線程永遠無法達到 Parallel 狀態。要達到並行狀態，最簡單的就是利用多線程和多進程。

注意到 Java 8 的  Collection 除了 stream() 方法外，還有個方法名字叫做  parallelStream(), 為什麼它是 parallel 呢？因為它使用的線程池是 ForkJoinPool.commonPool, 而這個池的大小是 CPU 內核數減一，主線程已經佔了一個核，希望 的是 parallelStream() 中每個任務都能理想的分配到不同的 CPU 內核上去並行執行。

---