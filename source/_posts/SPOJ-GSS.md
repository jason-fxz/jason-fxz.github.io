---
title: 'SPOJ GSS - Can you answer these queries '
date: 2020-11-20 21:40:05
tags: [线段树,平衡树,重链剖分]
categories:
- 题解
---


一些~~简单~~数据结构题，线段树或平衡树，一共八道题，其中 $\mathrm{I,II,IV}$ 过水懒得写。 


<!--more-->

## $\mathrm{GSS-III}$

长度为 $n$ 的序列 $A$，$q$ 次操作
1. 操作 $0\;x\;y$ 把 $A_x$ 修改为 $y$
2. 操作 $1\;l\;r$ 询问区间 $[l,r]$ 的最大子段和(非空)

$1\le n,q\le 5\times 10^4,\;\left|y\right|,\left|A_i\right|\le 10000$

***330ms 1.46GB***

线段树维护 **区间和、最大子段和、最大前缀和、最大后缀和** ，结构体+重载运算符是个好东西，核心代码：

```cpp
struct rec{
    int sum,mx,st,ed; //和，最大子段和、最大前缀和、最大后缀和
    rec() {}
    rec(int _a) {sum=st=ed=mx=_a;}
};
rec operator+(const rec a,const rec b) {
    rec t;
    t.mx=_max(a.ed+b.st,_max(a.mx,b.mx));
    t.st=_max(a.st,a.sum+b.st);
    t.ed=_max(b.ed,a.ed+b.sum);
    t.sum=a.sum+b.sum;
    return t;
}
```

------

{% spoiler "完整代码" %}

```cpp
#include <bits/stdc++.h>
#define ls (x<<1)
#define rs (x<<1|1)
#define mid ((l+r)>>1)
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
const int N = 50010;
struct rec{
    int sum,st,ed,mx;
    rec() {}
    rec(int _a) {sum=st=ed=mx=_a;}
};
rec val[N<<2];
int n,q,a[N];
rec operator+(const rec a,const rec b) {
    rec t;
    t.mx=_max(a.ed+b.st,_max(a.mx,b.mx));
    t.st=_max(a.st,a.sum+b.st);
    t.ed=_max(b.ed,a.ed+b.sum);
    t.sum=a.sum+b.sum;
    return t;
}
void pushup(int x) {val[x]=val[ls]+val[rs];}
void build(int l=1,int r=n,int x=1) {
    if(l==r) return val[x]=rec(a[l]),void();
    build(l,mid,ls);build(mid+1,r,rs);
    pushup(x);
}
rec query(int L,int R,int l=1,int r=n,int x=1) {
    if(L<=l&&r<=R) return val[x];
    if(L<=mid&&R>mid) return query(L,R,l,mid,ls)+query(L,R,mid+1,r,rs); 
    if(L<=mid) return query(L,R,l,mid,ls);
    if(R>mid) return query(L,R,mid+1,r,rs);
}
void modify(int p,int vl,int l=1,int r=n,int x=1) {
    if(l==r) return val[x]=rec(vl),void();
    if(p<=mid) modify(p,vl,l,mid,ls);
    else modify(p,vl,mid+1,r,rs);
    pushup(x);
}
int main() {
    red(n); 
    for(int i=1;i<=n;i++) red(a[i]);
    build(); red(q);
    for(int ts=1;ts<=q;ts++) {
        int op,x,y;
        red(op);red(x);red(y);
        if(op==0) modify(x,y);
        else printf("%d\n",query(x,y).mx);
    }
    return 0;
}
```

{% endspoiler %}

------

## $\mathrm{GSS-V}$

长度为 $n$ 的序列 $A$，$q$ 次询问：每次询问左端点在 $[x_1, y_1]$ 之间，且右端点在 $[x_2, y_2]$ 之间的最大子段和，数据保证 $x_1\leq x_2,y_1\leq y_2$ ，但是**不保证端点所在的区间不重合**

$1\le n,q\le 10000,\;\left|A_i\right|\le 10000$

***132ms 1.46GB***

> 这道题其实还可以加修改。

