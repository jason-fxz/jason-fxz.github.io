---
title: '2021省选训练44'
date: 1000-04-16 21:37:26
tags: [XJ]
categories: 题解
password: 'You know nothing'
message: '你无从知晓'
hidden: true
---

## T1 强度

### 题意

有一颗 $n$ 个节点的树以如下方式生成：  

- 根节点为 $1$；
- 对于点 $u(u\ne 1)$，其父亲节点将从所有满足 $v\mid u,v\ne u$ 中的 $v$ 中等概率随机选一个。  

求 $\sum_{i=1}^n\sum_{j=1}^n\mathrm{dist}(i,j)$ 的期望，对 $p$ 取模，$n\le 3\times 10^5$ ，保证 $p$ 是质数。

### 题解

显然有：
$$
\mathrm{dist}(u,v)=\mathit{dep}_u+\mathit{dep}_v-2\mathit{dep}_{\mathrm{LCA}(u,v)}
$$

深度的期望是很好求的，直接dp即可 $\displaystyle dep_{u}=\frac{1}{\sigma{(u)}}\sum_{v\mid u,v<u}dep_{v}+1$ ，其中 $\sigma(u)$ 表 $u$ 真因子个数。

那么重点在于如何计算后面那坨，形式化地，我们定义 $f_u$ 表示 ，在 $T_1$ 中满足 $u$ 是 $x$ 的祖先，在 $T_2$ 中 $u$ 是 $y$ 的祖先的点对 $(x,y)$ 的个数期望。

那么 $f_u=siz_u^2$ ，$siz_u$ 表示 $u$ 点子树大小的期望（同样dp可求）。再定义 $g_u$ 表示满足 $f_u$ 条件且 $u$ 是编号最大的那个，这样的话，其实就等价于 $\mathrm{LCA}(x,y)=u$（因为此时 $u-x$，$v-x$ 的路径不交上）。这个可以容斥，考虑减去 $\mathrm{LCA}(x,y)=ku,(k\ge 2)$ 的贡献：  
$$
g_u=siz^2_u-\sum_{k\ge 2} g_{ku}P_{u,ku}^2
$$
其中 $P_{u,ku}$ 表示 $ku$ 这个点在 $u$ 子树中的概率。至于为什么要乘两次，因为 $x,y$ 两点各算一次。$P_{u,ku}$ 很好求， $\displaystyle P_{u,ku}=\frac{1}{\sigma(u)}\sum_{j\mid k,j\ne k}P_{u,ju}$ 。

注意到 $g_u$ 其实就是满足 $\mathrm{LCA}(x,y)=u$ 的点对 $(x,y)$ 个数的期望。
$$
ans=2n\sum_{i=1}^n dep_i -2\sum_{i=1}^n g_idep_i
$$
做完了。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 300010;
int n;
ll mod;
ll Edep[N],Esiz[N]; // Edep[u] u点深度期望，Esiz[u] u 点子树大小期望
ll dp[N]; // dp[u] 表示 点对 (x,y) 满足 lca(x,y)=u 的个数的期望
ll inv[N];
ll P[N];
vector<int> D[N];// D[i] i 的所有真因子
template<typename Tp> void Add(Tp &x,Tp y) {x=(x+y)%mod;}
template<typename Tp> void Mul(Tp &x,Tp y) {x=(x*y)%mod;}

