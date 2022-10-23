---
title: 560. Subarray Sum Equals K

author: Aryido

date: 2022-10-23T11:03:19+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- map

comment: false

reward: false
---
<!--BODY-->
這題我看起來也是很技巧性的題目，一開始要把 subarray 的特性掌握的淋漓盡致，並且想到用 hashmap 來建立快速查找關係，真的有點困難...但也是這道題的魅力吧 ! 基本上 hashmap 題目大概都會偏向這種步驟應用，多注意可以讓自己視野開闊。

<!--more-->

---

這道題給了一個 array，要找出 **subarray sum**，等於 k 的數量。 **subarray sum** 有甚麼特性呢 ?
{{< alert warning >}}
**subarray sum** 就是
```
sum(x,y) where 0 <= x <= y <= len(array) - 1
```

```
sum(0, y) - sum(0, x) where 0 <= x <= y <= len(array) - 1
```
以上兩敘述等價
{{< /alert >}}

## 思路
當我們知道了 subarray sum 的特性，就可以利用 hashmap 加速查找。 hashmap 怎麼定義呢 ?
- key: sum(0, i)， i 表示 for loop 當前位置
- value: key 出現的次數，從1開始
如果當前 sum - k 存在於 map 內，那麼答案就 += map.get(sum-k)。

因為 sum - k 存在於 map 內，代表之前sum(0, x)出現過 t 次，這代表:
```
subarray sum = current sume - sum(0, x) = k ，這個多出現了 t 次，故要加到答案內。
```

遍歷 array 的數字，用 sum 來記錄到當前位置的累加和，並存入 map 中；若 map 內本來就有該 sum 的紀錄，則把 value + 1，會出現這樣的情況是因為 array 內可能有負值。建立 HashMap 的目的是為了可以快速的查找 sum-k 是否存在。

{{< alert danger >}}

注意一個很容易忘記的東西，初始化 map 後，要把{0,1} 加到 map 內。

比如說 array = [1] 且 k = 1 ，這個時候用上面流程會找去 map 裡面找 k - sum(0,0) = 1 - 1 = 0，故如果沒有存 0 會出錯。

{{< /alert >}}

---

# 解答
```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0,1);
        int ans = 0;
        int sum = 0;
        for(int num : nums){
            sum += num;
            if(map.containsKey(sum - k)){
                ans += map.get(sum - k);
            }
            map.put(sum, map.getOrDefault(sum, 0) + 1);
        }
        return ans;
    }
}
```
{{< alert info >}}
這邊有使用一個 code 精簡小技巧，多注意可以提申自己 code 的乾淨度。
```
map.put(sum, map.getOrDefault(sum, 0) + 1);

# 等價於

if(map.containsKey(sum)){
    int newCount = map.get(sum) + 1 ;
    map.put(sum, newCount);
}else{
    map.put(sum, 1);
}

```
{{< /alert >}}

是一個很優秀的解法只需要 O(n)， 用 hashTable 一邊紀錄累加，一邊算符合的 continuous subarrays。


---

## 思路
暴力解法，先建立一個累加和數組:
```
int size = nums.length;

int[] sumArray = new int[size];
sumArray[0] = nums[0];
for(int i = 1 ; i < size ; ++i ){
    sumArray[i] = sumArray[i - 1] + nums[i];
}
```

然後遍歷累加和數組的每個數字，首先看其是否為k，是的話 ans + 1，然後再往前的遍歷，可以快速求出所有的子數組之和，看是否為 k。

---

# 解答
```
class Solution {
    public int subarraySum(int[] nums, int k) {
        int ans = 0 ;
        int size = nums.length;

        int[] sumArray = new int[size];
        sumArray[0] = nums[0];
        for(int i = 1 ; i < size ; ++i ){
            sumArray[i] = sumArray[i - 1] + nums[i];
        }

        for(int i = 0 ; i < size ; ++i){
            if(sumArray[i] == k){
                ++ans;
            }
            for(int j = i - 1 ; j >= 0 ; j--){
                if(sumArray[i] - sumArray[j] == k){
                    ++ans;
                }
            }
        }

        return ans;
    }
}
```

就算是暴力解，基本上也要想出 **subarray sum** 的特性，也就是建立**累加和數組**的思想，才會通過時間測試。

有累加的 Array，才可以在 O(1) 的時間內求得任何一個 continuous subarrays 的總和。
找所有 continuous subarrays 的時間複雜度是 O(n²)

---