# 650：一道DP中等难度题引起的思考

题目描述：给定一个字符‘A’，输入整数n表示想要得到n个'A'。只有两种操作可以选择：复制当前所有的A，粘贴上次复制的内容。这个题很绕，似乎跟传统的dp问题不太相似，拿到以后也没有太多的办法，在看了题解之后，特意记录下来。

示例：

```
Input: 3
Output: 3
Explanation:
Intitally, we have one character 'A'.
In step 1, we use Copy All operation.
In step 2, we use Paste operation to get 'AA'.
In step 3, we use Paste operation to get 'AAA'.
```

解题思路：

其实这个问题可以转化成两个子问题，第一个子问题是每次粘贴几个‘A’。由题意可知，当前粘贴的子字符串（长度j）一定是当前总字符串（长度i）的倍数，即 i % j == 0。默认的最小j是1.那么同时，要让粘贴的次数最少，即 i / j 最小，j就要尽量的大。因此，可以得到本题的状态转移方程：

```
 dp[i] = dp[j] + i / j;
 dp[0] = 0, dp[1] = 0.
```

代码如下：

```java
     int[] dp = new int[n + 1];
        for(int i = 2; i <= n; i++){
            dp[i] = i;
            for(int j = 2; j < i /2; j++){
                if(i % j == 0)
                    dp[i] = dp[j] + i / j;
            }
        }
        return dp[n];
    }
```

提交之后只有35.31%。在讨论区看到一种改进的DP算法求解过程。首先我们要明确一个问题，任何大于1的整数都可以由若干质数之积表示，那么假设n=24，那么是3 * 8 还是4 * 6就不再重要了，因为都可以表示为 3 * 2 * 2 *2。那我们在遍历时，从n的一半开始，遇到能整除的质数，就可以停止了。这里j一定是从后向前的，因为每次复制最大质数个‘A’，实际的花费才是最小的！

代码如下：

```java
 public int minSteps(int n) {
        int[] dp = new int[n + 1];
        for(int i = 2; i <= n; i++){
            dp[i] = i;
            for(int j = i / 2; j >= 1; j--){
                if(i % j == 0){
                    dp[i] = dp[j] + i / j;
                    break;
                }
            }
        }
        return dp[n];
    }
```

最后一招可以不通过dp数组，空间复杂度O(1)。假设n=8，8 = 4 * 2,此时我只需要去找4的次数，5，6，7的次数我不需要关心。代码如下：

```java
public int minSteps(int n) {
    int res = 0;
        
    for (int k = n, i = 0; k > 1; k = i) {
        for (i = k >> 1; k % i != 0; i--);
        res += k / i;
    }
        
    return res;
}
```

这个方法100%。