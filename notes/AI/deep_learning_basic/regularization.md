---
title: "正则化"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 正则化

## 过拟合

出现

- 训练集上的准确率远大于测试集
- 训练集loss下降，但是某个时刻之后验证集的loss开始上升

即为过拟合，一般是由于训练集的数据不足而模型参数量太大，使得模型倾向于记忆训练集（优化的目标是训练集上的loss，一个极端的例子就是模型直接对训练集打表，训练集上loss为0，但是无泛化能力），泛化能力差

广义的正则化regularization就是解决模型过拟合的方法，让模型学习更简单的模式

## L1/L2正则化

在损失函数中添加与参数 $w$ 的大小相关的惩罚项，从而尽量得到简单的模型。分别指衡量参数向量 $w$ 大小的L1/L2范数（见[[矩阵理论]]）

下面的 $\lambda$ 是影响正则化强度的参数

### L1

$$L_1(w) = L(w) + \lambda ||w||_1 = L(w) + \lambda \sum_i |w_i|$$

参数更新规则是

$$w \leftarrow w - \eta \nabla L(w) - \eta \lambda \, \text{sign}(w)$$

参数受到一个固定大小的向零方向的量 $\eta \lambda$，L1正则化倾向于产生稀疏参数

### L2

$$L_2(w) = L(w) + \frac{\lambda}{2} ||w||^2_2 = L(w) + \frac{\lambda}{2} \sum_i w_i^2$$

新损失函数的梯度是

$$\nabla L_2(w) = \nabla L(w) + \lambda w$$

参数更新规则是

$$w \leftarrow w - \eta \nabla L(w) - \eta \lambda w = (1 - \eta \lambda)w - \eta \nabla L(w)$$

旧参数乘以了小于1的因子 $(1 - \eta \lambda)$ 整体地向0收缩，而且L2项产生的梯度和参数的大小相关，因而L2正则化倾向于产生均匀缩小的参数

加上可导的原因，L2正则化更为常用

## 权重衰减

在参数更新时，把旧参数乘以一个小于1的因子，让权重自然缩小

$$w \leftarrow (1 - \eta \lambda)w - \eta \nabla L(w)$$

$\lambda$ 为权重衰减系数，通常取1e-4

### 和L2正则化

用标准的梯度下降更新参数（比如不带动量的SGD）时，L2正则化和权重衰减是等价的，两者的参数更新公式都是 $w \leftarrow (1 - \eta \lambda)w - \eta \nabla L(w)$

所以，在PyTorch中，L2正则化和权重衰减的实现方式是一致的。例如在 `torch.optim.SGD` 源码中，`weight_decay` 参数直接在求出原始梯度后修改 `grad = grad + weight_decay * param`（也就是添加L2惩罚项的梯度）

之所以这么做，是因为梯度加法是优化器统一处理所有惩罚项（如L1、L2）的标准接口，代码复用性更好

然而对于不按标准梯度下降更新的优化器，比如动量法、Adam等自适应优化器，两者就不等价了，可见[[优化器-自适应学习率]]

### 代码

对于PyTorch，使用L2正则化最简单的方法是通过优化器的权重衰减

```py
import torch.optim as optim

# 定义优化器时设置 weight_decay
optimizer = optim.SGD(model.parameters(), lr=0.01, weight_decay=1e-4)
# 或者使用 AdamW
optimizer2 = optim.AdamW(model.parameters(), lr=0.001, weight_decay=1e-4)
```

对于Adam等不等价的优化器或者实现L1正则化，可以手动修改损失函数

```py
import torch

# 假设 model 是你的模型，criterion 是损失函数
l2_lambda = 0.001
l2_reg = torch.tensor(0., requires_grad=True)

# 遍历模型参数，计算其平方和
for param in model.parameters():
    l2_reg = l2_reg + torch.norm(param, p=2) ** 2

# 总损失 = 原始损失 + 正则化项
loss = criterion(output, target) + l2_lambda * l2_reg

# 然后正常进行反向传播
loss.backward()
optimizer.step()
```

## Dropout

随机让一些神经元在本次前向传播时置0，降低神经元之间的依赖

### 代码

Dropout层添加在激活函数之后，参数p是神经元被置零的概率，默认0.5

- 对于MLP，建议取0.5，输入层需要略小以保持原本信息
- 对于transformer，建议~0.1
- CNN不建议使用，因为CNN本身就需要局部相关性

```py
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(3072, 512),
            nn.ReLU(),
            nn.Dropout(0.3),

            nn.Linear(512, 256),
            nn.ReLU(),
            nn.Dropout(0.4),

            nn.Linear(256, 10),
            # 不加softmax，因为交叉熵损失自带
        )
```

在训练时，因为部分神经元置零会影响输出的分布，所以pytorch会对剩余神经元的输出乘以 $\frac{1}{1-p}$；推理时，Dropout层不会起作用，变为恒等输出

## 早停

在训练的epoch主循环中评估验证集的loss，如果patience个epoch后还没有改善，就停止训练

没有修改模型本身，但限制了训练步数，可探索的参数空间更小，等价于限制模型复杂度，所以也称作隐式正则化

## 数据增强

对现有的数据进行变换（翻转，裁切，颜色抖动）相当于扩大了数据量，来解决过拟合问题