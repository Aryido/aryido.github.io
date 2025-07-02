---
title: "Python : Type Hint"

author: Aryido

date: 2025-04-26T14:39:56+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

tags:

comment: false

reward: false
---

<!--BODY-->

> 我們都知道 Python 是一個動態的語言，代表每一個 variable 是什麼型別是在 runtime 的時候決定的，雖然很靈活可是當 code 量級上去之後因爲類型不正確引發的錯誤也會逐漸增加。但 Python 也是可以做到型別要求的，就是使用 「 Type Hint 」 或者叫 「 Type Annotation 」，中文稱呼蠻多種的例如「型別標註」、「型別提示」等等。 若有寫  Type Hint 的話，比較現代的 IDE 都會有一些自動顯示或補全輔助 :
> {{< image classes="fancybox fig-100" src="/images/python/ide-hint.jpg" >}}
> 
> Python 的 Type Hint 是從`3.5`開始萌芽逐漸引入直到現在，故有一些發展的歷史脈絡和演變，有一些寫法也漸漸更替，故簡單介紹和分析一下。

<!--more-->

首次導入了 Type Hint 可從〈PEP 484 – Type Hints〉開始，該 PEP 提案的共同發起人除了 Python 之父 Guido van Rossum，還有 Mypy 作者，接下來的每一版 Python 都有對 Type Hint 的擴充與增強，以下是一些筆記和感想 :

# Any
如果不對 variable 或返回值進行標註，那它的默認就是 Any。比較要注意的是 **function 返回值的標註**，可能有人會認為 function 沒有返回值的話，就是返回 Any 型別，但這個想法是錯誤的，**在 Python 裡若 function 沒有顯示返回值，則定義上是返回 None**。
```python
from typing import Any

def bar1():
    print("a")

def bar2() -> None:
    print("b")

# 以下寫法反而是有問題的，雖然沒有報錯，但實際語意上不應該被定爲 Any
def bar3() -> Any: 
    print("c")
```

若這 function 可能是 raise 一個 exception 或者會直接退出了程序，它是真的不返回，這個時候可以給返回的型別標註 `NoReTurn` :
```python
from typing import NoReturn

def fatal_error(message: str) -> NoReturn:
    raise RuntimeError(message)
```

{{< alert success >}}
由於大部分是認為<**顯式**表示類型>是要好<於**隱式**不表示的>，故其實 Python 官方是鼓勵 : 
> 無論是覺得這個地方現在還沒有想好怎麼標註，還是覺得這裡就是輸入或返回什麼都行，有標註一個 Any 都比把它留空還更好。

但我個人會覺得可能矯枉過正了，至少在 function 沒有返回值的時候，我會選擇把它留空不寫，而不會特別標註它要返回 None
{{< /alert >}}

---

# Type Hint 簡單範例說明

### Class 內有參數或回傳自己本身 class 的型別

這是一個常見的情況，舉一個簡單的會**錯誤**例子:

```python
class Node:
  def __init__(self, value: int, next: Node): # 不用運行就直接會報錯了: Node" is not defined
      self.value = value
      self.next = next

  def get_next(self) -> Node: # 不用運行就直接會報錯了:  ㄋ Node" is not defined
      return self.next
```

會報錯的原因是 function 在 Class 的裡面定義的時候，這個 Class 本身是還沒有出現的，所以導致 class 找不到。那解決方式有 : 

- ##### 早期版本是可以把這個類型 **<兩邊加上雙引號>**，讓它變成一個 String 
  這樣就解決了循環依賴的問題，這是一個合法的操作，也是被認可的。
```python
class Node:
  def __init__(self, value: int, next: "Node"):
      self.value: int = value
      self.next: "Node" = next

  def get_next(self) -> "Node":
      return self.next
```


- ##### 若是使用 `Python3.7` 以上的版本，可使用 `from __future__ import annotations`

