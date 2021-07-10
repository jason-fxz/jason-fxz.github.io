---
title: 'XJ contest1587 『树』（牛客『路径计数机』）'
date: 2020-08-17 22:01:56
tags: [LCA,树上差分]
categories: 题解
hidden: true
---


[XJ contest1587『树』](http://115.236.49.52:83/contest/1587/problem/2)
[牛客『路径计数机』](https://ac.nowcoder.com/acm/problem/53999)
树上差分，LCA
<!--more-->


## 题意

给你一棵 $n$ 个点的树和两个整数 $p, q$，求满足以下条件的四元组 $(a, b, c, d)$ 的个数：

1. $1\le a,b,c,d \le n$
2. 点 $a$ 到点 $b$ 的距离为 $p$
3. 点 $c$ 到点 $d$ 的距离为 $q$
4. 不存在一个点，它既在 $a$ 到 $b$ 的路径上，又在 $c$ 在 $d$ 的路径上

**注：数据范围** 

| 测试点编号 |    $n$     |   $p,q$    |  特殊性质  |
| :--------: | :--------: | :--------: | :--------: |
|     1      |  $\le 30$  |  $\le 30$  |     无     |
|     2      |  $\le 50$  |  $\le 50$  |     无     |
|    3，4    | $\le 200$  | $\le 200$  |     无     |
|     5      | $\le 3000$ |    $=2$    |     无     |
|     6      | $\le 3000$ | $\le 3000$ | 树是一条链 |
|     7      | $\le 3000$ | $\le 3000$ | 树随机生成 |
|  8，9，10  | $\le 3000$ | $\le 3000$ |     无     |

对于所有数据，满足 $1\le n,p,q\le 3000$ 。

## 思路

### ~~暴力骗分~~(比赛时人傻，只想到骗分)

首先能想到 $O(n^4)$ 暴力，枚举 $a,c$ 然后两个DFS暴力找路径。对于树是一条链也是一道简单数学题，$O(n)$ 即可，XJ的数据有点水，这样就能过 1~4,6 共五个点。

### 正解

看数据范围 $n\le 3000$ ，能猜测到这大概是一个 $O(n^2)$ 的算法，可以考虑枚举 $a,b$ 。它的贡献就等于 （***所有长度为 $q$ 的路径的数量-不合法的数量***） 。前者$O(n^2)$ 枚举即可，考虑如何求后者:  设$a,b$的$\rm{LCA}$ 为$u$；$c,d$的$\rm{LCA}$为 $v$ 

**分成两类求：**

第一种v在u的子树内：可以发现当且仅当v在a->b 这条路径上才有交（若v不在a->b这条路径上，那比v更深的点显然也不会在a->b的路径上）

![](1597671917061.png)



Tarjan预处理每对点的$\rm{LCA}$，复杂度 $O(n^2)$：可以用 $f[u]$ 表示以 $u$ 为 $\rm{LCA}$ 的长度为 $q$ 的路径有几条。预处理 $f[u]$ 直接 $O(n^2)$ 枚举 $c,d$ 若路径长度为 $q$，$f[\;\rm{LCA}(c,d)\;]++$ 即可。

对于一对 $(a,b)$, $v$ 在 $u$ 的子树内不合法的数量即为 $a$ 到 $b$  路径上的 $f$ 之和。


第二种 $v$ 在 $u$ 的子树外：若有交显然会经过 $u$ 到其父亲的那条边， 我们只要再算一个 $g[u]$  表示经过 $u$ 到其父亲那条边的长度为 $q$ 的路径数。这东西跟 $f$ 差不多，枚举 $c,d$ ，若路径长度为$q$ ，则 $c$ 到 $d$ 的路径上所有边的 $g$ 加一，可以树上差分。

![](1597671930132.png)


**So:** 一对合法 $(a,b)$ 的贡献为：
$$
Sumf-g[\;LCA(a,b)\;]-\sum_{u\in path(a,b)} f[u]
$$
$Sumf$ 为所有 $f$ 之和。最后那一坨 $f$ 树上差分就行了。

总复杂度 $O(n^2)$

## code

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 3005;
typedef long long ll;
int head[N],nxt[N<<1],pnt[N<<1],E;
int f[N],g[N],Sf,ts[N];
int faa[N],lca[N][N],dep[N],fa[N],n,p,q;
bool vis[N];ll ans;
void add(int u,int v) {E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;}
int findfa(int u) {return u==faa[u]?u:faa[u]=findfa(faa[u]);}
void ut(int u,int v) {faa[findfa(u)]=findfa(v);}
int dist(int u,int v) {return dep[u]+dep[v]-2*dep[lca[u][v]];}
void tarjan(int u) {
    vis[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(vis[v]) continue;
        dep[v]=dep[u]+1;fa[v]=u;
        tarjan(v);ut(v,u);
    }
    for(int v=1;v<=n;v++) if(v!=u){
        if(vis[v]) lca[v][u]=lca[u][v]=findfa(v);
    }
}
void dfs(int u) {
    Sf+=f[u];
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u]) continue;
        dfs(v);g[u]+=g[v];
    }
}
void init() {
    for(int i=1;i<=n;i++) faa[i]=i;
    for(int i=1;i<=n;i++) lca[i][i]=i;
    tarjan(1);
    for(int i=1;i<=n;i++)
    for(int j=1;j<=n;j++) if(dist(i,j)==q) {
        f[lca[i][j]]++;
        g[i]++,g[j]++;
        g[lca[i][j]]-=2;
    }   
    dfs(1);
}
void solve(int u) {
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u]) continue;
        solve(v);
        ts[u]+=ts[v];
    }
    ans-=1ll*ts[u]*f[u];
}
int main() {
    scanf("%d%d%d",&n,&p,&q);
    for(int i=1;i<n;i++) {
        int u,v;
        scanf("%d%d",&u,&v);
        add(u,v),add(v,u);
    }
    init();
    for(int i=1;i<=n;i++) {
        for(int j=1;j<=n;j++) if(dist(i,j)==p) {
            ans+=Sf-g[lca[i][j]];
            ts[i]++,ts[j]++;
            ts[lca[i][j]]--;
            ts[fa[lca[i][j]]]--;
        }
    }
    solve(1);
    printf("%lld\n",ans);
}
```