和上一道题的区别无非是左右端点不确定了，考虑延续上一题的做法，维护区间和、最大子段和、最大前缀和、最大后缀和，对于询问区间重不重合我们分类讨论。
1. $x_1\le y_1\lt x_2 \le y_2$ ： 那就是 $[x_1,y_1]$ 的最大后缀和 + $[y_1+1,x_2-1]$ 的区间和 + $[x_2,y_2]$ 的最大前缀和 
2. $x_1\le x_2 \le y_1 \le y_2$ ：再分类讨论一下即可， $[x_1,y_1]$ 的最大后缀 + $[y_1+1,y_2]$ 的最大前缀 或 $[x_1,x_2-1]$ 的最大后缀 + $[x_2,y_2]$ 的最大前缀 或 $[x_2,y_1]$ 的最大最大子段和。三种情况取 $\max$ 即可

线段树和上一题一样，所以说待修也是可以的。


------



{% spoiler "完整代码" %}

```cpp
#include <bits/stdc++.h>
#define ls (x<<1)
#define rs (x<<1|1)
#define mid ((l+r)>>1)
#define _max(x,y) ((x>y)?(x):(y))
#define _min(x,y) ((x<y)?(x):(y))
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 10010;
const int inf = 0x3f3f3f3f;
struct rec{
    int mx,sum,st,ed;
    rec(int _val) {mx=sum=st=ed=_val;}
    rec(){mx=sum=st=ed=0;}
};
rec val[N<<2];
int a[N],n,q,T;
rec operator+(const rec a,const rec b) {
    rec t;
    t.sum=a.sum+b.sum;
    t.mx=_max(a.ed+b.st,_max(a.mx,b.mx));
    t.st=_max(a.st,a.sum+b.st);
    t.ed=_max(b.ed,a.ed+b.sum);
    return t;
}
void pushup(int x) {val[x]=val[ls]+val[rs];}
void build(int l=1,int r=n,int x=1) {
    if(l==r) return val[x]=rec(a[l]),void();
    build(l,mid,ls);build(mid+1,r,rs);
    pushup(x);
}
rec query(int L,int R,int l=1,int r=n,int x=1) {
    if(L>R) return rec();
    if(L<=l&&r<=R) return val[x];
    if(L<=mid&&R>mid) return query(L,R,l,mid,ls)+query(L,R,mid+1,r,rs);
    return (L<=mid)?query(L,R,l,mid,ls):query(L,R,mid+1,r,rs);
}
int main() {
    red(T);
    while(T--) {
        red(n);
        for(int i=1;i<=n;i++) red(a[i]);
        build();
        red(q);
        for(int ts=1;ts<=q;ts++) {
            int a,b,c,d;red(a);red(b);red(c);red(d);
            if(b<c) {
                rec x=query(a,b),y=query(b+1,c-1),z=query(c,d);
                printf("%d\n",x.ed+y.sum+z.st);
            }else {
                int ans=-inf;
                rec x=query(a,c-1),y=query(c,b),z=query(b+1,d);                
                rec ll=x+y,rr=y+z;
                ans=_max(x.ed+y.sum+z.st,y.mx);
                ans=_max(ans,_max(ll.ed+z.st,x.ed+rr.st));
                printf("%d\n",ans);
            }
        }
    }
}
```

{% endspoiler %}

------


## $\mathrm{GSS-VI}$

长度为 $n$ 的序列 $A$，$q$ 次操作，操作类型如下：

1. `I p x` 在 $p$ 处插入插入一个元素 $x$ 
2. `D p` 删除 $p$ 处的一个元素
3. `R p x` 修改 $p$ 处元素的值为 $x$
4. `Q l r` 查询一个区间 $[l,r]$ 的最大子段和 

注: 在 $p$ 处插入 意思是在 $p-1$ 和 $p$ 之间

$1\le n \le 10^5,1\le q \le 10^5,0\le |A_i|,|x| \le 10^4$

***447ms 1.46GB***

又是最大子段和。。无非是把它搬到了平衡树上，可以写 fhqTreap 或者 Splay。我菜，只会 fhqTreap ，`merge,split` 解决一切问题。注意细节，每个节点还要维护当前点的值，在 `updata` 时把左右孩子和自己的最大子段和信息合并。

