---
title: '概率、期望 解题报告'
date: 2020-08-08 13:03:51
tags: [数学,概率、期望]
categories:
- 题解
---

[https://vjudge.net/contest/387981](https://vjudge.net/contest/387981)
概率、期望基础训练解题报告，部分简单题题解较简略

<!--more-->

## A [『SGU - 495』 Kids and Prizes](https://vjudge.net/problem/SGU-495/origin)

### 题意

有n个奖品放在n个盒子里，有m个小朋友轮流去选择一个盒子，若有奖品则拿走。无论有没有奖品都要将空盒子放回去。问最后总共获得奖品的期望。$n,m\le 1\times10^5$

### 题解

**一开始写了个假算法：**

$f[i][j]$ 表前i个人选了j个礼物的概率，
$f[i][j]+=f[i-1][j]\times (\cfrac{j}{n})$  没选中
$f[i][j]+=f[i-1][j-1]\times (\cfrac{n-j+1}{n})$ 选中
$f[1][1]=1$
$ans=\sum\limits_i^n i\times f[m][i]$

显然复杂度 $O(n^2)$ 过不了

**正解**：

$f[i]$ 表前i个人期望拿到奖品数

$f[i]=f[i-1]+\frac{n-f[i-1]}{n}$

### code

```cpp
#include <cstdio>
#include <iostream>
using namespace std;
const int N = 100010;
typedef double db;
db f[N],ans;
int n,m;
int main() {
    f[1]=1;
    cin>>n>>m;
    for(int i=2;i<=m;i++) {
        f[i]=f[i-1]+((db)n-f[i-1])/(db)(n);
    }
    printf("%.9lf\n",f[m]);
}   
```

---

## B [『ZOJ - 3640』 Help Me Escape](https://vjudge.net/problem/ZOJ-3640)

### 题意

一个人要逃离洞穴，洞穴有 $n$ 条路，他初始有 $f$ 的战斗力，每天随机到第 $i$ 条路上，每条路有难度值 $c_i$ ，若 $f>c_i$ 即可再花 $t_i$ 天逃离，其中 $t_i=\left \lfloor \frac{1+\sqrt{5}}{2} \times c_i^2 \right \rfloor$  ，否则 $f+=c_i$ ， 第二天继续随机到另一条道路上。求这个人期望逃离的天数。

$n\le 100,f\le 10000,c_i\le 10000$

### 题解

$g[f]$ 表示战斗力为f的期望逃脱天数，记忆化搜索，枚举当前状态的n种转移

$i\in [1,n]$

若$f>c[i]$ , $g[f]+=\frac{1}{n}t_i$

否则 $g[f]+=\frac{1}{n} (g[f+c[i]]+1)$

### code

```cpp
#include <cstring>
#include <cstdio>
#include <cmath>
using namespace std;
typedef double db;
const int N = 2000000;
const db gs = (1.0+sqrt(5))/2.0;
db g[N];
int n,c[N],ff;
db gett(int p) {
    return floor(gs*c[p]*c[p]);
}
db dfs(int f) {
    if(g[f]>0) return g[f];
    db ret=0;
    for(int i=1;i<=n;i++) {
        if(f>c[i]) ret+=gett(i)/(db)n;
        else ret+=(dfs(f+c[i])+1.0)/(db)n;
    }
    return g[f]=ret;
}
void init() {
    memset(g,0xcf,sizeof(g));
}
int main() {
    while(scanf("%d%d",&n,&ff)!=EOF) {
        for(int i=1;i<=n;i++) scanf("%d",&c[i]);
        init();
        db ans=dfs(ff);
        printf("%.3lf\n",ans);
    }
}
```

---

## C [『POJ - 2096』 Collecting Bugs](https://vjudge.net/problem/POJ-2096)

### 题意

有n种bug，s个系统，一个人一天等概率会找到某个系统的某个bug，求每个系统都被找到至少一种bug且每种bug都被至少找到一次的期望天数。

$n,s\le 1000$

### 题解

考虑倒推

$f[i][j]$ 表示已经找到i个不同系统j个不同bug的还需要的期望天数

$f[s][n]=0\quad ，f[0][0]$ 是答案

$ f[i][j]=f[i+1][j]\times \frac{(s-i)\times j}{ns}+f[i][j+1]\times \frac{i\times (n-j)}{ns}+f[i+1][j+1]\times \frac{(s-i)\times (n-j)}{ns}+f[i][j]\times \frac{i\times j}{ns}+1$

$\left(1-\frac{i\times j}{ns}\right)\times f[i][j] =f[i+1][j]\times \frac{(s-i)\times j}{ns}+f[i][j+1]\times \frac{i\times (n-j)}{ns}+f[i+1][j+1]\times \frac{(s-i)\times (n-j)}{ns}+1$

### code

```cpp
#include <cstdio>
using namespace std;
typedef double db;
const int N = 1005;
db f[N][N];
int n,s;
int main() {
    scanf("%d%d",&n,&s);
    for(int i=s;i>=0;i--) {
        for(int j=n;j>=0;j--) {
            if(i==s&&j==n) continue;
            f[i][j]+=f[i+1][j]* ((s-i)*j)/(n*s);
            f[i][j]+=f[i][j+1]* (i*(n-j))/(n*s);
            f[i][j]+=f[i+1][j+1]* ((s-i)*(n-j))/(n*s);
            f[i][j]+=1.0;
            f[i][j]/=(1.0-(db)(i*j)/(db)(n*s));
            // printf("f[%d][%d]=%.4lf\n",i,j,f[i][j]);
        }
    }
    printf("%.4lf\n",f[0][0]);
}
```

---

## D [『CF - 540D』 Bad Luck Island](https://vjudge.net/problem/CodeForces-540D)

### 题意

在一个岛上有三种生物，r个石头，s个剪刀，p个布。每天会随机一对生物相遇，若其不相同，则石头杀剪刀，剪刀杀布，布杀石头。求最终每种物种成为该岛上唯一一个物种的概率。

$r,s,p \le 100$

### 题解

$f[i][j][k]$ 表当前状态为 i 个石头,j个剪刀, k个布的概率

$tot=i\times j+i\times k+j\times k$
$f[i-1][j][k]+=f[i][j][k]\times \cfrac{i\times k}{tot}$
$f[i][j-1][k]+=f[i][j][k]\times \cfrac{i\times j}{tot}$
$f[i][j][k-1]+=f[i][j][k]\times \cfrac{j\times k}{tot}$

$f[r][s][p]=1 \quad ans_r=\sum_{i=1}^r f[i][0][0] \quad ans_s=\sum_{i=1}^s f[0][i][0] \quad ans_p=\sum_{i=1}^p f[0][0][i]$

### code

```cpp
#include <cstdio>
using namespace std;
typedef double db;
const int N = 105;
db f[N][N][N],pr,ps,pp;
int r,s,p;
bool check(int a,int b,int c) {
    if(a==0&&b==0) {
        pp+=f[a][b][c];return 1;
    }else if(a==0&&c==0) {
        ps+=f[a][b][c];return 1;
    }else if(b==0&&c==0) {
        pr+=f[a][b][c];return 1;
    }
    return 0;
}
int main() {
    scanf("%d%d%d",&r,&s,&p);
    f[r][s][p]=1;
    for(int i=r;i>=0;i--) {
        for(int j=s;j>=0;j--) {
            for(int k=p;k>=0;k--)if(i+j+k) {
                int tot=i*k+i*j+j*k;
                if(check(i,j,k)) continue;
                if(i-1>=0) f[i-1][j][k]+=f[i][j][k]*(db)(i*k)/(db)(tot);
                if(j-1>=0) f[i][j-1][k]+=f[i][j][k]*(db)(i*j)/(db)(tot);
                if(k-1>=0) f[i][j][k-1]+=f[i][j][k]*(db)(j*k)/(db)(tot);
            }
        }
    }
    printf("%.9lf %.9lf %.9lf\n",pr,ps,pp);
}
```

---

## E [『CF - 601C』 Kleofáš and the n-thlon](https://vjudge.net/problem/CodeForces-601C)

### 题意

m个人参加 n 项比赛。 每场比赛有一个分值，在[1,m]之间，且同一场比赛每个人得分两两不同。一个人的总分为每场比赛得分之和。

有一个人，给出他每场比赛的得分 $c_i$ ，求这个人总分排名的期望值。(排名定义为总分严格小于他的人数+1)

$n\le 100, m\le 1000$

### 题解

可将题意转化为求 总分<自己分数 的期望人数+1。因为除了这个人（后文称A）其他人的期望都是一样的，我们只要求出任意一人得 j 分的概率$p_j$，答案为
$$
(m-1)\sum\limits_{j=1}^{tot-1} p_j \;+1
$$

$\sum\limits_{j=1}^{tot} p_j$  为任意一人排在A前面的概率（tot为A的总分），期望人数即为 $(m-1)\sum\limits_{j=1}^{tot-1} p_j$ ，再加$1$就是期望排名

考虑如何求 $p_j$ ，设 $dp_{i,j}$ 表示在前 i 场中得 j 分的概率，显然：
$$
dp_{i,j}=\sum_{k=1,k\not=x_i}^{\min(j,m)} dp_{i-1,j-k}\times\frac{1}{m-1}
$$
暴力枚举 $O(n^2m^2)$ ，可以前缀和优化一下 ，设 $s_{i,j}=\sum_{k=0}^{j} dp_{i,k}$ ，然后：
$$
dp_{i,j}=\frac{1}{m-1}\left( s_{i-1,R}-s_{i-1,L-1}-dp_{i-1,j-x_i}[L\le j-x_i\le R]\right)\\
L=\max(j-m,i-1),R=\min(j-1,(i-1)\times m)
$$
复杂度 $O(n^2m)$

### code

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;
const int N = 105;
const int M = 1005;
typedef double db;
db dp[N][M*N],s[N][M*N];
int n,m,x[N],tot;
int main() {
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++) scanf("%d",&x[i]),tot+=x[i];
    dp[0][0]=1;
    for(int i=0;i<=tot;i++) s[0][i]=1;
    for(int i=1;i<=n;i++) {
        for(int j=i;j<=tot;j++) {
            int R=min(j-1,(i-1)*m),L=max(j-m,i-1);
            dp[i][j]+=s[i-1][R];
            if(L>0) dp[i][j]-=s[i-1][L-1];
            if(L<=j-x[i]&&j-x[i]<=R) dp[i][j]-=dp[i-1][j-x[i]];
            dp[i][j]/=(db)(m-1);
            s[i][j]=s[i][j-1]+dp[i][j];
        }
    }
    db ans=0;
    for(int i=1;i<tot;i++) ans+=dp[n][i];
    ans=ans*(db)(m-1)+1.0;
    printf("%.9lf\n",ans);
}
```

---

## F [『CF - 148D』 Bag of mice](https://vjudge.net/problem/CodeForces-148D)

### 题意

有一袋老鼠，w只白的，b只黑的，公主和龙轮流抽（公主先，抽完不放回），谁先抽到白鼠谁赢（每人抽到则龙赢），龙每次抽完一只老鼠还会随机吓跑一只老鼠（即袋中随机消失一只老鼠），求公主获胜的概率。

$0\le w,b\le 1000$

### 题解

$f[i][j]$ 游戏进行到i个白鼠，j个黑鼠的概率
$(w+b-i-j)\bmod 3=0$ 是公主选，否则龙选

公主选:
$ans+=f[i][j]\times \cfrac{i}{i+j}$
$f[i][j-1]+=f[i][j]\times \cfrac{j}{i+j}$

龙选:
$f[i][j-2]+=f[i][j]\times \cfrac{j}{i+j}\times\cfrac{j-1}{i+j-1}$
$f[i-1][j-1]+=f[i][j]\times\cfrac{j}{i+j}\times\cfrac{i}{i+j-1}$

$f[w][b]=1$

### code

```cpp
#include <cstdio>
using namespace std;
typedef double db;
const int N = 1005;
db f[N][N],ans;
int w,b;
int main() {
    scanf("%d%d",&w,&b);
    f[w][b]=1;
    if(w==0&&b==0) {
        printf("0.000000000\n");
        return 0;
    }
    for(int i=w;i>=0;i--) {
        for(int j=b;j>=0;j--) {
            if(i+j==0) continue;
            if((w+b-i-j)%3==0) {
                ans+=f[i][j]*(db)i/(db)(i+j);
                if(j-1>=0) f[i][j-1]+=f[i][j]*(db)j/(db)(i+j);
            }
            else {
                if(j-2>=0) f[i][j-2]+=f[i][j]*(db)j/(db)(i+j)*(db)(j-1)/(db)(i+j-1);
                if(i-1>=0&&j-1>=0) f[i-1][j-1]+=f[i][j]*(db)j/(db)(i+j)*(db)i/(db)(i+j-1);
            }
        }
    }
    printf("%.9lf\n",ans);
}
```

---

## G [『ZOJ - 3329』 One Person Game](https://vjudge.net/problem/ZOJ-3329)

### 题意

给你三个色子，分别能掷出$[1,K_1] ,[1,K_2],[1,K_3]$ 的点数，你有一个计数器，初始为0，每次掷色子，若三个色子点数正好分别为$a,b,c$ 则计数器清零，否则计数器加上当前三个色子点数之和，若计数器大于n则游戏结束。求游戏结束的期望步数。

$0\le n\le 500, 1<K_1,K_2,K_3\le 6 $

### 题解

$f_i$ 表示当前点数为i，期望还需多少步结束，$p_i$ 表示投出 $i$ 个点的概率(除去回到零的情况) , $p_0$ 表回零的概率 （$p_0$可以暴力预处理出）

$$
\begin{aligned}
f_i & = 0 \quad \tag{if  i > n}\\
f_i & = \sum_k p_kf_{i+k} \;+p_0f_0 +1\tag{if i<=n}
\end{aligned}
$$

$f_0$ 为答案，且在算每个 $f_i$ 时都被算到，$f_0$ 为定值，我们在推 $f_i$ 时将其表示为 $A_i+B_if_0$ 即可（A,B是能算出来的），最后到 $f_0=A_0+B_0f_0$ 即可求解

### code

```cpp
#include <cstdio>
#include <cstring>
const int N = 510;
typedef double db;
int n,K1,K2,K3,a,b,c,T;
db A[N],B[N],p[N];
void init() {
    memset(A,0,sizeof(A));
    memset(B,0,sizeof(B));
    memset(p,0,sizeof(p));
    for(int i=1;i<=K1;i++) {
        for(int j=1;j<=K2;j++) {
            for(int k=1;k<=K3;k++) {
                if(i==a&&j==b&&k==c) p[0]++;
                else p[i+j+k]+=1.0;
            }
        }
    }
    db P=K1*K2*K3;
    for(int i=0;i<=K1+K2+K3;i++) {
        p[i]/=P;
    }
}
int main() {
    scanf("%d",&T);
    for(int cs=1;cs<=T;cs++) {
        scanf("%d%d%d%d%d%d%d",&n,&K1,&K2,&K3,&a,&b,&c);
        init();
        for(int i=n;i>=0;i--) {
            for(int j=3;j<=K1+K2+K3;j++) {
                A[i]+=A[i+j]*p[j];
                B[i]+=B[i+j]*p[j];
            }
            B[i]+=p[0];
            A[i]+=1.0;
        }
        db ans=A[0]/(1.0-B[0]);
        printf("%.8lf\n",ans);
    }
}
```

---

## H [『CF - 908D』 New Year and Arbitrary Arrangement](https://vjudge.net/problem/CodeForces-908D)

### 题意

给你一个空字符串，你有$\frac{pa}{pa+pb}$的概率往字符串最后面加个a,$\frac{pb}{pa+pb}$的概率往字符串最后面加个b，当**子序列** 'ab' 的个数超过k时结束。求期望出现子序列 'ab' 的**个数**。

$ 1\le k\le 1000 , 1\le pa,pb\le 1000000$

### 题解

$f_{i,j}$ 表示，序列中有i个a，j对 'ab' 期望结束时 'ab' 的对数。 $A=\frac{pa}{pa+pb},B=\frac{pb}{pa+pb}$

初始状态一种，而结束状态无穷种，故考虑倒推：

$$
f_{i,j}=A\times f_{i+1,j}+B\times f_{i,i+j}
$$
$f_{i,j}$ 分别从 在结尾加a、b的状态转移过来

考虑什么时候结束，a的个数可以无穷个，但放一个b即可终止。也就是说当 $i+j\ge k$ 时，在结尾加若干个a（可以一个都不加）再加一个b结束。可得：

$$
f_{i,j}=\sum\limits_{r=0}^{\infty} A^rB\times (j+i+r)
$$
解释：r枚举a的个数，那放r个a一个b的概率为 $A^rB$ ，原本已经有 j 对 'ab' ，考虑最后一个b的贡献，(原本a的个数+新加的a的个数)  即 $i+r$ ，故结束时有$(j+i+r)$ 对 'ab' 。

上式可以化简:

$$
f_{i,j}=i+j+\frac{pa}{pb}
$$

具体证明略过（错位相减）

以上转移可以用记忆化搜索解决，那答案就是 $f_{0,0}$ 吗？
$$
f_{0,0}=A\times f_{1,0}+B\times f_{0,0}
$$
列出 $f_{0,0}$ 的转移方程，显然计算 $f_{0,0}$ 会进入死循环。移项后可以发现实际上 $f_{0,0}=f_{1,0}$ ，那答案只要计算 $f_{1,0}$ 即可。

### code

```cpp
#include <cstdio>
#include <cstring>
using namespace std;
const int N = 1005;
const int mod = 1e9+7;
typedef long long ll;
int pa,pb,k;
ll g[N][N],A,B;
ll pow(ll a,ll b,ll p) {
    ll r=1ll;
    while(b) {
        if(b&1) r=(r*a)%p;
        a=(a*a)%p;
        b>>=1;
    }
    return r;
}
ll inv(ll a) {
    return pow(a,mod-2,mod);
}
ll dp(int i,int j) {
    if(g[i][j]!=-1) return g[i][j];
    if(i+j>=k) return g[i][j]=(i+j+pa*inv(pb))%mod;
    return g[i][j]=(A*dp(i+1,j)+B*dp(i,i+j))%mod;
}
int main() {
    memset(g,-1,sizeof(g));
    scanf("%d%d%d",&k,&pa,&pb);
    ll sum=inv(pa+pb);
    A=(pa*sum)%mod,B=(pb*sum)%mod;
    printf("%lld\n",dp(1,0));
}
```

---

## I [『CF - 696B』 Puzzles](https://vjudge.net/problem/CodeForces-696B)

### 题意

给你一颗n个点的树，求每个点的期望dfn序（在dfs时，访问孩子的顺序随机）

$n\le 10^5$

### 题解

$f[u]$ 为u的期望dfn序，$g[u]$ 表示在进入u子树前，在同一个父亲的其他子树中期望走多久，u的父亲为fa

显然有如下关系：

$f[u]=f[fa]+1+g[u]$

考虑如何计算$g[u]$ ：  显然dfs一个子树会将其遍历完，遍历一个子树v的代价就是$size[v]$，随机遍历fa的子树的顺序就是一个排列，考虑其他子树对u的贡献: 当且仅当其排在u前面才有。~~大胆口糊~~，子树v的期望贡献就是$\frac{1}{2}size[v]$

$g[u]=\frac{1}{2}\sum\limits_{v\in son[fa],v\not=u} size[v]$

### code

```cpp
#include <cstdio>
using namespace std;
const int N = 100010;
int head[N],nxt[N],pnt[N],E;
int fa[N],n,siz[N];
double f[N];
void add(int u,int v) {
    E++;pnt[E]=v;nxt[E]=head[u];head[u]=E;
}
void dfs(int u) {
    siz[u]=1;
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        dfs(v);
        siz[u]+=siz[v];
    }
}
void dfs1(int u) {
    // printf("%d\n",u);
    for(int i=head[u];i;i=nxt[i]) {
        int v=pnt[i];
        f[v]=f[u]+1+(siz[u]-siz[v]-1)/2.0;
        dfs1(v);
    }
}
int main() {
    scanf("%d",&n);
    for(int i=2;i<=n;i++) {
        scanf("%d",&fa[i]);
        add(fa[i],i);
    }
    f[1]=1;
    dfs(1);
    dfs1(1);
    for(int i=1;i<=n;i++) printf("%.1lf ",f[i]); 
    printf("\n");  
}
```
