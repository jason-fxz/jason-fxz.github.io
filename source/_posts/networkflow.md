---
title: '网络流学习笔记'
date: 2020-11-20 21:43:44
tags: [网络流,图论]
categories:
- 📚算法笔记
---


~~时隔一年半~~开始重拾网络流

<!--more-->

## 图论~~基础~~

### 图的一些概念

图 $G=(V,E)$

**匹配** ：找到一个边集 $F\subseteq E$ 满足没有边共用顶点。

**边覆盖** ：图上任意点都至少是 $M \;(M\subseteq E)$ 中某条边的顶点。

**点覆盖** ：图上任意边至少有一个顶点在 $S\;(S\subseteq V)$ 中。

**独立集** ：点集 $S \subseteq V$ 满足任意两点间没有边相连。



### 一些结论

1. $|\text{最大独立集}|+|\text{最小点覆盖}|=|V|$ 
   - 证明：（自己瞎糊的），考虑要搞出独立集，就是要让所有的边消失（因为一条边存在就说明），就是在原图上删点，删点的时候把点周围的边也删掉，直到图上没有边剩余，也就是点覆盖。那么最大独立集就是要让剩下的点最多，等价于删掉的点最少，便是最小点覆盖。
2. 对于没有孤立点的图（**联通图**），$|\text{最大匹配}|+|\text{最小边覆盖}|=|V|$
   - 证明：这个比较显然，设最大匹配为 $x$ ，那么有 $2x$ 个点属于最大匹配，剩下 $|V|-2x$ 个点找不到匹配，那么最小边覆盖便有 $x$ 条边是最大匹配的边，再  $|V|-2x$  条边覆盖找不到匹配的点，等于 $|V|-x$
3. $|\text{匹配}|\le |\text{点覆盖}|$ 
   - 证明：考虑点覆盖中的一个点覆盖了若干条边（其中某些边还被其他点覆盖了），这写边中显然只能选一条作为匹配（因为共用顶点了），考虑到某些点可能还选不出一条边作匹配（被其他点夺走了边），匹配数肯定不大于点覆盖数。
4. 在**二分图**中，$|\text{最大匹配}|=|\text{最小点覆盖}|$ 
   - 证明：根据结论三，我们知道 $|\text{最大匹配}|\le|\text{最小点覆盖}|$ ，先在要证明 “ $\ge$ ”  。考虑最大匹配后的边有几种，一：匹配点—匹配边—匹配点；二：匹配点—未匹配边—未匹配点；三：匹配点—未匹配边—匹配点。显然只有这三种，不然就不是最大匹配。然后，对于第一种边连了两点 $u,v$ ，不可能 $u,v$ 都有第二种边，不然可以找到增广路，也便不是最大匹配。那么对于一个匹配我们只要选择 $u,v$ 中有第二种边的作为最小点覆盖中的点，即可覆盖所有边（第三种边显然会被两边的某点覆盖到）。
5. 根据二和四可得，在联通二分图中 $|\text{最小点覆盖}|+|\text{最小边覆盖}|=|V|$



## 最大流问题

我们有一张图，要求从源点流向汇点的最大流量（可以有很多条路到达汇点），就是我们的最大流问题。

增广路算法：EK，Dinic，ISAP。关于预流推进，~~算了吧~~

重提一下基本概念：

**流**：流函数 $f(u,v),u,v\in V$ 满足 
1. 容量限制，$f(u,v) \le c(u,v)$
2. 斜对称性，$f(u,v)=- f(v,u)$
3. 流量守恒，流入点的流量等于流出点的流量

**残量网络**：就是图中所有结点和 <u>**剩余容量**大于 0</u> 的边构成的子图。会有虚边，因为 $f(u,v)=-f(v,u)$ 。一般增广都是在<u>残量网络上进行的，而非原图</u>。

