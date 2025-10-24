---
title: "Python : Iterable, Iterator, Generator 綜合介紹"

author: Aryido

date: 2025-08-31T15:59:56+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

tags:

comment: false

reward: false
---

<!--BODY-->

> 前段時間發現自己對 Python 的 Iterable、Iterator、Generator 之間的差別並沒有很熟稔，我們都知道這三個都可以使用 for loop 來遍歷，再進一步思考一下所謂的 for-loop 是怎麼實現的。 首先已常見的 list 來說，它本身是一個有 index 的結構，可以一個一個拿出來，蠻符合 for-loop 的使用直覺 ; 但是 dict 也是可以用 for loop 走訪呀，而它並不是順序排列的 ; 甚至 open 的 file 都可以用 for loop 結構來讀取每個 row ，那這些為什麼也能用 for-loop 呢？ 這背後有兩個核心概念： **Iterable(可迭代對象)** 和 **Iterator(迭代器)** 。
>
> 當我們了解 Iterable 和 Iterator 之後，就可以進一步來了解 Generator ，同時再來把這三個做一個比較整理。

<!--more-->

# Iterable

Iterable 中文是「可迭代對象」，比較像是一個 data 保存的容器 container 。而它的定義是要 implement 實現 Iterator protocol 的以下**其中一個**方法 method ：

- `__getitem__()`
- `__iter__()`

首先來說 `__getitem__()`，以下是一個簡單實現範例 ：

```python
class Squares:
    def __init__(self, n):
        self.n = n

    def __getitem__(self, i):
        if i >= self.n:
            raise IndexError
        return i * i

for i in Squares(5):
    print(i)
```

for 迴圈會呼叫 `__getitem__` 直到遇到 StopIteration 或 IndexError 例外才停止。 如果沒有遇到就會無限重複下去。常見的 list、str、tuple 都有實作 `__getitem__` 方法，他們都是 Sequence 類型，本身已有這個 method 了。

{{< alert warning >}}
雖然上面有拋出，但 **for 其實會自行處理 IndexError 的，不需要 try-except**
{{< /alert >}}

{{< alert info >}}
`__getitem__` 是比較舊的 protocol ，現在比較建議實作 `__iter__`
{{< /alert >}}

再來舉例 `__iter__()` 的範例：

```python
class A:
    def __init__(self):
        self.a = 1
        self.b = 2
        self.c = 3

    def __iter__(self):
        # 為什麼 return 那邊，會需要用 `iter()` 將 `[self.a, self.b, self.c]` 包起來呢？
        # 因為 `__iter__()` 方法規定必須回傳 iterator，所以用 iter() 把 list 轉成 iterator
        return iter([self.a, self.b, self.c])

a = A()
for x in a:
    print(x)

```

常見的 dict、file、objects 都有實作 `__iter__` 。還有一個蠻常在 for-loop 用的 `range` ，雖然不符 Python 類別名稱首字母大寫的慣例，但它其實 `range(10)` 會建立物件，它也是一個 Iterable。

# Iterator

前面提到， **`__iter__()` 方法規定必須回傳 iterator** ，那 iterator 是什麼呢？ iterator 也有必須符合的 protocol，就是必須要實作 `__next__` 方法，該方法調用時會從容器中取得下一個資料。如果已經全部取出就會拋出 StopIteration exception。

除了 `__next__` 方法之外， Iterator 最好實作 `__iter__`，讓 **Iterator 也是 iterable**，實作很簡單，只要 **`__iter__` 回傳自己本身就可以了**。

{{< alert warning >}}
官網上是說建議實作 `__iter__` ，但覺得最好是把它實現會比較好，畢竟只要 **`__iter__` 回傳自己本身就可以了**，很簡單的
{{< /alert >}}

雖然上面說會拋出 StopIteration exception，但**for 其實也會自行處理 StopIteration 的**，也可以用 try-except 完成一樣的動作：

```python
r = range(4)

# for-loop
for x in r:
    print(x)

# try-except
it = r.__iter__()
try:
    while True:
        i = it.__next__()
        print(i)
except StopIteration:
    pass
```

另外 python 中以 `__` 開頭並且結尾的方法稱為 special method ，它是 Python 運行時會自動被調用的，基本上平時**不要直接調用它**。例如說 Python for-loop 運行時，就自動會使用 `__iter__` 以及 `__next__` 。 從這邊也可以開始進一步理解一些錯誤，例如：

```python
for i in 5:  # <-- 5 這邊錯了
    print(i)

# TypeError: 'int' object is not iterable
```

這時可以知道說 5 這個整數 int 不是「可迭代的物件(iterable)」。

如果真的要想要使用 `__iter__` 或 `__next__` 這些 special method ，正規的做法是用 build-in 的 `iter()` 與 `next()` ，所以上面範例可以改寫成 :

```
r = range(4)
it2 = iter(r)
try:
    while True:
        i = next(it2)
        print(i)
except StopIteration:
    pass
```

