---
title: 78. Subsets

author: Aryido

date: 2022-11-06T22:54:12+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- dfs
- backtrack

comment: false

reward: false
---
<!--BODY-->
>這題是很經典的問題: **冪集 power set** ，在數學上還蠻常見到的，理論上求得解答方式也很簡單，選或者不選就可以得出。但在程式上卻有點點難度，會被歸類到 Medium 等級。這邊使用 backtrack 模板來解題，但其實也有非遞回式的解法，都可看看並練習。
<!--more-->

---

## 思路
由於每一個數字只有兩種狀態，選或者不選，所以可以構造一棵二叉樹如下圖 :
```
                        []
                   /          \
                  /            \
                 /              \
              [1]                []
           /       \           /    \
          /         \         /      \
       [1 2]       [1]       [2]     []
      /     \     /   \     /   \    / \
  [1 2 3] [1 2] [1 3] [1] [2 3] [2] [3] []
```
左子樹表示選擇該層處理的節點，右子樹表示不選擇，整個添加的順序為：
- [1,2,3]
- [1,2]
- [1,3]
- [1]
- [2,3]
- [2]
- [3]
- []

{{< alert warning >}}
注意空集合 [] 也是一個 subset，但是這不是特殊 case，全部都不選就是 empty subset 了
{{< /alert >}}

{{< alert warning >}}
注意 java 的 array list 是傳地址，所以要加入解答時記得要```new ArrayList<>(curr)```，複製一份新的加進去
{{< /alert >}}

{{< alert warning >}}
```curr.remove(curr.size() - 1);```，代表狀態的復原
{{< /alert >}}

---

# 解答
```java
class Solution {
     List<List<Integer>> ans = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) {
        dfs(nums, 0 , new ArrayList<>());
        return ans;
    }

    public void dfs(int[] nums, int index, List<Integer> curr){
        if(index >= nums.length){
            ans.add(new ArrayList<>(curr));
            return;
        }
        curr.add(nums[index]);
        dfs(nums, index + 1 , curr);
        curr.remove(curr.size() - 1);
        dfs(nums, index + 1 , curr);
    }
}
```
時間複雜度可以這樣想，有 2^N 個 subsets，**但是每個 subset 不僅僅只花 O(1)**。每個 subset 的產生都是對於每個數字 “選或不選”，選是一個動作，不選也是一個動作，所以每個 subset 的產生其實都有 N 個動作，所以:

{{< alert danger >}}
時間複雜度:  N x 2^N
{{< /alert >}}

---

## 思路

比如對於題目中給的例子 [1,2,3] 來說，最開始是[]
- 處理1，就是[]先複製後上加1，變為 [1]，則現在有兩個子集 [] 和 [1]
- 處理2，在之前的子集基礎上，每個都加個2，可以分別得到 [2]，[1, 2]，則現在所有的子集合為 [], [1], [2], [1, 2]
- 處理3，可得 [3], [1, 3], [2, 3], [1, 2, 3], 再加上之前的子集就是所有的子集合了
- 可以歸納出就是之前所有的組合(不選)，再全部加入目前數字(選)!

---

# 解答
```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> ans = new ArrayList<>();
        ans.add(new ArrayList<>());
        for(int num : nums){
            int size = ans.size();
            for(int i = 0 ; i < size ; i++){
                List<Integer> curr = new ArrayList<>(ans.get(i));
                ans.add(curr);
                ans.get(i).add(num);
            }
        }
        return ans;
    }
}

// []
// [],[1]
// [2],[1,2],[],[1]
// [2,3],[1,2,3],[3],[1,3], [2],[1,2],[],[1]
```

{{< alert danger >}}
一開始寫的時候，把```int size = ans.size();```寫到 for 迴圈裡面所以錯了。

因為ans list 會一直增加數量，放到  for 迴圈裡面的話會造成死循環。
{{< /alert >}}


---
### 參考資料

- [[LeetCode] 78. Subsets 子集合](https://www.cnblogs.com/grandyang/p/4309345.html)
