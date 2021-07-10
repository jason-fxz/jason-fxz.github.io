---
title: 'CF19E - Fairy'
date: 2020-11-24 23:14:18
tags: [LCA,树上差分]
categories:
- 题解
---



[CF19E](https://codeforces.com/problemset/problem/19/E)

## 题意

给出一张无向图，求**删除一条边**后此图变成二分图的所有删边方案。

<!--more-->

## 题解

1. 二分图无奇环，无奇环就是二分图
2. 如果该图是二分图，则删边后还是二分图。

于是我们先随便搞一个生成树，这上面的边称为树边，剩下的非树边加上去一定会构成环，称构成奇环的边为 “坏边”，构成偶环的边为 “好边”，与非树边构成的环的树边被非树边 “覆盖”。

考虑搞出的奇环个数（这里的奇环仅仅只一条非树边和生成树构成的环）。

1. 没奇环，本就是二分图，输出所有边即可
2. 有一个奇环，可以删除这条坏边，或者删除被坏边覆盖但**没被好边覆盖**的边。
    > 证明：消除奇环必然要删除一条奇环上的边，考虑为什么要满足 **没被好边覆盖** ，删除被好边覆盖的边不能解决问题，还会存在奇环：这条边属于一个奇环又属于一个偶环，这两环有交，去掉重合部分，奇-偶=奇，还会存在一个奇环。
3. 有多个奇环，删除的边必然被所有坏边覆盖但没被好边覆盖（证明同上）。此时这条坏边显然不能删。


考虑实现，随便搞出个生成树，搞个倍增 LCA，对于边覆盖，树上差分即可。

注意图可能不连通。

## CODE
```cpp
#include <bits/stdc++.h>
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
const int N = 10010;
const int M = 14;
int head[N],id[N<<1],pnt[N<<1],nxt[N<<1],E;
int fa[N][M],far[N],n,m;
int tag[N][2],dep[N],flag,bl[N];
struct edge {
    int u,v,id;
}e[N];int ei;
int ans[N],ai;
int find(int u) {
    return far[u]==u?u:far[u]=find(far[u]);
}
void adde(int u,int v,int _id) {
    E++;pnt[E]=v;nxt[E]=head[u];id[E]=_id;head[u]=E;
}
void dfs(int u) {
    for(int i=1;i<M&&fa[u][i-1];i++) fa[u][i]=fa[fa[u][i-1]][i-1];
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u][0]) continue;
        fa[v][0]=u;
        dep[v]=dep[u]+1;
        bl[v]=id[i];
        dfs(v);
    }
}
int lca(int u,int v) {
    if(find(u)!=find(v)) throw;
    if(dep[u]>dep[v]) swap(u,v);
    int d=dep[v]-dep[u];
    for(int i=0;i<M;i++) if((d>>i)&1) v=fa[v][i];
    if(u==v) return u;
    for(int i=M-1;i>=0;i--) if(fa[u][i]!=fa[v][i]) 
        v=fa[v][i],u=fa[u][i];
    return fa[u][0];
}
void solve(int u) {
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u][0]) continue;
        solve(v);
        tag[u][0]+=tag[v][0];
        tag[u][1]+=tag[v][1];
    }
    if(tag[u][1]==flag&&tag[u][0]==0) {
        ans[++ai]=bl[u];
    }
}
int main() {
    red(n);red(m);
    for(int i=1;i<=n;i++) far[i]=i;
    for(int i=1;i<=m;i++) {
        int u,v;red(u);red(v);
        int fu=find(u),fv=find(v);
        if(fu!=fv) {
            far[fu]=fv;
            adde(v,u,i);adde(u,v,i);
        }else {
            ++ei;e[ei].v=v;e[ei].u=u;e[ei].id=i;
        }
    }
    for(int i=1;i<=n;i++) {
        if(fa[i][0]==0) dfs(i);
    }
    for(int i=1;i<=ei;i++) {
        int lc=lca(e[i].u,e[i].v);
        int cnt=dep[e[i].u]+dep[e[i].v]-dep[lc]*2+1;
        if(cnt&1) flag++,ans[++ai]=e[i].id;
        tag[e[i].u][cnt&1]++;
        tag[e[i].v][cnt&1]++;
        tag[lc][cnt&1]-=2;
    }
    if(flag>1) ai=0;
    for(int i=1;i<=n;i++) {
        if(fa[i][0]==0) solve(i);
    }
    if(flag==0) {
        printf("%d\n",m);
        for(int i=1;i<=m;i++) printf("%d ",i);
    }else {
        sort(ans+1,ans+ai+1);   
        printf("%d\n",ai);
        for(int i=1;i<=ai;i++) printf("%d ",ans[i]);
    }
    return 0;
}
```