int main() {
    read(n,mod);
    for(int i=1;i<=n;++i){
        for(int j=i+i;j<=n;j+=i) {
            D[j].push_back(i);
        }
    } 
    inv[0]=inv[1]=1;
    for(int i=2;i<=n;++i) inv[i]=(mod-mod/i)*inv[mod%i]%mod;
    for(int i=1;i<=n;++i) {
        ll t=inv[D[i].size()];
        for(int j=0;j<D[i].size();++j) {
            Add(Edep[i],t*Edep[D[i][j]]);
        }    
        Edep[i]++;
    }
    for(int i=n;i>=1;--i) {
        Esiz[i]++;
        ll t=inv[D[i].size()];
        for(int j=0;j<D[i].size();++j) {
            Add(Esiz[D[i][j]],t*Esiz[i]);
        }
    }
    ll ans=0;
    // for(int i=1;i<=n;++i) Add(ans,Edep[i]*2ll);

    for(int u=n;u>=1;--u) {
        int tot=n/u; // tot : u 的孩子 2u,3u,4u...tot*u
        for(int i=0;i<=tot;++i) P[i]=0; // P[i] i*u 在 u 子树内的概率
        P[1]=1;
        for(int i=2;i<=tot;++i) {
            for(int j=0;j<D[i].size();++j) Add(P[i],P[D[i][j]]);
            Mul(P[i],inv[D[i*u].size()]);
        }
        dp[u]=(Esiz[u]*Esiz[u])%mod;
        for(int i=2;i<=tot;++i) {
            Add(dp[u],(mod-1)*P[i]%mod*P[i]%mod*dp[i*u]%mod);
        }
        Add(ans,(mod-2)*dp[u]%mod*Edep[u]%mod);
    }

    for(int i=1;i<=n;++i) Add(ans,2ll*Edep[i]*n%mod);
    printf("%lld\n",ans);
    return 0;
}
```

---

## T2 切割

### 题意

![prob](./prob.png)

给你一个长度为 $n$ 的序列 $\{u_i\}$ 和 $\{d_i\}$ 表示上图所示的栅栏（所有边都是水平或竖直的），切一刀从某边界点出发**水平**或**竖直**知道边界点结束。问至少几刀使原图只留下矩形。

### 题解

其实就是给你一个凹多边形，让你把它变成凸多边形（只有水平、竖直边的多边形，若是凸的必然是矩形）。称向内凹的点为凹点，向外凸的点为凸点。一次切割只会改变切割边界点的性质，显然不会切割凸点，若割一个凹点会将其变成一个凸点+一个平面，若割到平面会将其变成两个凸点。当所有凹点都被删完自然就结束了。

考虑一次切割至少删除一凹点，若要删除两个凹点这两凹点必须在同一水平、竖直线上且之间没有经过边界。需要最大化删除两个凹点的操作次数。这些可能的操作很容易预处理，竖直的枚举一遍，水平的用单调栈。

并不是所有操作都可以随便选，如下图所示，红色的水平操作与蓝色的竖直操作向交，执行完一个，另一个就不行了。故给每个操作建点，相交的操作连边，要求最大独立集。考虑该图是二分图（水平操作互相不交，竖直操作互相不交），二分图中最大独立集等于总点数减**最大匹配**。

![P2](./P2.png)

跑网络流？不需要，考虑一个 $l$ 到 $r$ 的水平操作只会与 $[l,r]$ 内的竖直操作有交，可以把竖直操作看成点，水平操作为区间，点匹配区间，贪心即可。

### CODE

```cpp
#include <bits/stdc++.h>
template<typename _Tp>
inline void read(_Tp &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(_Tp)(ch^48),ch=getchar();
    if(fg) x=-x;
}
template<typename _Tp,typename... Args>void read(_Tp &t,Args &... args){read(t);read(args...);}
using namespace std;
typedef long long ll;
const int N = 500010;
struct pp{int l,r;};
ostream &operator<<(ostream &out,const pp &p) {
    return out<<"["<<p.l<<", "<<p.r<<"]";
}
bool operator<(const pp &a,const pp &b) {
    return a.r>b.r;
}
int n,u[N],d[N];
int cnt; // cnt 总共的凹点数
vector<int> pts; // 竖切操作
vector<pp> lins; // 横切操作 

