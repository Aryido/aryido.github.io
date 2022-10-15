---
title: Binary Search - 2 各式模板

author: Aryido

date: 2022-10-15T17:47:39+08:00

thumbnailImage: /images/algorithm/binary-search-logo.jpg

categories:
- algorithm

tags:
- binary-search

comment: false

reward: false
---
<!--BODY-->
> 前面有介紹了 Binary Search 的通用模板，但通用模板還是有缺點，就是要找的目標須在一個 array 內，這樣才能定義 index ，才能定義 *l = -1* 和 *r = N* 兩個在 array 區間外的 index。 但很多時候題目並不會有一個準確的 array 定義出來，還是需要了解各個模板才能比較好的去解答各式題目的邊界。

<!--more-->

---

# 模板

## 找一個準確值
- 初始 index : *l = 0*, *r = arr.length -1*
- 循環條件: *l <= r*
- 縮減空間: *l = mid + 1* , *r = mid - 1*

## 找一個模糊值
- 初始 index : *l = 0*, *r = arr.length -1*
- 循環條件: *l < r*
- 縮減空間:
  - *l = mid* , *r = mid - 1*
  - *l = mid + 1* , *r = mid*

---

# 例如給定一個 arr={1, 2, 3, 5, 5, 5, 8, 9}，找到第一個 **5** index

ans : 3
```java
// 通用型解法
int[] nums = {1, 2, 3, 5, 5, 5, 8, 9};
int l = -1;
int r = nums.length;
while(l + 1 < r) {
	int mid = l + (r - l) / 2;
	if ( nums[mid] < 5 ) {
		l = mid;
	} else {
		r = mid;
	}
}
// r and l 要返回哪個呢? 要返回 r
// 因為 nums[mid] < 5
// 所以 最後 l 的位置會是 "最後一個小於5" 的 index，r 才會是答案
System.out.println("find first 5 index: " +  r); //find first 5 index: 3
```

```java
// 模糊值解法
// 循環條件: l < r
// 假如 arr = {5, 6} , 經過循環則 l 和 r 都會在 5 上，若 l <= r 會死循環。
//
// 縮減空間: l = mid + 1 , r = mid
// 不能是 r = mid - 1 ，因為可能會排除答案
int[] nums = {1, 2, 3, 5, 5, 5, 8, 9};
int l = 0;
int r = nums.length;
while(l < r) {
	int mid = l + (r - l) / 2;
	if ( nums[mid] < 5 ) {
		l = mid + 1;
	} else {
		r = mid;
	}
}
System.out.println("find first 5 index: " +  r); //find first 5 index: 3
```

---

# 給定一個 arr={1, 2, 3, 5, 5, 5, 8, 9} 找到最後一個 **5** index
ans : 5
```java
// 通用型解法
int[] nums = {1, 2, 3, 5, 5, 5, 8, 9};
int l = -1;
int r = nums.length;
while(l + 1 < r) {
	int mid = l + (r - l) / 2;
		if ( nums[mid] <= 5 ) {
			l = mid;
		} else {
			r = mid;
		}
	}
// r and l 要返回哪個呢? 要返回 l
// 因為 nums[mid] <= 5
// 所以最後 l 的位置會是 "最後一個5" 的 index
System.out.println("find last 5 index: " +  l); //find last 5 index: 5
```

```java
// 模糊值解法
// 循環條件: l < r
// 假如 arr = {5, 6} , 經過循環則 l 和 r 都會在 5 上，若 l <= r 會死循環。
//
// 縮減空間: l = mid , r = mid - 1
// 不能是 l = mid + 1 ，因為可能會排除答案
int[] nums = {1, 2, 3, 5, 5, 5, 8, 9};
int l = -1;
int r = nums.length;
while(l < r) {
	int mid = l + (r - l) / 2;
		if ( nums[mid] <= 5 ) {
			l = mid;
		} else {
			r = mid - 1;
		}
	}
System.out.println("find last 5 index: " +  l); // find last 5 index: 5
```

---