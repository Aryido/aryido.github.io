---
title: "Python : Decorator 與 Type Hints"

author: Aryido

date: 2026-04-27T11:24:07+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

tags:
  - type-hint

comment: false

reward: false
---

<!--BODY-->
> 前面文章有討論過 [Python : Type Hint](https://aryido.github.io/posts/python/type-hint/)了，接下來思考替 Python 的裝飾器 decorator 加上型別標註的問題：型別檢查器經常無法正確推斷被裝飾函式資訊，這時會導致「type hint 無法運作」、「自動補全能力失效」等等問題。
> 
> 更麻煩的是，對於其他地方引用到被裝飾函式的程式部分時，有可能會因此出現紅線警示，但由於這並不是一個**真正的錯誤**，所以時常會看到有人手動加 `# type: ignore` 註解強制 IDE 取消 hint，但這實在不是個好方式...畢竟如果真的有出錯時，也會因為強制取消註解而無法提早發現失誤。
> 要解決此問題會使用到 Python 的進階型別功能如 **ParamSpec** 等等，可讓 decorator 也能擁有完整的型別安全顯示，可以參考 [PEP 612 – Parameter Specification Variables](https://peps.python.org/pep-0612/) 訴求和脈絡。

<!--more-->

# Function Signature

先介紹一下 Python 下的 function signature 定義，簡單來說它是 **Callable 物件的「呼叫規則 metadata」**，例如：每個參數的名稱、參數類別、`*args`、`**kwargs`、是否配置預設值、預設值的具體內容、參數綁定的型別註解（type hints），還有回傳值的資訊等等。
可使用 Python 標準函式庫 `inspect.signature()` 來取得。

{{< alert info >}}
除了 function 之外，實作了 __call__ 的 class 實例，這些都算 Callable 的 
{{< /alert >}}

也注意 Python 本身不支援靠「參數型別」或「參數個數差異」實現的靜態多載，所以 Signature 並不是給編譯器做多載匹配用，而常見實際用途集中在**框架參數解析**，舉例如: 
- API 請求參數綁定
- CLI 參數生成
- Decorator 參數透傳
- 文件自動生成
- type-checker

function signature  功用就像是一份「使用說明書」，看到 Signature 後就可以知道需要傳入什麼、怎麼傳、會拿回什麼。

{{< alert warning >}}
也注意 function signature 並不關心 function 內部做了什麼
{{< /alert >}}

現在知道了 Function 的 Signature 意思，就是指 **Function「怎麼呼叫、會回傳什麼」的描述**，這個詞經常被使用，要注意它的意義。

# Decorator 的 Type Hints 問題

給一個範例來說明：

```python
def deco(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper 

@deco       
def ex1(a: int, b: int) -> int:
    return a + b
fun1 = ex1


def ex2(a: int, b: int) -> int:
    return a + b
fun2 = deco(ex2) 

# 以上定義了 `fun1` 和 `fun2`，這兩個語意是相同的，但錯誤使用時，IDE 報錯卻不一樣：
fun1(1,2,3) # 在參數 3 位置出現紅線，ID直接報錯： Expected 2 positional arguments
fun2(1,2,3) # 無紅線，但執行時會出現 TypeError: ex2() takes 2 positional arguments but 3 were given
```
如上所寫，發現有 @deco 的 `fun1` 有紅線，但是一般函式呼叫的 `fun2` 卻沒有紅線，明明這兩個語意是完全相同的，為什麼呢？ 接下來分析一下！

首先看 fun1 的警示訊息：
```text
Expected 2 positional arguments Pylance (reportCallIssue)
```
知道是 Pylance 作用才會出現警示。

{{< alert warning >}}
Pyright 是型別檢查引擎，Pylance 是把 Pyright 包裝成 VS Code 擴充套件的產品，在 VSCode 裡看到的紅線、型別推論、hover 提示，核心都來自 Pyright
```text
│ Pylance（閉源，VS Code 擴充）   
│  ├ 額外編輯器功能               
│  └ Pyright 核心（開源）         
│      ├ 型別檢查                
│      ├ 型別推論                
│      └ 基本 language server   
```
{{< /alert >}}

接下來再把滑鼠移到 `fun1` 和 `fun2` 上看 hover 提示，就能看到兩者的型別不同：
```python
# fun1 顯示
(variable) def fun1(a: int, b: int) -> int

# fun2 顯示
(variable) def fun2(...) -> int
```
看來是 `fun2` 拿到的 function signature ，是 `deco` 這個函數內定義的廣義 `wrapper` 函數，而不是依照傳入的 `ex2` function 的 signature 資訊。這邊還蠻驚訝的，使用語法糖的 @deco 的 `fun1` 竟然比一般 function call 的表示，給出更符合我們需要的的訊息 ！？ 看來 Pyright 對此是有進一步的優化呢，可以參考[原始碼 GitHub 的 microsoft/pyright 的 getTypeOfDecorator](https://github.com/microsoft/pyright/blob/3afce1fa9c668e051f9f762c5e9d7b43d0494933/packages/pyright-internal/src/analyzer/decorators.ts#L487)，了解為什麼 IDE 顯示錯誤會不一致。簡單的來說：

> 因為 deco function 內回傳的函數 wrapper 參數是 `*args`, `**kwargs`，而 Pyright 在辨識 `fun2` 時，把 wrapper 當成是主要 signature 而不是參考傳入的函數的 signature。所以從 Pyright 的角度來看，`fun2` 變成了一個接受任意參數的函數，因此 IDE 認為正確就沒有紅線了。

# Python 的進階型別功能
接下來思考一下如何設計 decorator，在保持 decorator 靈活性的同時維持型別安全。為了達成目標，先來簡介必要的工具如： TypeVar、Callable、ParamSpec 等等，了解它們如何使用。

##### TypeVar 實現泛型的「型別參數」

泛型（generic）可寫一次 code 就讓其適用於多種型別，而且型別之間的關係不會遺失，例如說：

```python
def first(items: list[int]) -> int:
    return items[0]
```
這個函式只能用在 `list[int]`，如果想讓它也適用 `list[str]`，又想讓型別檢查器知道「傳入 list[str] 就回傳 str」，就需要泛型。

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]

first([1, 2, 3])
first(["a", "b"])
```
值得注意一下 Python 3.12 的新語法，可以不用再引入 TypeVar 了
```python
def first[T](items: list[T]) -> T:
    return items[0]
```

{{< alert warning >}}
可以再花時間參考一下 3.12 之後新型 TypeVar 搭配 Generic 用在類別上的作法，這裡不贅述
{{< /alert >}}


##### Callable 型別註記
在 Python 中由於函式也是個物件，而物件經常被當成參數傳入或者回傳值傳出，故把 function 當成傳入或回傳的東西也很常見的，那這種東西如何定義型別呢 ？ 我們稱為 **Callable**。

```python
from collections.abc import Callable   # 建議，Python 3.9 以上
#from typing import Callable            # 舊寫法，仍可用

# Callable[[參數型別1, 參數型別2, ...], 回傳型別]
Callable[[int, int], int]   # 接收兩個 int，回傳 int
Callable[[str], None]       # 接收一個 str，沒有回傳值
Callable[[], bool]          # 不接收參數，回傳 bool
Callable[..., int]          # 參數隨意，只要回傳 int

```
那其實不只函式，可呼叫的東西都算，例如：
```python
from collections.abc import Callable
f: Callable[[int], str]

f = str                 # ✔ 類別也能呼叫：str(1) → "1"
f = lambda x: str(x)    # ✔ lambda

class Formatter:
    def __call__(self, x: int) -> str:
        return f"<{x}>"

f = Formatter()         # ✔ 有 __call__ 的物件
```

##### ParamSpec
ParamSpec 是一種特殊的型別參數，它代表的不是單一型別，而是一整段參數列表，包含參數名稱、型別、預設值、關鍵字參數等，詳細可參考 Python typing lib 中的 [ParamSpec](https://docs.python.org/zh-tw/3/library/typing.html#typing.ParamSpec)，是 Python 3.10 才加入（PEP 612）的新功能，可補足 TypeVar 不完備的地方。

例如寫裝飾器時，想表達「回傳的函式跟傳進來的一模一樣」，但只用 TypeVar 是做不到的，因為 **TypeVar 只是代表一個單一型別，不是一整段參數列表**，以下更實際說明舉例：
```python
from typing import TypeVar
from collections.abc import Callable

R = TypeVar("R")

def deco(func: Callable[..., R]) -> Callable[..., R]: ...
```
雖然上述 deco 也表達了「回傳的函式跟傳進來的一模一樣」，但只有**回傳型別**保住了而 `Callable[..., R]` 把**輸入參數**的規格全丟了。 `TypeVar` 只能代表一個型別，但假如輸入參數是 `(a: int, b: str = "x", *, flag: bool)` 這樣子，那需要新的方式表達上述結構和訊息，這就是 ParamSpec 出現的原因。

### Decorator 標記正確型別解答
Decorator 很常見的狀況是作為 logger、exception handler、retry ，**本身只會做一些額外事項，接下來就把參數原封不動的送進被裝飾的 function 中**。在這樣的要求下，如何標記正確的型別呢讓 type-hint 順利工作呢？

先回頭看一下之前 deco function 的不足之處： 
```python
def deco(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper 
```
- 對輸入參數 `func` 的資訊並不足夠，`func` 應該只允許 Callable，不允許其他型態，否則使用 `func()`調用時會報錯

- 需要利用 `ParamSpec`來代指參數，而 `Typevar` 就代指回傳值，使得輸入和輸出皆有比較明確的定義

改進之後：
```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")

def deco(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        return func(*args, **kwargs)
    return wrapper
```
{{< alert warning >}}
也注意一下 `def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:`，有比較特殊的型別標示 `P.args` 和 `P.kwargs`
{{< /alert >}}
此時回頭看那些被裝飾的函數，現在就能看到正確的錯誤訊息了。

#  Celery 的 type hint 大問題
由前面的介紹，大概可以了解要替 Python 裝飾器加上 type hint 提醒，是需要花心思的，而這方面 Python 著名的一個任務調度框架 Celery 就做得蠻糟糕的... 以官網提供的範例為例 :
```python
from celery import Celery

app = Celery('tasks', broker='pyamqp://guest@localhost//')

@app.task
def add(x: int, y: int) -> int:
    return x + y

```
然後使用 `.delay()`  後會把 task 丟到背景執行。這是 Celery 很常用的方法之一，它不會在當下執行函式，而是把「要執行哪個 task 和其參數」包成訊息送到 broker，由 worker 取出來執行。

但在實際使用時會發現 `.delay()`，會讓 IDE 出現紅線...
```python
add.delay(4, 4) # Cannot access attribute "delay" for class "FunctionType" Attribute "delay" is unknown   
```
這邊主要是出在 Celery Sourcecode 的問題，[當使用 `@app.task` 來裝飾方法時，會轉成一個 `celery.app.task.Task` 類別的實例物件](https://github.com/celery/celery/blob/1a48583d40d31a7525474f83cf543b4194e113b5/celery/app/base.py#L1645)，但是 `app.task` 的**參數和回傳值都沒有型別標註**，故 Pyright 推論它的回傳型別時，只得到 Unknown，這時 Pyright 會假設裝飾過的函數，和原函式是一樣的，所以被裝飾過的 `add` 還是一個普通的 `(x: int, y: int) -> int` 函數。

但實際執行時，被[裝飾過的 `add` 其實已經是 Task 實例了，這時候 `.delay` 方法是存在的](https://github.com/celery/celery/blob/1a48583d40d31a7525474f83cf543b4194e113b5/celery/app/task.py#L574)，是可以正常的將 Task 包裝成訊息丟進 Broker 中，故實際上程式本身沒錯，紅線出現原因只是 Pyright 拿不到正確的型別資訊。

但這不單單只是看起來不舒服而已，如果新增或修正 code 的有改動到 task function ，例如多了一個 parameter 之類的，而使用者忘記改 caller ，也會比較難發現，畢竟 IDE 那邊看起來一直都是有紅色警示的。

### 解法
對於 Celery 型別標示不清楚問題，雖然有第三方 `celery-types` 可以使用，但此依賴並不是官方維護的，且還有 monkey patch 和 stub 問題需要處理。而使用 celery 目前最緊急需要的，只有參數的檢查和 IDE 補全功能而已，所以自己寫個 decorator 來裝飾也是不錯的解法，既簡短又容易維護。

同樣可以利用上面說的技巧對 Celery Task 加上 type hint ，解答如下：
```python
from celery.result import AsyncResult
from collections.abc import Callable
from typing import Protocol, ParamSpec, TypeVar, cast

T = TypeVar("T", covariant=True)
P = ParamSpec("P")

class CeleryTaskHint(Protocol[P, T]):
    def delay(self, *args: P.args, **kwargs: P.kwargs) -> AsyncResult: ...

    def __call__(self, *args: P.args, **kwargs: P.kwargs) -> T: ...


# `@hint` 代表有 `.delay` 這個方法
# 但如果忘了加 `@app.task` 這裡 type-checker 不會報錯，要執行時才會出 AttributeError
def _check(obj: object) -> None:
    if not hasattr(obj, "delay"):
        raise TypeError("Missing @app.task")


def hint(func: Callable[P, T]) -> CeleryTaskHint[P, T]:
    _check(func)
    return cast(CeleryTaskHint[P, T], func)

```
以上有一些要點要注意：

- ##### 為什麼使用 Protocol
    這裡用 Protocol，是因為要對一個無法修改、沒有型別資訊的第三方物件的某些 function 做修正。
    如果不用 Protocol ，例如說嘗試寫成：
    ```python
    from celery import Task

    def hint(func: Callable[P, T]) -> Task: ...
    ```
    雖然使用 `.delay()` 的紅線會消失，但 Celery 的 Task 的參數，沒有型別標註也不是泛型，所以如果寫錯參數的型別 `add.delay("1", 2)`，也並不會報錯，因為參數完全不檢查的，這樣還是沒完全解決問題。
    
    那把 `Task` 改寫成 `Task[P, T]`這個樣子呢？ 這樣就會踩到和 celery-type 的 stub 一樣的問題了 ; 再來換個方式直接自己定義一般類別的方法：
    ```python
    class CeleryTaskHint(Generic[P, T]):
        def delay(self, *args: P.args, **kwargs: P.kwargs) -> AsyncResult: ...
        def __call__(self, *args: P.args, **kwargs: P.kwargs) -> T: ...
    ```
    但這樣只有**繼承這個類別的物件**才算符合，那當然 Celery 動態產生的 Task 類別吧可能會繼承自己定義的類別，所以在型別上會發生錯誤。
    
    所以最後選擇 Protocol 是最後解答，它是結構型別（structural typing），並不看繼承關係，只看有沒有這些方法，也就是俗稱的 **duck typing**。

- ##### 為什麼 T 要使用 covariant
    Pyright 要求 Protocol 的型別參數必須宣告正確的變異性，不寫會報錯，且由於 `T` 只在回傳值，所以宣告成協變（covariant）是正確的。

    {{< alert warning >}}
    關於協變 covariant、逆變 contravarian、不變 invariant 本篇略過
    {{< /alert >}}

這樣寫還有一些問題，例如說：
- Celery Task 還有很多方法，這裡只寫了 `delay` 而已
- 使用 `@app.task(bind=True)` 時候會發生錯誤

但目前就先把最頭痛的型別標注問題解決了。

---

### 參考資料

- [Python Decorator 與 Type Hints 的完美結合 - TypeVar、ParamSpec、Concatenate、Protocol 深入探討](https://josix.tw/post/python-decorator-type-hints/)

- [Python inspect 實戰：看懂函式簽名、物件結構與開發工具自動化](https://dailypypy.org/learn/python-inspect/)

- [幫 Celery 加上 type hint (ParamSpec)](https://blog.dennymoon.dev/celery-type-hint/)