还是那句话，重载运算符是个好东西。

------


{% spoiler "完整代码" %}

```cpp
#include <bits/stdc++.h>
#define ls(p) (t[p].lc)
#define rs(p) (t[p].rc)
#define _max(x,y) ((x>y)?(x):(y))
#define _min(x,y) ((x<y)?(x):(y))
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 100010;
const int inf = 0x3f3f3f3f;
int n,a[N],q;
mt19937 rd(random_device{}());
struct rec{
    int mx,sum,st,ed;bool ept;
    rec(int _val) {mx=sum=st=ed=_val;ept=0;}
    rec(){mx=sum=st=ed=0;ept=1;}
    void operator!() {
        printf("%d %d %d %d\n",sum,mx,st,ed);
    }
};
rec operator+(const rec a,const rec b) {
    if(a.ept) return b;if(b.ept) return a; 
    rec t;t.ept=0;t.sum=a.sum+b.sum;t.mx=_max(a.ed+b.st,_max(a.mx,b.mx));
    t.st=_max(a.st,a.sum+b.st);t.ed=_max(b.ed,a.ed+b.sum);return t;
}
struct node {
    rec val;int rnd,lc,rc,siz,vl;
    node() {}
    node(int _val) {val=rec(_val);rnd=(int)rd();vl=_val;lc=rc=0;siz=1;}
}t[N<<2];
int tot,root;
int newnode(int val) {
    t[++tot]=val;
    return tot;
}
void updata(int p) {
    t[p].siz=t[ls(p)].siz+t[rs(p)].siz+1; 
    t[p].val=t[ls(p)].val+rec(t[p].vl)+t[rs(p)].val;
}
int merge(int p,int q) {
    if(!p||!q) return p+q;
    if(t[p].rnd<t[q].rnd) {
        rs(p)=merge(rs(p),q);
        updata(p);return p;
    }else {
        ls(q)=merge(p,ls(q));
        updata(q);return q;
    }
}
void split(int p,int k,int &x,int &y) {
    if(!p) x=0,y=0;
    else {
        if(t[ls(p)].siz>=k) y=p,split(ls(p),k,x,ls(p));
        else x=p,split(rs(p),k-t[ls(p)].siz-1,rs(p),y);
        updata(p);
    }
}
int main() {
    red(n);
    for(int i=1;i<=n;i++) {
        red(a[i]);
        root=merge(root,newnode(a[i]));
    }
    red(q);
    for(int ts=1;ts<=q;ts++) {
        char op[3];int a,b,x,y,z;
        scanf("%s",op);
        switch(op[0]) {
            case 'I':
                red(a);red(b);
                split(root,a-1,x,y);
                root=merge(x,merge(newnode(b),y));
                break;
            case 'D':
                red(a);
                split(root,a-1,x,y);
                split(y,1,y,z);
                root=merge(x,z);
                break;
            case 'R':
                red(a);red(b);
                split(root,a-1,x,y);
                split(y,1,y,z);
                t[y].val=rec(b);t[y].vl=b;
                root=merge(x,merge(y,z));
                break;
            case 'Q':
                red(a);red(b);
                split(root,a-1,x,y);
                split(y,b-a+1,y,z);
                printf("%d\n",t[y].val.mx);
                root=merge(x,merge(y,z));
                break;
        }
    }
}
```

{% endspoiler %}

------

## $\mathrm{GSS-VII}$

给定一棵树，有 $N(N \le 100,000)$ 个节点，每一个节点都有一个权值 $x_i (|x_i| \le 10000)$  ，你需要执行 $Q (Q \le 100,000)$ 次操作：

1. $1\;a\;b$ ：查询 $(a,b)$ 这条链上的最大子段和，可以为空（即输出 $0$)
2. $2\; a\;b\;c$ ：将 $(a,b)$ 这条链上的所有点权变为 $c (|c| \le 10000)$

**570ms 1.46GB**

把最大子段和搬到树上了。。。加一个重链剖分完事呗，区间修改也挺简单，多打个标记呗。

细节在询问，从两条链跳上来，最后合并的时候要把左边的 “reverse” 以下(就是交换最大前缀和,最大后缀和) ，因为两条链上来都是深度低的为左边。

