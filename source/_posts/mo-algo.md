---
title: '莫队二次离线'
date: 2021-03-01 21:53:37
tags: [莫队,分块]
categories:
- 📚算法笔记
---

## 引入

长度为 $n$ 的序列 $a$ ，有 $n$ 次询问（这里假设序列长度与询问次数同阶），每次询问 $[l,r]$ 中满足某种条件的点对数量。

## 分析

区间 $[l,r]$ 中与 $x$ 构成关键对个数为 $f([l,r],x)$ ，考虑右端点从 $r$ 到 $r'$ 答案变化：
$$
f([l,r],r+1)+f([l,r+1],r+2)+\dots+f([l,r'-1],r')
$$
显然有 $f([l,r],x)=f([1,r],x)-f([1,l-1],x)$ ，那么：
$$
\sum_{i=r}^{r'-1} f([1,i],i+1)-\sum_{i=r}^{r'-1} f([1,l-1],i+1)
$$
左边的可以预处理，右边的离线下来统一询问。

同理，左端点从 $l$ 到 $l'$ ：
$$
f([l,r],l-1) +f([l-1,r],l-2)+\dots+f([l'+1,r],l')
$$
显然有 $f([l,r],x)=f([l,n],x)-f([r+1,n],x)$ ，那么：
$$
\sum_{i=l'}^{l-1} f([i+1,n],i) - \sum_{i=l'}^{l-1} f([r+1,n],i)
$$
左边的可以预处理，右边的离线下来统一询问。

可以发现我们再次离线下来的询问有 $O(n\sqrt{n})$ 个（即指针移动次数），先只考虑右指针离线的询问。 每个询问形式如 $f([1,x],i)$  ，$x\in n$  我们在 $x$ 位置打上 $i$ 的标记，表示有 $i$ 点向 $[1,x]$ 询问关键对数。考虑 $x$ 枚举 $1\to n$ （修改次数 $O(n)$） 而询问次数有 $O(n\sqrt{n})$ ，故需要实现 $O(\sqrt{n})$ 修改，$O(1)$ 询问，这样总复杂度可到 $O(n\sqrt{n})$ 。事实上修改的复杂度不一定为 $O(\sqrt{n})$ ，我们设它为 $O(k)$，那么莫队二次离线的总复杂度为 $O(nk+n\sqrt{n})$ 。

## 总结

1. 预处理 $Fr(i) = f([1,i],i+1) ,\;Fl(i) = f([i+1,n],i)$ ，复杂度 $O(nk)$；
2. 对询问跑一遍莫队，答案算上已经预处理好的部分，同时离线指针移动的询问，如右端点 $f([1,l-1],r+1\sim r')$ ，左端点 $f([r+1,n],l'\sim l-1)$ ，复杂度 $O(n\sqrt{n})$；
3. 处理离线下来的询问，右端点的形如 $f([1,x],i)$ ，枚举 $x$ 从 $1$ 到 $n$ ，$O(k)$ 修改，$O(1)$ 查询，左端点的同理，复杂度 $O(nk+n\sqrt{n})$；
4. 对答案进行前缀（因为每个询问之计算了相对上一次的变化），复杂度 $O(n)$。

事实上对于不同的题目，我们主要考虑步骤 3 如何做到 $O(1)$ 询问，$O(k)$ 合理复杂度修改。

给一个版子：

```cpp
// 莫队二次离线
#include <bits/stdc++.h>
template <typename Tp>
inline void read(Tp &x) {
    x = 0; bool fg = 0; char ch = getchar();
    while(ch < '0' || ch > '9') {
        if(ch == '-') fg ^= 1;
        ch = getchar();
    }
    while(ch >= '0' && ch <= '9') x = (x << 1) + (x << 3) + (Tp)(ch ^ 48), ch = getchar();
    if(fg) x = -x;
}
using namespace std;
typedef long long ll;
const int N = 100010;
struct mds {
    int l, r, id, fg;
};
vector<mds> mdl[N], mdr[N];
vector<mds>::iterator mt;
vector<int>::iterator it;
int a[N], n, m, K;
int L[N], R[N], t[N], bl[N], B;
ll Fl[N], Fr[N], ans[N];
bool cmp(int x, int y) {
    if (bl[L[x]] == bl[L[y]]) return R[x] < R[y];
    return L[x] < L[y];
}
void init() {
    // init Fr,Fl
    for (int i = 1; i < n; ++i) {
        // Fr(i) = f([1,i],i+1)
    }
    for (int i = n - 1; i >= 1; --i) {
        // Fl(i) = f([i+1,n],i)
    }
    for (int i = 2; i <= n; ++i) Fl[i] += Fl[i - 1], Fr[i] += Fr[i - 1];
}
void solve() {
    for (int x = 1; x <= n; ++x) {
        // updata(a[x])
        for (mt = mdr[x].begin(); mt != mdr[x].end(); ++mt)
            for (int i = mt->l; i <= mt->r; ++i) ans[mt->id] += mt->fg * (1/*query(a[i])*/);
    }
    for (int x = n; x >= 1; --x) {
        // updata(a[x])
        for (mt = mdl[x].begin(); mt != mdl[x].end(); ++mt)
            for (int i = mt->l; i <= mt->r; ++i) ans[mt->id] += mt->fg * (1/*query(a[i])*/);
    }

}
int main() {
    read(n); read(m);
    for (int i = 1; i <= n; ++i) read(a[i]);
    B = sqrt(n); for (int i = 1; i <= n; ++i) bl[i] = (i - 1) / B + 1;
    for (int i = 1; i <= m; ++i) read(L[i]), read(R[i]), t[i] = i;
    sort(t + 1, t + m + 1, cmp);
    init();
    int l = 1, r = 1;
    for (int i = 1; i <= m; ++i) {
        int id = t[i];
        if (r < R[id]) {
            ans[id] += Fr[R[id] - 1] - Fr[r - 1];
            mdr[l - 1].push_back(mds{ r + 1, R[id], id, -1 }); r = R[id];
        }
        if (l > L[id]) {
            ans[id] += Fl[l - 1] - Fl[L[id] - 1];
            mdl[r + 1].push_back(mds{ L[id], l - 1, id, -1 }); l = L[id];
        }
        if (r > R[id]) {
            ans[id] -= Fr[r - 1] - Fr[R[id] - 1];
            mdr[l - 1].push_back(mds{ R[id] + 1, r, id, 1 }); r = R[id];
        }
        if (l < L[id]) {
            ans[id] -= Fl[L[id] - 1] - Fl[l - 1];
            mdl[r + 1].push_back(mds{ l, L[id] - 1, id, 1 }); l = L[id];
        }
    }
    solve();
    for (int i = 1; i <= m; ++i) ans[t[i]] += ans[t[i - 1]];
    for (int i = 1; i <= m; ++i) {
        printf("%lld\n", ans[i]);
    }
    return 0;
}
```

---

## 例题

### 第十四分块(前体)

[luogu P4887](https://www.luogu.com.cn/problem/P4887)

题意：给定 $n,m,k$，一个长度为 $n$ 序列 $a$ ，$m$ 次询问，每次询问区间 $[l,r]$ 内有多少点对 $(i,j),l\le i\lt j\le r$ 满足 $\mathrm{popcount}(a_i \oplus a_j)=k$ ，$1\le n,m\le 10^5,0\le a_i \lt 2^{15}$ 。

#### 题解

值域 $2^{14}$ ，枚举关键对复杂度为 $\binom{14}{k}$ 。对于 $f([1,i],i+1),i\in[1,n-1]$ 预处理复杂度 $O\left(\binom{14}{k}n\right)$ ，只需要记录 $cnt(x)$ 表示当前区间 $x$ 出现次数，对值 $y$ 找关键点对只需要枚举 $v,\mathrm{popcount}(v)=k$ 然后 $ans+\!\!= cnt(y\oplus j)$ 即可，$\mathrm{popcount}(j)=k$ 中 $j$ 的个数是 $\binom{14}{k}$ 。 $f([i+1,n],i),i\in[1,n-1]$ 的预处理同理。

对于离线下来的询问 $f([1,x],i)$ ，我们希望 $O(1)$ 查询，那么 $ct(x)$ 表示当前区间中异或上 $x$ 有 $k$ 个一的数的个数。添加一个 $a$ 的修改便是 $ct(v\oplus a)\!+\!+$ ，$v$ 满足 $\mathrm{popcount}(v)=k$。对于 $f([x,n],i)$ 同理。

#### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
using namespace std;
typedef long long ll;
const int N = 100010;
struct mds{int l,r,id,fg;};
vector<mds> mdl[N],mdr[N];
vector<mds>::iterator mt;
vector<int>::iterator it;
int a[N],n,m,K;
int L[N],R[N],t[N],bl[N],B;
ll Fl[N],Fr[N],ans[N];
// Fr(i) = f([1,i],i+1)  Fl(i) = f([i+1,n],i)
int popcount[1<<14],ct[1<<14];
vector<int> pct;
bool cmp(int x,int y) {
    if(bl[L[x]]==bl[L[y]]) return R[x]<R[y];
    return L[x]<L[y];
}
void init() {
    for(int i=1,x=1;i<(1<<14);++i,x=i) while(x) x-=x&-x,++popcount[i];
    for(int i=0;i<(1<<14);++i) if(popcount[i]==K) pct.push_back(i);
    for(int i=1;i<n;++i) {
        ++ct[a[i]];
        for(it=pct.begin();it!=pct.end();++it) Fr[i]+=ct[(*it)^a[i+1]];
    }
    memset(ct,0,sizeof(ct));
    for(int i=n-1;i>=1;--i) {
        ++ct[a[i+1]];
        for(it=pct.begin();it!=pct.end();++it) Fl[i]+=ct[(*it)^a[i]];
    }
    for(int i=2;i<=n;++i) Fl[i]+=Fl[i-1],Fr[i]+=Fr[i-1];
}
void solve() {
    memset(ct,0,sizeof(ct));
    for(int x=1;x<=n;++x) {
        for(it=pct.begin();it!=pct.end();++it) ++ct[(*it)^a[x]];
        for(mt=mdr[x].begin();mt!=mdr[x].end();++mt) for(int i=mt->l;i<=mt->r;++i) ans[mt->id]+=mt->fg*ct[a[i]];
    }
    memset(ct,0,sizeof(ct));
    for(int x=n;x>=1;--x) {
        for(it=pct.begin();it!=pct.end();++it) ++ct[(*it)^a[x]];
        for(mt=mdl[x].begin();mt!=mdl[x].end();++mt) for(int i=mt->l;i<=mt->r;++i) ans[mt->id]+=mt->fg*ct[a[i]];
    }

}
int main() {
    red(n);red(m);red(K);
    if(K>14) {for(int i=1;i<=m;++i) puts("0"); return 0;}
    for(int i=1;i<=n;++i) red(a[i]);
    B=sqrt(n); for(int i=1;i<=n;++i) bl[i]=(i-1)/B+1;
    for(int i=1;i<=m;++i) red(L[i]),red(R[i]),t[i]=i;
    sort(t+1,t+m+1,cmp);
    init();
    int l=1,r=1;
    for(int i=1;i<=m;++i) {
        int id=t[i];
        if(r<R[id]) {
            ans[id]+=Fr[R[id]-1]-Fr[r-1];
            mdr[l-1].push_back(mds{r+1,R[id],id,-1}); r=R[id];
        }
        if(l>L[id]) {
            ans[id]+=Fl[l-1]-Fl[L[id]-1];
            mdl[r+1].push_back(mds{L[id],l-1,id,-1}); l=L[id];
        }
        if(r>R[id]) {
            ans[id]-=Fr[r-1]-Fr[R[id]-1];
            mdr[l-1].push_back(mds{R[id]+1,r,id,1}); r=R[id];
        }
        if(l<L[id]) {
            ans[id]-=Fl[L[id]-1]-Fl[l-1];
            mdl[r+1].push_back(mds{l,L[id]-1,id,1}); l=L[id];
        }
    }
    solve();
    for(int i=1;i<=m;++i) ans[t[i]]+=ans[t[i-1]];
    for(int i=1;i<=m;++i) {
        printf("%lld\n",ans[i]);
    }
    return 0;
}

```

### 区间逆序对

[BZOJ3289 Mato的文件管理](https://darkbzoj.tk/problem/3289)

长度为 $n$  序列 $a$ ，有 $q$  次询问，每次询问 $[l,r]$ 内逆序对数。$1\le n,q\le 50000$

#### 题解

类似的，我们套用上面的板子，只需要考虑如何 $O(1)$ 询问关键对，$O(k)$ 合理复杂度修改。

考虑此时求的是逆序对数，$f([1,x],i)$ 即求 $a_j > a_i ,j\in [1,x]$ 的个数，即求值域上的后缀，考虑**值域分块**，便可做到 $O(\sqrt{n})$ 修改 $O(1)$ 查询。

#### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void red(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
using namespace std;
typedef long long ll;
const int N = 50010;
int n,q,a[N],L[N],R[N],ans[N],t[N];
int Fl[N],Fr[N]; // Fr(i) = f([1,i],i+1)  Fl(i) = f([i+1,n],i)
struct mds{int l,r,id;};
int bl[N],B;
vector<mds> mdl[N],mdr[N];
bool cmp1(int x,int y) {
    if(bl[L[x]]==bl[L[y]]) return R[x]<R[y];
    return bl[L[x]]<bl[L[y]];
}
struct trearr{
    int t[N],n;
    void init(int _n) {n=_n; memset(t,0,sizeof(int)*(n+1));}
    void add(int x,int v) {for(;x<=n;x+=x&-x) t[x]+=v;}
    int sum(int x) {int r=0;for(;x>0;x-=x&-x) r+=t[x];return r;}
}T;

int val[N],totval;
void init() {
    for(int i=1;i<=n;++i) val[++totval]=a[i];
    sort(val+1,val+totval+1);
    totval=unique(val+1,val+totval+1)-val-1;
    for(int i=1;i<=n;++i) a[i]=lower_bound(val+1,val+totval+1,a[i])-val;
    T.init(n);
    for(int i=1;i<n;++i) {
        T.add(n-a[i]+1,1);
        Fr[i]=T.sum(n-a[i+1]);
    }
    T.init(n);
    for(int i=n;i>1;--i) {
        T.add(a[i],1);
        Fl[i-1]=T.sum(a[i-1]-1);
    }
    for(int i=1;i<=n;++i) Fl[i]+=Fl[i-1],Fr[i]+=Fr[i-1];
}
void solve() {
    static int cnt[N],tg[230];
    memset(cnt,0,sizeof(cnt)); memset(tg,0,sizeof(tg));
    B=sqrt(totval); for(int i=1;i<=totval;++i) bl[i]=(i-1)/B+1;
    // a[x:n] 内 < i 的个数 
    for(int x=n;x>=1;--x) {
        for(int p=bl[a[x]]+1;p<=bl[totval];++p) ++tg[p];
        int ed=min(bl[a[x]]*B,totval);
        for(int i=a[x]+1;i<=ed;++i) ++cnt[i];
        for(auto &&v:mdl[x]) {
            for(int i=v.l;i<=v.r;++i) {
                if(v.id<0) ans[-v.id]-=cnt[a[i]]+tg[bl[a[i]]];
                else ans[v.id]+=cnt[a[i]]+tg[bl[a[i]]];
            } 
        }
    }
    memset(cnt,0,sizeof(cnt)); memset(tg,0,sizeof(tg));
    // a[1:x] 内 > i 的个数  cnt[i]+tg[bl[i]]
    for(int x=1;x<=n;++x) {
        for(int p=1;p<bl[a[x]];++p) ++tg[p];
        for(int i=(bl[a[x]]-1)*B+1;i<a[x];++i) ++cnt[i];
        for(auto &&v:mdr[x]) {
            for(int i=v.l;i<=v.r;++i) {
                if(v.id<0) ans[-v.id]-=cnt[a[i]]+tg[bl[a[i]]];
                else ans[v.id]+=cnt[a[i]]+tg[bl[a[i]]];
            } 
        }
    }
}
int main() {
    red(n); for(int i=1;i<=n;++i) red(a[i]);
    red(q); for(int i=1;i<=q;++i) red(L[i]),red(R[i]),t[i]=i;
    init(); B=sqrt(n); for(int i=1;i<=n;++i) bl[i]=(i-1)/B+1;
    sort(t+1,t+q+1,cmp1);
    int pl=1,pr=0;
    for(int i=1;i<=q;++i) {
        int id=t[i];
        if(pl>L[id]) {
            ans[id]+=Fl[pl-1]-Fl[L[id]-1];
            mdl[pr+1].push_back(mds{L[id],pl-1,-id}); pl=L[id];
        }
        if(pr<R[id]) {
            ans[id]+=Fr[R[id]-1]-Fr[pr-1];
            mdr[pl-1].push_back(mds{pr+1,R[id],-id}); pr=R[id];
        }
        if(pl<L[id]) {
            ans[id]-=Fl[L[id]-1]-Fl[pl-1];
            mdl[pr+1].push_back(mds{pl,L[id]-1,+id}); pl=L[id];
        }
        if(pr>R[id]) {
            ans[id]-=Fr[pr-1]-Fr[R[id]-1];
            mdr[pl-1].push_back(mds{R[id]+1,pr,+id}); pr=R[id];
        }
    }
    solve();
    for(int i=1;i<=q;++i) ans[t[i]]+=ans[t[i-1]];
    for(int i=1;i<=q;++i) {
        printf("%d\n",ans[i]);
    }
    return 0;
}
```
