---
title: '『SDOI2018』旧试题'
date: 2021-04-28 22:09:41
tags: [反演,数论    ]
categories:
- 题解
---

## 题意

**[luogu - P4619 [SDOI2018]旧试题](https://www.luogu.com.cn/problem/P4619)**

$$
\sum_{i=1}^A\sum_{j=1}^B\sum_{k=1}^C \sigma_0(ijk)
$$

$\sigma_0(x)$ 表 $x$ 约数个数。

多测 $T\le 10$，$1\le \sum \max(A,B,C)\le 2\times 2\times 10^5$。

<!--more-->

## 题解

下文简记 $\gcd(i,j)=(i,j)$，$\operatorname{lcm}(i,j)=[i,j]$

~~显然有结论~~，就当作 $\sigma_0$ 的性质吧。

$$
\sigma_0(ijk)=\sum_{x|i}\sum_{y|j}\sum_{z|k} [(x,y)=1][(x,y)=1][(y,z)=1]
$$

代回原式，按照套路写成 $\epsilon$

$$
\sum_{i=1}^A\sum_{j=1}^B\sum_{k=1}^C \sum_{x|i}\sum_{y|j}\sum_{z|k} \epsilon((x,y))\epsilon((x,y))\epsilon((y,z))
$$

反演，

$$
\sum_{i=1}^A\sum_{x|i}\sum_{j=1}^B\sum_{y|j}\sum_{k=1}^C\sum_{k|z} \sum_{d_1|x,d_1|y} \mu(d_2)\sum_{d_2|x,d_2|z} \mu(d_2)\sum_{d_3|y,d_3|z} \mu(d_3)\\
$$

显然有 $d_1|i,d_1|j,d_2|i,d_2|k,d_3|j,d_3|k$，即 $[d_1,d_2]\mid i,\,[d_2,d_3]\mid j,\,[d_1,d_3]\mid k$。  
交换求和顺序，$i',j',k'$ 分别表示为 $[d_1,d_2],[d_2,d_3],[d_1,d_3]$ 的几倍。  
再考虑 $\sum_{x\mid i}$， 因为有 $x\mid i,\,d_1\mid x,\,d_2\mid x$，所以 $x$ 为 $[d_1,d_2]$ 的倍数，且为 $x$ 为 $i$ 的因数，这样的 $x$ 有 $\sigma_0(i/[d_1,d_2])=\sigma_0(i')$ 个。$y,z$ 同理。

$$
\sum_{d_1=1}^{\min(A,B)} \sum_{d_2=1}^{\min(A,C)} \sum_{d_3=1}^{\min(B,C)}\mu(d_2)\mu(d_1)\mu(d_3)
\sum_{i'=1}^{\left\lfloor A/[d_1,d_2]\right\rfloor} \sigma_0\left(i'\right)
\sum_{j'=1}^{\left\lfloor B/[d_1,d_3]\right\rfloor} \sigma_0\left(j'\right)
\sum_{k'=1}^{\left\lfloor C/[d_2,d_3]\right\rfloor} \sigma_0\left(k'\right)
$$

考虑：

$$
\begin{aligned}
f(n)=&\sum_{i=1}^{n} \sigma_0\left(i\right)\\
=&\sum_{i=1}^{n} \sum_{d=1}^{n} [d\mid n]\\
=&\sum_{d=1}^{n} \left\lfloor \frac{n}{d}\right\rfloor
\end{aligned}
$$

$f(n)$ 可以单次 $O(\sqrt{n})$ 求；考虑实际意义的话 $f(n)$ 为 除数函数（即约数个数）前缀和，可以 $O(n)$ 预处理。

代回原式：

$$
\sum_{d_1=1}^{\min(A,B)} \sum_{d_2=1}^{\min(A,C)}\sum_{d_3=1}^{\min(B,C)}\mu(d_1)\mu(d_2)\mu(d_3)\cdot
f\left(\left\lfloor \frac{A}{[d_1,d_2]}\right\rfloor\right)
f\left(\left\lfloor \frac{B}{[d_1,d_3]}\right\rfloor\right)
f\left(\left\lfloor \frac{C}{[d_2,d_3]}\right\rfloor\right)
$$

搞了半天还是 $O(n^3)$ 的复杂度，~~一分没有~~。

考虑到我们算的是有序对 $(d_1,d_2,d_3)$ 对答案的贡献，然后就 ~~莫名其妙~~ 地想到转化成图上**三元环计数**。具体的：

- 每个点有点权，记 $val_u=\mu(u)$；
- $(u,v)$ 之间连边，边权 $w_{u,v}=[u,v]$；
- 对于一个有序对 $(u,v,w)$ 若两两不同则可以表示成图上一个*三元环*；
- 一个三元环 $(u,v,w)$ 有贡献当且仅当 $u\le\min(A,B)\wedge v\le \min(A,C)\wedge w\le \min(B,C)$，且其贡献为 $\displaystyle val_uval_vval_w\cdot f\left(\left\lfloor \frac{A}{w_{u,v}}\right\rfloor\right)f\left(\left\lfloor \frac{B}{w_{u,w}}\right\rfloor\right)f\left(\left\lfloor \frac{C}{w_{v,w}}\right\rfloor\right)$。

考虑把一些没必要的点、边删掉，删除 $val_u=0$ 的点，和 $w_{u,v}>\max(A,B,C)$ 的边 $(u,v)$。

**建图**：  
枚举边权 $w$，边 $(u,v)$ 满足 $[u,v]=w,\mu(u)\ne 0,\mu(v)\ne 0$，$u,v$ 不能有平方因子，那么 $w$ 也不能有，对 $w$ 质因数分解，$w=p_1p_2\dots p_r$ 需要满足 $\forall p$ 有 $p\mid u$ 或 $p\mid v$。预处理因子，枚举子集即可。

实测 $A,B,C=10^5$ 时边数只有 $760741$。

三元环计数有 $O(m\sqrt{m})$ 的方法（$m$ 为边数，假设 $n,m$ 同阶）：

> 首先要对所有的无向边进行定向：对于任何一条边，从**度数大的点连向度数小**的点，如果度数相同，从编号小的点连向编号大的点。此时这张图是一个*有向无环图*。  
> 之后枚举每一个点 $u$，然后将 $u$ 的所有相邻的点都标记上 『被 $u$ 访问了』，然后再枚举 $u$ 的相邻的点 $v$，然后再枚举 $v$ 的相邻的点 $w$，如果 $w$ 存在『被 $u$ 访问了』的标记，那么 $(u,v,w)$ 就是一个三元环了。  
> 而且每个三元环只会被计算一次

---

{% spoiler "证明" %}

1. **证明重定向后是无环图**

   > 假设存在有向环 $p_1\to p_2\to \dots \to p_k\to p_1$，那么有 $\deg_1\ge\deg_2\ge\dots\ge\deg_k\ge\deg_1$，即 $\deg_1=\deg_2=\dots=\deg_k$。对于度数相同的点编号小的连大的，即 $p_1\le p_2\le \dots\le p_k\le p_1$ ，即 $p_1=p_2=\dots=p_k$ ，然而不同的点编号显然不等，故假设不存在，因此该图无环

2. **证明每个三元环之被算一次**

    > 每个三元环只可能是这样：$u\to v,u\to w,v\to w$（因为不存在有向环）。而这正好只会被 $u$ 统计一次贡献。

3. **证明时间复杂度为 $O(m\sqrt{m})$**

    > 首先遍历 $u$ 相邻的点并打标记相当于遍历所有边，复杂的是 $O(n+m)$ 的。  
    > 然后考虑 $u$ 遍历到 $v$，$v$ 再访问 $w$ 的复杂度，记 $out_v$ 表示 $v$ 的出边数量，转化为求 $\sum_{u\to v} out_v$  
    > 分讨 $out_v$ 大小：  
    >
    > 1. 若 $out_v<\sqrt{m}$ ：因为 $u\to v$ 有 $O(m)$ 个，这样的复杂度 $O(m\sqrt{m})$
    > 2. 若 $out_v\ge\sqrt{m}$ ：因为 $u\to v$，那么 $\deg_u\ge\deg_v\ge \sqrt{m}$。考虑图中 $\deg>\sqrt{m}$ 的点最多 $\sqrt{m}$ 个。考虑 $u$ 有 $O(\sqrt{m})$ 个，且 $u$ 一次枚举 $u\to v\to w$ 是 $O(m)$ 的（边不会重复枚举），这样的复杂也是 $O(m\sqrt{m})$  
    >
    > 故总复杂度 $O(m\sqrt{m})$

{% endspoiler %}

---

注意我们枚举到的三元环是**无序**三元组，而统计的是**有序**三元组，还要手动枚举 $6$ 种情况。

注意还有 $(d_1,d_2,d_3)$ 其中两个相同的情况，这是 $O(m)$ 的，还有三个都相等的情况这是 $O(n)$ 的。

## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef pair<int,int> pii;
const int N = 200010;
const int mod = 1e9 + 7;
int gcd(int a, int b) {
    return !b ? a : gcd(b, a % b);
}
int p[N], mu[N], c[N], tot, f[N]; // f[i] 约数个数前缀和
vector<int> divs[N];
vector<pii> e[N];
struct edge {int u, v, w;} E[2000000];
pii vis[N];
int deg[N];
int n, T, A, B, C, totE;
bool v[N];
void init(int n) {
    mu[1] = 1; f[1] = 1;
    for (int i = 2; i <= n; ++i) {
        if (!v[i]) p[++tot] = i, mu[i] = -1, c[i] = 1, f[i] = 2, divs[i].push_back(i);
        for (int j = 1; j <= tot && p[j] * i <= n; ++j) {
            v[i * p[j]] = 1;
            divs[i * p[j]] = divs[i];
            if (i % p[j] == 0) {
                mu[i * p[j]] = 0;
                c[i * p[j]] = c[i] + 1;
                f[i * p[j]] = f[i] / (c[i] + 1) * (c[i] + 2);
                break;
            }
            divs[i * p[j]].push_back(p[j]);
            mu[i * p[j]] = -mu[i];
            f[i * p[j]] = f[i] * 2;
            c[i * p[j]] = 1;
        }
    }
    for (int i = 1; i <= n; ++i) f[i] = (f[i] + f[i - 1]) % mod;
}
ll calc(int u, int v, int w, int uv, int uw, int vw) {
    if (u > min(A, B) || v > min(A, C) || w > min(B, C)) return 0;
    int t = mu[u] * mu[v] * mu[w];
    ll r = (ll)f[A / uv] * f[B / uw] % mod * f[C / vw] % mod; 
    return t == 1 ? r : mod - r;
}
void solve() {
    totE = 0;
    for (int i = 1; i <= n; ++i) e[i].clear(), deg[i] = 0, vis[i] = pii(0, 0);
    for (int k = 1; k <= n; ++k) if (mu[k] != 0) {
        int c = divs[k].size(), ful = (1 << c) - 1;
        vector<int> id(ful + 1, 0);
        for (int sub = 0; sub <= ful; ++sub) {
            int u = 1;
            for (int i = 0; i < c; ++i) if ((sub >> i) & 1) u *= divs[k][i];
            id[sub] = u;
        }
        for (int sub = 0; sub <= ful; ++sub) {
            for (int ssub = sub; true; ssub = (ssub - 1) & sub) {
                int u = id[sub], v = id[ssub ^ ful];
                if (u < v) {
                    E[++totE] = edge{u, v, k};
                    ++deg[u], ++deg[v];
                }
                if(ssub == 0) break;
            }
        } 
    }
    ll ans = 0;
    for (int i = 1; i <= totE; ++i) {
        int u = E[i].u, v = E[i].v, w = E[i].w;
        ans = (ans + calc(u, u, v, u, w, w)) % mod;
        ans = (ans + calc(u, v, u, w, u, w)) % mod;
        ans = (ans + calc(v, u, u, w, w, u)) % mod;
        ans = (ans + calc(v, v, u, v, w, w)) % mod;
        ans = (ans + calc(v, u, v, w, v, w)) % mod;
        ans = (ans + calc(u, v, v, w, w, v)) % mod;
    }
    for (int u = 1; u <= n; ++u) if (mu[u] != 0) {
        ans = (ans + calc(u, u, u, u, u, u)) % mod;
    }
    for (int i = 1; i <= totE; ++i) {
        if (deg[E[i].u] < deg[E[i].v] || (deg[E[i].u] == deg[E[i].v] && E[i].u < E[i].v))
            swap(E[i].u, E[i].v);
        e[E[i].u].push_back(pii(E[i].v, E[i].w));
    }
    for (int u = 1; u <= n; ++u) {
        for (int i = 0; i < (int)e[u].size(); ++i)
            vis[e[u][i].first] = pii(u, e[u][i].second);
        for (int i = 0, v; i < (int)e[u].size(); ++i) {
            v = e[u][i].first;    
            for (int j = 0, w; j < (int)e[v].size(); ++j) {
                w = e[v][j].first;
                if (vis[w].first != u) continue;
                int uv = e[u][i].second, vw = e[v][j].second, uw = vis[w].second;
                ans = (ans + calc(u, v, w, uv, uw, vw)) % mod;
                ans = (ans + calc(u, w, v, uw, uv, vw)) % mod;
                ans = (ans + calc(v, u, w, uv, vw, uw)) % mod;
                ans = (ans + calc(v, w, u, vw, uv, uw)) % mod;
                ans = (ans + calc(w, u, v, uw, vw, uv)) % mod;
                ans = (ans + calc(w, v, u, vw, uw, uv)) % mod;
            }
        }
    }
    cout << ans << endl;
}
void rmain() {
    cin >> A >> B >> C;
    n = max(max(A, B), C);
    solve();
}
int main() {
    init(N - 1);
    cin >> T;
    while (T--) rmain();
    return 0;
}
```
