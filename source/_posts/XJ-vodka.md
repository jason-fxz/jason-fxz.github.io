---
title: XJ - vodka
date: 2021-04-02 21:34:17
tags: [生成函数,XJ]
categories: 题解
hidden: true
---


生成函数，期望

<!--more-->

### 题意

玩 $n$ 轮骰子，每次等概率从 $[1,K]$ 中掷出一个整数，记 $a_i$ 表示 $n$ 轮投掷中数值 $i$ 出现次数，求 $a_1^F\times a_2^F\times\dots\times a_L^F$ 的期望，答案对 $2003$ 取模。（若 $\bmod 2003$ 意义下不存在则输出 $0$ ）

$$
0\le n,K\le 10^9,0<F\le 10^3,L\le K 
$$

### 题解

根据题意列出：
$$
\mathbb{E}=\frac{1}{K^n}  \sum_{\sum {a} \;=n}\frac{n!}{\prod_{i=1}^K a_i!} \left(\prod_{i=1}^L a_i \right)^F\\
$$

其中 $\dfrac{1}{K^n}$ 指除以操作总数，右边是计算所有情况的总和，和 [CF891E - Lust](/2021/03/24/CF891E/) 类似 。

化简得到：

$$
\mathbb{E}=\frac{n!}{K^n}  \sum_{\sum {a} =n} \left(\prod_{i=1}^L \frac{a_i^F}{a_i!} \right)\cdot \left(\prod_{i=L+1}^K \frac{1}{a_i!}\right)
$$


考虑生成函数，整个 $\displaystyle \mathcal{F}(x)=\sum_{i} \frac{i^Fx^i}{i!}$  和 $\displaystyle\mathcal{G}(x)=\sum_{i} \frac{x^i}{i!}=e^x$

那么就是求：
$$
\begin{aligned}
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{i} \frac{i^Fx^i}{i!}\right)^L e^{(K-L)x}\\
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{i} \frac{x^i} {i!} \sum_{j=0}^{F}\begin{Bmatrix}F\\j\end{Bmatrix} i^{\underline{j}} \right)^L e^{(K-L)x}\\
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{j=0}^{F}\begin{Bmatrix}F\\j\end{Bmatrix} \sum_{i} \frac{x^i}{(i-j)!}  \right)^L e^{(K-L)x}\\
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{j=0}^{F}\begin{Bmatrix}F\\j\end{Bmatrix} \sum_{i} \frac{x^{i-j}x^j}{(i-j)!}  \right)^L e^{(K-L)x}\\
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{j=0}^{F}\begin{Bmatrix}F\\j\end{Bmatrix} e^x x^j  \right)^L e^{(K-L)x}\\
\mathbb{E}&=\frac{n!}{K^n} \left[x^n\right]\left(\sum_{j=0}^{F}\begin{Bmatrix}F\\j\end{Bmatrix} x^j  \right)^L e^{Kx}\\
\end{aligned}
$$


这个 $n$ 很大（~~还是一副不可做的样子~~），假设我们求出了 $b_i=[x^i]\left(\sum_{j=0}^F \begin{Bmatrix}F\\j\end{Bmatrix} x^j\right)^L$ ，那么：
$$
\mathbb{E}=\frac{n!}{K^n} \sum_{i=0}^n b_i \frac{K^{n-i}}{(n-i)!}=\sum_{i=0}^n b_i \frac{n^{\underline{i}}}{K^{i}}
$$
若 $n,i\ge 2003$ ，$n^{\underline{i}}$ 必然有 $2003$ 这个因子，答案必然 $0$ （不用担心 $K$ ，题目都说了「若答案在 $\bmod{2003}$ 下无意义输出 $0$」）。整个多项式~~快速~~龟速幂（其实暴力卷积就行了，复杂度 $O(\bmod^2 \log L)$ ）


### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 1005;
const int mod = 2003;
ll S2[N][N];
void init(int n) {
    S2[0][0]=1;
    for(int i=1;i<=n;++i) {
        for(int j=1;j<=i;++j) {
            S2[i][j]=(S2[i-1][j-1]+S2[i-1][j]*j)%mod;
        }
    }
}
int n,K,L,F;
struct poly{
    ll a[2005];
    poly() {memset(a,0,sizeof(a));}
    ll &operator[](const int &k) {return a[k];}
}f;
poly operator*(poly a,poly b) {
    poly r;
    for(int i=0;i<mod;++i) for(int j=0;j<=i;++j) r[i]=(r[i]+a[j]*b[i-j])%mod;
    return r;
}
poly operator^(poly a,int k) {
    poly r; r[0]=1;
    while(k) {
        if(k&1) r=r*a;
        a=a*a; k>>=1;
    }
    return r;
}
ll fpow(ll a,ll b=mod-2) {
    ll r=1;
    while(b) {
        if(b&1) r=r*a%mod;
        a=a*a%mod; b>>=1;
    }
    return r;
}
int main() {
    cin>>n>>K>>L>>F;
    if(K%mod==0) {
        puts("0");
        return 0;
    }
    init(F);
    for(int j=0;j<=F;++j){
        f[j]=S2[F][j];
    }
    f=f^L;
     
    ll iK=fpow(K);
    ll ans=0,tp=1;
    for(int i=0;i<mod;++i) {
        ans=(ans+tp*f[i])%mod;
        tp=tp*iK%mod*(n-i)%mod;
    }
    cout<<ans<<endl;
    return 0;
}
```