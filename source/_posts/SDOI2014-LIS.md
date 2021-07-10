---
title: 'SDOI2014 - LIS'
date: 2020-12-10 22:05:59
tags: [网络流]
categories:
- 题解
---


**[LOJ - 2196](https://loj.ac/p/2196)**

## 题意

给定长度为 $N$ 序列 $A$ ，序列中的每一项 $A_i$ 有删除代价 $B_i$ 和附加属性 $C_i$。
请删除若干项，使得 $A$ 的最长上升子序列长度减少至少 $1$，且付出的代价之和最小，并输出方案。如果有多种方案，请输出将删去项的附加属性排序之后，字典序最小的一种。

$$N\le 700 $$

<!--more-->

## 题解

求 LIS 跑一遍 DP 即可，记最大的 DP 值为 $mxdp$ ，从 DP 值等于 $mxdp$ 的位置倒推回去，按转移关系建边得到一个转移图，显然入度为零的都是 DP 值为一的，出度为零的都是 DP 值为 $mxdp$ 的。现在就是要求最小的代价使入度为零的点与出度为 $mxdp$ 的点不连通。

那不就是**最小割**吗。具体建图：
1. 超级源点 $S$ 连向所有 DP 值为零的点，边权 $+\infty$
2. 所有 DP 值为 $mxdp$ 的点连向超级汇点 $T$ 边权 $+\infty$
3. 对于所有 $dp[u]+1=dp[v],u\lt v,A_u\lt A_v$ ，$u$ 连向 $v$ ，边权 $+\infty$
4. 因为代价在点上，**拆点**，拆 $u$ 为 $u,u'$ ，$u$ 连 $u'$ 边权为 $B_u$

跑一个最大流可知最小代价。

剩下的便是求方案，这里要求 **字典序最小的最小割**，涉及到求最小割的**可行边** （某条边出现在某个最小割方案中）一条边 $(u,v)$ 为最小割的可行边的充要条件为：
1. 该边满流 。
2. 不存在 $u$ 到 $v$ 的增广路。

判断 $u$ 到 $v$ 的增广路 BFS 即可。

将边按照 $C$ 升序排序依次判断是否为最小割可行边，若是则加入答案，并需要删除它的影响，需要**退流** ：$u$ 向 $S$ 退流， $v$ 向 $T$ 退流，倒着跑 Dinic 。并删除 $(u,v)$ 及其反向边，流量都赋零。

## CODE

```cpp
#include <bits/stdc++.h>
#define P1(x) (x)
#define P2(x) (x+n)
#define SS (n*2+1)
#define TT (n*2+2)
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename T> void umin(T &x,T y) {if(x>y) x=y;}
template<typename T> void umax(T &x,T y) {if(x<y) x=y;}
const int N = 705;
const int inf = 0x3f3f3f3f;
struct newflow{
    static const int N = 1500;
    static const int M = 2000010;    
    int head[N],pnt[M],nxt[M],fl[M],dep[N],cur[N],E,mxflow;
    void init() { memset(head,-1,sizeof(head));E=-1; }
    int adde(int u,int v,int c) {
        // printf("%d %d %d\n",u,v,c);
        E++;pnt[E]=v;nxt[E]=head[u];fl[E]=c;head[u]=E;
        E++;pnt[E]=u;nxt[E]=head[v];fl[E]=0;head[v]=E;
        return E-1;
    }
    bool BFS(int S,int T) {
        memset(dep,0x3f,sizeof(dep));
        queue<int> q; dep[S]=0; q.push(S);
        while(!q.empty()) {
            int u=q.front();q.pop();
            for(int i=head[u];i!=-1;i=nxt[i]) {
                int v=pnt[i];
                if(fl[i]>0&&dep[v]==inf) {
                    dep[v]=dep[u]+1;
                    q.push(v);
                }
            }
        }
        return dep[T]!=inf;
    }
    int DFS(int u,int nf,int T) {
        if(u==T||nf==0) return nf;
        int flow=0,tf;
        for(int &i=cur[u];i!=-1;i=nxt[i]) {
            int v=pnt[i];
            if(dep[v]!=dep[u]+1||fl[i]<0) continue;
            tf=DFS(v,min(nf,fl[i]),T);
            nf-=tf;fl[i]-=tf;
            flow+=tf;fl[i^1]+=tf;
            if(nf==0) break;
        }
        return flow;
    }
    int Dinic(int S,int T) {
        mxflow=0;
        while(BFS(S,T)) {
            memcpy(cur,head,sizeof(cur));
            mxflow+=DFS(S,inf,T);
        }
        return mxflow;
    }
}G;
int Ts,n,A[N],B[N],C[N],tt[N];
int dp[N],mxdp,ans[N],ai,bl[N];
bool fg[N];
void clear() {
    for(int i=1;i<=n;++i) dp[i]=0,fg[i]=0,bl[i]=0;
    ai=0;
    mxdp=0;
}
bool cmp(int x,int y) {
    return C[x]<C[y];
}
void print() {
    for(int i=1;i<=n;++i) if(fg[i]) {
        printf("node %d : %d\n",i,G.fl[bl[i]]);
    }
    printf("------------------\n");
}
int main() {
    red(Ts);
    for(int cs=1;cs<=Ts;++cs) {
        red(n);
        for(int i=1;i<=n;++i) red(A[i]);
        for(int i=1;i<=n;++i) red(B[i]);
        for(int i=1;i<=n;++i) red(C[i]);
        clear();
        G.init();
        for(int i=1;i<=n;++i) {
            dp[i]=1;
            for(int j=1;j<i;++j) if(A[j]<A[i]) umax(dp[i],dp[j]+1);
            umax(mxdp,dp[i]);
            // printf("dp[%d]=%d\n",i,dp[i]);
        }
        for(int i=n;i>=1;--i) if(dp[i]==mxdp||fg[i]) {
            fg[i]=1;
            for(int j=1;j<i;++j) if(dp[j]+1==dp[i]&&A[j]<A[i]){
                fg[j]=1;
                G.adde(P2(j),P1(i),inf);
                // printf("%d' %d inf\n",j,i);
            }
        }
        for(int i=1;i<=n;++i) if(fg[i]){
            bl[i]=G.adde(P1(i),P2(i),B[i]);
            // printf("%d %d' %d\n",i,i,B[i]);
            if(dp[i]==mxdp) {
                G.adde(P2(i),TT,inf);
                // printf("%d' T inf\n",i);
            }
            if(dp[i]==1) {
                G.adde(SS,P1(i),inf);
                // printf("S %d inf\n",i);
            }
        }
        // cout<<"!!"<<endl;
        printf("%d ",G.Dinic(SS,TT));
        // print();
        for(int i=1;i<=n;++i) tt[i]=i;
        sort(tt+1,tt+n+1,cmp);
        for(int i=1;i<=n;++i) if(fg[tt[i]]) {
            int u=tt[i];
            if(G.BFS(P1(u),P2(u))) continue;
            ans[++ai]=u;
            G.Dinic(TT,P2(u));
            G.Dinic(P1(u),SS);
            // print();
            // G.fl[bl[u]]=G.fl[bl[u]^1]=0;
        }
        sort(ans+1,ans+ai+1);
        printf("%d\n",ai);
        for(int i=1;i<=ai;++i) {
            printf("%d ",ans[i]);
        }
        puts("");
    }
}
```
