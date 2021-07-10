---
title: XJ-交通运输
date: 2020-11-18 23:06:02
tags: [XJ,动态规划,dp优化,斜率优化]
categories: 题解
hidden: true
---


[XJ-交通运输](https://dev.xjoi.net/contest/1596/problem/2)

<!--more-->

## 题意
![](./P1.png)

**数据范围：** $1\le n \le 10^6 ,1\le maxc\le 10,1\le c_i\le maxc,1\le v_i\le 1000$


## 题解

先考虑一个暴力 $dp_i$ 表示到 $i$ 的最小花费。显然有一下转移方程：
$$
dp_i=\min_{0\le j<i,\,c_i\mid(i-j)} dp_{j}+\frac{i-j}{c_j}v_j
$$

注意这里要 $c_i\mid (i-j)$ ，即 $j$ 能跳到 $i$ 。看到式子中有 $i,j$ 相关的乘积，于是考虑斜率优化，展开得： 
$$
dp_i=dp_j+\lfloor\tfrac{i}{c_j}\rfloor v_j-\lfloor\tfrac{j}{c_j}\rfloor v_j
$$
注意到这里直接**下取整**，因为 $maxc$ 很小，我们直接枚举 $c$ ，然后对于每个 $c$ 再按照余数分类，也就是说我们保证 $i\equiv j \pmod c$，故这里的下取整也没问题了。

移项，改成斜截式：

$$
\underline{dp_j-\lfloor\tfrac{j}{c_j}\rfloor v_j}_{y} =\underline{-\lfloor\tfrac{i}{c_j}\rfloor}_k\cdot \underline{v_j}_{x}+\underline{dp_i}_{b}
$$

一个决策点为 $(v_j,dp_j-\lfloor\tfrac{j}{c_j}\rfloor v_j)$ ，斜率 $k=-\lfloor\tfrac{i}{c_j}\rfloor$ 随 $i$ 单调减，维护下凸壳更新答案可以线性。注意这里枚举 $c$ ，并按照除 $c$ 的余数再分不同的下凸壳分别维护。

考虑如何加点，$v_j$ 貌似没有单调性，但是考虑若 $j \lt k,v_j\ge v_k$，显然 $k$ 决策点应该比 $j$ 不劣，$k$ 跳到 $i$ 会经过 $j$ ，在 $j$ 处换单价更小的车显然不劣。故加点时可以把 $\ge v_j$ 的都弹掉。

## 细节

~~是那踩过无数次的坑~~

貌似斜率优化式子推好，经常容易错的就这几个

1. 斜率：这里斜率是**递减**的，故决策点是单调不增的，取队尾
2. 点间斜率比较，一种直接计算返回 `double` ，这种方法的缺点是掉精度，有时会出问题。另一种写两函数 $dx(i,j),dy(i,j) $返回 `long long`，化除为乘，精度没问题，一般来说也不会炸 `long long` ，但要注意 **乘上负数，不等式要变号** 

## CODE

```cpp
#include <bits/stdc++.h>
#define SZ(p) ((int)p.size())
#ifdef DEBUG
    #define see(x) cout<<#x<<"="<<x<<" "
    #define seeln(x) cout<<#x<<"="<<x<<endl
    #define put(x) cout<<x
#else 
    #define see(x) ;
    #define seeln(x) ;
    #define put(x) ;
#endif
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
typedef vector<int> vi;
typedef long long ll;
const int N = 1000010;
const int M = 11;
const int inf = 0x3f3f3f3f;
int n,maxc;
int dp[N],v[N],c[N];
vi que[M][M]; // que[c][r] 除数为c，除c数余r 的凸壳
void umin(int &x,int y) {if(x>y) x=y;}
ll dx(int i,int j) {
    return v[i]-v[j];
}
ll dy(int i,int j,int d) {
    return (ll)dp[i]-(ll)(i/d)*v[i]-(ll)dp[j]+(ll)(j/d)*v[j];
}
int main() {
    red(n);red(maxc);
    for(int i=0;i<n;i++) {
        red(c[i]);red(v[i]);
    }
    memset(dp,0x3f,sizeof(dp));
    dp[0]=0;
    for(int i=0;i<=n;i++) {
        // updata 
        if(i!=0) {
            for(int d=1,r;d<=maxc;d++) {
                r=i%d;
                vi &q=que[d][r];
                while(SZ(q)>=2&&(-(i/d))*dx(q[SZ(q)-1],q[SZ(q)-2])<=dy(q[SZ(q)-1],q[SZ(q)-2],d)) q.pop_back(); 
                if(SZ(q)) {
                    int j=q.back();
                    umin(dp[i],dp[j]+(i/d)*v[j]-(j/d)*v[j]);
                }
            }
        }

        // modify 
        if(dp[i]==inf||i==n) continue;
        int r=i%c[i];
        vi &q=que[c[i]][r];
        while(SZ(q)&&v[q.back()]>=v[i]) q.pop_back();
        // 这里一开始 “<=” 搞反了，乘负数不等式要变符号！！！！（又一次掉坑） 
        while(SZ(q)>=2&& dy(q[SZ(q)-1],q[SZ(q)-2],c[i])*dx(q[SZ(q)-1],i)<=dy(q[SZ(q)-1],i,c[i])*dx(q[SZ(q)-1],q[SZ(q)-2]))
        q.pop_back();
        q.push_back(i); 
    }
    for(int i=1;i<=n;i++) {
        printf("%d ",dp[i]==inf?-1:dp[i]);
    }
    printf("\n");
    return 0;
}
```
