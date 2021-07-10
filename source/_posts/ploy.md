---
title: '多项式学习笔记'
date: 2021-02-04 22:09:53
tags: [多项式]
categories:
- [📚算法笔记]
---

.
<!--more-->

### 多项式求逆

给定一个多项式 $g(x)$ ，求一个多项式 $f(x)$ 满足：
$$
f(x)\ast g(x)\equiv1 \pmod {x^n}
$$
也记作 $f(x)\equiv g^{-1}(x)\pmod{x^n}$。

令 $f_\ast(x)\equiv g^{-1}(x)\pmod {x^{n/2}}$：
$$
\begin{aligned}
f_\ast(x)&\equiv f(x) &\pmod{x^{n/2}}\\
f_\ast(x)- f(x)&\equiv0 &\pmod{x^{n/2}}\\
\big(f_\ast(x)- f(x)\big)^2&\equiv0 &\pmod{x^{n}}\\
f_\ast^2(x)-2f_\ast(x) f(x)+f^2(x)&\equiv0 &\pmod{x^{n}}\\
f_\ast^2(x)g(x)-2f_\ast(x)+f(x)&\equiv0 &\pmod{x^{n}}\\
\end{aligned}
$$

$$
f(x)\equiv f_\ast(x)\big(2-f_\ast(x)g(x)\big)\pmod{x^n}
$$

倍增即可。
时间复杂度 $O(n\log n)。$

### 多项式开方

给定一个多项式 $g(x)$ ，求一个多项式 $f(x)$ 满足：
$$
f^2(x)\equiv g(x) \pmod {x^n}
$$
令 $f_\ast^2(x)\equiv g(x)\pmod {x^{n/2}}$：
$$
\begin{aligned}
f^2_{\ast}(x)&\equiv g(x) &\pmod{x^{n/2}}\\
f^2_{\ast}(x)-g(x)&\equiv 0 &\pmod{x^{n/2}}\\
\big(f^2_{\ast}(x)-g(x)\big)^2&\equiv 0 &\pmod{x^{n}}\\
\big(f^2_{\ast}(x)+g(x)\big)^2&\equiv 4f_\ast^2(x)g(x) &\pmod{x^{n}}\\
\left(\frac{f^2_{\ast}(x)+g(x)}{2f_\ast(x)}\right)^2&\equiv g(x) &\pmod{x^{n}}\\
\frac{f^2_{\ast}(x)+g(x)}{2f_\ast(x)}&\equiv f(x) &\pmod{x^{n}}\\
\end{aligned}
$$

$$
f(x)\equiv 2^{-1}f_\ast(x)+2^{-1}g(x)f_\ast^{-1}(x)\pmod {x^n}
$$

倍增即可。
时间复杂度 $O(n\log n)$。

### 多项式带余除法

给定一个 $n$ 次多项式  $f(x)$，一个次数为 $m$ 的多项式 $g(x)$ ，求出多项式 $Q(x),R(x)$ 满足：
$$
\deg Q=n-m,\deg R <m
$$

$$
f(x)=Q(x)\ast g(x)+R(x)
$$

即 $Q(x)$ 为商，$R(x)$ 为余数。

定义一种变换（这里的 $R$ 更 $R(x)$ 没关系）：
$$
f^R(x)=f\left(\frac{1}{x}\right)x^{\deg f}
$$
其实就是把系数 `reverse` 一下。
考虑把几个多项式都搞成变换过的形式：
$$
\begin{aligned}
f(x)&=Q(x)\ast g(x)+R(x)\\[1ex]
f\left(\frac{1}{x}\right)&=Q\left(\frac{1}{x}\right)\ast g\left(\frac{1}{x}\right)+R\left(\frac{1}{x}\right)\\
x^n f\left(\frac{1}{x}\right)&=x^{n-m}Q\left(\frac{1}{x}\right)\ast x^mg\left(\frac{1}{x}\right)+x^{n-m+1}x^{m-1}\left(\frac{1}{x}\right)\\[1ex]
f^R(x)&=Q^R(x)\ast g^R(x)+x^{n-m+1}R^R(x)\\
\end{aligned}
$$
放在模 $x^{n-m+1}$ 在 $R(x)$ 就没了，然后 $\deg Q = n-m < n-m+1$ 不会受到影响。
$$
f^R(x)=Q^R(x)\ast g^R(x) \pmod{x^{n-m+1}}
$$
多项式求逆可求 $Q(x)$ ，再求 $R(x)=f(x)-Q(x)\ast g(x)$ 即可。

### 多项式求导/积分

对于多项式 $f(x)$ 。

求导：
$$
f'(x)=\sum_{i=0} [x^i]f(x) \cdot ix^{i-1}
$$
$$
[x^n]g(x)=(n+1)[x^{n+1}]f(x)
$$

积分：
$$
\int f(x)\,\mathrm{d}x = C+\sum_{i=0}[x^i]f(x)\cdot \frac{1}{i}x^{i+1}
$$
$$
[x^n]g(x)=\frac{1}{n+1}[x^{n-1}]f(x)
$$

复杂度都是 $O(n)$ 的。

### 多项式牛顿迭代

