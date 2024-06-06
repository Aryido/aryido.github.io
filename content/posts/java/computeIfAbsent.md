---
title: Java 技巧 - computeIfAbsent() 用法詳解

author: Aryido

date: 2022-09-17T12:44:22+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- language
- java

tags:
- map

comment: false

reward: false
---
<!--BODY-->
> 使用 HashMap 的方法 :
> ```jav
> computeIfAbsent(K key, Function remappingFunction)
>```
> 其中 *remappingFunction* 是一個 **Functional interface**
> - input 為 map 的 **key**
> - output 會成為 map 的 **value**
>
> HashMap 的 computeIfAbsent 方法，在 key 不存在時，會做 remappingFunction 的操作，所以再也不會因為漏寫 ```if x == null``` 而出現空指針的 bug 了。
<!--more-->

---
computeIfAbsent 是 java.util.Map 的默認方法，已在 Java 8 中引入。 computeIfAbsent 方法在與 **key 對應的value 不可用或為空時起作用**，在這種情況下，computeIfAbsent 方法會由給定 remappingFunction 計算的新值。

整理如下，使用時有兩種情況:
- {{< alert info >}}
若 key **不在** map 裡，則會把這個 **key** 和 **remappingFunction 的 output** 添加到 hashMap 裡。 返回值為 **remappingFunction 的 output**
{{< /alert >}}
- {{< alert info >}}
若 key **在** map 裡，則不會重新計算 value。 返回值為 現在 map 的 key 對應的 value
{{< /alert >}}

---

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("Shoes", 200);
map.put("Bag", 300);
map.put("Pant", 150);
// {Pant=150, Bag=300, Shoes=200}

// Shirt 不在 map 裡， 故會把 Shirt 和其對應 value 加到 map 裡
int shirtPrice = map.computeIfAbsent("Shirt", key -> 280); // shirtPrice = 280
// 返回值為 remappingFunction 的 output

// 更新後的 map = {Pant=150, Shirt=280, Bag=300, Shoes=200}
```
```java
HashMap<String, Integer> map = new HashMap<>();
map.put("Shoes", 200);
map.put("Bag", 300);
map.put("Pant", 150);
// {Pant=150, Bag=300, Shoes=200}

// Shoes 在 map 裡， 故甚麼都不會做。
int ShoesPrice = map.computeIfAbsent("Shoes", key -> 280); // ShoesPrice = 200
// 返回值為 key 對應的 value

```

{{< alert warning >}}
**computeIfAbsent** 務必熟悉此方法，可幫助簡化code。
{{< /alert >}}

---