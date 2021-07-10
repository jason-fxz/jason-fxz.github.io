---
title: 'XJ NOI 模拟赛'
date: 2021-07-07 20:40:30
updated: 2021-07-08 14:18:30
tags: [XJ]
categories: 题解
# hidden: true
sticky:
description: XJ - NOI2021赛前训练题19 『第 1 届长沙市一中全国赛模拟赛 HNFMS NOI 2021』
top_img:
cover:
---

**[题面](./pro.pdf)**

## 合成小丹

### 题意

给你 $n$ 个整数数，范围在 $[0,2^w-1]$，执行如下两种操作共 $n-1$：

1. 选择两数 $x,y$，删除它们并加入 $\left\lfloor \dfrac{x\mid y}{2}\right\rfloor$ （ $|$ 表示按位或 ）；
2. 选择一个数 $x$ 并删去。

你需要**最小化**最后剩下的那个数。

多测 $T\le 10$，$1\le n\le 10^5, 0\le w\le 60$ 。

### 题解

操作一看出 $x,y$ 按为或后右移一位。操作二就把最小的留下，不用的删掉。

**特例**：假设所以数都为 $2^k-1(k\in \Z)$，那么我们直接用 $k$ 代表数 $2^k-1$。两个相同的数 $x$ 可以弄出一个 $x-1$，最后取最小的数，这个直接模拟即可。

考虑原题，从上面的方法可以得到启发，先假设 $ans=(111\dots11)_2$，然后我们从高位到低位逐位判断能否将 $1$ 改成 $0$。  
现在枚举了一个 $ans$，判断是否可行等价于能否用 $n$ 根数中的一些整出一个 $X$ 是 $ans$ 的子集（ 即 $X \mid ans=ans$ ）。考虑合并操作形成的树：

<img src="./P1.png" width=55%></img>
<!-- ![P1](./P1.png) -->

事实上 $X = (a_1 \gg d_1) \mid (a_2\gg d_2)\mid\dots\mid(a_5\gg d_2)$ 其中 $d_i$ 表节点深度，$\gg$ 表位移。

$X$ 是 $ans$ 的子集，等价于选择的每个 $a_i$ 右移若干位是 $ans$ 的子集。我们要判断能否整出这个树，而位移位数等于其在树上的深度，固然深度越小的点越多越容易整出树。故对于每个 $a_i$ 找到最小位移 $m_i$，作为点的深度，然后参照特例的方法模拟即可。

这样的复杂度是 $O(nw^2)$ 的，缺点在于每次要花费 $O(w)$ 的时间找 $a_i$ 的最小位移。我们记 $ok_i$ 表示当前 $a_i$ 的合法位移情况，二进制下第 $j$ 表右移 $j$ 是否合法。每次我们只将 $ans$ 的某一位从 $1$ 改 $0$，也就是将 $a_i$ 右移之后这位为 $1$ 的移动全部改不合法。可以 $O(1)$ 实现：`ok[i] &= ~(a[i] >> d);` 其中 $d$ 表当前枚举 $ans$ 的第 $d$ 位。

总复杂度 $O(nw)$。

### CODE

```cpp
#pragma GCC optimize(2)
#include <bits/stdc++.h>
template <typename Tp>
inline void read(Tp &x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar();}
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename ...Args>
void read(Tp &t, Args &...args) { read(t); read(args...); }
using namespace std;
typedef long long ll;
const int N = 100010;
int T, n, w;
ll a[N], ok[N], bak[N];
int ct[64];

void rmain() {
    read(n, w);
    for (int i = 1; i <= n; ++i) read(a[i]), ok[i] = (1ll << (w + 1)) - 1;
    ll ans = (1ll << w) - 1;
    for (int d = w - 1; d >= 0; --d) {
        ans ^= (1ll << d);
        memset(ct, 0, sizeof(ct));
        memcpy(bak, ok, sizeof(bak));
        for (int i = 1; i <= n; ++i) {
            ok[i] &= ~(a[i] >> d);
            ct[(int)log2(ok[i] & -ok[i])]++;
        }
        for (int i = w; i >= 1; --i) ct[i - 1] += ct[i] / 2;
        if (!ct[0]) {
            ans ^= (1ll << d);
            memcpy(ok, bak, sizeof(ok));
        }
    }
    cout << ans << endl;
}

int main() {
    read(T);
    while (T--) rmain();
    return 0;
}
```

