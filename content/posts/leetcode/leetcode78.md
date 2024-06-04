---
title: "78. Subsets"

author: Aryido

first draft: 2022-11-06T22:54:12+08:00

date: 2023-10-17T22:11:25+08:00

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
>很經典的問題 : **冪集 the power set** ，在數學上還蠻常見到的，理論上求得解答方式也很簡單，選或者不選排列組合，就可以得出答案。但在程式上要實現卻有一點點難度，故會被歸類到 Medium 等級。這邊使用 Backtrack 模板來解題。
<!--more-->

---

## 思路
冪集是要蒐集所有 subset，這裡表示的方式是，**把每一層 inner list 狀態都保存下來**。注意一個很重要的假設: **nums 中的數字各不相同**，故可以不用思考數字重複的情況，所有答案都收集起來也不會有重複的 case。構造 tree 如下圖 :
{{< image classes="fancybox fig-100" src="/images/leetcode/78.jpg" >}}


{{< alert info >}}
注意空集合 [] 也是一個 subset，但是這不是特殊 case，全部都不選就是 empty subset 了
{{< /alert >}}

Subset 類別題目和 Permutation 題目不一樣的點，在於其需要在 tree 的所有節點，**執行加入結果集**這操作，而全排列只需要在葉子節點執行就可以了。

---

# 解答
```java
class Solution {
	List<List<Integer>> ans = new ArrayList<>();

	public List<List<Integer>> subsets( int[] nums ) {
		dfs( nums, new ArrayList(), 0 );
		return ans;
	}

	public void dfs( int[] nums, List<Integer> inner, int level ) {
		ans.add( new ArrayList( inner ) );

		for ( int i = level; i < nums.length; i++ ) {
			inner.add( nums[i] );
			dfs( nums, inner, i + 1 );
			inner.remove( inner.size() - 1 );
		}
	}
}
```

---

# 時間空間複雜度
假設 nums 有 N 個元素:
### 時間複雜度: ```O(N*2^N)```

Backtrack 時間複雜度，會由 recursion tree 的 **Node 個數**和 **Node 行為**決定。
- {{< hl-text red >}}Node 個數{{< /hl-text >}}即為**所有 subset 個數**:```2^N```
- {{< hl-text red >}}Node 行為{{< /hl-text >}}，主要是複製 List ，這邊就以可能複製的最長的 List，取花費時間的上限:```O(N)```

最後總體看一下，假設每個 Node 都花費最長```O(N)```時間，則時間複雜度:```O(N*2^N)```

### 空間複雜度：```O(N)```
recursion tree 會產生一個 recursion stack ，其最深度剛好就是 N 。再來 code 沒有創建額外的存儲空間，故每個 Node 還是常數空間複雜度。

總結以上，故空間複雜度為(深度 * 常數空間): ```O(N*1)=O(N)```

---
### 參考資料

- [[LeetCode] 78. Subsets 子集合](https://www.cnblogs.com/grandyang/p/4309345.html)

- [子集（ LeetCode 78 ）](https://www.algomooc.com/2891.html)

- [LeetCode 第 78 题 動畫](https://liweiwei1419.github.io/leetcode-solution-blog/leetcode-problemset/backtracking/0078-subsets.html#%E6%96%B9%E6%B3%95%E4%BA%8C%EF%BC%9A%E4%BD%BF%E7%94%A8%E4%BD%8D%E6%8E%A9%E7%A0%81)