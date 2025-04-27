---
title: "Python : Coroutine 和 async/await"

author: Aryido

date: 2025-03-15T19:46:08+08:00

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

> 之前有介紹了 [Asynchronous](https://aryido.github.io/posts/develop/concurrency-asynchronous/) ，而 Python 的 **Coroutine 是實現 Asynchronous 的一種設計方式**，且 Python 目前已經有非常直觀簡單的語法糖來定義 Asynchronous Code，使得程式寫起來就像普通的 「 Sequential Processing 順序執行 」任務那樣，但同時卻也可以對**目標函數標註做「等待」的動作，並在「等待」期間可以先去做其他任務**，達成非同步的功效，提高程式的並發性，而其重要的關鍵字就是 :
>
> - **`async` : 用來宣告 function 能夠有異步的功能成為 Coroutine function**
> - **`await` : 用來標記 Coroutine 切換暫停和繼續的位置**
>
> 而這兩個關鍵字是在 `Python3.5` 引入且在 `Python3.7` 成為**保留關鍵字**。它們在著名的 FastAPI 框架下的 path operation function 下也經常使用，接下來就簡單介紹一下吧。

<!--more-->

`async` 和 `await` 雖然是 Python 引入的一個新的語法，但它本質上還是一段非常正常的 Python 運行的「**單進程單線程**」的程序，它比較適合處理那些需要等待的任務。

# Coroutine(協程) 和 async 關鍵字

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

透過 `async def` 明確告知 Python 該函式具有非同步執行的能力，代表**定義一個 Coroutine function** ， 且調用 Coroutine Function 時返回的東西為一個 Coroutine Obj 或簡稱 Coroutine，而這也是 **Coroutine Obj 的基本定義，是指 Coroutine Function 返回的物件**。

{{< alert warning >}}
單單只說 Coroutine 這個詞的話，在溝通時會因為使用情景，有時是指 Coroutine Function 而有時是指 Coroutine Obj，算是個統稱或者簡化詞，這邊要注意一下！
{{< /alert >}}

Coroutine 它可以「等待」並在等待狀態時，將執行權**讓給**其他 Coroutine，之後可以**再返回來**繼續執行任務剩下的小部分，且可以多次的進行這樣的行為。

{{< alert success >}}
Python 的 Coroutine 可以對應於 Go 的 Goroutine
{{< /alert >}}

呼叫 `async def` 函數時，不會立即執行函數體，而是傳回一個 Coroutine ，故要運行 Coroutine 是個稍微麻煩的事情，所以在使用 Coroutine Function 上要注意一些事情，以下是一些常見錯誤或解決方式紀錄 :

{{< alert info >}}
以下會開始出現一些陌生詞: Event-Loop 、 Task 等等，但先記得就好，後續會再作解釋
{{< /alert >}}

## 使用 asyncio.run() 來啟動 Coroutine

由於 Coroutine 本身的特性有別於一般函式，需要透過 Event-Loop 排程後才能執行，若直接調用 Coroutine 的話**並不會運行任何 Coroutine 內的 code** ，例如 :

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

- > 由這範例主要的錯誤是**沒有正確進入 async 模式，讓 Event-Loop 控制任務的執行**，而解決方式是使用 `asyncio.run()`來調用，就可以啟動 Coroutine，解決沒有運行的問題。

## 在 async def 的函數內，要調用其他 async def 函數要使用 await

`await` 的功能是告訴 Event-Loop 這是一個標記點，且可以開始執行這個 coroutine 並等待它。沒使用 `await` 就呼叫 Coroutine 且也沒有做其他處理的話，coroutine 就只是被建出來，但沒被安排執行，此時 Event-Loop 沒辦法排程它例如 :

````python
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

````

例如上面範例有使用 `asyncio.run()` 來執行 `do()` ，但在其內部的 Coroutine Function `say_after(1, 'hello')` 沒有使用 `await` 關鍵字也沒有其他處理，接著執行程式後會發現在 terminal 上 :

- 馬上出現 `finished do function`
- 其並沒有「等待 1 秒之後才列印出 hello 字串然後再等待 1 秒之後才列印出後續的 `finished do function` 字串」這種行為。

  看起來 **`say_after(1, 'hello')` 這個函數根本沒有運行 !**

- > 要解決上面範例的錯誤最簡單是**使用 `await` 加在 `say_after(1, 'hello')` 前面**，之後就可以正常運行了

# await 關鍵字 ＆ Awaitable

上面已經給了 `await` 的一個例子，可以知道 `await` 語法會告知 Python 接下來執行的 code ，是一個**可在此處暫停，轉而執行其他工作**的東西。

{{< alert warning >}}
另外還是特別重申一下 `await` 需使用在 async def 函數內
{{< /alert >}}

在使用 `await` 關鍵字或是 `asyncio` 相關函式時，經常可以在文件中看到 awaitable 這個關鍵字，我們看定義後也可以知道 `await` 後面是必須 Coroutine 或是 awaitable obj 的，故以下把 `await` 後面常接的給列出來 :

- Coroutine
- Task (asyncio.Task)
- Future (asyncio.Future)

{{< alert info >}}
Future 和 Task 有什麼不同呢？ 目前簡單來說

- Task 不需要手動 `set_result()`，只要 Coroutine 執行完就會把 return 值當作它的 result 值
- Future 物件可以透過 `set_result()` 或 `set_exception()` 設定結果或例外
  {{< /alert >}}

# 總結

故上述的範例要變成正確的，可以修正為：

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

在此範例使用 Coroutine 需要注意的重點簡單來說有 :

- 會使用 async 用來宣告一個 native Coroutine
- 使用 `asyncio.run()` 來讓進入 async 模式啟動運行
- 簡單使用 `await` 來調用執行其他 Coroutine Function

---

# FastAPI 使用 await/async

大多數 Web 應用程式都會遇到一些和資料庫互動的 I/O 類型操作，這時 server 會需等待著資料庫回覆，此時在 synchronous 的情況下會阻塞著無法處理其他請求。若再有大量使用者，雖然阻塞時間可能只有微秒，但當所有時間累積起來，總體來看依然是**大量的等待時間**，這就是為什麼在 Web API 中，使用 Asynchronous 還算蠻不錯的優化。

{{< alert success >}}
這種非同步性也是讓 NodeJS 變得開始流行的其中一個原因，而 FastAPI 也能夠提供同級的效能
{{< /alert >}}

面對這種需要「等待」一下才能給出結果的任務，在 FastAPI 的一個 path operation functions 範例是這樣寫的：

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

這時可能會有個疑問，FastAPI 是怎麼調用 path operation function 的 ？ 雖然我們沒看到 `asyncio.run()` 出現，但目前不用太擔心，這是可以正常啟動的， 這邊 FastAPI 有幫我們做處理了，它會知道如何做正確的事情！而為什麼 FastAPI 可以這樣啟動呢？ 如果想知道可以參考 [AnyIO](https://anyio.readthedocs.io/en/stable/)，FastAPI 是基於此來做實現的。

---

### 參考資料

- [FastAPI- async and await](https://fastapi.tiangolo.com/async/#very-technical-details)

- [Python asyncio 從不會到上路](https://myapollo.com.tw/blog/begin-to-asyncio/)

- [Python\_非同步協程](https://hackmd.io/@yung1231/Bk3HMIxMw)
