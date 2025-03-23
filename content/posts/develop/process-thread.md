---
title: "淺談 Process、Thread"

author: Aryido

#date: 2022-09-24T18:07:17+08:00
date: 2025-03-13T19:27:31+08:00

thumbnailImage: "/images/others/os.jpg"

categories:
  - develop

tags:
  - asynchronous

comment: false

reward: false
---

<!--BODY-->

> 電腦運行時**任務的的單元是什麼呢**？ 這個涉及了「程式(Program)」、「進程(Process)」、「線程(Thread)」的概念，是面試時經常會被問到的題目，首先**默念默背**一下教科書上 Process 和 Thread 的簡單定義：
>
> - **Process： 資源分配的最小單位**
> - **Thread： CPU 執行的最小單位**
>
> 在實際生活中如點開一個聊天應用程式，這就是將 Program 活化成 Process 的例子，因此我們可以在電腦的資源管理器 Monitor 中看到 PID (Process ID) ; 再繼續以聊天室 Process 為例，我們可以同時「接受對方傳來的訊息」以及「發送自己的訊息給對方」，這就是同個 Process 中不同 Thread 的功勞。

<!--more-->

---

# Program - 常見翻譯 : 程式、程序

我們在 IDE 內所寫的且還尚未 load 入記憶體的 code 我們稱之為 Program，然後當 CPU 讀取硬碟中的 code 到 in-memory 中運行就是 Process，在 in-memory 中的可執行的 Program 實例就是 Process 。
{{<alert info>}}
若以 Java 的物件導向的觀念來比擬， Program 可以想成相當於 Java 的 Class 功能。而相同 Program 的 Process 可以多個同時存在，如同 Java 的 Class 一樣，是可以一直 new 出新的 instance 的
{{</alert>}}


---

# Process - 常見翻譯 : 進程

> **Process 是一個正在執行的 program 實例。它擁有獨立的記憶體空間(address space)和資源環境，是作業系統分配資源的最基本單位。**

- Operating System Principles :
  - 原則上一顆 CPU 一次只能執行一個 Process ，因此 Process 的使用需要被排程(Scheduling) => **恐龍本第五章**
  - 每個 Process 間在記憶體中 Memory-Management 是要注意的事情 => **恐龍本第八章**

{{<alert success>}}

- 每一個 Process 基本是互相獨立的
- Process **不是**基本執行單位，而是 Thread 的容器
- Process 之間一般來說要能溝通，要通過如 TCP/IP 端口、或者 socket 實現交互
- 多進程（multi-processing）在作業系統中，經常使用多個 CPU 來在同一時間上同時並行多個 Process

{{</alert>}}

## Program V.S Process

- Program 是存在硬碟中的**靜態 code**，可以長時間的保存在系統中

- Process 是 Program 運行的**實例**，存在着生命週期， Process 會隨着程序的終止而銷毀

---

# Thread - 常見翻譯 : 執行緒、線程

> **Thread 是 CPU 進行運算調度的最小單位，也是 Process 內部的最小執行單位，可與其他 Thread 共享一些資源，也有自己的執行狀態如 Stack、程式計數器（Program Counter, PC）等等。**

- Operating System Principles
  - 一個 Process 的 Global Variable 可以讓它的所有 Thread 共享，而每個 Thread 自己也有自己的專屬 Variable => **恐龍本第四章**
  - 多個 Thread 要存取同一個 Global Variable，為了確保在存取共享資源時，能夠按照預期順序執行以維持資料一致性與安全性，故考慮加「鎖」 => **恐龍本第六章**
  - 死結(Deadlock) => **恐龍本第七章**

{{<alert success>}}

- Process 底下的 Thread 可通過 Heap 這個 Process 中的一個屬性，來共享資源如記憶體、變數等

- Thread 會將變數保存在 Stack 中，而 Stack 內的變數只有它「自己這個 Thread 」可以使用，並不能給其它 Thread 

- 當一個 Thread 有 memory leak 的情況產生時，它可能會耗盡其它 Thread 的資源，進而導致整個 Process 停滯

- 多執行緒（multi-threading）是在一個 Process 中有多個 Thread，能夠以 **Concurrency** 的方式運作

- Multi-Thread 若其之間互搶資源，則可能產生 Deadlock

{{</alert>}}

## Process V.S Thread

{{< image classes="fancybox fig-100" src="/images/others/concurrency-asynchronous/process-thread.jpg" >}}

- Process 是 Thread 的容器， Process 中至少有一個 Thread ; 一個 Thread 只能屬於一個 Process

- 同一個 Process 內的 Thread 之間可以直接交流；但兩個 Process 想通訊，還需要須通過其他中間代理來實現

- Process 有獨立的地址空間，Thread 沒有單獨的地址空間

---

# 場景

電腦操作系統的各種基本功能的運行，是由多個「系統進程」來達成的，再來由 user 自己啓動的如 Word 文書應用程式，這又開啓一個「用戶進程」。再進一步的 Word 應用程式是可以同時進行「打字」，這需要一個 Thread 來接受鍵盤的按下事件 ; 接下來還需要一個 Thread 來幫把文檔在畫面顯示出來; 還有 「拼寫檢查」、「寫入硬碟儲存」等等工作... 以上這些運作就是由「**多執行緒(Multithreading)**」和「**多進程(Multiprocessing)**」來達成的。

---

### 參考資料

- [进程和线程的区别](https://www.youtube.com/watch?v=e3JQOgKw9BA&t=4s)

- [徹底理解進程、線程、多進程與多線程及其優缺點](https://www.readfog.com/a/1717973144104439808)

- [程序(進程)、執行緒(線程)、協程，傻傻分得清楚！](https://oldmo860617.medium.com/%E9%80%B2%E7%A8%8B-%E7%B7%9A%E7%A8%8B-%E5%8D%94%E7%A8%8B-%E5%82%BB%E5%82%BB%E5%88%86%E5%BE%97%E6%B8%85%E6%A5%9A-a09b95bd68dd)

- [OS筆記-Chapter 7: Deadlocks](https://hackmd.io/@Kipper/r1uOJ-6Gv)