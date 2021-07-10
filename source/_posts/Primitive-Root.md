---
title: 原根简记
date: 2020-12-29 21:47:21
tags: [数论    ]
categories:
- 📚算法笔记
---


### 定义

**阶**：对于 $a,m\in \mathbf{N^{\ast}},\gcd(a,m)=1$ 若 $n$ 为最小的正整数满足：
$$
a^n\equiv 1\pmod{m}
$$
称 $n$ 为 $a$ 模 $m$ 的阶，记作 $\delta_m (a)$ 。
显然若 $a^n \equiv 1\pmod{m}$ ，则 $\delta_m(a)\mid n$

<!--more-->

**原根**：$g,m\in \mathbf{N^{\ast}}$ 。若 $\gcd(g,m)=1$ 且 $\delta_m(g)=\varphi(m)$ 则称 $g$ 为模 $m$ 的原根。

### 性质 1

设 $m,a,b\in \mathbf{N}^{\ast},\gcd(a,m)=\gcd(b,m)=1$
$$
\delta_m(ab)=\delta_m(a)\delta_m(b)\iff \gcd\left(\delta_m(a),\delta_m(b)\right)=1
$$

### 性质 2

设 $k\in \mathbf{N},a,m\in \mathbf{N^{\ast}},\;\gcd(a,m)=1$

$$
\delta_m(a^k)=\frac{\delta_m(a)}{\gcd\left(\delta_m(a),k\right)}
$$

### 原根存在定理

模 $m$ 有原根，需满足 $m=2,4,p^{\alpha},2p^{\alpha}$ （其中 $p$ 为奇素数，$\alpha$ 为正整数 ）

### 原根判定定理

$\gcd(g,m)=1$ ，设 $p_1,p_2,\dots,p_k$ 为 $\varphi (m)$ 所有不同素因数
$$
g \text{是} m \text{原根}\iff\forall i\in[1,k] ,g^{\frac{\varphi(m)}{p_i}}\not\equiv1\pmod{m}
$$

### 求所有原根

设 $g$ 为 $m$ 的一个原根，则集合 $S=\{g^s\mid 1\le s\le \varphi(m),\gcd(s,\varphi(m))=1\}$ 给出 $m$ 的全部原根。因此，若 $m$ 有原根，则 $m$ 有 $\varphi\left(\varphi(m)\right)$ 个关于模 $m$ 两两互不同余的原根。

>若一个数 $m$ 存在原根，可以证明 $m$ 的最小原根在 $O(m^{0.25})$ 级别。
>求所有的原根时，枚举最小原根花费的时间一般都是可以接受的

### CODE

[luogu-P6091【模板】原根](https://www.luogu.com.cn/problem/P6091)

```cpp
// 原根
#include <bits/stdc++.h>
using namespace std;
const int N = 1000010;
int p[N],tot,phi[N];// p[] 素数，phi[] 欧拉函数
bool v[N],has[N]; // v[] 是否是素数, is[] 是否有原根
int Ts;
typedef long long ll;
ll fpow(ll a,ll b,ll p) {ll r=1;for(;b;b>>=1,a=(a*a)%p) if(b&1) r=(r*a)%p; return r;}
int gcd(int a,int b) {return !b?a:gcd(b,a%b);}
void init(int n) { // 预处理欧拉函数，素数，及那些数有原根
    phi[1]=1;
    for(int i=2;i<=n;++i) {
        if(!v[i]) p[++tot]=i,phi[i]=i-1;
        for(int j=1;j<=tot&&p[j]*i<=n;++j) {
            v[p[j]*i]=1;
            if(i%p[j]==0) {
                phi[i*p[j]]=phi[i]*p[j];
                break;
            }
            phi[i*p[j]]=phi[i]*(p[j]-1);
        }
    }
    has[2]=has[4]=1;
    for(int i=2;i<=tot;++i) for(ll j=p[i];j<=n;j*=p[i]) has[j]=1,(j*2<=n)&&(has[j*2]=1);
}
void devide(int x,vector<int> &ret) { // 分解质因数
    ret.clear();
    for(int i=1;1ll*p[i]*p[i]<=x;++i) if(x%p[i]==0){
        ret.push_back(p[i]);
        while(x%p[i]==0) x/=p[i];
    }
    if(x>1) ret.push_back(x);
}
bool hasroot(int n) { // 判断一个数是否有原根
    if(n==2||n==4) return 1;
    if(n%2==0) n/=2;
    if(n%2==0) return 0;
    for(int i=2;1ll*p[i]*p[i]<=n;++i) if(n%p[i]==0) {
        while(n%p[i]==0) n/=p[i];
        return (n==1);  
    }
    return 1;
}
void Getroot(int m,vector<int> &res) {
    res.clear();
    if(!has[m]) return ;
    vector<int> factor;
    devide(phi[m],factor);
    int g=1,mphi=phi[m];
    while(true) {
        bool fg=1;
        if(gcd(g,m)!=1) fg=0; //i不存在模n的阶
        else for(int i=0;i<(int)factor.size();++i) {
            if(fpow(g,mphi/factor[i],m)==1ll) {fg=0;break;}
        }
        if(fg) break;
        ++g;
    }
    ll pw=1;
    for(int s=1;(int)res.size()<phi[mphi];++s) {
        pw=(pw*g)%m;
        if(gcd(s,mphi)==1) res.push_back(pw);
    }
    sort(res.begin(),res.end());
}
int main() {
    init(1e6);
    scanf("%d",&Ts);
    while(Ts--) {
        int n,d; // 找 n 的所有原根
        scanf("%d%d",&n,&d);
        vector<int> ans;
        Getroot(n,ans);
        printf("%d\n",(int)ans.size());
        for(int i=d-1;i<(int)ans.size();i+=d) printf("%d ",ans[i]);
        printf("\n");
    }
}
```

---

### Reference

[Oi-wiki 原根](https://oi-wiki.org/math/primitive-root/)