------

{% spoiler "完整代码" %}

```cpp
#include <bits/stdc++.h>
#define ls (x<<1)
#define rs (x<<1|1)
#define mid ((l+r)>>1)
#define _max(x,y) ((x>y)?(x):(y))
#define _min(x,y) ((x<y)?(x):(y))
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 100010;
const int inf = 0x3f3f3f3f;
struct rec{
    int mx,sum,st,ed;
    rec(int _val) {sum=_val;mx=st=ed=_max(0,sum);}
    rec(int _val,int len) {sum=_val*len;st=ed=mx=_max(sum,0);}
    rec(){mx=sum=st=ed=0;}
};
rec val[N<<2];int tag[N<<2],len[N<<2];
int head[N],pnt[N<<1],nxt[N<<1],E;
int a[N],n,q,son[N],siz[N],top[N],tim,id[N],rk[N],fa[N],dep[N];
rec operator+(const rec a,const rec b) {
    rec t;t.sum=a.sum+b.sum;t.mx=_max(a.ed+b.st,_max(a.mx,b.mx));
    t.st=_max(a.st,a.sum+b.st);t.ed=_max(b.ed,a.ed+b.sum);return t;
}
void pushup(int x) {val[x]=val[ls]+val[rs];}
void pushdown(int x) {
    if(tag[x]==inf) return ;
    tag[ls]=tag[rs]=tag[x];
    val[ls]=rec(tag[x],len[ls]);
    val[rs]=rec(tag[x],len[rs]);
    tag[x]=inf;
}
void build(int l=1,int r=n,int x=1) {
    tag[x]=inf;len[x]=r-l+1;
    if(l==r) return val[x]=rec(a[rk[l]]),void();
    build(l,mid,ls);build(mid+1,r,rs);
    pushup(x);
}
rec query(int L,int R,int l=1,int r=n,int x=1) {
    if(L>R) return rec();
    if(L<=l&&r<=R) return val[x];
    pushdown(x);
    if(L<=mid&&R>mid) return query(L,R,l,mid,ls)+query(L,R,mid+1,r,rs);
    return (L<=mid)?query(L,R,l,mid,ls):query(L,R,mid+1,r,rs);
}
void modify(int L,int R,int vl,int l=1,int r=n,int x=1) {
    if(L>R) return;
    if(L<=l&&r<=R) return val[x]=rec(vl,len[x]),tag[x]=vl,void();
    pushdown(x);
    if(L<=mid) modify(L,R,vl,l,mid,ls);
    if(R>mid) modify(L,R,vl,mid+1,r,rs);
    pushup(x);
}
void print(int l=1,int r=n,int x=1) {
    printf("print [%d,%d] (%d) %d %d %d %d\n",l,r,x,val[x].sum,val[x].mx,val[x].st,val[x].ed);
    if(l==r) return ;
    pushdown(x);
    print(l,mid,ls);print(mid+1,r,rs);
}
void dfs1(int u) {
    siz[u]=1;top[u]=u;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u]) continue;
        fa[v]=u;dep[v]=dep[u]+1;dfs1(v);siz[u]+=siz[v];
        if(siz[son[u]]<siz[v]) son[u]=v;
    }
}
void dfs2(int u) {
    id[u]=++tim;rk[tim]=u;
    if(son[u]) {
        top[son[u]]=top[u];
        dfs2(son[u]);
    }
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==fa[u]||v==son[u]) continue;
        dfs2(v);
    }
}
void adde(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void Modify(int u,int v,int vl) {
    int fu=top[u],fv=top[v];
    while(fu!=fv) {
        if(dep[fu]>dep[fv]) swap(u,v),swap(fu,fv); 
        modify(id[fv],id[v],vl);
        v=fa[fv];fv=top[v];
    }
    if(dep[u]>dep[v]) swap(u,v);
    modify(id[u],id[v],vl);
}
rec Query(int u,int v) {
    rec ru,rv;
    int fu=top[u],fv=top[v];
    while(fu!=fv) {
        if(dep[fu]>dep[fv]) {
            ru=query(id[fu],id[u])+ru;
            u=fa[fu];fu=top[u];
        }else {
            rv=query(id[fv],id[v])+rv;
            v=fa[fv];fv=top[v];
        }
    }
    if(dep[u]>dep[v]) ru=query(id[v],id[u])+ru;
    else rv=query(id[u],id[v])+rv;
    swap(ru.st,ru.ed);
    return ru+rv;
}
int main() {
    red(n);
    for(int i=1;i<=n;i++) red(a[i]);
    for(int i=1;i<n;i++) {
        int u,v;red(u);red(v);
        adde(u,v);adde(v,u);
    }
    dfs1(1);dfs2(1);
    build();
    red(q);
    while(q--) {
        int op,a,b,c;
        red(op);red(a);red(b);
        if(op==1) {
            rec t=Query(a,b);
            printf("%d\n",t.mx);
        }else {
            red(c);
            Modify(a,b,c);
        }
    }
}
```

