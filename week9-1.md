---
title: 吴恩达笔记10-异常检测
top: false
cover: false
toc: true
mathjax: true
copyright: true
date: 2019-12-06 11:35:39
password:
summary:
tags:
  - 异常检测
categories:
  - Machine learning
  - 吴恩达
---

Week9-异常检测


> “黑中有白，白中有黑，没有绝对的白，也没有绝对的黑，黑可衬白，白可映黑。万物皆可转换”。

本文中对异常检测算法做了小结，主要包含：

1. 问题产生
2. 高斯分布
3. 算法使用场景
4. 八种无监督异常检测技术
5. 异常检测和监督学习对比
6. 特征选择

<!--MORE-->



### 异常检测Novelty Detection

> 异常是相对于其他观测数据而言有明显偏离的，以至于怀疑它与正常点不属于同一个数据分布。

异常检测是一种**用于识别不符合预期行为的异常模式的技术**，又称之为异常值检测。在商业中也有许多应用，如网络入侵检测（识别可能发出黑客攻击的网络流量中的特殊模式）、系统健康性监测、信用卡交易欺诈检测、设备故障检测、风险识别等

#### 问题动机

异常检测主要是运用于非监督学习的算法。问题的引出：通过飞机的检测开始。

> 检测飞机的引擎制造商生产了一批飞机引擎，测试了其中的一些特征变量，比如引擎运转时产生的热量，或者引擎的振动等，假设有m个引擎，$x^{(1)},x^{(2)},…,x^{(m)}$。绘制出如下图表：

![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9mqx90uuxj30zu0jsgvo.jpg)

对于给定的数据集，需要检测$x_{test}$是不是异常的，即这个测试数据不属于这组数据的几率是多少。从上图看出，在蓝色圈内属于该组的概率高，越是偏远的概率，属于该组的可能性就越低。
$$
\text { if } \quad p(x)\left\{\begin{array}{ll}{<\varepsilon} & {\text { anomaly }} \\ {>=\varepsilon} & {\text { normal }}\end{array}\right.
$$
另外两个异常检测的应用例子是

- 识别欺骗行为，通过用户多久登陆一次、访问过的页面、发布帖子的数量等建立模型，通过模型来识别那些不符合该模型的用户。
- 检测数据中心的使用情况：内存使用、被访问的磁盘数量、CPU负载等

#### 高斯分布

高斯分布也叫正态分布。分布满足：
$$
x \sim N\left(\mu, \sigma^{2}\right)
$$
概率密度函数为
$$
p\left(x, \mu, \sigma^{2}\right)=\frac{1}{\sqrt{2 \pi} \sigma} \exp \left(-\frac{(x-\mu)^{2}}{2 \sigma^{2}}\right)
$$
均值$\mu$为
$$
\mu=\frac{1}{m} \sum_{i=1}^{m} x^{(i)}
$$
方差$\sigma^2$为
$$
\sigma^{2}=\frac{1}{m} \sum_{i=1}^{m}\left(x^{(i)}-\mu\right)^{2}
$$
**高斯分布的样例为**

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9mratv2bzj30ye0ja44a.jpg)

当均值$\mu$相同的时候

- 方差的平方越大，图形是矮胖的
- 方差的平方越小，图形是瘦高型的

#### 使用场景

异常检测算法的使用场景一般是三种：

1. 在做特征工程的时候需要**对异常的数据做过滤**，防止对归一化等处理的结果产生影响
2. 对**没有标记输出**的特征数据**做筛选**，找出异常的数据
3. 对有标记输出的特征数据做二分类时，由于某些类别的训练**样本非常少，类别严重不平衡**，此时也可以考虑用非监督的异常点检测算法来做

#### 算法

算法的具体过程是

1. 对于给定的数据集$x^{(1)}, x^{(2)}, \ldots, x^{(m)}$，计算每个特征的$\mu;\sigma^2$
   $$
   \mu_j=\frac{1}{m} \sum_{i=1}^{m} x^{(i)}_j
   $$


$$
\sigma^{2}_j=\frac{1}{m} \sum_{i=1}^{m}\left(x^{(i)}_j-\mu_j\right)^{2}
$$

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9mrpxsqacj30vu0gg48h.jpg)

2. 利用高斯分布进行计算$p(x)$

$$
p(x)=\Pi^n_{j=1}p(x_j;\mu_j;\sigma^2_j)=\Pi^n_{j=1}\frac{1}{\sqrt{2 \pi} \sigma_j} \exp \left(-\frac{(x_j-\mu_j)^{2}}{2 \sigma^{2}_j}\right)
$$

3. 两个特征的训练集及特征非部分情况

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9mrvm1gtej31140eadlz.jpg)

4. 三维图表示的是密度函数，$z$轴为根据两个特征的值估计的$p(x)$的值

![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9msq0bk28j30nk0g4k28.jpg)

当 $p(x) > \varepsilon$时候，预测是正常数据， 否则为异常

#### 异常算法的设计

> 当我们开发一个异常检测系统时，从带标记（异常或正常）的数据着手
>
> - 从其中选择一部分正常数据用于构建训练集
> - 然后用剩下的正常数据和异常数据混合的数据构成交叉检验集和测试集。

### 八种无监督异常检测技术

1. 基于统计的异常检测技术
   1. MA滑动平均法
   2. 3—Sigma（拉依达准则）
2. 基于密度的异常检测
3. 基于聚类的异常检测
4. 基于K-Means聚类的异常检测
5. One Class SVM的异常检测
6. Isolation Forest的异常检测
7. PCA+MD的异常检测
8. AutoEncoder异常检测

### 异常检测和监督学习对比

异常检测中采用的也是带标记的数据，和监督学习类似。二者对比为：

| 异常检测                                                     | 监督学习                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 非常少量的正向类（异常数据 $y=1$）,<br />大量的负向类（$y=0$） | 同时有大量的正向类和负向类                                   |
| 许多不同种类的异常，非常难。根据非常 少量的正向类数据来训练算法。 | 有足够多的正向类实例，足够用于训练 算法，未来遇到的正向类实例可能与训练集中的非常近似。 |
| 未来遇到的异常可能与已掌握的异常、非常的不同。               |                                                              |
| 例如： 欺诈行为检测 生产（例如飞机引擎）检测数据中心的计算机运行状况 | 例如：邮件过滤器 天气预报 肿瘤分类                           |



当**正样本的数量很少**，甚至有时候是0，即出现了太多没见过的不同的异常类型，对于这些问题，通常应该使用的算法就是异常检测算法。

### 特征选择

异常检测算法是基于高斯分布的。当然不满足高斯分布也能处理，但是最好转成高斯分布。误差分析是特征选择中很重要的点。

有些异常数据可能出现较高的$p(x)$的值，被算法当做是正常数据。通过误差分析，增加新的特征得到新的算法，帮助我们更好地进行异常检测。

![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9msjnck6dj315e0n8k4a.jpg)

新特征获取：通过原有特征进行组合，得到新的特征

### 参考资料

1. 李航-统计学习方法
2. [八种无监督异常检测技术](https://www.csuldw.com/2019/03/24/2019-03-24-anomaly-detection-introduction/)

