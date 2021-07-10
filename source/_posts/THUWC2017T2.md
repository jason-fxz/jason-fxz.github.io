---
title: '『THUWC2017』在美妙的数学王国中畅游'
date: 2021-04-26 21:58:45
tags: [LCT,数学]
categories:
- 题解
---

泰勒展开，LCT

<!--more-->

## 题解

题目都告诉你了：

> 若函数 $f(x)$ 的 $n$ 阶导数在 $[a,b]$ 区间内连续，则对 $f(x)$ 在 $x_0 \ (x_0\in[a,b])$ 处使用 $n$ 次拉格朗日中值定理可以得到带拉格朗日余项的泰勒展开式
> $$f(x)=\sum_{k=0}^{n-1}\frac{f^{(k)}(x_0)(x-x_0)^k}{k!}+\frac{f^{(n)}(\xi)(x-x_0)^n}{n!},x\in[a,b]$$
> 其中，当 $x > x_0$ 时，$\xi\in[x_0,x]$。当 $x < x_0$ 时，$\xi\in[x,x_0]$。
> $f^{(n)}$ 表示函数 $f$ 的 $n$ 阶导数。

---

那摆明了全部泰勒展开，LCT 维护多项式和。

对于 $f(x)=e^{ax+b}$，在 $x_0=-\dfrac{b}{a}$ 处展开，有：

$$
\begin{aligned}
f(x)\approx& \sum_{k=0}^{n-1} \frac{a^k\left(x+\frac{b}{a}\right)^k}{k!}\\
=& \sum_{k=0}^{n-1} \frac{(ax+b)^k}{k!}\\
=& \sum_{k=0}^{n-1} \frac{1}{k!}\sum_{i=0}^k \binom{k}{i}a^ib^{k-i}x^i\\
=& \sum_{i=0}^{n-1} x^i \sum_{k=i}^{n-1} \frac{a^ib^{k-i}}{i!(k-i)!}
\end{aligned}
$$

对于 $f(x)=\sin(ax+b)$，在 $x_0=-\dfrac{b}{a}$ 处展开，有：

$$
f(x)\approx \sum_{k=0}^{n-1} \frac{s(k)a^k\left(x+\frac{b}{a}\right)^k}{k!}
=\sum_{k=0}^{n-1} \frac{s(k)(ax+b)^k}{k!}
$$

其中，

$$
s(k)=\begin{cases}
1 & k \bmod 4=1\\
-1 & k \bmod 4=3\\
0 & \text{otherwise}
\end{cases}
$$

化简一下，易得：

$$
f(x)\approx \sum_{i=0}^{n-1} x^i \sum_{k=i}^{n-1} s(k)\frac{a^ib^{k-i}}{i!(k-i)!}
$$

对于 $f(x)=ax+b$，不用展开。

其他的 Link-Cut-Tree 随便维护就行了，至于精度，展开到 $x^{19}$ 足矣。

## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef double db;
const int N = 100010;
const int D = 20;
struct poly {
    db e[D];
    poly() { memset(e, 0, sizeof(e)); }
    db& operator[](const int& k) { return e[k]; }
    friend poly operator+(const poly& a, const poly& b) {
        poly r;
        for (int i = 0; i < D; ++i) r[i] = a.e[i] + b.e[i];
        return r;
    }
    db operator()(const db& x) const {
        db r = 0, _x = 1;
        for (int i = 0; i < D; ++i) {
            r += e[i] * _x;
            _x *= x;
        }
        return r;
    }
};

void setf(poly& p, int op, db a, db b) {
    memset(p.e, 0, sizeof(p.e));
    static db s[2][4] = {
        { 0, 1, 0, -1},
        { 1, 1, 1, 1 },
    };
    if (op <= 2) {
        db tp = 1; // tp = a^i / i!
        for (int i = 0; i < D; ++i) {
            db cur = tp;
            for (int k = i; k < D; ++k) {
                p[i] += s[op - 1][k % 4] * cur;
                cur = cur * b / (db)(k - i + 1);
            }
            tp = tp * a / (db)(i + 1);
        }
    }
    else p[1] = a, p[0] = b;
}

