---
title: "Python: Dictionary"

author: Aryido

date: 2025-04-27T22:56:56+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

tags:

comment: false

reward: false

---

<!--BODY-->
> Dictionary 是 Python 用來儲存 key → value 的對應關係，其他語言類似的資料結構為 :
`HashMap(Java)` ; `Object/Map(JavaScrip)` ; `Map(Go)` 等等，以下筆記一些 Dictionary 小觀念和使用時機。注意 Dictionary 底層是 hash table，所以其 **key 必須是不可變的 (Immutable)**，這要特別注意 ; 另外[自 Python 3.7+ 起，Dictionary 有保證會「保留插入順序」](https://blog.gslin.org/archives/2020/02/17/9423/python-3-7-%E4%BF%9D%E8%AD%89-dict-%E5%85%A7%E5%AE%B9%E7%9A%84%E9%A0%86%E5%BA%8F/)，但還是盡量不要依賴這個行為會比較好。

<!--more-->

---

# dict 的初始化
```python
# 空字典
d1 = {}
d2 = dict()

# 使用大括號初始化 dict，但要注意 key 必須加上引號 "",否則會被當成變數
key1 = "qq"
value1 = 7
person = {
    "name": "Alice",
    key1: value1,
    "age": 25, 
}

# 使用關鍵字參數初始化 dict， 其 key 不用引號但必須符合 Python 變數命名規則（不能有空格或數字開頭）
person = dict(name="Alice", age=25)

# Dict Comprehension
squares = {x: x**2 for x in range(1, 3)} # {1: 1, 2: 4}
```

### defaultdict

defaultdict 在 Python 內建的模組 collections 內，其的優點是當 defaultdict 拿取一個不存在的 key 時，**不會噴 KeyError**，而是會根據預設的**工廠模式**產生預設值。再詳細舉例，原生 dict 中如果想幫一個不存在的鍵累加數值，得這樣寫：
```python
counts = {}
if 'key1' not in counts:
    counts['key1'] = 0
counts['key1'] += 1

```
使用 defaultdict 後，可簡化為：
```python
from collections import defaultdict

counts = defaultdict(int) # 預設值是 int()，即 0
counts['key1'] += 1      # 不存在會自動建立 0 再 +1

# 其他範例
d = defaultdict(list)
d = defaultdict(set)
d = defaultdict(lambda: "N/A")
```
注意一下所謂的**工廠模式**，其核心原理是利用 `__missing__` 這個 magic function，如果 key 不存在，它會呼叫這個方法。

### dict 可以使用 tuple 作 multi-key: d[k1, k2, k3]
這個技巧可以注意一下，因為在 Python 中，`d[k1, k2, k3]` 語法之所以可行，是因為 Python 會自動將逗號分隔的值，視為一個 tuple:
```python
locations = {}
# 故寫 `d[k1, k2]` 時等價於 `d[(k1, k2)]`，且
locations[25.03, 121.56, "2026-03-13"] = "Taipei 101"
locations[(25.03, 121.56, "2026-03-13")] = "Taipei 101"
```

{{< alert warning >}}
因為 tuple 是不可變的(Immutable)，故它具備雜湊值（Hashable），所以可以合法地作為 dict 的 key。那例如說一般的 list 就不可以作為 dict 的 key : 
```python
arr_list = [1,2]
d= {}
d[arr_list] = 123 # TypeError: unhashable type: 'list'
```
{{< /alert >}}

dict() 也可以將有「兩個值的 list 或 tuple」轉換成字典，轉換時會將第一個值當作 key ，第二個當成 value。但注意只能是兩個，多個會失敗。
```python
a = [['x','100'],['y','200'],['z','300']]
b = dict(a) # {'x': '100', 'y': '200', 'z': '300'}


a = [['x','100', '3'],['y','200', '4'],['z','300', '5']]
b = dict(a) # ValueError: dictionary update sequence element #0 has length 3; 2 is required

```

# dict 的新增和修改 element
```python
user = {
    "id": 123,
    "name": "Henry",
    "is_active": True
}

# 新增或修改 (如果 key 存在就覆蓋，不存在就新增)
user["role"] = "admin"
user.update(role="admin")

# update好處是可以一次處理多個鍵值對
user.update({"role": "admin", "status": "active"})
user.update(role="admin", status="active")


# 使用聯集運算子 | (Python 3.9+)
# 會回傳一個合併後的新字典，不會改動原本的 user
new_user = user | {"key1": "key1-1"}
# 如果想直接更新原字典，可以使用 |=
user |= {"key2": "key2-2"}

# 字典解構 (Dictionary Unpacking)
person = dict(name="Alice", age=25)
person = {**person, "role": "admin"}
```

有時候可能只想在 key 不存在時才新增，這時可以使用 `.setdefault()`：
```python
# 如果 "role" 已存在，什麼都不會做；如果不存在，則新增 role 為 default
user.setdefault("role", "default")
```

# dict 的讀取 element
利用 key 取出相對應的 value，可以使用 `[]`(square brackets) 或者 `dict.get(key)`。這兩種方式主要差異，在於 key 不存在 dict 時所引發的行為，中括號會拋出 KeyError ; 而`dict.get(key)`則會回傳 None ：

```python
user = {
    "id": 123,
    "name": "Henry",
    "is_active": True
}

print(user.get("phone")) # None
print(user["phone"]) # KeyError: 'phone'
```

也注意千萬不要用 `if d.get("key"):` 來判斷存在，因為如果 value 是 0, false 等等，就會被判定為 False。 要使用 `if "key" in d:`


# dict 刪除 element
```python
user = {
    "name": "Alice",
    "age": 25, 
}

# pop 會刪除指定的 Key 並取出該值
age = user.pop("age") # 25
# pop 也可以設定預設值
user.pop("age", None) 
# pop 不存在的 key 時，是會報錯的
age = user.pop("key1 ") # KeyError


# del 如果 Key 不存在，會拋出錯誤
del user["phone"] # KeyError
```

---

### 參考資料

- [Day11-Dictionary-操作](https://ithelp.ithome.com.tw/articles/10191479)

- [Leetcode 刷題解答與 Python 3 小技巧分享](https://www.ptt.cc/bbs/Soft_Job/M.1627032495.A.65E.html)

- [字典 dictionary](https://steam.oxxostudio.tw/category/python/basic/dictionary.html)


