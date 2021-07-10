---
title: 'XJ - Sequence on Tree'
date: 2020-11-21 21:55:18
tags: [XJ,动态规划,dp优化]
categories: 题解
hidden: true
---

<!--more-->

[XJ - Sequence on Tree](https://dev.xjoi.net/contest/1597/problem/3)


------


## 题意
小 Y 有一棵以 $1$ 为根的有根树，节点标号 $1,2,\dots,n$，节点 $i$ 的点权为 $w_i$。

对于任意的起始点 $1\le s\le n$，小 Y 想找到节点序列 $v_1,v_2,\dots,v_m$ 满足

- $v_1=s$，同时对于任意 $2\le i\le m$，$v_i$ 是 $v_{i−1}$ 的祖先。
- 价值 $f(s)=w_{v_1}+\sum ^m_{i=2}(w_{v_i}\, \operatorname{opt} \,w_{v_{i−1}})$ 最大，其中 $opt$ 可能是位运算 `AND`，`OR` 或者 `XOR`。

小 Y 想知道 $S=\sum ^n_{i=1} i⋅f(i)$ 的值，答案对 $10^9+7$ 取模。

$$1\le n\le 10^6,0\le w_i\lt 2^{30},\mathrm{opt}\in \{\mathrm{AND,OR,XOR}\}$$


## 题解

看上去十分毒瘤，~~然后写个 $n^2$ DP 就会发现~~ （ $O(n^2)$ DP 有手就行） ,$\mathrm{OR,XOR}$ 每个点的 $f(u)$ 只要一路到根就行了。

让我们来~~严谨~~证明一下：

对于 $\mathrm{OR}$ ，假设 $p$ 是 $u$ 的祖先，$v$ 是 $p,u$ 路径上的一点 （显然也是 $u$ 的祖先）
方案一 $p\to u$ ： $w_p |w_u$
方案二 $p\to v\to u$ ： $w_p|w_v+w_v|w_u$

方案二中多了 $w_v$ 的贡献，方案二相比一肯定不劣，数学归纳一下，全取完一定最优。

对于 $\mathrm{XOR}$，同样假设 $p$ 是 $u$ 的祖先，$v$ 是 $p,u$ 路径上的一点，我们考虑 $w$ 二进制中的任意一位（因为 $\mathrm{XOR}$ 不进位，每一位都是独立的）。$f_0,f_1$ 分别表示 $p\to u$ ；$p\to v\to u$ 这一位的贡献

$$
\begin{array}{l|c}
w_p & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 \\
w_v & 0 & 0 & 1 & 1 & 0 & 0 & 1 & 1 \\
w_u & 0 & 1 & 0 & 1 & 0 & 1 & 0 & 1 \\
f_0 & 0 & 1 & 0 & 1 & 1 & 0 & 1 & 0\\
f_1 & 0 & 1 & 2 & 1 & 1 & 2 & 1 & 0
\end{array}
$$

可以发现，无论什么取值，每位上 $f_1$ 比 $f_0$ 不劣，选 $p\to v\to u$ 比选 $p\to u$ 不劣，故全取完最优。

证毕。

剩下的就是考虑 $\mathrm{AND}$ 的情况了，还是考虑 $p$ 是 $u$ 的祖先，$v$ 是 $p,u$ 路径上的一点，令 $w_p \& w_u$ 的二进制最高位为 $k$ ，假如 $w_v$ 的二进制第 $k$ 位是 $1$ ，那么选 $p\to v\to u$ 比选 $p\to u$ 优秀。

前者相比后者多出了 $2\times 2^k$ 的贡献，而最多也只少了不到 $2^k$ 的贡献。

那么设计一个 DP ，$dp[u]$ 就表示 $f(u)-w_u$ ，在维护一个 $jp[u][i],i\in[0,29]$ ，表示 $u$ 祖先方向（包含 $u$ ），$w$ 的第 $i$ 位为 $1$ 且最深的点。更新 $dp[u]$ ，我们枚举按位与后的最高位 $k$，并用 $jp[fa_u][k]$ 来更新。并更新 $jp$ 。(详见代码)


## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1000010;
const int mod = 1e9+7;
typedef long long ll;
template<typename T> bool umax(T &x,T y) {return (x<y)?(x=y,true):(false);}
template<typename T> bool umin(T &x,T y) {return (x>y)?(x=y,true):(false);}
int n,q,opt,fa[N];
ll w[N];
string str;
ll calc(ll a,ll b) {
    if(opt==1) return a^b;
    if(opt==2) return a|b;
    if(opt==3) return a&b;
}
namespace sub1 {  // XOR ,OR
    ll ans=0,f[N];
    void solve() {
        for(int i=2;i<=n;i++) {
            f[i]=(f[fa[i]]+calc(w[i],w[fa[i]]))%mod;
        }
        for(int i=1;i<=n;i++) {
            f[i]=(f[i]+w[i])%mod;
            ans=(ans+f[i]*i)%mod;
        }
        printf("%lld\n",ans);
    }
}
namespace sub2 {
    ll dp[N],jp[N][30],ans; // jp[u][i] the last anc that (w>>i)&1==1
    void solve() {
        for(int i=0;i<=29;i++)
            if((w[1]>>i)&1) jp[1][i]=1;
        for(int u=2;u<=n;u++) {
            for(int i=0;i<=29;i++) {
                umax(dp[u],dp[jp[fa[u]][i]]+calc(w[u],w[jp[fa[u]][i]]));
                jp[u][i]=jp[fa[u]][i];
            }
            for(int i=0;i<=29;i++) if((w[u]>>i)&1) jp[u][i]=u;
        }
        for(int i=1;i<=n;i++) {
            dp[i]=(dp[i]+w[i])%mod;
            ans=(ans+dp[i]*i)%mod;
        }
        printf("%lld\n",ans);
    }
}
int main() {
    cin>>n>>str;
    if(str=="XOR") opt=1;
    else if(str=="OR") opt=2;
    else opt=3;
    for(int i=1;i<=n;i++) scanf("%lld",&w[i]);
    for(int i=2;i<=n;i++) scanf("%d",&fa[i]);
    if(opt==1||opt==2) sub1::solve();
    else sub2::solve();
 
}
```