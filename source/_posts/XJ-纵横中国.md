---
title: 'XJ - 纵横中国'
date: 2021-05-06 22:24:46
updated: 2021-05-07 22:24:46
tags: [线段树,凸包,XJ]
categories: 题解
hidden: true
---

线段树维护凸包,闵可夫斯基和

## 题意

给你一个长度为 $n$ 的序列 $\{a_i\}$，定义区间 $[L,R]$ 合法，需要满足 $\forall L\le l \le r\le R:\sum_{i=l}^r a_i\le k$。可以花费 $1$ 的代价使某位置 $-1$。$q$ 次询问，每次给出 $l_i,r_i,k_i$，问最少花费多少代价使区间 $[l,r]$ 合法。
$$
1 \le n,q \le 3\times 10^5,|a_i|\le 3\times 10^6,0\le k_i\le 10^{12}
$$

---

## 题解

考虑修改过的值为 $\{b_i\}$，令 $p_i=\sum_{j=1}^i a_i-b_i$（即修改次数前缀和），记 $s_i=\sum_{j=1}^i a_i$。区间 $[L,R]$ 合法即对于任意 $L-1\le i< j\le R$ 都满足 $s_j-s_{i}-(p_j-p_{i})\le k$，还有 $p_{i}\le p_{i+1}$，$p_{L-1}=0$，要最小代价即让 $p_R$ 最小。

这显然是个差分约束：

- $p_j-p_i\ge s_j-s_i-k$ 则 $i\to j$ 边权 $s_j-s_i-k$
- $p_{i+1}-p_i\ge 0$ 则 $i\to i+1$ 边权 $0$
- 答案就是 $L-1$ 到 $R$ 的最长路。

仔细考虑一下，上述的差分约束等价于在 $[L,R]$ 找若干个不交子段，需要最大化 $\texttt{子段和}-\texttt{子段个数}\times k$。

然后这就是经典的费用流模型（如果能建出费用流模型，可以证明 *最小/最大费用* 关于 *增广次数* 的函数是**凸函数**），令 $f(K)$ 表示恰好选 $K$ 个段的最大子段和，可得 $f(K)$ 是凸的。考虑 $(i,f(i))$ 构成的凸包，找到其与斜率为 $k$ 直线的切点便是最优解。

> 证明：  
> 我们要求的是 $\max\{f(x)-kx\}$，考虑其导函数的零点 $x_0$，满足 $f'(x_0)-k=0$，此时 $f'(x_0)=k$ 即 $x_0$ 为 $f(x)$ 斜率为 $k$ 的切点，又因为 $f(x)$ 是凸函数，故切点唯一。

考虑区间怎么办，来个线段树分治结构，每个节点维护对应区间的凸包，考虑如何合并两个子区间。$o$ 是当前节点，$ls,rs$ 分别为线段时上左右儿子，显然有 $f_o(i)=\max_{j+k=i} f_{ls}(j)+f_{rs}(k)$，发现这玩意是 **闵可夫斯基和**，可以做到线性时间合并。然后还有一个恶心的细节，考虑左区间的右端点和右区间的左端点同时被选择，此时段数要减一（即凸包向左移动一单位），故我们需要每个节点维护 不带限制、左端点必选、右端点必选、两端点都选 的凸包。注意到还会有两凸包区 $\max$，我们只要对应位置取 $\max$ 即可。

考虑一次询问对应 $\log$ 个区间，二分斜率可得到每个区间分别的切点，即每个区间的最优解（四种情况），然后顺序合并即可。这样跑是 $O((n+q)\log^2 n)$ ，复杂度有点大，考虑线段树维护的所有凸包总点数是 $O(n\log n)$ 的，故我们可以将询问离线，按照 $k$ 排序依次求出切点即可，这样免了二分，复杂度 $O((n+q)\log n)$。

## CODE

代码实现有些细节，注意 `vector` 实际分配的空间是当前的 $1.5\sim 2$ 倍，可能会 MLE，我们可以用 `shrink_to_fit()` 函数使分配空间正好变成 `size()` 大小。

