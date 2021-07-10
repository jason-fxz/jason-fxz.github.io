---
title: 'PGF 概率生成函数'
date: 2021-03-25 22:45:44
tags: [生成函数]
categories:
- 题解
---



在 OGF 普通生成函数的基础上，把系数换成随机变量取到某值的概率，具体的：
$$
{F}(x)=\sum_{i=0}^{\infty}P(X=i)x^i
$$

<!--more-->

其中 $X$ 表示随机变量，$P(X=i)$ 表示随机变量取 $i$ 的概率。

显然有以下结论：
$$
F(1)=1
$$
所有事件的概率和等于 $1$ ，没毛病。
$$
\mathbb{E}\left[X\right] =\sum_{i=0}^{\infty} iP(X=i)=F'(1)
$$

离散期望 $=$ 概率 $\times$ 数值，显然（$F'(1)$ 展开就是了）。
$$
\mathbb{E}\left[X^{\underline{k}}\right] =F^{(k)}(1)
$$

暴力展开易得，$F^{(k)}$ 表 $F$ 的 $k$ 阶导数。

## P4548 [CTSC2006]歌唱王国

[P4548 [CTSC2006]歌唱王国](https://www.luogu.com.cn/problem/P4548)

设 PGF $F(x),G(x)$ ，其中 $[x^k]F(x)$ 表示到 $k$ 位置正好结束的概率，$[x^k]G(x)$ 表示到 $k$ 位置没结束的概率。有以下等式：

$$
F(x)+G(x)=xG(x)+1\tag{1}
$$

含义为：当前没人获胜，再在后面加一个字符。

$$
G(x)\ast \left(\frac{1}{n}x\right)^m =\sum_{i=1}^{m} g_iF(x)\ast\left(\frac{1}{n}x\right)^{m-i} \tag{2}
$$

![PP1](.\PP1.png)

在黑串后加上红串，使其结束，但实际上已经结束了（下面的红串）。可以发现长度为 $len$ 的重复段是字符串的一个 $\mathrm{border}$ 。上面 $g_i=\left[\text{字符串中[1:i]是一个border}\right]$ 。

上面两个式子看起来一副不可做的样子，考虑 PGF 的性质，算期望显然是要算 $F'(1)$ 。

对 $(1)$ 求导，$x=1$ 带入：
$$
\begin{aligned}
F'(x)+G'(x)&=xG'(x)+G(x)\\
F'(x)&=(x-1)G'(x)+G(x)\\
F'(1)&=G(1)
\end{aligned}
$$
$x=1$ 带入 $(2)$：
$$
\begin{aligned}
G(1)\cdot \frac{1}{n^m} &=\sum_{i=1}^{m} g_iF(1)\frac{1}{n^{m-i}}\\
G(1) &=\sum_{i=1}^{m} g_in^i\\
\end{aligned}
$$
然后做完了。
$$
\mathbb{E}[X]=F'(1)=G(1)=\sum_{i=1}^m g_in^i
$$

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
const int N = 100010;
const int mod = 10000;
int n,m,t;
int a[N],g[N];
namespace strhash{
    const int p1 = 1019260817;
    const int bs = 792317;
    ll pw[N],hsh[N];
    void init(int n) {for(int i=1;i<=n;++i) hsh[i]=(hsh[i-1]*bs+a[i])%p1;}
    void pre(int n) {
        pw[0]=1;
        for(int i=1;i<=n;++i) pw[i]=pw[i-1]*bs%p1;
    }
    ll gethash(int l,int r) {
        return (hsh[r]-hsh[l-1]*pw[r-l+1]%p1+p1)%p1;
    }
};
using namespace strhash;
int main() {
    read(n,t);
    pre(N-1);
    for(int ts=1;ts<=t;++ts) {
        read(m);
        for(int i=1;i<=m;++i) read(a[i]);
        init(m);
        for(int i=1;i<=m;++i) {
            g[i]=(gethash(1,i)==gethash(m-i+1,m));
        }
        ll ans=0,pw=1; 
        for(int i=1;i<=m;++i) {
            pw=(pw*n)%mod;
            ans=(ans+g[i]*pw)%mod;
        }
        printf("%04lld\n",ans);
    }
    return 0;
}
```

## P3706 [SDOI2017]硬币游戏

[P3706 [SDOI2017]硬币游戏](https://www.luogu.com.cn/problem/P3706)

设 $F_i(x)$ 表示 $i$ 号人赢的 PGF，$G(x)$ 表示当前还没人获胜的 PGF。

$$
\sum_{i=1}^{n} F_i(1)=1\tag{1}
$$

根据定义，所有情况概率和为 $1$。还有， $F_i(1)$ 即第 $i$ 的人获胜的概率。

$$
xG(x)+1=\sum_{i=1}^{n} F_i(x)+G(x)\tag{2}
$$

含义为：当前没人获胜，再在后面加一个字符。

现在单独考虑一个人 $i$，仿照上一题，强制在没赢的状态下加上他的字符串。

$$
G(x)\ast\left(\frac{1}{2}x\right)^m
$$

这样会算多，如图:

![PP](.\PP.png)

在黑串后加上红串 $a$，实际已经构造出了绿串 $b$，此时 $a$ 串 $[1:len]$ 与 $b$ 串 $[m-len+1:m]$ 段相等，我们预处理出 $g_{a,b,k}$ 表示 $a$ 串的 $[1:k]$ 是否与 $b$ 串的 $[m-k+1:m]$ 相等。

$$
G(x)\ast\left(\frac{1}{2}x\right)^m=\sum_{k=1}^m \sum_{j=1}^{n} g_{i,j,k} F_j(x)\left(\frac{1}{2}x\right)^{m-k}\tag{3}
$$

按套路 $x=1$ 带入 $(2),(3)$: （你会发现 $(2)$ 代掉 $x$ 就成了 $(1)$ ，~~所以 $(2)$ 没用~~）

$$
\begin{aligned}
G(1)&=\sum_{k=1}^m\sum_{j=1}^n g_{i,j,k} F_j(1) 2^{k}\\
G(1)&=\sum_{j=1}^n\left(\sum_{k=1}^m 2^{k} g_{i,j,k}\right) F_j(1)
\end{aligned}
$$

当 $i\in[1,n]$ 再算上 $(1)$ 一共 $n+1$ 个方程。 $F_1(1),F_2(1),\dots,F_n(1),G(1)$ 一共 $n+1$ 个未知数，于是高斯消元。

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
typedef double db;
const int N = 305;
const int p1 = 1019260817;
const db eps = 1e-10;
const int bs = 37;
char s[N][N];
ll hsh[N][N],pw[N];
int n,m;
db a[N][N],pw2[N];
void guess(int n) {
    for(int j=1;j<=n;j++) {
        int mxp=j;
        for(int i=j+1;i<=n;i++)
            if(fabs(a[i][j])>fabs(a[mxp][j])) mxp=i;
        for(int i=1;i<=n+1;i++)
            swap(a[j][i],a[mxp][i]);
        for(int i=1;i<=n;i++) {
            if(i==j) continue;
            double tmp=a[i][j]/a[j][j];
            for(int k=1;k<=n+1;k++) {
                a[i][k]-=a[j][k]*tmp;
            }
        }
    }
    for(int i=1;i<n;i++) {
        printf("%.10lf\n",a[i][n+1]/a[i][i]);
    }
}

ll gethash(int i,int l,int r) {
    return (hsh[i][r]-hsh[i][l-1]*pw[r-l+1]%p1+p1)%p1;
}
int main() {
    read(n,m);
    for(int i=1;i<=n;++i) {
        scanf("%s",s[i]+1);
    }
    pw[0]=1; pw2[0]=1;
    for(int i=1;i<=m;++i) pw[i]=pw[i-1]*bs%p1;
    for(int i=1;i<=m;++i) pw2[i]=pw2[i-1]*2.0;
    for(int i=1;i<=n;++i) {
        for(int j=1;j<=m;++j) {
            hsh[i][j]=(hsh[i][j-1]*bs+s[i][j]-'a'+1)%p1;
        }
    }
    for(int i=1;i<=n;++i) {
        for(int j=1;j<=n;++j) {
            for(int k=1;k<=m;++k) {
                if(gethash(i,1,k)==gethash(j,m-k+1,m)) {
                    a[i][j]+=pw2[k];
                } 
            }
        }
        a[i][n+1]=-1;
    }
    for(int i=1;i<=n;++i) a[n+1][i]=1; a[n+1][n+2]=1;
    guess(n+1);
    return 0;
}
```
