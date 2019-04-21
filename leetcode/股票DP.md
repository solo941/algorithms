# 股票DP问题

leetcode上股票的DP问题共有6道，类型相同，总结在一起。

![pic](https://github.com/solo941/algorithms/blob/master/leetcode/pics/20190421232745.png)



## 121:只能买一次

```
Input: [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             Not 7-1 = 6, as selling price needs to be larger than buying price.
```

因为只能买一次，只需要在遍历时记录当前最大收益，也就需要双指针，另一指针记录当前最小购买的花费。

```java
public int maxProfit(int[] prices) {
        if(prices == null || prices.length == 0) return 0;
       int n = prices.length;
        int max = 0;
        int hold = prices[0];
        for(int i = 0; i < n; i++){
            if(prices[i] > hold){
                if(max < prices[i] - hold) max = prices[i] - hold;
            }
            else hold = prices[i];
        }
        return max;
    }
```



## 122:多次买卖

注意规则，买卖不能同时发生。这是一个典型的DP问题，买和卖是两个状态，可以根据当前股票价格进行转换，使用hold维护购买的收益，unhold维护卖出的收益，dp转移方程：

```
hold[i] = Math.max(hold[i-1], unhold[i-1] - prices[i]);
unhold[i] = Math.max(unhold[i-1], hold[i-1] + prices[i]);
hold[0] = -prices[0];
unhold[0] = 0;
```

这一方程贯穿股票DP问题始终。

```java
public int maxProfit(int[] prices) {
        if(prices == null || prices.length == 0) return 0;
        int n = prices.length;
        int[] hold = new int[n];
        int[] unhold = new int[n];
        hold[0] = -prices[0];
        unhold[0] = 0;
        for(int i = 1; i < n; i++){
            hold[i] = Math.max(hold[i-1], unhold[i-1] - prices[i]);
            unhold[i] = Math.max(unhold[i-1], hold[i-1] + prices[i]);
        }
        return unhold[n-1];
    }
```



## 714：多次交易手续费

这个题是123的延续，只不过买进时需要加手续费

```java
public int maxProfit(int[] prices, int fee) {
        if(prices == null || prices.length == 0) return 0;
       int n = prices.length;
        int[] hold = new int[n];
        int[] unhold = new int[n];
        hold[0] = -prices[0] - fee;
        unhold[0] = 0;
        for(int i = 1; i < n; i++){
            hold[i] = Math.max(hold[i-1], unhold[i-1] - prices[i] - fee);
            unhold[i] = Math.max(hold[i-1] + prices[i], unhold[i-1]);
        }
        return unhold[n-1];
    }
```



## 309: 交易时冷却时间

这道题与钱三道不同，规定卖出后需要等待一天，我们将这一天定义为cooldown状态，此时有三种状态相互转移，如下图所示：

![img](https://github.com/solo941/algorithms/blob/master/leetcode/pics/355346516-57b9bc4d5c228.jpg)

在unhold到hold之间加了个cooldown，DP转移方程随着变化

```
hold[i] = Math.max(hold[i-1], unhold[i - 1] - prices[i]);
unhold[i] = Math.max(unhold[i - 1], cooldown[i - 1]);
cooldown[i] = hold[i - 1] + prices[i];
```

只要明确状态的转移方式，也就没这么难了。示例如下：

```
Input: [1,2,3,0,2]
Output: 3 
Explanation: transactions = [buy, sell, cooldown, buy, sell]
```

代码：

```java
public int maxProfit(int[] prices) {
        if(prices == null || prices.length == 0) return 0;
        int n = prices.length;
        int[] hold = new int[n];
        int[] unhold = new int[n];
        int[] cooldown = new int[n];
        hold[0] = -prices[0];
        unhold[0] = 0;
        cooldown[0] = Integer.MIN_VALUE;
        for(int i = 1; i < n; i++){
            hold[i] = Math.max(hold[i-1], unhold[i - 1] - prices[i]);
            unhold[i] = Math.max(unhold[i - 1], cooldown[i - 1]);
            cooldown[i] = hold[i - 1] + prices[i];
        }
        return Math.max(cooldown[n-1], unhold[n-1]);
    }
```



## 123，188：限制交易次数

这两道题我合并为一道，123是交易次数为2，188交易次数为k，188的方法适用于123.这是此类问题中最难的，假设交易次数限制为2次，此时要考虑二维数组的dp转移方程。行数有次数控制，列数则由天数控制，这道题有点像空间优化前的0-1背包问题。状态转移方程可以表示为：

```
hold[i][j] = Math.max(hold[i][j-1], unhold[i-1][j - 1] - prices[j]);
unhold[i][j] = Math.max(hold[i][j - 1] + prices[j], unhold[i][j-1]);
hold[i][0] = -prices[0];
unhold[i][0] = 0;
```

这里一定要注意一种情况，如果交易日期长度少于交易的规定次数2倍时，要按另一种思路解决（否则会数组越界）。此时，最大的交易次数也是符合规定次数的，那么我们能赚钱就交易，这就转化成121的问题。双指针遍历时记录后一天的股票价格，如果有赚就交易。代码如下：

```java
public int maxProfit(int k, int[] prices) {
    if(prices == null || prices.length < 2) return 0;
        int n = prices.length;
        if(k > n / 2) return noLimit(prices);
        int[][] hold = new int[k + 1][n];
        int[][] unhold = new int[k + 1][n];
        for(int i = 1; i <= k; i++){
            hold[i][0] = -prices[0];
            unhold[i][0] = 0;
            for(int j = 1; j < n; j++){
                hold[i][j] = Math.max(hold[i][j-1], unhold[i-1][j - 1] - prices[j]);
                unhold[i][j] = Math.max(hold[i][j - 1] + prices[j], unhold[i][j-1]);
            }
        }
        return unhold[k][n - 1];
        
    }
     public int noLimit(int[] prices){
         int max = 0;
         for(int i = 0; i < prices.length - 1; i++){
             if(prices[i + 1] > prices[i])
                 max += prices[i + 1] - prices[i];
         }
         return max;
     }
```

