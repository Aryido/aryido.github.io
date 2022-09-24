---
title: Program、Process、Thread 差異

author: Aryido

date: 2022-09-24T18:07:17+08:00

thumbnailImage: "/images/others/os.jpg"

categories:
- os

comment: false

reward: false
---
<!--BODY-->
> Program/Process/Thread 是面試時經常會被問到的題目，就來筆記一下吧。

<!--more-->

# Program (程式、程序)
我們在 IDE 內所寫的 code，還尚未 load 入記憶體的 code，我們稱之為Program。
{{<alert warning>}}
- 相同 Program 的 Process 可以多個同時存在
{{</alert>}}

# Process ( 程序、進程 )
Process 是已經執行並且 load 到記憶體中的 Program 實例。在實際生活中，點開應用程式就是將 Program 活化成 Process ，因此我們可以在 monitor 中看到PID。
{{<alert warning>}}
- Process 是電腦中已執行 Program 的實例
- 每一個 Process 是互相獨立的
- Process 本身不是基本執行單位，而是 Thread 的容器
- Process之間可通過 TCP/IP 端口實現交互
- 一個 CPU 一次只能執行一個 Process，因此如何排程(Scheduling) 、如何管理記憶體是 OS 所關注的事
{{</alert>}}

# Thread (執行緒、線程 )
Process 是 Thread 的容器，在同一個 Process 中可以有很多個 Thread。以聊天室 Process 為例，可以同時接受對方傳來的訊息以及發送自己的訊息給對方，就是同個 Process 中不同 Thread 的功勞。
{{<alert warning>}}
- Thread 是 OS 能夠進行運算調度的最小單位
- 同一個 Process 會同時存在多個 Thread
- Process 底下的 Thread 共享資源，如記憶體、變數等，通過共享的內存空間來進行交互。
- Multithreading 可能同時存取或改變Global Variable，則可能發生 Synchronization 問題。若執行緒之間互搶資源，則可能產生 Deadlock
{{</alert>}}

## Program V.S Process
- Program 是存在硬碟中的**靜態code**，可以長時間的保存在系統中。

- Process 是 Program 運行的過程,存在着生命週期, Process 會隨着程序的終止而銷毀。 CPU 讀取硬碟中的 code 到內存中,在內存中的可執行的 Program 實例就是 Process。

## 場景
例如打開 word ，這就是開啓一個 Process，當我們使用鍵盤在某一行打了一些文字，我們需要一個 thread 來接受鍵盤的按下事件; 接下來需要一個 thread 來幫你把文檔排序把畫面顯示出來; 再來是可能需要儲存到硬碟中，故也需要一個 thread 來完成操作。以上就是常見的多執行續場景。

---