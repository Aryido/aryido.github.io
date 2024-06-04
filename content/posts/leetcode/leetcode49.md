---
title: "49. Group Anagrams"

author: Aryido

date: 2023-06-04T13:53:53+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- map

comment: false

reward: false
---
<!--BODY-->
> 所謂的兩字串互為 Anagrams ，意思就是兩個字串中，**字母出現的次數都一樣，只是位置不同**，比如題目說的 ate 、 eat 、 tea 它們就都互為 Anagrams 。如何判斷兩字串是否互為 Anagrams 是關鍵解題點，這題雖然歸為 Medium ，但是偏向 easy 的，需要注意的點是:
> - 熟悉一些 java 內建常用的字串處理 function ，寫起來會比較簡潔。
> - 想到使用 map 結構來儲存分組資料。
<!--more-->

---

# 思路

可以發現如果把**兩個 string 個別分割成 char array 後，並用同一種方式，個別排序 array 內元素，若會得到相同的結果，則互為 Anagrams** 。由於同一組 Anagrams 詞重新排序後都會得到相同的結果，以此作為 key，將所有相同的 Anagrams 詞都保存到一 list 中，就可一建立 map ，該 map 就是把 Anagrams 組建立完成的解答了。

---

# 解答
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        List<List<String>> ans = new ArrayList<>();
		Map<String, List<String>> map = new HashMap<>();
		for(String str : strs){
			char[] arr = str.toCharArray();
			Arrays.sort(arr);
			map.computeIfAbsent(Arrays.toString(arr), v -> new ArrayList()).add(str);
		}
		return new ArrayList<>( map.values() );
    }
}
```

{{< alert warning >}}
```char[] arr = str.toCharArray()```，是本題關鍵之一。toCharArray() 方法將 string 轉換為 char[]。特別注意是 ```char[]``` 而不是 ```Character[]```
{{< /alert >}}

{{< alert warning >}}
```computeIfAbsent```，觀念也可以再次複習一下。
{{< /alert >}}

{{< alert warning >}}
```new ArrayList<>( map.values() )```，也是蠻快的寫法，可以不用特別再用 for loop 了。
{{< /alert >}}

---

# 時間空間複雜度
假設 strs 有 N 個元素；而 K 為 strs 內最長單字的長度 :
### 時間複雜度: ```O(N*K*LogK)```
首先我們知道 for loop 會循環 N 次，而且每次過程中會做 :
- 把單字切成 CharArray ，假設最長時間為: ```O(K)```
- 排序 CharArray: ```O(KLogK)```
- 插入 Map 對應的 List: ```O(1)```

以上取最花時間的步驟的 ```O(KLogK)```。最後整體來看，時間複雜度: ```O(N*K*LogK)```
### 空間複雜度：```O(K*N)```
每次 for loop 都會切一個 CharArray ，其最大空間為 ```O(K)```並作為 Map key 存儲；總共循環 N 次，故最多會花費空間複雜度：```O(K*N)```

---

# Vocabulary

{{< alert info >}}
**anagram** [ˋænə͵græm]:

n.回文構詞法；換音造詞法；錯位詞

{{< /alert >}}

{{< alert info >}}
**phrase** [frez]: (R的發音要出來，念成 face 就不好了)

n.[C]片語，詞組；成語，慣用語

{{< /alert >}}

---

### 參考資料

- [[LeetCode]49. Group Anagrams 中文](https://www.youtube.com/watch?v=OAzLAsTB8Hg&t=133s)

- [49. Group Anagrams](https://walkccc.me/LeetCode/problems/0049/)