**増广路**：一条从源点到汇点路径上所有边<u>剩余容量</u>大于零的路径,$\texttt{没有増广路} \iff \texttt{当前已是最大流}$

### EK

流程：
1. BFS 暴力找増广路，到汇点就停，BFS时判断流合不合法
2. 再走一遍増广路，能增广的流量是路径上最小流，増广路上的边减去值，答案加上这个值，同时**反向边加上这个数值**。

**反向边** ：其实是一种反悔机制，感性理解: 一条新的增广路 $p\to v\to u \to q$ 流经了反向边 $(v,u)$ ，$p\to v$ 已经能满足一部分 $u\to v$ 的流量（$v$ 流量不变），于是可以退回一部分 $u\to v$ 的流量流向别处，这里便流向 $u\to q$

![](./D9FmM4.png)


一点细节，链式向前向星实现反向边，可以让边的下标从 $0$ 开始，正反建边，边 $i$ 的反向边为 $i \operatorname{xor} 1$

时间复杂度为 $O(nm^2)$ ，没啥用，不上代码了


### Dinic

相比 EK ，Dinic在每次增广前用 BFS 对**残量网络**进行分层，源点深度为 $0$ ，每个点的深度便是它到源点的最短距离，每次增广只走最短路。

流程：
1. BFS 对残量网络进行分层，若无法走到汇点则可停止增广（无增广路便是最大流）
2. DFS 找增广路，可**多路增广**，另外还有**当前弧优化**，（没有这两个优化复杂度是错的！）

**多路增广**：就是找到一条增广路流量还有剩余，可以利用剩余流量再找增广路。
**当前弧优化**： 被增广过的边不会再被增广，故可以在链式向前向星上稍加改动，每次从还没被增广的弧开始增广，详见代码。


复杂度为 $O(n^2m)$，证明~~咕咕咕~~

特别的，若是**单位网络**（所有边的流量均为 $1$ ，且除了源点和汇点外的所有点，都满足入边最多只有一条，或出边最多只有一条），复杂度为 $O(m\sqrt{n})$，可以求二分图匹配，而二分图匹配朴素的匈牙利算法复杂度 $O(mn)$

{% spoiler "Dinic代码实现" %}

```cpp
#include <bits/stdc++.h>
using namespace std;
template<size_t N,size_t M>
struct Dinic{
    const int inf = 0x3f3f3f3f;
    // fl为边的剩余流量
    int fl[M<<1],pnt[M<<1],nxt[M<<1],cur[N],head[N],dep[N],E,S,T;
    long long maxf;
    Dinic() {memset(head,-1,sizeof(head));E=-1;}
    void adde(int u,int v,int c) { // 建边，正反边同时建，^1 就是反边
        E++;pnt[E]=v;fl[E]=c;nxt[E]=head[u];head[u]=E;
        E++;pnt[E]=u;fl[E]=0;nxt[E]=head[v];head[v]=E;
    }
    bool BFS(int u) { // BFS 对点进行深度标号
        memset(dep,0x3f,sizeof(dep));dep[S]=0;
        queue<int> q;q.push(S);
        while(!q.empty()) {
            int u=q.front();q.pop();
            if(u==T) break; // 找到汇点就结束，显然是正确的，可以减小常数
            for(int i=head[u];i!=-1;i=nxt[i]){
                if(fl[i]>0&&dep[pnt[i]]==inf) // 残量网络中，边的剩余流量需要大于零
                    dep[pnt[i]]=dep[u]+1,q.push(pnt[i]);
            }
        }
        return dep[T]!=inf; // 若S，T不连同，说明无增广路，即可退出
    }
    int DFS(int u,int nf) { // DFS多路增广，nf为流入u的流量
        if(u==T||nf==0) return nf;
        int flow=0,tf; // flow为实际能流的流量
        for(int &i=cur[u];i!=-1;i=nxt[i]) { // &i=cur[u] 为当前弧优化 
            int v=pnt[i];
            if(dep[v]!=dep[u]+1) continue; // 只走最短路
            tf=DFS(v,min(nf,fl[i])); // tf 为流向 v 的流量
            if(tf==0) continue;
            flow+=tf;nf-=tf; 
            fl[i]-=tf;fl[i^1]+=tf; // 正边加上，反边减去
            if(nf==0) break;
        }
        return flow;
    }
    long long maxflow() {
        maxf=0;
        while(BFS(S)) {
            memcpy(cur,head,sizeof(cur)); // cur记录当前弧
            maxf+=DFS(S,inf);
        }
        return maxf;
    }
};
Dinic<205,5005> G;
int n,m,s,t;
int main() {
    scanf("%d%d%d%d",&n,&m,&s,&t);
    G.S=s,G.T=t;
    for(int i=1;i<=m;++i) {
        int u,v,c;
        scanf("%d%d%d",&u,&v,&c);
        G.adde(u,v,c);
    }
    printf("%lld\n",G.maxflow());
}
```
{% endspoiler %}

