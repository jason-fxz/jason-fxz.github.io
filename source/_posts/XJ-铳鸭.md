---
title: XJ - 铳鸭
date: 2020-12-25 22:03:45
tags: [线段树]
categories: 题解
hidden: true
---

## 题面

![P](./P.png)

确实是原题 **[BZOJ3838](https://darkbzoj.tk/problem/3838)**

<!--more-->

## 题解

考虑 **模拟费用流**（显然跑费用流铁定 TLE ，故这里只是 **模拟** ，来发现其本质在干哈）：

考虑建图：

1. 建 $n$ 个点 ，$\forall i\in[1,n),i\to i+1$ 连边容量 $+\infty$ 费用 $0$。
2. 建源汇点 $s',t$ ，对于所有 `(`，$s'\to i$ 容量 $1$，费用为该位置代价； 对于所有 `)`，$i\to t$ 容量 $1$，费用为该位置代价
3. 实际源点 $s$ 向 $s'$ 连边，容量为 $k/2$，费用为 $0$

以上建图正确性显然，跑最小费用最大流就完事了，现在来考虑以下这实际在干嘛。

![P2](./P2.png)

在做费用流时，每次最短路增广一单位流量，如上图 橙/绿 线就是一次合法增广。对于一次增广，如橙线经过的两端点 $a,b$ 就是这次选中的括号，显然是一个左括号一个右括号。再考虑这个图上增广路不可能从 $s'$ 出去再绕到 $s'$ ，也就是说之前某次增广 $s\to u$ 的流量不会退回来：**某个括号被选中后就不会删除了**。

再考虑增广路有几种情况：

1. 如橙线，左括号在右括号左边，$a\to b$ 路径上边权都是 $+\infty$ 显然合法，且给 $a,b$ 间反向边都加一
2. 如绿线，左括号在右括号右边，这时候就有限制了，需要 $c,d$ 间反向边都为正。且增广后给这些反向边都减一

总结以下，需要干如下的事情：

找一个左括号，一个右括号，使其代价和最小，并有限制：（ 即有一个长度为 $n-1$ 的序列 $S$ ，初始都为零 ）

- 若左括号在右括号左边：无限制，取了之后 $S[l\dots r-1]$ 区间加一
- 若左括号在右括号右边：需要 $\min_{i\in[l,r-1]} S_i\gt 0$ ，取了之后 $S[l\dots r-1]$ 区间减一

然后这玩意儿变得很 ~~simple~~ ，我们用线段树维护即可。

其实也挺毒瘤的，线段树上要维护九个值。具体的：

需要维护 $S$ 的区间 $\min$ ，支持区间加减

先考虑左括号在右括号左：
维护区间最小代价左括号 $mnL$，区间最小代价右括号 $mnR$ ，区间最小代价括号对 $mnLR$ ，合并的时候显然

$$
\begin{aligned}
mnL_x&=\min (mnL_{ls},mnL_{rs})\\
mnR_x&=\min (mnR_{ls},mnR_{rs})\\
mnLR_x&=\min (mnLR_{ls},mnLR_{rs},mnL_{ls}+mnR_{rs})\\
\end{aligned}
$$

再考虑左括号在右括号右，这种情况**有限制**，需要两括号间 $s$ 都大于零，于是我们维护两套信息，一套假设区间 $\min>0$ 一套假设区间 $\min=0$ 。

对于前者，我们只要和上面的一样维护即可，对于后者，考虑合并的时候两括号区间内 $S_i>0$ ，那么 $mnL'$ 维护左端开始不经过区间 $\min$ 位置的最小代价左括号，$mnR'$ 从右端开始不经过 $\min$ 位置的的最小代价右括号。

考虑合并，需要分讨：

若 $\min_{ls}=\min_{rs}$ ，即假设区间 $\min=0$ ，两边都有零。

$$
\begin{array}{ccccc|cccc}
\underbrace{\cdots\cdots}_{mnL_{ls}'} & 0 &\cdots & 0 & \underbrace{\cdots}_{mnR_{ls}'} & \underbrace{\cdots}_{mnL_{rs}'} & 0 & \cdots & \underbrace{\cdots\cdots}_{mnR_{rs}'} \\
\end{array}
$$

零的位置随便标的，没有关系，考虑 $mnRL_{x}'$ 除了合并两孩子内的，还可以等于 $mnR_{ls}'+mnL_{rs}'$ ，此外还可以发现 $mnL_x'=mnL_{ls}',\;mnR_x'=mnR_{rs}'$

若 $\min_{ls}\ne \min_{rs}$，有两种情况，这里只讨论左小于右（右小于左同理）

$$
\begin{array}{ccccc|c}
\underbrace{\cdots\cdots}_{mnL_{ls}'} & 0 &\cdots & 0 & \underbrace{\cdots}_{mnR_{ls}'} & \underbrace{\cdots\quad\cdots\quad\cdots\quad\cdots}_{mnL_{rs},mnR_{rs}} \\
\end{array}
$$

此时只可能左边有零，右边是没有限制的，可以发现：
$$
mnRL_{x}'=\min(mnRL_{ls}',mnRL_{rs},mnR_{ls}'+mnL_{rs})
$$

$$
mnL_x'=mnL_{ls}',\;mnR_x'=\min(mnR_{rs},mnR_{ls}')
$$

