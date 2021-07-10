---
title: '数论基础'
date: 2020-02-08 21:07:59
tags: [数论    ]
categories:
- 📚算法笔记
---

📕NOIP数论笔记，素数筛法、欧几里得、扩展欧几里得、欧拉函数、欧拉定理、费马小定理、逆元、求解同余方程（中国剩余定理）

<!--more-->

---

# 1 素数筛法

对于少量的判断一个n是不是素数，我们可以通过暴力 $O(\sqrt{n})$ 的复杂度解决，而当题目需要判断大量素数时便引入了筛法求素数。

## 1.1 朴素的筛法

遇到一个数把它的所有倍数标记掉，依次筛，没有标记的就是素数。复杂度 $O(n\log n)$

```cpp
void init(int n) { 
    int up = (int)sqrt(n) + 1; 
    for (int i = 2; i <= up; i++) 
        for (int j = i * i; j <= n; j += i)
            flag[j] = true; 
    for (int i = 2; i <= n; i++) { 
        if (!flag[i]) p[tot++] = i;
    }
}
```

## 1.2 线性筛素数

$O(n)$ 筛素数，每一个数只被其最小的素因子筛选到

```cpp
void init(int n) {
    for (int i = 2; i <= n; i++) { 
        if (!flag[i]) p[tot++] = i;
        for (int j = 0; j < tot; j++) { 
            if (i * p[j] > n) break;  
            flag[i * p[j]] = true; //p[j]是i * p[j]的最小素因子 
            if (i % p[j] == 0) { // 再往后p[j]就不是i * p[j]的最小素因子了 
                break; 
            } 
        } 
    } 
}
```

# 2 欧几里得

求两数最大公约数

## 2.1 辗转相减

求两数 $a,b$ 的最大公约数 $\gcd(a,b)$，假设 $a>b$ ，则 $\gcd(a,b)=gcd(a-b,b)$ ,直到其中一个为零，另一个就是 $\gcd$。

**证明：**  

- 令 $a=k_1\times g$，令 $b=k_2\times g$；
- 假设 $a,b$ 的最大公约数是 $g$ 则 $\gcd(k_1,k_2)=1$；
- 每次操作令 $a=a-b$，$a=(k_1-k_2)\times g$，$b=k_2 \times g$；
- 可以证明 $k_1-k_2$ 与 $k_2$ 还是互质的（反证法），那么当其中一个 $k$ 减到零，另一个 $k$ 为一便是 $g$ 了。

```cpp
int gcd(int x,int y) {
    if(x<y) return gcd(y,x);
    if(y==0) return x;
    return gcd(x-y,y); 
}
```

## 2.2 辗转相除

显然上面的做法复杂度太高，有许多多余的计算，当 $a=101,b=2$ 时就要减好几次，显然的目的是减到 $a<b$ 为止再交换 $a$ 和 $b$，那么就等同于 $a\bmod b$，故 $\gcd(a,b)=\gcd(a\bmod b,b)$

```cpp
int gcd(int a,int b) {
    return !b ? a : gcd(b,a % b);
}
```

# 3 扩展欧几里得

通过扩展欧几里得解决形如 $ax+by=c$ 的二元一次方程，求解的存在性，特解、通解。

## 3.1 解的存在性

