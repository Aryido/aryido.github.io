---
title: "Python : Coroutine 的核心 - Event-Loop"

author: Aryido

date: 2025-03-16T22:56:39+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - python

tags:
  - asynchronous

comment: false

reward: false
---

<!--BODY-->

> Python 的 Coroutine 發展已經逐漸穩定成熟，已經成為了提升 Python 程式效能的優秀解決方案之一，在之前簡單介紹 [Coroutine 和 await/async](https://aryido.github.io/posts/python/async-await/) 時，我們在範例 code 中一直有用到一個 Python buildin 模組 `asyncio` ，我們必須要利用它才能讓 Coroutine 執行非同步運作，這是因為 Coroutine 是無法直接運行的，只有變成了 Task 才能被管理執行。而要讓 Coroutine 運行的話必須要達成以下事情 :
>
> - **啟動 async 模式，讓 Event-Loop 控制程序的狀態**
> - **需 Coroutine 變成 Task 傳給 Event-Loop**
>
> `asyncio` 模組它的核心是 Event-Loop ，故接著來了解 Event-Loop 和其工作的執行單位 Task 吧 !

<!--more-->

還是使用之前的範例 :

```python
import asyncio

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)
    await asyncio.sleep(delay)

async def do():
    await say_after(1, 'hello')
    print("####### finished do function #######")

asyncio.run(do())

```

之前一直都有提到 Coroutine 是無法直接運行的，故這邊使用了 `asyncio.run(do())` 程式才開始運行，那 `asyncio.run()` 做了什麼呢？

> #### 啟動 `asyncio.run(main())`的作用是會把這個 main() 協程轉成 Task 放到 Event-Loop 內

接下來解釋上面提到的兩個關鍵字: 「Event-Loop」、「Task」

# Event-Loop(事件迴圈)

Event-Loop 是個 [Python 類別 BaseEventLoop](https://github.com/python/cpython/blob/70d71fb4eac2c728c8a6636482f537d5367bbed8/Lib/asyncio/base_events.py#L409) ，關鍵運作部分是有一個[無窮迴圈](https://github.com/python/cpython/blob/70d71fb4eac2c728c8a6636482f537d5367bbed8/Lib/asyncio/base_events.py#L627)，可不斷地 loop 來進行排程或執行其他 Task ，而 Event-Loop 將任務分成 2 種狀態 : 「scheduled」 和 「ready」:

- 只有處於 ready 狀態的任務才會被執行
- scheduled 狀態的任務，會看是否到達到/超過預定時間，是的話才會轉成 ready 狀態

Python 3.7 之後 Event-Loop 做了蠻好的封裝，官網上也說**原則上盡量都使用 `asyncio.run()` 來調用 Event-Loop 就可以了，盡量不要直接控制 Event-Loop 這個物件**。

# Task

前面文章有提到讓 Coroutine 變成 Task 的方式有 :

- 使用 `await` 關鍵字
- `asyncio.create_task()`
- `asyncio.gather()`

{{< alert warning >}}
await 後面必須接一個 Coroutine 或是 awaitable(之後會再解釋)
{{< /alert >}}

Event-Loop 一次只能執行 1 個 Task，如果該 Task 進入「正在等待執行結果的狀態」時(也就是到 code 的 `await` 位置)，那麼:

- 該 Task 會把主控權交還給 Event-Loop 且暫停(suspend)
- 再來 Event-Loop 會將等待的任務也放進排程中，然後在任務列表(deque)內選擇可執行的 Task 再接著執行

**可以將 Task 視為 Coroutine 的再包裝**，這可以由 `asyncio.create_task()` 感受到，因為他的輸入參數是一個 Coroutine ，然後經過此方法 Coroutine 會變成 Task 並會註冊到 Event-Loop 內。

# Awaitables

在使用 `await` 關鍵字或是 `asyncio` 相關函式時，經常可以在文件中看到 awaitables 關鍵字，其實他代表了以下 Python 物件 :

- Coroutines
- Tasks (asyncio.Task)
- Futures (asyncio.Future)

那這些有什麼小差別呢，下面也會簡介一下。

---

上面是一個簡單的 **single task** 範例，但注意 「 Event-Loop 的核心是可以有很多個 task ，然後 Event-Loop 它會決定哪個 task 要來運行 」。所以接下來是要討論在 async 模式下多個 task 有哪些運行情況 ?

# 單純多個 await 情況

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

# Terminal:
# started at 21:53:03
# hello
# world
# finished at 21:53:06
```

首先差不多 1 秒後 print 了 hello ，再來 2 秒後 print 了 world ，**總共花了 3 秒**。那來還原一下過程 :

- > `asyncio.run(main())` 啟動了 asyn 模式，並把這個 `main()` 轉為一個 Task 放到 Event-Loop 內

- > Event-Loop 任務列表內只有一個 main 這個 Task，所以開始運行 `main()`

- > `main()` 首先會運行 `print started at` ，再來運行 `await say_after(1, 'hello')`

- > `await` 的作用會把 `say_after(1, 'hello')` 這個 Coroutine 轉變成 Task 然後註冊到 Event-Loop 任務列表內
  >
  > `main()` 同時也會告訴 Event-Loop 它會等待 `say_after(1, 'hello')` 的結果，這時 **main 會把控制權還給 Event-Loop**

- > 現在 Event-Loop 任務列表內有兩個 Task :
  >
  > - `main()` : **要等待 `say_after(1, 'hello')`**
  > - `say_after(1, 'hello')`
  >
  > 目前 Event-Loop 只能讓 `say_after(1, 'hello')` 運行

- > `say_after(1, 'hello')` 內有 `await asyncio.sleep(1)`，故 `asyncio.sleep(1)`也會變成 Task 然後註冊到 Event-Loop 任務列表
  >
  > `say_after(1, 'hello')` 同時也會告訴 Event-Loop 它會等待 `asyncio.sleep(1)` ，這時 **say_after 會把控制權還給 Event-Loop**

- > 現在 Event-Loop 內有三個 Task :
  >
  > - `main()` : **要等待 `say_after(1, 'hello')`**
  > - `say_after(1, 'hello')` : **要等待 `asyncio.sleep(1)`**
  > - `asyncio.sleep(1)`
  >
  > Event-Loop 因為也沒有其他 Task 可以執行，故等待 1 秒後 sleep 運行結束並移除掉這個 Task

- > 現在 Event-Loop 內有兩個 Task :
  >
  > - `main()` : **要等待 `say_after(1, 'hello')`**
  > - `say_after(1, 'hello')`
  >
  > 所以 Event-Loop 只能讓 `say_after(1, 'hello')` 繼續運行且只剩下 print 了 hello 的部分， **say_after 打印完成後結束並移除掉這個 Task，然後把控制權還給 Event-Loop**

- > Event-Loop 只剩下 main 這個 Task，所以繼續運行 `main()` ，而接下來的剩下部分有 `await say_after(2, 'world')`，其流程和 `await say_after(1, 'hello')` 一樣，故不再贅述

- > Event-Loop 把 `await say_after(2, 'world')` 任務做完後，還是剩下 main 這個 Task，繼續運行剩下的 `print finished at` ， **main 打印完成後結束並移除掉這個 Task，然後把控制權還給 Event-Loop**

- > 最後 Event-Loop 內沒有任何任務，所以程式結束

{{< alert warning >}}
注意一下，所有控制權的返回都是**顯性的**，也就是 Event-Loop 其實沒辦法強行從 Task 拿回控制權，都是 Task 主動把控制權交回去。控制權交出的方式如上述所說有:

- `await`
- Task 運行結束

所以如果有一個 Task 裡面有**死尋循環**，Event-Loop 就可能會卡死了，要特別小心。
{{< /alert >}}

# 使用 asyncio.create_task()

上面的 code 運行後我們發現一個大問題 : 「 print hello 要 1 秒，再來 print world 要 2 秒，總共花了 3 秒 」，竟然沒有同時一起執行 ! ? 這樣 Coroutine 到底有什麼意義 ？

那這邊就是 `await` 不足的地方，為了解決這問題 asyncio 有提供了 `create_task()`，範例如下 :

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    task1 = asyncio.create_task(say_after(1, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

# Terminal:
# started at 00:00:15
# hello
# world
# finished at 00:00:17
```

上面這個範例的結果，發現**總共只花了 2 秒**，是我們期望的！ 那也來還原一下過程 :

- > `asyncio.run(main())` 啟動了 asyn 模式，並把這個 main() 作為一個 Task 放到 Event-Loop 內

- > Event-Loop 發現只有一個 main 這個 Task，所以先開始運行這個 main()

- > main() 首先會創建 `task1` 和 `task2` 這兩個 Task ，並且最重要的是**會把這兩個 「 Task 註冊到 Event-Loop 任務列表內 」**，這時控制權還在 main 上，故繼續會 `print started at`，接下來執行 `await task1`

- > 由於 `await` 後面直接就是是一個 Task 了，且已經註冊到 Event-Loop 任務列表內，故就省略了變成 Task 和註冊這些步驟
  >
  > **main 也會告訴 Event-Loop 它要等待 `task1` 並把控制權還給 Event-Loop**

- > 現在 Event-Loop 內有直接有三個 Task :
  >
  > - `main()` : **要等待 `task1`**
  > - `task1`
  > - `task2`
  >
  > Event-Loop 可能讓 `task1` 或 `task2` 運行

無論是執行 `task1` 或 `task2` 哪一個，由於內部都有 `asyncio.sleep()`，故都會馬上返回控制權給 Event-Loop ，而馬上會執行另一個 Task :

> #### 因此 `asyncio.sleep(1)` 和 `asyncio.sleep(2)` 基本上會同時進行，所以只需要等 2 秒就完成這個 code 了。

不太相信的話可以來隨機分析一遍，**如果是 `task2` 先運行** :

- > `task2` 內有 `await asyncio.sleep(2)`，故 `asyncio.sleep(2)` 也會變成 Task 然後註冊到 Event-Loop 任務列表內
  >
  > `task2` 同時也會告訴 Event-Loop 它會等待 `asyncio.sleep(2)` ，這時會把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 `task2`**
  > - `task1`
  > - `task2`: **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(2)`
  >
  > Event-Loop 可能讓 `task1` 或 `asyncio.sleep(2)` 運行

**如果是 `asyncio.sleep(2)` 運行** :

- > `sleep(2)` 開始執行後就休息了，所以馬上會把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 `task2`**
  > - `task1`
  > - `task2`: **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(2)` **要等待 2 秒**
  >
  > Event-Loop 只能執行 `task1`

- > `task1` 內有 `await asyncio.sleep(1)`，故 `asyncio.sleep(1)` 也會變成 Task 然後註冊到 Event-Loop 內
  >
  > `task1` 同時也會告訴 Event-Loop 它會等待 `asyncio.sleep(1)` ，這時會把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 `task2`**
  > - `task1` : **要等待 `asyncio.sleep(1)`**
  > - `task2` : **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(1)`
  > - `asyncio.sleep(2)` **要等待 2 秒**
  >
  > Event-Loop 只能執行 `asyncio.sleep(1)`

- > `sleep(1)`開始執行後馬上會把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 `task2`**
  > - `task1` : **要等待 `asyncio.sleep(1)`**
  > - `task2` : **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(1)` : **要等待 1 秒**
  > - `asyncio.sleep(2)` : **要等待 2 秒**
  >
  > 故基本上 `sleep(1)` 和 `sleep(2)` 是同時的，然後等待 1 秒後 `asyncio.sleep(1)` 首先運行結束並移除掉這個 Task，然後把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 `task2`**
  > - `task1`
  > - `task2` : **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(2)` : **還要再等待 1 秒**
  >
  > Event-Loop 只能執行 `task1` 且只剩下 print 了 hello，打印完成後結束並移除掉這個 Task，然後把控制權還給 Event-Loop

{{< alert warning >}}
特別注意到這裡時，其實 `asyncio.sleep(2)` 已經休息 1 秒了，故只要在等待 1 秒就完成休息了
{{< /alert >}}

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` **要等待 `task2`**
  > - `task2` : **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(2)` : **還要再等待 1 秒**
  >
  > Event-Loop 只能執行 `main()` ，其下一步是 `await task2`，但由於 `task2` 已經在 Event-Loop 任務列表內， 故不用執行什麼， main 直接把控制權還給 Event-Loop

- > 現在 Event-Loop 這些 Task :
  >
  > - `main()` : **要等待 task2**
  > - `task2` : **要等待 `asyncio.sleep(2)`**
  > - `asyncio.sleep(2)` : **還要再等待 1 秒**
  >
  > 再等待 1 秒後 `asyncio.sleep(2)` 也結束了，移除掉這個 Task，然後把控制權還給 Event-Loop

- > 現在 Event-Loop 內有這些 Task :
  >
  > - `main()` : **要等待 task2**
  > - `task2`
  >
  > Event-Loop 只能讓 `task2` 運行，且只剩下 print 了 world ， **`task2` 打印完成後結束並移除掉這個 Task，然後把控制權還給 Event-Loop**

- > Event-Loop 發現只剩 main 這個 Task，所以繼續運行 main() 剩下的部分，而接下來是運行 print finished at ，**main 打印完成後結束並移除掉這個 Task，然後把控制權還給 Event-Loop**

- > 最後 Event-Loop 內沒有任何任務了，程式結束

以上很詳細的把程式可能的一條流程支線給演練出來了，希望可以更了解整個 Python 非同步的執行邏輯。

# Coroutine 的 return value

Coroutine 的返回值也是蠻重要的，這該怎麼做呢 ？ 而 `await` 有一個重要功用是把 Coroutine 或 Task 的返回值拿出來，故首先我們修改一下原始 code :

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{delay} - {what}"

async def main():
    task1 = asyncio.create_task(say_after(1, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    result1 = await task1
    result2 = await task2

    print(result1)
    print(result2)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

這裡展示一個簡單範例，該怎麼拿到 `async def` 函數的返回值。

# 使用 asyncio.gather()

承上 code 應該會思考，難道我有 10 個 Task 我就要寫 10 行 await 嗎 ？ 在這裡 `asyncio` 提供了 `gather()` 來簡化這個問題。 但要注意 `gather()` 並不是一個 Coroutine Function，它會返回一個叫 Future 的東西，而 Future 也是需要 `await` 的。

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{delay} - {what}"

async def main():
    task1 = asyncio.create_task(say_after(1, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    result = await asyncio.gather(task1, task2)

    print(result)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

`asyncio.gather()` 的作用是可**執行多個 Coroutine，並可收集每個的 return value 存於 list 中**， 更進一步來說 `gather()` 輸入參數其實可以是多個 Coroutine 、Task 或者 Future，故可以不用特地把 Coroutine 轉成 Task 再放入 gather() 內也沒關係，因此可以再把 code 簡化成 :

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{delay} - {what}"

async def main():

    print(f"started at {time.strftime('%X')}")

    result = await asyncio.gather(say_after(1, 'hello'), say_after(2, 'world'))

    print(result)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

---

### 參考資料

- [【python】asyncio 的理解与入门，搞不明白协程？看这个视频就够了。](https://www.youtube.com/watch?v=brYsDi-JajI)

- [Python asyncio 從不會到上路](https://myapollo.com.tw/blog/begin-to-asyncio/)

- [Python\_非同步協程](https://hackmd.io/@yung1231/Bk3HMIxMw)

- [Python 协程和事件循环](https://rgb-24bit.github.io/blog/2019/python-coroutine-event-loop.html)

- [你知道 asyncio 的 event loop 是怎麼 loop 的嗎？談 event loop 的排程與執行](https://myapollo.com.tw/blog/asyncio-how-event-loop-works/)