### ISAP 

先咕咕


## 最小割问题

**割**：对于一个网络流图 $G=(V,E)$ ，其割的定义为一种 <u>*点的划分方式* </u>：将所有的点划分为 $S$ 和 $T=V-S$ 两个集合，其中源点 $s\in S$ ，汇点 $t\in T$ 。

**割的容量**：我们的定义割 $(S,T)$ 的容量 $c(S,T)$ 表示所有从 $S$ 到 $T$ 的边的容量之和，即 $c(S,T)=\sum_{u\in S,v\in T} c(u,v)$ 。

**最小割**：就是容量最小的割呗。

### 定理

**最大流最小割定理**：最大流等于最小割

证明:

1. $c(S,T)_{\max} \ge f(s,t)_{\min}$ 这个比较显然，最大流不可能比割掉的那几条边的容量还大
2. $c(S,T)_{\max} \le f(s,t)_{\min}$ 考虑Dinic的退出条件，残量网络上 $s$ 没有到 $t$ 的合法路径，也就是说最小割的边一定被流满了

故，$c(S,T)_{\max} = f(s,t)_{\min}$



### 求方案

求最小割根据定理，跑最大流即可，那如何输出方案呢？

根据证明中的二，既然最小割的边一定被流满了（剩余流量为$0$）,我们可以通过从源点 $s$ 开始 DFS ，每次走残量大于 $0$ 的边，找到所有 $S$ 点集内的点。

## 费用流

相比最大流，多加了一个单位流量的费用 $w(u,v)$ 。含义为当 $(u,v)$ 流量为 $f(u,v)$ 时，需要费用 $f(u,v)\times w(u,v)$ 。

费用也满足**斜对称性**： $w(u,v)=-w(v,u)$

**最小费用最大流**：就是在最大流的前提下满足总费用最小

### SSP

在最大流的 EK 算法求解最大流的基础上，把 BFS 找增广路改为 SPFA 找**单位费用之和**最小的增广路 就行了。
其实就是把 $w(u,v)$ 看成边权跑最短路就行了
~~显然和 EK 一样没啥卵用~~

### 类 Dinic

类似与 SSP，现在就是在 Dinic 基础上，把求分层图的 BFS ，改为 SPFA （为什么不用Dijkstra?因为有负权边）以 $w(u,v)$ 做为边权跑最短路。

关于建反边，反边的费用与正边相反，因为退流的同时也把费用退回了。

{% spoiler "SPFA-Dinic费用流" %}

