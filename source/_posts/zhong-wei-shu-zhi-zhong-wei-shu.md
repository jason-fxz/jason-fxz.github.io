---
title: 'XJ - 中位数之中位数'
date: 2020-09-19 11:34:30
tags: [二分,树状数组,XJ]
mathjax: true
categories: 题解
hidden: true
---


## 题意

给出一个长度为 $n$ 的序列 $a$，首先求出其所有区间的中位数，将这些中位数构成的集合记为 $S$，求 $S$ 中所有数的中位数。

<!--more-->

这里定义的中位数指：对于 $m$ 个数，将其从小到大排序后，第 $(\lfloor\frac{m}{2}\rfloor+1)$ 个数即为中位数，例如 $(10,30,20)$ 的中位数为 $20$，$(10,30,20,40)$ 的中位数为 $30$，$(10,10,10,20,30)$ 的中位数为 $10$ 。

对于$30\%$的数据: $1≤n≤300$
对于$50\%$的数据: $1≤n≤3000$
对于 $100\%$的数据: $1\le n\le 100000,1\le a_i\le10^9$

## 题解

### 50pts

暴力枚举区间，固定左端点，右端点从左往右，对顶堆维护中位数。
记录每个数作为中位数有多少个，复杂度 $\mathcal{O}(n^2\log n)$

### 正解

思路挺妙，

考虑二分枚举答案，对于当前的 $mid$ ，$\mathrm{check}(mid)$ 求出不大于 $mid$ 的中位数有多少个。

记 $f[i]$ 表示前 $i$ 个数中大于 $mid$ 的个数，那么满足 $r-l>2\times(f[r]-f[l])$ 的区间 $[l,r]$ 都可行（即大于 $mid$ 的数的个数不超过区间一半，那当前区间的中位数一定不大于 $mid$ ），展开移项：

$$
r-2\times f[r] > l-2\times f[l]
$$

然后就是一个顺序对问题，树状数组做即可。


## code

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100010;
typedef long long ll;
int n,a[N],b[N],f[N],tot;
inline void red(int &x) {
    x=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(ch^48),ch=getchar();
}
struct trearr {
    int tr[N<<2];
    void add(int x,int vl) {for(;x<=n*4;x+=x&(-x)) tr[x]+=vl;}
    int sum(int x) {int r=0;for(;x>0;x-=x&(-x)) r+=tr[x];return r;}
    void clear() {memset(tr,0,sizeof(tr));}
}t;
ll check(int mid) {
    t.clear();
    for(int i=1;i<=n;i++) {
        f[i]=f[i-1]+(a[i]>mid);
    }
    ll res=0;
    for(int i=1;i<=n;i++) {
        res+=t.sum(i-2*f[i]+2*n-1);
        t.add(i-2*f[i]+2*n,1);
    }
    return res;
}
int main() {
    red(n);
    for(int i=1;i<=n;i++) {red(a[i]);b[i]=a[i];}
    sort(b+1,b+n+1);tot=unique(b+1,b+n+1)-b-1;
    for(int i=1;i<=n;i++) a[i]=lower_bound(b+1,b+tot+1,a[i])-b;
    int L=1,R=tot,B=-1,M;
    ll pp=1ll*(n+1)*n/4ll+1ll;
    while(L<=R) {
        M=(L+R)>>1;
        if(check(M)>=pp) R=M-1,B=M; 
        else L=M+1;
    }
    printf("%d\n",b[B]);
}
```