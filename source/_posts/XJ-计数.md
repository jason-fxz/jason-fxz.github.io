---
title: 'XJ - 计数'
date: 2021-05-06 20:54:37
updated: 2021-05-06 20:54:37
tags: [XJ,dp优化,组合数学]
categories: 题解
hidden: true
---

## 题意

$$
\max(L(a)-x,0)= \sum_{i=x+1}^{n} [L(a)\ge i]
$$

考虑一个 DP 求最长相同子段小于 $L$ 的序列个数。

$$
dp_0=1
$$

$$
dp_i=(k-1)\sum_{j=1}^{L-1} dp_{i-j}
$$

这有点 naive，注意到放的第一段可以有 $k$ 种选择而不是 $k-1$，那么对答案 $\times\dfrac{k}{k-1}$ 修正一下即可。

前缀和优化 $f_n=\sum_{i=0}^n dp_i$。

$$
dp_i=(k-1)(f_{i-1}-f_{i-L})
$$

$$
\begin{aligned}
f_i&=f_{i-1}+dp_{i}\\
&=f_{i-1}+(k-1)(f_{i-1}-f_{i-L})\\
&=kf_{i-1}-(k-1)f_{i-L}
\end{aligned}
$$

到此我们有了 $O(n^2)$ 的做法，枚举 $L$ 的复杂度似乎是不能避免的，考虑优化 $f_n-f_{n-1}$ 的计算。

这是个经典模型：

> 考虑一个一般化的转移：
>
> $$
> f_0=1
> $$
>
> $$
> f_i=Af_{i-a}+Bf_{i-b}
> $$
>
> 其中 $A,B,a,b$ 都是常数，求 $f_n$。考虑其组合意义，从 $0$ 出发，每次可以走 $a$ 步或者 $b$ 步，其中走 $a$ 步有 $A$ 种方案，走 $b$ 步有 $B$ 种方案，求从 $0$ 到 $n$ 路径方案数。
>
> 枚举走 $a$ 步的次数 $i$，自然有：
>
> $$
> f_n=\sum_{i=1}^{n/a} [b\mid(n-ai)]\binom{i+(n-ai)/b}{i} A^{i}B^{(n-ai)/b}
> $$
>.

本题直接套用即可:

$$
f_n=\sum_{i=1}^{n/L} \binom{i+n-Li}{i} (1-k)^{i}k^{n-Li}
$$

由于是调和级数，复杂度是 $O(n\log n)$ 的。

## CODE

```cpp
#include <bits/stdc++.h>
template <typename Tp>
inline void read(Tp& x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar();}
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename... Args>
void read(Tp& t, Args& ...args) { read(t); read(args...); }
using namespace std;
typedef long long ll;
const int N = 1000010;
const int mod = 998244353;
int n, K, x;
template <typename Tp> void Add(Tp& x, Tp y) { x = (x + y) % mod; }
template <typename Tp> void Mul(Tp& x, Tp y) { x = (x * y) % mod; }
 
ll qpow(ll a, ll b = mod - 2) {
    ll r = 1; a = (a % mod + mod) % mod;
    for (; b; b >>= 1, a = a * a % mod) if (b & 1) r = r * a % mod;
    return r;
}
 
// ll calc0(int x) {
//     if (x == 1) return 0;
//     memset(dp, 0, sizeof(dp));
//     dp[0] = 1;
//     for (int i = 1; i <= n; ++i) {
//         dp[i] = dp[i - 1] * K - ((i - x < 0) ? 0 : dp[i - x]) * (K - 1);
//         dp[i] = (dp[i] % mod + mod) % mod;
//     }
//     return (dp[n] - dp[n - 1] + mod) % mod * qpow(K - 1) % mod * K % mod;
// }
ll inv[N], fac[N], ifac[N];
ll pwK[N], pwK1[N];
void init(int n) {
    inv[0] = inv[1] = fac[0] = fac[1] = ifac[0] = ifac[1] = 1;
    for (int i = 2; i <= n; ++i) fac[i] = fac[i - 1] * i % mod;
    for (int i = 2; i <= n; ++i) inv[i] = (mod - mod / i) * inv[mod % i] % mod;
    for (int i = 2; i <= n; ++i) ifac[i] = ifac[i - 1] * inv[i] % mod;
    pwK[0] = pwK1[0] = 1;
    for (int i = 1, _ = (1 - K + mod) % mod; i <= n; ++i) {
        pwK[i] = pwK[i - 1] * K % mod;
        pwK1[i] = pwK1[i - 1] * _ % mod;
    }
}
ll C(int n, int m) {
    return fac[n] * ifac[m] % mod * ifac[n - m] % mod;
} 
ll cal(int L, int n) {
    ll ret = 0;
    for (int i = 0, j; i * L <= n; ++i) {
        j = n - L * i;
        Add(ret, C(i + j, i) * pwK1[i] % mod * pwK[j] % mod);
    }
    return ret;
}
ll calc(int L) {
    static ll a = qpow(K - 1) * K % mod;
    if (L == 1) return 0;
    ll r = (cal(L, n) - cal(L, n - 1) + mod) % mod;
    return r * a % mod;
}
int main() {
    read(n, K, x); init(n);
    ll ans = 0;
    for (int L = x + 1; L <= n; ++L) {
        ll tmp = (qpow(K, n) - calc(L) + mod) % mod;
        Add(ans, tmp);
    }
    cout << ans << endl;
    return 0;
}
```