```cpp
#include <bits/stdc++.h>
#define ls (x << 1)
#define rs (x << 1 | 1)
#define mid ((l + r) >> 1)
#define SZ(x) (int)x.size()
template <typename Tp>
inline void read(Tp& x) {
    x = 0; bool fg = 0; char ch = getchar();
    while (ch < '0' || ch > '9') { if (ch == '-') fg ^= 1; ch = getchar();}
    while (ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if (fg) x = -x;
}
template <typename Tp, typename... Args>
void read(Tp& t, Args& ...args) { read(t); read(args...); }
template <typename Tp> void umax(Tp& x, Tp y) { if (x < y) x = y; }
using namespace std;
typedef long long ll;
typedef vector<ll> vt;
const int N = 300010;
const ll inf = 0x3f3f3f3f3f3f3f3f;
vt D[N << 2][4]; // 0 两边不保证 1 右边保证 2 左边保证 3 都保证
int n, Q, a[N];
void merge(vt &x, const vt& a, const vt& b, bool fg = 0) { // 闵可夫斯基和
    x.clear();
    if (!SZ(a) || !SZ(b)) { return ; }
    x.reserve(SZ(a) + SZ(b));
    x.push_back(a[0] + b[0]);
    int p = 1, q = 1; ll w = a[0] + b[0];
    while (p < SZ(a) || q < SZ(b)) {
        if (q >= SZ(b) || (p < SZ(a) && a[p] - a[p - 1] > b[q] - b[q - 1]))
            x.push_back(w += a[p] - a[p - 1]), ++p;
        else x.push_back(w += b[q] - b[q - 1]), ++q;
    }
    if (fg && !x.empty()) x.erase(x.begin());
}
// void print(int x, int l, int r) {
//     printf("[%d, %d] >> \n", l, r);
//     for (int i = 0; i < 4; ++i) {
//         printf("%d>  ", i);
//         for (int j = 0; j < (int)D[x][i].size(); ++j) {
//             printf("%lld%c", D[x][i][j], " \n"[j == (int)D[x][i].size() - 1]);
//         }
//     }
// }
void getmax(vt& x, const vt& a, const vt& b) {
    x.reserve(SZ(a) + SZ(b));
    int p = 0;
    while (p < SZ(a) && p < SZ(b)) {
        x.push_back(max(a[p], b[p])); ++p;
    }
    while (p < SZ(a)) x.push_back(a[p]), ++p;
    while (p < SZ(b)) x.push_back(b[p]), ++p;
}
vt tmp1, tmp2;
void build(int l = 1, int r = n, int x = 1) {
    if (l == r) {
        D[x][0] = { 0, max(0, a[l]) };
        D[x][1] = D[x][2] = D[x][3] = { -inf, a[l] };
        return ;
    }
    build(l, mid, ls); build(mid + 1, r, rs);
    merge(tmp1, D[ls][0], D[rs][0]); merge(tmp2, D[ls][1], D[rs][2], 1); getmax(D[x][0], tmp1, tmp2);
    merge(tmp1, D[ls][0], D[rs][1]); merge(tmp2, D[ls][1], D[rs][3], 1); getmax(D[x][1], tmp1, tmp2); 
    merge(tmp1, D[ls][2], D[rs][0]); merge(tmp2, D[ls][3], D[rs][2], 1); getmax(D[x][2], tmp1, tmp2);
    merge(tmp1, D[ls][2], D[rs][1]); merge(tmp2, D[ls][3], D[rs][3], 1); getmax(D[x][3], tmp1, tmp2);
    D[x][0].shrink_to_fit(), D[x][1].shrink_to_fit(), D[x][2].shrink_to_fit(), D[x][3].shrink_to_fit();
}
struct que { int l, r, id; ll k; }q[N];
ll dp[2], tmp[2], ans[N];

void query(int L, int R, ll k, int l = 1, int r = n, int x = 1) {
    if (L <= l && r <= R) { 
        ll tp[4] = {0, 0, 0, 0};
        for (int t = 0; t < 4; ++t) {
            while (SZ(D[x][t]) > 1 &&
                D[x][t].back() - D[x][t][SZ(D[x][t]) - 2] < k) D[x][t].pop_back();
            tp[t] = D[x][t].back() - k * (SZ(D[x][t]) - 1);
        }
        tmp[0] = max(dp[0] + tp[0], dp[1] + tp[2] + k);
        tmp[1] = max(dp[0] + tp[1], dp[1] + tp[3] + k);
        dp[0] = tmp[0], dp[1] = tmp[1];
        return ;
    }
    if (L <= mid) query(L, R, k, l, mid, ls);
    if (R > mid) query(L, R, k, mid + 1, r, rs);
}
 
int main() {
    read(n, Q);
    for (int i = 1; i <= n; ++i) read(a[i]);
    build();
    for (int i = 1; i <= Q; ++i) {
        read(q[i].l, q[i].r, q[i].k); q[i].id = i;
    }
    sort(q + 1, q + Q + 1, [](que& x, que& y) {
        return x.k < y.k;
    });
    for (int i = 1; i <= Q; ++i) {
        dp[0] = 0; dp[1] = -inf;
        query(q[i].l, q[i].r, q[i].k);
        ans[q[i].id] = max(dp[0], dp[1]);
    }
    for (int i = 1; i <= Q; ++i) printf("%lld\n", ans[i]);
    return 0;
}
```