## 路过中丹

### 题意

懒。

### 题解

发现区间 $[l,r]$ 若存在形如 $x\_x$ 这种子串的，一遍就能走完，这个特判前缀和预处理下就能完成。

剩下的就是用若干个偶回文串覆盖掉整段区间。
考虑那些点无法被覆盖到。首先可以找到以位置 $i$ 为右端点/左端点 最小的偶回文长度（可以先 二分+哈希/manacher 找到每个点为回文中心的最长回文半径，再用栈易得上面这玩意）。

那么对于一个点 $i$ 我们得到了覆盖它的最小偶回文的左端点 $L_i(L_i<i)$ 和右端点 $R_i(R_i>i)$ 。但且仅当询问 区间 $[l,r]$ 满足 $L_i<l\le i\le r<R_i$ 时 $i$ 点不能被覆盖，该询问失败。

这显然可以扫描线。

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
const int N = 1000010;
int n, q, L[N], R[N], cnt[N], len[N];
char s[N], ans[N];
namespace strhash {
    typedef pair<ll, ll> pll;
    const ll p1 = 1019260817;
    const ll p2 = 1019260819;
    const ll bs = 517;
    pll operator+(const pll a, const pll b) { return pll((a.first + b.first) % p1, (a.second + b.second) % p2); }
    pll operator-(const pll a, const pll b) { return pll((a.first - b.first + p1) % p1, (a.second - b.second + p2) % p2); }
    pll operator*(const pll a, const pll b) { return pll((a.first * b.first) % p1, (a.second * b.second) % p2); }
    pll pw[N], f[N], b[N];
    void init() {
        pw[0] = pll(1, 1);
        for (int i = 1; i <= n; ++i) pw[i] = pw[i - 1] * pll(bs, bs);
        for (int i = 1; i <= n; ++i) f[i] = f[i - 1] * pll(bs, bs) + pll(s[i] - 'a' + 1, s[i] - 'a' + 1);
        for (int i = n; i >= 1; --i) b[i] = b[i + 1] * pll(bs, bs) + pll(s[i] - 'a' + 1, s[i] - 'a' + 1);
    }
    pll getf(int l, int r) { return f[r] - f[l - 1] * pw[r - l + 1]; }
    pll getb(int l, int r) { return b[l] - b[r + 1] * pw[r - l + 1]; }
}
struct que {
    int l, id;
};
vector<que> qu[N];
vector<int> qR[N];

struct fenwick {
    int t[N];
    void add(int x, int v) { for (; x <= n; x += x & - x) t[x] += v; }
    int sum(int x) { int r = 0; for (; x > 0; x -= x & -x) r += t[x]; return r; }
    void modify(int l, int r, int v) { add(l, v); add(r + 1, -v); }
    int ask(int x) { return sum(x); }
} T;

