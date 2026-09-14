---
title: "参数初始化"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 参数初始化

## 为什么

如果所有参数都初始化为0，则每一层的神经元都会给出相同的输出，反向传播时每一层的梯度也一样，一层网络相当于退化为一个神经元

所以应当随机初始化参数，但是我们希望初始化后的网络，能让数据在传播过程中保持稳定，也就是数据均值和方差稳定；否则可能会造成饱和（对于simgoid如果激活值过大）、训练不稳定（梯度过大）、震荡（不同特征尺度差异让损失函数的等高线从圆形变成狭长椭圆，梯度方向偏离最优方向，优化器来回震荡）等问题

有两种常用的初始化手段

## Xavier初始化

约定：

- 考虑一个使用对称激活函数 $f$ 的密集神经网络，且满足 $f'(0)=1$
- 假设在初始化阶段，网络处于线性工作区，即 $f'(s) \approx 1$
- 权重独立采样，均值为0，且输入特征同分布，方差记为 $Var[x]$

具体推导可见[[Xavier初始化]]论文阅读笔记，下面是极简版的推导

考虑没有激活函数和偏置的线性层，输出神经元 $y=\sum_{i=1}^n w_i x_i$，假设 $x_i$ 和 $w_i$ 独立且均值为0，那么对于正向传播过程

$$\text{Var}(y) = n \cdot \text{Var}(w) \text{Var}(x)$$

为了维持输入输出的方差稳定即 $\text{Var}(y) = \text{Var}(x)$ 得到

$$\text{Var}(w) = \frac{1}{n} \quad (n = \text{fan}_\text{in})$$

对于反向传播过程同理，结果为$\text{Var}(w) = \frac{1}{\text{fan}_\text{out}}$

由于两者不可能同时满足（除非每层的宽度都一致），取调和平均数就得到了

$$\text{Var}(W) = \frac{2}{\text{fan}_\text{in}+\text{fan}_\text{out}}$$

即Xavier初始化

适用于sigmoid/tanh等激活函数

## Kaiming初始化

ReLU激活函数不再是对称的，在0处不满足线性假设，Xavier初始化推导的约定假设不成立。从直观上看，ReLU在负区间的值为0，相当于减少了一半的方差

所以Kaiming初始化将减少的方差补上，重新推导得到

$$\text{Var}(X) = \frac{2}{\text{fan}_\text{in}} = \frac{2}{\text{fan}_\text{out}}$$

这里无论是采用扇入还是扇出，结果都是一样的，都能使网络收敛。作者在论文中统一使用fan_in

## 关于Pytorch

见pytorch笔记nn.init章节，下面是大概内容

PyTorch 对 `nn.Linear` 和 `nn.Conv2d` 的初始化使用 `kaiming_uniform_(weight, a=√5)`，等价于从均匀分布 $U[-\frac{1}{\text{fan}_\text{in}},\frac{1}{\text{fan}_\text{in}}]$ 中采样

这种默认方式并非针对 ReLU 的标准 Kaiming 初始化——标准的做法需要额外乘上增益因子 √2，而 PyTorch 默认没有这一步，导致权重的标准差只有标准 Kaiming 初始化的 40% 左右

对于浅层网络这个差异影响不大，但在深层网络中可能引发梯度消失问题，需要手动修正

> 具体原因可见[gitub issue](https://github.com/pytorch/pytorch/issues/57109)或者我整理的笔记[[为什么pytorch默认初始化不是标准kaiming_260816]]。

手动初始化时，核心概念是增益因子gain，不同的激活函数需要不同的增益值，可以通过 `calculate_gain(nonlinearity, param)` 来查询推荐数值

## 扩展阅读

1. [Xavier初始化原文](https://proceedings.mlr.press/v9/glorot10a/glorot10a.pdf)
2. [Kaiming初始化原文](https://arxiv.org/abs/1502.01852)
3. [知乎 Xavier初始化论文阅读](https://zhuanlan.zhihu.com/p/1993864135151227307)
4. [知乎 Kaiming初始化论文阅读](https://zhuanlan.zhihu.com/p/1999932426844125029)
5. [PyTorch文档 nn.init](https://docs.pytorch.org/docs/2.13/nn.init.html)