Iterator 表示一個 data streaming object ，可以使用 `__next__` 從 object 內取得下一個新的 data ，由於持續的 `__next__` ，故 Iterator 在跑完一個 for loop 後，就無法重複使用了，這也是 Iterator 和 Iterable 的主要差異:

- Iterable 能被重複迭代
- Iterator 迭代完後就會結束了

有時候我們可能會需要自己實作 Iterable 和 Iterator ， 常見的就是 LinkedList ，如果想要 for-loop 來遍歷的話，大概實現會是 :

```python
class NodeIter:
    def __init__(self, node):
        self.curr_node = node

    def __next__(self):
        if self.curr_node is None:
            raise StopIteration
        node = self.curr_node # 先取目前節點
        self.curr_node = self.curr_node.next # 再前進
        return node

    def __iter__(self):
        return self


class Node:
    def __init__(self, name):
        self.name = name
        self.next = None

    def __iter__(self):
        return NodeIter(self)

node1 = Node("node1")
node2 = Node("node2")
node3 = Node("node3")
node1.next = node2
node2.next = node3

for node in node1:
    print(node.name)

node_iter = iter(node1)

for n in node_iter:
    print(n.name)

```

`Node` 是 iterable ，因為它實現了 `__iter__` 並返回 iterator ; `NodeIter` 是一個 iterator ，因為它實現了 `__next__`，而實現 `__iter__` 主要可以避免 `node_iter = iter(node1)` 這一段會出現錯誤。

