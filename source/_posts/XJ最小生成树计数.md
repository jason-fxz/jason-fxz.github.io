---
title: 'XJ - 最小生成树计数'
date: 2020-11-22 10:28:57
tags: [XJ,动态规划]
categories: 题解
hidden: true
---

[XJ-最小生成树计数](https://dev.xjoi.net/contest/1586/problem/3)
<!--more-->

## 题目描述
![D3XSud.png](./D3XSud.png)
![D3OxjH.png](./D3OxjH.png)
![D3XpDA.png](./D3XpDA.png)

## 题解

官方题解：
![D3XMEq.md.png](./D3XME.png)


思路上讲的很清楚了，这里就讲一下实现上的问题。
我们可以预处理 $S$ ，其实就是 $n$ 的拆分数（我们只要知道当前有**几个联通快**和**它们的大小**即可，并不需要知道具体是什么），$n=40$ 时不到 $37500$ ，用一串 `vector` 记录，在暴力预处理出 $S$ 之间的转移（建一个“转移图” $G$），（方便卡常），之后在这上面 DP

枚举放第几条 树边 ，并计算它之前放非树边的方案数。在 $G$ 上完成转移。

详见代码

## CODE

```cpp
// #pragma comment(linker,"/stack:200000000")
// #pragma GCC optimize("Ofast,no-stack-protector")
// #pragma GCC target("sse,sse2,sse3,ssse3,sse4,popcnt,abm,mmx,avx,tune=native")
#include <bits/stdc++.h>
#define re register
#define see(a) cerr<<#a<<" = "<<a
#define seeln(a) cerr<<#a<<" = "<<a<<endl
using namespace std;
typedef vector<int> vi;
typedef long long ll;
const int N = 45;
const int M = 37500;
const int MAX = 10000000;
const int mod = 1e9+7;
  
int head[M],nxt[MAX],pnt[MAX],wth[MAX],E;
int a[N],n,nn,idcnt,c,C[M];// C 记录一种情况所有 联通快的最大边数之和
ll fac[N*N]={1,1},invfac[N*N]={1,1};
vi now,F[M]; // F 是预处理的各联通快大小
map<vi,int> mp;
ll dp[N][M];
  
ll fpow(ll a,ll b) {ll r=1;for(;b;b>>=1,a=(a*a)%mod) if(b&1) r=(r*a)%mod;return r;}
void DFS(int tot,int st) {
    if(tot==n) {
        mp[now]=++idcnt;
        F[idcnt]=now;
        C[idcnt]=c;
        return ;
    }
    for(int i=st;i+tot<=n;i++) {
        now.push_back(i);
        c+=i*(i-1)/2;
        DFS(tot+i,i);
        c-=i*(i-1)/2;
        now.pop_back();
    }
}
ll A(int n,int m) {
    // n choose m
    if(n<0||m<0) return 0;
    if(n<m) return 0;
    return fac[n]*invfac[n-m]%mod;
}
void adde(int u,int v,int w) {E++;pnt[E]=v;nxt[E]=head[u];wth[E]=w;head[u]=E;}
void init() {
    // 预处理
    nn=n*(n-1)/2;
    for(int i=2;i<=nn;i++) fac[i]=fac[i-1]*i%mod,invfac[i]=fpow(fac[i],mod-2);
    DFS(0,1);
    vi tmp;
    for(re int i=1;i<=idcnt;++i) {
        for(re int p=0;p<F[i].size();++p) {
            for(re int q=p+1;q<F[i].size();++q) {
                int newnum=F[i][p]+F[i][q];
                tmp.clear();
                bool needins=1;
                for(int r=0;r<F[i].size();r++) {
                    if(p==r||q==r) continue;
                    if(needins&&F[i][r]>=newnum) needins=0,tmp.push_back(newnum);
                    tmp.push_back(F[i][r]);
                }
                if(needins) tmp.push_back(newnum);
                int v=mp[tmp];
                // 预处理放树边的转移贡献，即将两个联通块合并的方案数，就等于两个联通快的点数相乘
                adde(i,v,F[i][p]*F[i][q]%mod);
            }
        }
    }
}
void add(ll &x,ll y) {x%=mod;y%=mod;x+=y;x%=mod;}
void mul(ll &x,ll y) {x%=mod;y%=mod;x*=y;x%=mod;}
int main() {
    scanf("%d",&n);
    init();
    for(int i=1;i<n;i++) scanf("%d",&a[i]);
    sort(a+1,a+n);
    dp[0][1]=1;
    for(re int i=0;i<n;++i) { // i 枚举第几条树边
        for(re int j=1;j<=idcnt;++j) {
            // j 枚举 联通情况
            ll now=dp[i][j]*A(C[j]-a[i],a[i+1]-a[i]-1)%mod;// 放非树边，贡献就是 C[j]-a[i] 个不合并联通块的边，选 a[i+1]-a[i]-1（第i-1和i条树边 之间的非树边个数） 的排列
            if(now==0) continue;
            for(re int k=head[j];k;k=nxt[k]) {
                // k 枚举转移边，（合并哪两个联通快）
                int v=pnt[k],w=wth[k]%mod;
                add(dp[i+1][v],w*now); //树边（转移贡献已经预处理过了）
            }
        }
    }
    ll ans=dp[n-1][idcnt];
    mul(ans,fac[nn-a[n-1]]); //最后的非树边
    printf("%lld\n",ans);
    return 0;
}
```