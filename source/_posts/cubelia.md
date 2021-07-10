---
title: cubelia
date: 2020-12-25 21:52:41
tags: [RMQ,单调栈]
categories:
- 题解
---



### 题意

定义一个长度为 $n$ 的序列 $A$ 的最大前缀和 $\mathrm{maxpre}(1,n)$ 为 $\max_{i=1}^{n}sum_i$ ，其中 $sum_i = \sum_{i=1}^n A_i$  

现给你长度为 $n$ 的序列 $a$，以及 $m$ 个询问 $q$ ，每次给出 $l,r$ ，让你求出 $\sum_{i=l}^{r}\sum_{j=i}^{r} \mathrm{maxpre}(i,j)$

**强制在线**， $n\leq 2\times 10^6,q\leq 10^7$

<!--more-->

### 题解

把 $\mathrm{maxpre}(l,r)$ 拆开来看，那么 $\mathrm{maxpre}(l,r) = \max_{i=l}^{r} sum_i - sum_{l-1}$ ，那么一次询问为 
$$
\sum_{i=l}^{r}\sum_{j=i}^{r} \mathrm{maxpre}(i,j) = \sum_{i=l}^{r}\sum_{j=i}^{r} \max_{k=i}^{j} sum_k - \sum_{i=l}^{r}\sum_{j=i}^{r}a_{i-1}
$$

显然后面那坨东西可以直接预处理等差数列搞掉，现在只考虑前面的 $\sum_{i=l}^{r}\sum_{j=i}^{r} \max_{k=i}^{j} sum_k$ ，即所有子区间的最大 $sum$ 值怎么求，这个方法就和 [HNOI2016 序列](https://www.luogu.com.cn/problem/P3246) 的在线做法一样了，至于这个做法怎么想出来的，我也不知道。。。

我们用 $[l_1,r_1][l_2,r_2]$ 表示所有左端点落在 $[l_1,r_1]$ ，右端点落在 $[l_2,r_2]$ 的所有区间的最大 $sum$ 之和

先处理出区间 $[l,r]$ 中 $sum$ 最大值所在的位置 $ps$ ，那么所有跨越 $ps$ 的区间的最大 $sum$ 值都是 $sum_{ps}$ （因为它最大），总共产生 $(r-ps+1)(ps-l+1) \times sum_{ps}$  的贡献，即处理了 $[l,ps][ps,r]$ 

又因为一个区间 $[l,r]$ 的询问可被拆为： 
$$
[l,r][l,r] = [l,ps][ps,r] + [l,ps-1][l,ps-1] + [ps+1,r][ps+1,r]
$$ 
所以考虑如何处理后两项基本相同结构的询问（显然处理出 $ps$ 是有用的，但至于什么用处请看后文分析）

以左半段为例，它可以被拆分为：
$$
[l,ps-1][l,ps-1] = [l,n][l,n]-[ps,n][ps,n]-[l,ps-1][ps,n]
$$
其中又出现两个相同结构的东西。形式化的来讲，我们又要求的就是  $[i,n][i,n]$，而这个东西预处理走起

不难发现 $[i,n][i,n] = [i+1,n][i+1,n]+[i,i][i,n]$ ，这个递推式（形如 $f(x) = f(x+1) + delta$）显然只要知道增量 $[i,i][i,n]$ 的值为多少就可以了，因为初值为 $0$

$[i,i][i,n]$ 的求法就是单调栈的基础应用，左端点为 $i$ ，右端点为 $[i,n]$ 中每一个取值，显然单调栈一个元素计入它的高度（$sum_x$) 和宽度（有几个区间的最大值为 $sum_x$ ），每次把 $sum_i$ 压进栈，看看能更新几个区间的最大值，就像求面积一样，得到增量 $delta$ ，不妨简记为 $F(i)$

