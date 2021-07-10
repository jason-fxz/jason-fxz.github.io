---
title: '[牛客] 排列计数机'
date: 2020-09-25 21:24:04
tags: [线段树,动态规划,数学]
categories:
- 题解
---


[『牛客』排列计数机](https://ac.nowcoder.com/acm/problem/54000)

## 题意

定义一个长为 $k$ 的序列 $A_1,A_2,\dots,A_k$ 的权值为：对于所有 $1\le i\le k$，$\max(A_1,A_2,\dots,A_i)$ 有多少种不同的取值。
给出一个 $1$ 到 $n$ 的排列 $B_1,B_2,\dots,B_n$，求 $B$ 的所有非空子序列的权值的 $m$ 次方之和。
答案对 $10^9+7$ 取模。

**数据范围:**  $1\le n \le 10^5 ,1\le m \le 20,$ 保证 $B$ 是 $1$ 到 $n$ 的排列。



<!--more-->

## 题解

这种序列上的问题多是DP。

### 暴力DP

首先先考虑一个比较暴力的DP

$f_{v,j}$ 表示最大值为 $v$ ，序列的权值为 $j$ 的方案数。按顺序枚举 $i$ ：
$$
f_{A_i,j}=\sum_{k\in[1,A_i-1]} f_{k,j-1} \quad ,j\in[2,i]\\
f_{A_i,1}=1\\ \tag{1}
$$

即：前面所有最大值小于 $A_i$ 的序列末端加上 $A_i$ ，会多一的权值； $A_i$ 本身构成一种权值为 $1$ 的序列。
$$
f_{v,j}=f_{v,j}\times 2 \quad,v\in[A_i+1,n],j\in[1,i]  \tag{2}
$$

即：前面所有最大值小于 $A_i$ 的序列末端加上 $A_i$ ，权值不会改变，但方案数多了一倍。

统计答案：
$$
ans=\sum_{v=1}^n \sum_{j=1}^n f_{v,j}\times j\,^m
$$
复杂度 $\mathcal{O}(n^3)$



### 正解

答案肯定形如 $k_1\cdot 1^m+k_2\cdot 2^m+k_3\cdot 3^m+\dots+ k_n\cdot n^m$  。其中 $k_i$ 表示权值为 $i$ 的序列有多少个。  

设 $dp_{v,j}$ 表示序列最大值为 $v$ ，所有方案权值的 $j$ 次方和 , （即 $k_1'\cdot 1^j+k_2'\cdot 2^j+k_3'\cdot 3^j+\dots+ k_n'\cdot n^j$ ）。并且用线段树来维护 $dp_{v,j}$ 。（具体得说：开 $m$ 棵线段树维护 $v$ 这维区间 $j$ 次和）

参考上面提到的转移，对于转移 $f_{v,j}=f_{v,j}\times 2 \quad,v\in[A_i+1,n],j\in[1,i]$  ：
$$
dp_{v,j}\leftarrow k_1'\times 2\cdot 1^j+k_2'\times 2\cdot 2^j+k_3'\times 2\cdot 3^j+\dots+ k_n'\times 2\cdot n^j\\
=dp_{v,j}\times 2\\
v\in[A_i+1,n],j\in[0,m]
$$
相当于在 $m$ 颗线段树中把 $[A_i+1,n]$ 的区间都乘二

对于转移 $f_{A_i,j}=\sum_{k\in[1,A_i-1]} f_{k,j-1} \quad ,j\in[2,i]$ ，意思就是所有最大值小于 $A_i$ 的方案都可以更新  $dp_{A_i}$

先记 $S_j=\sum\limits_{v\in[1,A_i-1]} dp_{v,j}$ 表示最大值小于 $A_i$ 所有方案权值 $j$ 次方和，按其含义也可以写成这样 $S_j=k_1\cdot 1^j+k_2\cdot 2^j+\dots+k_n\cdot n^j$ ，其中 $k_i$ 表构成权值为 $i$ 的序列的方案数。现在用所有最大值小于 $A_i$ 的方案更新 $A_i$ ，每种方案序列的权值都会加一，相当于使 $k_i$ 表示权值为 $i+1$ 的序列的方案数，即：
$$
dp_{A_i,j}\leftarrow k_1\cdot(1+1)^j+k_2\cdot(2+1)^j+\dots+k_n\cdot(n+1)^j+1
$$
（$+1$ 是因为只有 $A_i$ 也是一种序列方案且权值为 $1$），其中有形如 $(x+1)^j$ 的式子，根据二项式展开定理：
$$
(x+1)^j= \binom{j}{0} x^j+\binom{j}{1}x^{j-1}+\dots +\binom{j}{j}x^{0}\\
(x+1)^j= \sum\limits_{i=0}^{j} \binom{j}{j-i} x^i
$$

可进行如下化简：
$$
k_1\cdot(1+1)^j+k_2\cdot(2+1)^j+\dots+k_n\cdot(n+1)^j+1\\
=k_1\cdot\sum\limits_{i=0}^{j} \binom{j}{j-i} 1^i+k_2\cdot\sum\limits_{i=0}^{j} \binom{j}{j-i} 2^i+\dots+k_n\cdot\sum\limits_{i=0}^{j} \binom{j}{j-i} n^i\\
= \sum\limits_{i=0}^{j} \binom{j}{j-i} \left(k_1\cdot 1^i+k_2\cdot2^i+\dots+k^n\cdot n^i\right)\\
=\sum\limits_{i=0}^{j} \binom{j}{j-i} \;S_i
$$
总结一下，按顺序扫 $A_i$ 。对于每个 $A_i$ ：

+ $m$ 颗线段树中 $[A_i+1,n]$ 的部分全部乘二
+ 先处理出 $S_j=\text{第j颗线段树中} [1,A_i-1]\text{的区间和}$ ，将第 $j$ 棵树上的 $[A_i,A_i]$ 改为 $\left(\sum\limits_{k=0}^{j} \binom{j}{j-k} \;S_k\right)\;+1$

复杂度 $\mathcal{O}(nm^2+nm\log n)$



## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 100010;
const int M = 21; 
const ll mod = 1e9+7;
int n,m,a[N];
ll sum[N<<2][M],tag[N<<2][M],C[M][M];
inline int red() {
    int r=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') r=(r<<1)+(r<<3)+(ch^48),ch=getchar();
    return r;
}
void pushup(int x,int k) {
    sum[x][k]=(sum[x<<1][k]+sum[x<<1|1][k])%mod;
}
void updata(int x,int k,int vl) {
    (sum[x][k]*=vl)%=mod;
    (tag[x][k]*=vl)%=mod;  
}
void pushdown(int x,int k) {
    updata(x<<1,k,tag[x][k]);
    updata(x<<1|1,k,tag[x][k]);
    tag[x][k]=1;
}
ll query(int p,int k,int x=1,int l=1,int r=n) { //第k颗线段树，询问小于等于p位置的和
    if(r<=p) return sum[x][k];
    int mid=(l+r)>>1;pushdown(x,k);
    return (query(p,k,x<<1,l,mid)+(p>mid?query(p,k,x<<1|1,mid+1,r):0))%mod;
}
void multi(int p,int k,int x=1,int l=1,int r=n) { //第k颗线段树，大于等于p位置的乘二
    if(r<p)return;
    if(l>=p) {updata(x,k,2);return;}
    int mid=(l+r)>>1;pushdown(x,k);
    if(p<=mid) multi(p,k,x<<1,l,mid);multi(p,k,x<<1|1,mid+1,r);
    pushup(x,k);
}
void modify(int p,int k,int vl,int x=1,int l=1,int r=n) {  //第k颗线段树，单点修改
    if(l==r) {sum[x][k]=vl;return;}
    int mid=(l+r)>>1;pushdown(x,k);
    if(p<=mid) modify(p,k,vl,x<<1,l,mid);else modify(p,k,vl,x<<1|1,mid+1,r);
    pushup(x,k);
}
void init() {
    C[0][0]=1;
    for(int i=1;i<=m;i++) {
        C[i][0]=1;
        for(int j=1;j<=i;j++)C[i][j]=(C[i-1][j-1]+C[i-1][j])%mod;
    }
    for(int i=1;i<=(n<<2);i++) for(int j=0;j<=m;j++) tag[i][j]=1;
}
ll tmp[M],tp[M];
int main() {
    n=red(),m=red();
    init();
    for(int i=1;i<=n;i++) {
        a[i]=red();
        memset(tp,0,sizeof(tp));
        memset(tmp,0,sizeof(tmp));
        for(int j=0;j<=m;j++) tp[j]=query(a[i],j);
        for(int j=0;j<=m;j++) {
            for(int k=0;k<=j;k++) {
                (tmp[j]+=C[j][j-k]*tp[k]%mod)%=mod;
            }
        }
        for(int j=0;j<=m;j++) {
            modify(a[i],j,tmp[j]+1);
            multi(a[i]+1,j);
        }
    }
    printf("%lld\n",query(n,m));
}
```
