---
title: 'XJ - 赌博（gamble）'
date: 2020-11-24 23:15:40
tags: [XJ,动态规划,组合数学]
categories: 题解
hidden: true
---


[XJ-赌博(gamble)](https://dev.xjoi.net/contest/1595/problem/2)


## 题意

![DNkAXj.png](./DNkAXj.png)

## 题解


**看懂题意很重要**，真是一道阅读理解题，我们考虑到推 $dp$。$dp_{i,j}$ 表示 A 赢了 $i$ 场， B 赢了 $j$ 场的情况下，我当前赚/亏的钱。


$$
\begin{array}{c|ccccc}
dp & 0 & 1 & \cdots&n-1 &n\\
\hline
0 & & & & & -2^{2n-1}\\
1 &  & & & & -2^{2n-1}\\
\vdots & & & & & -2^{2n-1}\\
n-1& & & & 0& -2^{2n-1}\\
n & 2^{2n-1} &2^{2n-1}  &2^{2n-1} & 2^{2n-1} 
\end{array}
$$

考虑 $dp_{n-1,n-1}$ ，下一场等概率分胜负，要满足最后要么赢 $2^{2n-1}$ 要么输 $2^{2n-1}$ ，这场肯定赌 $2^{2n-1}$ 元，且在这时赢的钱等于输的钱，即表格中的 $0$ 。

考虑一般情况也一样，对于 $dp_{i,j}$ 。
$$
\left\{\;
    \begin{matrix}
        dp_{i,j}+x=dp_{i+1,j}\\[1.2ex]
        dp_{i,j}-x=dp_{i,j+1}
    \end{matrix}
\right.
$$

A 赢则赢 $x$ 钱，B 赢则则输 $x$ 钱。 $dp_{i,j}=\cfrac{dp_{i+1,j}+dp_{i,j+1}}{2}$

考虑在状况为A 赢了 $i$ 场， B 赢了 $j$ 场的情况下，下一场赌啥：就是上面 $x$ ，显然 $x=dp_{i+1,j}-dp_{i,j}$

这就是 $O(n^2)$ 做法。考虑如何优化，那就 ~~打表找规律~~ 。我们打出 $f_{i,j}=dp_{i+1,j}-dp_{i,j}$

$$
\begin{array}{c|cc}
f_{i,j} & 0& 1& 2& 3& 4\\
\hline
0 & 140 &140 &120 &80  &32  \\
1 & 140 &160 &160 &128 &64  \\
2 & 120 &160 &192 &192 &128 \\
3 & 80  &128 &192 &256 &256 \\
4 & 32  &64  &128 &256 &512 
\end{array}
$$

然后就能瞪出结论：

$$
\begin{array}{c|cc}
f_{i,j} & 0& 1& 2& 3& 4\\
\hline
0 &C_{8}^{4}\cdot 2^{1}&C_{7}^{4}\cdot 2^{2}&C_{6}^{4}\cdot 2^{3}&C_{5}^{4}\cdot 2^{4}&C_{4}^{4}\cdot 2^{5}\\[1.1ex]
1 &C_{7}^{3}\cdot 2^{2}&C_{6}^{3}\cdot 2^{3}&C_{5}^{3}\cdot 2^{4}&C_{4}^{3}\cdot 2^{5}&C_{3}^{3}\cdot 2^{6}\\[1.1ex]
2 &C_{6}^{2}\cdot 2^{3}&C_{5}^{2}\cdot 2^{4}&C_{4}^{2}\cdot 2^{5}&C_{3}^{2}\cdot 2^{6}&C_{2}^{2}\cdot 2^{7}\\[1.1ex]
3 &C_{5}^{1}\cdot 2^{4}&C_{4}^{1}\cdot 2^{5}&C_{3}^{1}\cdot 2^{6}&C_{2}^{1}\cdot 2^{7}&C_{1}^{1}\cdot 2^{8}\\[1.1ex]
4 &C_{4}^{0}\cdot 2^{5}&C_{3}^{0}\cdot 2^{6}&C_{2}^{0}\cdot 2^{7}&C_{1}^{0}\cdot 2^{8}&C_{0}^{0}\cdot 2^{9}\\[1.1ex]
\end{array}
$$

得出：
$$f_{i,j}=\binom{2\cdot n-i-j-2}{n-i-1}\times 2^{i+j+1}$$

复杂度 $O(n)$

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
const int N = 100010;
const int M = 2010;
const int mod = 1e9+7;
const int inv2 = 500000004;
typedef long long ll;
int n;
ll dp[M][M],inv[N<<1]={1,1},fac[N<<1]={1,1},ifac[N<<1]={1,1},pw[N<<1]={1};
ll fpow(ll a,ll b) {
    ll r=1;for(;b;b>>=1,a=(a*a)%mod) if(b&1) r=(r*a)%mod;return r;
}
ll C(int n,int m) {
    return fac[n]*ifac[m]%mod*ifac[n-m]%mod;
}
ll getval(int i,int j) {
    return C(n*2-2-i-j,n-1-i)*pw[i+j+1]%mod;
}
int main() {
    red(n);
    for(int i=1;i<=2*n;i++) pw[i]=pw[i-1]*2ll%mod;
    for(int i=2;i<=2*n;i++) fac[i]=fac[i-1]*i%mod;
    for(int i=2;i<=2*n;i++) inv[i]=inv[mod%i]*(mod-mod/i)%mod;
    for(int i=2;i<=2*n;i++) ifac[i]=ifac[i-1]*inv[i]%mod;
    int c[2]={0,0};
    for(int i=1;;i++) {
        printf("%lld ",getval(c[0],c[1]));
        int t;
        scanf("%d",&t);
        c[t]++;
        if(c[t]>=n) break;
    }
    printf("\n");
    return 0;
}
```


