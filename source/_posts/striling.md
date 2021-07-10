---
title: '斯特林数学习笔记'
date: 2021-02-04 22:01:38
tags: [计数]
categories:
- [📚算法笔记]
---

蒟蒻的斯特林数笔记整理，未完待续。

<!--more-->



## 相关公式

**二项式定理**
$$
(a+b)^n =\sum_{k=0}^n \binom{n}{k} \,a^kb^{n-k}
$$


**二项式反演**
$$
f_n=\sum_{i=0}^n (-1)^i \binom{n}{i} g_i \iff g_n=\sum_{i=0}^n (-1)^i \binom{n}{i} f_i
$$

$$
f_n=\sum_{i=0}^n \binom{n}{i} g_i \iff g_n=\sum_{i=0}^n (-1)^{n-i}\binom{n}{i}f_i
$$



**斯特林数的一些恒等式**
$$
\begin{Bmatrix}n\\m\end{Bmatrix}=\frac{1}{m!}\sum_{k=0}^m (-1)^{m-k} \binom{m}{k}k^n
$$

{% spoiler "推导" %}

$$
\begin{aligned}
x^{n} &= \sum_{k=0}^{n} \begin{Bmatrix}n\\k\end{Bmatrix}x^{\underline{k}}\\
x^{n} &= \sum_{k=0}^{n} \begin{Bmatrix}n\\k\end{Bmatrix}\binom{x}{k}{k!}\\
x^{n} &= \sum_{k=0}^{x}\binom{x}{k} \begin{Bmatrix}n\\k\end{Bmatrix}{k!}\\
\begin{Bmatrix}n\\x\end{Bmatrix} {x!}&=\sum_{k=0}^x(-1)^{x-k} \binom{x}{k} k^n\\
\end{aligned}
$$

{% endspoiler %}


$$
x^n =\sum_{k=0}^n \begin{Bmatrix}n\\k\end{Bmatrix} x^{\underline{k}}=\sum_{k=0}^n (-1)^{n-k}\begin{Bmatrix}n\\k\end{Bmatrix} x^{\overline{k}}
$$

$$
m^{n}=\sum_{k=0}^{m} \begin{Bmatrix}n\\k\end{Bmatrix}m^{\underline{k}}=\sum_{k=0}^{m} \begin{Bmatrix}n\\k\end{Bmatrix}\binom{m}{k} \cdot k!
$$

$$
x^{\underline{n}} =\sum_{k=0}^n \begin{bmatrix}n\\k\end{bmatrix} (-1)^{n-k} x^k
$$

$$
x^{\overline{n}}=\sum_{k=0}^n \begin{bmatrix}n\\k\end{bmatrix} x^k
$$

$$
n!=\sum_{k=0}^n \begin{bmatrix}n\\k\end{bmatrix}
$$

**斯特林反转公式**
$$
\sum_{k=m}^n (-1)^{n-k} \begin{bmatrix}n\\k\end{bmatrix}\begin{Bmatrix}k\\m\end{Bmatrix}=[m=n]
$$

$$
\sum_{k=m}^n (-1)^{n-k} \begin{Bmatrix}n\\k\end{Bmatrix}\begin{bmatrix}k\\m\end{bmatrix}=[m=n]
$$

**斯特林反演**
$$
f(n)=\sum_{k=0}^n \begin{Bmatrix}n\\k \end{Bmatrix}g(k) \iff g(n)=\sum_{k=0}^n (-1)^{n-k} \begin{bmatrix}n\\k \end{bmatrix}f(k) 
$$

## 例题

### **[CF932E Team Work](https://codeforces.com/problemset/problem/932/E)**

