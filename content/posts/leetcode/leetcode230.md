---
title: 230. Kth Smallest Element in a BST

author: Aryido

date: 2022-09-11T16:12:22+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java
- binary-tree
- DFS

comment: false

reward: false
---
<!--BODY-->
> 這是一道關於二叉搜索樹 Binary Search Tree 的題目。提示是讓我們用*中序遍歷In-Order*來解題。 可以複習一下 DFS 解法的 Pre-Order、In-Order Post-Order。 另外這道題的 Follow up 可以多思考，是假設該 BST 被修改的很頻繁，而且查找第 k 小元素的操作也很頻繁，問如何優化。

<!--more-->
## 思路1
我們知道 BST 使用 In-Order 的話，可以把所有值，由小到大依序列出。 這題我依序列出的資料加入 List，並限制List大小，最後取最尾端元素，就是第k小的數。

# 解答
```java
class Solution {

    public int kthSmallest(TreeNode root, int k) {
        List<Integer> list = new ArrayList<>();
        dfs(root, list, k);
        return (int)list.get(k-1);
    }

    public void dfs(TreeNode root, List<Integer> list, int k){
        if(root == null){
           return;
        }

        dfs(root.left, list, k);
        if(list.size() == k){
            return;
        }
        list.add(root.val);
        dfs(root.right, list, k);
    }
}
```
## 思路2
由於 BST 的性質，可以快速定位出第 k 小的元素是在左子樹還是右子樹。

- 首先計算出左子樹的結點個數總和cnt
- 如果 k 小於等於左子樹cnt，說明第 k 小的元素在左子樹中，直接對左子結點調用遞歸即可。
- 如果 k 大於 cnt+1，說明目標值在右子樹中，對右子結點調用遞歸函數，注意此時的 k 應改為 k-cnt-1，應為已經減少了 cnt+1 個結點。
- 如果k正好等於 cnt+1，說明當前結點即為所求，返回當前結點值即可

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        int count = countNodes(root.left);
        if (k <= count) {
            return kthSmallest(root.left, k);
        } else if (k > count + 1) {
            return kthSmallest(root.right, k-count-1);
        }
        return root.val;
    }

    public int countNodes(TreeNode n) {
        if (n == null) return 0;
        return 1 + countNodes(n.left) + countNodes(n.right);
    }
}
```
# Follow up
> 假設該 BST 被修改的很頻繁，而且查找第k小元素的操作也很頻繁，問如何優化。

解法利用 *思路2* 來快速定位目標所在的位置。進一步需修改原 tree 結構，使其保存包括當前結點和其左右子樹結點的個數。

定義了新 class，然後要生成新樹，還是用遞歸的方法生成新樹，注意生成的結點的 count 值要累加其左右子結點的 count 值。

在遞歸函數中，不能直接訪問左子結點的 count 值，因為左子節結點不一定存在，所以要先判斷。如果不存在的話，當此時k為1的時候，直接返回當前結點值，否則就對右子結點調用遞歸函數，k自減1。

{{< alert info >}}
這個寫法最大的好處，是 generateMyTreeNode 一次之後，後續再換不同 k 值，可以直接利用已存的左右子樹結點的個數來求解答案，解決查找第 k 小元素很頻繁的問題。
{{< /alert >}}

```java
class Solution {
    class MyTreeNode{
        int val;
        int count = 1;
        MyTreeNode left;
        MyTreeNode right;
        MyTreeNode(int val) {
            this.val = val;
        }
    }

    public MyTreeNode generateMyTreeNode(TreeNode root){
        if(root == null){
            return null;
        }

        MyTreeNode node = new MyTreeNode(root.val);
        node.left = generateMyTreeNode(root.left);
        node.right = generateMyTreeNode(root.right);

        if(node.left != null){
            node.count += node.left.count;
        }

        if(node.right != null){
            node.count += node.right.count;
        }
        return node;

    }

    public int kthSmallest(TreeNode root, int k) {
        MyTreeNode myTreeNode = generateMyTreeNode(root);
        return count(myTreeNode, k);
    }

    public int count(MyTreeNode myNode, int k){
        if(myNode.left != null){
            int cnt = myNode.left.count;

            if(k <= cnt){
                return count(myNode.left, k);
            } else if(k > cnt+1){
                return count(myNode.right, k-cnt-1);
            }

            return myNode.val;
        }else{
            if(k == 1){
                return myNode.val;
            }
            return count(myNode.right, k-1);
        }
    }
}
```
---