```python
from __future__ import annotations

class Node:
  def __init__(self, value: int, next: Node):
      self.value = value
      self.next = next

  def get_next(self) -> Node:
      return self.next
```

- ##### 若是使用 `Python3.11` 以上的版本，可使用 typing 模組的 Self 型別提示: `from typing import Self`
```
from typing import Self

class A:
    def clone(self) -> Self:
        return A()
```


{{< alert success >}}
目前推薦使用 **typing 模組的 Self 型別提示**，但主要還是可以看團隊的偏好決定就可以了，我本身沒有太多堅持
{{< /alert >}}

### 標註容器裡面 item 的型別

這要注意一下，標註 **python容器** 裡面 item 的型別方式，有不同寫法，使用<列表>舉例的話：

- ##### 如果是 `Python3.8` 或更舊版本的話，需要從 typing 這個 lib 裡引用`大寫 List`作為 Type Hint 的類型
  
```python
from typing import List

def duplicate_first_element(values: List[int]) -> List[int]: # 是使用大寫 List
  if not values:
      return []
  first = values[0]
  return [first] * len(values)
```

- ##### `小寫 list` 後面直接加方括號是 `python3.9` 才支持的 :
  

```python
def duplicate_first_element(values: list[int]) -> list[int]: # 內建 list 就好，不用特別 import
  if not values:
      return []
  first = values[0]
  return [first] * len(values)
```

如果是想寫一個對現在版本都正在支持的應用，就要用 `typing List` 因為它在 `3.9` 版本之前也能用 ; 如果程式只需要支持 `3.9` 以後的版本，就用 `Build-in list`。

{{< alert info >}}
同理 <字典 dict> 型別:
- `python3.9` 以上(含)：直接使用 dict
- `python3.9` 以下：需引用 `typing`，使用`from typing import Dict`
{{< /alert >}}


{{< alert success >}}
那目前是覺得**盡量使用 Build-in list 這個小寫寫法**。因為 Type List、Dict 從 `Python3.9`版本開始就 Deprecated 了，而在 `Python3.9` 版本發布那年的 5 年後 Type List 也會被刪除
{{< /alert >}}

### Literal
Literal 是 `Python 3.8` 才加進來 `typing` 模組裡的，是用來限制變數或參數只能是某幾個特定的值。比如說有一個 Person 裡面有一個 Gender，而 Gender 想用 String 的方式存就好，可能不需要開一個 `Enum`，這個時候可以通過 Literal 規定 Gender 傳只能是 Male 或者是 Female 字串，其他的都不可以 :
```python
from typing import Literal

class Person:
    def __init__(self, name: str, gender:  Literal["Male", "Female"]):
        self.name = name
        self.gender = gender

# 若寫成 man ，則不用運行就直接會報錯了
# Argument of type "Literal['man']" cannot be assigned to parameter "gender" of type "Literal['Male', 'Female']"
Person("tom", "man")

gender: Literal["Male", "Female"] = "Male"
Person("tom", gender)

```
由於很多人會直接用 String 來表示有限的狀態，故有時候可能會出現一些 typo 打錯字的情形，這個 Literal 就可以很幫變幫忙檢查。

再來 `Literal["Male", "Female"]` 其實也蠻推薦**不要寫死當作一個型別**，可以移出來聲明一個新的型別變數，會有一些優點:
- 例如要增加不同 Gender 類型時，只要在聲明上共同修正就好

- 給這個型別取一個名稱，也能增加一些可讀性 :
```python
from typing import Literal

Gender = Literal["Male", "Female"]

class Person:
    def __init__(self, name: str, gender: Gender):
        self.name = name
        self.gender = gender

gender: Gender = "Male"
Person("tom", gender)
```

