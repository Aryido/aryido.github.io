---
title: Java 技巧 - 處理Map<K, Collection<T>>

author: Aryido

date: 2022-09-06T23:28:59+08:00

thumbnailImage: "/images/java/java-bean-logo3.jpg"

categories:
- java

tags:
- map

comment: false

reward: false
---
<!--BODY-->

> 經常有一些業務邏輯要用 Map 來解決，如果再多懂得一些 Map 的方法，是可以寫出精簡的 code 的。這裡展示一些優雅處理 ```Map<K, Collection<T>>``` 類型的方式。

<!--more-->

---

其實蠻常遇到 ```Map<K, List<V>>``` 這樣的集合，當我們想在 key 對應 value 集合裡面添加 element 時:

```java
Map<K, List<V>> map = new HashMap<>();

// 有可能 NullPointException，因為找不到 key 時 map.get(key) 為null.
map.get(key).add(val);

// 初階處理方式
if(!map.containsKey(key)){
    map.put(key, new ArrayList<>());
}else{
    map.get(key).add(val);
}

// 進階寫法
map.computeIfAbsent(key, k -> new ArrayList<>());
map.get(key).add(val);
```

---

接下來是個人比較推薦的寫法。這樣寫也能展現自己對與語言的基礎掌握。

```
// 優雅寫法
map.computeIfAbsent(key, k -> new ArrayList<>()).add(val);



```

{{< alert warning >}}
因為 ArrayLis t是 reference 引用，故 computeIfAbsent 回傳值和 map.get(key) 是指向地址完全相同的 ArrayList。所以直接 add 是會加到 map 對應的集合裡面的
{{< /alert >}}

---

Guava library 的 Multimap 也不錯，但是需要另外安裝
```java
Multimap<K,V> multimap = ArrayListMultimap.create();
multimap.put(key, val);
```
{{< alert info >}}
Guava 是一個 Goolge 開源的 Java 通用library，核心庫有例如：集合、字串處理、I/O 等工具。
{{< /alert >}}

---

# 整理

- 不論是開發還是刷題都很常用到
    ```java
    map.computeIfAbsent(key, k -> new ArrayList<>()).add(val);
    ```
- ```computeIfAbsent``` 使用時有兩種情況；
  - 1. 若 key **不在** map 裡，則會把這個 **key** 和 **remappingFunction 的 output** 添加到 hashMap 裡。 返回值為 **remappingFunction 的 output**
  - 2. 若 key **在** map 裡，則直接返回 key 對應的 value

---