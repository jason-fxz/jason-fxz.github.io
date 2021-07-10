---
title: '数树(voltississimo)'
date: 2020-11-01 11:02:49
tags: [XJ,动态规划,容斥原理,树形dp]
categories: 题解
hidden: true
---

## 数树 (voltississimo)

**[problem](https://dev.xjoi.net/contest/1585/problem/2)**

$Rasis$ 给了你一颗 $n$ 个点的树,节点编号为 $1$ − $n$，**$Rasis$ 还给这棵树上的每条边 $(u, v)$ 规定了一个方向**。$Rasis$ 想给这棵树上每个点分配一个权值 $a_i$ ，满足 $a_1 \dots a_n$ 组成一个 $1$ − $n$ 的排列且不存在一条从 $u$ 指向 $v$ 的边满足 $a_v = a_u + 1$ （注意从 $u$ 指向 $v$ 意味着这个约束是单向的）。Rasis 想知道这么分配权值的方案数对 $998244353$ 取模的值。

$n\le 5000$


> 两种分配方案不同,当且仅当存在一个节点的权值在两种方案中不相同。

![BtrUWd.png](https://s1.ax1x.com/2020/10/30/BtrUWd.png)

#### $n\le 18$ 容斥

暴力枚举 $S\subseteq  E$ ，$S$ 中的边所对应的条件强制不满足（ 即 $u\to v$ 则强制 $a_v=a_u+1$ ），相当于把 $a_v,a_u$ 捆绑在一起。当然 $S$ 有可能不合法，如 $u\to v_1,u\to v_2 \in S$ ，那么 $a_{v_1}=a_{v_2}$ 显然不合法，$S$ 合法需要满足每个点的入度出度都不超过一

状压枚举即可，$O(n\cdot 2^n)$ 
$$
ans=\sum_{i=0}^{n-1} (-1)^i\sum_{|S|=i 且S合法} (n-i)!
$$

#### 树是一条链

容斥的特殊情况，枚举到的 $S$ 一定都的特殊情况，枚举到的  一定都是合法的， 即为  的方案数， 是合法的，$\binom{n-1}{i}$ 即为 $|S|=i$ 的方案数，$O(n)$ 
$$
\binom{n-1}{0}n!-\binom{n-1}{1}(n-1)!+\binom{n-1}{2}(n-2)!-\binom{n-1}{3}(n-3)!\dots\\
$$

$$
ans=\sum_{i=0}^{n-1} (-1)^i\binom{n-1}{i} (n-i)!
$$

#### 外向树

为保证 $S$ 合法，外向树上每个结点的出边只能选一条，$dp[i]$ 表示在原树内选 $i$ 条边合法的方案数，简单背包 $O(n^2)$
$$
ans=\sum_{i=0}^{n-1} (-1)^idp[i] (n-i)!
$$

#### 正解

在外向树的基础上进一步~~瞎搞~~，无非是原本外向树上的边都是向下，现在有向上的。还是考虑 dp 出原树内选 $i$ 条边合法的方案。

设状态为 $dp[u][j][0/1/2/3]$ ，表示在 $u$ 子树内选 $j$ 条边，满足条件：

- **0**  $u$ 的子树内没与 $u$ 连边
- **1**  $u$ 的子树内有一条孩子连向 $u$ 的边
- **2**  $u$ 的子树内有一条 $u$ 连向孩子的边
- **3**  $u$ 的子树内有一条孩子连向 $u$ 的边 + 一条 $u$ 连向孩子的边（ $u$不能再连了）

然后就是树形背包转移，转移比较~~显然~~吧：
$$
dp[u][j+k][c]\leftarrow dp[u][j][c]\cdot dp[v][0,1,2,3]\quad c\in\{0,1,2,3\},v\in son_u
$$
$$
dp[u][j+k+1][1]\leftarrow dp[u][j][0]\cdot dp[v][k][0,1]\quad v\in son_u,v\to u
$$
$$
dp[u][j+k+1][3]\leftarrow dp[u][j][2]\cdot dp[v][k][0,1]\quad v\in son_u,v\to u
$$
$$
dp[u][j+k+1][2]\leftarrow dp[u][j][0]\cdot dp[v][k][0,2]\quad v\in son_u,u\to v
$$
$$
dp[u][j+k+1][3]\leftarrow dp[u][j][1]\cdot dp[v][k][0,2]\quad v\in son_u,u\to v
$$

### CODE

```cpp
#include <bits/stdc++.h>
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
const int N = 5010;
const ll mod = 998244353;
int head[N],nxt[N<<1],pnt[N<<1],E,c[N<<1],n;
void adde(int u,int v,int cc) {E++;pnt[E]=v;c[E]=cc;nxt[E]=head[u];head[u]=E;}
ll dp[N][N][4],fac[N]={1,1};
int sz[N];
/*
dp[u][j]  在u这颗子树内，选了j条边，
[0]: 当前u下方没连边
[1]: 有一条连向u的边
[2]: 有一条u连出去的边
[3]: 有一条连向u的边+有一条u连出去的边（u不能再连了）

dp[u][0] <- dp[v][0-3]
dp[u][1] <- one(v0->u) (dp[v0][0]+dp[v0][1])   other dp[v][0-3]
dp[u][2] <- one(v0<-u) (dp[v0][0]+dp[v0][2])   other dp[v][0-3]
dp[u][3] <- one(v0->u) (dp[v0][0]+dp[v0][1]) one(v1<-u) (dp[v1][0]+dp[v1][2])  other dp[v][0-3]
*/
void add(ll &x,ll y) {y%=mod;x%=mod;x+=y;x%=mod;}
ll mul(ll x,ll y) {x%=mod;y%=mod;return x*y%mod;}
void dfs(int u,int f) {
    sz[u]=1;
    dp[u][0][0]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        if(v==f) continue;
        dfs(v,u);
        for(int j=sz[u];j>=0;j--) {
            for(int k=sz[v];k>=0;k--) {
                if(k>=1) {
                    add(dp[u][j+k][0],mul(dp[u][j][0],dp[v][k][0]+dp[v][k][1]+dp[v][k][2]+dp[v][k][3]));
                    add(dp[u][j+k][1],mul(dp[u][j][1],dp[v][k][0]+dp[v][k][1]+dp[v][k][2]+dp[v][k][3]));
                    add(dp[u][j+k][2],mul(dp[u][j][2],dp[v][k][0]+dp[v][k][1]+dp[v][k][2]+dp[v][k][3]));
                    add(dp[u][j+k][3],mul(dp[u][j][3],dp[v][k][0]+dp[v][k][1]+dp[v][k][2]+dp[v][k][3]));
                }
                if(c[i]==-1) {
                    add(dp[u][j+k+1][1],mul(dp[u][j][0],dp[v][k][0]+dp[v][k][1]));
                    add(dp[u][j+k+1][3],mul(dp[u][j][2],dp[v][k][0]+dp[v][k][1]));
                }else if(c[i]==1) {
                    add(dp[u][j+k+1][2],mul(dp[u][j][0],dp[v][k][0]+dp[v][k][2]));
                    add(dp[u][j+k+1][3],mul(dp[u][j][1],dp[v][k][0]+dp[v][k][2]));
                }
            }
        }
        sz[u]+=sz[v];
    }
}
int main() {
    red(n);
    for(int i=2;i<=n;i++) fac[i]=fac[i-1]*(ll)i%mod;
    for(int i=1,u,v;i<n;i++) {
        red(u);red(v);
        adde(u,v,1);adde(v,u,-1);
        // v!=u+1  u!=v-1
    }
    dfs(1,0);
    ll ans=0;
    for(int i=0;i<=n-1;i++) {
        ans+=(((i&1)?-1ll:1ll)*((dp[1][i][0]+dp[1][i][1]+dp[1][i][2]+dp[1][i][3])%mod)*fac[n-i])%mod;
        ans=(ans%mod+mod)%mod;
    }
    printf("%lld\n",ans);
}
```

