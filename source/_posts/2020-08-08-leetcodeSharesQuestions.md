---
layout: post
title: leetcode股票问题系列总结
tag: 技术笔记
date: 2020-8-8
category: Technology blog
---
### leetcode股票问题系列总结

![141853](141853.png)

首先我们对本题所有状态进行一次穷举

```java
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            profit[状态1][状态2][...] = 择优(选择1，选择2...)
```

在这个问题里面，每天都有三种选择，也就是买入，卖出，无操作。但这三种操作是有顺序的，因为卖出必须在买入之后，买入必须在卖出之后，那么无操作的情况也要分两种，一种是持有股票选择了无操作，一种则是没有股票选择了无操作，我们可能对交易次数还有限制，也就是说你的买入是有前提限制的。

所以本类题目所有的状态其实有三个，第一个是天数，第二个是允许交易的最大次数，第三个则是当前的持有状态，在这里我们通过一个三维数组实现

```java
//这里第一个i代表天数，第二个k代表当前交易剩下的次数，第三0 or 1则代表了持有和未持有的两种状态。
profit[i][k][0 or 1]
//i需要小于总的交易天数且大于等于0，k剩下的交易次数则必须小于总的交易次数且大于等于1。
0 <= i <= n-1, 1 <= k <= K
//这里所有的状态一共就有n * K * 2钟
for 0 <= i < n:
    for 1 <= k <= K:
        for s in {0, 1}:
            profit[i][k][s] = max(买入, 卖出, 无操作)//所有的操作全部穷举
```

那我们每天有哪些可能性呢？我们来列举一下

```java
profit[i][k][0] = max(profit[i-1][k][0], profit[i-1][k][1] + prices[i])
              max(   选择不做操作 ,             选择卖出      )

/*解释：今天我没有持有股票，有两种可能：
要么是我昨天[i - 1]就没有持有[0]，然后今天选择不做操作，所以我今天还是没有持有；
要么是我昨天[i - 1]持有股票[1]，但是今天我 卖出[ +prices[i] ] 了，所以我今天没有持有股票了。*/

profit[i][k][1] = max(profit[i-1][k][1], profit[i-1][k-1][0] - prices[i])
              max(   选择不做操作  ,           选择买入        )

/*解释：今天我持有着股票，有两种可能：
要么我昨天[i - 1]就持有着股票[1]，然后今天选择不做操作，所以我今天还持有着股票；
要么我昨天[i - 1]本没有持有[0]，但今天我选择买入[ -prices[i] ]，所以今天我就持有股票了。*/
```

接下里就是整个框架的base case

```java
profit[-1][k][0] = 0
//因为 i 是从 0 开始的，所以 i = -1 意味着还没有开始，这时候的利润当然是 0 。
profit[-1][k][1] = -infinity
//还没开始的时候，是不可能持有股票的，用负无穷表示这种不可能。
profit[i][0][0] = 0
//因为 k 是从 1 开始的，所以 k = 0 意味着根本不允许交易，这时候利润当然是 0 。
profit[i][0][1] = -infinity
//不允许交易的情况下，是不可能持有股票的，用负无穷表示这种不可能。
```

再整理一下，便可以得到整套系列题的通用解法

```java
base case：
profit[-1][k][0] = profit[i][0][0] = 0
profit[-1][k][1] = profit[i][0][1] = -infinity

状态转移方程：
profit[i][k][0] = max(profit[i-1][k][0], profit[i-1][k][1] + prices[i])
profit[i][k][1] = max(profit[i-1][k][1], profit[i-1][k-1][0] - prices[i])
```

面对这道题

我们直接带入框架可得到

```java
profit[i][1][0] = max(profit[i-1][1][0], profit[i-1][1][1] + prices[i])
profit[i][1][1] = max(profit[i-1][1][1], profit[i-1][0][0] - prices[i]) 
            = max(profit[i-1][1][1], -prices[i])
```

简化后，我们直接写出代码

```java
public static int maxProfit(int[] prices) {
    //数组为空或长度为0时没有收益返回零
        if(prices == null || prices.length == 0)
            return 0;
    //这里确定交易天数
        int n = prices.length;
    //这里定义状态方程的数组，因为本题只允许交易一次，简化是K就对本题没有影响了
        int profit[][] = new int[n][2];
    //base case确定
        profit[0][0] = 0;
        profit[0][1] = -prices[0];
        for(int i = 1;i < n;i++){
    //套用框架
            profit[i][0]=Math.max(profit[i - 1][0], profit[i - 1][1] + prices[i]);
            profit[i][1]=Math.max(profit[i - 1][1], -prices[i]);
        }
        return profit[n - 1][0];
    }
```

