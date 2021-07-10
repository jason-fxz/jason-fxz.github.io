---
title: 'XJ - 数数'
date: 2021-05-07 21:38:08
updated: 2021-05-07 21:38:08
tags: [反演,数论,XJ]
categories: 题解
hidden: true
---

## 题意

求：

$$
\sum_{i=1}^n\sum_{d|i}\mu(d)\sigma_0^2(\frac{i}{d})
$$

多测 $T\le 10$，$n\le 10^9$

## 题解

$$
\begin{aligned}
&\quad\sum_{i=1}^n\sum_{d|i}\mu(d)\sigma_0^2(\frac{i}{d})\\
&=\sum_{d=1}^n \mu(d)\sum_{i=1}^{\left\lfloor n/d\right\rfloor} \sigma_0^2(i)
\end{aligned}
$$

记 $f(n)=\sum_{i=1}^n \sigma_{0}^2(i)$。

$$
\sum_{d=1}^n \mu(d) f(\left\lfloor n/d\right\rfloor)
$$

数论分块，还要杜教筛求 $\mu$ 的前缀和。还要 $\sigma_0^2(i)$ 的前缀和？！显然我只会线性筛 $O(n)$。或者 Min_25 筛貌似可以搞。

---

整点阳间的，考虑 $\sum_{d|n}\mu(d)\sigma_{0}^2(\frac{n}{d})$：
$$
\begin{aligned}
&=\sum_{d|n}\mu(d)\sum_{xd|n}\sum_{yd|n}1\\
&=\sum_{X|n}\sum_{Y|n}\sum_{d|X,d|Y}\mu(d)\\
\end{aligned}
$$
注意到 $\sum_{d|X,d|Y}\mu(d)=[\gcd(X,Y)=1]$ 故我们可以认为 $X,Y$ 是互质的。

考虑带回：
$$
\begin{aligned}
&\quad\sum_{i=1}^n \sum_{X|i}\sum_{Y|i}\sum_{d|X,d|Y}\mu(d)
\\
&=\sum_{X=1}^n\sum_{Y=1}^n \sum_{d|X,d|Y}\mu(d)\left\lfloor \frac{n}{XY}\right\rfloor\\
&=\sum_{d=1}^n \mu(d)\sum_{d|X}\sum_{d|Y}\left\lfloor \frac{n}{XY}\right\rfloor\\
&=\sum_{d=1}^n \mu(d)\sum_{d^2|S} \left\lfloor \frac{n}{S}\right\rfloor\sum_{XY=S}[d\mid X\wedge d\mid Y]\\
&=\sum_{d=1}^n \mu(d)\sum_{d^2|S} \left\lfloor \frac{n}{S}\right\rfloor\sigma_0\left(\frac{S}{d^2}\right)\\
&=\sum_{d=1}^{\sqrt{n}} \mu(d)\sum_{S=1}^{\lfloor n/d^2\rfloor} \left\lfloor \frac{\left\lfloor n/d^2\right\rfloor}{S}\right\rfloor\sigma_0\left(S\right)\\
\end{aligned}
$$
令 $\displaystyle g(n)=\sum_{i=1}^n\left\lfloor \frac{n}{i}\right\rfloor\sigma_0(i)$ ，数论分块可求，考虑还要算 $\sigma_0(i)$ 前缀和。
$$
\sum_{i=1}^n \sigma_0(i)=\sum_{i=1}^n \sum_{d|i}1=\sum_{d=1}^n \left\lfloor \frac{n}{d}\right\rfloor
$$
数论分块 $O(\sqrt{n})$ 可解。算一次 $g(n)$ 根号套根号，复杂度？~~反正不大~~。

原式：
$$
\sum_{d=1}^{\sqrt{n}}\mu(d)g(\left\lfloor n/d^2\right\rfloor)
$$

复杂度？不会算，应该 $O(n^{2/3})$ 左右吧。

## CODE

```cpp
#pragma GCC optimize(2)
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
 
int p[N], tot, mu[N], d[N], ct[N];
int T, n;
bool v[N];
 
void init(int n) {
    for (int i = 2; i <= n; ++i) {
        if (!v[i]) p[++tot] = i, mu[i] = -1, ct[i] = 1, d[i] = 2;
        for (int j = 1; j <= tot && p[j] * i <= n; ++j) {
            v[i * p[j]] = 1;
            if (i % p[j] == 0) {
                mu[i * p[j]] = 0;
                d[i * p[j]] = d[i] / (ct[i] + 1) * (ct[i] + 2);
                ct[i * p[j]] = ct[i] + 1;
                break;
            }
            mu[i * p[j]] = -mu[i];
            d[i * p[j]] = d[i] * 2;
            ct[i * p[j]] = 1;
        }
    }
    mu[1] = 1; d[1] = 1;
    for (int i = 2; i <= n; ++i) d[i] += d[i - 1];
}
ll sig0(int n) {
    if (n < N) return d[n];
    ll ret  = 0;
    for (int l = 1, r; l <= n; l = r + 1) {
        r = n / (n / l);
        ret += (ll)(n / l) * (r - l + 1);
    }
    return ret;
}
ll g(int n) {
    ll ret = 0;
    for (int l = 1, r; l <= n; l = r + 1) {
        r = n / (n / l);
        ret += (sig0(r) - sig0(l - 1)) * (n / l);
    }
    return ret;
}
ll calc(int n) {
    ll ret = 0;
    for (int d = 1; d * d <= n; ++d) if (mu[d]) {
        ret += g(n / d / d) * mu[d];
    }
    return ret;
}
int main() {
    init(N - 1);
    cin >> T;
    while (T--) {
        cin >> n;
        cout << calc(n) << endl;
    }
    return 0;
}
```
