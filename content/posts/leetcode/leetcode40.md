---
title: "40. Combination Sum II"

author: Aryido

date: 2023-10-10T19:15:23+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- dfs
- backtrack

comment: false

reward: false
---
<!--BODY-->
> 這題是 39. Combination Sum 的進階，給定一個 array 和 target ，找出 candidates 中所有可以使數字和為 target 的組合，但 **candidates 中的每個數字，在每個組合中只能使用一次**。對比 39. 中的數字是可以重複使用，這題是不能重複使用的，但兩題本質沒有區別，依然是使用 DFS 和 backtrack 思想求解。寫到現在其實已經有感覺到一定的模板了，但多多比較與其他 backtrack 題型的差異才是最重要的。
<!--more-->

---

# 思路
跟 [39. Combination Sum](https://leetcode.com/problems/combination-sum/)相比，需要在先前的基礎上修改:
- for loop 裡加上```if ( i != 0 && candidates[i] == candidates[i - 1] && !visited[i - 1] ) continue```，這樣可以防止解答中出現重複項
- 因為數字不能重複使用，遞迴呼叫function 時，裡面的參數換成```i+1```


---

# 解答
```java
class Solution {
	List<List<Integer>> ans = new ArrayList<>();

	public List<List<Integer>> combinationSum2( int[] candidates, int target ) {
		Arrays.sort( candidates );
		dfs( candidates, target, new ArrayDeque<>(), 0, new boolean[candidates.length] );
		return ans;
	}

	public void dfs( int[] candidates, int target, Deque<Integer> stack, int level, boolean[] visited ) {
		if ( target == 0 ) {
			ans.add( new ArrayList<>( stack ) );
			return;
		}
		for ( int i = level; i < candidates.length; i++ ) {
			if ( candidates[i] > target ) {
				return;
			}
			if ( i != 0 && candidates[i] == candidates[i - 1] && !visited[i - 1] ) {
				continue;
			}
			visited[i] = true;
			stack.push( candidates[i] );
			dfs( candidates, target - candidates[i], stack, i + 1, visited );
			stack.pop();
			visited[i] = false;
		}
	}
}
```

{{< alert warning >}}
如果沒有以下這一段，會發生甚麼事情呢
```
if ( i != 0 && candidates[i] == candidates[i - 1] && !visited[i - 1] ) {
	continue;
}
```
**這會導致出現重複組合**。

例如```[1,1,2,5,6,7,10]；target=8```，答案會變成 ```[[6,1,1],[5,2,1],[7,1],[5,2,1],[7,1],[6,2]]```，其中```[5,2,1]```重複了。因為在程式中，兩個 1 是在不同位置，會把它們同時收集起來，但他們視為同個 combination。
{{< /alert >}}


---

# 時間空間複雜度
假設 candidates 內有 N 個元素 :

### 時間複雜度 : ```O(N * 2^N)```
Backtrack 時間複雜度，會由 Recursion tree 的 **Node 個數**和 **Node 行為**決定。

##### {{< hl-text red >}}
Node 個數 :
{{< /hl-text >}}

> 首先題目要求 **candidates 中的每個數字在每個組合中只能使用一次**，所以每個元素分成**選/不選**兩種，故整個 Recursion tree 最多 Node 個數有 ```2^N```。這個是最大值，因為我們知道有 target ，可能 Recursion tree 遍歷到一半就發現總和超過 target 而開始返回。

##### {{< hl-text red >}}
Node 行為 :
{{< /hl-text >}}

> - 非葉子節點: 有插入、移除元素的操作，還有判斷 target、改變 visited 內容，但都是常數級的。
> - 葉子節點: 會需要 copy list ，假設最壞情況是 list 長度為 ```N```，故時間複雜度是 ```O(N)```
>
> 這邊就假設所有 Node 行為都需要花 ```O(N)``` 時間來做事，這邊也是取最大值



承上所有分析，雖然有 array 排序的演算法複雜度 ```O(NlogN)```，但不影響整個時間複雜度分析，故時間複雜度為: ```O(Node個數*Node行為) = O(N * 2^N)```

### 空間複雜度 : ```O(N)```
Recursion tree 深度優先搜索（DFS）會產生一個 recursion stack ，假設答最深深度 ```N```。 再來 code 中沒有創建額外的存儲空間，故每個 Node 還是常數空間複雜度。

總結以上，故空間複雜度為(深度 * 常數空間): ```O(N*1)=O(N)```

---
### 參考資料

- [Leetcode/BackTracking/040_Combination_Sum_II.java](https://github.com/Deadbeef-ECE/Interview/blob/master/Leetcode/BackTracking/040_Combination_Sum_II.javak)