{% endspoiler %}

------


## $\mathrm{GSS-VIII}$


给你长度为 $N$ 的序列 $A (0 \le A_i \lt 2^{32})$ 下标从零开始

你需要支持 $Q$ 次操作

1. `I pos val` 插入一个数字在第 $pos$ 个位置之前，$0 \le val \lt 2^{32}$  如果 $pos=len$，那么你需要将这个数字放到序列末尾
2. `D pos` 删除第 $pos$ 个元素
3. `R pos val` 将第 $pos$ 个元素变为 $val(0 \le val \lt 2^{32})$
4. `Q l r k` 询问 $\left(\sum\limits_{i=l}^{r} A[i] \times (i - l + 1) ^ k\right) \bmod 2^{32}$ ，保证 $0 \le k \le 10$

$1\le N,Q \le 10^5$

***1.50s 1.46GB***

开始毒瘤了，首先这个删除插入肯定要平衡树，那查询的东西怎么维护呢？

发现 $k\le 10$ 那就暴力维护 这个区间 “k次阶梯和” （~~自己瞎取的名儿~~），形式化的定义，（ $a$ 为一个序列，$siza$ 是 $a$ 的长度 ）：
$$
g(a,k)=\sum_{i=1}^{siza} i^k \cdot a_i
$$

那来考虑怎么维护，**怎么合并信息** ，我们要合并 $a$ 序列和 $b$ 序列，且让 $a$ 在 $b$ 前，定义 $g(c,k)=g(a,k)\star g(b,k)$ :
$$
g(c,k)=\sum_{i=1}^{siza} i^k\cdot a_i+\sum_{i=1}^{sizb} (i+siza)^k\cdot b_i\\
$$

然后就是大胆推式子，其实也不难：

$$
\begin{aligned}
&\sum_{i=1}^{sizb} (i+siza)^k\cdot b_i\\
=&\sum_{i=1}^{sizb} \left(\sum_{t=0}^k \binom{k}{t}\cdot i^tsiza^{k-t}\right)b_i\\
=&\sum_{i=1}^{sizb} \sum_{t=0}^k \left(siza^{k-t}\binom{k}{t}\right) \cdot i^t  b_i \\
=&\sum_{t=0}^k \left(siza^{k-t}\binom{k}{t}\right)\sum_{i=1}^{sizb} i^t\cdot b_i \\
=&\sum_{t=0}^k \left(siza^{k-t}\binom{k}{t}\right)g(b,t)
\end{aligned}
$$

然后便轻松得到了：

$$
g(c,k)=g(a,k)+\sum_{t=0}^k \left(siza^{k-t}\binom{k}{t}\right)g(b,t)\\
$$

写成结构体+重载运算符，方便调用：

```cpp
struct rec{
    ui g[11],siz;bool ept;
    rec() {memset(g,0,sizeof(g));siz=0;ept=1;}
    rec(ui val) {for(re int i=0;i<=10;++i) g[i]=val;siz=1;ept=0;}
};
rec operator*(const rec a,const rec b) {
    if(a.ept) return b;if(b.ept) return a;
    rec r;r.siz=a.siz+b.siz;r.ept=0;
    for(int k=0;k<=10;k++) {
        r.g[k]=a.g[k];
        ui pw=1;
        for(int t=0;t<=k;t++,pw*=a.siz) {
            r.g[k]+=pw*C[k][k-t]*b.g[k-t]; // C[n][m] 为组合数
        }
    }
    return r;
}
```