$$
\begin{aligned}
&\sum_{i=1}^n \binom{n}{i}\times i^k\\
=&\sum_{i=1}^n \binom{n}{i}\times \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix} i^{\underline{j}}\\
=& \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix}\sum_{i=1}^n i^{\underline{j}}   \binom{n}{i}\\
=& \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix}\sum_{i=1}^n \frac{i!}{(i-j)!}\times \frac{n!}{i!(n-i)!}\\
=& \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix}\sum_{i=1}^n \frac{n!}{(i-j)!(n-i)!}\\
=& \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix}\sum_{i=1}^n \binom{n-j}{n-i} \times n^{\underline{j}}\\
=& \sum_{j=0}^k\begin{Bmatrix}k\\j\end{Bmatrix}n^{\underline{j}}\times 2^{n-j}\\
\end{aligned}
$$

### **[「HEOI2016/TJOI2016」求和](https://www.luogu.com.cn/problem/P4091)**

$$
\begin{aligned}
&\sum_{i=0}^{n} \sum_{j=0}^i \begin{Bmatrix}i\\j\end{Bmatrix}2^j\cdot j!\\
=&\sum_{i=0}^{n} \sum_{j=0}^n 2^j \begin{Bmatrix}i\\j\end{Bmatrix}\cdot j!\\
=&\sum_{i=0}^{n} \sum_{j=0}^n 2^j \sum_{k=0}^j (-1)^{j-k} \binom{j}{k}k^i\\
=&\sum_{j=0}^n 2^j \sum_{k=0}^j (-1)^{j-k} \binom{j}{k} \sum_{i=0}^{n}k^i\\
=&\sum_{j=0}^n 2^j \sum_{k=0}^j (-1)^{j-k}\frac{j!}{k!(j-k)!} \frac{k^{n+1}-1}{k-1}\\
=&\sum_{j=0}^n 2^j\cdot j! \sum_{k=0}^j \frac{(-1)^{j-k}}{(j-k)!} \frac{k^{n+1}-1}{k!(k-1)}\\
\end{aligned}
$$

$$
A(i)=\frac{(-1)^{i}}{i!} ,\;B(i)=\frac{i^{n+1}-1}{i!(i-1)}\to\; A\ast B
$$

### **[CF961G Partitions](http://codeforces.com/problemset/problem/961/G)**

$$
ans=P\times \sum w_i
$$

$$
\begin{aligned}
P&=\sum_{s=1}^{n} s \binom{n-1}{s-1}\begin{Bmatrix}n-s\\k-1\end{Bmatrix}\\
&=\sum_{s=1}^{n}s\binom{n-1}{s-1}\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t} t^{n-s}\\
&=\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t}\sum_{s=1}^{n} t^{n-s} s\binom{n-1}{s-1}\\
&=\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t}\sum_{s=1}^{n} t^{n-s} (s-1+1)\binom{n-1}{s-1}\\
&=\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t}\Bigg((n-1)\sum_{s=1}^{n} t^{n-s} \binom{n-2}{s-2}+\sum_{s=1}^n t^{n-s}\binom{n-1}{s-1}\Bigg)\\
&=\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t}\Bigg((n-1)\sum_{s=0}^{n-2} t^{n-2-s} \binom{n-2}{s}+\sum_{s=0}^{n-1} t^{n-1-s}\binom{n-1}{s}\Bigg)\\
&=\frac{1}{(k-1)!} \sum_{t=0}^{k-1} (-1)^{k-1-t} \binom{k-1}{t}\left((n-1)(t+1)^{n-2}+(t+1)^{n-1}\right)\\
\end{aligned}
$$



## 求斯特林数


$$
\begin{bmatrix}n\\k\end{bmatrix}=\begin{bmatrix}n-1\\k-1\end{bmatrix}+(n-1)\begin{bmatrix}n-1\\k\end{bmatrix},\quad\begin{bmatrix}n\\0\end{bmatrix}=[n=0]
$$

$$
\begin{Bmatrix}n\\k\end{Bmatrix}=\begin{Bmatrix}n-1\\k-1\end{Bmatrix}+k\begin{Bmatrix}n-1\\k\end{Bmatrix},\quad \begin{Bmatrix}n\\0\end{Bmatrix}=[n=0]
$$

