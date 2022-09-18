---
title: computeIfAbsent() 用法詳解

author: Aryido

date: 2022-09-17T12:44:22+08:00

thumbnailImage: "/images/java/java-bean-logo4.jpg"

categories:
- java

tags:
- java
- map

comment: false

reward: false
---
<!--BODY-->
> 使用 HashMap 的方法 **computeIfAbsent(K key, Function remappingFunction)** ，其中 *remappingFunction* 是一個 input 為 **key**，output 為 **value** 的一個 **Functional interface**。
>
>使用時有兩種情況；
> - 1. 若 key **不在** map 裡，則會把這個 **key** 和 **remappingFunction 的 output** 添加到 hashMap 裡。 返回值為 **remappingFunction 的 output**
> - 2. 若 key **在** map 裡，則不會重新計算 value。 返回值為 key 對應的 value

<!--more-->

**computeIfAbsent** 務必熟悉此方法，可幫助簡化code。

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

---