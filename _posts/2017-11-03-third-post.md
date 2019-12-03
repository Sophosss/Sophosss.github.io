---
title: 神经网络几个要点
author: Sophosss
layout: post
---
### Neural Networks

神经网络是有史以来发明的最美丽的编程范例之一。在传统的编程方法中，我们告诉计算机该做什么，将大问题分解成计算机可以轻松执行的许多小而精确定义的任务。相比之下，在神经网络中，我们不需要告诉计算机如何解决我们的问题，它会从数据中学习，找出自己解决手头问题的方法。

下面介绍一些基础步骤要点和一些技巧。

- 数据预处理

  均值减法： `X -= np.mean(X, axis = 0)`

  归一化： `X /= np.std(X, axis = 0)`

- 权重初始化

  不推荐使用全零初始化，应使用小随机数初始化，提高收敛速度。建议将神经元权重向量初始化为如下所示，其中n为输入数据量：

  $\omega$ = $$\frac{np.random.randn(n)}{sqrt(n)}$$

  常用的激活函数有sigmoid：将实数压缩到[0,1]，tanh：将实数压缩到[-1,1]，这两个都存在饱和问题。此外还有relu：&fnof;(x) = max(0,x)，当神经元为relu时推荐如下所示：

  $\omega$ = np.random.randn(n) * sqrt($$\frac{2.0}{n}$$)}

- 批量初始化

  Batch Normalization的目的是让激活数据在训练开始前通过一个网络，网络处理数据使其服从标准高斯分布：

  ![theme](https://Sophosss.github.io/assets/images/5.png)

- 正则化
  1. L1正则化：$\lambda$$\omega$
  2. L2正则化：$\frac {1}{2}$$\lambda$$\omega$<sup>2</sup>
  3. 最大范式约束：参数更新方式不变，满足w<sub>2</sub> < c，c一般为3或4。
  4. 随机失活：dropout

通过交叉验证获得一个全局使用的L2正则化强度，与此同时在所层后面使用随机失活也很常见。

偏差和权重合并技巧：&fnof;(x<sub>i</sub> , $\omega$) = $\omega$x<sub>i</sub> ，即x<sub>i</sub> 增加一个维度，数值为常量1。

```
1 2 3   11   7         1 2 3 7   11
4 5 6   12 + 8   <->   4 5 6 8   12
7 8 9   13   9         7 8 9 9   13
                                 1
  w     x    b            w      x
```

一个三层的神经网络可以类比看作S = $\omega$<sub>3</sub>max(0, $\omega$<sub>2</sub>max(0,$\omega$<sub>1</sub>))，参数$\omega$<sub>1</sub>$\omega$<sub>2</sub>$\omega$<sub>3</sub>将通过随机梯度下降学习到，中间隐含层尺寸是网络的超参数。