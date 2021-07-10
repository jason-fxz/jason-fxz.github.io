---
title: XJ-集合
date: 2020-12-3 21:57:52
tags: [并查集,XJ]
categories: 题解
hidden: true
---

<!--more-->

## 题意

![1](./1.png)
![2](./2.png)
![3](./3.png)
![4](./4.png)

## 题解

有两种操作，都涉及到合并，考虑用 **并查集+打标记** 来维护。因为路径压缩会改变树的形态，不方便标记的处理，我们只需要按秩合并，这样能保证树高 $\log n$ 。维护两个并查集 $A,B$ 分别处理上面的加，赋值操作。

考虑一次询问，应该先找到 $x$ 最近一次的赋值操作，再算这之后到现在加了多少。

考虑在 $B$ 中维护赋值操作，对于操作 2，直接按秩合并两个集合。对于操作 4 ，找到集合的根结点，在根结点打上 tag 表示这个集合的一次赋值，同时要记录操作时间（因为时间靠后的可以覆盖掉之前的）。要查询 $x$ 最近一次赋值，只要不断跳父亲取时间最靠后的即可。但这里有个问题，假设 $u$ 打上了 tag ，之后 $v$ 又合并到 $u$ 上，$u$ 上这个之前打的 tag 会影响到 $v$ 。解决方法很简单，我们再记录连上 $v\to u$ 这条边的时间 $t_v$ ，**时间小于 $t_v$ 的 tag 都忽略。**

考虑维护 $A$ ，操作 1 按秩合并，操作 3 同打 tag ，考虑加法可以前缀和，那每个点开个 vector 维护 tag ，新加的 tag 只要 push_back 就行了。对于询问，同样跳父亲，统计 $tim$ 到现在加的总和，二分即可 （$tim$ 是 $B$ 上查到的最近一次赋值的时间）。同样要注意小于连边时间的 tag 需忽略。

总复杂度 $O(n\log n)$ 。

## CODE

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010;
typedef long long ll;
template <typename T>
inline void red(T &x) {
    x=0;bool f=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') f^=1; ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(f) x=-x;
}
int n,m;
struct DAT{
    int ti; ll val;
    DAT(int _ti,ll _val):ti(_ti),val(_val){}
    DAT(){}
    bool operator<(const DAT a)const {return ti<a.ti;} 
    DAT operator+(DAT a) {return (ti>a.ti)?(*this):a;    }
};
struct DSU1 {
    int fa[N],dep[N],ti[N]; // ti 表示这条边连上的时间 
    vector<DAT> tag[N]; // 某点打的加法tag，这里的val是前缀和
    int find(int u) {return fa[u]==u?u:find(fa[u]);}
    void ut(int u,int v,int t) {
        int fu=find(u),fv=find(v);
        if(dep[fu]>dep[fv]) swap(u,v),swap(fu,fv);
        dep[fv]=max(dep[fv],dep[fu]+1); fa[fu]=fv; ti[fu]=t;
    }
    void addtag(int u,int val,int t) {
        int fu=find(u);
        ll lst=tag[fu].at(tag[fu].size()-1).val;
        tag[fu].push_back(DAT(t,lst+val));
    }
    ll ask(int u,int tim) { // 询问, tim 之后的对 u 的加的贡献
        // (tim,now] 
        ll ret=0; int lim=tim; 
        while(1) {
            int ps=upper_bound(tag[u].begin(),tag[u].end(),DAT(lim,0))-tag[u].begin();
            if(ps==0) ps++;
            ret+=tag[u].at(tag[u].size()-1).val-tag[u].at(ps-1).val;
            if(fa[u]==u) break;
            else lim=max(tim,ti[u]),u=fa[u]; // 小于连边时间的tag也需要忽略
        }
        return ret;
    }
    void init() {
        for(int i=1;i<=n;++i) fa[i]=i,dep[i]=1,tag[i].push_back(DAT(0,0)); //为了方便，插一个时间为0的空操作
    }
}A;
struct DSU2{ // 维护赋值操作
    int fa[N],dep[N],ti[N]; // ti 表示这条边连上的时间 
    DAT tag[N];
    int find(int u) {return fa[u]==u?u:find(fa[u]);}
    void addtag(int u,int val,int t) { // u所在集合推平
        int fu=find(u);
        tag[fu]=DAT(t,val);
    }
    DAT ask(int u) { // 询问u最后的推平时间
        DAT r=tag[u];
        while(u!=fa[u]) {
            if(ti[u]<tag[fa[u]].ti) { // 如果连边的时间在父亲tag之后，父亲的tag无影响
                r=r+tag[fa[u]];
            }
            u=fa[u];
        } 
        return r;
    }
    void ut(int u,int v,int t) {
        int fu=find(u),fv=find(v);
        if(dep[fu]>dep[fv]) swap(u,v),swap(fu,fv);
        dep[fv]=max(dep[fv],dep[fu]+1); fa[fu]=fv; ti[fu]=t;
    }
    void init() {
        for(int i=1;i<=n;++i) fa[i]=i,dep[i]=1;
    }
}B;
int main() {
    red(n);red(m);
    A.init();B.init();
    for(int i=1;i<=m;++i) {
        int op,x,y;
        red(op);
        switch(op) {
            case 1: 
                red(x);red(y);
                A.ut(x,y,i);
                break;
            case 2:
                red(x);red(y);
                B.ut(x,y,i); 
                break;
            case 3: 
                red(x);red(y);
                A.addtag(x,y,i);
                break;
            case 4: 
                red(x);red(y);
                B.addtag(x,y,i);
                break; 
            case 5: 
                // 先在B上找到最后一次推平
                red(x);
                ll ans=0;
                DAT tp=B.ask(x);
                ans+=tp.val;
                ans+=A.ask(x,tp.ti);
                printf("%lld\n",ans);
                break; 
        }
    }
    return 0;
}
```
