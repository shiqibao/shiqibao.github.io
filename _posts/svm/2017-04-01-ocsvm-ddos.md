---
    author: baoshiqi
    comments: true
    date: 2017-04-01
    layout: post
    title: OC-SVM在ddos攻击中的应用
    categories:
    - machine learning
    tags:
    - svm
---
* content
{:toc}


# 基于单类支持向量机的ddos检测应用层
## 摘要
分布式ddos攻击利用网络协议和服务的复杂性和多样性来进行攻击。这类攻击相比于其他ddos攻击更加难以防止。单类支持向量机只需要正例样本进行训练，适用于异常点检测场景。在这种检测策略下，从正常用户的session中先抽取7个特征，然后用OC SVM训练模型，最后用模型预测。通过仿真实现展示检测方法的数据结果。
## 引言
提出OC主要是解决不平衡分类问题，目的是找到包含最多正例的超球面。基于此观点，OC判别测试点是不是属于正常类。因为只需要正常数据训练，所以对于检测ddos攻击很有效。总结如下：

1. 根据攻击用户和正常用户的区别从用户的session中抽取7个特征
2. 通过OC建立的正常用户的浏览模型检测ddos
3. 根据真实网站展示结果

## 检测模型和算法
### 检测算法
使用oc建立正常用户模型，然后新session如果脱离了正常用户模型就被检测成反常的。OC算法是通过非线性核函数的手段隐式的将数据从输入空间映射到特征空间。数据映射后，训练数据会被归属于一个类。这样在特征空间中就找到了一个可以将测试数据分割开的边界。

在训练之前需要数据预处理，OC算法输入是数据矩阵的数据结构。
$$
\begin{equation}       %开始数学环境
\left(                 %左括号
  \begin{array}{ccc}   %该矩阵一共3列，每一列都居中放置
    X_{11} &... & X_{1k} &... & X_{1p}\\  %第一行元素
    ... &... & ... &... & ...\\
    X_{j1} &... & X_{jk} &... & X_{jp}\\  
    ... &... & ... &... & ...\\
    X_{n1} &... & X_{nk} &... & X_{np}\\  
  \end{array}
\right)                 %右括号
\end{equation}
$$
数据矩阵是正常数据集，是一个n行p列的矩阵，有n个样本，p维特征。
#### 用户session产生
对于用户来说，浏览记录是一个请求序列。我们假设客户端IP，两个连续请求间隔小于m(默认是1800)秒认为是相同的session
#### 特征选择
在正常用户和攻击者的session之间有许多不同，例如请求率，请求的资源，请求次序，session时间等。根据这些不同，可以选取特征。

1. 资源流行度：
在网站中资源有不同的访问频率，这里的资源具有灵活的意义：可以是网页，图片，视频，音频或者文字等。大约10%的网页贡献了80-90%的请求。
定义资源流行度为：
$$
pop_i = \frac{AC_i}{AC_{all}}
$$
其中，$AC_i$是资源i在一段时间内的访问次数，$AC_{all}$是所有资源在一段时间内的访问次数。如果A资源的访问次数大于B资源，就说明A的流行度高于B。

2. 转移概率
在一个网站中，两个资源之间的转移概率是不同的。如下公式表达了如何计算转移概率：
$$P_ij = \frac{Trans_{i->j}}{Trans_{i->all}}$$
$Trans_{i->j}$ 代表从资源i到资源j的转移次数，$Trans_{i->all}$ 代表资源i到其他所有资源的转移次数。
3. 历史转移矩阵
历史转移矩阵记录从一个资源到其余所有资源的转移次数，定义如下：
$$
\begin{equation}       %开始数学环境
\left(                 %左括号
  \begin{array}{ccc}   %该矩阵一共3列，每一列都居中放置
    t_{11} &... & t_{1i} &... & t_{1n}\\  %第一行元素
    ... &... & ... &... & ...\\
    t_{i1} &... & t_{ii} &... & t_{in}\\  
    ... &... & ... &... & ...\\
    X_{n1} &... & X_{ni} &... & X_{nn}\\  
  \end{array}
\right)                 %右括号
\end{equation}
$$
$t_{ij}$ 是资源i到资源j的转移次数。

介绍了如上概念和定义后，介绍OC算法所选择的特征。这些特征都可以从网页服务日志中获取。
1. $N_{session}$ : 在1个session中所有的请求数。
session中需要有足够多的请求容量才能抵御ddos攻击
2. $POP_{session}$ :在一个session内所有请求的平均流行度。
因为简单实现，所以一些ddos使用随机序列发起攻击。随机请求攻击的检测比简单请求一次复杂。研究表明，大部分的资源都是相对的低流行度，随机请求所产生的 $POP_{session}$低于正常用户session产生的 $POP_{session}$。
3.  $P_{session}$ ：session中所有邻近请求的平均转移概率。
对于随机请求攻击，它的请求序列是随机产生的。对比与正常用户的session，攻击session的$P_{session}$ 通常小于大部分正常session的$P_{session}$ 。
4. $Size_{session}$ ：session中所有请求的总大小。
攻击者可能请求例如大视频这种大型资源，就是说虽然是一个简单的请求，但是会消耗很大的带宽。本文认为$Size_{session}$ 就是session负载。
5. $D{session}$:session 的持续时间
在一个session中，从第一次请求到最后一次请求的时间区间。通常，正常用户不会在一个网站上待太长时间。对于隐蔽攻击来说，或许需要较长的时间才能达到攻击效果。
6.  $replycode_{n}$: http状态码：
　　200 - 服务器成功返回网页
　　404 - 请求的网页不存在
　　503 - 服务器超时
统计学上讲，从网站服务器上返回的http请求状态吗有不同的出现频率。通常来说，为了达到更好的攻击效果需要更有效的网站请求。所以攻击session会有更多的200回复码，除非它模仿正常用户session中的代码分布，但反过来这会削弱攻击效果。
7.  动态：动态页面出现次数
正常情况下，动态页面包括数据库的查询或者插入，复杂脚本的计算和其他具有高负载的操作。一些小规模的攻击工具会攻击这类易受损的网页。
### 检测过程
根据使用正常数据集训练好的OC模型，预测新的session x，如果返回1，判为正常；如果返回-1判为异常，将请求IP加入黑名单。

