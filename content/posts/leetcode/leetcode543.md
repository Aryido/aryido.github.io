---
title: "543. Diameter of Binary Tree"

author: Aryido

date: 2023-04-16T15:59:57+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- dfs

comment: false

reward: false
---
<!--BODY-->
> 這題是要求 Binary Tree 的 diameter ，要注意一下 diameter 的定義並**不等於深度** ! 根據題目中的例子，可了解所謂 diameter 的定義，是兩點之間的最遠距離。雖然 Binary Tree 的 diameter 並不等於深度，但是和深度有非常大的關係，所以解法用 DFS 是比較直觀的想法。(雖然題目難度說是 easy，但我個人覺得應該算初階 medium...)
<!--more-->

---

{{< alert info >}}
兩個節點的路徑之長度以兩者之間的邊數所代表
{{< /alert >}}

diameter 並**不一定會經過 root 結點**，下圖是個反例。
{{< image classes="fancybox fig-100" src="/images/leetcode/543-counterexample.jpg" >}}

## 思路
觀察下範例中 [4,2,1,3] 和 [5,2,1,3]，可以發現 diameter 剛好是 root 的左右兩個子樹的深度之和。
再經過抽象化，可以想成: 後序遍歷 postorder
- 對每一個 node 求出其左右子樹深度之和，這個值作為一個**答案候選值**
- 左右子結點分別調用 dfs 求直徑
- dfs 回傳左右子樹深度比較大的，再加上 1 代表比之前更深一層了

# 解答
```java
class Solution {
    private int ans = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        dfs(root);
        return ans;
    }

    public int dfs(TreeNode root){
        if(root == null){
            return 0;
        }
        int leftLength = dfs(root.left);
        int rightLength = dfs(root.right);
        ans = Math.max(leftLength + rightLength, ans);
        return  Math.max(leftLength, rightLength) + 1;
    }
}
```

---

# Vocabulary

{{< alert info >}}
**diameter** [daɪˋæmətɚ] : (不要念成鑽石...diamond)

n.[C]直徑
{{< /alert >}}

---