---
title: '快速莫比乌斯/沃尔什变换 (FMT/FWT)' 
date: 2021-01-06 21:57:03
tags: [多项式]
categories:
- 📚算法笔记
---

我们已知可以用 FFT 求加法卷积 $C_i=\sum_{j+k=i}A_j\cdot B_k$。

但形如 $C_i=\sum_{j\bigoplus k=i}A_j\cdot B_k$ (其中 $\bigoplus$ 为一个位运算符) 这样的位运算卷积，FFT 便束手无策。于是就要用到 『快速莫比乌斯变换』 FMT / 『快速沃尔什变换』 FWT 。

<!--more-->

## 快速莫比乌斯变换（FMT）

### 与运算

$$
C_i=\sum_{i=j\&k} A_j\cdot B_k
$$

定义变换：

$$
\mathrm{FMT}(A)_i=\sum_{j\&i=i} A_j\\
$$

这里可以把 $i,j$ 看作集合（二进制下为一表示该位有，为零表该位无），$\mathrm{FMT}(A)_i$ 就表示 $i$ 的所有超集的和。或者把每位看作数组中的一维，那么就类似于高维前缀和。

我们需要证明这个变换满足 $\mathrm{FMT}(C)=\mathrm{FMT}(A)\cdot \mathrm{FMT}(B)$ ：
$$
\begin{aligned}
&\left(\mathrm{FMT}(A)\cdot \mathrm{FMT}(B) \right)_i  \\[1.5ex]
=&\left(\sum_{j\&i=i}A_j \right)\left(\sum_{k\&i=i}B_k \right) &\text{根据定义}\\[2ex]
=&\sum_{j\&i=i}\sum_{k\&i=i} A_jB_k & \text{和式性质}\\[1.5ex]
=&\sum_{(k\&j)\&i=i} A_jB_k&\text{$
\left\{
\begin{matrix}
j\&i=i\\
k\&i=i
\end{matrix}
\right.
\to (k\&j)\&i=i
$}
\end{aligned}
$$

根据定义可证：
$$
\begin{aligned}
&\mathrm{FMT}(C)_i=\sum_{p\&i=i} C_p,\; C_p=\sum_{p=j\&k} A_j\cdot B_k\\
\implies&\mathrm{FMT}(C)_i=\sum_{(j\&k)\&i=i}A_j\cdot B_k\\
\implies&\mathrm{FMT}(C)=\mathrm{FMT}(A)\cdot \mathrm{FMT}(B)
\end{aligned}
$$
考虑如何快速实现变换：

$A\to \mathrm{FMT}(A)$  ，$\deg A=2^n$

分治：$A$ 拆成前 $2^{n-1}$ 项记 $A_0$ ，和后 $2^{n-1}$ 项记 $A_1$  。（前者下标在二进制下最高位都为 $0$ ，后者都为 $1$ ）
$$
\fbox{
$\mathrm{FMT}(A)=\mathrm{merge} \big(\mathrm{FMT}(A_0)+\mathrm{FMT}(A_1),\mathrm{FMT}(A_1)\big)$
}
$$
$+$ 表示对应位相加，$\mathrm{merge}$ 表示把两个序列“拼起来”，具体的：
$$
\mathrm{FMT}(A)_i=\begin{cases}
\mathrm{FMT}(A_0)_i + \mathrm{FMT}(A_1)_i & \text{if  $i \lt2^{n-1}$ }\\
\mathrm{FMT}(A_1)_{i-2^{n-1}}  & \text{if  $i \ge 2^{n-1}$ }
\end{cases}
$$

$$
\mathrm{FMT}(A)=A \quad \text{if $n=0$}
$$

$\mathrm{FMT}(A)\to A$ ，记变换为 $\mathrm{IFMT}(A)$ ，和上边差不多，就是反过来：
$$
\fbox{
$\mathrm{IFMT}(A)=\mathrm{merge}\big(\mathrm{IFMT}(A_0)-\mathrm{IFMT}(A_1),\mathrm{IFMT}(A_1)\big)$
}
$$
具体的：
$$
\mathrm{IFMT}(A)_i=\begin{cases}
\mathrm{IFMT}(A_0)_i-\mathrm{IFMT}(A_1)_i & \text{if  $i \lt2^{n-1}$ }\\
\mathrm{FMT}(A_1)_{i-2^{n-1}}  & \text{if  $i \ge 2^{n-1}$ }
\end{cases}
$$

$$
\mathrm{IFMT}(A)=A \quad \text{if $n=0$}
$$

复杂度都是 $O(n\log n)$ 的。

#### CODE

```cpp
inline void FMT_AND(ll f[],int n,int fg) { // fg=1 for FMT ; fg=0 for IFMT
    for(re int h=2,k=1;h<=n;h<<=1,k<<=1)
        for(re int i=0;i<n;i+=h)
            for(re int j=0;j<k;++j)
                f[i+j]=MOD(f[i+j]+f[i+j+k]*fg+mod);
}
```

---

### 或运算

$$
C_i=\sum_{i=j|k} A_j\cdot B_k
$$

定义变换：

$$
\mathrm{FMT}(A)_i=\sum_{j|i=i} A_j\\
$$
证明略（和与运算差不多），这里直接给出变换的快速实现：
$$
\fbox{
$\mathrm{FMT}(A)=\mathrm{merge} \big(\mathrm{FMT}(A_0),\mathrm{FMT}(A_0)+\mathrm{FMT}(A_1)\big)$
}
$$

$$
\fbox{
$\mathrm{IFMT}(A)=\mathrm{merge}\big(\mathrm{IFMT}(A_0),\mathrm{IFMT}(A_1)-\mathrm{IFMT}(A_0)\big)$
}
$$

#### CODE

```cpp
inline void FMT_OR(ll f[],int n,int fg) { // fg=1 for FMT ; fg=0 for IFMT
    for(re int h=2,k=1;h<=n;h<<=1,k<<=1)
        for(re int i=0;i<n;i+=h)
            for(re int j=0;j<k;++j)
                f[i+j+k]=MOD(f[i+j+k]+f[i+j]*fg+mod);
}
```

## 快速沃尔什变换（FWT）

与前者不一样了，这个要稍微麻烦一些。

### 异或运算

$$
C_i=\sum_{i=j\oplus k} A_j\cdot B_k
$$

这里的 $\oplus$ 表异或 ($\operatorname{xor}$) 。

我们定义 $a\circ b=\mathrm{popcount}(a\& b)\bmod 2$ ，$\mathrm{popcount}(x)$ 表示 $x$ 二进制下一的个数。

先证明一个性质 $(i\circ j)\oplus (i\circ k)=i\circ(j \oplus k)$ ,[^1]
$$
\begin{aligned}
&(i\circ j)\oplus (i\circ k)=i\circ(j \oplus k)\\
\iff& \mathrm{popcount}(i\& j)\oplus \mathrm{popcount}(i\& k)\equiv\mathrm{popcount}\big(i\& (j\oplus k)\big) \pmod{2}\\
\iff& \mathrm{popcount}(i\& j)+ \mathrm{popcount}(i\& k)\equiv\mathrm{popcount}\big(i\& (j\oplus k)\big) \pmod{2}\\
\end{aligned}
$$

设 $x=i\&j,y=i\&k,z=i\&(j\oplus k)$ ，考虑二进制下任意一位 ，该位上：
$$
\begin{array}{c|c}
x &  y & z & \text{explain} &\text{check}\\
\hline
1 & 1 & 0 & \text{$j,k$ 这位必然都为一，异或为零} & 1+1\equiv 0\pmod {2}\\
1 & 0 & 1 & \text{$j,k$ 这位必然不同，异或为一} & 1+0\equiv 1\pmod {2}\\
0 & 1 & 1 & \text{$j,k$ 这位必然不同，异或为一} & 0+1\equiv 1\pmod {2}\\
0 & 0 & 0 & \text{$i$ 这位为零显然成立，为一时 $j,k$ 这位都为零也成立} & 0+0\equiv 0\pmod {2}\\
\end{array}
$$
证毕。

设计变换

$$
\mathrm{FWT}(A)_i =\sum _{i\circ j=0} a_j-\sum _{i\circ j=1} a_j
$$

十分优美对吧…… 同样的，要证明 $\mathrm{FWT}(C)=\mathrm{FWT}(A)\cdot \mathrm{FWT}(B)$

$$
\begin{aligned}
&\big(\mathrm{FWT}(A)\cdot \mathrm{FWT}(B) \big)_i\\[1.5ex]
=& \left(\sum _{i\circ j=0} A_j-\sum _{i\circ j=1} A_j\right) \left(\sum _{i\circ k=0} B_k-\sum _{i\circ k=1} B_k\right) \\[2ex]
=& \left( \sum _{i\circ j=0}A_j \right) \left(\sum _{i\circ k=0} B_k\right)
-\left(\sum _{i\circ j=1} A_j\right)\left(\sum _{i\circ k=0} B_k\right)\\[2ex]
&\quad -\left(\sum _{i\circ j=0}A_j\right)\left(\sum _{i\circ k=1} B_k\right)
+\left(\sum _{i\circ j=1} A_j\right)\left(\sum _{i\circ k=1} B_k\right)\\[2ex]
=&\sum_{i\circ (j\oplus k)=0} A_jB_k -\sum_{i\circ (j\oplus k) =1} A_jB_k\\[2ex]
=& \;\mathrm{FWT}(C)_i
\end{aligned}
$$

考虑快速变换：

$$
\fbox{
$\mathrm{FWT}(A)=\mathrm{merge}\Big(\mathrm{FWT}(A_0)+\mathrm{FWT}(A_1),\mathrm{FWT}(A_0)-\mathrm{FWT}(A_1)\Big)$
}
$$

证明：$A_0,A_1$ 都是不考虑二进制下当前最高位的（也就是看作 $0$) ，现在考虑算上这位的变化。
对于前一半的数，最高位为 $0\&0=0,0\&1=0$ 不变。
对于后一半的数，最高位为 $1\&0=0,1\&1=1$ ，$\circ$ 运算结果改变，故后者要变号 。

