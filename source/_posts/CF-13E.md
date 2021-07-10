---
title: 'CF-13E - Holes | 弹飞绵羊'
date: 2020-11-18 23:17:17
tags: [分块]
categories:
- 题解
---


[CF-13E](http://codeforces.com/problemset/problem/13/E)  
[HNOI2010-弹飞绵羊](https://www.luogu.com.cn/problem/P3203)

双倍经验～～

## 题意

有 $N$ 个洞，每个洞有相应的弹力 $k_i$，能把这个球弹到 $i+k_i$ 的位置。当球的位置 $\gt N$时即视为被弹出

$M$ 次询问，共有两种操作:
1. $0\;a\;b$ 把 $a$ 位置的弹力改成 $b$
2. $1\;a$ 在 $a$ 处放一个球，输出最后一次落在哪个洞，球被弹出前共被弹了多少次。

$1\le N,M\le 100,000$


<!--more-->

## 题解

naive的想法可以暴力维护 $f_{i}$ 表示最后跳到的点，$c_i$ 表示步数。

$$
f_i=\begin{cases}
f_{i+k_i} & i+k_i\le n\\
i & i+k_i\gt n
\end{cases}
\quad
c_i=\begin{cases}
c_{i+k_i}+1 & i+k_i\le n\\
1 & i+k_i\gt n
\end{cases}
$$

预处理 $O(n)$ ，修改一个点 $O(1)$ ，然而改了一个点后，能跳到它的点都要更新，更新关系是一个树状结构，$fa_i=i+k_i$ ，操作 2 让 $fa_a=a+b$ ，那 $a$ 的整颗子树都会受到影响，LCT？可惜我不会。。。

考虑分块，块的大小为 $S$ ，在块内维护不同跳入点跳出需要的步数及跳出后到达的点，再维护一个跳出前的一个点，更上面的式子没啥区别，从后往前更新就形了，块内更新 $O(S)$ ，预处理 $O(n)$。

至于查询，也挺简单，从一个块跳到下一个块是 $O(1)$ 的，跳出 $N$ 是特判一下，返回跳出前的点。最多跳 $\frac{n}{S}$ 个块，查询复杂度 $O(\frac{n}{S})$

姑且认为查询和修改次数同阶， $S$ 取 $\sqrt{n}$ 时总复杂度 $O(n+m\sqrt{n})$ 

## CODE

```cpp
#include <bits/stdc++.h>
#define re register
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 100010;
const int B = 320;
int por[N],n,m;
int L[N],R[N],tot,bl[N],T;
//jp[p][i] 在块 p 的第i个位置，跳ct[p][i]次跳出块且到 jp[p][i] 的位置 ,pr[p][i] 为跳出之前位置
int jp[B][B],pr[B][B],ct[B][B]; 
int ask(int &pos) {
    int cnt=0;
    while(1) {
        int np=bl[pos];
        int nxt=jp[np][pos-L[np]];
        cnt+=ct[np][pos-L[np]];
        if(nxt>n) {
            pos=pr[np][pos-L[np]];
            break;
        }
        pos=nxt;
    }
    return cnt;
}
void updata(int p) {
    for(re int i=R[p];i>=L[p];--i) {
        re int id=i-L[p];
        if(i+por[i]>R[p]) {
            pr[p][id]=i;
            jp[p][id]=i+por[i];
            ct[p][id]=1;
        }else {
            pr[p][id]=pr[p][id+por[i]];
            jp[p][id]=jp[p][id+por[i]];
            ct[p][id]=ct[p][id+por[i]]+1;
        }
    }
}
int main() {
    red(n);red(m);
    for(re int i=1;i<=n;++i) red(por[i]);   
    T=B-1;tot=n/T;
    for(re int i=1;i<=tot;++i) L[i]=(i-1)*T+1,R[i]=i*T;
    if(R[tot]<n) {
        L[tot+1]=R[tot]+1;
        R[++tot]=n;
    }
    for(re int p=1;p<=tot;++p) {
        for(re int i=L[p];i<=R[p];++i) bl[i]=p;
        updata(p);
    }
    for(re int ts=1;ts<=m;++ts) {
        re int op,a,b;red(op);red(a);
        if(op==0) {
            red(b);
            por[a]=b;
            updata(bl[a]);
        }else {
            int r=ask(a);
            printf("%d %d\n",a,r);
        }
    }
    return 0;
}
```
