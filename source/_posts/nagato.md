---
title: ' 长门有希的序列(nagato)'
date: 2020-10-06 21:52:01
tags: [线段树,树状数组]
mathjax: true
categories:
- 题解
---

~~某联考题，长门有希的序列(nagato)~~
Update 2021-4-23: 发现原题，[[THUSC2015]平方运算](https://www.luogu.com.cn/problem/P4681)

[跳过废话](##题意简述)

## 题目背景~~都是废话~~

> 「他一见钟情的对象,并不是我。」
语调平稳得像是在念论文。
「他看到的我并不是我,而是资讯统合思念体。」
我静静聆听。长门又以同样的语调继续说明。
「只是他不晓得他看到的是什么。毕竟人类只是有机生命体,在意识层面上和资讯统合思念体是
天差地远。恐怕他看到的是那超越时空的智慧与日积月累的知识吧。尽管他透过终端机读取到的资讯
仅有九牛一毛,但那资讯带来的压力已足以令他为之倾倒。」
所以他才会会错意......吗?我看着长门参差不齐的刘海,叹了一口气。中河感受到的长门内在,
事实上只是资讯统合思念体的某一端。虽然我不是很清楚,但是长门的确实拥有人类无法比拟的庞大
历史、知识量等奇妙的力量。一不小心误闯进去的中河,为何会茫然自失一点也不足为奇了。
......
「我还是有点好奇,你到底是怎么消除他拥有的能力的呢? 」
尽管清楚那是人类所无法理解的知识,我仍然不自量力地提问。也许我也像中河那样抓狂了吧。
「那并不复杂,我可以告诉你,也许你可以理解。」
你是认真的?我有点不敢相信自己的耳朵。
「当然。他的超感应能力归根结底还是因为三年前凉宫春日造成的资讯爆炸,所以他的能力在我
们看来也是以资讯的形式存在的。在人类这样的有机生命体中,受制于形式这些资讯只能以链状的形
式彼此相连。而与资讯统合思念体交互时,总是链中的一段正在发挥作用。要么是资讯统合思念体让
这些资讯全部变成它们的平方,要么是这些资讯将它们的总和发送给资讯统合思念体......」
长门的嘴一张一合,平稳的语调中传达出大量的信息,话语中提到的链状结构总让我回想起生物
课上学到的 RNA。春日在生物课上常常发呆,但是她的成绩却比我高得多。
但果然还是听不懂啊,我让长门不用继续说下去了,这些信息只能使我的脑子拧成一团乱麻。
长门低下了头,数秒后又抬起头注视着我,神情中多了一丝落寞,她的眉毛下降了几毫米,那是
只有我才能察觉到的差异。
凛冽的夜风吹得我耳畔刺痛,颤抖的双腿不禁后退了半步,但视野中长门的身影却突然潜入黑
暗,像是快要溶解进虚空中的幽魂。

<!--more-->

## 题意简述

给你一个长度为 $n$ 的整数数列，初始值为 $a_1,a_2,a_3,\dots,a_n$，现在有 $q$ 次操作，有以下两种形式：

1. $1\;l\;r$ ：将区间 $[l,\,r]$ 中的每个元素变为自身的平方。即对每个 $l \le i \le r$,执行 $a_i \leftarrow a^2_i$ 。
2. $2\;l\;r$ ： 询问区间 $[l,\,r]$ 中的所有元素的和,对 $p = 998244353$ 取模。即输出 $\left(\sum\limits_{i=l}^r a_i\right) \bmod p$ 。

$1\le n,q\le 2\times 10^5,\;1\le a_i \lt p,\;op\in\{1,2\},\;1\le l \le r\le n$ 。

## 题解

首先肯定需要观察以下，对于 $a_i$ ，若其平方了 $k$ 次，则它的贡献为  $\large a_i^{2^k} \bmod p$ ，用一下欧拉定理，对指数取模（$\varphi(998244353)=998244352$）： $\large a_i^{2^k \bmod (p-1)} \bmod p$ 。

然后你打个 $2^k \bmod (p-1)$ 的表，就会惊奇的发现 $k\ge 23$ 之后就循环了，循环节为 $24$ ，打表数据为证：

```plain
0 1
1 2
2 4
3 8
4 16
5 32
6 64
7 128
8 256
9 512
10 1024
11 2048
12 4096
13 8192
14 16384
15 32768
16 65536
17 131072
18 262144
19 524288
20 1048576
21 2097152
22 4194304
23 8388608  <----
24 16777216
25 33554432
26 67108864
27 134217728
28 268435456
29 536870912
30 75497472
31 150994944
32 301989888
33 603979776
34 209715200
35 419430400
36 838860800
37 679477248
38 360710144
39 721420288
40 444596224
41 889192448
42 780140544
43 562036736
44 125829120
45 251658240
46 503316480
47 8388608  <----
```

~~于是乎一切都好搞了~~

具体的：

1. 还没循环的 $a_i$ ，用树状数组维护区间和，修改的话暴力把每个 $a_i$ 平方。并用 `set` 维护还没到循环的 $a_i$ 的下标，（不开O2小心TLE，~~STL常数真的大~~）。

2. 对于开始循环的 $a_i$ ，用线段树维护其和，每个 $a_i$ 有 $24$ 个取值，对其平方一次相当于取下一个，修改操作就区间集体取下一个。$sum[x][0～23]$ 是线段树中一个节点，$sum[x][0]$ 是当前位，其他是候选位， $k$ 次操作就是让其转 $k\bmod 24$ 个位子，具体的： $\sum_{i=0}^{24} sum'[x][i] \leftarrow sum[x][(i+k)\% 24]$ 。至于标记，记一个 $tag[x]$ 表该新增区间操作次数即可。

具体见代码：

```cpp
#pragma GCC optimize(2)
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef set<int>::iterator si;
const int mod = 998244353;
const int N = 200010;
const int M = 50;
const int D = 24;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
// 2^(0~22) 不循环，23之后开始循环 (循环节 24)
ll sum[N<<2][D],tag[N<<2],tmp[D],a[N];
set<int> S; // 储存还没到循环的ai
int n,q,cnt[N]; 
struct trearr {
    ll tr[N];
    void add(int x,ll v){for(;x<=n;x+=x&-x) tr[x]=(tr[x]+v)%mod;}
    ll sum(int x) {ll r=0;for(;x>0;x-=x&-x) r=(r+tr[x])%mod;return r;}
}t;
int st[N],top;
// 执行操作 sum'[x][i] ← sum[x][(i+tt)%24]
void updata(int x,int tt) {
    tag[x]+=tt;tt%=D;
    for(int i=0;i<D;i++) tmp[i]=sum[x][(i+tt)%D];
    for(int i=0;i<D;i++) sum[x][i]=tmp[i];
}
// 下传标记
void pushdown(int x) { 
    tag[x]%=D;
    if(tag[x]) { updata(x<<1,tag[x]); updata(x<<1|1,tag[x]);tag[x]=0;}
}
void pushup(int x) {for(int i=0;i<D;i++) sum[x][i]=(sum[x<<1][i]+sum[x<<1|1][i])%mod;}
// 在线段树中插入进入循环的ai
void insert(int p,ll v,int l=1,int r=n,int x=1) {
    if(l==r) {
        for(int i=0;i<D;i++,v=(v*v)%mod) sum[x][i]=v;
        return ;
    }
    int mid=(l+r)>>1;pushdown(x);
    if(p<=mid) insert(p,v,l,mid,x<<1);
    else insert(p,v,mid+1,r,x<<1|1);
    pushup(x);
}
// 线段树上区间操作
void modify(int L,int R,int l=1,int r=n,int x=1) {
    if(L<=l&&r<=R) {
        updata(x,1);
        return ;
    }
    int mid=(l+r)>>1;pushdown(x);
    if(L<=mid) modify(L,R,l,mid,x<<1);
    if(R>mid) modify(L,R,mid+1,r,x<<1|1);
    pushup(x);
}
ll query(int L,int R,int l=1,int r=n,int x=1) {
    if(L<=l&&r<=R) return sum[x][0];
    int mid=(l+r)>>1;ll ret=0;pushdown(x);
    if(L<=mid) ret=query(L,R,l,mid,x<<1);
    if(R>mid) ret=(ret+query(L,R,mid+1,r,x<<1|1))%mod;
    return ret;
}
// 询问包含 树状数组 和 线段树 两者的答案
ll ask(int l,int r) {
    return (((t.sum(r)-t.sum(l-1))%mod+mod)%mod+query(l,r))%mod;
}
void change(int l,int r) {
    // 在循环开始之前暴力搞，树状数组维护即可
    si it=S.lower_bound(l);
    si jt=S.upper_bound(r);
    top=0;
    modify(l,r); // 在线段树中区间操作
    for(si p=it;p!=jt;p++) {
        cnt[*p]++;
        t.add(*p,-a[*p]);
        a[*p]=a[*p]*a[*p]%mod;
        t.add(*p,a[*p]);
        if(cnt[*p]>=23) st[++top]=*p; //已经到达循环的丢到线段树中，从树状数组中删除，见下
    }
    while(top) {
        int pos=st[top--];
        t.add(pos,-a[pos]);
        insert(pos,a[pos]);
        S.erase(pos);
    }
}
int main() {
    red(n);red(q);
    for(int i=1;i<=n;i++) red(a[i]),S.insert(i),t.add(i,a[i]);
    for(int i=1;i<=q;i++) {
        int op,l,r;red(op);red(l);red(r);
        if(op==1) {
            change(l,r);
        }else {
            printf("%lld\n",ask(l,r));
        }     
    }
}
```
