---
title: '[牛客] 货物分组'
date: 2020-09-24 22:00:01
tags: [动态规划,dp优化,线段树,单调栈]
categories:
- 题解
---


[『牛客』货物分组](https://ac.nowcoder.com/acm/problem/53870)

## 题意简述

$n$ 个物品，第 $i$ 个物品重量为 $a_i$ ，要求按顺序连续放入若干箱子，每个箱子物品总重不超过 $W$ ，箱子从 $1$ 开始按序编号，对于第 $i$ 个箱子 ，其代价为 
$$i \times \text{(箱中货物总重)}+\text{(箱中最大货物重量)}-\text{(箱中最小货物重量)}$$

求最小总代价

<!--more-->

**注：数据范围** 
对于 $10\%$ 的数据，$n\leq 10$
对于 $30\%$ 的数据，$n\leq 500$
对于 $60\%$ 的数据，$n\leq 5000$
对于 $100\%$ 的数据，$n\leq 10^5,1\leq a_i\leq W\leq 10^5$


## 题解

### 30pts

先记 $s_n=\sum_{i=1}^{n} a_i$

考虑暴力dp，设状态 $f_{i,j}$ 表示前 $i$ 个物品当前分到第 $j$ 个箱子的最小代价，考虑转移：
$$
f_{i,j}=\min\limits_{k< i ,s_i-s_k\le W}  f_{k,j-1}+j\cdot(s_i-s_k)+\mathrm{cost}(k+1,i)
$$

其中 $\mathrm{cost}(k+1,i)$ 计算 $a_{k+1},a_{k+2},\dots,a_i$ 这段的极差（最大值减最小值），用ST表可以维护

上述dp状态 $\mathcal{O}(n^2)$ ，转移 $\mathcal{O}(n)$ 显然不够优秀，可以把状态优化掉一维。

### 60pts

可以 **费用提前计算** ，去掉原本 $j$ 这维，转移方程如下：
$$
f_i=\min\limits_{j< i} \; f_j+s_n-s_j+\mathrm{cost}(j+1,i)
$$
复杂度 $\mathcal{O}(n^2)$ 

### 100pts

唯一可以优化的就是转移了，$f_j+s_n-s_j$ 是定值，唯一烦人的是 $\mathrm{cost}$ ，$\mathrm{cost}$ 确实有单调性，但是和 $f_j+s_n-s_j$ 加起来就不一定了。

记 $g_j=f_j+s_n-s_j+\mathrm{cost}(j+1,i)$ ，再线段树中维护 $g$ ，然后我们要关心的是当 $i$ 变成 $i+1$ 时这些 $\mathrm{cost}(j+1,i)$ 会怎么变：

$\mathrm{cost}(j+1,i) = \max\limits_{j+1\le p\le i} a_p-\min\limits_{j+1\le p\le i} a_p$  显然需要分开考虑，为了方便，这里我们先讨论 $\max$ 。 首先 $\max\limits_{j+1\le p\le i} a_p$ 显然随 $j$ 单调递减（非严格递减），形象一点画出来大概这样：（其中 r5就等于i）

![](1600956052732.png)

现在处理 $a_{i+1}$ 的影响（看下图），显然4，5两段最大值会更新到 $a_{i+1}$ ，实际在线段树中的操作为 $[l_5,r_5]$ 区间加 $a_{i+1}-c_5$ ，$[l_4,r_4]$ 区间加 $a_{i+1}-c_4$ 。

![](1600956065474.png)

然后我们把原本的4，5段合并了，顺带把 $i+1$ 的位置也算上，此时的$r_4'=i+1，c_4'=a_{i+1}$

![](1600956075932.png)

图上的东西用个单调栈维护即可，记录相同值的区间左右端点和当前max值，更新时维护单调栈，并在线段树中更新最值。min的维护同理，无非是单调递增罢了。

然后是dp的更新 $f_i=\mathrm{querymin}(Lp,i-1)$ ，querymin是在线段树中查询区间最值，$Lp$ 是满足 $s_i-s_{Lp}$ 的最小值，不符合条件不断向右移就行了。

计算一下复杂度，线段树修改和查询都是 $\log n$ ，查询 $n$ 次数 ，修改次数为即为单调栈中区间合并次数，故也是 $\mathcal{O}(n)$ 的。

故复杂度 $\mathcal{O}(n\log n)$ 

## CODE
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 100010;
const ll inf = 0x3f3f3f3f3f3f3f3f;
ll tag[N<<2],mn[N<<2],f[N],s[N];
int n,a[N],W;
inline int red() {
    int r=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') r=(r<<1)+(r<<3)+(ch^48),ch=getchar(); 
    return r;
}
void updata(int x,ll vl) {mn[x]+=vl;tag[x]+=vl;}
void pushup(int x) {mn[x]=min(mn[x<<1],mn[x<<1|1]);}
void pushdown(int x) {if(tag[x]) {updata(x<<1,tag[x]);updata(x<<1|1,tag[x]);tag[x]=0;}}
void modify(int L,int R,ll vl,int l=1,int r=n+1,int x=1) {
    if(L<=l&&r<=R) { updata(x,vl);return ;}
    int mid=(l+r)>>1;pushdown(x);
    if(L<=mid) modify(L,R,vl,l,mid,x<<1);
    if(R>mid) modify(L,R,vl,mid+1,r,x<<1|1);
    pushup(x);
}
ll query(int L,int R,int l=1,int r=n+1,int x=1) {
    if(L<=l&&r<=R) {return mn[x];}
    int mid=(l+r)>>1;pushdown(x);ll ret=inf;
    if(L<=mid) ret=query(L,R,l,mid,x<<1);
    if(R>mid) ret=min(ret,query(L,R,mid+1,r,x<<1|1));
    return ret;
}
struct rec {
    int l,r,c;
    rec(int _l,int _r,int _c):l(_l),r(_r),c(_c){}
    rec(){}
};
struct sta {
    rec a[N];int tot=0;
    int size() {return tot;}
    bool empty() {return tot==0;}
    rec top() {return a[tot];}
    void pop() {tot--;}
    void push(rec x) {a[++tot]=x;}
}smx,smn;
int main() {
    n=red(),W=red();
    for(int i=1;i<=n;i++) a[i]=red(),s[i]=s[i-1]+a[i];
    int Lp=0;
    f[0]=0;
    modify(1,1,s[n]);
    for(int i=1;i<=n;i++) {
        int tmpr=i,tmpl=i;
        while(!smx.empty()&&smx.top().c<=a[i]) {
            rec t=smx.top();smx.pop();
            tmpl=t.l;
            modify(t.l,t.r,a[i]-t.c);
        }
        smx.push(rec(tmpl,tmpr,a[i]));
        tmpl=i;
        while(!smn.empty()&&smn.top().c>=a[i]) {
            rec t=smn.top();smn.pop();
            tmpl=t.l;
            modify(t.l,t.r,t.c-a[i]);
        }
        smn.push(rec(tmpl,tmpr,a[i]));
        while(s[i]-s[Lp]>W) Lp++; 
        f[i]=query(Lp+1,i);
        modify(i+1,i+1,f[i]+s[n]-s[i]);
    }
    printf("%lld\n",f[n]);
    return 0;
}
```