逆变换就反一下：

$$
\fbox{
$\mathrm{IFWT}(A)=\mathrm{merge}\Bigg(\frac{\mathrm{IFWT}(A_0)+\mathrm{IFWT}(A_1)}{2},\frac{\mathrm{IFWT}(A_0)-\mathrm{IFWT}(A_1)}{2}\Bigg)$
}
$$

#### CODE

```cpp
inline void FWT_XOR(ll f[],int n,int fg) { // fg=1 for FWT; fg=1/2(inv 2) for IFWT
    ll t;
    for(re int h=2,k=1;h<=n;h<<=1,k<<=1)
        for(re int i=0;i<n;i+=h)
            for(re int j=0;j<k;++j)
                t=f[i+j+k],f[i+j+k]=MOD(f[i+j]-t+mod),f[i+j]=MOD(f[i+j]+t),
                f[i+j]=MOD(f[i+j]*fg),f[i+j+k]=MOD(f[i+j+k]*fg);

}
```

---

### 同或运算

$$
C_i=\sum_{i=j\odot k} A_j\cdot B_k
$$

与异或类似，这里只给出结论。

定义 $a\circ b=\mathrm{popcount}(a\mid b)\bmod 2$

$$
\mathrm{FWT}(A)_i =\sum _{i\circ j=0} a_j-\sum _{i\circ j=1} a_j
$$

$$
\fbox{
$\mathrm{FWT}(A)=\mathrm{merge}\Big(\mathrm{FWT}(A_1)-\mathrm{FWT}(A_0),\mathrm{FWT}(A_1)+\mathrm{FWT}(A_0)\Big)$
}
$$

$$
\fbox{
$\mathrm{IFWT}(A)=\mathrm{merge}\Bigg(\frac{\mathrm{IFWT}(A_1)-\mathrm{IFWT}(A_0)}{2},\frac{\mathrm{IFWT}(A_1)+\mathrm{IFWT}(A_0)}{2}\Bigg)$
}
$$

## Reference

- [Oi-wiki 快速沃尔什变换](https://oi-wiki.org/math/poly/fwt/#_4)
[^1]: 证明参考 [a___的博客](https://www.luogu.com.cn/blog/35137/solution-p4717)
