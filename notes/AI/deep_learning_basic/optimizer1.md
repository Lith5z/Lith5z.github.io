---
title: "优化器 SGD"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 梯度下降 Gradient Descent

$$\text{GD:}\quad \Delta \theta = - \eta \cdot \nabla L(\theta)$$

## 批量

全批量的梯度下降Batch Gradient Descent/BGD，在每一个epoch中计算所有数据得到梯度，然后更新一次参数

结果是绝对准确的，如果损失函数是个下凸函数，就一定能收敛到全局最优解

但是在深度学习中数据量和参数量都很大，这样做的开销太大，不够现实

另一种做法是随机梯度下降Stochastic Gradient Descent/SGD，只根据一条样本计算梯度，然后就更新参数

一条样本的梯度显然不能代表完整的数据集，这种偏差被称作“噪声”，有利于跳出局部最优，但是不够稳定；而且这样不能很好地利用现代GPU的并行特性

综合得到的方法是小批量梯度下降Mini-batch Gradient Descent，在每一个epoch中，把数据集分为许多大小为32、64、128等大小的batch来计算梯度，每一个batch更新一次参数

这种方法最为可行。事实上**一般提到SGD，说的就是小批量梯度下降**，而且AdamW等优化器也是分batch计算梯度的

## 学习率

既然梯度本身也有数值大小，为什么还需要乘以学习率？

这是“梯度决定方向，学习率决定步长”这句话误导人的地方

梯度作为一个向量，这个向量的反方向是下降最快的方向，这是没有问题的；但是梯度的每一个分量都是偏导数，梯度的模一般都不是1，真正的步长是 $\eta ||\nabla_{\theta}L(\theta)||$ 的值

学习率只是一个**缩放因子**，避免 $||\nabla_{\theta}L(\theta)||$ 的值对于优化过程太大，导致无穷发散或者震荡

## 一阶优化

对损失函数进行一阶泰勒展开

$$\Delta L \approx \frac{\partial L}{\partial \theta} \cdot \Delta \theta$$

然后把梯度下降得到的 $\Delta \theta$ 带入损失函数，就得到

$$\Delta L \approx -\eta \cdot \left( \frac{\partial L}{\partial \theta} \right)^2$$

这说明损失函数的下降幅度和梯度的**平方**成正比，也说明梯度下降只利用损失函数的一阶曲率信息

牛顿法这样的优化算法可以利用二阶曲率，理论上收敛会比梯度下降快很多，但是由于牛顿法需要计算黑塞矩阵，在参数多的时候代价太大了

## 动量法 Momentum

$$v_{t+1} = \beta v_t + \nabla L(\theta_t) $$

$$\Delta \theta = - \eta \cdot v_{t+1}$$

其中 $\beta$ 是动量衰减系数/摩擦系数，一般取0.9

$v_t$ 可以理解为速度，实际上是梯度**指数加权后的滑动平均**，假设初始速度 $v_0 = 0$，逐层展开一式就是：

$$v_{t+1} = -\eta \left( \nabla L(\theta_t) + \beta \nabla L(\theta_{t-1}) + \beta^2 \nabla L(\theta_{t-2}) + \cdots + \beta^{t} \nabla L(\theta_0) \right)$$

所以当前更新方向 $v_{t+1}$ 由当前梯度与所有历史梯度的共同决定，其中近期梯度权重最大，远期权重趋近于零

- 加速收敛：如果近期梯度方向一致，就会叠加
- 抑制震荡：近期梯度方向的正负项会相互抵消
- 理论上两倍于模型规模的显存开销：普通SGD只需要存储参数，动量法需要保存每个参数的历史梯度 $v_t$（不考虑参数的瞬时梯度，后同）

在CNN网络的训练中，SGD+Momentum的组合可能比AdamW的效果更好