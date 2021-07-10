---
title: '维护序列'
date: 2020-09-22 22:26:46
tags: [分块,XJ]
categories: 题解
hidden: true
---


[👇跳过题面](#题解)

## 题目描述
给出一个长度为 $n$ 的序列 $a_1,a_2,\dots,a_n$ 。你需要实现一个数据结构，支持以下操作：
- $1\;l \;r \;x$ 将 $a_l,a_{l+1},a_{l+2},\dots,a_r$ 修改为 $x$ 。
- $2\; x\; y$ 查询序列中所有满足 $a_i=x,a_j=y$ 的点对 $(i,j)$ 里 $|i−j|$ 的最小值（$i$ 和 $j$ 可以相同），如找不到满足条件的点对，输出 $-1$ 。

<!--more-->

### 输入
第一行输入两个正整数 $n,m,f (1\le n,m\le 10^5,f\in\{0,1\})$，表示序列长度、操作数，是否强制在线。$f=0$ 表示没有强制在线，$f=1$ 表示强制在线。

接下来输入一行 $n$ 个正整数 $a_1,a_2,\dots,a_n​ (1\le a_i\le n)$，描述初始序列。

接下来 $m$ 行，每行先输入一个数 $opt \;(opt\in\{1,2\})$，表示操作类型。

若 $opt=1$，则再输入三个正整数 $l,r,x$ 保证解密之后满足 $(1\le l\le r\le n,\,1\le x\le n)$，意义如题目所述；
若 $opt=2$，则再输入两个正整数 $x,y$ 保证解密之后满足 $(1\le x,\,y\le n)$，意义如题目所述。

若 $f=1$，你需要对输入 $l,r,x,y$ 都异或上次询问的答案 $lastans$ 进行解密，初始时 $lastans=0$。若 $lastans=−1$，则异或 $0$。

### 输出描述
对于每个 $opt=2$ 的操作，输出所求的最小值。

### 示例
**Input**

```
4 5 0
1 2 3 2
2 1 3
1 1 2 1
2 1 3
1 4 4 1
2 2 3
```
**Output**
```
2
1
-1
```


### 时空界限
**5s/512MB**

## 题解

时空界限开这么大，$n,m$ 也就 $10^5$ ，而这维护的东西又很神奇，故：~~显然~~是个分块

设每个块的大小为 $S$ ，一共 $\left\lceil \frac{n}{S} \right\rceil$ 个块

**块内维护啥？** 反正块小，~~想怎么暴力怎么来~~ ，块内直接维护**两两数间最小间隔**（一个块内最多 $S$ 种数），记 $f[p][i][j]$ (第p个块中i，j两数最小间隔，这里的i,j是块中数 映射后的下标)，每个块可以做到预处理复杂度 $\mathcal{O}(S^2)$ 。

具体做法就是扫一遍块中的数，边扫边更新某数最新的下标 ，同时for一遍块中元素，更新 $f$ ，代码如下：
```cpp
int lst[B];memset(lst,-1,sizeof(lst));
for(int i=L[p];i<=R[p];i++) {
    lst[id[p][a[i]]]=i; //id[p][a] 是某数在p块内的映射下标
    for(int j=1;j<=tot[p];j++) { //tot[p]是p块内不同数的个数
        if(lst[j]!=-1) {
            checkmin(f[p][id[p][a[i]]][j],i-lst[j]);
            checkmin(f[p][j][id[p][a[i]]],f[p][id[p][a[i]]][j]);
        }
    }
}
```

---


**注：** 这里提一下如何映射某数在某块内的下标：其实也很暴力，开一个 $id[p][i]$ 记 $i$ 在第 $p$ 个块中的下标，等于0则表示该块内没这个数。为了之后方便清空，再开一个栈记录哪几个数在块中出现即可。  ~~然后你会惊奇地发现~~，空间复杂度 $\mathcal{O}(\frac{n^2}{S})$ 不会MLE吗？没事，关于 $S$ 的取值我们后文再分析。

映射代码如下：
```cpp
for(int i=L[p];i<=R[p];i++) {
    if(!id[p][a[i]]) id[p][a[i]]=++tot[p],st[p][tot[p]]=a[i];
}
```


---

**那跨块的呢？** 可以维护某个数在块内位置的 **极右值和极左值**，方便查询。预处理比较方便，此处不赘述。


**如何查询？** 
- 块内的贡献已经用 $f$ 预处理好了，每个块直接访问便是了
- 跨块的显然是某两块 $l,r,(l< r)$ 中： $r$ 块中某数的极左值减去 $l$ 块中另一数的极右值。 做法与 $f$ 的预处理差不多

查询的复杂度 $\mathcal{O}(\frac{n}{S})$

**如何修改？** 
- 对于整块被覆盖的，我们记一个 $tag[p]$ 表示在p块内全是某数。
- 对于没有完全覆盖的块，可以直接暴力重构

修改的复杂度 $\mathcal{O}(\frac{n}{S}+S^2)$

**$S$ 取值?**

让我们来算一下时间复杂度，以便确定 $S$ 的取值，假设 $N,M$ 大小一样，最坏复杂度为 $\mathcal{O}(nS^2+\frac{n^2}{S})$ （预处理复杂度 $\mathcal{O}(nS)$ 可忽略，操作的复杂度都按修改算）

可以得出当 $nS^2=\frac{n^2}{S}$ 即 $S=\sqrt[3]{n}$ 时最优，此时复杂度为 $\mathcal{O}(n^{\frac{5}{3}})$

然后 n=1e5 代进去， 2e8能过？？？（话说还会MLE）
~~显然出题人不会那么毒瘤~~，修改次数不是太大的话，$S$ 可以取大一点 ， 实测 $S=87$ 时比较完美，空间也正好卡进。


说完了，上代码

## CODE
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100010;
const int M = 1164;
const int B = 87;
int id[M][N],st[M][B],tot[M]; // id第i个块中某数id，st存第i个块中的数
int f[M][B][B]; // 在第i个块中两数间最短距离
int g[M][B][2]; // 第i个块某数最左位置，最右位置
int L[M],R[M],T,tag[M];
int a[N],n,m,ff;
inline void red(int &x) {
    x=0;char ch=getchar();
    while(ch<'0'||ch>'9') ch=getchar();
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(ch^48),ch=getchar();
}
void checkmin(int &x,int y) {if(x==-1||x>y) x=y;}
void checkmax(int &x,int y) {if(x==-1||x<y) x=y;}
void build(int p) {
    while(tot[p]>0) id[p][st[p][tot[p]--]]=0;
    memset(f[p],0x3f,sizeof(f[p]));
    memset(g[p],-1,sizeof(g[p]));
    for(int i=L[p];i<=R[p];i++) {
        if(!id[p][a[i]]) id[p][a[i]]=++tot[p],st[p][tot[p]]=a[i];
        checkmin(g[p][id[p][a[i]]][0],i);
        checkmax(g[p][id[p][a[i]]][1],i);
    }
    int lst[B];memset(lst,-1,sizeof(lst));
    for(int i=L[p];i<=R[p];i++) {
        lst[id[p][a[i]]]=i;
        for(int j=1;j<=tot[p];j++) {
            if(lst[j]!=-1) {
                checkmin(f[p][id[p][a[i]]][j],i-lst[j]);
                checkmin(f[p][j][id[p][a[i]]],f[p][id[p][a[i]]][j]);
            }
        }
    }
}
int query(int x,int y) {
    int ans=-1;
    for(int p=1;p<=T;p++) {
        if(tag[p]) {if(tag[p]==x&&tag[p]==y) return 0;}
        else if(id[p][x]!=0&&id[p][y]!=0) checkmin(ans,f[p][id[p][x]][id[p][y]]);
    }
    int xx=-1,yy=-1;
    for(int p=1;p<=T;p++) {
        if(tag[p]) {
            if(xx!=-1&&tag[p]==y) checkmin(ans,L[p]-xx);
            if(yy!=-1&&tag[p]==x) checkmin(ans,L[p]-yy); 
            if(tag[p]==x) xx=R[p];
            if(tag[p]==y) yy=R[p];
            continue;
        }
        if(xx!=-1&&id[p][y]!=0) checkmin(ans,g[p][id[p][y]][0]-xx);
        if(yy!=-1&&id[p][x]!=0) checkmin(ans,g[p][id[p][x]][0]-yy);
        if(id[p][x]!=0) xx=g[p][id[p][x]][1];
        if(id[p][y]!=0) yy=g[p][id[p][y]][1];
    }
    return ans;
}
void modify(int x,int y,int vl) {
    int l=(x-1)/(B-1)+1,r=(y-1)/(B-1)+1;
    // printf("modify [%d,%d] to %d  l %d  r %d",x,y,vl,l,r);
    if(l==r) {
        // cout<<"!"<<endl;
        if(tag[l]) {
            for(int i=L[l];i<=R[l];i++) a[i]=tag[l];
            tag[l]=0;
        }
        for(int i=x;i<=y;i++) a[i]=vl;
        build(l);
        return ;
    }
    if(L[l]!=x) l++;if(R[r]!=y) r--;
    for(int p=l;p<=r;p++) tag[p]=vl;
    if(L[l]!=x) {
        if(tag[l-1]) {
            for(int i=L[l-1];i<=R[l-1];i++) a[i]=tag[l-1];
            tag[l-1]=0;
        }
        for(int i=x;i<L[l];i++) a[i]=vl;
        build(l-1);
    }
    if(R[r]!=y) {
        if(tag[r+1]) {
            for(int i=L[r+1];i<=R[r+1];i++) a[i]=tag[r+1];
            tag[r+1]=0;
        }
        for(int i=R[r]+1;i<=y;i++) a[i]=vl;
        build(r+1);
    }
}
int main() {
    red(n),red(m),red(ff);
    for(int i=1;i<=n;i++) red(a[i]);
    for(T=1;T<=n/(B-1);T++) {
        L[T]=(T-1)*(B-1)+1;
        R[T]=T*(B-1);
    }T--;
    if(R[T]<n) L[T+1]=R[T]+1,R[++T]=n;
    for(int p=1;p<=T;p++) build(p);
    int lstans=0;
    while(m--) {
        int op,x,y,z;
        red(op),red(x),red(y);
        if(op==1) {
            red(z);
            if(ff) x^=lstans,y^=lstans,z^=lstans;
            modify(x,y,z);
        }
        else {
            if(ff) x^=lstans,y^=lstans;
            lstans=query(x,y);
            printf("%d\n",lstans);
            if(lstans==-1) lstans=0;
        }
    }
}
```