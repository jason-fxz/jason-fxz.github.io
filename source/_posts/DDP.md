---
title: 动态DP
date: 2021-03-03 21:55:24
tags: [动态规划,矩阵乘法]
categories:
- 📚算法笔记
---

把某些简单的树形DP改成带修改的，就变得十分不可做，便需要使用「动态DP」。“动态”顾名思义就是带修改的呗。

<!--more-->

## 【模板】"动态 DP"&动态树分治

[luogu-P4719](https://www.luogu.com.cn/problem/P4719) ：$n$ 个点的树，每个点有权值，$q$ 次询问，每次修改一个点的权值后求**最大权独立集**。
$$
1\le n,q \le 10^5
$$
不考虑修改就是一个简单的DP问题：
设 $f_{u,1}$ 表 $u$ 子树中选择 $u$ 点的最大权独立集，$f_{u,0}$ 表不选 $u$ 的最大权独立集，原来的转移方程：
$$
f_{u,0}=\sum_{v\in son(u)} \max\big(f_{v,0},f_{v,1}\big)
$$

$$
f_{u,1} =a_u+\sum_{v\in son(u)} f_{v,0}
$$

上式十分繁琐，我们简化一下：对树进行重链剖分，设 $g_{u,0}$ 表示 $u$ 的轻儿子都不选的最大权独立集，$g_{u,1}$ 表示可选可不选。
$$
g_{u,0} = \sum_{v\in lightson(u)} f_{v,0}
$$

$$
g_{u,1} = \sum_{v\in lightson(u)} \max\big(f_{v,0},f_{v,1}\big)
$$

那么有（$s$ 是 $u$ 重儿子）：
$$
f_{u,0}=g_{u,1} +\max \big(f_{s,0},f_{s,1}\big)
$$

$$
f_{u,1}=g_{u,0} +a_{u}+f_{s,0}
$$

考虑写成矩阵乘法的形式：
$$
\begin{bmatrix}
g_{u,1} & g_{u,1}\\
g_{u,0}+a_u & -\infty\\
\end{bmatrix}
\times
\begin{bmatrix}
f_{s,0}\\
f_{s,1}
\end{bmatrix}
=
\begin{bmatrix}
f_{u,0}\\
f_{u,1}
\end{bmatrix}
$$
这里的矩乘定义为 :
$$
A_{i,j}=\max \big\{B_{i,k}+C_{k,j}\big\}
$$
单位矩阵应该写成：
$$
I=
\begin{bmatrix}
0 & -\infty \\
-\infty & 0
\end{bmatrix}
$$


树剖后，在每条重链链底（也就是叶子节点）记录 $f_{u,0}=0,f_{u,1}=a_u$ ，在链上用线段树维护转移矩阵的积。

**询问**：求一个点的值即从链底乘到当前节点。

**修改**：对当前点的转移矩阵进行修改，然后求出链顶的值，链顶一定是另一条链某点的轻儿子，再修改那个点的转移矩阵…… 到根为止。

以上算法的复杂度为 $O(k^3q\log ^2n)$ ，其中 $k$ 为矩阵大小。

**卡常**：矩阵乘法循环展开，树剖线段树对**每条重链单独**开一颗线段树，fastIO。

### CODE

未卡常版：可过 [【模板】"动态 DP"&动态树分治](https://www.luogu.com.cn/problem/P4719)

```cpp
#include <bits/stdc++.h>
#define ls (x<<1)
#define rs (x<<1|1)
#define mid ((l+r)>>1)
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template <typename _Tp,typename... Args>void red(_Tp &t,Args &... args){red(t);red(args...);}
using namespace std;
typedef long long ll;
const int N = 100010;
const int inf = 0x3f3f3f3f;
int head[N],pnt[N<<1],nxt[N<<1],E;
int siz[N],top[N],bot[N],son[N],dep[N],id[N],rk[N],fa[N],tim;
int n,m,a[N],f[N][2];
struct mat {
    int e[2][2];
    int *operator[](const int k) {return e[k];}
    mat() {e[0][0]=e[1][1]=e[0][1]=e[1][0]=-inf;}
    void init() {e[0][0]=e[1][1]=0;e[0][1]=e[1][0]=-inf;}
};
mat operator*(mat A,mat B) {
    mat C;
    for(int i=0;i<2;++i) 
        for(int j=0;j<2;++j) 
            for(int k=0;k<2;++k) 
                C[i][j]=max(C[i][j],A[i][k]+B[k][j]);
    return C;
}
mat M[N<<2],G[N];
void adde(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void dfs1(int u,int f,int d) {
    dep[u]=d; fa[u]=f; siz[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs1(v,u,d+1); siz[u]+=siz[v];
        if(siz[son[u]]<siz[v]) son[u]=v;
    }
}
void dfs2(int u,int topf) {
    top[u]=topf; id[u]=++tim; rk[tim]=u;
    if(son[u]) dfs2(son[u],topf); else bot[topf]=u;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==son[u]||v==fa[u]) continue;
        dfs2(v,v);
    }
}
void build(int l=1,int r=n,int x=1) {
    if(l==r) return M[x]=G[rk[l]],void();
    build(l,mid,ls); build(mid+1,r,rs);
    M[x]=M[ls]*M[rs];
}
mat query(int L,int R,int l=1,int r=n,int x=1) {
    if(L<=l&&r<=R) return M[x];
    if(L<=mid&&R>mid) return query(L,R,l,mid,ls)*query(L,R,mid+1,r,rs);
    return (L<=mid)?query(L,R,l,mid,ls):query(L,R,mid+1,r,rs);
}
mat tmp;
void modify(int ps,int l=1,int r=n,int x=1) {
    if(l==r) return M[x]=tmp,void();
    if(ps<=mid) modify(ps,l,mid,ls);
    else modify(ps,mid+1,r,rs);
    M[x]=M[ls]*M[rs];
}

void pre(int u) {
    f[u][1]=a[u]; f[u][0]=0;
    G[u][1][0]=a[u]; G[u][1][1]=-inf;
    G[u][0][0]=G[u][0][1]=0;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa[u]) continue;
        pre(v);
        f[u][0]+=max(f[v][1],f[v][0]);
        f[u][1]+=f[v][0];
        if(v!=son[u]) {
            G[u][0][0]+=max(f[v][1],f[v][0]);
            G[u][0][1]=G[u][0][0];
            G[u][1][0]+=f[v][0];
        }
    }
}

void Modify(int u,int val) {
    int fu=top[u];
    G[u][1][0]+=val-a[u]; a[u]=val;
    while(u!=0) {
        mat t=query(id[fu],id[bot[fu]]);
        tmp=G[u]; modify(id[u]);
        mat tt=query(id[fu],id[bot[fu]]);
        u=fa[fu],fu=top[u];
        G[u][0][0]+=-max(t[0][0],t[1][0])+max(tt[0][0],tt[1][0]);
        G[u][0][1]=G[u][0][0];
        G[u][1][0]+=tt[0][0]-t[0][0];

    }
}
void print(mat A) {
    printf("-- matrix --\n");
    for(int i=0;i<2;++i){
        for(int j=0;j<2;++j) {
            printf("%d ",A[i][j]);
        }
        printf("\n");
    }
    printf("-- ------ --\n");
}
signed main() {
    red(n,m);
    for(int i=1;i<=n;++i) red(a[i]);
    for(int i=1;i<n;++i) {
        int u,v; red(u,v);
        adde(u,v); adde(v,u);
    }
    dfs1(1,0,1); dfs2(1,1);
    pre(1); build();
    while(m--) {
        int x,y;red(x,y);
        Modify(x,y);
        mat A=query(id[1],id[bot[1]]);
        printf("%d\n",max(A[0][0],A[1][0]));
    }
    return 0;
}

```


卡常版，可过 [【模板】"动态DP"&动态树分治（加强版）](https://www.luogu.com.cn/problem/P4751)

```cpp
#include <bits/stdc++.h>
#define ls(x) T[x].l
#define rs(x) T[x].r
#define mid ((l+r)>>1)
namespace fast_io {
    #define MAX_INPORT (1<<22)   // 一次读入量
    #define MAX_OUTPORT (1<<22)  // 一次输出量
    char buf[MAX_INPORT],*p1=buf,*p2=buf,obuf[MAX_OUTPORT],*O=obuf,*end=obuf+MAX_OUTPORT;
    inline char gc() {
        #ifdef DEBUG
            return getchar();
        #endif
        if(p1!=p2) return *p1++;
        p1=buf;
        p2=p1+fread(buf,1,MAX_INPORT,stdin);
        return p1==p2?EOF:*p1++;
    }
    template<typename _Tp>    // 读入任意类型整数
    inline void red(_Tp &x) {
        x=0;bool fg=0;char ch=gc();
        while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=gc();}
        while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=gc();
        if(fg) x=-x;
    }
    template <typename _Tp,typename... Args>void red(_Tp &t,Args &... args){red(t);red(args...);}
    inline void puc(char ch) {      // 输出字符
        #ifdef DEBUG
            return putchar(ch),void();
        #endif
        if(O!=end) *O++=ch;
        else {fwrite(obuf,O-obuf,1,stdout);O=obuf;*O++=ch;}
    }
    template<typename _Tp>   
    void _prt(_Tp x) {
        if(x>(_Tp)9) _prt(x/(_Tp)10);
        puc((char)(x%10+(_Tp)'0'));
    }
    template<typename _Tp>     // 读出任意类型整数
    void prt(_Tp x) {if(x<0) puc('-'),x=-x;_prt(x);}
    void close() {fwrite(obuf,O-obuf,1,stdout);} // 输出
}
using fast_io::red;
using fast_io::prt;
using fast_io::puc;

using namespace std;
typedef long long ll;
const int N = 1000010;
const int inf = 0x3f3f3f3f;
int head[N],pnt[N<<1],nxt[N<<1],E;
int siz[N],top[N],bot[N],son[N],dep[N],id[N],rk[N],fa[N],tim;
int n,m,a[N],f[N][2];
struct mat {
    int e[2][2];
    int *operator[](const int k) {return e[k];}
    mat() {e[0][0]=e[1][1]=e[0][1]=e[1][0]=-inf;}
    void init() {e[0][0]=e[1][1]=0;e[0][1]=e[1][0]=-inf;}
};
mat G[N];
struct node {
    int l,r; mat M;
}T[N<<1]; int tot,rt[N];
mat operator*(mat A,mat B) {
    mat C;
    C[0][0]=max(C[0][0],A[0][0]+B[0][0]);
    C[0][0]=max(C[0][0],A[0][1]+B[1][0]);
    C[0][1]=max(C[0][1],A[0][0]+B[0][1]);
    C[0][1]=max(C[0][1],A[0][1]+B[1][1]);
    C[1][0]=max(C[1][0],A[1][0]+B[0][0]);
    C[1][0]=max(C[1][0],A[1][1]+B[1][0]);
    C[1][1]=max(C[1][1],A[1][0]+B[0][1]);
    C[1][1]=max(C[1][1],A[1][1]+B[1][1]);
    return C;
}
inline int newnode() {++tot; T[tot].l=T[tot].r=0; return tot;}
int build(int l,int r) {
    int p=newnode();
    if(l==r) {T[p].M=G[rk[l]];return p;}
    ls(p)=build(l,mid); rs(p)=build(mid+1,r);
    T[p].M=T[ls(p)].M*T[rs(p)].M;
    return p;
}
mat tmp;
void modify(int p,int ps,int l,int r) {
    if(l==r) return T[p].M=tmp,void();
    if(ps<=mid) modify(ls(p),ps,l,mid);
    else modify(rs(p),ps,mid+1,r);
    T[p].M=T[ls(p)].M*T[rs(p)].M;
}
mat query(int p,int L,int R,int l,int r) {
    if(L<=l&&r<=R) return T[p].M;
    if(L<=mid&&R>=mid) return query(ls(p),L,R,l,mid)*query(rs(p),L,R,mid+1,r);
    return (L<=mid)?query(ls(p),L,R,l,mid):query(rs(p),L,R,mid+1,r);
}
inline mat ask(int p) {return T[p].M;}
void adde(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void dfs1(int u,int f,int d) {
    dep[u]=d; fa[u]=f; siz[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs1(v,u,d+1); siz[u]+=siz[v];
        if(siz[son[u]]<siz[v]) son[u]=v;
    }
}
void dfs2(int u,int topf) {
    top[u]=topf; id[u]=++tim; rk[tim]=u;
    if(son[u]) dfs2(son[u],topf);
    else {
        rt[topf]=build(id[topf],id[u]);
        bot[topf]=u;
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==son[u]||v==fa[u]) continue;
        dfs2(v,v);
    }
}


void pre(int u) {
    f[u][1]=a[u]; f[u][0]=0;
    G[u][1][0]=a[u]; G[u][1][1]=-inf;
    G[u][0][0]=G[u][0][1]=0;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa[u]) continue;
        pre(v);
        f[u][0]+=max(f[v][1],f[v][0]);
        f[u][1]+=f[v][0];
        if(v!=son[u]) {
            G[u][0][0]+=max(f[v][1],f[v][0]);
            G[u][0][1]=G[u][0][0];
            G[u][1][0]+=f[v][0];
        }
    }
}

void Modify(int u,int val) {
    int fu=top[u];
    G[u][1][0]+=val-a[u]; a[u]=val;
    mat tt,t;
    while(u!=0) {
        t=T[rt[fu]].M;
        tmp=G[u]; modify(rt[fu],id[u],id[fu],id[bot[fu]]);
        tt=T[rt[fu]].M;
        u=fa[fu],fu=top[u];
        G[u][0][0]+=-max(t[0][0],t[1][0])+max(tt[0][0],tt[1][0]);
        G[u][0][1]=G[u][0][0];
        G[u][1][0]+=tt[0][0]-t[0][0];

    }
}

void print(mat A) {
    printf("-- matrix --\n");
    for(int i=0;i<2;++i){
        for(int j=0;j<2;++j) {
            printf("%d ",A[i][j]);
        }
        printf("\n");
    }
    printf("-- ------ --\n");
}
signed main() {
    red(n,m);
    for(int i=1;i<=n;++i) red(a[i]);
    for(int i=1;i<n;++i) {
        int u,v; red(u,v);
        adde(u,v); adde(v,u);
    }
    dfs1(1,0,1);pre(1); dfs2(1,1);    
    int lst=0;
    while(m--) {
        int x,y;red(x,y);
        x^=lst;
        Modify(x,y);
        mat A=T[rt[1]].M;
        lst=max(A[0][0],A[1][0]);
        prt(lst);puc('\n');
    }
    fast_io::close();
    return 0;
}
```



## 保卫王国

题意：$n$ 个点的树，每个点有代价（正的），每条边上的两点至少选一个，最小化代价。 $q$ 次询问，每次询问钦定 $u,v$ 两点的选取情况，求满足这个条件的最小代价。

题意显然可以转化为 总代价-最大权独立集 。然后就是钦定的事了：
$$
\begin{bmatrix}
g_{u,1} & g_{u,1}\\
g_{u,0}+a_u & -\infty\\
\end{bmatrix}
\times
\begin{bmatrix}
f_{s,0}\\
f_{s,1}
\end{bmatrix}
=
\begin{bmatrix}
f_{u,0}\\
f_{u,1}
\end{bmatrix}
$$

强制 $u$ 不选，即让 $f_{u,1}=-\infty$：
$$
\begin{bmatrix}g_{u,1}&g_{u,1} \\ -\infty & -\infty\end{bmatrix}
$$
强制 $u$ 选，即让 $f_{u,0}=-\infty$：
$$
\begin{bmatrix}-\infty&-\infty \\ g_{u,0}+a_u & -\infty\end{bmatrix}
$$

### CODE


```cpp
#pragma GCC optimize(2)
#include <bits/stdc++.h>
#define ls(x) T[x].l
#define rs(x) T[x].r
#define mid ((l+r)>>1)
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template <typename _Tp,typename... Args>void red(_Tp &t,Args &... args){red(t);red(args...);}
using namespace std;
typedef long long ll;
const int N = 1000010;
const ll inf = 0x3f3f3f3f3f3f3f;
int head[N],pnt[N<<1],nxt[N<<1],E;
int siz[N],top[N],bot[N],son[N],dep[N],id[N],rk[N],fa[N],tim;
int n,m;
ll a[N],f[N][2],sum;
struct mat {
    ll e[2][2];
    ll *operator[](const int k) {return e[k];}
    mat() {e[0][0]=e[1][1]=e[0][1]=e[1][0]=-inf;}
    void init() {e[0][0]=e[1][1]=0;e[0][1]=e[1][0]=-inf;}
};
mat G[N];
struct node {
    int l,r; mat M;
}T[N<<1]; int tot,rt[N];
mat operator*(mat A,mat B) {
    mat C;
    C[0][0]=max(C[0][0],A[0][0]+B[0][0]);
    C[0][0]=max(C[0][0],A[0][1]+B[1][0]);
    C[0][1]=max(C[0][1],A[0][0]+B[0][1]);
    C[0][1]=max(C[0][1],A[0][1]+B[1][1]);
    C[1][0]=max(C[1][0],A[1][0]+B[0][0]);
    C[1][0]=max(C[1][0],A[1][1]+B[1][0]);
    C[1][1]=max(C[1][1],A[1][0]+B[0][1]);
    C[1][1]=max(C[1][1],A[1][1]+B[1][1]);
    return C;
}
inline int newnode() {++tot; T[tot].l=T[tot].r=0; return tot;}
int build(int l,int r) {
    int p=newnode();
    if(l==r) {T[p].M=G[rk[l]];return p;}
    ls(p)=build(l,mid); rs(p)=build(mid+1,r);
    T[p].M=T[ls(p)].M*T[rs(p)].M;
    return p;
}
mat tmp;
void modify(int p,int ps,int l,int r) {
    if(l==r) return T[p].M=tmp,void();
    if(ps<=mid) modify(ls(p),ps,l,mid);
    else modify(rs(p),ps,mid+1,r);
    T[p].M=T[ls(p)].M*T[rs(p)].M;
}
mat query(int p,int L,int R,int l,int r) {
    if(L<=l&&r<=R) return T[p].M;
    if(L<=mid&&R>=mid) return query(ls(p),L,R,l,mid)*query(rs(p),L,R,mid+1,r);
    return (L<=mid)?query(ls(p),L,R,l,mid):query(rs(p),L,R,mid+1,r);
}
inline mat ask(int p) {return T[p].M;}
void adde(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void dfs1(int u,int f,int d) {
    dep[u]=d; fa[u]=f; siz[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs1(v,u,d+1); siz[u]+=siz[v];
        if(siz[son[u]]<siz[v]) son[u]=v;
    }
}
void dfs2(int u,int topf) {
    top[u]=topf; id[u]=++tim; rk[tim]=u;
    if(son[u]) dfs2(son[u],topf);
    else {
        rt[topf]=build(id[topf],id[u]);
        bot[topf]=u;
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==son[u]||v==fa[u]) continue;
        dfs2(v,v);
    }
}
void pre(int u) {
    f[u][1]=a[u]; f[u][0]=0;
    G[u][1][0]=a[u]; G[u][1][1]=-inf;
    G[u][0][0]=G[u][0][1]=0;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa[u]) continue;
        pre(v);
        f[u][0]+=max(f[v][1],f[v][0]);
        f[u][1]+=f[v][0];
        if(v!=son[u]) {
            G[u][0][0]+=max(f[v][1],f[v][0]);
            G[u][0][1]=G[u][0][0];
            G[u][1][0]+=f[v][0];
        }
    }
}

void Modify(int u,int op,ll val) {
    int fu=top[u];
    if(op==0) {
        G[u][1][0]=val;
    }else {
        G[u][0][0]=G[u][0][1]=val;
    }
    mat tt,t;
    while(u!=0) {
        t=T[rt[fu]].M;
        tmp=G[u]; modify(rt[fu],id[u],id[fu],id[bot[fu]]);
        tt=T[rt[fu]].M;
        u=fa[fu],fu=top[u];
        G[u][0][0]+=-max(t[0][0],t[1][0])+max(tt[0][0],tt[1][0]);
        G[u][0][1]=G[u][0][0];
        G[u][1][0]+=tt[0][0]-t[0][0];
    }
}
void print(mat A) {
    printf("-- matrix --\n");
    for(int i=0;i<2;++i){
        for(int j=0;j<2;++j) {
            printf("%d ",A[i][j]);
        }
        printf("\n");
    }
    printf("-- ------ --\n");
}
signed main() {
    char QAQ[5];
    red(n,m); scanf("%s",QAQ);
    for(int i=1;i<=n;++i) red(a[i]),sum+=a[i];
    for(int i=1;i<n;++i) {
        int u,v; red(u,v);
        adde(u,v); adde(v,u);
    }
    dfs1(1,0,1);pre(1); dfs2(1,1);
    while(m--) {
        int u,v,x,y; red(u,x,v,y);
        x^=1; y^=1;
        if((fa[u]==v||fa[v]==u)&&x==1&&y==1) {
            puts("-1");
            continue;
        }
        ll pre1=G[u][x^1][0],pre2=G[v][y^1][0];
        Modify(u,x,-inf); Modify(v,y,-inf);
        mat A=T[rt[1]].M;
        printf("%lld\n",sum-max(A[0][0],A[1][0]));
        Modify(u,x,pre1); Modify(v,y,pre2);
    }
    return 0;   
}

```



