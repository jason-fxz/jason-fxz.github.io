---
title: 'Help Shrek and Donkey'
date: 2020-10-29 22:56:24
tags: [博弈论,动态规划]
categories:
- 题解
---

**[CF-98E](https://codeforces.com/problemset/problem/98/E)**

## 题意

A 有 $n$ 张牌， B 有 $m$ 张牌，桌上还有一张反扣着的牌，牌的编号在 $[1, n + m + 1]$ 之间且互不相同。
用这些牌进行博弈，每轮每人可以进行如下操作中的一个:
1. 猜桌面上的反扣着的牌上的数字，猜对则获得胜利，猜错则对方胜利。
2. 询问对方是否有某张牌，若拥有则对方需要出示这张牌，否则继续游戏。

若 A 和 B 都绝顶聪明，求 A 的胜率。  $n, m \le 1000$

<!--more-->

##  题解

> “首先我们清楚的知道双方的策略必然包含随机因素，因为这是一个非完全信息博弈，同时由于这是一个零和博弈，双方的策略都希望能够最小化对方获胜的概率。那么模型就很清楚了，混合策略纳什均衡。” ——搬自[zxyoi](https://blog.csdn.net/zxyoi_dreamer/article/details/102373071)

$dp_{n,m}$ 表先手n张牌，后手m张牌，先手胜的概率，我胜即你输，显然 $dp_{n,m}=1-dp_{m,n}$
然后边界显然： 
1. $dp_{n,0}=1$ 先手知道桌上牌，必胜
2. $dp_{0,m}=\frac{1}{m+1}$ 先手无牌，只能盲猜

对于 $dp_{n,m}$ 其他转移，首先不会轻易猜桌上的牌。
对于询问，先手可以“**欺骗**”或“**猜测**”，后手可以认为先手 欺骗/猜测
“**欺骗**”便是先手询问自己的牌，对手肯定没有，以此误导对手猜桌上的牌。“**猜测**”自然是询问不是自己的牌，大概率能排除对手的牌


分类讨论：
- A. 先手欺骗，后手认为欺骗，相当于后手知道了先手一张牌
$$A=1-dp_{m,n-1}$$
- B. 先手欺骗，后手认为猜测。后手会认为这就是桌上的牌，于是先手必胜 
$$B=1$$
- C. 先手猜测，后手认为猜测。$\frac{m}{m+1}$ 概率后手有，后手牌减一继续，$\frac{1}{m+1}$ 概率后手无（后手知道桌上牌），先手输
$$C=\frac{m}{m+1}\times(1-dp_{m-1,n})$$
- D. 先手猜测，后手认为欺骗，$\frac{m}{m+1}$ 概率后手有，同C，$\frac{1}{m+1}$ 概率后手无，后手不会选，先手能赢
$$\frac{m}{m+1}\times (1-dp_{m-1,n})+\frac{1}{m+1}$$


 先手\后手| 欺骗 | 猜测 |
:----: | :----: | :--: |
  欺骗   |  A   |  B  |
  猜测   |  D   |  C  |

设先手欺骗概率为 $p$ ,根据纳什均衡, (**后手欺骗猜测时先手获胜期望要相同**)

$$
Ap+D(1-p)=Bp+C(1-p)\\
p=(C-D)/(A-D-B+C)
$$

再把 $p$ 带回，
$$
dp_{n,m}=Ap+D(1-p)
$$

## CODE
```cpp
#include <bits/stdc++.h>
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 1005;
int n,m;
double dp[N][N];
double DFS(int n,int m) {
    if(dp[n][m]>=0) return dp[n][m];
    if(n==0) return dp[n][m]=1.0/(m+1);
    if(m==0) return dp[n][m]=1.0;
    double A=1.0-DFS(m,n-1),B=1.0,C=1.0*m/(m+1.0)*(1.0-DFS(m-1,n)),D=C+1.0/(m+1.0);
    double p=(C-D)/(A-D-B+C);
    return dp[n][m]=A*p+(1.0-p)*D;
}
int main() {
    cin>>n>>m;
    for(int i=0;i<=max(n,m);i++)for(int j=0;j<=max(n,m);j++) dp[i][j]=-1;
    printf("%.9lf %.9lf\n",DFS(n,m),1.0-DFS(n,m));
    return 0;
}
```