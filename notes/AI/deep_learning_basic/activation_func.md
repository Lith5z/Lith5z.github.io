---
title: "激活函数"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 激活函数

要求：

- 非线性的
- 连续，只有有限个点不可导
- 保证层数增多后梯度稳定：导数的绝对值不能过大，也不能过小

英文术语：

- 预激活值pre-activation value：激活函数的输入值
- 激活值activation value：激活函数的输出值

## Sigmoid

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

导数是

$$ \sigma'(x) = \frac{-e^{-z}}{(1+e^{-z})^2} = \frac{1-(1+e^{-z})}{(1+e^{-z})^2} = (\frac{1}{1+e^{-z}} - 1) \cdot \frac{1}{1 + e^{-z}} \\ = \sigma(z) (1 - \sigma(z))$$

<figure>
  <img src="/assets/images/machine_learning/sigmoid_pic.webp" alt="sigmoid_pic" />
</figure>

优点：

- 输出范围是 $(0,1)$，符合概率分布，用于输出

缺点：

- 指数的计算量
- 在饱和时导数值非常小，网络层数深容易出现梯度消失到0

### Softmax

常用于多分类的输出，其中 $K$ 为输出类别的总数，输入值 $x_i$ 又称作Logits

$$\text{Softmax}(x_i) = \frac{e^{x_i}}{\sum_{j=1}^K e^{x_j}}$$

---

记函数为 $S_i$，导数计算如下：

由于这是个多变量函数，对哪个变量求导会影响分子的导数，所以分类讨论

**若 $i=j$**

$$
\begin{aligned}
\frac{\partial S_i}{\partial x_i}
&= \frac{e^{x_i} \cdot \sum_{k=1}^K e^{x_k} - e^{x_i} \cdot e^{x_i}}{\big(\sum_{k=1}^K e^{x_k}\big)^2} \\
&= \frac{e^{x_i}}{\sum_{k=1}^K e^{x_k}} \cdot \frac{\sum_{k=1}^K e^{x_k} - e^{x_i}}{\sum_{k=1}^K e^{x_k}} \\
&= S_i (1 - S_i)
\end{aligned}
$$

**若 $i \neq j$**

$$
\begin{aligned}
\frac{\partial S_i}{\partial x_j}
&= \frac{0 \cdot \sum_{k=1}^K e^{x_k} - e^{x_i} \cdot e^{x_j}}{\big(\sum_{k=1}^K e^{x_k}\big)^2} \\
&= -\frac{e^{x_i}}{\sum_{k=1}^K e^{x_k}} \cdot \frac{e^{x_j}}{\sum_{k=1}^K e^{x_k}} \\
&= -S_i S_j
\end{aligned}
$$

综上，记 $\delta_{ij} = \begin{cases}1, & i = j \\ 0, & i \neq j \end{cases}$ 统一写成

$$
\frac{\partial S_i}{\partial x_j} = S_i (\delta_{ij} - S_j)
$$

---

特点：

- 较大的Logits占据绝对主导，尽管40和50差别不大，但是输出值差距巨大

一般来说，还会带一个温度参数 $T$

$$\text{Softmax}(x_i,T) = \frac{e^{x_i/T}}{\sum_{j=1}^K e^{x_j/T}}$$

- $T$ 增大时，所有输入值都更加平均，输出的概率分布更加均匀
- $T$ 减少时，概率向最大的Logits对应的类别集中
- $T \to 0$ 时，输出最大的Logits对应的类别，相当于 $argmax$

## ReLU

线性整流函数Rectified Linear Unit

$$\text{ReLU}(x) = \max(0,x)$$

导数是

$$\text{ReLU}'(x) = \begin{cases}1, & x \gt 0 \\ 0, & x \leq 0 \end{cases}$$

<figure>
  <img src="/assets/images/deep_learning_basic/ReLU_pic.webp" alt="ReLU" />
</figure>

> ReLU在x=0处不可导，pytorch人为定义了这一点的导数是0，因而给出上面的公式。实际上，因为是浮点数运算，一个神经元输入正好等于0的情况很小，这个人为定义的影响可以忽略。

优点：

- 正半轴的导数是1，缓解梯度消失
- 计算简单

缺点：

- 如果某次更新后 $x \lt 0$，那么这个神经元的导数将永远是0，无法进行更新，相当于失活

## GeLU

$$\text{GeLU}(x) = x \cdot \Phi(x)$$

其中 $\Phi(x)$ 是标准正态分布的累积分布函数，图像和sigmoid类似

<figure>
  <img src="/assets/images/deep_learning_basic/GeLU_pic.webp" alt="GeLU" />
</figure>

在2023年之前的大模型中GELU已经成为事实标准

> pytorch提供了一个参数`approximate`用于决定GeLU的计算，默认设置为`'none'`会调用erf计算，更精确；设置为`'tanh'`会近似计算，速度快很多。很多预训练Transformer模型（如 GPT）就采用了这种近似。

## Swish/SiLU

$$\text{SiLU}(x) = x \cdot \sigma(x)$$

<figure>
  <img src="/assets/images/deep_learning_basic/SiLU_pic.webp" alt="SiLU" />
</figure>