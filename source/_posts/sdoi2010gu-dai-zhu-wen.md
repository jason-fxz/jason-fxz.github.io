---
title: ' [SDOI2010]古代猪文'
date: 2020-09-10 22:20:44
tags: [数论    ,CRT,Lucas]
categories:
- 题解
---
[[SDOI2010]古代猪文](https://www.luogu.com.cn/problem/P2480)

简述一下题意就是让你求
$$
g^{\sum_{k\mid n} \binom{n}{k} } \bmod 999911659
$$
如果 $g=999911659$ 显然答案等于 $0$ ，否则根据欧拉定理上式等价于：
$$
g^{\sum_{k\mid n} \binom{n}{k} \bmod999911658} \bmod 999911659 \\
$$
注：$\varphi(999911659)=999911658$  。

<!--more-->

然后发现 $999911658$ 不是素数，有些数不存在对其的逆元，而且这个模数太大，Lucas会TLE 。怎么办？

可以发现 $999911658=2\times3\times4679\times35617$ ，将模数拆成了四个小质数的乘积。

设$f(t)=\sum_{k\mid n} \binom{n}{k} \bmod t$ ，计算$f(999911658)$ 有点困难，但计算 $f(2),f(3),f(4679),f(35617)$ 可以使用Lucas可以在合理的时间内解决。有了 $f(2),f(3),f(4679),f(35617)$ 就可以对 $f(999911658)$ 进行限定。相当于现在需要求一个 $x$ （即 $f(999911658)$ ）满足如下同余方程：
$$
\left\{
\begin{matrix}
x &\equiv & f(2) & \pmod{2}\\
x &\equiv & f(3) & \pmod{3}\\
x &\equiv & f(4679) & \pmod{4679}\\
x &\equiv & f(35617) & \pmod{35617}
\end{matrix}
\right.
$$
这里的四个模数两两互质，故可以用CRT求解，最后算出快速幂算出 $g^x \bmod 999911659$ 即可。

## code

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll mod = 999911659;
ll fact[500001];
ll a[5],m[5]={0,2,3,4679,35617};
ll fpow(ll a,ll b,ll p){
    ll r=1;
    while(b) {
        if(b&1) r=(r*a)%p;
        a=(a*a)%p;b>>=1;
    }
    return r;
}
ll inv(ll a,ll b) {
    return fpow(a,b-2ll,b);
}
ll CRT() {
    ll M=1,x=0;
    for(int i=1;i<=4;i++) M*=m[i];
    for(int i=1;i<=4;i++) {
        (x+=(M/m[i]*inv(M/m[i],m[i])%M*a[i]%M))%=M;
    }
    return x;
}
ll C(ll n,ll m,ll p) {
    if(n<m) return 0;
    return fact[n]*inv(fact[m],p)%p*inv(fact[n-m],p)%p;
}
ll lucas(ll n,ll m,ll p) {
    if(n<m) return 0;
    if(!m) return 1;
    return C(n%p,m%p,p)*lucas(n/p,m/p,p)%p;
}
ll n,g;
int main() {
    fact[0]=1;
    cin>>n>>g;
    if(g==mod) return cout<<0<<endl,0;
    for(int j=1;j<=4;j++) {
        for(ll i=1;i<=m[j];i++) fact[i]=(fact[i-1]*i)%m[j];
        ll x=n;
        for(ll i=1;i*i<=x;i++)if(x%i==0) {
            a[j]+=lucas(n,i,m[j]);
            a[j]%=m[j];
            if(i*i!=x) {
                a[j]+=lucas(n,x/i,m[j]);
                a[j]%=m[j];
            }
        }
    }
    ll as=CRT();
    cout<<fpow(g,as,mod)<<endl;
}
```
