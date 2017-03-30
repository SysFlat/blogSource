---
title: modular-exponentiation
date: 2017-03-30 10:17:00
categories: 密码学
tags:
---
众所周知，密码算法的应用范围十分广泛。一个好的密码算法在保证功能正常的前提下，在实际应用中也必须有非常高效的执行效率。然而涉及到的运算动不动就是成千上百位的大数运算Σ( ° △ °|||)︴。一个1024位的数的存储就很不容易了，更不用说这些大数的幂运算，运算量会爆炸性的增长，想想就觉得头大。下面就就密码算法中常用的大数模幂的实现方法进行一些讨论。
<!-- more -->
# 模幂运算
模幂的基本形式是**a^b mod p**，对于这样的形式来说，我们最先想到的算法便是将b个a依次相乘，类似这样：
## 原始算法
```C
res = 1;
i = 1;
while(i <= b)
{
    res *= a;
    i++;
}
res = res % p;
```
不用说算法的执行效率有多低，a和b都是大数运算，中间累乘起来的结果早就已经溢出了。但是我们有这样的一个性质：**(a*b)mod p =((a mod p) * (b mod p) mod p)**。
通过这样的性质我们可以对上面的算法进行优化，把每一步的计算都限定在小于p的范围内进行，比如这样：
## 优化1
```C
res = 1;
i = 1;
while(i <= b)
{
    res = (a % p) * (res % p) % p;
    i++;
}
res = res % p;
```
这样的优化效果如何，举个栗子你就知道了：
```
81^3 mod 7 = (81 % 7)*(81 % 7)*(81 % 7) % 7 = (4*4*4) % 7 = (2*4) % 7 =1
```
这样的效果就是可以保证在整个计算的过程中都不出现比模数p大的数参与运算，这样就可以大大加快算法的执行过程。
## 优化2
上面的算法我们都没有特别考虑大数的情况，如果**a^b mod p**中的指数b也是一个比较大的数的话，那么while循环的次数就显得太过于恐怖。如何优化，请看：
![算法示意](/img/2017.3.29.png)
通过上面的分析，我们的程序就变成了这样：
```C
res = 1;
base = a % p;
while(b != 0)
{
    res = res % p;
    if(b&1) res = (res * base) % p;
    base *= base;
    base = base % p;
    b = b >> 1;
}
```
通过这种优化，可以看出我们的循环次数已经大大减少（循环次数位b的二进制长度）同时在内部计算的时候我们也时刻保证参与运算的数不会大于模数p，这样的算法就是我们需要的。
## 优化3
通过上面的优化这个算法的实现应基本能够满足我们的需求，但是在密码学计算中模幂的模数一般都是一个比较大的素数，这样就可能会有另外的简化运算的方法。要想知道这个方法，就不得不提到欧拉函数了。

### 欧拉定理（费马-欧拉定理）

对于任何的正整数p，如果正整数a与p互为质数，那么就有下面的公式成立：**a^f(p) mod p ≡ 1**其中f(p)是不能被p整除且小于p的正整数的个数，比如`f(15) = 8`,因为满足这样的正整数有`{1,2,4,7,8,11,13,14}`
### 费马小定理
定理描述如下：a是不能被质数p整除的正整数，则有**a^(p-1) ≡ 1 (mod p)**。
其实这个定理是上面定理的一个特殊情况，由上面的欧拉定理我们很容易知道对于质数p，f(p)=p-1。
所以在计算大数模幂运算的开始，我们都最好根据上面的定理先进行一下判断。虽然可能用到这样定理的可能性很小，但是聊胜于无，况且对于概率这种东西谁能说得清楚呢。



 

  
<blockquote class="blockquote-center">完</blockquote>