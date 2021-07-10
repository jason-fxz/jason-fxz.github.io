---
title: 'XJ - 数数题'
date: 2021-04-08 15:15:05
tags: [生成函数,XJ,概率、期望]
categories: 题解
hidden: true
---


## 题意

给你 $n$ 个整数数 $a_1,a_2,\dots,a_n$ 令 $x_i\in[0,a_i]$ 为等概率随机**实数**，求 $k=1,2,\dots,m$ 时 $\left(\sum\limits_{i=1}^n x_i\right)^k$ 的期望值，对 $998244353$ 取模。

$n,m\le 5000$  **70pts**；$n,m\le 100000$  **100pts**

<!--more-->

## 题解

可以用多项式定理把这玩意展开，其中 $\binom{k}{p_1,p_2,\dots,p_n}=\frac{k!}{p_1!p_2!\dots p_n!}$ 表示多重集组合数。
$$
\left(\sum_{i=1}^n x_i\right)^k=\sum_{p_1+p_2+\dots+p_n=k} \binom{k}{p_1,p_2,\dots,p_n} \prod_{i=1}^n x_{i}^{p_i}
$$
实数不会？先试试整数，即 $x_i\in[0,a_i]\cap \mathbb{Z}$ ，枚举所有情况之和再乘上总方案分之一。
$$
\mathbb{E}=\frac{1}{\prod (a_i+1)}\sum_{p_1+p_2+\dots+p_n=k} \binom{k}{p_1,p_2,\dots,p_n} \prod_{i=1}^n\left(\sum_{x=0}^{a_i}x^{p_i} \right)
$$
那考虑实数呢？ 要枚举到每个 $x_i \in [0,a_i]$ ，那就积分吧，注意 $\prod (a_i+1)$ 改成 $\prod a_i$  
$$
\mathbb{E}=\frac{1}{\prod a_i}\sum_{p_1+p_2+\dots+p_n=k} \binom{k}{p_1,p_2,\dots,p_n} \prod_{i=1}^n\left(\int_{0}^{a_i}x^{p_i}\,{\mathrm{d}}x \right)
$$
这个积分显然 $\int_0^{a} x^p \,\mathrm{d}x=\frac{1}{p+1}a^{p+1}$ ，进一步化简
$$
\mathbb{E}=k!\sum_{p_1+p_2+\dots+p_n=k}  \prod_{i=1}^n\frac{a_i^{p_i}}{(p_i+1)!}
$$
按照套路，搞个生成函数 
$$
F_i(x)=\sum_{j=0}^{+\infty} \frac{a_i^{j}}{(j+1)!} x^j\\
$$

$$
\mathbb{E}=k![x^k]\prod_{i=1}^n F_i(x)
$$

你已经可以 $O(nm\log m)$ 了，松一点说不定 70pts。

~~按照套路~~ ，右边这个求积化乘为加，（我们熟知 $a\times b=\exp (\ln a+ \ln b)$），但直接求 $\ln F_i(x)$ 显然不大行。

----

~~想到这里就没然后了~~ ，后面挺妙的。

----

不如考虑 $\displaystyle F(x)=\sum_{j=0}^{+\infty} \frac{x^j}{(j+1)!}$ ，令 $G(x)=\ln F(x)$ 我们要算：
$$
\exp \left(\sum_{i=1}^n G(a_ix)\right)
$$
$G(x)$ 是一个已知的多项式（$F(x)$ 的 $x^0$ 系数是 $1$，多项式 $\ln F(x)$ 可以求出前 $m$ 项）
$$
[x^k] \left(\sum_{i=1}^n G(a_ix)\right)=[x^k]G(x) \sum_{i=1}^n a_i^k
$$
现在的问题是求 $k\in[1,m],\;\sum_{i=1}^n a_i^k$ ，其实就是求下面这个多项式 $x^k$ 的系数 
$$
\sum_{i=1}^{n} \left(1+a_ix+a_i^2x^2+a_i^3x^3 +\dots \right)=\sum_{i=1}^n\frac{1}{1-a_ix}
$$
分治通分即可。

## CODE

又长又臭（偷懒copy了一下多项式式全家桶的板子）的代码，就上个剪贴板吧

[https://www.zybuluo.com/jason-fan/note/1185093](https://www.zybuluo.com/jason-fan/note/1185093)