以上式子可以 $O(n^2)$ 推出斯特林数。
$$
\begin{Bmatrix}n\\m\end{Bmatrix}=\frac{1}{m!}\sum_{k=0}^m (-1)^{m-k} \binom{m}{k}k^n
$$
上式可以 $O(m\log n)$ 求一个第二类斯特林数。

### **第二类斯特林数·行**

求 $\begin{Bmatrix}n\\0\end{Bmatrix},\begin{Bmatrix}n\\1\end{Bmatrix},\begin{Bmatrix}n\\2\end{Bmatrix},\dots,\begin{Bmatrix}n\\n\end{Bmatrix}$ 的值，$O(n\log n)$ 的复杂度。
$$
\begin{aligned}
\begin{Bmatrix}n\\m\end{Bmatrix}&=\frac{1}{m!}\sum_{k=0}^m (-1)^{m-k} \binom{m}{k}k^n\\
&=\frac{1}{m!}\sum_{k=0}^m (-1)^{m-k} \frac{m!}{k!(m-k)!}k^n\\
&=\sum_{k=0}^m \frac{(-1)^{m-k}}{(m-k)!} \frac{k^n}{k!}\\
\end{aligned}
$$

$$
[x^k]g(x)= \frac{(-1)^k}{k!}\,,\;[x^k]f(x)=\frac{k^n}{k!}
$$

$$
\begin{Bmatrix}n\\m\end{Bmatrix}=[x^m](g\ast f)(x)
$$

### **第一类斯特林数·行**

求 $\begin{bmatrix}n\\0\end{bmatrix},\begin{bmatrix}n\\1\end{bmatrix},\begin{bmatrix}n\\2\end{bmatrix},\dots,\begin{bmatrix}n\\n\end{bmatrix}$ 的值，$O(n\log n)$ 的复杂度。
$$
x^{\overline{n}} =\prod_{i=0}^{n-1} (x+i) =\sum_{i=0}^n\begin{bmatrix}n\\i\end{bmatrix}x^i
$$

$$
\begin{bmatrix}n\\m\end{bmatrix}= [x^m]\; x^{\overline{n}}
$$

考虑求 $x^{\overline{n}}$ ，倍增：
$$
\begin{aligned}
x^{\overline{2n}}&=x^{\overline{n}}(x+n)^{\overline{n}}\\
\prod_{i=0}^{2n-1} (x+i)&=\prod_{i=0}^{n-1} (x+i) \times\prod_{i=0}^{n-1} (x+n+i)\\
f(x)&=f_0(x)\ast f_0(x+n)
\end{aligned}
$$
有时候不是 $n$ 是奇数那就先算出 $n-1$ 再乘上 $(x+n-1)$ ，这个线推完事了。

问题是已知 $f(x)$ ，求 $f(x+c)$ ，$c$ 是常数（这里其实就是 $n$），为了方便这里用 $a$ 表示 $f(x)$ 的系数。 
$$
\begin{aligned}
f(x+c) &= \sum_{i=0}^{n} a_i (x+c)^i\\
&=\sum_{i=0}^{n} a_i \sum_{k=0}^i \binom{i}{k}  c^{i-k}x^k \\
&=\sum_{i=0}^{n} \sum_{k=0}^i \binom{i}{k} a_i c^{i-k}x^k \\
\end{aligned}
$$
提取 $x^k$ 系数就行了：
$$
\begin{aligned}
\big[x^k\big]f(x+c) &=\sum_{i=k}^n \binom{i}{k} a_ic^{i-k}\\
&=\sum_{i=k}^n \frac{i!}{k!(i-k)!} a_ic^{i-k}\\
&=\sum_{i=k}^n \frac{i!}{k!(i-k)!} a_ic^{i-k}\\
&=\frac{1}{k!}\sum_{i=k}^n a_i\,i!\cdot \frac{c^{i-k}}{(i-k)!}\\
&=\frac{1}{k!}\sum_{i=0}^{n-k} a_{i+k}\,(i+k)!\cdot \frac{c^{i}}{i!}
\end{aligned}
$$
复杂度：
$$
T(n)=T\left(\frac{n}{2}\right)+O(n\log n)=O(n\log n)
$$

