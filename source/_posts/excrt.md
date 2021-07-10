---
title: 'EXCRT '
date: 2020-09-10 22:19:17
tags: [数论    ,CRT]
categories:
- 📚算法笔记
---

名为扩展中国剩余定理，但解法与CRT没啥关系 [关于CRT](https://blog.jasonfan.ml/post/math1/#8-%E6%B1%82%E8%A7%A3%E5%90%8C%E4%BD%99%E6%96%B9%E7%A8%8B%E4%B8%AD%E5%9B%BD%E5%89%A9%E4%BD%99%E5%AE%9A%E7%90%86)

## 简介

考虑求解如下线性同余方程组，不保证 $\forall i,j,\;m_i\perp m_j$

$$
\left\{\;
\begin{matrix}
x & \equiv & a_1 & \pmod{m_1} \\
x & \equiv & a_2 & \pmod{m_2} \\
x & \equiv & a_3 & \pmod{m_3} \\
  &   \vdots     & & \\
x & \equiv & a_n & \pmod{m_n} \\
\end{matrix}
\right.
$$

<!--more-->

CRT在处理线性同余方程组时用的是构造法，构造出解为：

$$
x=M_{1}s_{1}a_{1}+M_{2}s_{2}a_{2}+\cdots+M_{n}s_{n}a_{n}
$$

其中 $M_i=\prod_{j\not=i} m_j\;,\;s_i\equiv M_i^{-1}\pmod{m_i}$ ,（具体证明不再解释）可以发现这种做法的前提是求的出 $M_i$ 在模 $m_i$ 意义下的逆元，当 $\forall i,j,\;m_i\perp m_j$ 时，显然 $M_i \perp m_i$ ，故一定有逆元。但此时不保证 $m$ 两两互质，便存在 $M_i \not\perp m_i$ ，则求不出 $M_i$ 在模 $m_i$ 意义下的逆元。

## 推理

那咋办？我们干脆放弃构造的想法，换一种思路，首先看只有两个方程的情况：
$$
\left\{\;
\begin{matrix}
x & \equiv & a_1 &\pmod{m_1} \\
x & \equiv & a_2 &\pmod{m_2} \\
\end{matrix}
\right.
$$
以下写法是等价的：
$$
x=k_1m_1+a_1=k_2m_2+a_2\;\mid \;k_1,k_2\in\Z
$$
进行移项得：
$$
m_1k_1-m_2k_2=a_2-a_1
$$
这里 $a_2-a_1,\;m_1,\;m_2$ 都是已知的，这样的不定方程可以用扩展欧几里得求解，当 $\gcd(m_1,m_2) \nmid (a_2-a_1)$ 时无解。
然后可计算出 $x'=k_1m_1+a_1$ ，此时的 $x'$ 满足 $x'\bmod m_1=a_1,\;x'\bmod m_2=a_2$ ，实际上就可以将原方程组合并成一个新同余式：
$$
x=x' \pmod{\mathrm{lcm}(m_1,m_2)}
$$
当处理同余原方程组时，将其依次合并，最后留下一个方程就是解。在过程中若扩展欧几里无解则原方程组无解。

## 伪代码

```plain
input n,a[ ],m[ ]
for i in range(2,n)
    if (a[i]-a[i-1])%gcd(m[i-1],-m[i])!=0   无解
    用exgcd求解不定方程 (m[i-1])x+(-m[i])y=(a[i]-a[i-1])
    m[i] <- lcm( m[i], m[i-1] )
    a[i] <- ( m[i-1] * x + a[i-1] ) mod m[i]
return a[n]
```

## 几道习题

- [【模板】扩展中国剩余定理（EXCRT）](https://www.luogu.com.cn/problem/P4777)
- [[TJOI2009]猜数字](https://www.luogu.com.cn/problem/P3868)
