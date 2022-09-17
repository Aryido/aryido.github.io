---
title: "Java Skill"

author: Aryido

date: 2022-09-06T23:28:59+08:00

thumbnailImage: "/images/java/java-bean-logo3.jpg"

categories:
- java

tags:
- java
- map

comment: false

reward: false
---
<!--BODY-->

> 優雅處理 ```Map<K, Collection<T>>``` 類型的方式。

<!--more-->

在寫code時，其實蠻常遇到 Map<K, List<V>> 這樣的集合。 當我們想在對應key的集合裡面添加element時:

```java
Map<K, List<V>> map = new HashMap<>();

//有可能 NullPointException，因為找不到 key 時 map.get(key) 為null.
map.get(key).add(val);

//初階處理方式
if(!map.containsKey(key)){
    map.put(key, new ArrayList<>());
}else{
    map.get(key).add(val);
}

//進階寫法
map.computeIfAbsent(key, k -> new ArrayList<>());
map.get(key).add(val);

//優雅寫法
//因為ArrayList是reference引用，故 computeIfAbsent 回傳值和 map.get(key) 是指地址完全相同的 ArrayList
map.computeIfAbsent(key, k -> new ArrayList<>()).add(val);

//Guava library 的 Multimap 也不錯，但是刷題時不支援
Multimap<K,V> multimap = ArrayListMultimap.create();
multimap.put(key, val);

```

---