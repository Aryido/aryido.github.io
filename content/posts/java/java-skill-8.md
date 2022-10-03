---
title: JAVA Map方法： merge & compute 比對

author: Aryido

date: 2022-10-01T16:36:17+08:00

thumbnailImage: "/images/java/java-bean-logo1.jpg"

categories:
- java

tags:
- map

comment: false

reward: false
---
<!--BODY-->
> Java 8 因為引入了 lambda 這樣的 functional programming，故Map 系列有了許多新增方法，感覺還是很好用的，簡單做一些相關介紹 ...

<!--more-->

---

# compute
compute 方法可以指定 key ，用指定的 Lambda 運算，來決定 key 的對應 value ，這是它之所以命名為 compute 的原因。

compute 是返回新的值。 更詳細的說:
- key 有對應的 value 時
   1. Lambda 的傳回值若**不為** null，以新值取代舊值
   2. Lambda 的傳回值若**為** null，將 key -  value 移除

{{< alert warning >}}
注意 computeIfAbsent 和 compute 兩個 lambda function 參數不一樣!
```java
map.computeIfAbsent( "b", k -> "v" );

map.compute( "b", (k,v) -> "wwww" );
```

{{< /alert >}}

---

# merge
merge 方法的 Lambda 比 compute **多了一個參數**，可以指定 value。

返回值:
1. 如果 key 對應的 value 不存在，則返回該 value 值
2. 如果存在，則返回 Lambda 的值。

```java
HashMap<String, Integer> map = new HashMap<>();

map.put("Shoes", 200);
map.put("Bag", 300);
map.put("Pant", 150);
// HashMap: {Pant=150, Bag=300, Shoes=200}

int value1 = map.merge("Shirt", 100, (oldValue, newValue) -> oldValue + newValue);
// value1 = 100
// map =  {Pant=150, Shirt=100, Bag=300, Shoes=200}
// reduce: int returnedValue = map.merge("Shirt", 100, Integer::sum );

int value2 = map.merge("Shoes", 12, (oldValue, v) -> oldValue - v);
// value2 = 188
// map = {Pant=150, Shirt=100, Bag=300, Shoes=188}
```

{{< alert info >}}
### merge 方法適用情況
比如分組求和這類的操作，雖然 stream 中有相關 groupingBy() 方法，但如果你想在循環中做一些其他操作的時候，merge() 還是一個挺不錯的選擇。

例如有一個學生列表，包含id、科目分數，要求求得每個學生的總成績。

```java
int[] o11 = new int[]{1, 70}; //id1 學生數學 70
int[] o12 = new int[]{1, 80}; //id1 學生英文 80
int[] o13 = new int[]{1, 65}; //id1 學生國文 65

int[] o21 = new int[]{2, 68};
int[] o22 = new int[]{2, 70};
int[] o23 = new int[]{2, 90};

int[] o31 = new int[]{3, 80};
int[] o32 = new int[]{3, 85};
int[] o33 = new int[]{3, 70};

List<int[]> list = new ArrayList<>();
list.add(o11);
list.add(o12);
list.add(o13);
list.add(o21);
list.add(o22);
list.add(o23);
list.add(o31);
list.add(o32);
list.add(o33);

Map<Integer, Integer> studentScoreMap = new HashMap<>();
list.forEach(array -> studentScoreMap.merge(
    array[0],
    array[1],
    (oldV, newV) -> oldV + newV)
);
// studentScoreMap = {1=215, 2=228, 3=235}
// list.forEach(array -> studentScoreMap.merge(array[0], array[1], Integer::sum ));


// 用 computeIfAbsent 比較多行
Map<Integer, Integer> studentScoreMap2 = new HashMap<>();
list.forEach(array -> {
    Integer integer = studentScoreMap2.computeIfAbsent( array[0], k -> 0 );
    int newValue = integer + array[1];
    studentScoreMap2.put(array[0], newValue);
});
// studentScoreMap2 = {1=215, 2=228, 3=235}

```

{{< /alert >}}

{{< alert warning >}}
merge 參數是 **舊value 和 新value**

compute 參數是 **鍵值對**
```java
map.merge("Shirt", 100, (oldValue, newValue) -> oldValue + newValue);

map.compute( "b", (k,v) -> "wwww" );
```

{{< /alert >}}

---