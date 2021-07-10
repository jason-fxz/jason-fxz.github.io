---
title: 'XJ - 拆分'
date: 2021-05-04 22:17:59
updated: 2021-05-05 22:17:59
tags: [XJ,动态规划,组合数学]
categories: 题解
hidden: true
---

## 题意

现有一整数 $n$，求满足以下条件的整数序列个数：

- $a_1+a_2+\dots+a_m=n$
- $0<a_1\le a_2\le\dots\le a_m$
- $\forall i\ne j:|a_i-a_j|\ge k$
- $h_{m\bmod p} =1$

给出 $n,k,p,\{h_i\}$ 其中 $1\le n\le 10^6$，$1\le p \le k\le n$。

然后 $k=0$ 且 $n>5000$ 时 $p=1$。

## 题解

话说要是 $k=0,p\ne 1,n\le 10^6$ 那这题貌似没法做，所以这数据范围出的可真是……  

### k=0,p=1

要求**拆分数**。$n$ 很大，$O(n^2)$ dp不现实，需要用到**五边形数**。

五边形数：$\dfrac{i(3n-1)}{2}$。

广义五边形数：即 $i$ 取 $0,1,-1,2,-2,3,-3,\dots$ 前几项为 $0,1,2,5,7,12,15$。

欧拉函数（不是数论那个）：

$$
\phi(x)=\prod_{i=1}^{\infty} (1-x^i)
$$

五边形数定理：

$$
\phi(x)=\sum_{i=-\infty}^{+\infty} (-1)^i x^{i(3i-1)/2}
$$

因为用到广义五边形数所以 $i$ 会取负数。其实就是把这玩意展开，某些项会被消掉：

$$
(1-x)(1-x^2)(1-x^3)(1-x^4)\cdots=1-x-x^2+x^5+x^7-x^{12}-x^{15}\cdots
$$

与拆分数的关系：$p(k)$ 表示正整数 $k$ 的拆分数，那么有

$$
\frac{1}{\phi(x)}=\sum_{k=0}^{\infty} p(k)x^k
$$

即

$$
p(x)\phi(x)=1
$$

那么我们可以用 $O(n\log n)$ 的时间对 $\phi(x)$ 多项式求逆，得到 $1\sim n$ 的拆分数。

### k>0

枚举拆分出来的个数为 $m$，即：

$$
a_1+a_2+\dots+a_m=n \quad(a_i+k\le a_{i+1})
$$

考虑差分 $a_i'=a_i-a_{i-1}-k$，此时

$$
a_j = (j-1)k+\sum_{i=1}^j a_i'
$$

总和为：

$$
\begin{aligned}
\sum_{j=1}^m a_j &= \sum_{j=1}^m (j-1)k+\sum_{j=1}^{m} \sum_{i=1}^j a_i'\\
n - \frac{m(m-1)k}{2}&= (a_1)+(a_1+a_2)+\dots+(a_1+a_2+\dots+a_m)
\end{aligned}
$$

右边 $(a_1),(a_1+a_2),\dots$ 这些东西单调不减，且都大于零，相当于 $n-\dfrac{m(m-1)k}{2}$ 的拆成 $m$ 个的拆分数，等价于用不超过 $m$ 个数拆分 $n-\dfrac{m(m-1)k}{2}-m$ 的方案数。

考虑用不超过 $m$ 个数拆分 $n$，等价于 $n$ 个无标号小球放入 $m$ 个无标号盒子，盒子可空的方案数。这显然是有 $O(nm)$ 的 DP 做法，$k>0$ 时 $m$ 是 $O(\sqrt{n})$ 级别的，滚动数组优化一下空间即可，复杂度 $O(n\sqrt{n})$。

## CODE