int stk[N],top;
int h[N];
int main() {
    read(n);
    for(int i=1;i<=n;++i) read(u[i],d[i]);
    for(int i=2;i<=n;++i) {
        if(u[i]!=u[i-1]) ++cnt;
        if(d[i]!=d[i-1]) ++cnt;
        if(u[i]!=u[i-1]&&d[i]!=d[i-1]) {
            pts.push_back(i);
        }
    }
    for(int i=2;i<=n;++i) {
        if(u[i-1]==u[i]) continue;
        h[i]=min(u[i-1],u[i]);
        if(u[i-1]<u[i]) {
            stk[++top]=i;
        }else {
            while(top>0&&h[stk[top]]>h[i]) --top;
            if(top>0&&h[stk[top]]==h[i]) {
                lins.push_back(pp{stk[top],i});               
                --top;
            }
        }
    }
    top=0;
    for(int i=2;i<=n;++i) {
        if(d[i-1]==d[i]) continue;
        h[i]=min(d[i-1],d[i]);
        if(d[i-1]<d[i]) {
            stk[++top]=i;
        }else {
            while(top>0&&h[stk[top]]>h[i]) --top;
            if(top>0&&h[stk[top]]==h[i]) {
                lins.push_back(pp{stk[top],i});               
                --top;
            }
        }
    }
    priority_queue<pp> Q;
    int mch=0;
    sort(lins.begin(),lins.end(),[](pp a,pp b) {return a.l<b.l;});
    int j=0;
    for(int i=0;i<(int)pts.size();++i) {
        while(j<(int)lins.size()&&lins[j].l<=pts[i]) {
            Q.push(lins[j]); ++j;
        }
        while(!Q.empty()&&Q.top().r<pts[i]){
            Q.pop();           
        }
        if(!Q.empty()&&Q.top().r>=pts[i]) {
            ++mch;Q.pop();
        }
    }
    cout<<cnt-(pts.size()+lins.size()-mch)<<endl;
    return 0;
}

```

---

## T3 字符串

### 题意

令 $S_0=\texttt{b},S_1=\texttt{a},S_{n}=S_{n-1}+S_{n-2}$，其中 $+$ 表字符串拼接。给你 $n$ 和 $q$ 个询问 $i_1,i_2,\dots,i_q$，问你 $S_n$ 后缀排序中排名第 $i$ 的后缀编号，对 $p$ 取模。

$$
1\le n\le 10^{18},1\le q\le 5\times 10^5,1\le p\le 10^9,1\le i_j\le \min(10^{18},|S_n|)
$$

### 题解

<!-- 
$S_n=S_{n-1}S_{n-2}=S_{n-2}S_{n-3}S_{n-2}$

令 $k=n-3$，若为 $k$ 为奇数

$$
\begin{aligned}
S_{k+1}S_{k}&=S_{k+1}S_{k-1}S_{k-2}\\
&=S_{k+1}S_{k-1}S_{k-3}S_{k-4}\\
&=S_{k+1}S_{k-1}S_{k-3}\dots S_{4}S_2S_1\\
&=S_{k+1}S_{k-1}S_{k-3}\dots S_{4}S_1S_0S_1
\end{aligned}
$$

断言 $S_{k}S_{k+1}=S_{k+1}S_{k-1}S_{k-3}\dots S_4S_1S_1S_0$：

- 基础情况，$k=1$，$S_1S_2=S_1S_1S_0$ 显然成立；
- 考虑 $k-2$ 已经成立，
  $$
  \begin{aligned}
  S_{k}S_{k+1}&=S_kS_{k}S_{k-1}\\
  &=S_{k}S_{k-1}S_{k-2}S_{k-1}\\
  &=S_{k}S_{k-1}S_{k-1}S_{k-3}\dots S_{4}S_1S_1S_0\\
  &=S_{k+1}S_{k-1}S_{k-1}S_{k-3}\dots S_{4}S_1S_1S_0
  \end{aligned}
  $$
  证毕。

$S_{k+1}S_{k}$ 和 $S_kS_{k+1}$ 相差不多，就最后两项一个是 $S_0S_1$，一个是 $S_1S_0$ -->

求后缀排名，$n$ 这么大，后缀排序显然不现实，考虑建 SAM，再在 SAM 上路径计数。

建出整个 SAM 同样不现实，那么来考察一下这个 SAM 有什么特殊的性质，~~手模一下可以发现~~：

![P3](./P3.png)

这个 SAM 由一条前缀链 + $n-2$ 条额外的边，具体的，由 $F_{i-1}-2$ 连向 $F_{i}-1$ 的边，$i$ 为奇数转移边是 $\texttt{a}$，偶数则为 $\texttt{b}$。

证明先gugu
