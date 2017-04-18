---
layout: post
title:  "次梯度"
date:   2017-04-11
categories:
- math
tags:
- svm
---

* content
{:toc}
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

## 次导数
设f在实数域上是一个凸函数，定义在数轴上的开区间内。这种函数不一定是处处可导的，例如绝对值函数$f(x) = |x|$ 。对于下图来说，对于定义域中的任何x0，我们总可以作出一条直线，它通过点(x0, f(x0))，并且要么接触f的图像，要么在它的下方。直线的斜率称为函数的次导数。次导数的集合称为函数f在x0处的次微分。

![不可微函数](https://upload.wikimedia.org/wikipedia/commons/4/4e/Subderivative_illustration.png)

### 定义
对于所有x，我们可以证明在点$x_0$ 的次导数的集合是一个非空闭区间[a,b]，其中a和b是单测极限。

$$ a = \lim_{x->x^-_0} \frac {f(x)- f(x_0)}{x-x_0}$$

$$ b = \lim_{x->x^+_0} \frac {f(x)- f(x_0)}{x-x_0}$$

一定存在，且a<=b，在[a,b]内的所有次导数是f在x0的次微分。
### 例子
凸函数$f(x)=|x|$。在原点的次微分是[-1,1]。当x0<0时，次微分是单元素集合{-1},而x0>0时，次微分单元素集合是{1}。
### 性质
当函数在x0处可导时，次微分只有一个点组成，这个点就是函数在x0处的导数。
## 次梯度法
次梯度方法(subgradient method)是传统的梯度下降方法的拓展，用来处理不可导的凸函数。它的优势是比传统方法处理问题范围大，劣势是算法收敛速度慢。但它对不可导函数有很好的处理方法。
通过求函数在点的每一分量的次导数可以求出函数在该点的次梯度。

