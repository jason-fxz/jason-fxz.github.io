---
title: 'XJ - 蹦蹦炸弹'
date: 2020-12-2 22:02:30
tags: [XJ,并查集,MST]
categories: 题解
hidden: true
---


<!--more-->

[蹦蹦炸弹](https://dev.xjoi.net/contest/1609/problem/4)

## 题意

给你一个图，$n$ 个结点，$m$ 种连边方式，每一种连边方式用 $x,y,l,w$ 四个参数描述， 意思是，对所有 $0\le i<l$ ，在编号为 $x+i$ 和 $y+i$ 的结点之间连接了一条权值为 $w$ 的边。请你输出这张图的 $\mathrm{MST}$（最小生成树） 边权值和。

$$
1\le n\le10^5,1\le m\le 5\times 10^5,x\ne y,\max(x,y)+l-1\le n,\;0\le w\le10^9
$$

## 题解

高级并查集 ~~套路题~~。

考虑我们一般求 $\mathrm{MST}$ 用 $\mathrm{Kruskal}$ 算法，从小到大枚举边，将不在一个连通块内的点合并，那么这里就是一次性判断 $(x,y),(x+1,y+1),\dots,(x+l-1,y+l-1)$ 这些点对中有多少个点对不在同一连通块内，并将不在连通块中的点合并。

然而普通的并查集判断这么多次肯定会炸。

考虑到每次连边点都是连续的，我们可以用倍增的思想优化并查集。我们开 $\log_2{n}$ 个并查集，如果 $x,y$ 在第 $i$ 个并查集中属于一个集合，则在原图中 $(x,y),(x+1,y+1),\dots,(x+2^i-1,y+2^i-1)$  **每一个点对**都分别在同一连通块内。

考虑如何查询+合并，若现在要使 $x,x+1,\dots,x+l-1$ 对应与 $y,y+1,\dots,y+l-1$ 连边。类似 ST 表，令 $k=\lfloor \log_2 l\rfloor$ ，在第 $k$ 个并查集上合并 $x,y$ 和 $x+l-2^k,y+l-2^k$ 。

在合并第 $k$ 个并查集的 $u,v$ 时，如果此时 $u,v$ 在同一个连通块内直接返回，不然合并结点，并且递归合并 $k-1$ 号并查集的 $u,v$ 和 $u+2^{k-1},v+2^{k-1}$ 并返回合并成功的次数。若 $k=0$ 就直接合并并返回 $1$ 即可。

考虑复杂度为什么正确，递归调用时遇到已经合并就返回，且无论在那个并查集上，最多合并 $n-1$ 次，共 $\log n$ 个并查集，故并查集的复杂度是 $O(n\log n \alpha(n) )$   

总复杂度 $O(n\log n \alpha(n) +m\log m)$ 

## CODE

```cpp
#include <bits/stdc++.h>
#define re register
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
using namespace std;
typedef long long ll;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename T> bool umax(T &x,T y) {return (x<y)?(x=y,true):(false);}
template<typename T> bool umin(T &x,T y) {return (x>y)?(x=y,true):(false);}
const int N = 100010;
const int M = 21;
struct edge{
    int x,y,l,w;
    bool operator<(const edge a) const {
        return w<a.w;
    }
}e[N*5];
int n,m;
int fa[M][N];
// fa[j][i] 含义是 [i,i+2^j-1] 的点搞在一起 与 其他点的连通关系，
int find(int j,int u) {
    return fa[j][u]==u?u:fa[j][u]=find(j,fa[j][u]);
}
void init() {
    for(re int j=log2(n);j>=0;--j) {
        for(re int i=1;i<=n;++i) fa[j][i]=i;
    }
}
int merge(int j,int u,int v) {
    int fu=find(j,u),fv=find(j,v);
    if(fu==fv) return 0;
    if(j==0) {
        fa[j][fu]=fv;
        return 1;
    }else {
        int a=merge(j-1,u,v)+merge(j-1,u+(1<<(j-1)),v+(1<<(j-1)));
        fa[j][fu]=fv;
        return a;
    }
}
int main() {
    red(n);red(m);
    for(re int i=1;i<=m;++i) {
        red(e[i].x);red(e[i].y);red(e[i].l);red(e[i].w);
    }
    init();
    sort(e+1,e+m+1);
    int cnt=0;
    ll ans=0;
    for(int i=1;i<=m;i++) {
        int k=log2(e[i].l);
        int ct=merge(k,e[i].x,e[i].y)+merge(k,e[i].x+e[i].l-(1<<k),e[i].y+e[i].l-(1<<k));
        cnt+=ct;
        ans+=1ll*ct*e[i].w;
    }
    printf("%lld\n",ans);
    return 0;
}
```
