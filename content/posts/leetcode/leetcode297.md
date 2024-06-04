---
title: 297. Serialize and Deserialize Binary Tree

author: Aryido

date: 2022-09-25T14:57:06+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- leetCode

tags:
- binary-tree

comment: false

reward: false
---
<!--BODY-->
> 這題使用深度優先 Depth First Traversal 來遍歷，並使用 Pre-Order 方式記錄樹的節點值；Deserialize 時有用到 queue 來儲存節點 value 值。
> 之前文章也分享過，在想要 Copy Tree 時適合使用Pre-Order。這題有點符合 Copy Tree 的情境，但是是把 value 存下來。

<!--more-->
# Serialize & Deserialize 介紹
- Serialize 序列化就是將一個程式中的 object 轉化為一個可以存進一個文件或者內存緩衝器中的格式，通常是 byte array 或 string。
- Deserialize 反序列化，就是把反過來把 byte array 或 string 還原成程式中的 object。

## 思路
首先先做 Serialize 部分:
{{< alert info >}}
1. 用 StringBuilder() 來建造字串
2. StringBuilder()其中用空白隔開每個節點值
3. 若遇到無法往下遍歷的 null 節點部分，我用井字號 `#` 代替
4.  使用 **pre-order**
{{< /alert >}}

再來做 Deserialize 部分:
{{< alert info >}}
1. 輸入的 data 先使用 split 切開分隔符號，我 Serialize 時是用空白隔開每個節點值，故使用``split(" ")``
2. 小技巧: 使用 Collections.addAll 方法把切割資料放入 queue 中
3. 注意! 因為 Serialize 使用 pre-order，故 Deserialize 也要使用 **pre-order** 造出 TreeNode
{{< /alert >}}

# 解答
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        if(root == null){
            return null;
        }
        StringBuilder builder = new StringBuilder();
        serialize(root, builder);
        return builder.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
         if(data == null){
            return null;
        }
        String[] split = data.split(" ");
        Queue<String> queue = new ArrayDeque<>();
        Collections.addAll(queue, split);
        return buildTree(queue);
    }

    public void serialize(TreeNode node, StringBuilder builder) {
        if(node != null){
            builder.append(node.val).append(" "); //pre-order
            serialize(node.right, builder);
            serialize(node.left, builder);
        }else{
            builder.append("#").append(" ");
        }
    }

    public TreeNode buildTree(Queue<String> queue){
        if(queue.isEmpty()){
            return null;
        }
        String val = queue.poll();
        if(val.equals("#")){
            return null;
        }
        TreeNode node = new TreeNode(Integer.valueOf(val));
        node.right = buildTree(queue);
        node.left = buildTree(queue);
        return node;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec ser = new Codec();
// Codec deser = new Codec();
// TreeNode ans = deser.deserialize(ser.serialize(root));
```

# Vocabulary
{{< alert info >}}
**transmit**[trænsˋmɪt] :

vt.傳送

transfer、transmit 基本上意思一樣，但常用於軟體工程的 data 傳送。 deliver 有實體物品交付的概念，例如貨物的交付就常用 deliver 來表示。
{{< /alert >}}

{{< alert info >}}
**clarification**[klærəfəˋkeʃən] :
**clarify**[ˋklærə͵faɪ]

n.（意義等的）澄清、說明

vt.澄清；闡明

{{< /alert >}}

---