---
title: "15. 3Sum"

author: Aryido

date: 2023-02-09T22:56:05+08:00

thumbnailImage: /images/leetcode/logo.jpg

categories:
- LeetCode

tags:
- java

comment: false

reward: false
---
<!--BODY-->
> 算是 Two Sum 的一種另類進階，從 *nums* 中找出和為 0 的三個  element ，並組成 *List of list* 。特別注意，不能有兩個內容一樣的 list。因為整個題目並沒有對 *nums* 的 *index* 有任何要求，故可以*把 nums 排序*，為解題拓開另一種思路。
<!--more-->

---

## 思路
對原 *nums* 進行排序，然後開始 *for loop* 排序後的 *nums* 。這裡注意 ! 不是遍歷到最後一個停止，而是到倒數第三個就可以了。 接著處理

- 三數和小於 0，則將左邊那個指針 *j* 右移，使得三數和增大一些。
- 三數和大於 0，則將右邊那個指針 *k* 左移，使得三數和變小一些。
- 三數和等於 0，為答案之一，要記錄下來。

# 解答
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        Set<List<Integer>> ans = new HashSet<>();
        for(int i = 0 ; i < nums.length -2 ; i++){
            int j = i + 1;
            int k = nums.length - 1;

            while(j < k){
                if(nums[i] + nums[j] + nums[k] == 0){
                    List<Integer> list = List.of(nums[i] , nums[j] ,nums[k]);
                    ans.add(list);
                    j++;
                    k--;
                } else if( nums[i] + nums[j] + nums[k] > 0){
                    k--;
                } else {
                    j++;
                }
            }
        }
        return ans.stream().map(e -> e).collect(Collectors.toList());
    }
}
```
---

{{< alert info >}}
**Java** 有些常用的**lib**或語法，記得的話其實都可以幫助寫出簡潔的 code。
```
## java9 提供的 List.of
List<Integer> list = List.of(nums[i] , nums[j] ,nums[k]);

```
{{< /alert >}}


{{< alert info >}}
題目的條件 ```-10^5 <= nums[i] <= 10^5```，故三數和不會超過```Integer.MAX_VALUE```。
{{< /alert >}}

{{< alert warning >}}
可以仔細西考一下指針若遇到重複數字怎麼辦，以下進階優化 :
```
if(nums[i] == 0){
    return ans;
}
while (i < k && nums[i] == nums[i + 1]) i++;
while (j < k && nums[j] == nums[j + 1]) j++;
while (j < k && nums[k] == nums[k - 1]) k--;
```
{{< /alert >}}

---

## Refactor
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> ans = new ArrayList<>();
        for(int i = 0 ; i < nums.length -2 ; i++){
            int j = i + 1;
            int k = nums.length - 1;
            while(j < k){
                if(nums[i] + nums[j] + nums[k] == 0){
                    List<Integer> list = List.of(nums[i] , nums[j] ,nums[k]);
                    ans.add(list);
                    if(nums[i] == 0){
                        return ans;
                    }
                    while (i < k && nums[i] == nums[i + 1]) i++;
                    while (j < k && nums[j] == nums[j + 1]) j++;
                    while (j < k && nums[k] == nums[k - 1]) k--;
                    j++;
                    k--;

                } else if( nums[i] + nums[j] + nums[k] > 0){
                    while (j < k && nums[k] == nums[k - 1]) k--;
                    k--;
                } else {
                    while (j < k && nums[j] == nums[j + 1]) j++;
                    j++;
                }
            }

        }
        return ans;
    }
}
```

{{< alert danger >}}

在移動指針的時候，多加 *while* 迴圈判斷，但看起來時間反而花更多了...
```
 } else if( nums[i] + nums[j] + nums[k] > 0){
    while (j < k && nums[k] == nums[k - 1]) k--;
    k--;
} else {
    while (j < k && nums[j] == nums[j + 1]) j++;
    j++;
}
```
{{< /alert >}}

---