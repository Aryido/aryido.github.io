---
title: "Reservoir sampling"

author: Aryido

date: 2023-10-21T18:33:30+08:00

thumbnailImage: /images/algorithm/reservoir-logo.jpg

categories:
- algorithm

comment: false

reward: false
---
<!--BODY-->
> Reservoir sampling 是一個隨機演算法，其目的是在只遍歷一遍的情況下，從大數據 N 的資料流中，隨機選取出 k 個元素，且每筆資料選中的機率都要一樣。這個場景強調了幾件事：
> - 集合 N 很大且不可知，所以**不能一次存入記憶體**
> - 時間複雜度為 ```O(N)```
> - 隨機選取 k 個數，每個數被選中的機率為```k/N```
>
> 本來面對這種問題，比較直接的想法是利用隨機數演算法，求 random(N) 得到隨機數，但是因**資料流極大**，無法一次都讀到記憶體內，這就表示不能像數組一樣根據 index 獲取元素；而且題目強調**只能遍歷一遍```O(N)```**，代表也不能再採用分塊方式儲存資料，之後再隨機遍歷。為了解決這個問題，可以使用 Reservoir sampling ，非常的巧妙。

<!--more-->
---

# Reservoir Sampling Simple Case
以下 Code 為簡化版範例，當 for loop 跑完整個大數 N 時，result 將以 ```1/N```的機率，指向 ```[1，N]``` 內的任一個數字，是不是感覺很神奇呢 !

```
public class ReservoirSamplingSimpleTest {

	private final int N = 100000;
	private final Random random = new Random();

	public int reservoirSimpleSampling() {
		int result = 0;
		for ( int i = 0; i < N; i++ ) {
			int r = random.nextInt( i + 1 );
			if ( r < 1 ) {
				result = i + 1;
			}
		}
		return result;
	}
}
```

### 分析
假定 N 是一個大於 0 的大數字 :

> 假定大數字 ```N = 1``` 時，result 是 1 的機率是多少呢 ? 因為 for loop 只會循環一次，而其內的亂數 r 是在 ```[0，1)```間，故 ```r <= 1``` 的機率會是 100%，故 result  一定為 1

也就是說 ```result = 1``` 的機率為 ```1```。

> 假定大數字 ```N = 2``` 時 :
>  - 先反過來看，思考 result 是 2 的機率是多少呢 ? 因為要選到 2， for loop 一定要在最後，而亂數 r 那時是在 ```[0，2)```間，故 ```r <= 1```的機率為 ```1/2```，也就是說```result = 2```的機率為 ```1/2```
>  - 再來看選到 1 的機率是多少呢 ? 結果會是 **1 第一次被選到而且不能替換掉**；所以在 ```N = 2``` 情況下是 :
> ```
> (1 被選到機率)*(不能被 2 替換掉的機率) =
> 1 * (1-(1/2)) = 1/2
> ```

承上知道數字 ```result = {1, 2}``` 每個被選到的機率都是 ```1/2```

> 假定大數字 ```N = 3``` 時 :
>   - 先反過來看， result 是 3 的機率是多少呢 ? 因為要選到 3， 一定要在最後 for loop ，而其亂數 r 那時是在 ```[0，3)```間，而 ```r <= 1```的機率為 ```1/3```，也就是說```result = 3```的機率為 ```1/3```
>   - 再來看選到 2 的機率是多少呢 ? 結果會是 **2 第一次被選到而且不能被替換掉**；所以在```N = 3```情況下是 :
> ```
> (2被選到機率)*(不能被 3 替換機率) =
> 1/2 * (1-(1/3)) = 1/3
> ```
>   - 再來看選到 1 的機率是多少呢 ? 在```N = 3```情況下是:
>   ```
>   ( 1 被選到機率)*(不能被 2 替換機率)*(不能被 3 替換機率) =
>   1 * (1-(1/2))*(1-(1/3)) =
>   1* (1/2) * (2/3) = 1/3
>   ```
承上知道數字 ```{1, 2, 3}``` 每個被選到的機率都是 ```1/3```。

再繼續分析下去，機率圖表如下 :
{{< image classes="fancybox fig-100" src="/images/algorithm/reservoir-sampling.jpg" >}}

呈上類推，使用**數學歸納法**可分析證明，知道若數字有 N 個，可以得知 ```result = {1, 2, 3, ... , N}``` 每個被選到的機率都是 ```1/N```。

---

# Reservoir Sampling General Case
再來看一下進階版 General Case ，上面簡單範例是選一個數字而已，如果現在要選 K 個數字，每個數字被選擇的機率為 ```K/N``` 該怎麼表示呢 ?

```
public class ReservoirSamplingTest {
	private final int N = 10;
	private final Random random = new Random();

	private int[] sampling( int K ) {
		int[] result = new int[K];
		for ( int i = 0; i < K; i++ ) {
			result[i] = i + 1;
		}

		for ( int i = K; i < N; i++ ) {
			int r = random.nextInt( i + 1 );
			if ( r < K ) {
				result[r] = i + 1;
			}
		}

		return result;
	}
}
```
> - 對於第 i 的數 ```（i ≤ K)```，在第 K 的數之前，被選中的機率為 1 。
>
> - 當到第 K+1 的數時，第 i 的數被第 K+1 的數替換的機率 =  第 K+1 的數被選中的機率 * i 被選中替換的機率 : ```[K/(K+1)]*(1/K) = 1/(K+1)```；
故第 i 的數被保留的機率為 ```1 - 1/(K+1) = K/(K+1)```。
> - 同理可以推得，第 i 的數不被第 K+2 個數替換的機率為 ```1 - [K/(K+2)]*(1/K) = (K+1)/(K+2)```。

以此類推，運行到第 N 個數時，被保留的機率 = 被選中的機率 * 不被替換的機率，即：
    ```
    1 * [K/(K+1)]*[(K+1)/(K+2)]*[(K+2)/(K+3)]*....*[(N-1)/(N)] = K/N
    ```

> - 對於第 j 的數 ```（j > K)```，第 j 的數被選中的機率為 ```k/j```。
> - 不被 j+1 的數替換的機率為 ```1 - [k/(j+1)]*[1/k]```

以此類推，運行到第 N 個數時，被保留的機率 = 被選中的機率 * 不被替換的機率，即：
    ```
    K/j * [j/(j+1)]*[(j+1)/(j+2)]*[(j+2)/(j+3)]*....*[(N-1)/(N)] = K/N
    ```

承上類推，使用數學歸納法來分析證明，可以知道若數字有 N 個，隨機選 K 個，
每個元素被選到的機率都為 ```K/N```

---

# 時間空間複雜度

### 時間複雜度 : ```O(N)```
在 Code 過程中，result 是一個一個的看機率換數字的，沒有複雜的過程；而且只有一層 for loop，所以很明顯時間複雜度為```O(N)```

### 空間複雜度 :  ```O(1)```
無論我們 N 是多少，記憶體中始終只儲存了 result ，所以它的空間複雜度為```O(1)```

---

### 參考資料

- [【经典算法题】蓄水池抽样算法](https://www.youtube.com/watch?v=aMhe_Riny5E&t=12sY)

- [蓄水池抽样算法（Reservoir Sampling）](https://www.jianshu.com/p/7a9ea6ece2af)

- [蓄水池采样算法](https://www.cnblogs.com/snowInPluto/p/5996269.html)

