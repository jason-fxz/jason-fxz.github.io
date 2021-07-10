---
title: '[USACO18FEB]Taming the Herd G'
date: 2020-09-06 13:22:22
tags: [动态规划]
categories:
- 题解
---
 [『USACO18FEB』 Taming the Herd G](https://www.luogu.com.cn/problem/P4267)
动态规划

<!--more-->


## 题面



#### 题目描述

Early in the morning, Farmer John woke up to the sound of splintering wood. It was the cows, and they were breaking out of the barn again! Farmer John was sick and tired of the cows' morning breakouts, and he decided enough was enough: it was time to get tough. He nailed to the barn wall a counter tracking the number of days since the last breakout. So if a breakout occurred in the morning, the counter would be 0 that day; if the most recent breakout was 3 days ago, the counter would read 3. Farmer John meticulously logged the counter every day.

The end of the year has come, and Farmer John is ready to do some accounting. The cows will pay, he says! But something about his log doesn't look quite right...

Farmer John wants to find out how many breakouts have occurred since he started his log. However, he suspects that the cows have tampered with his log, and all he knows for sure is that he started his log on the day of a breakout. Please help him determine, for each number of breakouts that might have occurred since he started the log, the minimum number of log entries that must have been tampered with.

#### 输入格式

The first line contains a single integer $N$ ($1 \leq N \leq 100$), denoting the number of days since Farmer John started logging the cow breakout counter. The second line contains $N$ space-separated integers. The $i$ th integer is a non-negative integer $a_i$ (at most $100$), indicating that on day $i$ the counter was at $a_i$, unless the cows tampered with that day's log entry.

#### 输出格式

The output should consist of $N$ integers, one per line. The $i$ th integer should be the minimum over all possible breakout sequences with $i$breakouts, of the number of log entries that are inconsistent with that sequence.

#### 样例输入

```markdown
6
1 1 2 0 0 1
```

#### 样例输出

```markdown
4
2
1
2
3
4
```

#### 样例说明

If there was only 1 breakout, then the correct log would look like 0 1 2 3 4 5, which is 4 entries different from the given log.

If there were 2 breakouts, then the correct log might look like 0 1 2 3 0 1, which is 2 entries different from the given log. In this case, the breakouts occurred on the first and fifth days.

If there were 3 breakouts, then the correct log might look like 0 1 2 0 0 1, which is just 1 entry different from the given log. In this case, the breakouts occurred on the first, fourth, and fifth days.

And so on.

---



## 题意简述

奶牛喜欢出逃，有个计数器，出逃那天为0，否则为上一天示数+1。现在给你一个天数为 $N$ 的不一定正确的计数器示数序列 $a$，让你分别求出逃天数为 $d\in[1,n]$ 的合法序列与给你序列的最大匹配。  



## 题解

因为 $n\leq 100$ ，可以 $O(n^3)$ 暴力dp

设 $dp[i][j][k]$ 表前 $i$ 天，逃 $j$ 次，上次逃是第 $k$ 天的最小不匹配数，显然有以下转移


$$
\begin{matrix}
dp[i][j][k] & \underset{\min}{\leftarrow} dp[i-1][j][k]+[a[i]\not  = i-k]  \qquad\text{(1)}\\
dp[i][j][i] & \underset{\min}{\leftarrow} dp[i-1][j-1][k]+[a[i]\not  = 0] \qquad\text{(2)}
\end{matrix}
 $$

（1）：当前不逃离，延续上一次逃离 （2）：当前逃离

最后逃离天数为 $d$ 的答案就是 $\min\limits_{i=1}^{n} dp[n][d][i]$



## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 105;
const int inf = 0x3f3f3f3f;
int dp[N][N][N]; //dp[i][j][k]表前i天，逃了j次，上次逃是第k天最小不匹配数
int a[N],n;
void checkmin(int &x,int y) {if(x>y) x=y;}
int main() {
    scanf("%d",&n);
    for(int i=1;i<=n;i++) scanf("%d",&a[i]);
    memset(dp,0x3f,sizeof(dp));
    dp[1][1][1]=(a[1]!=0);
    for(int i=1;i<=n;i++) {
        for(int j=1;j<=i;j++) {
            for(int k=j;k<=i;k++) {
                checkmin(dp[i][j][k],dp[i-1][j][k]+(a[i]!=i-k));
            }
            for(int k=j-1;k<i;k++) {
                checkmin(dp[i][j][i],dp[i-1][j-1][k]+(a[i]!=0));
            }
        }
    }
    for(int i=1;i<=n;i++) {
        int ans=inf;
        for(int j=1;j<=n;j++) {
            checkmin(ans,dp[n][i][j]);
        }
        printf("%d\n",ans);
    }
}
```