```cpp
#include <bits/stdc++.h>
using namespace std;
template<size_t N,size_t M>
struct SPFA_Dinic{
    const int inf = 0x3f3f3f3f;
    int fl[M<<1],cst[M<<1],pnt[M<<1],nxt[M<<1],cur[N],head[N],dis[N],E,S,T;
    bool vis[N];
    int maxf,minc;
    SPFA_Dinic() {memset(head,-1,sizeof(head));E=-1;}
    void adde(int u,int v,int c,int w) { // (u,v) c容量限制，w单位费用
        E++;pnt[E]=v;fl[E]=c;cst[E]=w;nxt[E]=head[u];head[u]=E;
        E++;pnt[E]=u;fl[E]=0;cst[E]=-w;nxt[E]=head[v];head[v]=E;
    }
    bool SPFA() {
        memset(dis,0x3f,sizeof(dis));
        queue<int> q;q.push(S);
        dis[S]=0;vis[S]=1;
        while(!q.empty()) {
            int u=q.front();q.pop();vis[u]=0;
            for(int i=head[u];i!=-1;i=nxt[i]){
                int v=pnt[i];
                if(fl[i]>0&&dis[v]>dis[u]+cst[i]) {
                    dis[v]=dis[u]+cst[i];
                    if(!vis[v]) q.push(v),vis[v]=1;
                }
            }
        }
        return dis[T]!=inf;
    }
    int DFS(int u,int nf) {
        if(u==T||nf==0) return nf;
        int flow=0,tf;vis[u]=1;
        for(int &i=cur[u];i!=-1;i=nxt[i]) { 
            int v=pnt[i];
            if(!vis[v]&&fl[i]&&dis[v]==dis[u]+cst[i]) {
                tf=DFS(v,min(nf,fl[i]));
                if(tf) {flow+=tf;nf-=tf;fl[i]-=tf;fl[i^1]+=tf;minc+=tf*cst[i];}
            }
            if(nf==0) break;
        }
        vis[u]=0;
        return flow;
    }
    void MFMC() {
        maxf=minc=0;
        int cnt=0;
        while(SPFA()) {
            memcpy(cur,head,sizeof(cur));
            maxf+=DFS(S,inf);
        }
    }
};
SPFA_Dinic<5005,50005> G;
int n,m,s,t;
int main() {
    scanf("%d%d%d%d",&n,&m,&s,&t);
    G.S=s,G.T=t;
    for(int i=1;i<=m;++i) {
        int u,v,c,w;
        scanf("%d%d%d%d",&u,&v,&c,&w);
        G.adde(u,v,c,w);
    }
    G.MFMC();
    printf("%d %d\n",G.maxf,G.minc);
    return 0;
}
```

{% endspoiler %}



### Dijkstra 费用流

