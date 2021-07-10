---
title: '神J上树'
date: 2020-09-27 22:30:37
tags: [重链剖分,单调栈,倍增]
categories:
- 题解
---
[『牛客』神J上树](https://ac.nowcoder.com/acm/problem/54067)


## 题意简述

给一棵 $n$ 个点的树，每条边有边权 $w_i$，定义 $dis(u,v)$ 为 $u$ 到 $v$ 简单路的长度。神J可以在树上移动，且每次只能从一个节点 $u$ 跳向其子孙节点 $v$ ，且代价为 $u\times dis(u,v)$ 。给出 $m$ 个询问，问是否能从 $s$ 跳到 $t$ ，若能，求最小代价。

$n,m\le 3\times 10^5,1\le w_i\le 10^7$

<!--more-->

## 题解

判断 $s$ 是不是 $t$ 的祖先，用dfn序即可。 

再观察以下，显然最小代价是要**每一个点跳到第一个比它小的点**，最后再一步跳到 $t$ 。

我们不妨先考虑在序列上怎么做： 处理每个点后第一个比它小的点可以用单调栈扫一遍，然后这东西可以倍增，预处理出从某个点跳 $2^j$ 到达点及这一段的代价。 在查答案时从 $s$ 开始跳，跳到不越过 $t$ 的最后的 $u$ ，再手动计算 $u$ 到 $t$ 的代价即可。

现在是树上问题，于是想到树链剖分，将树剖成若干重链后，每条重链都可以按照序列上的问题预处理。

可以发现 ，$s$ 到 $t$ 的路径是若干重链（或重链的一部分）组成的。那么分成几种情况:

1. 在某条重链里跳：根序列上差不多，尽量向下跳，注意不要跳出在这条链上的结束位置

2. 跨链的跳：假设上次跳到的点为 $lst$ ，不在该链上。 那就要找到该链上第一个比 $lst$ 小的点，若当前链顶 $u> lst$ ，则从 $u$ 不断向下跳（跳到最后一个$>lst$ 的点后再多跳一位），可以证明这样找到的是该链第一个比 $lst$ 小的点，然后计算一下 $lst$ 到 $u$ 的代价即可。**注意不能跳出界**，当然也可能找不到比 $lst$ 小的点，那就保存 $lst$ 直接到下一条链即可。

3. 最后还没到 $t$ ：$lst$ 到 $t$ 手动计算即可


~~这题写起来真的毒瘤~~ ，主要是链之间跳细节挺多，详见代码

## CODE

```cpp
#include <bits/stdc++.h>
#define fi first
#define se second
using namespace std;
const int N = 300010;
const int M = 20;
typedef long long ll;
typedef pair<int,int> pii;
int head[N],pnt[N<<1],nxt[N<<1],E,wth[N<<1];
int n,m,dep[N],fa[N],top[N],siz[N],son[N],id[N],bot[N],rk[N],tim,rid[N];
int jump[N][M]; // jump[u][j] u 点在重链的方向上向下跳 2^j 次到的点
ll dis[N],ss[N][M]; //dis[u] u 到根的距离， ss[u][j] u 倍增向下跳 2^j 的代价 
bool vis[N];
inline int red() {
    int x=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(ch^48),ch=getchar();
    return x;
}
void add(int u,int v,int w) {
    E++;pnt[E]=v;nxt[E]=head[u];wth[E]=w;head[u]=E;
}
// 日常树剖
void dfs1(int u) {
    siz[u]=1;top[u]=bot[u]=u;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u]) continue;
        dep[v]=dep[u]+1;fa[v]=u;dis[v]=dis[u]+wth[i];dfs1(v);
        siz[u]+=siz[v];
        if(siz[son[u]]<siz[v]) son[u]=v;
    }
}
void dfs2(int u) {
    id[u]=++tim;rk[tim]=u;
    if(son[u]) {
        top[son[u]]=top[u];
        dfs2(son[u]);
        bot[u]=bot[son[u]];
    }
    for(int i=head[u];i;i=nxt[i]) {
        if(pnt[i]==fa[u]||pnt[i]==son[u]) continue;
        dfs2(pnt[i]);
    }
    rid[u]=tim;
}
// 倍增预处理
int sta[N],tp;
void init() {
    for(int i=1;i<=n;i++)if(!vis[i]) {
        int u=bot[i];tp=0;
        while(u!=fa[top[i]]) {
            vis[u]=1;
            while(tp&&sta[tp]>u) tp--;
            if(tp) jump[u][0]=sta[tp],ss[u][0]=(dis[sta[tp]]-dis[u])*(ll)u;
            sta[++tp]=u;
            u=fa[u];
        }
    }
    for(int j=1;j<M;j++) {
        for(int i=1;i<=n;i++) {
            jump[i][j]=jump[jump[i][j-1]][j-1];
            ss[i][j]=ss[i][j-1]+ss[jump[i][j-1]][j-1];
        }
    }
}
// 核心代码，从s跳到t，真的难写
ll solve(int u,int v) {
    vector<pii> a;
    while(top[v]!=top[u]) { //先把s到t的路径搞出来
        a.push_back(pii(top[v],v));
        v=fa[top[v]];
    }
    a.push_back(pii(u,v));
    reverse(a.begin(),a.end());
    int lst=u;ll ans=0;
    for(int i=0;i<(int)a.size();i++) { //从上向下跳
        u=a[i].fi;
        //找当前链中第一个 <lst 的点，先跳到最后一个 >lst 的点
        //其中id[jump[u][j]]<=id[a[i].se]保证不跳出界
        for(int j=M-1;j>=0;j--) if(jump[u][j]&&id[jump[u][j]]<=id[a[i].se]&&jump[u][j]>lst) u=jump[u][j];
        //从最后一个 >lst 的点向下再跳一位；当然，若u本身 <lst就不用跳了
        if(u>lst) u=jump[u][0];
        //若找到的点是否合法，lst跳u
        if(u!=0&&id[u]<=id[a[i].se]&&u<lst) ans+=(dis[u]-dis[lst])*(ll)lst,lst=u;
        //在链中尽量向下跳
        for(int j=M-1;j>=0;j--) {
            if(jump[u][j]&&id[jump[u][j]]<=id[a[i].se]&&jump[u][j]<lst) ans+=ss[u][j],lst=u=jump[u][j];
        }
    }
    //lst跳t
    ans+=(dis[a.back().se]-dis[lst])*(ll)lst;
    return ans;
}
int main() {
    n=red(),m=red();
    for(int i=1;i<n;i++) {
        int u=red(),v=red(),w=red();
        add(u,v,w);add(v,u,w);
    }
    dfs1(1);dfs2(1);init();
    while(m--) {
        int s=red(),t=red();
        if(id[t]<id[s]||id[t]>rid[s]) {printf("-1\n");continue;}// t不是s的祖先
        ll ans=solve(s,t);
        printf("%lld\n",ans);
    }
}
```