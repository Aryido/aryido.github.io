---
title: "Python : Coroutine 的核心 - Event-Loop"

author: Aryido

date: 2025-03-16T22:56:39+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

tags:
  - asynchronous

comment: false

reward: false
---

<!--BODY-->

> Python 的 Coroutine 發展已經逐漸穩定成熟，已經成為了提升 Python 程式效能的優秀解決方案之一，在之前簡單介紹 [Coroutine 和 await/async](https://aryido.github.io/posts/python/async-await/) 時，我們在範例 code 中一直有用到一個 Python buildin 模組 `asyncio`，它提供了一套完整的工具和接口，用於建立非同步應用程式，其核心是 Event-Loop，會追蹤所有註冊的任務，並根據任務的狀態調度它們的執行。 故接著來了解 Event-Loop 和其工作的執行單位 Task 吧 !

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

Task 是 Event-Loop 的調度基本單位，可進行更細粒度的控制如可以被取消、等待或加入任務群組。 **可以將 Task 視為 Coroutine 的再包裝**，這由 `asyncio.create_task()` 能感受到，因為他的輸入參數是一個 Coroutine ，然後經過此方法 Coroutine 會變成 Task 並會註冊到 Event-Loop 內。而讓 Coroutine 變成 Task 的方式有以下方法 :

- `asyncio.create_task()`
- `asyncio.gather()`

Event-Loop 一次只能執行 1 個 Task ，**最常見的頂層運行中的 Task 就是 `asyncio.run()` 啟動的 Coroutine** ，這基本上就是一個 「運行中的 Task」。

接下來如果目前「運行中的 Task」 進入「正在等待執行結果的狀態」時，也就是到 code 的 `await` 位置，由於 `await` 後面可以接 Coroutine 或 Awaitable obj ，故簡單分成兩種狀況 :

- **`await` 後面接 Coroutine** :

  - 這時其實是繼續同步調用直到這個 Coroutine 結束或是遇到下一個 `await` 標記點，「運行中的 Task」並不會交出控制權給 Event-Loop ，**這代表 Asynchronous 可能還不完善**，可以看之後舉例了解

- **`await` 後面接 Awaitable obj** :
  - 這時的情形會比較重要，「運行中的 Task」**會把主控權交還給 Event-Loop** 且暫停

{{< alert warning >}}
注意一下，所有控制權的返回都是**顯性的**，也就是 Event-Loop 其實沒辦法強行從 Task 拿回控制權，都是 Task 主動把控制權交回去。控制權交出的方式如上述所說有:

- 遇到 `await`
- Task 運行結束

所以如果有一個 Task 裡面有**死尋循環**，Event-Loop 就可能會卡死了，要特別小心。
{{< /alert >}}

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

首先差不多 1 秒後 print 了 hello ，再來 2 秒後 print 了 world ，**總共花了 3 秒**。

# 使用 asyncio.create_task()

上面的 code 運行完後發現一個大問題 : 「 print hello 要 1 秒，再來 print world 要 2 秒，總共花了 3 秒 」，竟然沒有同時一起執行 ! ? 這樣 Coroutine 到底有什麼意義 ？ 那這就是 `await` 不足的地方，為了解決這問題 asyncio 有提供了 `create_task()`，範例如下 :

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

上面這個範例的結果，發現**總共只花了 2 秒**，是我們期望的！

這範例最重要的是 `asyncio.create_task()`，代表有把**兩個任務都先添加到 Event-Loop 裡面了**，故 `task1` 和 `task2` 會**同時執行**，因為它們是獨立的 Task，故代表 :

- `await task1` 會等待 task1 完成，但不會影響 task2 的執行
- `await task2` 會等待 task2 完成，確保 main() 不會過早結束

{{< alert warning >}}
上面說明部分可以嘗試著調整 delay 時間和註解掉 `await task2` 來做一些測試，會發現一些特別的事情，例如 :

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    task1 = asyncio.create_task(say_after(3, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    await task1
    #await task2

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

# Terminal :
# started at 18:15:41
# world
# hello
# finished at 18:15:44

```

特別的地方是會先等 2 秒後顯示 world，在等 1 秒顯示 hello ! **雖然沒有 `await task2` 但他還是有被排程！**

再例如 :

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
    #await task2

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

# Terminal :
# started at 18:18:09
# hello
# finished at 18:18:10

```

承前例我們知道 `task2` 是有被排到 Event-Loop 內的，但因為沒有 `await task2`，所以等 1 秒後顯示 hello 後就直接結束了，沒有等 `task2` 完成。
{{< /alert >}}

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

- [注意 await 後接一個 Coroutine 時，會把 Coroutine 轉成 Task 是錯誤的](https://www.youtube.com/watch?v=K0BjgYZbgfE&t=684s)

- [python asyncio 的理解与入门，搞不明白协程？看这个视频就够了。](https://www.youtube.com/watch?v=brYsDi-JajI)

- [Python asyncio 從不會到上路](https://myapollo.com.tw/blog/begin-to-asyncio/)

- [Python\_非同步協程](https://hackmd.io/@yung1231/Bk3HMIxMw)

- [Python 协程和事件循环](https://rgb-24bit.github.io/blog/2019/python-coroutine-event-loop.html)

- [你知道 asyncio 的 event loop 是怎麼 loop 的嗎？談 event loop 的排程與執行](https://myapollo.com.tw/blog/asyncio-how-event-loop-works/)
