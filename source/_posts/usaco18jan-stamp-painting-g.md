---
title: '[USACO18JAN] Stamp Painting G'
date: 2020-09-06 13:26:03
tags: [动态规划,dp优化]
categories:
- 题解
---
 [『USACO18JAN』 Stamp Painting G](https://www.luogu.com.cn/problem/P4187)
动态规划，dp优化
<!--more-->



## 题面



#### 题目描述

Bessie has found herself in possession of an $N$-unit long strip of canvas ($1 \leq N \leq 10^6$) and she intends to paint it. However, she has been unable to acquire paint brushes. In their place she has $M$ rubber stamps of different colors ($1 \leq M \leq 10^6$), each stamp $K$ units wide ($1 \leq K \leq 10^6$). Astounded by the possibilities that lie before her, she wishes to know exactly how many different paintings she could conceivably create, by stamping her stamps in some order on the canvas.

To use a stamp, it must first be aligned with exactly $K$ neighboring units on the canvas. The stamp cannot extend beyond the ends of the canvas, nor can it cover fractions of units. Once placed, the stamp paints the $K$ covered units with its color. Any given stamp may be used multiple times, once, or even never at all. But by the time Bessie is finished, every unit of canvas must have been painted at least once.

Help Bessie find the number of different paintings that she could paint, modulo $10^9 + 7$. Two paintings that look identical but were painted by different sequences of stamping operations are counted as the same.

For at least 75% of the input cases, $N,K \leq 10^3$.

#### 输入格式

The first and only line of input has three integers, $N$, $M$, and $K$. It is guaranteed that $K \leq N$.

#### 输出格式

A single integer: the number of possible paintings, modulo $10^9 + 7$.

#### 样例输入

```markdown
3 2 2
```

#### 样例输出

```markdown
6
```



---



## 题意简述

用 $M$ 种颜色长度都为 $K$ 的图章涂一个长度为 $N$ 的画布，即每次选择一个长度为 $K$ 的区间都染成一种颜色（覆盖先前的颜色），随便填涂但画布要填满，问画布最终有多少种状态。



## 题解

先不考虑图章大小的（假设 $K=1$ ），显然 $ans=m^n$ ，但这其中有些是不合法的，首先有个显然的结论：因为图章可以覆盖，除了最后以下连续 $K$ 个相同颜色，其他格子都可做到任意颜色。  于是那些最大连续颜色长度 $<K$ 的都不合法，其他都合法。

考虑动态规划，设 $dp[i][j]$ 表示前 $i$ 个格子当前连续 $j$ 个相同颜色的方案数：

初始化  $dp[1][1]=m$ 
颜色与上次相同 $dp[i][j]=dp[i-1][j-1]\quad j\in[2,K-1]$
颜色与上次不同 $dp[i][1]=dp[i-1][j]\times (m-1)\quad j\in[1,K-1]$
统计答案 $ans = m^n-\sum_{i=1}^{K-1} dp[n][i]$

以上做法复杂度  $O(n^2)$ 可拿75 pts

考虑优化dp，其实稍微观察以下转移就会发现：
$dp[i][2～K-1]$ 直接继承 $dp[i-1][1～K-2]$ ，$dp[i][1]=(m-1)\sum_{j=1}^{K-1} dp[i-1][j]$

可以设 $sum=\sum_{j=1}^{K-1} dp[i][j]$ ，每次更新 $sum'=sum+sum\times(m-1)-dp[i-1][K-1]$ ，其中 $dp[i-1][K-1]$ 可用一个队列来维护。

复杂度 $O(n)$ 



## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1000100;
const ll mod = 1e9+7;
int n,m,k;
ll ans=1;
int main() {
    cin>>n>>m>>k;
    for(int i=1;i<=n;i++) ans=(ans*m)%mod;
    ll sum=m;
    deque<ll> q;
    q.push_back(m);
    for(int i=2;i<=n;i++) {
        q.push_front(sum*(ll)(m-1)%mod);
        if(q.size()==k) sum=((sum-q.back())%mod+mod)%mod,q.pop_back();
        sum=(sum+q.front())%mod;
    }
    ans=((ans-sum)%mod+mod)%mod;
    cout<<ans<<endl;
}
```
