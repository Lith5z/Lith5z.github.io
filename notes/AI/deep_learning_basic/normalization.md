---
title: "归一化"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 归一化

## 为什么

[[参数初始化]]只能保持训练的第一个epoch的层间激活值和梯度的稳定，训练开始之后参数进行更新，导致初始化起作用的前提条件 $\text{Var}(W) = \begin{cases} \frac{1}{n}, & \text{Xavier} \\ \frac{2}{n}, & \text{Kaiming} \end{cases}$ 不再成立，数据的分布会逐渐漂移，且随着训练的进行漂移幅度增大

在[[参数初始化]]中讲解过分布漂移的影响，这里不再赘述

对数据进行归一化，虽然会改变数据的尺度，但是因为权重本身可以调节，而且神经网络学习主要是数据的组合模式，所以对学习结果不会有明显负面影响

> 深度学习语境下的归一化其实就是标准化。
> 统计学里，归一化Normalization是指Min-Max缩放，也就是对进行 $x' = \frac{x - x_{\text{min}}}{x_\text{max}-x_\text{min}}$ 变化让样本统一到 $[0,1]$ 区间中；标准化Standardization则是 $x' = \frac{x - \mu}{\sigma}$ 得到均值为0、标准差为1的分布。
> 受论文*Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift*的影响，虽然Normalization直译为归一化，在深度学习领域内执行的是统计学的标准化的操作。

## 批归一化 BatchNorm

假设一个mini-batch有128个样本，每个样本有256个特征，批归一化就是对每个特征列的所有样本进行的（如果形状为128,256那么就是竖向的）；推理时一次只有一个样本，通过维护滑动平均值进行

$$\hat{x}_j = \frac{x_j - \mu_j}{\sqrt{\sigma_j^2 + \epsilon}}$$

然后进行缩放和平移，这里的参数 $\beta,\gamma$ 也是可学习的

$$y_j = \gamma_j \cdot \hat{x}_j + \beta_j$$

用于CNN和MLP

## 层归一化 LayerNorm

对于NLP之类的序列任务

- 序列长度不固定，组成一个batch的时候往往要对短序列进行填充（例如用[PAD]），如果使用BN，一个特征列下的无意义token会影响有意义的数据
- 对于BN来说，batch_size越小得到的均值方差越受噪声影响，而LN的计算是对各个样本进行，不依赖batch_size，适合变长序列和分布式训练
- BN在推理和训练时的行为不一致

```plain
mini-batch：
            ---------> LN方向：每个样本的所有特征
样本1: [a, b, c] -> LN计算均值、方差
样本2: [d, e, f] -> LN计算均值、方差
样本3: [g, h, i] -> LN计算均值、方差
        |  |  |
   |    V  V  V
   |   BN计算均值、方差
   |
   V  BN方向：每个特征的所有样本
```

具体计算方法类似BN

## RMSNorm

在LN基础上改动了计算方式，改为除以均方根RMS，并去掉均值平移和参数 $\beta$

$$y = \frac{x}{\text{RMS}(x)} \odot \gamma$$

$$\text{RMS(x)} = \sqrt{\frac{1}{d} \sum_{i=1}^d x_i^2 + \epsilon}$$

大大减小了计算量，基本不损失性能，成为LLM的首选

## 代码

无论是BN、LN还是RMSNorm都是对预激活值进行归一化

这里调用一维的BN，因为一个样本linear层的输出是一个一维向量

```py
def make_deep_mlp_bn(depth=6, width=256, in_dim=784, out_dim=10):
    layers = [nn.Flatten()]
    last = in_dim
    for _ in range(depth):
        linear = nn.Linear(last, width)
        nn.init.kaiming_normal_(linear.weight, nonlinearity='relu')
        nn.init.zeros_(linear.bias)
        layers += [linear, nn.BatchNorm1d(width), nn.ReLU()]
        last = width
    layers.append(nn.Linear(last, out_dim))
    return nn.Sequential(*layers)
```

## 扩展阅读

1. [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167)
2. [Layer Normalization](https://arxiv.org/abs/1607.06450)