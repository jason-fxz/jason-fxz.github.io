---
title: 'BSGS & EXBSGS'
date: 2020-09-07 22:29:36
tags: [数论    ,数学,BSGS]
categories: 📚算法笔记
mathjax: true
---

📘数论知识，高次同余方程求解，形如：$a^x=n\pmod{p}$ 。BSGS （Baby Steps Giant Steps）又名大步小步算法，又名~~北上广深~~ 。


<!--more-->
## 前置知识

* **逆元**，[数论基础-乘法逆元](/post/math1/#7-乘法逆元)
* 哈希表 或 `map`

## BSGS

对于如下的同余式，且满足 $a\perp p$ ，求 $x$ 
$$
a^x\equiv n\pmod{p}
$$

### 解法
首先一个显然的结论 $x\in[0,p-1]$ ，根据欧拉定理 $a^{\varphi(p)}\equiv 1\pmod{p}$ ，且 $\varphi(p)<p$，$x$ 大于 $p-1$ 后 $a^x$ 的取值肯定在之前就出现过，要得到最小的解只要考虑 $x\in[0,p-1]$ 即可。

令 $x=A\times\lceil \sqrt{p}\rceil-B\;|\;A,B\in \N$ ，对于给定 $x$ 且 $B\in\left[1,\lceil\sqrt{p}\rceil \right]$，一定能找到对应的 $A\in\left[1,\lceil\sqrt{p}\rceil \right]$ ，证明如下：

上式可转换为：
$$
x+B=A\times\lceil \sqrt{p}\rceil\\
$$
因为 $(x+B)$ 有 $\lceil \sqrt{p}\rceil$ 个连续的两两不同的取值 ，所以一定存在 $\left(x+B\right)\bmod \lceil \sqrt{p}\rceil=0$ ，而且 $(x+B)$ 一定不超过 $\lceil \sqrt{p}\rceil^2$ ，故存在 $A\in\left[1,\lceil\sqrt{p}\rceil\right]$ 使上式成立。
$$
\begin{matrix}
a^{A\times\lceil \sqrt{p}\rceil-B}& \equiv& n &\pmod{p}\\
a^{A\times\lceil \sqrt{p}\rceil} &\equiv &n\times a^{B} &\pmod{p}\\
\end{matrix}
$$
$a,n$ 已知，然后可以$O(\sqrt{p})$ 枚举 $B$ ，用哈希记录下每个 $n\times a^B$ 对应的 $B$ 值，再 $O(\sqrt{p})$ 枚举 $A$ ，再哈希表中是否有与 $a^{A\times\lceil \sqrt{p}\rceil}$ 相等的 $n\times a^B$ ，若有则可以得到 $x=A\times\lceil \sqrt{p}\rceil-B$ 。

总复杂是 $O(\sqrt{p})$ ，使用 `map` 会多一个 $\log$ 

### code

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
int gcd(ll a,ll b) {return !b?a:gcd(b,a%b);}
//a^x=n (mod p)
ll BSGS(ll a,ll n,ll p) {
    ll ans=p,t=ceil(sqrt(p)),dt=1;
    map<ll,ll> hash;
    for(ll B=1;B<=t;B++) dt=(dt*a)%p,hash[(dt*n)%p]=B;
    for(ll A=1,num=dt;A<=t;A++,num=(num*dt)%p) if(hash.find(num)!=hash.end()) ans=min(ans,A*t-hash[num]); 
    return ans;
}
ll a,p,n;
int main() {
    cin>>p>>a>>n;
    ll ans=BSGS(a,n,p);
    if(ans==p) printf("no solution\n");
    else printf("%lld\n",ans);
}
```


---

## EXBSGS

还是如下同余式，但是不保证 $a\perp p$
$$
a^x\equiv n\pmod{p}
$$

### 解法

不保证 $a\perp p$ 就有一个问题，$a$ 没有对 $p$ 的逆元，故无法使用普通的BSGS解决，于是乎我们想办法使$a,p$ 互质，令 $d_1=\gcd(a,p)$ ，若 $d_1\nmid n$ 则无解，否则得：
$$
\frac{a}{d_1}\cdot a^{x-1}\equiv\frac{n}{d_1} \pmod{\frac{p}{d_1}}
$$

此时，若 $a\not\perp p$，则继续令 $d_2=\gcd(a,\frac{p}{d_1})$ ，若 $d_2\nmid \frac{n}{d_1}$ 则无解，否则得：
$$
\frac{a^2}{d_1d_2}\cdot a^{x-2}\equiv\frac{n}{d_1d_2}\pmod{\frac{p}{d_1d_2}}
$$
以此类推，设 $D=\prod\limits_{i=1}^k d_i$ ，则：
$$
\frac{a^k}{D}\cdot a^{x-k}\equiv\frac{n}{D}\pmod{\frac{p}{D}}
$$
这里的 $k$ 是 $\log$ 级别的，因为每次 $p$ 至少小了一半。此时我们已经得到了 $a\perp \frac{p}{D}$ ，易证 $\frac{a^k}{D} \perp \frac{p}{D}$ ，可以算出 $\frac{a^k}{D}$ 的逆元，移到右边去：
$$
a^{x-k}\equiv\frac{n}{D} \cdot\left(\frac{a^k}{D}\right)^{-1}\pmod{\frac{p}{D}}
$$
剩下的就是一个平凡的BSGS，算出 $x-k$ ，答案再加上 $k$ 即可。

### code

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll gcd(ll a,ll b) {return !b?a:gcd(b,a%b);}
ll exgcd(ll a,ll b,ll &x,ll &y) {
    if(!b) {x=1,y=0;return a;}
    ll d=exgcd(b,a%b,x,y);
    swap(x,y);y-=a/b*x;
    return d;
}
ll inv(ll a,ll b) {
    ll x,y;exgcd(a,b,x,y);
    return (x%b+b)%b;
}
ll BSGS(ll a,ll n,ll p) {
    ll ans=p,t=ceil(sqrt(p)),dt=1;
    map<ll,ll> hash;
    for(ll B=1;B<=t;B++) dt=(dt*a)%p,hash[(dt*n)%p]=B;
    for(ll A=1,num=dt;A<=t;A++,num=(num*dt)%p) if(hash.find(num)!=hash.end()) ans=min(ans,A*t-hash[num]); 
    return ans;
}
//a^x=n (mod p)
ll EXBSGS(ll a,ll n,ll p) {
    int k=0;ll d,ad=1;
    while((d=gcd(a,p))!=1) {
        if(n%d) return -1;
        n/=d,p/=d;ad*=a/d;ad%=p;k++;
    }
    n=(n*inv(ad,p))%p;
    ll res=BSGS(a,n,p);
    if(res==p) return -1;
    return res+k; 
}
ll a,p,n;
int main() {
    while(cin>>a>>p>>n) {
        if(a==0&&p==0&&n==0) break;
        ll ans=EXBSGS(a,n,p);
        if(ans==-1) printf("No Solution\n");
        else printf("%lld\n",ans);
    }
}
```





 