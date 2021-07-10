---
title: 'Innopolis Open 2020-2021 qualification contest 1'
date: 2020-11-22 22:22:27
tags: 
categories:
- 题解
---



> Innopolis Open 2020-2021, ualification, contest 1 ,Russia, Innopolis, November, 22, 2020

**[en-statements](./prob.pdf)**


![DGmNu9.png](./DGmNu9.png)

## ~~前言~~

真就暴毙，总共 $300min$ ，迟到 $25min$，前 $90min$ 暴切 ABCD，rank $36$ 吃了个饭+晚读 耗时 $60min$ ，然后成功在 $200min$ 后的取得 rank $133$ 的“好成绩” ，调了 两三个小时的 E，人都傻了

<!--more-->

## problems

### A - The Battle of Giants

过水已隐藏

### B - Tetris Remastered

原题，就是 2018 提高组 T1 铺设道路


### C - Optimal Truck

定义一个快递有体积和价值两种元素，现在皮特想要帮 $n$ 个人送快递，第 $i$ 个人有 $m_i$ 个快递，描述为 $(w_{i,j},c_{i,j})$ 即体积和价值，可惜皮特只能帮每个人运送至多一个快递。为了运快递，皮特买了辆卡车，要求卡车的容积要大于等于要运的快递中体积的最大值，同时皮特希望尽量盈利，所以他会给你 $q$ 个问题，每个问题给出 $x_i$ ，要求求出在盈利至少 $x_i$ 的情况下，卡车最少的容积。

$$1\le n,q\le 10^5, 1\le \sum m_i \le 5\cdot 10^5,1\le x_i,c_i,w_i\le 10^9$$


#### 题解

水题，将所有快递按体积排序，要记录是哪个人的，把询问也按照体积排序，双指针扫一遍即可。

#### CODE

```cpp
#include <bits/stdc++.h>
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
using namespace std;
typedef long long ll;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename T> bool umax(T &x,T y) {return (x<y)?(x=y,true):(false);}
template<typename T> bool umin(T &x,T y) {return (x>y)?(x=y,true):(false);}
const int N = 100010;
const int M = 500010;
typedef long long ll;
int n,m[M];
int nc[N],a,b,Q;
struct rec{
    int w,c,id;
    bool operator<(const rec a) const{ 
        return w<a.w;
    }
}p[M];int tot;
struct que {
    int x,id;
    bool operator<(const que a)const {
        return x<a.x;
    } 
}qr[N];
ll sum,noww,ans[N];
int main() {
    red(n);
    for(int i=1;i<=n;i++) {
        red(m[i]);
        for(int j=1;j<=m[i];j++) {
            red(a);red(b);
            tot++;p[tot].w=a;p[tot].c=b;
            p[tot].id=i;
        } 
    }
    red(Q);
    for(int i=1;i<=Q;i++) {
        red(qr[i].x);qr[i].id=i;
    }
    sort(p+1,p+tot+1);
    sort(qr+1,qr+Q+1);
    for(int i=1,j=1;i<=Q;i++) {
        while(j<=tot&&sum<qr[i].x) {
            sum-=nc[p[j].id];
            noww=p[j].w;
            umax(nc[p[j].id],p[j].c);
            sum+=nc[p[j].id];
            j++;
        }
        if(sum>=qr[i].x) {
            ans[qr[i].id]=noww;
        }else {
            ans[qr[i].id]=-1;
        }
    }
    for(int i=1;i<=Q;i++) {
        printf("%lld ",ans[i]);
    }
    printf("\n");
    return 0;
}
```


### D - Bookshelf Sorting

$n$ 本书，编号 $1$ 到 $n$ ，现在给你序列 $a_i$ ，是打乱的书的编号（即 $n$ 的排列），一次操作是抽出一本书将其放到序列开头或结尾，问至少操作几次使书的编号升序。且有 $q$ 次修改，每次修改交换 $a_{x_i},a_{y_i}\;,(1\le x_i\lt y_i \le n)$ ，你需要在每次修改后输出答案。（修改是永久的）

$$2 \le n \le 2 \cdot 10^5, 0 \le q \le 2 \cdot 10^5$$


#### 题解

显然题意可以转化为求序列 $a_i$ 中最长 $+1$ 子序列，答案就是 $n$ 减去它。