namespace LCT {
#define ls ch[x][0]
#define rs ch[x][1]
    int f[N], ch[N][2], rev[N];
    poly val[N], sum[N];
    inline bool Get(int x) { return ch[f[x]][1] == x; }
    inline bool isRoot(int x) { return ch[f[x]][0] != x && ch[f[x]][1] != x; }
    inline void PushUp(int x) { sum[x] = sum[ls] + sum[rs] + val[x]; }
    inline void Setrev(int x) { if (x) swap(ls, rs), rev[x] ^= 1; }
    inline void PushDown(int x) {
        if (rev[x]) {
            Setrev(ls); Setrev(rs); rev[x] = 0;
        }
    }
    inline void Rotate(int x) {
        int y = f[x], z = f[y], k = Get(x);
        if (!isRoot(y)) ch[z][Get(y)] = x;
        ch[y][k] = ch[x][k ^ 1]; f[ch[x][k ^ 1]] = y;
        ch[x][k ^ 1] = y; f[y] = x; f[x] = z;
        PushUp(y); PushUp(x);
    }
    void Update(int x) {
        if (!isRoot(x)) Update(f[x]);
        PushDown(x);
    }
    inline void Splay(int x) {
        Update(x);
        for (int fa; fa = f[x], !isRoot(x); Rotate(x)) {
            if (!isRoot(fa)) Rotate(Get(fa) == Get(x) ? fa : x);
        }
        PushUp(x);
    }
    inline void Access(int x) {
        for (int p = 0; x;p = x, x = f[x]) {
            Splay(x); rs = p; PushUp(x);
        }
    }
    inline void MakeRoot(int x) {
        Access(x); Splay(x); Setrev(x);
    }
    inline int FindRoot(int x) {
        Access(x); Splay(x);
        while (ls) PushDown(x), x = ls;
        Splay(x);
        return x;
    }
    inline void Split(int x, int y) {
        MakeRoot(x); Access(y); Splay(y);
    }
    inline void Cut(int x, int y) {
        MakeRoot(x); Access(y); Splay(y);
        ch[y][0] = f[x] = 0;
    }
    inline void Link(int x, int y) {
        MakeRoot(x); MakeRoot(y); f[x] = y;
    }
    inline bool Check(int x, int y) {
        MakeRoot(x);
        return FindRoot(y) == x;
    }
#undef ls
#undef rs
}

int n, m; char type[5];

int main() {
    scanf("%d%d%s", &n, &m, type);
    for (int i = 1; i <= n; ++i) {
        int op; db a, b;
        scanf("%d%lf%lf", &op, &a, &b);
        setf(LCT::val[i], op, a, b);
        LCT::PushUp(i);
    }
    cerr << "init ok." << endl;
    while (m--) {
        char op[20];
        int u, v, f; db a, b, x;
        scanf("%s", op);
        if (op[0] == 'a') {
            scanf("%d%d", &u, &v); ++u, ++v;
            LCT::Link(u, v);
        } else if (op[0] == 'd') {
            scanf("%d%d", &u, &v); ++u, ++v;
            LCT::Cut(u, v);
        } else if (op[0] == 'm') {
            scanf("%d%d%lf%lf", &u, &f, &a, &b); ++u;
            LCT::Split(u, u);
            setf(LCT::val[u], f, a, b);
            LCT::PushUp(u);
        } else {
            scanf("%d%d%lf", &u, &v, &x); ++u, ++v;
            if(!LCT::Check(u, v)) {
                puts("unreachable");
                continue;
            }
            LCT::Split(u, v);
            printf("%.20e\n", LCT::sum[v](x));
        }
    }
    

    return 0;
}
```