有一个显然的定理：$\gcd(k_1,k_2)=1$ 则方程 $k_1x+k_2y=1$ 一定有解(上面的辗转相减告诉我们对于互质的两数 $k_1,k_2$，辗转相减可以减到 $1$。对于上式，乘上 $\gcd(a,b)$ :    

$$k_1\gcd(a,b)x+k_2\gcd(a,b)y=\gcd(a,b)$$

$$ax+by=\gcd(a,b)$$ 

一定有解。

那么若 $c$ 是 $\gcd(a,b)$ 的倍数,即 $c\bmod \gcd(a,b)=0$ 则有解

## 3.2 特解与通解

对于原方程 $ax+by=c$，我们其实只要求出 

$$ax+by=\gcd(a,b)$$ 

的解再乘上 $\dfrac{c}{gcd(a,b)}$ 就行了。我们考虑缩小规模，假设我们已经求出
$$ (a-b)x_1+by_1=gcd(a-b,b) $$

的解 $x_1,y_1$，那么变化一下式子可得  

$$ ax_1-bx_1+by_1=\gcd(a,b) $$

$$ ax_1+b(y_1-x_1)=\gcd(a,b) $$  

观察一下式子，是不是 $x=x_1,y=y_1-x_1$。
所以我们可以把原问题变成一个规模更小的子问题，求出子问题的解，推出当前的解，故递归求解。
考虑到辗转相减可以用辗转相除替代

$$ bx_1+a\bmod by_1=\gcd(b,a\bmod b) $$

由("/"是取整除)

$$ a\bmod b=a-a/b\times b $$

代入得

$$ bx_1+(a-a/b\times b)y_1=gcd(b,a\bmod b) $$

等价于

$$ ay_1+b(x_1-a/b\times y_1)=gcd(a,b) $$

得

$$ x=y_1,y=x_1-a/b\times y_1 $$

不断缩小规模最后 $a=\gcd(a,b),b=0$ ，得方程 $\gcd(a,b)\times x+0\times y=\gcd(a,b)$，得到一组特解 $x=1,y=0$，将这个方程反推回去可得原方程解。

```cpp
int exgcd(int a,int b,int &x,int &y) {
    if(!b) {
        x=1,y=0;
        return a;
    }else {
        int r=exgcd(b,a%b,y,x)
        y-=(a/b)*x;
        return r;
    }
}
```

函数执行完毕后代码里面的 $x,y$ 就是方程 $ax+by=\gcd(a,b)$ 的一组特解，通解的变化规律就是 $x$ 和 $y$ 的值往相反的方向变化，比如 $x$ 变大一些 $y$ 变小一些，为使得答案不变，设这个变化的最小单位值为 $d_1,d_2$。

$$ a(x+d_1)+b(y-d_2)=\gcd(a,b) $$

$$ a\times d_1=b\times d_2 $$

同除 $\gcd(a,b)$，令 $k_1=a/\gcd(a,b),k2=b/\gcd(a,b)$

$$ k_1\times d_1 = k_2\times d_2$$

$k_1,k_2$ 互质，显然 $d_1=k_2,d_2=k_1$

故通解为

$$ (x+k\times d_1,y-k\times d_2) $$

即

$$ (x+k\times b/\gcd(a,b),y-k\times a/\gcd(a,b)) $$

# 4 欧拉函数

## 4.1 定义

欧拉函数 $\varphi(n)$ 表示小于或者等于 $n$ 的正整数中与 $n$ 互质的数的数目， 例如 $\varphi(8) = 4$, 因为 $1,3,5,7$ 与 $8$ 互质。 

## 4.2 欧拉函数值
特殊的 $\varphi(1)=1$, $\varphi(p)=1$,（$p$ 是素数）；若 $n$ 是素数 $p$ 的 $k$ 次幂
$$ \varphi(n)=\varphi(p^k)=p^k-p^{k-1}=(p-1)\cdot p^{k-1} $$

相当于总数减去p的倍数。利用类似的想法我们考虑求 $\varphi(p_1^{k_1}\cdot p_2^{k_2})$，通过简单容斥我们知道，它应该等于总数减去 $p_1$ 的倍数 $p_2$ 的倍数，再加上 $p_1,p_2$ 的倍数

$$
\begin{aligned}
\varphi(p_1^{k_1}\cdot p_2^{k_2})&=p_1^{k_1}\cdot p_2^{k_2}-p_1^{k_1-1}\cdot p_2^{k_2}-p_1^{k_1}\cdot p_2^{k_2-1}+p_1^{k_1-1}\cdot p_2^{k_2-1}\\
&=\left(p_1^{k_1}-p_1^{k_1-1}\right)\left(p_2^{k_2}-p_2^{k_2-1}\right)
\end{aligned}
$$

而我们发现
$$ \varphi\left(p_1^{k_1}\right)\cdot \varphi\left(p_2^{k_2}\right) = \left(p_1^{k_1}-p_1^{k_1-1}\right)\left(p_2^{k_2}-p_2^{k_2-1}\right)$$
即 $\varphi\left(p_1^{k_1}\cdot p_2^{k_2}\right)=\varphi\left(p_1^{k_1}\right)\cdot \varphi\left(p_2^{k_2}\right)$，这体现了欧拉函数是积性函数的性质,  

> **积性函数**: 对于两个互质的 $n,m$，若 $f(n\cdot m)=f(n)\cdot f(m)$，称 $f$ 为积性函数

我们将 $n$ 因式分解

$$ n=p_1^{k_1}p_2^{k_2}\dotsb p_r^{k_r}$$

$$ \varphi(n)=\prod_{i=1}^r p_i^{k_i-1}(p_i-1)=n\cdot \prod_{i=1}^r (\frac{p_i-1}{p_i})$$

## 4.3 欧拉函数的性质
1. 欧拉函数是积性函数，当正整数 $m,n$ 互质时，$\varphi(m\cdot n) = \varphi(m)\cdot \varphi(n)$ 
2. 当$n$是奇数时，$\varphi(2\cdot n) = \varphi(n)$
3. $\varphi(n) = \varphi(n/p)\cdot p$, ($p$ 能整除 $n/p$),
4. $\varphi(n) = \varphi(n/p)\cdot (p−1)$, ($p$ 与 $n/p$ 互质，且 $p$ 是质数) 
5. $\sum_{d|n} \varphi(d)=n$ ($n$等于它所有因子的欧拉函数之和)

性质2证明：$\varphi(2)=1$,再利用积性函数
性质3证明：可以利用上一小节最后一个式子来推 
性质4证明：利用积性函数性质和$\varphi(p)=p-1$
性质5证明：设$f(n)=\sum_{d|n}\varphi(d)$，令$n,m$互质
$$ f(nm)=\sum_{d|nm}\varphi(d)=\sum_{d_1|n}\varphi(d_1)\cdot \sum_{d_2|m}\varphi(d_2)=f(n)\cdot f(m) $$
（因为 $n,m$ 互质，那么它们的因子也相互质，故 $nm$ 的因子可以看出 $n$ 的因子乘上 $m$ 的因子）
所有 $f(n)$ 是积性函数

$$ f(p^c)=\sum_{d|p^c} \varphi(d)=\varphi(1)+\varphi(p)+\varphi(p^2)+\dotsc +\varphi(p^c)=p^c$$

因此

$$ f(n)=\prod_{i=1}^r f(p_i^{c_i})=\prod_{i=1}^r p_i^{c_i}=n$$

## 4.4 代码实现

**法一**：利用定义$\varphi(n)=n\cdot \prod\limits _{i=1}^r (\frac{p_i-1}{p_i})$，通过普通筛求
```cpp
void init(int n) {
    varphi[1]=1;
    for(int i=2;i<=n;i++)
        varphi[i]=i;
    for(int i=2;i<=n;i++) {
        if(varphi[i]=i) //素数，没有被筛到故phi没变过
            for(int j=i;j<=n;j+=i)
                varphi[j]=varphi[j]/i*(i-1); //先除再乘防溢出
    }
}
```
**法二**：线性筛递推，利用性质3性质4 ： $\varphi(n) = \varphi(n/p)\cdot p$, ($p$ 能整除 $n/p$) ; $\varphi(n) = \varphi(n/p)\cdot (p−1)$, （ $p$ 与 $n/p$ 互质，且 $p$ 是质数）

```cpp
void init(int n) {
    for(int i=2;i<=n;i++) {
        if(!flag[i]) {
            p[tot++]=i;
            varphi[i]=i-1; //是素数\varphi(p)=p-1
        }
        for(int j=0;j<tot;j++) {
            if(i*p[j]>n) break;
            flag[i*p[j]]=1;
            if(i%p[j]==0) {
                varphi[i*p[j]]=varphi[i]*p[j]; //性质3
                break;
            }
            varphi[i*p[j]]=varphi[i]*(p[j]-1); //性质4
        }
    }
}
```

**法三**：利用性质5  $\sum_{d|n} \varphi(d)=n$

```cpp
void init(int n) {
    for(int i=1;i<=n;i++) varphi[i]=i;
    for(int i=1;i<=n;i++) {
        for(int j=i+i;j<=n;j+=i) {
            varphi[j]-=varphi[i];
        }
    }
}
```

# 5 欧拉定理

欧拉定理的定义是：如果 $a,n$ 为正整数，且 $\gcd(a,n)=1$ 则有：
$$ a^{\varphi(n)}\equiv 1 \pmod{n}$$ 

**证明：**
令集合

$$ S=\left\{ x_1,x_2,\dotsc,x_{\varphi(n)}\right\} $$

表示所有的小于 $n$ 并且与 $n$ 互质的数；再令集合

$$ T=\left\{a\cdot x_1\bmod n,a\cdot x_2\bmod n,...,a\cdot x_{\varphi(n)}\bmod n\right\} $$ 

由于 $\gcd(a,n) = 1,\gcd(x_i,n) = 1$， 所以 $\gcd(a\cdot x_i,n) = 1$，所以 

$$ gcd(a\cdot x_i\bmod n,n) = 1  $$

容易通过反证得出 $T$ 中的元素是两两不相同的，因此集合 $S$ 跟 $T$ 是相同的，所以我们构造出

$$
\begin{aligned}
&\;a^{\varphi(n)}\cdot x_1\cdot x_2\cdot\dotso\cdot x_{\varphi(n)}& \pmod{n} \\
\equiv& \;a\cdot x_1 \cdot a \cdot x_2 \cdot \dotso \cdot a \cdot x_{\varphi(n)} &\pmod{n} \\
\equiv& \;x_1 \cdot x_2 \cdot \dotso \cdot x_{\varphi(n)} &\pmod{n}
\end{aligned}
$$

所有 $a^{\varphi(n)}\equiv 1 \pmod{n}$,欧拉定理得证

## 5.1 一个应用

求最小的 $k$ 满足 $a^k \equiv 1 \pmod{n}$，$k$ 一定是 $\varphi(n)$ 的因子

# 6 费马小定理

费马小定理是欧拉定理的特殊情况，即当 $p$ 为素数时

$$ a^{p-1}\equiv 1\pmod{p} $$

# 7 乘法逆元

当我们要计算 $\dfrac{a}{b} \bmod c$ 时，又因为 $a,b$ 很大，不能用高精保存时，典型的例子就是高精取模。这个时候我们可以用到乘法逆元，将除法转换成乘法并提前取模。

设：

$$ inv\cdot b\equiv 1 \pmod{c} $$

我们称 $inv$ （也可以记 $b^{-1}\pmod{c}$）为 $b$ 对 $c$ 的逆元，就可以将除法转化为乘法，即 $\dfrac{a}{b}\bmod c=(a\cdot inv)\bmod c$

那是否一定存在逆元呢？我们考虑下式

$$ inv\cdot b\equiv 1 \pmod{c} $$

等价于

$$ inv\cdot b+k\cdot c=1 $$

$b,c$ 是已知的，本质上是求上述的二元一次方程的一个正整数解，那么用扩展欧几里得就可以解决，若 $\gcd(b,c)=1$ 则有解，即存在逆元。

```cpp
int inv(int b,int c) {
    int x,y;
    exgcd(b,c,x,y);
    x=(x%c+c)%c; //避免负数
    return x
}
```

特殊的，当 $c$ 是质数时，可利用费马小定理

$$ b^{c-1}\equiv 1\pmod{c} $$

同乘 $inv$

$$ b^{c-2}\equiv inv \pmod{c} $$

所以 $inv=b^{c-2}\bmod c$，用快速幂可以解决

# 8 求解同余方程：中国剩余定理

给你如下同余方程组，求解 $x$

$$
\begin{cases}
x \equiv a_1 \pmod{m_1}\\
x \equiv a_2 \pmod{m_2}\\
\dotsc \\
x \equiv a_n \pmod{m_n}\\
\end{cases}
$$

其中 $m_1,m_2,\dotsc,m_n$ 两两互质，我们设

$$ M=\prod_{i=1}^n m_i $$

令

$$ M_i=M/m_i $$

$s_i$是同余方程
$$ M_i\times s_i\equiv 1\pmod{m_i} $$
的一个解，则同余方程组的解为
$$ x=\sum_{i=1}^n a_i\cdot M_i\cdot s_i $$

**证明**：

由于 $i\ne j,\gcd(m_i,m_j)=1$，则 $\gcd(M_i,m_i)=1$，故必然存在 $s_i$ 为 $M_i$ 对 $m_i$ 的逆元。

考察乘积 $a_is_iM_i$ 可知

$$ a_is_iM_i\equiv a_i\cdot 1\equiv a_i \pmod{m_i} $$

$$\forall j\ne i:a_is_iM_i\equiv 0 \pmod{m_j} $$

所以解 $x=a_1s_1M_1+a_2s_2M_2+\dotsc+a_ns_nM_n$满足
$$ x=a_is_iM_i+\sum_{j≠ i} a_js_jM_j\equiv a_i+\sum_{j≠ i} 0 \equiv a_i \pmod{m_i}$$

说明 $x$ 就是方程的一个解。另外特解可以表示为

$$ x+k\cdot M $$

具体做法就是做 $n$ 次 $\operatorname{exgcd}$ 求出 $s_i$，再求出 $x$

```cpp
int crt(int n, int a[], int m[]) { 
    int mul = 1; 
    for (int i = 0; i < n; i++) { 
        mul = mul * m[i]; 
    } 
    int ret = 0; 
    for (int i = 0; i < n; i++) { 
        int x, y; 
        exgcd(mul / m[i], m[i], x, y); 
        x = (x % m[i] + m[i]) % m[i];
        ret += mul / m[i] * x * a[i]; 
        ret %= mul;
    } 
    return ret;
}
```

# END

Updata 2021-3-27 : 修复了一些 bug，使公式表达更规范