给定一个多项式 $g(x)$ ，求模 $x^n$ 意义下的 $f(x)$ 满足：
$$
g\big(f(x)\big)\equiv 0\pmod{x^n}
$$
结论：
$$
f(x)\equiv f_\ast(x)-\frac{g\big(f_\ast(x)\big)}{g'\big(f_\ast(x)\big)}\pmod{x^n}
$$
$f_\ast (x)$ 是已经找到的在模 $x^{n/2}$ 意义下的解，上式是一个倍增。

干啥的，方便推导一些东西。

#### 求逆

$$
f(x)^{-1}\equiv h(x)\pmod{x^n}
$$

设 $g\big(f(x)\big)=f^{-1}(x)-h(x)$ ，有方程 $g\big(f(x)\big)\equiv 0 \pmod{x^n}$ 。牛顿迭代一下：
$$
\begin{aligned}
f(x)&\equiv f_\ast (x) -\frac{f_\ast^{-1}(x)-h(x)}{-f_\ast^{-2}(x)}&\pmod{x^n}\\
f(x)&\equiv 2f_\ast(x)-f_\ast^2(x)h(x) &\pmod{x^n}
\end{aligned}
$$

#### 开方

$$
f^2(x)\equiv h(x)\pmod{x^n}
$$

设 $g\big(f(x)\big)=f^{2}(x)-h(x)$ ，有方程 $g\big(f(x)\big)\equiv 0 \pmod{x^n}$ 。牛顿迭代一下：
$$
\begin{aligned}
f(x)&\equiv f_\ast (x)-\frac{f_\ast^2(x)-h(x)}{2 f_\ast(x)} &\pmod{x^n}\\
f(x)&\equiv 2^{-1}f_\ast(x)+2^{-1}h(x)f_\ast^{-1}(x)&\pmod {x^n}\\
\end{aligned}
$$

#### EXP

$$
\ln f(x)\equiv h(x) \pmod{x^n}
$$

设 $g\big(f(x)\big)=\ln f(x)-h(x)$ ，有方程 $g\big(f(x)\big)\equiv 0 \pmod{x^n}$ 。牛顿迭代一下：
$$
\begin{aligned}
f(x)&\equiv f_\ast(x)-\frac{\ln f_\ast(x)-h(x)}{f^{-1}_\ast(x)} \pmod{x^n}\\
f(x)&\equiv f_\ast(x)\big(1-\ln f_\ast (x)+h(x)\big) \pmod{x^n}\\
\end{aligned}
$$

---

以上推倒的式子复杂度都为：
$$
T(n)=T\left(\frac{n}{2}\right)+O(n\log n)=O(n\log n)
$$
话说这里推导的求逆和开方和之前的是一样的，但显然用牛顿迭代推导更方便。

### 多项式对数函数

给定一个多项式 $g(x)$ ，求一个多项式 $f(x)$ 满足：
$$
f(x)\equiv \ln g(x) \pmod{x^n}
$$
具体的由麦克劳林级数定义：
$$
\ln f(x)=-\sum_{i=1}^{+\infty} \frac{\big(1-f(x)\big)^i}{i}
$$
若 $\ln g(x)$ 存在需满足 $[x^0]g(x)=1$
有 $\ln' x=\frac{1}{x}$，两边同时求导，注意链式法则。再积分就完事了。
$$
\begin{aligned}
f(x)&\equiv \ln g(x) &\pmod{x^n}\\
f'(x)&\equiv \frac{g'(x)}{g(x)} &\pmod{x^n}\\
f(x)&\equiv \int \frac{g'(x)}{g(x)}\;\mathrm{d} x &\pmod{x^n}\\
\end{aligned}
$$
一次求导，一次求逆，一次乘法，一次积分完事了。
复杂度 $O(n \log n)$ 。

### 多项式指数函数

给定一个多项式 $g(x)$ ，求一个多项式 $f(x)$ 满足：
$$
f(x)\equiv \exp g(x) \pmod{x^n}
$$
具体的由麦克劳林级数定义：
$$
\exp f(x)=\sum_{i=0}^{+\infty} \frac{f^i(x)}{i!}
$$
若 $\exp g(x)$ （即 $e^{g(x)}$）存在需满足 $[x^0]g(x)=0$。

注：$\exp' x= \exp x$

通过牛顿迭代给出了一种 $O(n\log n)$ 的倍增做法: [EXP](####EXP) 。再来考虑一下普通做法：
$$
\begin{aligned}
f(x)&\equiv \exp g(x) &\pmod{x^n}\\
f'(x)&\equiv f(x)g'(x)&\pmod{x^n}
\end{aligned}
$$

提取系数：
$$
\begin{aligned}
[x^n]f'(x)&=\sum_{i=0}^n[x^{n-i}]f(x)\cdot [x^{i}]g'(x)\\
(n+1)[x^{n+1}]f(x)&=\sum_{i=0}^n [x^{n-i}]f(x)\cdot (i+1)[x^{i+1}]g(x)\\
[x^n]f(x)&=\frac{1}{n}\sum_{i=1}^{n} [x^{n-i}]f(x)\cdot i[x^{i}]g(x)
\end{aligned}
$$

注：$i=1$ 枚举因为 $[x^0]g(x)=0$

分治 FFT ，复杂度 $O(n\log^2 n)$ ，~~实际上常数小~~，比倍增的快。

### 多项式快速幂

给定一个多项式 $g(x)$ ，求一个多项式 $f(x)$ 满足：
$$
f(x)\equiv  g^k(x) \pmod{x^n}
$$
如果只是 niave 多项式乘法+快速幂，复杂度 $O(n\log n \log k)$。

显然有更好的做法：
$$
f(x)=g^k(x)=e^{(\ln g(x)) k}
$$
会 EXP,Ln 便完事了，复杂度 $O(n\log n)$ ，但需要满足 $[x^0]g(x)=1$ 。

注意复杂度与 $k$ 无关，$k$ 只要 $\bmod 998244353$ 即可。（$k$ 跟 $\ln g(x)$ 是数乘）