回归左半段，$[l,n][l,n]-[ps,n][ps,n]$ 已经被我们求出，还差一个 $[l,ps-1][ps,n]$ ，这就是我们为什么要求出区间最大值所在的位置 $ps$ 的原因，考虑对于任意位于 $[l,ps-1]$ 的左端点 $x$ ，当他的右端点在 $[ps,n]$ 时区间的最大值与 $[l,ps-1]$ 中 $sum$ 的取值无关。因为能肯定的是 $sum_{ps}>=sum_x,x\in [l,ps-1]$ ，当 $x$ 取值固定的时候它的贡献 $[x,x][ps,n]$ 其实就等于 $F(ps) $

道理是很简单的，但是总感觉有些别扭，所以还是证明下，当 $x\in [l,ps-1],i\in[ps,n]$ 时，$[x,x][i,i] = [ps,ps][i,i]$ ，即

$$
[x,x][ps,n] = \sum_{i=ps}^{n} [x,x][i,i] = \sum_{i=ps}^{n} [ps,ps][i,i] = F(ps)
$$

所以左半段求解完毕，右半段同理

复杂度瓶颈在于区间最值，这个用 **分块 ST表** 就可以了，$O(n)$ - 期望 $O(1)$  。

### CODE

[「HNOI2016」序列 数据加强版 加强版](https://loj.ac/p/6376)

```cpp
#include <bits/stdc++.h>
#define MOD(x) ((x<mod)?(x):((x)%mod))
#define fi first
#define se second
using namespace std;
template <typename T>
inline void red(T &x) {
    x = 0;
    bool f = 0;
    char ch = getchar();

    while (ch < '0' || ch > '9') {
        if (ch == '-')
            f ^= 1;

        ch = getchar();
    }

    while (ch >= '0' && ch <= '9')
        x = (x << 1) + (x << 3) + (T)(ch ^ 48), ch = getchar();

    if (f)
        x = -x;
}
typedef long long ll;
typedef pair<int, int> pii;
const int N = 4194304;
const int mod = 1e9 + 7;
int n, q, a[N];
namespace gen {
int A, B, C, P;
ll lastAns;
inline int rnd() {
    return A = (A * B + (C ^ (int)(lastAns & 0x7fffffffLL)) % P) % P;
}
void init() {
    red(A);
    red(B);
    red(C);
    red(P);
}
}
inline int gmin(int x, int y) {
    return a[x] < a[y] ? x : y;
}
namespace RMQ {
const int B = 32;
const int M = 23;
int bl[N], F[M][N], ps[N], pre[N], suf[N], LOG[N], tot;
// bl 属于哪个块，F 块间ST表 ps块内最值下标 pre/suf 块内前缀/后缀最值
void init() {
    for (int i = 1; i <= n; ++i) {
        bl[i] = (i - 1) / B + 1;

        if (bl[i] != bl[i - 1]) {
            ps[bl[i - 1]] = pre[i - 1];
            pre[i] = i;
        } else
            pre[i] = gmin(pre[i - 1], i);
    }

    ps[bl[n]] = pre[n];
    tot = bl[n];

    for (int i = n; i >= 1; --i) {
        if (bl[i] != bl[i + 1]) {
            suf[i] = i;
        } else
            suf[i] = gmin(suf[i + 1], i);
    }

    LOG[0] = -1;

    for (int i = 1; i <= tot; ++i)
        LOG[i] = LOG[i >> 1] + 1;

    for (int i = 1; i <= tot; ++i)
        F[0][i] = ps[i];

    for (int j = 1; j <= LOG[tot]; ++j) {
        for (int i = 1; i <= tot; ++i) {
            if (i + (1 << j) - 1 > tot)
                break;

            F[j][i] = gmin(F[j - 1][i], F[j - 1][i + (1 << (j - 1))]);
        }
    }
}
int ask(int l, int r) { // 返回区间最值位置
    if (bl[l] == bl[r]) {
        int p = l;

        for (int i = l + 1; i <= r; ++i)
            p = gmin(p, i);

        return p;
    }

    if (bl[l] + 1 == bl[r])
        return gmin(suf[l], pre[r]);

    int L = bl[l] + 1, R = bl[r] - 1;
    int k = LOG[R - L + 1];
    return gmin(gmin(F[k][L], F[k][R - (1 << k) + 1]), gmin(suf[l], pre[r]));
}
}
ll f1[N], f2[N], F1[N], F2[N];
// F1[i] 表示左端点为i的所有区间min的和, f1[i]为F1后缀 ;F2[i] 右端点为i，f2[i] 为 F2 前缀

pii st[N]; // pii(a,cnt)
int top;
void prepare() {
    for (int i = n, cnt; i >= 1; --i) {
        F1[i] = F1[i + 1];
        cnt = 1;

        while (top > 0 && st[top].fi >= a[i]) {
            F1[i] -= (ll)st[top].fi * st[top].se;
            cnt += st[top].se;
            --top;
        }

        F1[i] += (ll)a[i] * cnt;
        st[++top] = pii(a[i], cnt);
    }

    top = 0;

    for (int i = n; i >= 1; --i)
        f1[i] = f1[i + 1] + F1[i];

    for (int i = 1, cnt; i <= n; ++i) {
        F2[i] = F2[i - 1];
        cnt = 1;

        while (top > 0 && st[top].fi >= a[i]) {
            F2[i] -= (ll)st[top].fi * st[top].se;
            cnt += st[top].se;
            --top;
        }

        F2[i] += (ll)a[i] * cnt;
        st[++top] = pii(a[i], cnt);
    }

    top = 0;

    for (int i = 1; i <= n; ++i)
        f2[i] = f2[i - 1] + F2[i];
}
void solve(int l, int r) {
    int p = RMQ::ask(l, r);
    ll ans = (ll)a[p] * (p - l + 1) * (r - p + 1);
    // left = [l,n][l,n] - [p,n][p,n] - [l,p-1][p,n]
    ans += f1[l] - f1[p] - (ll)(p - l) * F1[p];
    // rigth = [1,r][1,r] - [1,p][1,p] - [p+1,r][1,p]
    ans += f2[r] - f2[p] - (ll)(r - p) * F2[p];
    gen::lastAns = ans;
}
int main() {
    red(n);
    red(q);
    for (int i = 1; i <= n; ++i)
        red(a[i]);
    prepare();
    RMQ::init();
    gen::init();
    ll ANS = 0;
    for (int ts = 1; ts <= q; ++ts) {
        int l = gen::rnd() % n + 1, r = gen::rnd() % n + 1;

        if (l > r)
            swap(l, r);

        solve(l, r);
        ANS = (ANS + gen::lastAns) % mod;
        (ANS < 0) &&(ANS += mod);
    }
    printf("%lld\n", ANS);
}
```

**Cubelia**

```cpp
#pragma GCC optimize(2)
#include <bits/stdc++.h>
#define MOD(x) ((x<mod)?(x):((x)%mod))
#define fi first
#define se second
using namespace std;
template <typename T>
inline void red(T &x) {
    x=0;bool f=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') f^=1; ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(f) x=-x;
}
typedef long long ll;
typedef pair<int,int> pii;
const int N = 2097152;
const int mod = 998244353;
int n,q; ll a[N];
namespace gen{
    int S,A,B,P,tp;
    ll lastans=0;
    inline int Rand() { S=(S*A%P+(B^(tp*lastans)))%P;S=S<0?-S:S;return S;}
    void init() {red(S);red(A);red(B);red(P);red(tp);}
}
inline int gmax(int x,int y) {return a[x]>a[y]?x:y;}
namespace RMQ {
    const int B = 64;
    const int M = 22;
    int bl[N],F[M][N],ps[N],pre[N],suf[N],LOG[N],tot;
    // bl 属于哪个块，F 块间ST表 ps块内最值下标 pre/suf 块内前缀/后缀最值
    void init() {
        for(int i=1;i<=n;++i) {
            bl[i]=(i-1)/B+1;
            if(bl[i]!=bl[i-1]) {
                ps[bl[i-1]]=pre[i-1];
                pre[i]=i;
            }else pre[i]=gmax(pre[i-1],i);
        }
        ps[bl[n]]=pre[n]; tot=bl[n];
        for(int i=n;i>=1;--i) {
            if(bl[i]!=bl[i+1]) {
                suf[i]=i;
            }else suf[i]=gmax(suf[i+1],i);
        }
        LOG[0]=-1;
        for(int i=1;i<=tot;++i) LOG[i]=LOG[i>>1]+1;
        for(int i=1;i<=tot;++i) F[0][i]=ps[i];
        for(int j=1;j<=LOG[tot];++j) {
            for(int i=1;i<=tot;++i) {
                if(i+(1<<j)-1>tot) break;
                F[j][i]=gmax(F[j-1][i],F[j-1][i+(1<<(j-1))]);
            }
        }
    }
    int ask(int l,int r) { // 返回区间最值位置
        if(bl[l]==bl[r]) {
            int p=l;
            for(int i=l+1;i<=r;++i) p=gmax(p,i);
            return p;
        }
        if(bl[l]+1==bl[r]) return gmax(suf[l],pre[r]);
        int L=bl[l]+1,R=bl[r]-1;
        int k=LOG[R-L+1];
        return gmax(gmax(F[k][L],F[k][R-(1<<k)+1]),gmax(suf[l],pre[r]));
    }
}
ll f1[N],f2[N],F1[N],F2[N];
// F1[i] 表示左端点为i的所有区间max的和, f1[i]为F1后缀 ;F2[i] 右端点为i，f2[i] 为 F2 前缀
ll G[N],g[N];
pii st[N]; // pii(a,cnt)
int top;
void prepare() {
    for(int i=n,cnt;i>=1;--i) {
        F1[i]=F1[i+1]; cnt=1;
        while(top>0&&st[top].fi<=a[i]) { F1[i]-=(ll)st[top].fi*st[top].se; cnt+=st[top].se; --top;}
        F1[i]+=(ll)a[i]*cnt; st[++top]=pii(a[i],cnt);
    } top=0;
    for(int i=n;i>=1;--i) f1[i]=f1[i+1]+F1[i];
    for(int i=1,cnt;i<=n;++i) {
        F2[i]=F2[i-1]; cnt=1;
        while(top>0&&st[top].fi<=a[i]) { F2[i]-=(ll)st[top].fi*st[top].se; cnt+=st[top].se; --top;}
        F2[i]+=(ll)a[i]*cnt; st[++top]=pii(a[i],cnt);
    } top=0;
    for(int i=1;i<=n;++i) f2[i]=f2[i-1]+F2[i];
 
    for(int i=n;i>=0;--i) G[i]=G[i+1]+a[i]*(n-i+1);
    for(int i=n;i>=0;--i) g[i]=g[i+1]+a[i];
}
void solve(int l,int r) {
    int p=RMQ::ask(l,r);
    ll ans=(ll)a[p]*(p-l+1)*(r-p+1);
    // left = [l,n][l,n] - [p,n][p,n] - [l,p-1][p,n]
    ans+=f1[l]-f1[p]-(ll)(p-l)*F1[p];
    // rigth = [1,r][1,r] - [1,p][1,p] - [p+1,r][1,p]
    ans+=f2[r]-f2[p]-(ll)(r-p)*F2[p];
    ll s=g[l-1]-g[r];
    ll tmp=G[l-1]-G[r]-s*(ll)(n+1-r);
    ans-=tmp;
    gen::lastans=ans;
}
int main() {
    red(n);red(q);
    for(int i=1;i<=n;++i) red(a[i]);
    for(int i=1;i<=n;++i) a[i]+=a[i-1];
    prepare(); RMQ::init();
    gen::init();
    ll ANS=0;
    for(int ts=1;ts<=q;++ts) {
        int l=gen::Rand()%n+1,r=gen::Rand()%n+1;
        if(l>r) swap(l,r);
        solve(l,r);
        ANS=(ANS+gen::lastans)%mod;
        (ANS<0)&&(ANS+=mod);
    }
    printf("%lld\n",ANS);
}
/*
5 3
55419 228586 -483578 -148471 -100617
907442592 999221847 909035853 910239943 1
*/
```