有时候，~~关于SPFA，它死了~~，需要更好的最短路算法求增广路，显然 Dijkstra 的复杂度比 SPFA 优秀，唯一的缺陷是不能处理负权边，（因为反边的费用是负的），于是便有一种神奇的方法把边搞正来，类似于 [Johnson 全源最短路](https://studyingfather.com/archives/2475)

给每个点加一个“势”$h_u$，然后对于一条边 $(u,v)$，将其边权当作 $w_{u\to v} +h_u-h_v$ ，这样从 $S$ 到点 $i$ 的最短路径长度为 $h_S-h_i+\sum w$ ，由于 $h_S-h_i$ 为常数，所以在这张图上求最短路减去 $h_S-h_i$ 即为原图最短路，那么只要使 $w_{u\to v} +h_u-h_v\ge 0$ 即可，移项得：
$$
h_u+w_{u\to v} \ge h_v
$$
这不就是最短路三角形不等式吗，跑一趟 SPFA ，$h_u=dis_u$ 即可。

考虑到没扩展一次图会发生变化，存在某次增广后 $u\to v$ 权为 $w(w\ge 0)$ 的边变成反边 $v\to u$ 权为 $w'=-w$ ，要对 $h_i$ 进行一些更改：
记上一次跑完最短路距离标为 $dis_i'$，显然有:
$$
\begin{aligned}
dis_u'+w'&=dis_v'\\
dis_u'+h_u-h_v+w'&=dis_v'\\
(h_u+dis_u')-(h_v+dis_v')+w'&=0\\
(h_v+dis_v')-(h_u+dis_u')+w&=0
\end{aligned}
$$
也就是说，每次最短路次增广后，$h_u\operatorname{+=}dis_u$ 即可

~~代码实现，先咕咕咕。~~

Updata 2021-2-14: 更新了 Dij 费用流

{% spoiler "Dijkstra费用流" %}

```cpp
#include <bits/stdc++.h>
#define fi first
#define se second
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
typedef pair<int,int> P;
const int inf = 0x3f3f3f3f;
int n,m,s,t;
struct networkflow{
    static const int N = 5010, M = 100010;
    int head[N],pnt[M],fl[M],nxt[M],cst[M],pre[N],lst[N],E=-1;
    int dis[N],h[N]; bool vis[N];
    int minc,maxf,n;
    void init(int _n) {
        memset(head,-1,sizeof(head));E=-1; n=_n;
    }
    void adde(int u,int v,int f,int c) {
        ++E;pnt[E]=v;nxt[E]=head[u];head[u]=E;fl[E]=f;cst[E]=c;
        ++E;pnt[E]=u;nxt[E]=head[v];head[v]=E;fl[E]=0;cst[E]=-c;
    }
    bool SPFA(int S,int T) {
        memset(dis,0x3f,sizeof(dis));
        memset(vis,0,sizeof(vis));
        dis[S]=0; vis[S]=1; queue<int> q; q.push(S);
        while(!q.empty()) {
            int u=q.front(); q.pop();
            vis[u]=0;
            for(int i=head[u];i!=-1;i=nxt[i]) {
                int v=pnt[i];
                if(fl[i]>0&&dis[u]+cst[i]<dis[v]) {
                    dis[v]=dis[u]+cst[i]; pre[v]=u; lst[v]=i;
                    if(!vis[v]) q.push(v),vis[v]=1;
                }
            }
        }
        return dis[T]<inf;
    }
    void work(int S,int T) {
        for(int i=0;i<=n;++i) h[i]+=dis[i];
        int flow=inf;
        for(int u=T;u!=S;u=pre[u]) flow=min(flow,fl[lst[u]]);
        maxf+=flow; minc+=flow*h[T];
        for(int u=T;u!=S;u=pre[u]) {
            fl[lst[u]]-=flow;
            fl[lst[u]^1]+=flow;
        }
    }
    bool Dij(int S,int T) {
        priority_queue<P,vector<P>,greater<P> > q;
        memset(dis,0x3f,sizeof(dis));
        memset(vis,0,sizeof(vis));
        dis[S]=0; q.push({0,S});
        while(!q.empty()) {
            int u=q.top().se; q.pop();
            if(vis[u]) continue; vis[u]=1;
            for(int i=head[u];i!=-1;i=nxt[i]) {
                int v=pnt[i];
                if(fl[i]>0&&dis[v]>dis[u]+cst[i]+h[u]-h[v]) {
                    dis[v]=dis[u]+cst[i]+h[u]-h[v];
                    pre[v]=u; lst[v]=i;
                    q.push({dis[v],v});
                }
            }
        }
        return dis[T]<inf;
    }
    void MFMC(int S,int T) {
        minc=maxf=0;
        memset(h,0,sizeof(h));
        if(SPFA(S,T)) work(S,T); else return ;
        while(Dij(S,T)) work(S,T);
    }
}G;
int main() {
    red(n);red(m);red(s);red(t);
    G.init(n);
    for(int i=1;i<=m;++i) {
        int a,b,c,d; red(a);red(b);red(c);red(d);
        G.adde(a,b,c,d);
    }
    G.MFMC(s,t);
    printf("%d %d\n",G.maxf,G.minc);
    return 0;
}
```


{% endspoiler %}


## Reference

[Oi-wiki 网络流](https://oi-wiki.org/graph/flow/)