{{< alert success >}}
`Literal` 和 `Enum` 哪個比較好呢，我偏向看實際應用場景。 `Literal` 寫法很輕量方便 ，而 `Enum` 可以更詳細定義一些方法例如：
```python
from enum import Enum

class Status(Enum):
    SUCCESS = "success"
    ERROR = "error"
    PENDING = "pending"

    def with_prefix(self, prefix: str) -> str:
        return f"{prefix}{self.value}"
```
- 只是想要簡單型別限制 → 用 Literal。
- 想要定義多功能、高可擴充性 → 用 Enum。
{{< /alert >}}

### NewType

說明 NewType 之前，先舉個例子來思考，像這種**把型別直接換一個名字的方式**，有什麼壞處呢 ？ 就是編譯器會認為這兩個型別是等價的，例如:
```python
UserId = int
AttackPoint = int

class Player:
    uid: UserId
    attack: AttackPoint

    def __init__(self, uid: UserId, attack: AttackPoint):
        self.uid = uid
        self.attack = attack

    def update_attack(self, atk: AttackPoint):
        self.uid = atk  # 這邊有問題，但編譯器沒報錯


player1 = Player(100101, 1)
new_attack = 10
player1.update_attack(new_attack)

# 以下都正常運行，但是邏輯上發生大錯誤
print(f'player1 id: {player1.uid}') # id 100101 >>  10 ; 意外更新成 id，超級大錯誤
print(f'player1 attack: {player1.attack}') # attack 還是 1 ; 攻擊力沒變

```
我們有了 UserId 和 AttackPoint 這兩個新型別名稱，但由於只是給 `int` 重新起了兩個名字，編譯器就會認為 Attack 和 UID 都是 `int` ，所以編譯器沒有報錯，但明顯我們知道 Attack 和 UID 不應該是一樣的，在  `self.uid = atk` 是人為疏失寫錯。為了解決這個問題， Python 引入了 NewType ˋ這個功能，它會產生一個**獨立的新型別** :
```python
from typing import NewType

UserId = NewType('UserId', int)
AttackPoint = NewType('AttackPoint', int)

class Player:
    uid: UserId
    attack: AttackPoint

    def __init__(self, uid: UserId, attack: AttackPoint):
        self.uid = uid
        self.attack = attack

    def update_attack(self, atk: AttackPoint) -> None:
        self.uid = atk  # 不用運行就直接會報錯了，會出現 Cannot assign to attribute "uid" for class "Player*" "AttackPoint" is not assignable to "UserId"player1 = Player(100101, 1)

```
這樣就不會發生 `self.uid = atk` 這種事情了，因為編譯器會直接幫忙指出錯誤。當然這種用法也會產生一個新問題，就是沒有辦法在用 `int` 來 assign 值了，因為編譯器認為 UserID 跟 AttackPoint 都不是 `int` 而是不同類型:
```python
# 不用運行就直接會報錯了，不能單純用 int 來賦值了
# player1 = Player(100101, 1) 

player1 = Player(UserId(100101), AttackPoint(1))
```

{{< alert success >}}
從另一個角度， NewType 寫法讓 python 變得更不 python 了，我認為這樣實在有點太多了，但用還是不用、怎麼用、用多少還是可以聽一下團隊的意見
{{< /alert >}}


### Union 和 bitwise or operator |

bitwise or operator 在 python 符號就是指 `|` ，英文名稱可以記憶一下。這個符號的出現其實不難想像成因，例如一個常見的情況，例如當我們輸入 argument 的時候，有兩個或兩個以上可能的類型，即是要表達「某幾種型別內的任一種都可以」，則 :

- ##### `Python3.5` 開始可以使用 Typing 模組中 Union :

  例如 `Union[str, int]` 表示字串或整數其中一種 :

``` python
from typing import Union

def stringify(value: Union[str, int]) -> str:
    return str(value)
```

- ##### 在 `Python3.10` 版本之後，可以簡寫成 `|` :

``` python
def stringify(value: str | int) -> str:
    return str(value)
```

### 用抽象類別表示 Type Hint ，例如 Sequence ＆ Iterable

