---
title: '推 式 子'
date: 2021-04-28 22:04:51
tags: [数论    ,反演]
categories:
- 题解
---

基础莫反，推式子。

<!--more-->

## 「SPOJ 5971」LCMSUM

**[SPOJ LCMSUM](https://www.luogu.com.cn/problem/SP5971)**

$T$ 组数据，每次求：

$$
\sum_{i=1}^n \operatorname{lcm}(i,n)
$$

$$
T\le 10^5,n\le 10^6
$$

$$
\begin{aligned}
&\sum_{i=1}^n \operatorname{lcm}(i,n)\\
=&\sum_{i=1}^n \frac{i\times n}{\gcd(i,n)}\\
=&\frac{1}{2}\left( \sum_{i=1}^{n-1} \frac{i\times n}{\gcd(i,n)} +\sum_{i=n-1}^{1} \frac{i\times n}{\gcd(i,n)}\right) + n\\
=&\frac{1}{2}\left( \sum_{i=1}^{n-1} \frac{i\times n}{\gcd(i,n)} +\sum_{i=n-1}^{1} \frac{i\times n}{\gcd(n-i,n)}\right)+n \\
=&\frac{1}{2}\sum_{i=1}^{n} \frac{n^2}{\gcd(i,n)} +\frac{n}{2}\\
\end{aligned}
$$

考虑相同的 $\gcd(i,n)$ 一起计算，$\gcd(i,n)=d$ 的 $i$ 有 $\varphi(\frac{n}{d})$ 个。

$$
\begin{aligned}
&\frac{1}{2}\sum_{d\mid n} \frac{n^2\varphi\left(\frac{n}{d}\right)}{d}+\frac{n}{2}\\
=&\frac{n}{2}\sum_{d\mid n} \frac{n}{d}\varphi\left(\frac{n}{d}\right)+\frac{n}{2}\\
=&\frac{n}{2}\left(\sum_{d\mid n} d\cdot \varphi(d)\right) +\frac{n}{2}  
\end{aligned}
$$

---

设：

$$
f(n)=\sum_{d|n} d\cdot\varphi(d)
$$

令

$$
n=p_1^{c_1} \cdot p_2^{c_2}\cdots p_k^{c_k}
$$

考虑 $f(p^c)$，其约数只有 $p^0,p^1,\dots,p^c$

$$
f(p^c)=\sum_{i=0}^c p^i\cdot\varphi(p^i)=\sum_{i=0}^c p^{2i-1}\cdot(p-1)
$$

考虑 $f(p_1^{c_1}\cdot p_2^{c_2})$，其中 $\gcd(p_1,p_2)=1$

$$
\begin{aligned}
f(p_1^{c_1}\cdot p_2^{c_2})=&\sum_{d_1\mid p_1^{c_1}} \sum_{d_2\mid p_2^{c_2}} d_1\cdot d_2\cdot \varphi(d_1\cdot d_2)\\
=&\sum_{d_1\mid p_1^{c_1}} \sum_{d_2\mid p_2^{c_2}} d_1\varphi(d_1) \cdot d_2\varphi(d_2)\\
=&\sum_{d_1\mid p_1^{c_1}}  d_1\varphi(d_1) \cdot \sum_{d_2\mid p_2^{c_2}}d_2\varphi(d_2)\\
=&f(p_1^{c_1})\cdot f(p_2^{c_2})
\end{aligned}
$$

考虑线性筛筛 $f(n)$ 。首先有 $f(p^c)=f(p^{c-1})+ p^{2c-1}(p-1)$，再考虑由 $f(i),f(p)$ 推 $f(i\cdot p)$。

若 $\gcd(i,p)=1$：

$$
f(i\cdot p) = f(i)\cdot f(p)
$$

若 $p\mid i$，令 $i=p^{c-1} \times i'$，注意到线性筛中 $p$ 是 $p\times i$ 最小的素数，再额外维护一个 $c_n$ 表是 $n$ 中最小的素数的次数即可。

---

## Crash的数字表格

[Crash的数字表格](https://www.luogu.com.cn/problem/P1829)

求：

$$
\sum_{i=1}^n\sum_{j=1}^m \operatorname{lcm}(i,j) \pmod{20101009}
$$

$$
n,m\le 10^7
$$

$$
\begin{aligned}
=&\sum_{i=1}^n\sum_{j=1}^m \frac{i\times j}{\gcd(i,j)}\\
=&\sum_{i=1}^n\sum_{j=1}^m \sum_{d|i,d|j,\gcd(\frac{i}{d},\frac{j}{d})=1} \frac{i\times j}{d}\\
=&\sum_{d=1}^n \sum_{i=1}^{\lfloor n/d\rfloor}\sum_{j=1}^{\lfloor m/d\rfloor} [\gcd(i,j)=1]\frac{id\times jd}{d}\\
=&\sum_{d=1}^n d \sum_{i=1}^{\lfloor n/d\rfloor}\sum_{j=1}^{\lfloor m/d\rfloor} [\gcd(i,j)=1] \cdot i \cdot j\\
\end{aligned}
$$

令 $f(n,m)=\sum_{i=1}^{n}\sum_{j=1}^{m} [\gcd(i,j)=1] \cdot i \cdot j$

$$
\begin{aligned}
f(n,m)=&\sum_{i=1}^{n}\sum_{j=1}^{m} \epsilon(\gcd(i,j)) \cdot i \cdot j\\
=&\sum_{i=1}^n\sum_{j=1}^m \sum_{d\mid \gcd(i,j)} \mu(d)\cdot i \cdot j\\
=&\sum_{i=1}^n\sum_{j=1}^m \sum_{d\mid i,d\mid j} \mu(d)\cdot i \cdot j\\
=&\sum_{d=1}^{\min(n,m)}\mu(d)\cdot d^2 \sum_{i=1}^{\lfloor n/d\rfloor}\sum_{j=1}^{\lfloor m/d\rfloor} i\cdot j\\
\end{aligned}
$$

令 $g(x)=\mu(x)\cdot x^2$，线性筛可求；令 $h(n,m)=\sum_{i=1}^n\sum_{j=1}^m i\cdot j=\frac{ij(1+i)(1+j)}{4}$

$$
f(n,m)=\sum_{d=1}^{\min(n,m)} g(d) \cdot h(\left\lfloor \tfrac{n}{d}\right\rfloor,\left\lfloor \tfrac{m}{d}\right\rfloor)
$$

预处理 $g(n)$ 前缀和，数论分块 $f(n,m)$ 在 $O(\sqrt{n})$ 内可求。

原式：

$$
\sum_{d=1}^{n} d \cdot f(\left\lfloor \tfrac{n}{d}\right\rfloor,\left\lfloor \tfrac{m}{d}\right\rfloor)
$$

同样数论分块，算 $O(\sqrt{n})$ 次 $f$，总复杂度 $O(n)$。

---

## 『SDOI2015』约数个数和

$T$ 祖数据，求

$$
\sum_{i=1}^n \sum_{j=1}^m \sigma_0 (ij)
$$

$$
T,n,m\le 50000
$$
（$\sigma_0(x)$ 表 $x$ 约数个数）

有性质：

$$
\sigma_0(ij)=\sum_{x\mid i}\sum_{y\mid j} [\gcd(x,y)=1]
$$

那么：

$$
\begin{aligned}
\sigma_0(nm)=&\sum_{x\mid n}\sum_{y\mid m} [\gcd(x,y)=1]\\
=&\sum_{x\mid n}\sum_{y\mid m} \sum_{d\mid \gcd(x,y)} \mu(d)\\
=&\sum_{x\mid n}\sum_{y\mid m} \sum_{d\mid x,d\mid y} \mu(d)\\
=&\sum_{d=1}^{n} \mu(d) \sum_{d\mid x,x\mid n} \sum_{d\mid y,y\mid m} 1\\
=&\sum_{d|\gcd(n,m)} \mu(d)\cdot \sigma_0\left(\frac{n}{d}\right) \cdot \sigma_0\left(\frac{m}{d}\right)
\end{aligned}
$$

回到原式：

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m \sigma_0(ij)=&\sum_{i=1}^n\sum_{j=1}^m\sum_{d|i,d|j} \mu(d)\cdot \sigma_0\left(\frac{i}{d}\right) \cdot \sigma_0\left(\frac{j}{d}\right)
\\
=&\sum_{d=1}^{\min(n,m)} \mu(d)\sum_{i=1}^{\left\lfloor n/d\right\rfloor}\sum_{j=1}^{\left\lfloor m/d\right\rfloor} \sigma_0(i)\sigma_0(j)\\
\end{aligned}
$$

令 $s(n)=\sum_{i=1}^n \sigma_0(i)$，那么

$$
\sum_{d=1}^{\min(n,m)} \mu(d) s\left(\left\lfloor \frac{n}{d}\right\rfloor\right) s\left(\left\lfloor \frac{m}{d}\right\rfloor\right)
$$

$s(n)$ 可预处理，每次求答案数论分块即可，总复杂度 $O(n+T\sqrt{n})$

---

## 简单的数学题

[luogu-P3768 简单的数学题](https://www.luogu.com.cn/problem/P3768)

求

$$
\sum_{i=1}^n\sum_{j=1}^n ij\gcd(i,j) \pmod{p}
$$

$$
n\le 10^{10},\,5\times 10^8 \le p \le 1.1\times 10^9,\;p\text{为质数}
$$

用 $\varphi\ast 1=\mathrm{id}$ 反演：

$$
\begin{aligned}
&\sum_{i=1}^n\sum_{j=1}^n ij\gcd(i,j)\\
=&\sum_{i=1}^n\sum_{j=1}^n ij\sum_{d|\gcd(i,j)} \varphi(d)\\
=&\sum_{i=1}^n\sum_{j=1}^n ij\sum_{d|i,d|j} \varphi(d)\\
=&\sum_{d=1}^{n} \varphi(d)d^2 \sum_{i=1}^{\left\lfloor n/d\right\rfloor} \sum_{j=1}^{\left\lfloor n/d\right\rfloor} ij\\
\end{aligned}
$$

令

$$
s(n)=\sum_{i=1}^n\sum_{j=1}^n ij=\frac{n^2(n+1)^2}{4}
$$

$O(1)$ 可求。带回原式：

$$
\sum_{d=1}^{n} \varphi(d)d^2\cdot s(\left\lfloor n/d\right\rfloor)
$$

可以*数论分块* $O(\sqrt{n})$ ，但是需要 $10^{10}$ 范围的 $\varphi(d)d^2$，需要*杜教筛*。

具体的，令 $f(n)=\varphi(n)n^2=(\mathrm{id}^2\varphi)(n)$，$F(n)=\sum_{i=1}^n f(i)$，令 $g(n)=(\mathrm{id}^2)(n)$，$G(n)=\sum_{i=1}^n g(i)$：

$$
h(n)=(f\ast g)(n)=\sum_{d\mid n} d^2\varphi(d)\cdot \frac{n^2}{d^2}= n^2\sum_{d|n} \varphi(d) = n^3
$$

$$
H(n)=\sum_{i=1}^n h(i)=\sum_{i=1}^n i^3=\frac{1}{4}n^2(n+1)^2
$$

$$
G(n)=\sum_{i=1}^n g(i)=\sum_{i=1}^n i^2 =\frac{1}{6}n(n+1)(2n+1)
$$

[杜教筛](https://blog.jasonfan.ml/2020/12/25/%E6%9D%9C%E6%95%99%E7%AD%9B/)：

$$
g(1)F(n) = H(n) - \sum_{d=2}^n g(d) F(\left\lfloor n/d\right\rfloor)
$$
