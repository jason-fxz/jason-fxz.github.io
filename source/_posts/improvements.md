---
title: 'Improvements '
date: 2020-10-29 22:54:48
tags: [思维题,动态规划]
categories:
- 题解
---


**[GYM-100553I](https://codeforces.com/gym/100553/attachments)**
**[in zh-CH](https://ioihw20.duck-ac.cn/problem/203)**


## 题意简述

~~原题说的是鬼话~~，这里是人话转述：

有 $n+1$ 个点 $x_0,x_1,\dots,x_n$ 其中 $x_0=0$ ，$x_i$ 两两不同且 $1\le x_i\le n$ ，视第 $i$ 个区间为 $(x_{i-1},x_i)$ ，定义 $A,B$ 两区间不相交为 $A\cap B \ne\phi,A\not\subseteq B，B\not\subseteq A$。要使没有区间相交，有一些 $x_i$ 要改变，求出不用改变的 $x_i$ 最多有几个。

$1\le n\le 2\times 10^5$

<!--more-->

## 题解

没有区间相交，就是说任意一对区间要么包含要么交集为空。

仔细想想，可以发现，若最终 $n$ 个区间合法，对于 $i\lt j$ ，第 $j$ 个区间不能包含第 $i$ 个区间。

手画一下图可以反证若包含一定是不合法的。

然后每一个区间都 要么被之前包含要么于之前交集为空。随着 $i$ 的增加，区间能取到的左端点一定是单调不减的，同理能取到的右端点一定是单调不增的。

设 $p[x_i]=i$ ，可以发现 $p$ 先增后减，于是只要枚举 $p$ 从中间某切开，向左最长单调下降子序列+向右最长单调下降子序列-1 更新答案即可，于是变成了简单dp，~~有手就行~~

## code
```cpp
#include <bits/stdc++.h>
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
#define re register
#define fi first
#define se second
using namespace std;
typedef long long ll;
typedef pair<int,int> pii;
typedef pair<ll,ll> pll;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename T> bool umax(T &x,T y) {return (x<y)?(x=y,true):(false);}
template<typename T> bool umin(T &x,T y) {return (x>y)?(x=y,true):(false);}
const int N = 200010;
int p[N],n,L[N],R[N];
struct trearr {
    int t[N];
    void modify(int x,int vl) {for(;x<=n;x+=x&-x) umax(t[x],vl);}
    int ask(int x) {int r=0;for(;x>0;x-=x&-x) umax(r,t[x]);return r;} 
}f1,f2;
// L[i] 向左的最长下降子序列，R[i] 向右
int main() {
    freopen("improvements.in","r",stdin);
    freopen("improvements.out","w",stdout);
    red(n);
    for(int i=1;i<=n;i++) {
        int x;red(x);
        p[x]=i;
    }
    for(int i=1;i<=n;i++) {
        L[i]=f1.ask(p[i]-1)+1;
        f1.modify(p[i],L[i]);
    }
    for(int i=n;i>=1;i--) {
        R[i]=f2.ask(p[i]-1)+1;
        f2.modify(p[i],R[i]);
    }
    int ans=0;
    for(int i=1;i<=n;i++) umax(ans,L[i]+R[i]-1);
    printf("%d\n",ans);
    return 0;
}
```