我们再来看看系列问题的第二题

![capture_20200808150500881](capture_20200808150500881.bmp)

这里有一个变化，就是不会对你的交易次数做限制，因此我们的代码变化一下

```java
  public int maxProfit(int[] prices) {
           if(prices == null || prices.length == 0)
            return 0;
        int n = prices.length;
        int profit[][] = new int[n][2];
        profit[0][0] = 0;
        profit[0][1] = -prices[0];
        for(int i = 1; i < n; i++){
            //在这里因为K是无穷，k与k - 1等效，所以也可以省略，但注意买入的时候我的状态有可能是上次卖出后的状态，因此profit[i - 1][0]不能省略了
            profit[i][0]=Math.max(profit[i - 1][0], profit[i - 1][1] + prices[i]);
            profit[i][1]=Math.max(profit[i - 1][1], profit[i - 1][0] - prices[i]);
        }
        return profit[n - 1][0];
    }
```

系列问题的第三题

![capture_20200808151044651](capture_20200808151044651.bmp)

在这里我们的交易次数被限制了，变成了2次，因此我们的引入k值。

```java
public int maxProfit(int[] prices) {
    if(prices == null || prices.length == 0)
            return 0;
    //限制两次交易次数
        int k = 2;
        int n = prices.length;
    //定义三维数组
        int profit[][][] = new int[n][k + 1][2];
    //嵌套循环穷举所有可能性
        for(int i = 0;i < n;i++){
            for(int j = k;j > 0;j--) {
                if(i == 0){
                    // base case
                    profit[i][j][0] = 0;
                    profit[i][j][1] = -prices[i];
                    continue;
                }
                //套用框架
                profit[i][j][0] = Math.max(profit[i - 1][j][0], profit[i - 1][j][1] + prices[i]);
                profit[i][j][1] = Math.max(profit[i - 1][j][1], profit[i - 1][j - 1][0] - prices[i]);
            }
        }
        return profit[n - 1][k][0];
    }
```

最后一题

![批注 2020-08-08 151748](批注 2020-08-08 151748.png)

这里便不会限制你的交易次数了

本来我开始的解法是

```java
  if(prices == null || prices.length == 0)
            return 0;
        int n = prices.length;
        int[][][] profit = new int[n][k + 1][2];
        for(int i = 0; i < n;i ++){
            for(int j = k; j > 0;j --){
                if(i == 0){
                    profit[i][j][0] = 0;
                    profit[i][j][1] = -prices[i];
                    continue;
                }
                profit[i][j][0] = Math.max(profit[i - 1][j][0], profit[i - 1][j][1] + prices[i]);
                profit[i][j][1] = Math.max(profit[i - 1][j][1], profit[i - 1][j - 1][0] - prices[i]);
            }
        }
        return profit[n - 1][k][0];
```

但提交的时候发生了这件事

![7e3587cfae6846b6e0eddfcf06ad8f5](7e3587cfae6846b6e0eddfcf06ad8f5.png)

k为10亿！？

远远的超过了数组长度，也就是说，我可以无限交易，这样便可以不用讨论动态规划，直接退化贪心算法即可解决问题，k >= n / 2就可以，买入卖出毕竟需要两天

```java
 public int maxProfit(int k, int[] prices) {
         if(prices == null || prices.length == 0)
            return 0;
        int n = prices.length;
     //当K大于所有天数的一半的时候
        if(k >= prices.length / 2)
            return greedy(prices);
        int[][][] profit = new int[n][k + 1][2];
        for(int i = 0; i < n;i ++){
            for(int j = k; j > 0;j --){
                if(i == 0){
                    profit[i][j][0] = 0;
                    profit[i][j][1] = -prices[i];
                    continue;
                }
                profit[i][j][0] = Math.max(profit[i - 1][j][0], profit[i - 1][j][1] + prices[i]);
                profit[i][j][1] = Math.max(profit[i - 1][j][1], profit[i - 1][j - 1][0] - prices[i]);
            }
        }
        return profit[n - 1][k][0];
    }
//退化为贪心
    private int greedy(int[] prices){
        int max = 0;
        for(int i = 1; i < prices.length; ++i) {
            if (prices[i] > prices[i - 1]) {
                max += prices[i] - prices[i-1];
            }
        }
        return max;
    }
```

> 参考
>
> [一个方法团灭 6 道股票问题](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/solution/yi-ge-fang-fa-tuan-mie-6-dao-gu-piao-wen-ti-by-l-3/)