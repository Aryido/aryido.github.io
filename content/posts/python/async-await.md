---
title: "Python : Coroutine 和 await/async"

author: Aryido

date: 2025-03-15T19:46:08+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - python

tags:
  - asynchronous

comment: false

reward: false
---

<!--BODY-->

> 之前有介紹了 [Asynchronous](https://aryido.github.io/posts/develop/concurrency-asynchronous/) ，而 Python 的 **Coroutine 是實現 Asynchronous 的一種設計方式**，且 Python 目前已經有非常直觀簡單的語法糖來定義 Asynchronous Code，使得程式寫起來就像普通的 「 Sequential Processing 順序執行 」任務那樣，但同時卻也可以對**目標函數標註做「等待」的動作，並在「等待」期間可以先去做其他任務**，達成非同步的功效，提高程式的並發性，而其重要的關鍵字就是 :
>
> - **`await` : 用來標記 Coroutine 切換暫停和繼續的位置**
> - **`async` : 用來宣告 function 能夠有異步的功能成為 Coroutine function**
>
> 而這兩個關鍵字是在 `Python3.5` 引入且在 `Python3.7` 成為**保留關鍵字**。它們在著名的 FastAPI 框架下的 path operation function 也經常使用，接下來就簡單介紹一下吧。

<!--more-->

# Coroutine(協程)

Asynchronous code 代表程式語言有辦法在 code 中的某個地方標註等待，且在等待時間中可以另外去做一些其他工作，最後再回來把剩下的事情做完，而 Python 的異步函數常使用 :

- **`async def` 定義**
- **或裝飾上 `@asyncio.coroutine`**

如下程式表示，而這些異步函數也被稱為 「**Coroutine Function**」 :

```python
import asyncio

async def load_file_1(path):
    pass

# Python 3.10 之後停止支援，故較少使用了，目前也不推薦繼續使用
@asyncio.coroutine
def load_file_2(path):
    pass
```

Coroutine Function 在調用時會返回一個 「Coroutine」，這也是 Coroutine 的基本定義，是指 Coroutine Function 返回的物件，它可以「等待」並在等待狀態時，將執行權**讓給**其他 Coroutine，之後可以**再返回來**繼續執行任務剩下的小部分，且可以多次的進行這樣的行為。

{{< alert success >}}
Python 的 Coroutine 可以對應於 Go 的 Goroutine
{{< /alert >}}

運行 Coroutine 是個稍微麻煩的事情，簡單形容的話必須要達成以下事情 :
- #### 要啟動 async 模式，也就是讓 Event-Loop 控制我們程序的狀態
- #### 再來是把 Coroutine 變成 Task 傳給「事件迴圈(Event-Loop)」

故在使用 Coroutine Function 上要注意一些事情 :

- ## 需啟動 async 模式，讓 Event-Loop 控制我們程序之後 Coroutine 才能正確開始調用執行

  由於 Coroutine 本身的特性有別於一般函式，一定要透過 Event-Loop 排程後執行，若直接調用的話**並不會運行任何 Coroutine code** ，例如 :

  ````python
  import asyncio

  async def say_after(delay, what):
      await asyncio.sleep(delay)
      print(what)
      await asyncio.sleep(delay)

  async def do():
      await say_after(1, 'hello')
      print("####### finished do function #######")

  do() # 注意，這裡直接調用了 Coroutine Function，這有問題

  # Terminal:
  # ```
  # RuntimeWarning: coroutine 'do' was never awaited
  #   do()
  # RuntimeWarning: Enable tracemalloc to get the object allocation traceback
  # ```
  ````

由這範例主要的錯誤是沒有正確**進入 async 模式，而最簡單是使用 `asyncio.run()` 就可以進入 async 模式了。**

- ## Coroutine 需變成 Task 給 Event-Loop 才能被排程

    若沒使用 `await` 就呼叫 Coroutine 且也沒有做其他處理，此時 Coroutine 並沒轉變成 Task ，故 Event-Loop 沒辦法排程它，**會無法運行！** 例如 :
    
    ``` python
    import asyncio

    async def say_after(delay, what):
        await asyncio.sleep(delay)
        print(what)
        await asyncio.sleep(delay)

    async def do():
        say_after(1, 'hello') # 注意，這裡沒有用 await
        print("####### finished do function #######")

    asyncio.run(do())

    # Terminal:
    # ```
    # RuntimeWarning: coroutine 'say_after' was never awaited
    #   say_after(1, 'hello')
    # RuntimeWarning: Enable tracemalloc to get the object allocation traceback
    # ####### finished do function #######
    # ```

    ```

    例如上面範例有使用 `asyncio.run()` 來執行 `do()` ，但在其內部的 Coroutine Function `say_after(1, 'hello')` 沒有加 await 關鍵字也沒有其他處理。接著執行程式後會發現在 terminal 上 : 

    - 馬上出現 `finished do function`
    - 並沒有「等待 1 秒之後才列印出 hello 字串然後再等待 1 秒之後才列印出後續的 `finished do function` 字串」這種行為。 
    
    
    看起來 **`say_after(1, 'hello')` 這個函數根本沒有運行 !** 導致不合預期的原因是 Coroutine 沒有成功轉成 Task 給 Event-Loop 排程。
    
    
那要讓 Coroutine 變成 Task 該怎麼做呢？有以下幾種方法 :
- **使用 `await` 關鍵字**
- `asyncio.create_task()`
- `asyncio.gather()`


{{< alert warning >}}
另外注意一下 `await` 需使用在 async def 函數內
{{< /alert >}}

## 總結
上述的範例要變成正確的，可以修正為：

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

---

# FastAPI 使用 await/async

大多數 Web 應用程式都會遇到一些和資料庫互動的 I/O 類型操作，這時 server 會需等待著資料庫回覆，此時在 synchronous 的情況下會阻塞著無法處理其他請求。若再有大量使用者，雖然阻塞時間可能只有微秒，但當所有時間累積起來，總體來看依然是**大量的等待時間**，這就是為什麼在 Web API 中，使用 Asynchronous 還算蠻不錯的優化。

{{< alert success >}}
這種非同步性也是讓 NodeJS 變得開始流行的其中一個原因，同時 FastAPI 也能夠提供的同級效能
{{< /alert >}}

面對這種需要「等待」一下才能給出結果的任務，在 FastAPI 的一個 path operation functions 範例可以這樣寫：

```python
@app.get('/burgers')
async def read_burgers():
    burgers = await get_burgers(2)
    return burgers
```

先專注 `await` 這個關鍵字，它告訴 Python 會**需要等待 `get_burgers` 這個函數的返回，而在此期間可以先去做其他事情**。而為了要讓 `await` 這件事情正確運作， `get_burgers` 函數必須支援異步性，也就 `get_burgers` 是需註明 `async def` 的，範例如下:

```python
async def get_burgers(number: int):
    # Do some asynchronous stuff to create the burgers
    return burgers
```


這時可能會有個疑問，FastAPI 是怎麼調用 path operation function 的 ？ 雖然我們沒看 `asyncio.run()` ，但目前不用太擔心，是可以正常啟動的， 這邊 FastAPI 有幫我們做處理了，它會知道如何做正確的事情！而為什麼 FastAPI 可以這樣啟動呢？ 如果想知道可以參考 [AnyIO](https://anyio.readthedocs.io/en/stable/)，FastAPI 是基於此來做實現的。

---

### 參考資料

- [FastAPI- async and await](https://fastapi.tiangolo.com/async/#very-technical-details)

- [Python asyncio 從不會到上路](https://myapollo.com.tw/blog/begin-to-asyncio/)

- [Python\_非同步協程](https://hackmd.io/@yung1231/Bk3HMIxMw)