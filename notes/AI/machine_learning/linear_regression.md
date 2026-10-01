---
title: "线性回归"
date: 2026-09-05
tags: [ML, 线性回归, AI]
---

# 线性回归 Linear Regression

最简单的机器学习模型，用于回归问题，下面以单特征的线性回归为例

从神经网络的视角来看，单特征的线性回归就是一个无隐藏层的网络，输出层只有一个神经元

## 输出

对于一条有d个特征的样本 $x \in \R^d$，对应一个同样d维权重向量

$$\hat y = w^T x + b$$

- $w$和$x$都是向量，$b$为标量
- 结果$\hat y$是标量

如果把n条样本（dataloader里n取batch_size，dataset里是总样本数）并为一个矩阵 $X \in \R^{n \times d}$，每个行向量是一个样本

$$\hat y = X w + b$$

- $b$数学上是向量，但是往往是标量广播得到
- 结果$\hat y \in \R^n$是向量

## 计算损失

和[[感知器]]中的用于分类的离散的输出不同，线性回归的输出是**连续的**。所以前者我们用正确的比例来衡量模型的准确性，后者需要一个[[损失函数]]

所有的损失函数，都是以模型的权重为自变量，以数据集为参数的函数，输出结果是一个反映准确性的标量

在线性回归中，常用的损失函数是均方误差：

$$MSE = \frac{1}{n} \sum (\hat y - y)^2$$

通过损失函数，而不单纯根据结果来更新参数还有一个好处：如果感知器找到了100%正确的分界线，就直接停止更新，而线性回归（或者说，这里是分类就是逻辑回归）会找到泛化能力最好的边界

<figure>
  <img src="/assets/images/machine_learning/linear_sepre_pic.webp" alt="linear_sepre_pic" />
</figure>

感知器在右图就停止更新，而最优化损失函数会让逻辑回归/线性回归走到左图的位置

## 权重更新

紧接着更新权重就变成了一个**最优化问题**，让损失函数取得最小值的权重正确率最高

$$\argmax_{w,b} L(w,b)$$

> 对于线性回归，损失函数MSE是凸函数，上式可以直接解出全局最优解
> 
> $$ w = (X^T X)^{-1} X^T y $$
> 
> 大数据量的情况下，解析方法的开销可能会很大，而优化迭代方法可行性更高。
> 具体推导可见[[Hessian矩阵]]中关于最小二乘的内容。

最通用的优化方法是梯度下降，计算损失函数的负梯度得到最小值的方向

$$ \text{GD:}\quad \theta \leftarrow \theta - \eta \nabla_{\theta} L(\theta) $$

$\theta$ 是模型的所有参数，$\eta$ 是学习率 / learning_rate，$L$ 是损失函数。

如果损失函数是 MSE，可以直接写出解析梯度，比数值方法（有限差分）快得多，而且没有误差（注意梯度是向量）

$$\nabla_w L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i) \, x_i$$

$$\nabla_b L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i)$$

参数的具体更新公式是

$$w \leftarrow w - \eta \cdot \nabla_w L$$

$$b \leftarrow b - \eta \cdot \nabla_b L$$

### 矩阵的情况

约定样本矩阵 $\mathbf{X}\in\mathbb{R}^{N\times d}$，标签 $\mathbf{y}\in\mathbb{R}^N$，权重 $\mathbf{w}\in\mathbb{R}^d$
 
$$L = \|\mathbf{X}\mathbf{w} - \mathbf{y}\|^2 = (\mathbf{X}\mathbf{w} - \mathbf{y})^\mathsf{T}(\mathbf{X}\mathbf{w} - \mathbf{y})$$

逐元素写出

$$L = \sum_{i=1}^N\left(\sum_{j=1}^d X_{ij}w_j - y_i\right)^2$$

对分量 $w_k$ 求偏导

$$\frac{\partial L}{\partial w_k} = 2\sum_{i=1}^N \left(\sum_j X_{ij}w_j - y_i\right) X_{ik}$$

把所有偏导堆成列向量

$$\frac{\partial L}{\partial \mathbf{w}} = 2\mathbf{X}^\mathsf{T}(\mathbf{X}\mathbf{w} - \mathbf{y}) \in \mathbb{R}^d$$

这就是写成矩阵形式的梯度

## 特征标准化

梯度下降要求各特征的数值范围相近，但是得到的数据，各个特征的尺度很可能不同

比如在泰坦尼克数据集里，船舱等级 `Pclass` (1–3) 和 年龄 `Age` (0–80) 相差几十倍，而根据MSE损失下梯度的解析式

$$\nabla_w L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i) \, x_i$$

可以看到梯度大小和特征的值是正相关的

假设学习率$\eta$是固定的，如果$\eta$对于 `Pclass` 来说合适，更新的幅度对于 `Age` 来说就会非常大；如果对于 `Age` 合适，更新的幅度对于 `Pclass` 来说就会非常小

所以大尺度特征会主导梯度，导致收敛缓慢甚至发散

这时候需要对数据进行标准化处理，把每个特征缩放到均值 0、标准差 1

$$ x' = \frac{x - \mu}{\sigma} $$

## 参考资料

1. [动手学深度学习 线性回归](https://zh-v2.d2l.ai/chapter_linear-networks/linear-regression.html)

## 代码实现

根据动手学深度学习的线性回归例子简单实现了一下，玩具性质的。使用PyTorch框架而不是自己动手写

```py
import torch
from torch import nn, optim

N = 1000
BATCH_SIZE = 10
LR = 0.01
EPOCHS = 5

true_w = torch.tensor([2.0, -3.4])
true_b = 4.2
device = torch.device("cuda")


class LinearData(torch.utils.data.Dataset):
    def __init__(self, w, b, N):
        super().__init__()
        self.features, self.labels = self.data_generation(w, b, N)
    def __getitem__(self, index):
        return self.features[index], self.labels[index]
    def __len__(self):
        return self.features.shape[0]

    @staticmethod
    def data_generation(w, b, N):
        """
        return tuple (X,y)
        X: tensor shaped (N,d), sample matrix
        y: tensor shaped (N), label vector
        which N is num of samples, d is num of features
        using y = Xw + b + epsilon
        """
        X = torch.normal(0, 1, (N, len(w)))  # real data
        y = X @ w + b  # real label
        y += torch.normal(0, 0.1, y.shape)  # +noise
        return X, y.reshape((-1, 1))  # nn.linear输出(N,1)


class DataLoader(torch.utils.data.DataLoader):
    def __init__(self, dataset, batch_size, shuffle=True):
        super().__init__(dataset=dataset, batch_size=batch_size, shuffle=shuffle)


class LinearModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Linear(2, 1)

    def forward(self, x):
        x = self.model(x)
        return x


dataset = LinearData(true_w, true_b, N)
data_loader = DataLoader(dataset, batch_size=BATCH_SIZE)

model = LinearModel().to(device)
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), LR)

for epoch in range(EPOCHS):
    cur_loss = 0
    for X, y in data_loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        y_pred = model(X)
        loss = criterion(y_pred, y)
        loss.backward()
        optimizer.step()

        cur_loss += loss.item() * BATCH_SIZE
    print(f"Epochs:{epoch + 1}, Loss:{(cur_loss / N):.3f}")

for name, param in model.named_parameters():
    print(f"{name}:\n{param.detach().cpu()}")
```