int main() {
    read(n);
    scanf("%s", s + 1);
    strhash::init();
    read(q);
    for (int i = 1; i <= n; ++i) {
        if (i > 1 && i < n && s[i - 1] == s[i + 1]) cnt[i]++;
        cnt[i] += cnt[i - 1];
    }
    for (int i = 1; i < n; ++i) {
        int L = 1, R = min(i, n - i), M;
        while (L <= R) {
            M = (L + R) >> 1;
            if (strhash::getf(i + 1, i + M) == strhash::getb(i - M + 1, i)) {
                L = M + 1;
                len[i] = max(len[i], M);
            } else R = M - 1;
        }
    }
    vector<int> stk;
    for (int i = 1; i <= n; ++i) {
        while (!stk.empty() && stk.back() + len[stk.back()] < i) stk.pop_back();
        if (!stk.empty()) L[i] = 2 * stk.back() - i + 1;
        if (len[i]) stk.push_back(i);
    }
    while (!stk.empty()) stk.pop_back();
    for (int i = n; i >= 1; --i) {
        if (len[i]) stk.push_back(i);
        while (!stk.empty() && stk.back() - len[stk.back()] + 1 > i) stk.pop_back();
        if (!stk.empty())
            R[i] = 2 * stk.back() - i + 1;
        else R[i] = n + 1;
    }
    while (!stk.empty()) stk.pop_back();
    for (int i = 1; i <= q; ++i) {
        int l, r; read(l, r);
        if (cnt[l] < cnt[r - 1]) ans[i] = '1';
        else qu[r].push_back({ l, i });
    }
    // 扫描线
    for (int x = 1; x <= n; ++x) qR[R[x]].push_back(x);
    for (int x = 1; x <= n; ++x) {
        for (auto &id : qR[x])
            T.modify(L[id] + 1, id, -1);
        T.modify(L[x] + 1, x, 1);
        for (auto qq : qu[x]) {
            if (T.ask(qq.l) == 0)
                ans[qq.id] = '1';
            else
                ans[qq.id] = '0';
        }
    }
    ans[q + 1] = '\0';
    printf("%s\n", ans + 1);
    return 0;
}
```

## 膜拜大丹

### 题意

略.

### 题解

首先发现都是偶环，且只要找二元环就行了（因为多元环必然可以连出一个二元环，从而又空出了几个点，必然更优）。

能形成二元环的点间连边，其实就是跑二分图最大匹配，但一个点可以被匹配多次，naive 的匹配必然不行。考虑改图的性质

考虑左边的点 $i$ 和右边的点 $j$ 。能形成二元环条件是 $i\le b_j\wedge a_i\ge j$ 。不如看出平面上的点，左边的点 $(i,a_i)$ 为黑点，右边的点 $(b_j,j)$ 为白点，平面上当且仅当白点在黑点右下方则可以配。

**从右往左**扫瞄线，遇到白点加入 `set`（从上到下）遇到黑点**从上到下**匹配白点。这样的匹配显然是对的，它不会被撤销，考虑倘若被撤销一定找到了一条交错增广路。

![P2](./P2.png)

不难发现，增广的过程中最后两个点若能匹配但未匹配，则与我们的策略矛盾，所以不可能出现这样的增广路。我们按照该策略匹配是最优的做法。

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
const int N = 500010;
int n, m, a[N], b[N], c[N], d[N];
struct pp {
    int x, y; mutable int cnt;
} p[N << 1]; int tot;
struct cmp {
    bool operator()(const pp &a, const pp &b) {
        return a.y == b.y ? a.x > b.x : a.y < b.y;
    }
};
int main() {
    read(n, m);
    for (int i = 1; i <= n; ++i) read(a[i]);
    for (int i = 1; i <= m; ++i) read(b[i]);
    for (int i = 1; i <= n; ++i) read(c[i]);
    for (int i = 1; i <= m; ++i) read(d[i]);
    for (int i = 1; i <= n; ++i) p[++tot] = { i, a[i], c[i] };
    for (int i = 1; i <= m; ++i) p[++tot] = { b[i], i, -d[i] };
    sort(p + 1, p + tot + 1, [](pp a, pp b) -> bool {
        return a.x == b.x ? (a.y == b.y ? a.cnt < b.cnt : a.y < b.y) : a.x > b.x;
    });
    ll ans = 0;
    multiset<pp, cmp> S;
    for (int i = 1; i <= tot; ++i) {
        if (p[i].cnt < 0) {
            S.insert({ p[i].x, p[i].y, -p[i].cnt });
        } else if (p[i].cnt > 0) {
            auto it = S.upper_bound(p[i]);
            while (it != S.begin() && p[i].cnt > 0) {
                --it;
                if (it->cnt >= p[i].cnt) ans += p[i].cnt, it->cnt -= p[i].cnt, p[i].cnt = 0;
                else ans += it->cnt, p[i].cnt -= it->cnt, it->cnt = 0;
                if (it->cnt == 0)
                    it = S.erase(it);
            }
        }
    }
    cout << ans << endl;
    return 0;
}
```
