---
layout: post
title: '【集训队作业2018】矩形题解'
date: 2018-03-28
author: Deluxurous
cover: '/assets/img/mathematics.png'
categories: OI
tags: 数学 数论
excerpt: ' '
---

# 题目大意
本题中约定 $0^0 = 1$。

Snuke 使用动态规划解决了一道题目。具体来说，她设计了如下递推式：

$$F(i, j) =
\begin{cases}
  0, & \text{if $i = 0$;} \\
  f_i, & \text{if $i > 0$ and $j = 0$;} \\
  aF(i - 1, j) + bF(i, j - 1) + c, & \text{if $i > 0$ and $j > 0$.}
\end{cases}$$

其中 $i,j$ 是非负整数，$a,b,c$ 是给定的常数，$f_i$ 是给定的数列。

Snuke 需要求出 $F(n,m)$，所以她对于 $1 \le i \le n, 1 \le j \le m$ 计算了 $F(i,j)$，这些值形成了一个 $n \times m$ 的矩阵。

Takahashi 希望你告诉她这个矩阵。为了避免过多的输出，你只需要输出这个矩阵的哈希值。

具体来说，给定整数 $h$ 和质数 $p$，你需要输出

$$\left(
\sum_{i = 1}^{n} \sum_{j = 1}^{m} F(i, j) \, h^{(i - 1)m + (j - 1)}
\right) \bmod p$$

的值。

# 题解

一道分类讨论+推式子好题。

我们分两步计算贡献。分别是每个 $f_i$ 以及 $c$ 的贡献。

## Step 1

对于每个 $f_i$ 我们可以发现其总贡献为

$$\begin{aligned}&\sum_{x=1}^{n}f_x\sum_{i=x}^{n}\sum_{j=1}^{m}\binom{i-x+j-1}{j-1}a^{i-x}b^{j}h^{(i-1)m+(j-1)} \\
\Rightarrow &\sum_{x=1}^{n}f_x\sum_{i=x}^{n}a^{i-x}h^{(i-1)m}\sum_{j=0}^{m}\binom{i-x+j-1}{j-1}b^{j}h^{j-1} \\
\Rightarrow &\sum_{x=1}^{n}f_x\sum_{i=0}^{n-x}a^ih^{(i+x)m}\sum_{j=0}^{m-1}\binom{i+j}{j}b^{j+1}h^{j}
\end{aligned}$$

那么如何计算这个式子呢？我们定义一个函数 $F(x)$ 为

$$\sum_{j=0}^{m-1}\binom{x+j}{j}b^{j+1}h^{j}$$

那么我们可以得到:

$$\begin{aligned}
F(x) &= \sum_{j=0}^{m-1}\binom{x+j}{j}b^{j+1}h^{j} \\
&=\sum_{j=0}^{m-1}\binom{x+j-1}{j}b^{j+1}h^j+\sum_{j=0}^{m-1}\binom{x+j-1}{j-1}b^{j+1}h^j \\
&=F(x-1)+\sum_{j=0}^{m-2}\binom{x+j}{j}b^{j+2}h^{j+1}\\
&=F(x-1)+bh\sum_{j=0}^{m-1}\binom{x+j}{j}b^{j+1}h^{j}-\binom{x+m-1}{m-1}b^{m+1}h^{m}\\
&=F(x-1)+bhF(x)-\binom{x+m-1}{m-1}b^{m+1}h^{m}\\
&=\frac{F(x-1)-\binom{x+m-1}{m-1}b^{m+1}h^{m}}{1-bh}
\end{aligned}$$

再定义函数 $R(x)$ 为

$$\binom{x+m-1}{m-1}b^{m+1}h^{m}$$

那么有

$$\begin{aligned}
R(x)&=\frac{x+m-1}{x}\binom{x+m-2}{m-1}b^{m+1}h^m \\
&=\frac{x+m-1}{x}R(x-1)
\end{aligned}$$

所以我们可以得到

$$F(x)=\frac{F(x-1)-R(x)}{1-bh}$$

当 $bh=1$ 时，上述公式不适用，需要特判。根据之前的推导过程，有

$$\begin{aligned}
(1-bh)F(x)&=F(x-1)-R(x) \\
0&=F(x-1)-R(x) \\
F(x)&=R(x+1)
\end{aligned}$$

这样就可以 $O(n)$ 递推 $F(x)$ 和 $R(x)$ 了。

对于初值问题，当 $bh\not=1$ 时，有

$$F(0)=\sum_{j=0}^{m-1}b^{j+1}h^{j}=\frac{b-b^{m+1}h^{m}}{1-bh}$$

当 $bh=1$ 时，有

$$F(0)=\sum_{j=0}^{m-1}b=mb$$

对于函数 $R(x)$ ，没有特殊情况。

$$R(0)=b^{m+1}h^{m}$$

