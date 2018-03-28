---
layout: post
title: '莫比乌斯反演与杜教筛'
date: 2018-01-25
author: Deluxurous
cover: '/assets/img/mathematics.png'
categories: OI
tags: 数学 数论
excerpt: '简单讲一下数论相关吧'
draft: true
---

### 前置博文

[Skywalkert的博客](http://blog.csdn.net/skywalkert/article/details/50500009)

[abclzr](https://www.cnblogs.com/abclzr/p/6242020.html)

这些博文写的不错啊，挺好懂的

# 数论函数

认识几个函数：

## 莫比乌斯函数 $ \mu $

$$\mu(n) = \begin{cases}
   1      &\text{if } n = 1  \\
   (-1)^k &\text{if } n = p_1p_2p_3\cdots p_k (p_1 \cdots p_k为互不相同的质数) \\
   0      &\text{otherwise } \mathrm{i.e.} 当 n 存在平方因子时 
\end{cases} $$

$\mu$ 函数也被称作**莫比乌斯函数**。

### 性质

**性质一** &nbsp;&nbsp; $ \mu $ 是一个不完全积性函数， 有：

$$ \mu(a) \cdot \mu(b) = \mu(a \cdot b) , 当 a \perp b $$

这里 $ a \perp b $ 表示 $a$ 与 $b$ 互质。

**性质二** &nbsp;&nbsp; 当 $ n $ 不等于 $ 1 $ 时，$ n $ 所有因子的莫比乌斯函数值的和为 $ 0 $ ，

$$ \sum_{d|n} \mu(d) = 0 \ , \ n \not= 1 $$

当 $ n = 1 $ 时， 上式等于 $ 0 $ 。

**性质三** &nbsp;&nbsp; 在无穷极限中， 有：

$$ \sum_{i=1}^{\infty} \frac{\mu(i)}{i} = 0 $$

## 欧拉函数 $ \varphi $

$$ \varphi(n) = 1 \sim n 中与 n 互质的数的个数 $$

通式为：

$$ \varphi(n) = n \prod_{i=1}^{k} \left( 1 - \frac{1}{p_i} \right) $$

其中 $ p_1, p_2, \cdots, p_k $ 为 $ n $ 的所有的质因数，且 $ x \not= 0 $。

$$ \varphi(1) = 1 $$

### 性质

**性质一** &nbsp;&nbsp; $ \varphi $ 是一个不完全积性函数，有：

$$ \varphi(a) \cdot \varphi(b) = \varphi(a \cdot b) , 当 a \perp b $$

这里 $ a \perp b $ 表示 $a$ 与 $b$ 互质。

**性质二** &nbsp;&nbsp; 当 $ n $ 为奇数时， 有：

$$ \varphi(2n) = \varphi(n) $$

当 $ n $ 为质数时，有：

$$ \varphi(n) = n - 1 $$

当 $ n $ 为大于 $ 2 $ 的正整数时， $ \varphi(n) $ 是偶数。

**性质三** &nbsp;&nbsp; 有如下柿子：

$$ \sum_{i=1}^{n} i \left[\gcd(n, i) = 1 \right] = \frac12 \left( n \cdot \varphi(n) + \left[ n = 1 \right] \right) $$

## 其他数论函数

1. $ 1(n) = 1 $， 完全积性

2. $ \mathrm{id}(n) = n $， 完全积性

3. $ \mathrm{id}^k(n) = n^k $， 完全积性

4. $$ \varepsilon(n) = \begin{cases}
    1 &\text{if } n = 1 \\
    0 &\text{otherwise}
\end{cases} $$， 完全积性

5. $ \sigma_k(n) = \sum_{d \mid n} d^k $，表示 $ n $ 的约数的 $ k $ 次幂的和。

6. $ \sigma(n) = \sigma_1(n) = \sum_{d \mid n} d $， 表示 $ n $ 的约数之和。

7. $ \tau(n) = \sigma_0(n) = \sum_{d \mid n} 1 $， 表示 $ n $ 的约数个数。

## 狄利克雷卷积

设 $ f(n), g(n) $ 是两个数论函数， 他们的狄利克雷卷积定义为：

$$ h(n) = \sum_{d \mid n} f(d) \cdot g(\frac{n}{d}) $$

记为 $ f \ast g $。公式也可以这样写：

$$ h(n) = \sum_{ab = n} f(a) \cdot g(b) $$

两个数论函数的狄利克雷卷积也是一个数论函数。

### 性质

1. 结合律：
  $$ \left( f \ast g \right) \ast h = f \ast \left ( g \ast h \right) $$

2. 交换律：
  $$ f \ast g = g \ast f $$

未完待续……