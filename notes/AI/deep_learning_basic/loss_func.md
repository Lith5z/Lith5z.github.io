---
title: "损失函数"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 损失函数

没有衡量模型

梯度推导部分，均假设 $\hat y = f(z)$，$f$ 为某个[[激活函数]]

## 均方差损失 MSE Loss

$$\text{MSE:} \quad L = \frac{1}{n} \sum_i (\hat y_i - y_i)^2$$

用于回归任务

### 梯度推导

$$\frac{\partial L}{\partial z} = \frac{\partial L}{\partial \hat y} \, \frac{\partial \hat y}{\partial z} = \frac{2}{n}(\hat y - y) f'(z)$$

线性回归使用sigmoid作为激活函数，梯度推导可见[[线性回归]]

### 代码

```py
def mse_loss(y_true, y_pred):
    """
    y_true: 真实值，形状 (n,) n = num of sample
    y_pred: 预测值，形状相同
    返回: 标量（平均损失）
    """
    return np.mean((y_true - y_pred) ** 2) # 向量化写法

# 如果使用pytorch，则直接使用
criterion = nn.MSELoss()
loss = criterion(pred, target)
```

## 交叉熵损失 Cross Entropy Loss

对于多分类问题，如果有 $K$ 个类别

$$\text{CE:} \quad L = -\frac{1}{N} \sum_i^N y_i \cdot \log(\hat y_i)$$

这是向量写法，$i$ 遍历样本，其中

- $\hat y = f(z)$ 其中 $z$ 是模型原始输出logits向量
- $\hat y = [\hat y_i , \cdots, \hat y_K]$ 是每个标签的预测概率构成的向量
- $y_i = [y_1, \cdots, y_K]$ 是样本的真实标签向量，这是一个独热one-hot向量，有且仅有真实类别 $y = 1$，其余为0
- 向量点乘

如果写成逐元素写法，即

$$L = -\frac{1}{N} \sum_{i=1}^N \sum_{c=1}^K y_{i,c} \log(\hat y_{i,c})$$

对于二分类问题，只有两个类别，通常令正类标签为1，负类标签为0，上式退化为

$$\text{BCE:} \quad L = -\frac{1}{N} \sum_i^N [y_i \log(\hat y) + (1-y_i) \log(1 - \hat y)]$$

求和后只剩真实类别那一项，和多分类相同

用于分类任务

### 梯度推导

可见[[逻辑回归]]，这里简略带过二分类的

$$\frac{\partial L}{\partial z} = \frac{\partial L}{\partial \hat y} \, \frac{\partial \hat y}{\partial z} = \frac{\hat y_i - y_i}{\hat y_i (1-\hat y_i)} f'(z)$$

### 代码

二分类

```py
def binary_cross_entropy(y_true, y_pred):
    """
    y_true: 真实标签 0 或 1，形状 (n,)
    y_pred: 预测概率，形状 (n,)，范围 (0,1)
    返回: 标量（平均损失）
    """
    # 防止 log(0) 产生 -inf
    eps = 1e-15
    y_pred = np.clip(y_pred, eps, 1 - eps)
    
    loss = - (y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred))
    return np.mean(loss)

# 如果使用pytorch，则直接使用
criterion = nn.BCEWithLogitsLoss()
loss = criterion(logits, target.float())
```

多分类

```py
def cross_entropy_loss(y_true, y_pred):
    """
    y_true: 整数标签，形状 (n,)，每个值在 [0, K-1]
    y_pred: 每行是样本的类别概率向量，形状 (n, K)
    返回: 标量（平均损失）
    """
    n = y_true.shape[0]
    eps = 1e-15
    y_pred = np.clip(y_pred, eps, 1 - eps)
    
    # 取每个样本真实类别对应的预测概率
    correct_class_probs = y_pred[np.arange(n), y_true] # 花哨索引
    return - np.mean(np.log(correct_class_probs))

# 如果使用pytorch，则直接使用
criterion = nn.CrossEntropyLoss()
loss = criterion(logits, target.long())
```

特别注意，**在使用pytorch的交叉熵时，输入的是logits而不是概率**，所以定义模型结构的时候**最后一层不需要添加**`nn.Softmax(dim=-1)`