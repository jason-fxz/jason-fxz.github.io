---
title: '「XXI Open Cup GP of Koread」'
date: 2021-04-16 21:35:14
tags: [Open Cup]
categories:
- 题解
---

「XXI Open Cup. Grand Prix of Koread」

- [GYM 102759](https://codeforces.com/gym/102759)
<!-- - [题面(英文)](./statement.html) -->
- [官方题解(英文)](./gp7sol.pdf)
- [jiangly大佬的题解](https://www.luogu.com.cn/blog/jiangly/xxi-open-cup-gp-of-korea)

Update 2021-4-18: 终于刷完了 QAQ。E,F,H 是之前写的，题解先咕了……

<!--more-->

![233](./233.png)

## A - Advertisement Matching

**[GYM102759A](https://codeforces.com/gym/102759/problem/A)**

### 题意

有 $n$ 个广告商，第 $i$ 个有 $a_i$ 份广告，有 $m$ 个人，第 $i$ 个人最多看 $b_i$ 份**不同**的广告。$q$ 次修改，每次将 $a_x\pm 1$ 或者 $b_x\pm 1$，修改是**累计的**。问每次修改后是否能将所有广告发完。（保证所有 $a_i,b_i$ 在任意时刻为非负整数）

$$
1\le n,m,q\le 250000,0\le a_i,b_j\le 250000
$$

### 题解

这显然有一个网络流做法，给 $n$ 个广告商分别建一个点，源点向其连容量 $a_i$ 的边，称为 A 类点，再给 $m$ 个人分别建一个点，向汇点连 $b_i$ 的边，称为 B 类点。两类点间两两连容量为 $1$ 的边，跑最大流。只要最大流等于 $\sum a_i$ 就可行。

然而并不需要跑最大流，这张图很特殊，我们可以「手算」。

为了方便，先将 $a_i$ **降序排序**。

**最大流等于最小割**，对于任意一种割，设割完后，与源点联通的 A 类点构成点集 $S$，与源点联通的 B 类点构成点集 $T$。显然当前割的大小为：

$$
\sum_{i\notin S} a_i +\sum_{j\in T}b_j +|S|(m-|T|)
$$

我们要的是**最小**割不妨枚举 $|S|=k$:

$$
\min_{k=0}^n \left\{
\sum_{i\notin S} a_i +\sum_{j\in T}b_j +k(m-|T|) \right\}
$$

因为 $a_i$ 已经降序，又要最小，显然 $S$ 不选是最后 $n-k$ 个。

$$
\min_{k=0}^n \left\{
\sum_{i=k+1}^n a_i +\sum_{j\in T}b_j +\sum_{j\notin T}k \right\} \\
$$

等价于

$$
\min_{k=0}^n \left\{
\sum_{i=k+1}^n a_i +\sum_{j=1}^m\min(b_j,k)  \right\} \\
$$

需要最小割（上式）等于 $\sum a_i$。化简一下即：

$$
\forall k\in[0,n] : \sum_{i=1}^m \min(b_i,k) \ge \sum_{i=1}^k a_i
$$

考虑这玩意怎么维护，令 $c_k=\sum_{i=1}^m \left[b_i\le k\right]$ 则：

$$
\forall k\in[0,n]:\sum_{i=1}^k c_i-a_i \ge 0
$$

线段树维护 $c_i-a_i$ 的前缀和的 $\min$ 即可。

对于修改操作，修改 $b$ 挺好搞的，线段树上对应 $[b_x,n]$ 区间加/减。若修改 $a$，要维护 $a$ 的不降序列 $d$，若 $+1$ 则在 $d$ 中找到最左边等于 $a_x$ 的值 $+1$，若 $-1$ 则在 $d$ 中找到最右边等于 $a_x$ 的值 $-1$。在线段树上做对应的区间修改。

### CODE

```cpp
#include <bits/stdc++.h>
#define int long long 
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 250010;
int a[N],b[N],n,m,q;

namespace seg {
    #define ls (x<<1)
    #define rs (x<<1|1)
    #define mid ((l+r)>>1)
    int mn[N<<2],tag[N<<2];
    void pushup(int x) {
        mn[x]=min(mn[ls],mn[rs]);
    }
    void update(int x,int vl){
        tag[x]+=vl; mn[x]+=vl;
    }
    void pushdown(int x) {
        if(tag[x]==0) return ;
        update(ls,tag[x]);update(rs,tag[x]);tag[x]=0;
    }
    void modify(int L,int R,int vl,int l=1,int r=n,int x=1) {
        if(L>R) return ;
        if(L<=l&&r<=R) return update(x,vl);
        pushdown(x);
        if(L<=mid) modify(L,R,vl,l,mid,ls);
        if(R>mid) modify(L,R,vl,mid+1,r,rs);
        pushup(x);
    }
    void build(int a[],int l=1,int r=n,int x=1) {
        if(l==r) {
            mn[x]=a[l];
            return ;
        }    
        build(a,l,mid,ls); build(a,mid+1,r,rs);
        pushup(x);
    }
    #undef ls
    #undef rs
    #undef mid
}
int c[N],d[N];
int aa[N];
signed main() {
    read(n,m);
    for(int i=1;i<=n;++i) read(a[i]),aa[i]=-a[i];
    for(int i=1;i<=m;++i) read(b[i]);
    sort(aa+1,aa+n+1);
    for(int i=1;i<=m;++i) c[min(n,b[i])]++;
    for(int i=n-1;i>=1;--i) c[i]+=c[i+1];
    for(int i=2;i<=n;++i) c[i]+=c[i-1];
    for(int i=1;i<=n;++i) {
        d[i]=d[i-1]+aa[i];
        c[i]+=d[i];
    }
    seg::build(c);
    read(q);
    for(int ts=1;ts<=q;++ts) {
        int op,x;
        read(op,x);
        if(op==1) {
            int ps=lower_bound(aa+1,aa+n+1,-a[x])-aa;
            aa[ps]--;
            seg::modify(ps,n,-1);
            a[x]++;
        }else if(op==2){
            int ps=upper_bound(aa+1,aa+n+1,-a[x])-aa-1;
            aa[ps]++;
            seg::modify(ps,n,1);
            a[x]--;
        }else if(op==3) {
            b[x]++;
            seg::modify(b[x],n,1);
        }else if(op==4){
            seg::modify(b[x],n,-1);
            b[x]--;
        }
        printf("%d\n",seg::mn[1]>=0);
    }
    return 0;
}
```

---

## B - Cactus Competition

**[GYM102759B](https://codeforces.com/gym/102759/problem/B)**

### 题意

给你一个长度为 $N$ 的序列 $\left\{A_i\right\}$ 和一个长度为 $M$ 的序列 $\left\{B_i\right\}$，表示一个 $N\times M$ 的网格图，其中 $(i,j)$ 格子的权值为 $A_i+B_j$。一个人从 $(S,1)$ 出发到 $(T,M)$，每次只能从 $(i,j)$ 到 $(i+1,j)$ 或 $(i,j+1)$ ，且不能经过 $A_i+B_j<0$ 的格子。问有多少对可行的 $S,T$。
$$
1\le N,M\le 2\times 10^5,-10^9\le A_i,B_i\le 10^9
$$

### 题解

显然任意两行间可以走的格子是包含关系，任意两列同理。

考虑如何判定 $(1,1)$ 能否到 $(N,M)$ ，若出现以下情况必然不行：

1. 某一行全不行，即 $\min\{A_i\}+\max\{B_i\}<0$ ；
2. 某一列全不行，即 $\max\{A_i\}+\min\{B_i\}<0$；
3. 起点被隔开，即存在 $(r,c)$ ，$\forall i\in[1,c]:A_r+B_i<0$ 且 $\forall i\in[1,r]: A_i+B_c<0$；
4. 终点被隔开，即存在 $(r,c)$ ，$\forall i\in[c,M]:A_r+B_i<0$ 且 $\forall i\in[r,N]: A_i+B_c<0$。

然后可以证明若上述情况不存在则一定可以，感性理解的话~~显然~~。具体证明见：[link](https://www.luogu.com.cn/blog/jiangly/xxi-open-cup-gp-of-korea)。

回到原问题，限制一和限制二可以确定每个起点对应终点的区间（双指针随便搞）。限制三：考虑枚举行 $x$，找最小的 $j$ 满足 $A_x+B_j\ge 0$，再令 $y\;(y\in[1,j-1])$ 使 $B_y$ 最小，再向上找第一个 $p$ 满足 $A_p+B_y\ge 0$，此时 $[p+1,x]$ 的起点全部不行（以上操作排序+倍增即可）。限制四同理。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 200010;
const int M = 21;
int A[N],B[N],n,m;
int t[N],mnB[2][N];
int premx[M][N],sufmx[M][N];
int vs[2][N];
int sum[N];

int main() {
    read(n,m);
    for(int i=1;i<=n;++i) read(A[i]);
    for(int i=1;i<=m;++i) read(B[i]);
    if(*max_element(A+1,A+n+1)+*min_element(B+1,B+m+1)<0) {
        puts("0");return 0;
    }
    mnB[0][1]=1;
    for(int i=2;i<=m;++i) {
        mnB[0][i]=mnB[0][i-1];
        if(B[i]<B[mnB[0][i]]) mnB[0][i]=i;
    }
    mnB[1][m]=m;
    for(int i=m-1;i>=1;--i) {
        mnB[1][i]=mnB[1][i+1];
        if(B[i]<B[mnB[1][i]]) mnB[1][i]=i;
    }
    for(int i=1;i<=n;++i) t[i]=i;
    sort(t+1,t+n+1,[](int x,int y){
        return A[x]>A[y];
    });    
    for(int i=1;i<=n;++i) sufmx[0][i]=premx[0][i]=A[i];
    for(int i=1;i<M;++i) {
        for(int j=1;j<=n;++j) {
            if(j-(1<<i)+1<1) continue;
            premx[i][j]=max(premx[i-1][j],premx[i-1][j-(1<<(i-1))]);
        }
        for(int j=1;j<=n;++j) {
            if(j+(1<<i)-1>n) continue;
            sufmx[i][j]=max(sufmx[i-1][j],sufmx[i-1][j+(1<<(i-1))]);
        }
    }
    for(int i=1,j=0;i<=n;++i) {
        int x=t[i],p=x;
        while(j+1<=m&&B[j+1]+A[x]<0) ++j;
        if(j==0) continue;
        int y=mnB[0][j];
        for(int i=M-1;i>=0;--i) if((p-(1<<i)+1>=1)&&premx[i][p]+B[y]<0) p-=(1<<i);
        // p+1 ~ x 
        vs[0][p+1]++;
        vs[0][x+1]--;
    }

    for(int i=1,j=m+1;i<=n;++i) {
        int x=t[i],p=x;
        while(j-1>=1&&B[j-1]+A[x]<0) --j;
        if(j==m+1) continue;
        int y=mnB[1][j];
        for(int i=M-1;i>=0;--i) if((p+(1<<i)-1<=n)&&sufmx[i][p]+B[y]<0) p+=(1<<i);
        // x ~ p-1
        vs[1][x]++;
        vs[1][p]--;
    }

    for(int i=1;i<=n;++i) {
        vs[0][i]+=vs[0][i-1];
        vs[1][i]+=vs[1][i-1];
        sum[i]=sum[i-1]+(vs[1][i]==0);
    }
    int MXB=*max_element(B+1,B+m+1),MNB=*min_element(B+1,B+m+1);
    ll ans=0;
    int l=1,r=1;
    for(int i=1;i<=n;++i) if(vs[0][i]==0&&A[i]+B[1]>=0){
        if(r<=i) {
            r=i;
            while(r+1<=n&&A[r+1]+MXB>=0) ++r; 
        }
        if(l<=i) {
            l=i;
            while(l<=n&&A[l]+MNB<0) ++l;
        }
        if(l<=r) {
            ans+=sum[r]-sum[l-1];
        }
    }
    printf("%lld\n",ans);
    return 0;
}
```

---

## C - Economic One-way Roads

**[GYM102759C](https://codeforces.com/gym/102759/problem/C)**

### 题意

给你 $N$ 个点，需要你加边使其变成**强连通图**，加边有限制，对于两点 $x,y(x<y)$：

- 告诉你 $x,y$ 直接不允许连边
- $x\to y$ 代价为 $a_{x,y}$，$y\to x$ 代价 $a_{y,x}$，你**必须只能**连一条边，且花费相应的代价。

问你最小代价，或判无解。$N\le 18$

### 题解

看这数据范围，考虑状压。

**耳分解**：

> 有向图  $G=(V,E)$ 强连通当且仅当可以通过以下方法构造：
>
> 1. 任选一个点 $s$，令 $S=\left\{s\right\}$。重复以下过程直到 $S=V$。
> 2. 任选两个点 $u,v\in S$。$u$ 和 $v$ 可以相同。
>    1. 选择 $k\pod{k\ge 0}$ 个不同的点 $w_1,w_2,\ldots,w_k\not\in S$。
>    2. 连接 $u\to w_1\to w_2\to\cdots\to w_k\to v$，并将 $w_1,w_2,\ldots ,w_k$ 加入 $S$。
>
> 路径 $u\to w_1\to w_2\to \cdots \to w_k\to v$ 被称作耳。

设 $f(S)$ 表示 $S$ 中的构成强联通图的最小花费，$dp(S,u,w)$ 表示 $S$ 中的点要么是强联通的，要么是正在构造的「耳」，$u,w\in S$ ，$u$ 表示「耳」的当前点，$w$ 表示「耳」的期望终点（故 $w$ 是在 $S$ 中强联通分量里的）。如左图所示 黑色的环表示强联通分量，红色的部分为正在构造的「耳」。

![P1](./P1.png)

枚举不在 $S$ 中的点 $x$ 进行转移，强制选 $u\to x$ 的边，若选 $x\to w$ 的边则更新 $f$（已经构建完「耳」），否则更新 $dp$。$u$ 和 $w$ 可以重合需要特判。

### CODE

```cpp
#pragma GCC optimize(2)
#pragma GCC optimize(3)
#pragma GCC optimize(3,"Ofast","inline")
#include <bits/stdc++.h>
#define re register
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 19;
const int M = 1 << 19;
const int inf = 0x3f3f3f3f;
/*
dp[mask][u][w]
表示 当前的强联通图和耳在 mask 中
当前耳朵构造到 u 点，终止节点为 w

f[mask] 表示 mask 这些点构成强联通图最小代价。

*/
int a[N][N],dp[M][N][N],f[M];
int n,ful,cst[M][N];
template<typename Tp> void umin(Tp &x,Tp y) {if(x>y)x=y;}

void pt(int mask) {
    for(int i=0;i<n;++i) {
        putchar('0'+((mask>>i)&1));
    }
    
}
int main() {
    read(n); ful=(1<<n)-1;
    for(int i=0;i<n;++i) for(int j=0;j<n;++j) read(a[i][j]);
    for(re int mask=1;mask<=ful;++mask) {
        for(re int x=0;x<n;++x) if(!((mask>>x)&1)) {
            for(re int v=0;v<n;++v) if((mask>>v)&1) {
                if(a[x][v]==-1) continue;
                cst[mask][x]+=min(a[x][v],a[v][x]);
            }
        }
    }
    memset(dp,0x3f,sizeof(dp));
    memset(f,0x3f,sizeof(f));
    for(int i=0;i<n;++i) f[1<<i]=dp[1<<i][i][i]=0;
    for(re int mask=1;mask<=ful;++mask) {
        for(re int i=0;i<n;++i) if((mask>>i)&1){
            for(re int j=0;j<n;++j) if((mask>>j)&1) {
                umin(dp[mask][i][j],f[mask]);
                // 用 f 更新 dp 表示新开一个耳
            }
        }
        for(re int u=0;u<n;++u)if((mask>>u)&1) {
            for(re int w=0;w<n;++w)if((mask>>w)&1) {
                if(dp[mask][u][w]>=inf) continue;
                for(re int x=0;x<n;++x) if(!((mask>>x)&1)) {
                    if(a[u][x]==-1) continue;
                    int tmp=mask;
                    if((tmp>>u)&1) tmp^=(1<<u);
                    if((tmp>>w)&1) tmp^=(1<<w);
                    int cost=cst[tmp][x]+a[u][x],nxt=mask|(1<<x);
                    if(u==w) {
                        umin(dp[nxt][x][w],dp[mask][u][w]+cost);
                    }else {
                        if(a[x][w]!=-1) {
                            umin(f[nxt],dp[mask][u][w]+cost+a[x][w]);
                            umin(dp[nxt][x][w],dp[mask][u][w]+cost+a[w][x]);
                        }else {
                            umin(dp[nxt][x][w],dp[mask][u][w]+cost);
                        }
                    }
                }
            }
        }
    }
    if(f[ful]>=inf) puts("-1");
    else printf("%d\n",f[ful]);
    return 0;
}
```

---

## D - Just Meeting

**[GYM102759D](https://codeforces.com/gym/102759/problem/D)**

### 题意

给你一张 $n$ 个点，$m$ 条边的无向图，带边权，不保证联通，需要你将它补成一张完全图（原来不存在的边由你钦定边权，原本存在的边不能改动），边权范围 $[1,10^7]$ 。这张图需要满足：对于任意三元环，不能存在一条边严格小于另外两条边。最小化边权之和。

$$
3\le n\le 3\times 10^5,0\le m\le 3\times 10^5
$$

### 题解

首先，对于不在同一联通块中的两点 $u,v$，我们直接连边权为 $1$ 就行了。

证明：考虑 $u,v$ 和另外一点 $w$ 构成的三元环。若 $w$ 不属于 $u,v$ 任意一个联通块，那三条边必然都是 $1$，满足条件。若 $w$ 属于 $u$ 或 $v$ 的联通块，那至少两条边是 $1$，肯定也满足条件。且这样放已经让边权最小了。

故我们只需要考虑单独一个联通块中的情况。

不妨先考虑树。对于所有距离为二的点 $u,v$，我们钦定边权为 $u,v$ 路径上的最小边权（若 $u,v$ 路径上那两条边不等，那只能这样。若相等，显然取小一点更优）。接着所有距离大于二的点，钦定其边权为路径上的最小边权，可以归纳证明这一定是合法的。

对于任意图，我们整一个最大生成树。然后同理，若非树边不等于我们期望的值则输出无解。

证明：考虑最大生成树的性质，对于非树边 $u,v$ 该边权一定小于等于树上 $u,v$ 路径上的最小边。若该边等于 $u,v$ 路径上的最小边，那它符合我们的构造。若小于，那必然会出现不合法三元环。

考虑如何求答案。对于同一联通块中边，考虑最大生成树上每条边的贡献。~~不会要 LCT 吧~~ 倒着加边并查集维护就行了。对于不同联通块中边，总边数减去所有联通块中连的边数。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 300010;
const int M = 21;
struct edge{
    int x,y,w;
}e[N],ee[N];int tote;
bool tag[N],vs[N];
int head[N],pnt[N<<1],nxt[N<<1],wth[N<<1],E;
int far[N],sz[N],dep[N],fa[M][N],mn[M][N];
int n,m;
ll ans;
void adde(int u,int v,int w) {
    ++E;pnt[E]=v;nxt[E]=head[u];wth[E]=w;head[u]=E;
}
int find(int u) {
    return far[u]==u?u:far[u]=find(far[u]);
}
void ut(int u,int v) {
    u=find(u);v=find(v);
    sz[v]+=sz[u];far[u]=v;
}
void MST() {
    sort(e+1,e+m+1,[](const edge &a,const edge &b){return a.w>b.w;});
    for(int i=1;i<=n;++i) far[i]=i;
    for(int i=1;i<=m;++i) {
        if(find(e[i].x)!=find(e[i].y)) {
            tag[i]=1; ee[++tote]=e[i];
            ut(e[i].x,e[i].y);
            adde(e[i].x,e[i].y,e[i].w);
            adde(e[i].y,e[i].x,e[i].w);
        }
    }
}
int cnt;
void dfs(int u,int f,int d) {
    ++cnt;
    fa[0][u]=f;dep[u]=d;
    for(int i=1;i<M&&fa[i-1][u];++i) {
        fa[i][u]=fa[i-1][fa[i-1][u]];
        mn[i][u]=min(mn[i-1][u],mn[i-1][fa[i-1][u]]);
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        mn[0][v]=wth[i];
        dfs(v,u,d+1);
    }
}
int minedge(int u,int v) {
    if(dep[u]<dep[v]) swap(u,v);
    int r=0x3f3f3f3f,d=dep[u]-dep[v];
    for(int i=0;i<M;++i) if((d>>i)&1) {
        r=min(r,mn[i][u]);
        u=fa[i][u];
    }
    if(u==v) return r;
    for(int i=M-1;i>=0;--i) if(fa[i][u]!=fa[i][v]){
        r=min(r,min(mn[i][u],mn[i][v]));
        u=fa[i][u];v=fa[i][v];
    }
    return min(r,min(mn[0][u],mn[0][v]));
}
int main() {
    read(n,m);
    ans=(ll)n*(n-1)/2;
    for(int i=1;i<=m;++i) {
        read(e[i].x,e[i].y,e[i].w);
    }
    MST();
    for(int i=1;i<=n;++i) if(dep[i]==0) {
        cnt=0;dfs(i,0,1);
        ans-=(ll)cnt*(cnt-1)/2;
    }
    for(int i=1;i<=m;++i) if(!tag[i]){
        int w=minedge(e[i].x,e[i].y);
        if(w!=e[i].w) {puts("-1");return 0;}
    }
    for(int i=1;i<=n;++i) far[i]=i,sz[i]=1;

    for(int i=1;i<=tote;++i) {
        ans+=(ll)ee[i].w*sz[find(ee[i].x)]*sz[find(ee[i].y)];
        ut(ee[i].x,ee[i].y);
    }
    printf("%lld\n",ans);
    return 0;
}
```

---

## G - LCS 8

**[GYM102759G](https://codeforces.com/gym/102759/problem/G)**

### 题意

给你一个长度为 $N$ 的字符串 $S$（都是大写字母），再给你一个很小的 $K$，请你计算有多少个长度为 $N$ 的字符串 $T$ 满足其与 $S$ 的**最长公共子序列**至少为 $N-K$，答案对 $10^9+7$ 取模。

$$
1\le N \le 50000,0\le K\le 3
$$

### 题解

那么 **DP 套 DP**。​

假如我们得到了一个确定的 $T$，考虑我们怎么算 $S$ 与 $T$ 的 LCS。显然有简单 DP $f_{i,j}$ 表示 $S$ 中前 $i$ 位与 $T$ 中前 $j$ 位 LCS。
$$
f_{i,j}=\max\left\{f_{i-1,j},f_{i,j-1},f_{i-1,j-1}+[S_i=T_j]\right\}
$$
这复杂度 $O(n^2)$ 显然不行。

考虑题目有限制 LCS 大于等于 $n-K$ ，那么显然 $|i-j|>K$ 的 $f_{i,j}$ 是没用的，那么 DP 状态仅有 $f_{i,i-K},\dots,f_{i,i-1},f_{i,i},\dots,f_{i,i+K},i\in[1,n]$。

考察  $f_{i,i-K},\dots,f_{i,i+K}$ 的有用状态有多少个， 首先 $f_{i,i}\le i$ ，而且如果要整出答案 $f_{i,i}\ge i-K$ （即不能失配超过 $K$ 位） 。然后这些 $f$ 相邻差值最多为 $1$ ，差分一下必然是长度为 $2K$ 的 01 序列，状压即可。这样总情况数是 $O(2^{2K}(K+1))$ 的。

形式化地，记 $dp(i,j,mask)$ 表示枚举到 $T$ 的第 $i$ 位，$f_{i,i}=i-j$ ，$f_{i,i-K}\dots f_{i,i+K}$ 的差分序列为 $mask$，$T$ 构造的方案数。转移时，通过 $j,mask$ 对 $f_{i,i-K}\dots f_{i,i+K}$ 解压，枚举 $T$ 后放什么字符，算出 $f_{i+1,i+1-K}\dots f_{i+1,i+1+K}$，再压缩成 $j',mask'$ 更新 $dp(i+1,j',mask')$。

答案就是 $\sum_{j=0}^K dp(N,j,mask)$，复杂度 $O(N2^{2K}K^2|\Sigma|)$（$|\Sigma|$ 表字符集大小），松一松能过的（确信）。

### CODE

```cpp
#pragma GCC optimize(2)
#include <cstdio>
#include <cstring>
#define re register
const int N = 50010;
const int M = 1<<6;
const int mod = 1e9+7;
template<typename Tp> void Add(Tp &x,Tp y) {x=(x+y)<mod?(x+y):(x+y-mod);} 
template<typename Tp> inline Tp max(const Tp &x,const Tp &y) {return x>y?x:y;}
char S[N];
int n,k,ful,m,K;
int dp[2][4][M];
int f[2][8];
int main() {
    scanf("%s",S+1); n=strlen(S+1); scanf("%d",&k);
    bool V=0; m=6;K=3;
    dp[0][0][0]=1; ful=(1<<m)-1;
    for(re int i=1;i<=n;++i) {
        memset(dp[V^1],0,sizeof(dp[V^1]));
        for(re int mask=0;mask<=ful;++mask) for(re int p=0;p<=K;++p) if(dp[V][p][mask]){
            memset(f,0,sizeof(f)); 
            /*
                    0   1   2   3   4   5   6   7 
            f[i-1] i-4 i-3 i-2 i-1   
            f[i]                    i  i+1 i+2 i+3
            */
            for(re int j=0;j<m;++j) f[0][j+1]=f[0][j]+((mask>>j)&1);
            int dif=i-1-p-f[0][K];
            for(re int j=0;j<=m;++j) f[0][j]+=dif;

            for(re int c='A';c<='Z';++c) {
                int nxt=0,np=0;
                for(re int j=1;j<=m+1;++j) {
                    if(i+j-4<1) continue;
                    f[1][j]=max(f[0][j-1]+(S[i+j-4]==c),max(f[0][j],f[1][j-1]));
                }
                for(re int j=1;j<=m;++j) nxt|=(f[1][j+1]-f[1][j])?(1<<(j-1)):0;
                np=i-f[1][4];
                if(np<=K) Add(dp[V^1][np][nxt],dp[V][p][mask]);
            }        
        }
        V^=1;
    }
    int ans=0;
    for(re int p=0;p<=k;++p) {
        for(re int mask=0;mask<=ful;++mask) {
            Add(ans,dp[V][p][mask]);
        }
    }
    printf("%d\n",ans);
    return 0;
}
```

---

## I - Query On A Tree 17

**[GYM102759I](https://codeforces.com/gym/102759/problem/I)**

### 题意

给你一颗大小为 $N$ 的树，以 $1$ 为根，带点权，最初所有点点权为 $0$，有 $Q$ 次操作：

- `1 u` 给 $u$ 子树内所有点权值加 $1$；
- `1 u v` 给 $u$ 到 $v$ 最短路径上所有点权值加 $1$。
  
操作是累加的，每次操作后输出树的带权重心（若有多个输出离根最近的）

$$
2\le N \le 10^5,1\le Q\le 10^5
$$

### 题解

首先以带权重心为根，其最大子树小于等于总权值的一半，不然进入该子树一定更优。且若有多个重心，必然是树上一条链。

考虑如果确定以 $1$ 为根，随便选一个点 $u$ ，记 $S=\sum_{i=1}^n A_{i}$ ，若 $siz_u\le \dfrac{S}{2}$ ，那么父亲方向的子树大小为 $\dfrac{S}{2}-siz_u\ge\dfrac{S}{2}$ ，$u$ 跳到父亲更优。那我们要找的点满足 $siz_{u}>\dfrac{S}{2}$ 。

考虑将点的权值填到 DFN 序上，此时一个子树权值和对于序列上一段区间，由于我们要找的点 $x$ 满足 $siz_{x}>\dfrac{S}{2}$ ，对于序列上区间和 $>\dfrac{S}{2}$ ，那么这个区间的**带权中位数**所在的位置必然在这个区间中，在树上即在 $x$ 的子树中。

故我们需要维护子树权值和，DFN 序区间权值和，支持子树、链修改，每次二分找到区间中位数所在点 $u$，倍增跳到满足 $siz_x>\dfrac{S}{2}$ 的最深的点 $x$ 即可。维护信息树剖+线段树。复杂度 $O(n\log n+q\log^2n)$。

### CODE

```cpp
#include <bits/stdc++.h>
#define ls (x<<1)
#define rs (x<<1|1)
#define mid ((l+r)>>1)
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 100010;
const int M = 21;
int head[N],pnt[N<<1],nxt[N<<1],E;
int fa[M][N],dep[N],top[N],siz[N],son[N],id[N],rk[N],tim;
int n,q;
void adde(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void dfs1(int u,int f,int d) {
    fa[0][u]=f;dep[u]=d;siz[u]=1;
    for(int i=1;i<M&&fa[i-1][u];++i) fa[i][u]=fa[i-1][fa[i-1][u]];
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs1(v,u,d+1);
        siz[u]+=siz[v];
        if(siz[v]>siz[son[u]]) son[u]=v; 
    }
}
void dfs2(int u,int topf) {
    top[u]=topf;id[u]=++tim;rk[tim]=u;
    if(son[u]) {
        dfs2(son[u],topf);
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[0][u]||v==son[u]) continue;
        dfs2(v,v);
    }
}
ll sum[N<<2],tag[N<<2],len[N<<2];
void update(int x,ll vl) {
    tag[x]+=vl;sum[x]+=vl*len[x];
}
void build(int l=1,int r=n,int x=1) {
    len[x]=r-l+1;if(l==r) return ;
    build(l,mid,ls);build(mid+1,r,rs);
}
void pushdown(int x) {
    if(tag[x]) {update(ls,tag[x]);update(rs,tag[x]); tag[x]=0;}
}
void pushup(int x) {
    sum[x]=sum[ls]+sum[rs];
}
void modify(int L,int R,int l=1,int r=n,int x=1) {
    if(L<=l&&r<=R) return update(x,1);
    pushdown(x);
    if(L<=mid) modify(L,R,l,mid,ls);
    if(R>mid) modify(L,R,mid+1,r,rs);
    pushup(x);
}
ll query(int L,int R,int l=1,int r=n,int x=1) {
    if(L>R) return 0;
    if(L<=l&&r<=R) return sum[x];
    pushdown(x); ll ret=0;
    if(L<=mid) ret+=query(L,R,l,mid,ls);
    if(R>mid) ret+=query(L,R,mid+1,r,rs);
    return ret;
}
void Modify_tree(int u) {
    modify(id[u],id[u]+siz[u]-1);
}
void Modify_chain(int u,int v) {
    int fu=top[u],fv=top[v];
    while(fu!=fv) {
        if(dep[fu]<dep[fv]) swap(fu,fv),swap(u,v);
        modify(id[fu],id[u]);
        u=fa[0][fu];fu=top[u];
    }
    if(dep[u]<dep[v]) swap(u,v);
    modify(id[v],id[u]);
}

void Print(int l=1,int r=n,int x=1) {
    if(l==r) {
        printf("%lld ",sum[x]);
        return ;
    }
    pushdown(x);
    Print(l,mid,ls);Print(mid+1,r,rs);
}

int main() {
    read(n);
    for(int i=1;i<n;++i) {
        int u,v;read(u,v);
        adde(u,v);adde(v,u);
    }
    dfs1(1,0,0);dfs2(1,1);build();
    read(q);
    for(int ts=1;ts<=q;++ts) {
        int op,x,y;read(op);
        if(op==1) {
            read(x);Modify_tree(x);
        }else {
            read(x,y);Modify_chain(x,y);
        }
        ll S=query(1,n);
        int L=1,R=n,B,Mid;
        while(L<=R) {
            Mid=(L+R)>>1;
            ll t=query(1,Mid);
            if(t*2ll>=S) {
                B=Mid;R=Mid-1;
            }else L=Mid+1;
        }
        int u=rk[B];
        for(int i=M-1;i>=0;--i){
            int f=fa[i][u];
            if(f&&query(id[f],id[f]+siz[f]-1)*2<=S) u=f;
        }
        if(query(id[u],id[u]+siz[u]-1)*2<=S) u=fa[0][u];
        printf("%d\n",u);
    } 
    return 0;
}
```

---

## J - Remote Control

**[GYM102759J](https://codeforces.com/gym/102759/problem/J)**

### 题意

题目讲得挺清楚的。QAQ（~~我懒~~）

### 题解

考虑若不撞墙，可以直接算出终点，若知道在操作序列的第几次撞墙，那么终点也唯一确定了。

考虑预处理这些，需要知道 某个位置出发会在第几次操作撞墙、在某次操作撞墙后终点是啥。

前者简单，操作序列反向（`L` 变 `R`,`U` 变 `D` ...），走到的位置若没标记则记上当前操作位置。

对于后者倒着处理，假设已经处理好 $i$ 之后每次操作撞墙后的终点，并维护了只管 $i$ 之后的操作序列，那些位置会在操作序列第几次撞墙。不断迭代计算就可以了。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 300010;
struct pt {
    int x,y;
    pt(int _x=0,int _y=0) {x=_x;y=_y;}
    bool operator<(const pt &a) const{return (x!=a.x)?(x<a.x):(y<a.y);}
};
const int fig[4][2]={{1,0},{-1,0},{0,1},{0,-1}};// R L U D
char op[N];
int n,q,dx[N],dy[N];
map<pt,int> cr,pre; // cr(pt) 某个位置出发会在第几步撞墙
pt go[N]; // go[i] 第 i 个命令撞到墙后何去何从
int id(char c) {
    if(c=='R') return 0;
    if(c=='L') return 1;
    if(c=='U') return 2;
    if(c=='D') return 3;
    return -1;
}
void init() {
    for(int i=1;i<=n;++i) {
        dx[i]=dx[i-1]+fig[id(op[i])][0];
        dy[i]=dy[i-1]+fig[id(op[i])][1];
    }
    int tx=-dx[n],ty=-dy[n];

    for(int i=n;i>=1;--i) {
        cr[pt(tx,ty)]=i;
        tx+=fig[id(op[i])][0];
        ty+=fig[id(op[i])][1];
    }
    int ex=0,ey=0;// 源点坐标
    for(int i=n;i>=1;--i) {
        // pre(pt) 表忽略 [0,i-1] 的操作，哪些点会在第几步撞墙
        tx=-fig[id(op[i])][0];ty=-fig[id(op[i])][1];
        // 第 i-1 步在 (tx,ty) ，第 i 步还在 (tx,ty) 
        if(pre.find(pt(tx+ex,ty+ey))!=pre.end()) {
            go[i]=go[pre[pt(tx+ex,ty+ey)]];
        }else {
            go[i]=pt(tx+dx[n]-dx[i],ty+dy[n]-dy[i]);
        }
        ex-=tx;ey-=ty;
        pre[pt(ex+tx,ey+ty)]=i;
    }
}
int main() {
    read(n);
    scanf("%s",op+1);
    init();
    read(q);

    for(int i=1;i<=q;++i) {
        int x,y;read(x);read(y);
        if(cr.find(pt(x,y))!=cr.end()) {
            int t=cr[pt(x,y)];
            printf("%d %d\n",go[t].x,go[t].y);
        }else {
            printf("%d %d\n",x+dx[n],y+dy[n]);
        }
    }
    return 0;
}
```

---

## K - Sewing Graph

**[GYM102759K](https://codeforces.com/gym/102759/problem/K)**

### 题意

$n$ 个点的桌布（点互不相同），让你构造一个长度为 $k$ 的序列 $s$ 满足下列条件

在桌布正面，用一条无向边连通 $s_{2i-1}$ 和 $s_{2i},1\le i\le \left\lfloor \frac{k}{2}\right\rfloor$  
在桌布背面，用一条无向边连通 $s_{2j}$ 和 $s_{2j+1},1\le j\le \left\lfloor \frac{k-1}{2}\right\rfloor$

要求，同一面的 $n$ 个点都被这一面上的边连通，同一面的边只能在顶点处相交。输出最小的 $k$ 和此时的序列

$n\le 1000$

### 题解

这显然是最水的题了，$k=2n-1$ 显然，考虑一种螺旋构造：

![螺旋构造](./PP.png)

做完了

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
typedef double db;
const int N = 1005;
struct vec{
    ll x,y;
    vec(){}
    vec(ll _x,ll _y):x(_x),y(_y){}  
    vec operator+(const vec &t) {return vec(x+t.x,y+t.y);}
    vec operator-(const vec &t) {return vec(x-t.x,y-t.y);}
    ll len2() {return x*x+y*y;}
    bool operator<(const vec &t) {return (x==t.x)?(y<t.y):(x<t.x);}
};
ll crs(const vec &a,const vec &b) {return a.x*b.y-a.y*b.x;}
ll dot(const vec &a,const vec &b) {return a.x*b.x+a.y*b.y;}
vec pts[N],tmp[N];
bool vs[N];
int n,s[N];
int main() {
    read(n);
    for(int i=1;i<=n;++i) {
        read(pts[i].x);read(pts[i].y);
    }
    int st=1;
    for(int i=1;i<=n;++i) {
        if(pts[i]<pts[st]) st=i;
    }
    s[1]=st;vs[st]=1;
    for(int i=2;i<=n;++i) {
        int nxt=-1;
        for(int j=1;j<=n;++j) if(!vs[j]){
            tmp[j]=pts[j]-pts[st];
            if(nxt==-1||crs(tmp[j],tmp[nxt])<0||(crs(tmp[j],tmp[nxt])==0&&tmp[j].len2()<tmp[nxt].len2())) {
                nxt=j;
            }
        }
        vs[nxt]=1; s[i]=nxt;
        st=nxt;
    }
    printf("%d\n",2*n-1);
    for(int i=1;i<=n;++i) printf("%d ",s[i]);
    for(int i=n-1;i>=1;--i) printf("%d ",s[i]);
    puts("");
    return 0;
}
```

---

## L - Steel Slicing 2

**[GYM102759L](https://codeforces.com/gym/102759/problem/L)**

### 题意

![prob](./prob.png)

给你一个长度为 $n$ 的序列 $\{h_i\}$ 和 $\{l_i\}$ 表示上图所示的栅栏（所有边都是水平或竖直的），切一刀从某边界点出发**水平**或**竖直**知道边界点结束。问至少几刀使原图只留下矩形。

### 题解

其实就是给你一个凹多边形，让你把它变成凸多边形（只有水平、竖直边的多边形，若是凸的必然是矩形）。称向内凹的点为凹点，向外凸的点为凸点。一次切割只会改变切割边界点的性质，显然不会切割凸点，若割一个凹点会将其变成一个凸点+一个平面，若割到平面会将其变成两个凸点。当所有凹点都被删完自然就结束了。

考虑一次切割至少删除一凹点，若要删除两个凹点这两凹点必须在同一水平、竖直线上且之间没有经过边界。需要最大化删除两个凹点的操作次数。这些可能的操作很容易预处理，竖直的枚举一遍，水平的用单调栈。

并不是所有操作都可以随便选，如下图所示，红色的水平操作与蓝色的竖直操作向交，执行完一个，另一个就不行了。故给每个操作建点，相交的操作连边，要求最大独立集。考虑该图是二分图（水平操作互相不交，竖直操作互相不交），二分图中最大独立集等于总点数减**最大匹配**。

![P2](./P2.png)

跑网络流？不需要，考虑一个 $l$ 到 $r$ 的水平操作只会与 $[l,r]$ 内的竖直操作有交，可以把竖直操作看成点，水平操作为区间，点匹配区间，贪心即可。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 500010;
struct pp{int l,r;};
ostream &operator<<(ostream &out,const pp &p) {
    return out<<"["<<p.l<<", "<<p.r<<"]";
}
bool operator<(const pp &a,const pp &b) {
    return a.r>b.r;
}
int n,u[N],d[N];
int cnt; // cnt 总共的凹点数
vector<int> pts; // 竖切操作
vector<pp> lins; // 横切操作 
int stk[N],top;
int h[N];
int main() {
    read(n);
    for(int i=1;i<=n;++i) read(u[i],d[i]);
    for(int i=2;i<=n;++i) {
        if(u[i]!=u[i-1]) ++cnt;
        if(d[i]!=d[i-1]) ++cnt;
        if(u[i]!=u[i-1]&&d[i]!=d[i-1]) {
            pts.push_back(i);
        }
    }
    for(int i=2;i<=n;++i) {
        if(u[i-1]==u[i]) continue;
        h[i]=min(u[i-1],u[i]);
        if(u[i-1]<u[i]) {
            stk[++top]=i;
        }else {
            while(top>0&&h[stk[top]]>h[i]) --top;
            if(top>0&&h[stk[top]]==h[i]) {
                lins.push_back(pp{stk[top],i});               
                --top;
            }
        }
    }
    top=0;
    for(int i=2;i<=n;++i) {
        if(d[i-1]==d[i]) continue;
        h[i]=min(d[i-1],d[i]);
        if(d[i-1]<d[i]) {
            stk[++top]=i;
        }else {
            while(top>0&&h[stk[top]]>h[i]) --top;
            if(top>0&&h[stk[top]]==h[i]) {
                lins.push_back(pp{stk[top],i});               
                --top;
            }
        }
    }
    priority_queue<pp> Q;
    int mch=0;
    sort(lins.begin(),lins.end(),[](pp a,pp b) {return a.l<b.l;});
    int j=0;
    for(int i=0;i<(int)pts.size();++i) {
        while(j<(int)lins.size()&&lins[j].l<=pts[i]) {
            Q.push(lins[j]); ++j;
        }
        while(!Q.empty()&&Q.top().r<pts[i]){
            Q.pop();           
        }
        if(!Q.empty()&&Q.top().r>=pts[i]) {
            ++mch;Q.pop();
        }
    }
    cout<<cnt-(pts.size()+lins.size()-mch)<<endl;
    return 0;
}
```