在實際上應用上， list 和 Tuple 用法上是幾乎一樣的，故在當作 argument 的時候把 list 跟 Tuple 傳哪個都應該一樣，所以說一般情況下不會寫死 list 當 argument 的型別 ，而是用它的一個更抽象的型別叫做 `Sequence` :

```python
from typing import Sequence

def my_sum(values: Sequence[int]) -> int:
    return sum(values)
```

{{< alert info >}}
那 Sequence 對 list、Tuple 可以用，它也對 Range ，甚至對 Byte 也可以用，這個是相對更普遍的用法
{{< /alert >}}

還有另一種情況，是要 function 的某個 argument 可以接受任何 Iterable 物件，那會想到 Iterable 有 list、tuple、set， 所以 Typing 裡寫法會像是 `Union[set, list, tuple]` 這樣，但這有些問題，例如
- **Iterable 物件**有窮舉了嗎？

    {{< alert warning >}}
其實在上述中至少還少了 `dict` 跟 `str`， 雖然這兩個型別也是可遍歷，但是沒有放入 `Union[set, list, tuple]` 中，所以無法成為 function 的輸入參數
{{< /alert >}}
  
- 寫成 `Union[set, list, tuple, dict, str]` 也太長了

- 如果是自定義了新的 Iterable 物件，也要自己記得手動加進去 Union 裡面相當麻煩

所以其實蠻適合用**抽象類別**來定義型別的，只要某個類別符合該抽象類別定出的規則，該類別就可以是該抽象類別的一員，例如說：
- Iterable 是要實作 `__iter__()`
- Sequence 是要實作 `__getitem__(index)` 和  `__len__()`

| 項目             | Iterable                         | Sequence                        |
|:-----------------|:----------------------------------|:--------------------------------|
| 基本定義         | 能「被迭代」的物件                 | 有「順序」且能「索引」的物件       |
| 必要支援方法     | `__iter__()`                      | `__getitem__(index)` + `__len__()` |
| 是否有順序       | 不一定有順序                       | 一定有順序（可以用索引取資料）    |
| 能否 for loop    | 可以                               | 可以                             |
| 能否隨機存取     | 不行                               | 可以（用 `obj[0]`, `obj[1]` 這樣取） |
| 常見例子         | `list`、`set`、`dict`、`generator` | `list`、`tuple`、`str`            |


### Optional 

由於一個參數有可能是一個類型或者 None ，這個 Pattern 過於常見了，所以特別給了 `Optional` 的型別形式 ，這會比 Union None 更清晰一些，兩個是完全等價的 : 

``` python
from typing import Union, Optional

def greet(name: Union[str, None]) -> str:
    if name is None:
        return "Hello, guest!"
    return f"Hello, {name}!"

# 如果是 `Union` vs `Optional`的話，比較建議使用 Optional
def greet(name: Optional[str]) -> str:
    if name is None:
        return "Hello, guest!"
    return f"Hello, {name}!"
```

由於 `Python3.10` 之後， 支持 `|`，所以其實也可以 :
``` python

def greet(name: str | None) -> str:
    if name is None:
        return "Hello, guest!"
    return f"Hello, {name}!"
```