```cpp
#pragma GCC optimize(2)
#pragma GCC optimize(3)
#pragma GCC optimize(3,"Ofast","inline")
#include <bits/stdc++.h>
#define clr(f, n) memset(f, 0, sizeof(long long) * (n))
#define cpy(f, g, n) memcpy(f, g, sizeof(long long) * (n))
template <typename Tp>
inline void read(Tp& x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar(); }
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename... Args>
void read(Tp& t, Args& ...args) { read(t); read(args...); }
using namespace std;
typedef long long ll;
const int N = 1000010;
const int mod = 998244353;
int n, K, p, h[N];
template <typename Tp> void umin(Tp& x, Tp y) { if (x > y) x = y; }
template <typename Tp> void umax(Tp& x, Tp y) { if (x < y) x = y; }
template <typename Tp> void Add(Tp& x, Tp y) { x = (x + y) % mod;}

namespace sub1 { // O(n^{1.5})
    int dp[2][N]; // dp[i][j] i 个盒子，j 个球，都无标号，可以为空

    void solve() {
        dp[0][0] = 1;
        bool V = 0;
        int ans = 0;
        for (int i = 1; ; ++i) {
            int tp = n - i * (i - 1) / 2 * K;
            if (tp < 0) break;
            dp[V ^ 1][0] = 1;
            for (int j = 1; j <= tp - i + 1; ++j) {
                if (i != 1) {
                    if (j < i) dp[V ^ 1][j] = dp[V][j];
                    else dp[V ^ 1][j] = (dp[V][j] + dp[V ^ 1][j - i]) % mod;
                }
                else dp[V ^ 1][j] = 1;
            }
            V ^= 1;
            // tp 个球 -> i 个盒子
            if (h[i % p]) {
                // printf("%d -> %d : %d\n", tp, i, dp[V][tp - i]);
                Add(ans, dp[V][tp - i]);
            }
        }
        printf("%d\n", ans);

    }
}

namespace sub3 {  // K = 0 P != 1 O(n^2)
    const int N = 5005;
    int f[N][N];
    // n 个球 m 个盒子，不为空  都无序
    void solve() {
        for (int i = 0; i <= n; ++i) f[0][i] = 1;
        for (int i = 0; i <= n; ++i) f[i][1] = 1;

        for (int i = 1; i <= n; ++i) {
            for (int j = 2; j <= n; ++j) {
                if (j > i) f[i][j] = f[i][i];
                else f[i][j] = (f[i - j][j] + f[i][j - 1]) % mod;
            }
        }
        int ans = 0;
        for (int i = 1; i <= n; ++i) if (h[i % p]) {
            // n 个球 i 个盒
            Add(ans, f[n - i][i]);
        }
        printf("%d\n", ans);
    }
}
ll qpow(ll a, ll b = mod - 2) {
    ll r = 1;
    for (; b; b >>= 1, a = a * a % mod) if (b & 1) r = r * a % mod;
    return r;
}
inline ll Mod(ll x) {
    return x < mod ? x : x % mod;
}
namespace sub2 { // K = 0, P = 1
    const int N = 1 << 23;
    const int _g = 3;
    const int _ig = qpow(3);
    int rev[N], rev_n;
    void prerev(int n) {
        if (n == rev_n) return;
        rev_n = n;
        for (int i = 0;i < n;++i) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) ? (n >> 1) : 0);
    }
    // NTT : fg=1 DFT fg=-1 IDFT
    void NTT(ll f[], int n, int fg) {
        prerev(n);
        for (int i = 0;i < n;++i) if (i < rev[i]) swap(f[i], f[rev[i]]);
        for (int h = 2;h <= n;h <<= 1) {
            ll Dt = qpow((fg == 1) ? _g : _ig, (mod - 1) / h), w;
            int len = h >> 1;
            for (int j = 0;j < n;j += h) {
                w = 1;
                for (int k = j;k < j + len;++k) {
                    ll tmp = Mod(f[k + len] * w);
                    f[k + len] = f[k] - tmp; (f[k + len] < 0) && (f[k + len] += mod);
                    f[k] = f[k] + tmp; (f[k] >= mod) && (f[k] -= mod);
                    w = Mod(w * Dt);
                }
            }
        }
        if (fg == -1) {
            ll invn = qpow(n, mod - 2);
            for (int i = 0;i < n;++i) f[i] = Mod(f[i] * invn);
        }
    }
    void p_inv(ll g[], int n, ll f[]) {
        static ll sav[N];
        int nn = 1 << (int)ceil(log2(n));
        clr(f, n * 2);
        f[0] = qpow(g[0], mod - 2);
        for (int h = 2; h <= nn; h <<= 1) {
            cpy(sav, g, h); clr(sav + h, h);
            NTT(sav, h << 1, 1); NTT(f, h << 1, 1);
            for (int i = 0;i < (h << 1);++i) {
                f[i] = f[i] * (2ll - f[i] * sav[i] % mod + mod) % mod;
            }
            NTT(f, h << 1, -1); clr(f + h, h);
        }
        clr(f + n, nn * 2 - n);
    }
    ll f[N], g[N];
    void solve() {
        int nn = 1 << (int)ceil(log2(n + 1));
        int i = 0;
        while (true) {
            int d = (ll)i * (3 * i - 1) / 2;
            if (d >= nn) break;
            f[d] = (i & 1) ? -1 : 1;
            if (i <= 0) {
                i = -i + 1;
            }
            else i = -i;
        }
        p_inv(f, nn, g);
        printf("%d\n", g[n]);
    }
}
int main() {
    read(n, K, p);
    for (int i = 0; i < p; ++i) read(h[i]);
    if (K == 0) {
        if (p != 1) sub3::solve();
        else sub2::solve();
    }
    else sub1::solve();
    return 0;
}
```