[Iteration tools](https://medium.com/@yuhanschen/python-iterator-iterable-iteration-tools-a6240352d045) 套件有蠻多方便的 function 可以利用的，有空可以看一下，例如說：

```python
from itertools import compress, takewhile, dropwhile, cycle, zip_longest

#### compress ####
data = ['a', 'b', 'c', 'd', 'e', 'f']
selectors = [True, False, 1, 0, None] # f 會沒有配對到，這時自動配對為 None
compress_iter = compress(data, selectors)
print(list(compress_iter)) #['a','c']

#### takewhile, dropwhile ####
numbers = [1,3,5,7,11,12]
iter_1 = takewhile(lambda x: x<5, numbers)
print(list(iter_1)) #[1,3]
iter_2 = dropwhile(lambda x: x<5, numbers)
print(list(iter_2)) #[5, 7, 11, 12]

#### cycle ####
cycle_iter = cycle("circle")
for _ in range(10):
    print(next(cycle_iter)) # c, i, r, c, l, e, c, i, r, c

#### zip, zip_longest ####
zip_iter = zip([1,2,3], [10,20], ['a','b','c','d'])
for iter in zip_iter:
    print(iter)
# >> (1, 10, 'a')
# >> (2, 20, 'b')

zip_iter = zip_longest([1,2,3], [10,20], ['a','b','c','d'], fillvalue="NA")
for iter in zip_iter:
    print(iter)
# >> (1, 10, 'a')
# >> (2, 20, 'b')
# >> (3, 'NA', 'c')
# >> ('NA', 'NA', 'd')
```

---

# Generator

知道了 Iterable 和 Iterator ，接下來說明 Generator ，中文翻譯是「 生成器 」。由於 Generator 就是一種特殊的 Iterator，故也可以使用 next 和 for-loop 迭代。主要優勢是可以用 Generator 來迭代一個可能很大的序列，由於在迭代的過程中所產生的值都是動態的，不需要將整個序列儲存在記憶體中。以下給一個簡單的範例 ：

```python
def gen(num):
    while num > 0:
        yield num
        num -= 1

g1 = gen(3)
for n in g1:
    print(n)

g2 = gen(5)
first = next(g2)
for n in g2:
    print(n)

```

再來介紹一下上面一直有看到的 yield 。 Python 在編譯時期發現一個 function 內有 yield 關鍵詞的時候，它就不會把這個 function 當成一個普通的 function 來處理， Python 會給這個 function 打一個標籤，指示這是一個 Generator，調用時**生成器 function 會返回一個生成器 object**。

{{< alert warning >}}
生成器 function 和 生成器 object，我們有時候都叫生成器。
{{< /alert >}}

### Yield

yield 是 python 關鍵字，繼續用上面的範例來說明：

- `g2 = gen(5)` 代表你給 num 賦值了 5 ，因為 function 內有 yield ，故 gen(5) 不會執行函式本體，是產生一個「生成器 object」，再來賦值給 `g2`
- `first = next(g2)` 才會開始執行函式，所以是運行 gen function 並且參數是 `num = 5` ， code 判斷了 `5 > 0`，然後就 **yield 回傳 5 並且 function 就暫停在這裡了**，此時候 `first = 5`
- 再來下面的 for-loop，我們知道每一次相當於都 call 一次 next，在進行一次 gen function 會從 yield 地方出發，故再來 `num - 1 = 4`，繼續 while 然後判斷了 `4 > 0`，然後就又 **yield 但回傳 4 並且 function 就暫停在這裡**...，以此類推到迴圈結束

生成器有時候也能讓 code 變更加簡潔，例如把之前實現的 Linked-List 用 Generator 方式實現：

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.next = None

    def __iter__(self):
        node = self
        while node is not None:
            yield node
            node = node.next


# demo
node1 = Node("node1")
node2 = Node("node2")
node3 = Node("node3")
node1.next = node2
node2.next = node3

for node in node1:
    print(node.name)

```

範例中可以發現 generator 和普通 function 執行流程的不同。 普通 function 是順序執行，遇到 `return` 語句就會返回 ; 而 generator 函會在每次調用 `next()` 的時候執行，遇到 `yield` 語句返回，再次執行時從上次返回的 `yield` 語句處繼續執行。

除了自己實作生成器, 也可以利用生成式 (generator expression) 產生 generator ，簡單注意一下和 List comprehension 的分別：

```python
# 注意是 () 不是 []
g = (i ** 2 for i in [1, 2, 3])
# 透過 collections.abc 式判斷是否為 Iterable、Iterator
import collections.abc
isinstance(g, collections.abc.Iterable) # True
isinstance(g, collections.abc.Iterator) # True
print(hasattr(g, '__next__')) # True
print(hasattr(g, '__iter__')) # True


a = [x**2 for x in range(100)]  # list comprehension
b = (x**2 for x in range(100))  # generator
print(a)   # [0, 1, 2, 3, ..., 100]
print(b)   # <generator object <genexpr> at 0x7fbb6facba50>

```

> 那 generator 有什麼用途呢？

如果想要印出 `0 ~ 100` 的平方時，用 list comprehension 會這樣寫:

```python
powers = [x**2 for x in range(100)]
```

此時 list 都存放在記憶體中，如果今天是一千萬筆資料，會有點消耗記憶體，這時可考慮使用 generator，它在迭代的過程中所產生的值都是動態的，不需要將整個序列儲存在記憶體中。

{{< alert info >}}
另一種角度，可以把 python generator 想成是 producer-consumer 模式中的 producer 。
{{< /alert >}}

# 補充

### 問題 1.

```python
def f1():
    yield(1)     # 使用 yield
    yield(2)
    yield(3)
g = f1()          # 賦值給變數 g
print(next(g))   # 1
print(next(g))   # 2
print(next(g))   # 3
```

> 為什麼上方的程式碼要使用 `g = f1()` 呢？

因為調用 generator 函式會建立一個 generator 物件，多次調用 generator 函式會創建多個「相互獨立」的 generator，下面給一個錯誤範例：

```python
def f2():
    yield(1)
    yield(2)
    yield(3)
print(next(f2()))   # 1
print(next(f2()))   # 1
print(next(f2()))   # 1
```

### 問題 2.

```python
def gen(num):
    while num > 0:
        yield num
        num -= 1

    return
```

> 說明有無 return 的差別

如果 return 後面沒有接回傳值，那這個 return 可有可無 ; 但有回傳值的 return，會發生:

- `return X` 會以 `StopIteration(X)` 結束，X 不會出現在 for 的結果裡
- 但可以被：
  - 手動迭代時抓到 StopIteration.value
  - 用 yield from 接住

```python
def g2(n):
    while n > 0:
        yield n
        n -= 1
    return 123  # 有回傳值


# 只有手動抓 StopIteration 才看得到 return 值
it = g2(0)
try:
    next(it)
except StopIteration as e:
    print(e.value)  # 123

# 或用 yield from 接住
def outer():
    v = yield from g2(0)
    yield v

print(next(outer()))  # 123
```

### 問題 3.

Python 的官方文件中，每個 function 接受什麼類型的參數都會寫出來，譬如常用的 `set(iterable)`, 文件上已經清楚表明它接受 iterable。

```python
def generate_values():
    for x in (1, 1, 2, 2, 3, 3):
        yield x


set([x for x in generate_values()]) # 不要這樣寫

set(generate_values()) # 這樣子寫才對

```

---

### 參考資料

- [帶你搞懂 Python 的 Iterable, Iterator 與 Generator](https://myapollo.com.tw/blog/python-iterable-iterator-generator/)

- [Python\] 關鍵字 yield 和 return 究竟有什麼不同?](https://ithelp.ithome.com.tw/articles/10258195)

- [【python】对迭代器一知半解？看完这个视频就会了。涉及的每个概念，都给你讲清楚！](https://www.youtube.com/watch?v=vfQdRp_PhhQ&list=PLSo-C2L8kdSNAdlCQQ84dlszsiD-gRM5J&index=11)

- [【python】生成器是什么？怎么用？能干啥？一期视频解决你所有疑问！](https://www.youtube.com/watch?v=tWOD43HQIRA&list=PLSo-C2L8kdSNAdlCQQ84dlszsiD-gRM5J&index=12)

- [for 的奧秘–可走訪 (可迭代, iterable) 物件](https://hackmd.io/@meebox/Hy5Dn6cWc)

- [產生器 generator](https://steam.oxxostudio.tw/category/python/basic/generator.html)