{{< alert success >}}
結論上由於 Python 官方 [PEP 604](https://peps.python.org/pep-0604/#proposal)是建議都用 `|` 表示，故用 `|` 可能是比較好的選擇。但在我自己的感覺上，我是比較偏好 `Optional` 的
{{< /alert >}}

### Callable

Callable 的定義是有實作 `__call__`的東西，可以被當作 function 一樣調用，在 Type Hint 上 Callable 裡面的方括號要放兩個內容 : 
- 第一個是 argument 的 type ，也用方括號擴起來，代表可以多個
- 第二個是返回的 type
``` python
from typing import Callable

def executor(func: Callable[[], None]) -> None:
    func()
```
故這裡範例意思是：
- executor 這個函數接受一個 func 參數
- 而 func 是一個不接受任何參數、也不回傳任何東西的函數

比如說我們寫一個非常簡單的 Decorator : 
```python
def my_dec(func):
    def wrapper(*args, **kwargs):
        print("start")
        ret = func(*args, **kwargs)
        print("end")
        return ret
    return wrapper

my_dec(1)
```
`my_dec(1)` 在語法上它是被允許的，但是顯然這是一個錯誤的調用，我們明顯希望這個 argument func 是 Callable，所以可以修正為 : 
```python
from typing import Callable

def my_dec(func: Callable):
    def wrapper(*args, **kwargs):
        print("start")
        ret = func(*args, **kwargs)
        print("end")
        return ret
    return wrapper

# Argument of type "Literal[1]" cannot be assigned to parameter "func" of type "(...) -> Unknown" in function "my_dec"
# my_dec(1) 不用運行就直接會報錯了
```
接下來再進階一些：
```python
from typing import Callable

def my_dec(func: Callable[[int, int], int]):
    def wrapper(a: int, b: int) -> int:
        print(f"args = {a}, {b}")
        ret = func(a, b)
        print(f"result = {ret}")
        return ret
    return wrapper

@my_dec
def add(a: int, b: int) -> int:
    return a + b

# 不用運行就報錯了
# Argument of type "(a: int) -> int" cannot be assigned to parameter "func" of type "(int, int) -> int" in function "my_dec"
@my_dec
def absolute(a: int) -> int:
    return abs(a)
```

---

# Python Type Hint 支援版本總整理

| Python 版本 | 主要 type hint 新增或改變內容 |
|:------------|:-----------------------------|
| 3.5          | 第一次正式加入 `typing` 模組（`List`、`Dict`、`Optional`、`Union`、`Callable`） |
| 3.6          | 支援 `variable annotations`（變數型別註解，PEP 526） |
| 3.7          | `from __future__ import annotations` 可以延遲型別解析（PEP 563） |
| 3.8          | 加入 `Literal`、`Final`、`TypedDict`（需要 `typing_extensions` 支援） |
| 3.9          | 小寫 built-in 泛型支援（可以寫 `list[int]`、`dict[str, int]`，不用再 `List[int]`） |
| 3.10         | 型別聯集新語法 **\|**（例如可以寫 int \| str） |

---

# 心得

Python 在推廣 Type Hint 的過程中有幾個可以深思的做法 :

> - type-hint 是幾乎沒有 runtime 懲罰的，也就是說你並不會因為寫了 Type Hint 導致你的 code 運行變慢，這是一個非常好的工程理念：**引入一些新的 feature 不會產生其他問題**

> - gradual typing 漸進式類型標註: 這裡體現為大家可以嘗試使用  Type Hint 但並不強制要求，不是在有和沒有之間做選擇，而是可以漸漸的在自己的 code 裡逐漸加入，可以讓更多的人在無傷的情況下去嘗試增加這樣的 feature，感受到它的好處

---

### 參考資料

- [typing — Support for type hints](https://docs.python.org/3/library/typing.html)

- [python】Type Hint 入门与初探，好好的 python 写什么类型标注？](https://www.youtube.com/watch?v=HYE85bqNoGw)

- [python】Type Hint 的进阶知识，这下总该有你没学过的内容了吧？](https://www.youtube.com/watch?v=6rgBwA7TRfE)

- [Python 微進階 Day28 - type hint(型別提示)](https://ithelp.ithome.com.tw/m/articles/10338998)

- [How should I use the Optional type hint?](https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint)

- [Python Type Hints 教學：我犯過的 3 個菜鳥錯誤](https://haosquare.com/python-type-hints-3-beginner-mistakes/)

- [《強健的 Python》筆記：如何有效導入 Type Hints](https://blog.kyomind.tw/robust-python-01/)
