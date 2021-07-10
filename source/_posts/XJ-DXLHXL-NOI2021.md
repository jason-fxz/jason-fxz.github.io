---
title: 'XJ NOI 模拟赛'
date: 2021-07-08 19:40:50
updated: 2021-07-08 19:40:50
tags: [XJ]
categories: 题解
# hidden: true
sticky:
description: XJ - NOI2021赛前训练题20 『2020~2021 年信息学多校联合训练 NOI2021 模拟赛』
top_img:
cover:
---

**[题面](./pro.pdf)**

## T1 究竟

鸽了

## T2 Rhuzerv

### 题意

阅读理解题，仔细看题面。

### 题解

先确定总格子数，枚举 $2$ 格子的个数 $p$，那么 $1$ 格子个数为 $n-2p$，对应权值为 $w_1^{n-2p}\cdot w_2^p$ ，$1,2$ 的排列方案为 $\dbinom{n-p}{p}$。再考虑黑白，黑格子数为 $s$，白格子数即为 $n-p-s$ ，且根据题意必然长这样：

$$
\blacksquare\blacksquare\blacksquare\square\blacksquare\blacksquare\cdots\blacksquare\square\blacksquare\blacksquare\square
$$

一段黑格子对应一个白格子。等价于有 $n-p-s$ 段，每段长度 $\ge 2$ 拼起来长度为 $n-p$ 。题目中所谓的选集合就是连续一段黑中选一个，白的反正选法唯一。等价于每段长度 $\ge 0$ 拼起来长度为 $L=2s-n+p$ ，有 $c=n-p-s$ 段，且长度为 $l$ 的段贡献为 $l+1$。

考虑生成函数 $F(x)=1+2x+3x^2+4x^3+\cdots$ ，简单观察发现 $F(x)$ 为函数 $G(x)=1+x+x^2+x^3+x^4\dots$  的导数：

$$
\begin{aligned}
F(x)&=G'(x)\\
&=\left(\frac{1}{1-x}\right)'\\
&=\frac{1}{(1-x)^2}
\end{aligned}
$$

求的是：

$$
\begin{aligned}{}
[x^{L}](F(x))^{c}&=[x^L]\left(\frac{1}{1-x}\right)^{2c}\\
&=[x^L](1+x+x^2+x^3+x^4+\cdots)^{2c}
\end{aligned}
$$

这挺好求的，等价于 $L$ 个相同球放 $2c$ 个不同盒子，盒子可以为空，隔板法得

$$
\binom{L+2c-1}{2c-1}
$$

那么答案：

$$
\sum_{p}w_1^{n-2p}w_2^p \binom{n-p}{p}\binom{n-p-1}{2n-2p-2s-1}
$$

### CODE

```cpp
#include <bits/stdc++.h>
template <typename Tp>
inline void read(Tp &x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar();}
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename... Args>
void read(Tp &t, Args &...args) { read(t); read(args...); }
using namespace std;
typedef long long ll;
const int N = 10000010;
const int mod = 998244353;
int n, w1, w2, s;
ll inv[N], fac[N], ifac[N];
ll pw1[N], pw2[N];
ll C(int n, int m) {
    if (n < m) return 0;
    return fac[n] * ifac[m] % mod * ifac[n - m] % mod;
}
int main() {
    read(n, w1, w2, s);
    inv[0] = inv[1] = fac[0] = fac[1] = ifac[0] = ifac[1] = 1;
    for (int i = 2; i <= n + 1; ++i) fac[i] = fac[i - 1] * i % mod;
    for (int i = 2; i <= n + 1; ++i) inv[i] = (mod - mod / i) * inv[mod % i] % mod;
    for (int i = 2; i <= n + 1; ++i) ifac[i] = ifac[i - 1] * inv[i] % mod;
    pw1[0] = pw2[0] = 1;
    for (int i = 1; i <= n; ++i) pw1[i] = pw1[i - 1] * w1 % mod, pw2[i] = pw2[i - 1] * w2 % mod;
    ll ans =  0;
    for (int p = 0; n - 2 * p >= 0 && n - p - s >= 1; ++p) {
        ll cnt = C(n - p, p) % mod * C(n - p - 1, 2 * n - 2 * p - 2 * s - 1) % mod;
        ans = (ans + pw1[n - 2 * p] * pw2[p] % mod * cnt % mod) % mod;
    }
    cout << ans << endl;
    return 0;
}
```

## T3 签door♂题

### 题意

