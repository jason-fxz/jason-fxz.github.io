---
title: '小 w 的魔术扑克'
date: 2020-09-21 22:10:44
tags: [图论,XJ]
categories: 题解
hidden: true
---

## 题面

### 题目描述

小 w 喜欢打牌，某天小 w 与 dogenya 在一起玩扑克牌，这种扑克牌的面值都在 1 到 n，原本扑克牌只有一面，而小 w 手中的扑克牌是双面的魔术扑克（正反两面均有数字，可以随时进行切换），小 w 这个人就准备用它来出老千作弊。小 w 想要打出一些顺子，我们定义打出一个 l 到 r 的顺子需要面值为从 l 到 r 的卡牌各一张。小 w 想问问你，他能否利用手中的魔术卡牌打出这些顺子呢？

<!--more-->

### 输入描述

首先输入一行 2 个正整数 n，k，表示牌面为 1∼n，小 w 手中有 k 张魔术扑克牌。
然后输入 k 行，每行两个数字，表示卡牌的正面和反面的面值。
接下来输入一行一个正整数 q，表示 q 组查询，然后每组占一行查询输入两个整数 l,r。表示查询小 w 能否打出这么一个 l 到 r 的顺子。

### 输出描述

对于输出"`Yes`"表示可以，"`No`"表示不可以。（不含引号）
每个查询都是独立的，查询之间互不影响。

### 数据范围

对于 $100\%$ 的测试点，保证 $1\leq n\leq 10^5,1\leq k\leq 10^5,1\leq q\leq 10^5,1\leq l\leq r\leq n$。

### 示例

**输入**:

```
5 3
1 2
2 3
4 4
3
1 2
2 4
1 4
```

**输出**:
```
Yes
Yes
No
```
**说明**
对于顺子1~2，可以选择第一张卡牌作为'1'使用，选择第二张卡牌作为'2'使用。
对于顺子2~4，可以选择第一张卡牌作为'2'使用，选择第二张卡牌作为'3'使用，选择第三张卡牌作为'4'使用。
对于顺子1~4，由于牌的数目都不够，显然无法打出。


**输入**:
```
4 3
1 1
2 2
4 4
3
1 2
1 4
4 4
```
**输出**:
```
Yes
No
Yes
```


## 题解

初看此题，dp？数据结构？二分图？然后发现行不通。

于是考虑建图：对于每一张牌，将其对应点数连起来，示例1 就搞成这样：
<img src="https://s1.ax1x.com/2020/09/18/w4DLl9.png" width="300px"></img>

让每张牌对应一个数等价与让没条边指向一个点:
<img src="https://s1.ax1x.com/2020/09/18/w4DqSJ.png" width="300px"></img>

建完边后我们得到了若干个联通块，若一个联通块中 *边数* $\ge$ *点数* ,那么显然该联通块内所有点都能取到。
需要我们考虑的，是当联通块为树的时候，此时 $m$ 个点 $m-1$ 条边，显然所有点不能被同时取，记此联通块中最大的点为 $MAX$ ，最小的点为 $MIN$ ，那一定不存在 $MIN$ ～ $MAX$ 的顺子。因为要取 $MIN$ ～ $MAX$ 的话这整颗树都要，然而必然有一点取不到，故不存在方案。
进一步的，包含 $MIN$ ～ $MAX$ 的询问都不可行。

于是我们只需找到为树的联通块，标记一段区间 $[\;MIN\,,\,MAX\;]$  ，询问时判断是否包含任意被标记的区间即可。

找联通块及联通块内的最值用并查集维护，如何标记区间呢，对于区间 $[l,r]$ 可以记录 $c[l]=r$ ，再对 $c$ 进行后缀最小值，对于询问 $L,R$ ，若 $c[L]\le R$ 即包含了一个区间，否则不包含。

时间复杂度 $\mathcal{O}(n)$。

## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5+10;
int head[N],nxt[N<<1],pnt[N<<1],E;
int n,m,q,fa[N],mx[N],mn[N],sz[N],chk[N];
bool ok[N];
int find(int u) {return u==fa[u]?u:fa[u]=find(fa[u]);}
void ut(int u,int v) {
    int fu=find(u),fv=find(v);
    if(fu==fv) {ok[fu]=1;return ;} 
    if(sz[fv]>sz[fu]) swap(fu,fv);
    fa[fv]=fu;sz[fu]+=sz[fv];
    mx[fu]=max(mx[fu],mx[fv]);
    mn[fu]=min(mn[fu],mn[fv]);
}
int main() {
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)fa[i]=mx[i]=mn[i]=i,sz[i]=1;
    for(int i=1;i<=m;i++) {
        int u,v;scanf("%d%d",&u,&v);
        ut(u,v);
    }
    memset(chk,0x3f,sizeof(chk));
    for(int i=1;i<=n;i++) {
        int u=find(i);
        if(ok[u]) continue;
        ok[u]=1;
        chk[mn[u]]=mx[u];
    }
    for(int i=n;i>=1;i--) chk[i]=min(chk[i],chk[i+1]);
    scanf("%d",&q);
    for(int i=1;i<=q;i++) {
        int l,r;scanf("%d%d",&l,&r);
        printf("%s\n",(r<chk[l])?"Yes":"No");
    }
}
```