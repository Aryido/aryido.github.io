---
title: 26. Remove Duplicates from Sorted Array

author: Aryido

date: 2022-12-25T15:05:58+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 這道題要我們從**有序數組**中去除重複項，題目難度雖然被歸為 easy 等級，但在**條件限制**上的討論，蠻多東西可以釐清討論的。 英文方面寫得蠻長，記得看到最後，有些限制如 :
> - *O(1)* extra memory
> - The relative order of the elements should be kept the same.
>
> 所以**不要用 Set 或另外 array 去寫**。


<!--more-->

---
兩個重點條件限制 :
- *O(1)* extra memory

- The relative order of the elements should be kept the same.

以下都使用我比較不符合我思考的方式來解題，當作練習。

## 思路
使用**快慢指針**來遍歷 array :
- **lower** 和 **faster** 都從 index 0 開始
- **lower**之前的 index 都是處理好的數字

接下來開始循環邏輯 :

- 若 ```faster == 0``` ，則為初始狀態，**快慢指針**都前進。

- 因為定義 **lower** 之前的 index 均為處理好的資料，故若條件 ```nums[lower - 1 ] != nums[faster]``` ，則 **快慢指針**都前進，並把 ```nums[lower]``` 換成  ```nums[faster]```

{{< alert info >}}
很自然的，可以把 ```faster == 0``` 和 ```nums[lower - 1 ] != nums[faster]``` 條件併在一起。因為初始狀態 :
-  ``` faster = 0 ```
-  ``` lower = 0 ```

這樣 ```nums[lower] = nums[faster]``` 改動其實等於沒改。
{{< /alert >}}

---

# 解答
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int faster = 0;
        int lower = 0;
        while( faster < nums.length){
            if(faster == 0 || nums[lower - 1] != nums[faster]){
                nums[lower++] = nums[faster++];
            }else{
                faster++;
            }
        }
        return lower;
    }
}
```
{{< alert success >}}
``` nums[lower++] = nums[faster++];``` 是個蠻簡潔不錯的寫法，可以在賦值完之後，才再讓指針前進。想提醒自己這個部分，才特別註記筆記這題的。
{{< /alert >}}

{{< alert warning >}}
思考了一下，這題的條件 :
- The relative order of the elements should be kept the same.

好像沒有用，因為雙指針的題型，會改變 order 的是**反向指針**，這題應該不能用反向指針去解題...
{{< /alert >}}

---

## 其它思路
一樣是使用**快慢指針**來遍歷 array ，最開始時兩個指針都指向第一個數字，但
- 兩個指針指的數字相同，則快指針向前走一步
- 如果不同，則慢指針先向前一步，把慢指針對應 index 數字換成快指針對應 index 數字，再來快指針前進

兩個指針都會向前走一步，這樣當快指針走完整個數組後，慢指針當前的**坐標加 1** 就是數組中不同數字的個數。

# 解答
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int faster = 0;
        int lower = 0;
        while( faster < nums.length ){
            if (nums[lower] == nums[faster]){
                ++faster;
            }else{
                nums[++lower] = nums[faster++];
            }
        }
        return lower + 1;
    }
}
```

用 for loop 也可以解 !

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int j = 0;
        for(int i = 0 ; i < nums.length ; i++){
            if(nums[i] != nums[j]){
                nums[++j] = nums[i];
            }
        }

        return j + 1 ;
    }
}
```
