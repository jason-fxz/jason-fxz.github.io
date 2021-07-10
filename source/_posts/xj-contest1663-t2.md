---
title: 'XJ-contest1663 T2 『异或』'
date: 2020-09-14 22:09:11
tags: [trie树,XJ]
mathjax: true
categories: 题解
hidden: true
---

## 题面

### 题目描述

有 $n$ 个数字排成一排，我们定义一个区间的价值为这个区间内所有数字的异或和，那么有多少区间的价值不小于 $k$ 呢？

<!--more-->

### 输入格式

第一行用空格隔开的两个正整数 $n$ 和 $k$。
接下来一行 $n$ 个数字。

### 输出格式

一个数表示答案。  

### 数据范围:

对于$30\%$的数据，$n\leq100$。  
对于$50\%$的数据，$n\leq1000$。  
对于$100\%$的数据，$n\leq10^6$，数字,$k\leq10^9$。  

### 输入样例

**样例一**
```
3 2 
1 2 3
```
**样例二**
```
3 3 
8 8 4
```
**样例三**
```
3 3 
8 2 1
```
### 输出样例
**样例一**
```
3
```
**样例二**
```
5
```
**样例三**
```
4
```

---

## 题解
首先异或和可以用前缀和优化，记 $s_n=\bigoplus\limits_{i=1}^n a_i$ ，那 $l$ 到 $r$ 这一段的异或值为 $s_r\bigoplus s_{l-1}$ ，证明显然 （因为 $a\bigoplus a=0$）

于是我们便想到枚举 $r$ ，找有多少个 $l$ 是符合条件的，即 $s_r \bigoplus s_{l-1} \ge k$ 。如果只是等于 $k$ 可以想到用 01trie 存下 $s_i\; ,(i<r)$，每次找 $k\bigoplus s_r$ 有几个。

那大于 $k$ 如何考虑? 
比 $k$ 大？（记这个数为$t$），显然，如果高位相同且当前某位 $k$ 是 $0$，$t$ 是 $1$ ，那低位 $t$ 无论取什么都比 $k$ 大，这正好对应着 01trie 上某一个中间节点（高位确定，低位随意）。

于是乎我们只要按照 $k\bigoplus s_r$ 在 01trie 上搜 ，若 $k$ 的当前位为 $0$ 则答案加上这一位取反在 01trie 上统计的数量。 （详见代码）


## code
```cpp
#include <bits/stdc++.h>
#define re register
using namespace std;
const int N = 1000010;
int ch[N*31][2],siz[N*31],tot; 
int n,k,s[N];
inline int red() {
    int r=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') r=r*10+ch-'0',ch=getchar();
    return r;
}
void insert(int x) { //trie 插入
    int p=0;bool d;
    for(re int i=30;i>=0;--i) {
        d=(x>>i)&1;
        if(!ch[p][d]) ch[p][d]=++tot;
        p=ch[p][d];
        siz[p]++;
    }
}
int find(int x) {
    re int p=0,res=0;bool d;
    for(re int i=30;i>=0;--i) {
        d=(x>>i)&1;
        if(!((k>>i)&1)) res+=siz[ch[p][d^1]]; //若k这一位为0，可将取1，这样无论后面怎么取都大于k
        if(ch[p][d]==0) break;
        p=ch[p][d];
        if(i==0) res+=siz[p]; //s[i] xor k 的数量 
    }
    return res;
}
int main() {
    scanf("%d%d",&n,&k);
    for(re int i=1;i<=n;++i) {
        s[i]=red();
        s[i]^=s[i-1]; //异或前缀和
    }
    insert(0);
    long long ans=0;
    for(re int i=1;i<=n;++i) {
        ans+=find(s[i]^k); //找满足条件l的个数
        insert(s[i]); 
    }
    printf("%lld\n",ans);
}
```