------

{% spoiler "完整代码" %}

```cpp
#include <bits/stdc++.h>
#define ls(p) (t[p].lc)
#define rs(p) (t[p].rc)
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
typedef unsigned int ui;
mt19937 rd(random_device{}());
ui n,q,a[N];
struct rec{
    ui g[11],siz;bool ept;
    rec() {memset(g,0,sizeof(g));siz=0;ept=1;}
    rec(ui val) {for(re int i=0;i<=10;++i) g[i]=val;siz=1;ept=0;}
};
ui C[20][20];
rec operator*(const rec a,const rec b) {
    if(a.ept) return b;if(b.ept) return a;
    rec r;r.siz=a.siz+b.siz;r.ept=0;
    for(int k=0;k<=10;k++) {
        r.g[k]=a.g[k];
        ui pw=1;
        for(int t=0;t<=k;t++,pw*=a.siz) {
            r.g[k]+=pw*C[k][k-t]*b.g[k-t];
        }
    }
    return r;
}
void init() {
    C[0][0]=1;
    for(int i=1;i<=15;i++) {
        C[i][0]=1;
        for(int j=1;j<=15;j++) C[i][j]=C[i-1][j]+C[i-1][j-1];
    }
}
struct node{
    rec dat;ui rnd,lc,rc,val;
    node() {}
    node(ui vl) {rnd=rd();lc=rc=0;val=vl;dat=rec(vl);}
}t[N<<1];ui tot,root;
ui newnode(ui _vl) {
    t[++tot]=node(_vl);
    return tot;
}
void updata(int p) {
    t[p].dat=t[ls(p)].dat*rec(t[p].val)*t[rs(p)].dat;
}
ui merge(ui p,ui q) {
    if(!p||!q) return p+q;
    if(t[p].rnd<t[q].rnd) {
        rs(p)=merge(rs(p),q);
        updata(p);return p;
    }else {
        ls(q)=merge(p,ls(q));
        updata(q);return q;
    }
}
void split(ui p,ui k,ui &x,ui &y) {
    if(!p) x=0,y=0;
    else {
        if(t[ls(p)].dat.siz>=k) y=p,split(ls(p),k,x,ls(p));
        else x=p,split(rs(p),k-t[ls(p)].dat.siz-1u,rs(p),y);
        updata(p);
    }
}
void pt(ui p) {
    if(p==0) return ;
    pt(ls(p));
    printf("%u ",t[p].val);
    pt(rs(p));
}
ui build(ui l=1u,ui r=n) {
    if(l>r) return 0;
    ui mid=(l+r)>>1;
    return merge(build(l,mid-1u),merge(newnode(a[mid]),build(mid+1u,r)));
}
int main() {
    init();
    red(n);
    for(int i=1;i<=n;i++) {
        red(a[i]);
    }
    root=build();
    red(q);
    for(int ts=1;ts<=q;ts++) {
        char op[3];ui a,b,c,x,y,z;
        scanf("%s",op);
        switch(op[0]) {
            case 'I':
                red(a);red(b);a++;
                split(root,a-1u,x,y);
                root=merge(x,merge(newnode(b),y));
                break;
            case 'D':
                red(a);a++;
                split(root,a-1u,x,y);
                split(y,1u,y,z);
                root=merge(x,z);
                break;
            case 'R':
                red(a);red(b);a++;
                split(root,a-1u,x,y);
                split(y,1u,y,z);
                t[y].dat=rec(b);t[y].val=b;
                root=merge(x,merge(y,z));
                break;
            case 'Q':
                red(a);red(b);red(c);a++,b++;
                split(root,a-1u,x,y);
                split(y,b-a+1u,y,z);
                printf("%u\n",t[y].dat.g[c]);
                root=merge(x,merge(y,z));
                break;
        }
    }
    return 0;
}
```

{% endspoiler %}

------

## 后记

其实仔细思考，这些数据结构体都不难，~~主要是难调~~，封装，重载运算符是个好东西，能是代码简洁，清晰易懂，~~方便复制~~，不容易出错
