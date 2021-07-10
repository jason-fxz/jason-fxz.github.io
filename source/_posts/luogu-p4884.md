---
title: '[luogu-P4884]多少个1？'
date: 2020-09-13 22:25:29
tags: [BSGS,数论    ]
categories:
- 题解
---

[多少个1？](https://www.luogu.com.cn/problem/P4884)
BSGS，数论

一句话题意： 满足  $\underbrace{111\dots11}_n \equiv K \pmod{m}$ ，给你 $K,m$ ，求符合条件 $n$ 的最小值。

<!--more-->

$30 \%$ 的数据保证 $m\leq 10^6$。
$60 \%$ 的数据保证 $m\leq 5\times 10^7$。
$100 \%$ 的数据保证 $m\leq 10^{11}\;,\;0<K<m$ 。

## 60pts

这档分应该很好拿，原式可以拆成这样：
$$
10^0+10^1+10^2+\cdots +10^{n-1} \equiv K \pmod{m}
$$
又因 $10 \perp m$ ，$10^x \bmod m$ 在 $x\in[0,m-2]$ 正好分别对应 $m-1$ 种取值，发现 $\sum_{i=1}^{m-1}i=\frac{(1+m-1)\times(m-1)}{2}=m\cdot\frac{m-1}{2}$ ，能被 $m$ 整除（$m-1$是偶数），即 $\underbrace{111\dots11}_{m-1} \equiv 0 \pmod{m}$  ，故我们只要暴力枚举 $n$ 即可，复杂度 $\mathcal{O}(m)$ 。

## 100pts

$\underbrace{111\dots11}_n$ 这东西看着很难受，捣鼓以下可以的得到：

$$
\underbrace{111\dots11}_n=\frac{10^n-1}{9}
$$

那么：

$$
\begin{matrix}
& \underbrace{111\dots11}_n \equiv K \pmod{m}\\
\iff& \cfrac{10^n-1}{9} \equiv K \pmod{m} \\
\iff& 10^n \equiv 9K+1 \pmod{m} \\
\end{matrix}
$$

$m$ 是质数，那就跑一发BSGS即可，复杂度 $\mathcal{O}(\sqrt{m})$ 。

还有一点要注意，$m\leq10^{11}$ 直接乘 `long long` ，注意使用快速乘。由于 $m$ 比较小，这里提供一种 $\mathcal{O}(1)$ 的快速乘（原理是乘法分配律，自行体会）：

```cpp
ll mul(ll a, ll b, ll P){
    ll L = a * (b >> 25LL) % P * (1LL << 25) % P;
    ll R = a * (b & ((1LL << 25) - 1)) % P;
    return (L + R) % P;
}
```

## code

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
ll m, K, n;
ll mul(ll a, ll b, ll P)
{
    ll L = a * (b >> 25LL) % P * (1LL << 25) % P;
    ll R = a * (b & ((1LL << 25) - 1)) % P;
    return (L + R) % P;
}
ll gcd(ll a, ll b) { return !b ? a : gcd(b, a % b); }
ll BSGS(ll a, ll n, ll p)
{
    if (a % p == 0 && n != 0)
        return p;
    ll ans = p, t = ceil(sqrt(p)), dt = 1;
    map<ll, ll> hash;
    for (ll B = 1; B <= t; B++)
        dt = mul(dt, a, p), hash[mul(dt, n, p)] = B;
    for (ll A = 1, num = dt; A <= t; A++, num = mul(num, dt, p))
        if (hash.find(num) != hash.end())
            ans = min(ans, A * t - hash[num]);
    return ans;
}
int main()
{
    cin >> K >> m;
    K = (K * 9ll + 1ll) % m;
    ll r = BSGS(10, K, m);
    cout << r << endl;
}
```