~~然后就完事了~~，注意以上维护的都是下标，为了方便建议重载运算符。每次找最小的括号对，若 $\min=0$ 取 $mnRL'$ ，不然取 $mnRL$。$mnLR$ 无论如何都可以取。然后把对应两括号下标 $p_1,p_2$ 代价改为 $+\infty$ ，并对 $[p_1,p_2-1]$ 进行区间加减，**注意** $[p_1,p_2-1]$ 而不是 $[p_1,p_2]$ 。

然后真的结束了

## CODE

```cpp
#include <cstdio>
#include <cstdlib>
#include <cstring>
#define _mn(X) gmin(X[ls],X[rs])
template <typename T>
inline void red(T &x) {
    x=0;bool f=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') f^=1; ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(f) x=-x;
}
using namespace std;
typedef long long ll;
const int N = 2000010;
const int inf = 0x3f3f3f3f;
char s[N];
int a[N],n,K;
struct dat{
    int p1,p2;
    dat(){}
    dat(int _p1,int _p2):p1(_p1),p2(_p2) {}
};
inline int gmin(int x,int y) {return (a[x]<a[y])?x:y;}
inline dat gmin(dat x,dat y) {return (a[x.p1]+a[x.p2]<a[y.p1]+a[y.p2])?x:y;}
namespace segtree {
    #define ls (x<<1)
    #define rs (x<<1|1)
    #define mid ((l+r)>>1)
    int mn[N<<2],tag[N<<2];
    int mnL[N<<2],mnR[N<<2],_mnL[N<<2],_mnR[N<<2];
    dat mnLR[N<<2],_mnRL[N<<2],mnRL[N<<2];
    // mn 括号序列前缀和最小值,tag 区间+/-懒标记
    // mnL 区间最小代价 ‘(’ ; mnR 区间最小代价 ‘)’
    // mnLR 区间最小代价 ‘()’
    // mnRL 区间最小代价 ‘)(’ 
    // 加 ‘_’ 表示带限制，即假设区间 min=0 ; 其他都假设区间 min!=0
    // 因为要返回位置，上面除了 mn/tag 以外的都存其所在位置 
    void pushup(int x) {
        mn[x]=(mn[ls]<mn[rs])?mn[ls]:mn[rs];
        // normal 
        mnL[x]=_mn(mnL);
        mnR[x]=_mn(mnR);
        mnLR[x]=gmin(_mn(mnLR),dat(mnL[ls],mnR[rs]));
        mnRL[x]=gmin(_mn(mnRL),dat(mnR[ls],mnL[rs]));
        // suppose min = 0
        if(mn[ls]==mn[rs]) { // case1 : mn[ls]==mn[rs]
            _mnL[x]=_mnL[ls]; 
            _mnR[x]=_mnR[rs];
            _mnRL[x]=gmin(_mn(_mnRL),dat(_mnR[ls],_mnL[rs]));
        }else {
            if(mn[ls]<mn[rs]) {
                _mnL[x]=_mnL[ls];
                _mnR[x]=gmin(_mnR[ls],mnR[rs]);
                _mnRL[x]=gmin(gmin(_mnRL[ls],mnRL[rs]),dat(_mnR[ls],mnL[rs]));
            }else {
                _mnL[x]=gmin(mnL[ls],_mnL[rs]);
                _mnR[x]=_mnR[rs];
                _mnRL[x]=gmin(gmin(mnRL[ls],_mnRL[rs]),dat(mnR[ls],_mnL[rs]));
            }
        }
    }
    inline void updata(int x,int vl) { mn[x]+=vl; tag[x]+=vl;}
    inline void pushdown(int x) {
        if(tag[x]==0) return ; 
        updata(ls,tag[x]); updata(rs,tag[x]); tag[x]=0;
    }
    void modify(int L,int R,int vl,int l=1,int r=n,int x=1) {
        if(L<=l&&r<=R) return updata(x,vl);
        pushdown(x);
        if(L<=mid) modify(L,R,vl,l,mid,ls);
        if(R>mid) modify(L,R,vl,mid+1,r,rs);
        pushup(x);
    }
    void del(int p,int l=1,int r=n,int x=1) {
        if(l==r) {
            mnL[x]=mnR[x]=_mnL[x]=_mnR[x]=0;
            a[l]=inf;
            return ;
        }
        pushdown(x);
        if(p<=mid) del(p,l,mid,ls);
        else del(p,mid+1,r,rs);
        pushup(x);
    }
    void build(int l=1,int r=n,int x=1) {
        if(l==r) {
            mnL[x]=(s[l]=='(')?l:0;
            mnR[x]=(s[l]==')')?l:0;
            _mnL[x]=_mnR[x]=0;
            return ;
        }
        build(l,mid,ls); build(mid+1,r,rs);
        pushup(x);
    }
    // void print(int l=1,int r=n,int x=1) {
    //     printf("[%d,%d] mn:%d tag:%d L:%d R:%d L':%d R':%d",l,r,mn[x],tag[x],mnL[x],mnR[x],_mnL[x],_mnR[x]);
    //     printf(" LR:%d,%d RL:%d,%d RL':%d,%d \n",mnLR[x].p1,mnLR[x].p2,mnRL[x].p1,mnRL[x].p2,_mnRL[x].p1,_mnRL[x].p2);
    //     if(l==r) return ;
    //     // pushdown(x);
    //     print(l,mid,ls);print(mid+1,r,rs);
    //     // pushup(x);
    // }
}
using namespace segtree;

int main() {
    scanf("%s",s+1);s[0]='+';
    n=strlen(s+1);
    a[0]=inf;
    for(int i=1;i<=n;++i) red(a[i]);
    build();
    red(K);
    if(K&1) printf("-1\n");
    else {
        ll ans=0;
        for(int ts=1;ts<=K/2;++ts) {
            dat A=mnLR[1];
            if(mn[1]==0) A=gmin(A,_mnRL[1]);
            else A=gmin(A,mnRL[1]);
            if(A.p1==0||A.p2==0||a[A.p1]==inf||a[A.p2]==inf) {
                puts("-1");
                return 0;
            }
            ans+=a[A.p1]+a[A.p2];
            // 注意区间 加/减 是 [l,r-1] 不是 [l,r]
            if(s[A.p1]=='(') modify(A.p1,A.p2-1,1); 
            else modify(A.p1,A.p2-1,-1);
            del(A.p1); del(A.p2);
        }
        printf("%lld\n",ans);
    }    
}

```