给你一个 $2n$ 个点，连了 $n-1$ 条边的森林。你需要再连 $n$ 条边使其变成树，连边的方式如下：你给出一个 $1\dots 2n$ 的排列 $P$，连接 $P_i \leftrightarrow P_{2n-i+1}(i\in[1,n])$ ，让你给出合法的 $P$ 且字典序最小。

### 题解

我们直接钦定排列的前 $n$ 项为 $1\dots n$ （可以证明这样一定有解），然后依次考虑后面的位置填什么。令 $1\dots n$ 为黑点，$n+1\dots 2n$ 为白点，现在有 $n+1$ 个联通块，我们进行如下操作：从大到小依次考虑黑点 $x$，每次找到白点 $y$ 满足与黑点不联通且黑点白点至少有一个不孤立，将 $x,y$ 所在联通块合并，删除 $x,y$。

不能连接两个孤立的点因为要构成树，考虑为什么每次都能找到对应白点，考虑反证，无非两种情况：

1. 黑点孤立，白点都孤立，那么总联通块数至少 $1+n+1=n+2$ （$1$个黑点，$n$ 个白点，剩下所有黑点一个块内）比实际联通块数 $n+1$ 大，故不成立。
2. 黑点不孤立，但白点都与黑点联通，那么总联通块数至多 $1+n-1=n$ （$1$ 个黑点和所有白点，剩下 $n-1$ 个黑点都孤立）$<n+1$ 故也不成立。

得证。

至于如何实现，并查集 + set 维护最小白点 / 不独立最小白点 （同一联通块内若有多个白点只选最小的丢进 set），细节挺多。

### CODE

```cpp
#include <bits/stdc++.h>
template <typename Tp>
inline void read(Tp &x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar();}
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename... Args>
void read(Tp &t, Args &...args) { read(t); read(args...); }
using namespace std;
typedef long long ll;
const int N = 200010;
int far[N], n, col[N], tot;
set<int> B[N], W[N];
bool vs[N];
int find(int x) { return x == far[x] ? x : far[x] = find(far[x]); }
void merge_set(set<int> &S, set<int> &T) {
    if (S.size() < T.size()) swap(S, T);
    S.insert(T.begin(), T.end());
    T.clear();
}
bool merge(int x, int y) {
    x = find(x), y = find(y);
    if (x == y) return false;
    far[y] = x;
    merge_set(B[x], B[y]);
    merge_set(W[x], W[y]);
    return true;
}
set<int> allW, dW;
int main() {
    read(n);
    for (int i = 1; i <= 2 * n; ++i) {
        far[i] = i;
        if (i <= n) B[i].insert(i);
        else W[i].insert(i);
    }
    for (int i = 1; i < n; ++i) {
        int x, y; read(x, y);
        merge(x, y);
    }
    for (int i = 1; i <= n; ++i) printf("%d\n", i);
    for (int i = n + 1; i <= n * 2; ++i) {
        if (vs[find(i)]) continue;
        vs[find(i)] = 1;
        allW.insert(*W[find(i)].begin());
        if (W[find(i)].size() + B[find(i)].size() > 1) dW.insert(*W[find(i)].begin());
    }
    for (int i = n; i >= 2; --i) {
        int x = find(i);
        if (B[x].size() + W[x].size() > 1) {
            int j = *allW.begin();
            if (find(j) != x)
                allW.erase(allW.begin());
            else {
                j = *++allW.begin();
                allW.erase(++allW.begin());
            }
            printf("%d\n", j);
            int y = find(j);
            if (dW.count(j)) dW.erase(j);
            B[x].erase(i); W[y].erase(j);
            if (W[x].size()) allW.erase(*W[x].begin()), dW.erase(*W[x].begin());
            merge(x, y);
            if (!W[x].empty() && W[x].size() + B[x].size() > 1 && dW.count(*W[x].begin()) == 0) dW.insert(*W[x].begin());
            if (W[x].size() >= 1) allW.insert(*W[x].begin());
        } else {
            int j = *dW.begin(), y = find(j);
            dW.erase(dW.begin()); allW.erase(j);
            printf("%d\n", j);
            B[x].erase(i); W[y].erase(j);
            merge(x, y);
            if (!W[x].empty() && W[x].size() + B[x].size() > 1 && dW.count(*W[x].begin()) == 0) dW.insert(*W[x].begin());
            if (W[x].size() >= 1) allW.insert(*W[x].begin());
        }
    }
    printf("%d\n", *allW.begin());
    return 0;
}
```