接下去我们定义函数 $G(x)$ 为

$$\sum_{i=0}^{n-x}a^ih^{(i+x-1)m}F(i)$$

推一下式子：

$$\begin{aligned}
G(x)&=\sum_{i=0}^{n-x+1}a^ih^{(i+x-1)m}F(i)-a^{n-x+1}h^{nm}F(n-x+1) \\
&=h^mG(x-1)-a^{n-x+1}h^{nm}F(n-x+1)
\end{aligned}$$

这个递推式也没有特殊情况。初值为

$$G(0)=\sum_{i=0}^{n}a^ih^{(i-1)m}F(i)$$

综上，这一部分的贡献总和就是

$$\sum_{x=1}^nf_xG(x)$$

计算复杂度 $O(n)$ 。

## Step 2

接下去计算 $c$ 的贡献。其总贡献为：

$$\begin{aligned} &\sum_{x=1}^{n}\sum_{y=1}^{m}\sum_{i=0}^{x-1}\sum_{j=0}^{y-1}\binom{i+j}{j}a^ib^jch^{(x-1)m+(y-1)} \\
\Rightarrow \;&c\sum_{x=1}^{n}h^{(x-1)m}\sum_{i=0}^{x-1}a^i\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{i+j}{j}b^j
\end{aligned}$$

我们定义函数 $P(x)$ 为

$$a^{x}\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{x+j}{j}b^j$$

再推一波式子：

$$\begin{aligned}
P(x)&=a^{x}\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{x+j}{j}b^j\\
&= a^x\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{x+j-1}{j}b^j+a^x\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{x+j-1}{j-1}b^j\\
&=aP(x-1)+a^x\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-2}\binom{x+j}{j}b^{j+1} \\
&=aP(x-1)+a^x\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}\binom{x+j}{j}b^{j+1}-a^x\sum_{y=0}^{m-1}h^{y}\binom{x+y}{y}b^{y+1} \\
&=aP(x-1)+bP(x)-a^xF(x)\\
&=\frac{aP(x-1)-a^xF(x)}{1-b}
\end{aligned}$$

当 $b=1$ 时，递推式又会萎。

$$\begin{aligned}
(1-b)P(x)&=aP(x-1)-a^xF(x) \\
0&=aP(x-1)-a^xF(x) \\
P(x)&=a^xF(x+1)
\end{aligned}$$

$P(x)$ 的初值问题比较麻烦。

当 $b\not=1, h\not=1, bh\not=1$ 时，有

$$
\begin{aligned}P(0)&=\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}b^j \\
&=\sum_{y=1}^mh^{y-1}\frac{1-b^y}{1-b} \\
&=\frac{b}{1-b}\left(\sum_{y=1}^mh^{y-1}-\sum_{y=1}^mh^{y-1}b^{y}\right) \\
&=\frac{1}{1-b}\left(\frac{1-h^m}{1-h}-\frac{b-h^mb^{m+1}}{1-bh}\right)
\end{aligned}$$

当 $b=1, h\not=1$ 时，有

$$P(0)=F(1)$$

当 $b\not=1, h=1$ 时，有

$$\begin{aligned}
P(0)&=\sum_{y=1}^m\sum_{j=0}^{y-1}b^j \\
&=\sum_{y=1}^m\frac{1-b^y}{1-b} \\
&=\frac{1}{1-b}\sum_{y=1}^m(1-b^y) \\
&=\frac{1}{1-b}\left(m-\frac{b-b^{m+1}}{1-b}\right)
\end{aligned}$$

当 $b=h=1$ 时，有

$$\begin{aligned}
P(0)&=\sum_{y=1}^m\sum_{j=0}^{y-1}1 \\
&=\frac{m(m+1)}{2}
\end{aligned}$$

当 $b\not=1, h\not=1, bh=1$ 时，有

$$
\begin{aligned}P(0)&=\sum_{y=1}^mh^{y-1}\sum_{j=0}^{y-1}b^j \\
&=\frac{b}{1-b}\left(\sum_{y=1}^mh^{y-1}-\sum_{y=1}^mh^{y-1}b^{y}\right) \\
&=\frac{1}{1-b}\left(\frac{1-h^m}{1-h}-mb\right)
\end{aligned}$$

然后这一部分的贡献就是

$$c\sum_{x=1}^nh^{(x-1)m}\sum_{i=0}^{x-1}P(i)$$

$O(n)$ 计算 $P(x)$ 之后前缀和一下，再 $O(n)$ 计算贡献。

## Step 3

最后，把上面两部分贡献数值相加就是最终答案。总复杂度 $O(n)$ 。

我感到以上某些式子有更简单的推法能得到更简单的答案，并且这个做法常数略大，uoj 上跑了 18291ms ，用了 Fastdiv 之后 6321ms ，快三倍，成功挤进第一页 233333。

再%%%lsk