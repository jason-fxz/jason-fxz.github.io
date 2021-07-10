---
title: 'POI2014 - HOT Hotels'
date: 2021-03-02 21:58:47
tags: [树形dp]
categories:
- 题解
---

长链剖分优化树形DP板题。


[luogu-P5905](https://www.luogu.com.cn/problem/P5904)

给你一颗 $n$ 个点的树，求有多少点对 $(i,j,k)$ 满足 $i,j,k$ 两两距离相等。

<!--more-->

### 题解

$u,v,w$ 两两距离相等只有两种情况，（1） $c$ 是 $u,v,w$ 三点的 $\mathrm{LCA}$ 且 $u,v,w$ 到 $c$ 的距离一样。（2）$c$ 是 $u,v$ 的 $\mathrm{LCA}$ 且 $w$ 在 $c$ 的子树外，$u,v,w$ 到 $c$ 距离相等。

考虑暴力 DP，设 $f(u,i)$ 表示 $u$ 的子树中到 $u$ 距离为 $i$ 的点的个数， $g(u,i)$  表示 $u$ 子树中存在多少点对 $(v,w)$ （记 $c=\mathrm{LCA}(v,w)$）满足 $\mathrm{dis}(v,c)=\mathrm{dis}(w,c)=\mathrm{dis}(u,c)+i$ 。

按照普通树形 DP 的套路，当前 $u$ 点，$v$ 是 $u$ 当前访问的子节点，$g(u,i),f(u,i)$ 都是 $u$ 和之前子树合并的结果：

1. $ans+= f(v,i)\times g(u,i+1)$
2. $ans+=g(v,i)\times f(u,i-1)$
3. $g(u,i)+=g(v,i+1)$
4. $g(u,i)+=f(u,i)\times f(v,i-1)$
5. $f(u,i)+=f(v,i-1)$

这样可以做到 $O(n^2)$ 的复杂度，考虑到 **DP 的转移只与深度有关**，可以用长链剖分优化到 $O(n)$ 。这里有个细节， $g(u,i)=g(v,i+1)$ 对应的指针是向左移动的，分配的空间需要**开两倍**。然后重儿子除了直接提供当前节点的 $f,g$ 对答案还有 $g[v][1]$ 的贡献。

### CODE

$O(n^2)$ naive DP：可以过这个 [luogu-P3565](https://www.luogu.com.cn/problem/P3565)

```cpp
#include <bits/stdc++.h>
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp> bool umax(_Tp &x,_Tp y) {return (x<y)?(x=y,true):(false);}
template<typename _Tp> bool umin(_Tp &x,_Tp y) {return (x>y)?(x=y,true):(false);}
using namespace std;
typedef long long ll;
const int N = 5010;
ll ans;
vector<ll> f[N],g[N];
int head[N],pnt[N<<1],nxt[N<<1],E;
int n,hei[N];
void adde(int u,int v) {++E;pnt[E]=v;nxt[E]=head[u];head[u]=E;}
void dfs(int u,int fa) {
    hei[u]=1; 
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa) continue;
        dfs(v,u);
        umax(hei[u],hei[v]+1);
    }
    f[u].resize(hei[u]+2); g[u].resize(hei[u]+2);
    f[u][0]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa) continue;
        for(int i=0;i<=hei[v];++i) {
            ans+=f[v][i]*g[u][i+1];
            ans+=g[v][i+1]*f[u][i];
        }
        for(int i=1;i<=hei[v];++i) {
            g[u][i]+=f[u][i]*f[v][i-1];
            g[u][i]+=g[v][i+1];
            f[u][i]+=f[v][i-1];
        }
    }
    // ans+=g[u][0];
    // for(int i=0;i<=hei[u];++i) printf("f[%d][%d]=%lld %c",u,i,f[u][i],(i==hei[u])?'\n':' ');
    // for(int i=0;i<=hei[u];++i) printf("g[%d][%d]=%lld %c",u,i,g[u][i],(i==hei[u])?'\n':' ');
}
int main() {
    red(n);
    for(int i=1;i<n;++i) {
        int u,v; red(u);red(v);
        adde(u,v);adde(v,u);
    }
    dfs(1,0);
    cout<<ans<<endl;
    return 0;
}
```

$O(n)$ 长剖版：

```cpp
#include <bits/stdc++.h>
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp> bool umax(_Tp &x,_Tp y) {return (x<y)?(x=y,true):(false);}
template<typename _Tp> bool umin(_Tp &x,_Tp y) {return (x>y)?(x=y,true):(false);}
using namespace std;
typedef long long ll;
const int N = 100010;
ll *f[N],*g[N],ans,buf[N<<2],*cur=buf;
int head[N],pnt[N<<1],nxt[N<<1],E;
int n,hei[N],fa[N],son[N],top[N];
void adde(int u,int v) {++E;pnt[E]=v;nxt[E]=head[u];head[u]=E;}
void apply(int u) {f[u]=cur; cur+=hei[u]*2; g[u]=cur; cur+=hei[u]*2;}
void dfs(int u) {
    f[u][0]=1;
    if(son[u]) {
        f[son[u]]=f[u]+1;
        g[son[u]]=g[u]-1;
        dfs(son[u]);
        ans+=g[son[u]][1];
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa[u]||v==son[u]) continue;
        apply(v); dfs(v);
        for(int i=0;i<=hei[v];++i) {
            ans+=f[v][i]*g[u][i+1];
            ans+=g[v][i+1]*f[u][i];
        }
        for(int i=1;i<=hei[v];++i) {
            g[u][i]+=f[u][i]*f[v][i-1];
            g[u][i]+=g[v][i+1];
            f[u][i]+=f[v][i-1];
        }
    }
}
void dfs1(int u,int f) {
    fa[u]=f; hei[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs1(v,u);
        if(hei[v]>hei[son[u]]) son[u]=v;
    }
    hei[u]=hei[son[u]]+1;
}
void dfs2(int u,int topf) {
    top[u]=topf;
    if(son[u]) dfs2(son[u],topf);
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i]; if(v==fa[u]||v==son[u]) continue;
        dfs2(v,v);
    }
}
int main() {
    red(n);
    for(int i=1;i<n;++i) {
        int u,v; red(u);red(v);
        adde(u,v);adde(v,u);
    }
    dfs1(1,0); dfs2(1,1);
    apply(1); dfs(1);
    cout<<ans<<endl;
    return 0;
}
```


