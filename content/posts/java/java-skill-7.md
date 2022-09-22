---
title: JAVA Map方法：compute、computeIfAbsent、put、putIfAbsent

author: Aryido

date: 2022-09-22T21:58:05+08:00

thumbnailImage: "/images/java/java-bean-logo6.jpg"

categories:
- java

tags:
- java
- LeetCode

comment: false

reward: false
---
<!--BODY-->
Map 是 Java 的其中一類interface，不是collection，也不會繼承 Collection interface。
JDK8 的 Map API 有不少便利的預設方法，以下可以介紹一下。
<!--more-->
# 不管存不存在key，都設值：

## put
put : 返回舊值，如果沒有則返回 null
```java
HashMap<String, String> map = new HashMap<>();
map.put("b","B");
String value1 = map.put( "b", "v" ); // value1 = B
String value2 = map.put( "c", "test" ); // value2 = NUll
```

## compute（相當於put,只不過返回的是新值）

compute : 返回新值。當 key 不存在時，執行 value 計算方法，計算 value
```java
HashMap<String, String> map = new HashMap<>();
map.put("b","B");
String value1 = map.compute( "b", (k,v) -> "wwww" ); // value1 = wwww
String value2 = map.compute( "c", (k,v) -> "test" ); // value2 = test
```
### compute 方法適用情況
統計一個 List<String> 中每個元素出現的次數
```java
List<String> list = Arrays.asList("a", "b", "b", "c", "c", "c", "d", "d", "d", "f", "f", "g");

HashMap<String, Integer> map = new HashMap<>();
list.forEach(str -> map.compute(str, (k, v) -> v == null ? 1 : v + 1)); // 此時：新值 = 舊值 + 1
// map = {a=1, b=2, c=3, d=3, f=2, g=1}
```

# 如果 key 不存在，再put：
## putIfAbsent
putIfAbsent : 返回舊值，如果沒有則返回 null
```java
HashMap<String, String> map = new HashMap<>();
map.put("b","B");
String value1 = map.putIfAbsent( "b", "v" ); // value1 = B
String value2 = map.putIfAbsent( "c", "test" ); // value2 = null
```

## computeIfAbsent
computeIfAbsent : 存在時返回存在的值，不存在時返回新值
```java
HashMap<String, String> map = new HashMap<>();
map.put("b","B");
String value1 = map.computeIfAbsent( "b", k -> "v" ); // value1 = B
String value2 = map.computeIfAbsent( "c", k -> "test" ); // value2 = test
```
### computeIfAbsent 方法適用情況
 JDK1.8 的 API 中說到在需要生成一個類似於 Map<K, Collection> 的結構時，computeIfAbsent 很適合這種情況

---
想想 put 用了這麼久，我卻忘記它有返回什麼...， 以上方法筆記一下，希望有機會用到時可以淋漓盡致!


---