考虑DP，先记 $pos_{a_i}=i$ ，表示 $a_i$ 出现的位置，那就是求 $pos$ 的最长上升子段（要连续的） ，$q$ 次 $O(n)$ DP，有手就行。

考虑优化，$pos$ 的最长上升子段可以用线段树维护，只要维护区间最长上升子段，前缀最长上升子段，后缀最长上升子段，和区间左右端点的值即可，每次修改只是单点修改，复杂度 $O(n+q\log n)$

#### CODE

`namespace sub1` 是暴力DP

```cpp
#include <bits/stdc++.h>
#define _max(x,y) ((x>y)?x:y)
#define _min(x,y) ((x<y)?x:y)
using namespace std;
typedef long long ll;
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
int a[N],n,q,x[N],y[N];
int re[N],f[N],ans;
namespace sub1 {
    void work() {
        for(int i=1;i<=n;i++) re[a[i]]=i;
        ans=0;
        for(int i=1;i<=n;i++) {
            if(re[i-1]<re[i]) f[i]=f[i-1]+1;
            else f[i]=1;
            umax(ans,f[i]);
        }
        ans=n-ans;
        printf("%d\n",ans);
    }
    void solve() {
        work();
        for(int ts=1;ts<=q;ts++) {
            swap(a[x[ts]],a[y[ts]]);
            work();
        }
        
    }
}
namespace ok {
    #define ls (x<<1)
    #define rs (x<<1|1)
    #define mid ((l+r)>>1)
    int Lt[N<<2],Rt[N<<2],val[N<<2],st[N<<2],ed[N<<2],len[N<<2];
    void pushup(int x) {
        val[x]=_max(val[ls],val[rs]);
        st[x]=st[ls];ed[x]=ed[rs];
        if(Rt[ls]<Lt[rs]) {
            umax(val[x],ed[ls]+st[rs]);
            if(val[ls]==len[ls]) umax(st[x],len[ls]+st[rs]);
            if(val[rs]==len[rs]) umax(ed[x],len[rs]+ed[ls]);
        }
        Lt[x]=Lt[ls];Rt[x]=Rt[rs];
    }
    void updata(int x,int vl) {
        Lt[x]=Rt[x]=vl;
        val[x]=st[x]=ed[x]=1;
    }
    void build(int l=1,int r=n,int x=1) {
        len[x]=r-l+1;
        if(l==r) return updata(x,re[l]),void();
        build(l,mid,ls);build(mid+1,r,rs);
        pushup(x);
    }
    void modify(int p,int vl,int l=1,int r=n,int x=1) {
        if(l==r) return updata(x,vl),void();
        if(p<=mid) modify(p,vl,l,mid,ls);
        else modify(p,vl,mid+1,r,rs);
        pushup(x);
    }
    void solve() {
        for(int i=1;i<=n;i++) re[a[i]]=i;
        build();
        printf("%d\n",n-val[1]);
        for(int i=1;i<=q;i++) {
            re[a[x[i]]]=y[i];
            re[a[y[i]]]=x[i];
            modify(a[x[i]],y[i]);
            modify(a[y[i]],x[i]);
            swap(a[x[i]],a[y[i]]);
            printf("%d\n",n-val[1]);
        }
    }
}
int main() {
    red(n);red(q);
    for(int i=1;i<=n;i++) red(a[i]);
    for(int i=1;i<=q;i++) red(x[i]),red(y[i]);
    // sub1::solve();
    ok::solve();
    return 0;
}
```


### E -  Nice Shape

平面上 $n$ 个点（都是整点，其实也无所谓），一次操作可以改变一个点的 $x$ 坐标或 $y$ 坐标。但点不能重合。你需要搞出 $4$ 个点围成一个矩形，具体的四个点的坐标可以描述为 $(x_1,y_1),(x_2,y_1),(x_1,y_2),(x_2,y_2),\;x_1\ne x_2,y_1\ne y_2$ 问最少移动次数。$t$ 组测试数据

$$1\le t \le 25000,4\le n\le 10^5,\sum n \le 10^5$$

#### 题解

可以发现最多四步，确定一个对角，剩下两点分别两步即可，。。。。。。。。
大型分类讨论题

然后不知道为毛写炸了，


先咕咕