---
title: "逻辑回归"
date: 2026-09-05
tags: [ML, 逻辑回归, AI]
---

# 逻辑回归 Logistic Regression

多分类逻辑回归又称softmax回归

在[[线性回归]]的基础之上增加一个softmax函数（二分类退化为sigmoid），并换用交叉熵作为损失函数

从神经网络的视角来看，逻辑回归就是一个无隐藏层的网络，输出层的神经元数量和类别数相等

<figure>
  <img src="/assets/images/machine_learning/NN_softmax_pic.webp" alt="NN_softmax_pic" />
</figure>

## 输出

对于其中的一个类别来说，就相当于线性回归，原始输出的范围是任意的，称为logits

要将得到分类的概率，我们必须保证输出非负的且总和为1，这就需要把原始输出 $z$ 映射到概率的区间 $[0,1]$ 上，这一步使用softmax

对于**一条样本** $x\in\mathbb{R}^{1\times d}$，多分类逻辑回归先计算所有类别的 logits

$$z=xW+b$$

- 权重矩阵 $W\in\mathbb{R}^{d\times K}$（理解为从特征到类别的线性变换）
- 每个输出神经元都有自己的偏置，故**偏置是向量** $b\in\mathbb{R}^{1\times K}$
- 因此 $z\in\mathbb{R}^{1\times K}$，第 $k$ 个元素 $z_k$ 就是第 $k$ 类的 logit

经过 softmax 后，第 $k$ 类的概率为

$$\hat y_k=\frac{e^{z_k}}{\sum_{j=1}^K e^{z_j}}$$

如果写成矩阵形式，那么**一个 batch** 组成的样本 $X \in \mathbb{R}^{n \times d}$

$$Z = XW + b$$

$$\hat Y = \text{softmax}(Z)$$

- 因为 $XW \in \mathbb{R}^{n \times K}$，这里会在维度0上对 $b$ 进行广播
- $\hat Y \in \mathbb{R}^{n \times K}$，每一行是一条样本的各个类别的概率，故 softmax 是逐行进行计算的

## 损失

在线性回归中，线性输出配合MSE损失时，如果以模型参数为自变量，损失函数是凸函数，因此可以通过梯度下降找到全局最优解

逻辑回归使用交叉熵作为损失函数（可见[[似然估计]]），搭配softmax，又得到一个凸函数

设第 $i$ 个样本的真实标签为 one-hot 向量 $\boldsymbol{y}_i$，模型输出的概率向量为 $\hat{\boldsymbol{y}}_i$，则**单个样本**的损失为

$$J_i=-\sum_{k=1}^K y_{ik}\log(\hat y_{ik})$$

- $y_{ik}$ 是第 $i$ 个样本是否属于第 $k$ 类
- $\hat y_{ik}$ 是模型预测第 $i$ 个样本属于第 $k$ 类的概率

因为 one-hot 标签中只有真实类别对应的元素为1，其余元素为0，所以实际上每个样本的损失只保留真实类别对应的一项。假设第 $i$ 个样本属于第 $c$ 类，那么

$$J_i=-\log(\hat y_{ic})$$

预测真实类别的概率 $\hat y_{ic}$ 越接近1，损失越接近0；如果这个概率越接近0，损失会趋于无穷大。因此，交叉熵会迫使模型不仅预测正确，还要对正确类别保持较高的置信度

代入softmax，得到最后的表达式

$$J_i =-\log\left( \frac{e^{z_{ic}}} {\sum_{j=1}^K e^{z_{ij}}} \right) =-z_{ic}+\log\left(\sum_{j=1}^K e^{z_{ij}}\right)$$

可以看到，对于样本 $x_i$ 的正确类 $z_{ic}$，softmax的指数部分和交叉熵的对数部分相互抵消了，只剩下一个logits；第二项则是一个log-sum-exp形式

实际计算时会再结合softmax的平移不变性，可以避免指数运算造成数值溢出

这也是PyTorch中使用逻辑回归时，不需要在模型最后添加`nn.Softmax(dim=-1)`的原因：直接将 logits 传给`nn.CrossEntropyLoss()`即可，该损失函数内部会以数值稳定的方式完成log-softmax和交叉熵的计算

对包含 $n$ 个样本的**batch**，取平均，得到总体损失，其他部分同理

$$J = -\frac{1}{n} \sum_{i=1}^n \sum_{k=1}^K y_{ik} \log(\hat y_{ik})$$

## 多分类的梯度

得到损失函数之后，就可以按着[[线性回归]]的梯度下降去更新参数了

二分类的推导很简单，这里略去，下面重点求多分类的梯度

### 逐元素形式

接下来推导 $\nabla_{w_k} J$，其中 $w_k\in\mathbb{R}^{d\times 1}$ 是 $W$ 的第 $k$ 列

对单个样本 $(x_i, y_i)$，用链式法则。注意损失中每个 $\hat y_{ij}$ 都有贡献，而每个 $\hat y_{ij}$ 又都依赖 $z_{ik}$，所以要加上对 $j$ 的求和：

$$\frac{\partial J_i}{\partial w_k}=\sum_{j=1}^K \frac{\partial J_i}{\partial \hat y_{ij}} \cdot \frac{\partial \hat y_{ij}}{\partial z_{ik}} \cdot \frac{\partial z_{ik}}{\partial w_k}$$

逐项计算，其中

$$\frac{\partial J_i}{\partial \hat y_{ij}} = -\frac{y_{ij}}{\hat y_{ij}}$$

第二项比较复杂，二分类里 sigmoid 的导数是标量 $\hat y(1-\hat y)$，但 softmax 的每个输出 $\hat y_{ij}$ 都通过分母依赖**所有** $z_{i1},\dots,z_{iK}$，所以导数要分 $j=k$ 和 $j \neq k$ 两种情况：

$$\frac{\partial \hat y_{ij}}{\partial z_{ik}} = \begin{cases} \hat y_{ik} (1 - \hat y_{ik}) & j = k \\ -\hat y_{ij} \hat y_{ik} & j \neq k \end{cases}$$

$$\frac{\partial z_{ik}}{\partial w_k} = x_i^T$$

代入导数，把求和拆成 $j=k$ 的一项和 $j \neq k$ 的其余项：

$$\frac{\partial J_i}{\partial w_k}=-\left[ \frac{y_{ik}}{\hat y_{ik}} \cdot \hat y_{ik} (1-\hat y_{ik}) + \sum_{j \neq k} \frac{y_{ij}}{\hat y_{ij}} \cdot (-\hat y_{ij} \hat y_{ik}) \right] x_i^T=-\left[ y_{ik} (1-\hat y_{ik}) - \hat y_{ik} \sum_{j \neq k} y_{ij} \right] x_i^T$$

用 one-hot 和为1的性质 $\sum_{j \neq k} y_{ij} + y_{ik} = 1$ 整理括号里的项：

$$= -\left[ y_{ik} - \hat y_{ik} \left( y_{ik} + \sum_{j \neq k} y_{ij} \right) \right] x_i^T = (\hat y_{ik} - y_{ik})\, x_i^T$$

对所有样本取平均，最后的结果是

$$\nabla_{w_k} J = \frac{1}{n} \sum_{i=1}^n (\hat y_{ik} - y_{ik})\, x_i^T$$

同理可得偏置的梯度：

$$\nabla_{b_k} J = \frac{1}{n} \sum_{i=1}^n (\hat y_{ik} - y_{ik})$$

和二分类的形式完全一样

### 矩阵形式

下面的内容改编自[知乎-矩阵求导术](https://zhuanlan.zhihu.com/p/24709748)，非常值得一看；或者笔记[[矩阵微积分]]中也要大致摘要

$J_i = -\boldsymbol{y}_i^T\log\text{softmax}(W^T\boldsymbol{x}_i)$，求 $\frac{\partial J_i}{\partial W}$

其中，令 $\boldsymbol{x}_i=x_i^T\in\mathbb{R}^{d\times 1}$，$\boldsymbol{y}_i=y_i^T\in\mathbb{R}^{K\times 1}$，它们分别是第 $i$ 个样本和标签的列向量；$W\in\mathbb{R}^{d\times K}$，$J_i$ 是标量。这里 $\text{softmax}(\boldsymbol{a}) = \frac{\exp(\boldsymbol{a})}{\boldsymbol{1}^T\exp(\boldsymbol{a})}$，其中 $\exp(\boldsymbol{a})$ 表示逐元素求指数，$\boldsymbol{1}$ 代表全1向量

代入softmax

$$J_i = -\boldsymbol{y}_i^T \left(\log (\exp(W^T\boldsymbol{x}_i))-\boldsymbol{1}\log(\boldsymbol{1}^T\exp(W^T\boldsymbol{x}_i))\right) = -\boldsymbol{y}_i^TW^T\boldsymbol{x}_i + \log(\boldsymbol{1}^T\exp(W^T\boldsymbol{x}_i))$$

注意逐元素log满足等式 $\log(\boldsymbol{u}/c) = \log(\boldsymbol{u}) - \boldsymbol{1}\log(c)$，以及 $\boldsymbol{y}_i$ 满足 $\boldsymbol{y}_i^T \boldsymbol{1} = 1$

求微分

$$dJ_i =- \boldsymbol{y}_i^TdW^T\boldsymbol{x}_i+\frac{\boldsymbol{1}^T\left(\exp(W^T\boldsymbol{x}_i)\odot(dW^T\boldsymbol{x}_i)\right)}{\boldsymbol{1}^T\exp(W^T\boldsymbol{x}_i)}$$

再套上迹并做交换，注意可化简 $\boldsymbol{1}^T\left(\exp(W^T\boldsymbol{x}_i)\odot(dW^T\boldsymbol{x}_i)\right) = \exp(W^T\boldsymbol{x}_i)^TdW^T\boldsymbol{x}_i$，这是根据等式 $\boldsymbol{1}^T (\boldsymbol{u}\odot \boldsymbol{v}) = \boldsymbol{u}^T \boldsymbol{v}$

$$dJ_i = \text{tr}\left(-\boldsymbol{y}_i^TdW^T\boldsymbol{x}_i+\frac{\exp(W^T\boldsymbol{x}_i)^TdW^T\boldsymbol{x}_i}{\boldsymbol{1}^T\exp(W^T\boldsymbol{x}_i)}\right) =\text{tr}(-\boldsymbol{y}_i^TdW^T\boldsymbol{x}_i+\text{softmax}(W^T\boldsymbol{x}_i)^TdW^T\boldsymbol{x}_i) = \text{tr}(\boldsymbol{x}_i(\text{softmax}(W^T\boldsymbol{x}_i)-\boldsymbol{y}_i)^TdW^T)$$

所以 $\frac{\partial J_i}{\partial W}=\boldsymbol{x}_i(\text{softmax}(W^T\boldsymbol{x}_i)-\boldsymbol{y}_i)^T = \boldsymbol{x}_i(\hat{\boldsymbol{y}}_i - \boldsymbol{y}_i)^T$，这一公式符合上面的逐元素推导

## 扩展阅读

1. [动手学深度学习 softmax回归](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression.html)
2. 实际工程对softmax的优化：[动手学深度学习 softmax回归的简洁实现](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression-concise.html#subsec-softmax-implementation-revisited)

## 代码实现

有意思的是，最后收敛的准确率也还是80%左右，这说明在这个**线性的**3参数模型（单纯捕捉sex age Pclass）的极限基本就是如此了。无论是逻辑回归和线性回归，其实都在训练一个超平面 $w^T x + b = 0$ 把两类情况分开，所以差别不大

不过下面的代码把循环写法改成向量写法，效率会高非常多

```py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

DEBUG = True

# PassengerId Survived Pclass Name Sex Age SibSp Parch Ticket Fare Cabin Embarked
X_train = pd.read_csv("D:\\dataset\\titanic\\train.csv")  # 891 samples
X_test = pd.read_csv("D:\\dataset\\titanic\\test.csv")  # 418 samples
y_test = pd.read_csv("D:\\dataset\\titanic\\gender_submission.csv")

# 打算只对Pclass(int) Sex(str) Age(int)进行拟合，需要对Sex处理一下
# female -> -1, male -> 1
X_train["Sex"] = [-1 if s == "female" else 1 for s in X_train["Sex"]]
X_test["Sex"] = [-1 if s == "female" else 1 for s in X_test["Sex"]]

# Age有大约20%缺失项，做一下填充
# 只对训练集进行fit，再对测试集根据得到的数据进行transform
median = X_train["Age"].median()
X_train["Age"] = X_train["Age"].fillna(median)
X_test["Age"] = X_test["Age"].fillna(median)

# 特征标准化
feature_cols = ["Pclass", "Sex", "Age"]
mean = X_train[feature_cols].mean()
std = X_train[feature_cols].std()
X_train[feature_cols] = (X_train[feature_cols] - mean) / std
X_test[feature_cols] = (X_test[feature_cols] - mean) / std

class LogisticRegression:
    def __init__(self, learning_rate=0.01, n_epochs=100, batch_size=20):
        self.lr = learning_rate
        self.epochs = n_epochs
        self.batch_size = batch_size
        self.w = None
        self.b = None
        self.epoch_record = []
        self.acc_record = []

    def predict(self, X):
        """X: (m, 3) batch matrix, returns output matrix (m,)"""
        return 1 / (1 + np.exp(-(X @ self.w + self.b)))

    def loss(self, X, y):
        """向量化交叉熵损失"""
        y_pred = np.clip(self.predict(X), 1e-15, 1 - 1e-15) # 防止log无意义
        return -np.mean(y * np.log(y_pred) + (1 - y) * np.log(1 - y_pred))

    def fit(self, X_df):
        # 一次性提取 numpy 矩阵，后续全部向量化运算
        X = X_df[feature_cols].values # (n,3)
        y = X_df["Survived"].values # (n,)
        n = X.shape[0]

        self.w = np.zeros(X.shape[1]) # (3,)
        self.b = 0.0

        for epoch in range(self.epochs):
            if DEBUG and (epoch + 1) % 10 == 0:
                print(f"In epochs {epoch + 1}")

            self.epoch_record.append(epoch + 1)
            correct = 0

            for start in range(0, n, self.batch_size):
                end = min(start + self.batch_size, n)
                X_batch = X[start:end] # (m,3)
                y_batch = y[start:end] # (m,1)
                m = end - start  # 当前 batch 的样本数

                # 向量化前向 + 准确率统计
                y_pred = self.predict(X_batch) # (m,)
                correct += np.sum(np.round(y_pred) == y_batch) # batch中正确的总量

                # 向量化梯度 + 更新
                error = y_pred - y_batch # (m,)
                self.w -= self.lr * (X_batch.T @ error) / m # 括号内是(3,)对应w的shape
                self.b -= self.lr * np.mean(error) # batch中error向量的均值，就是求和再平均

            self.acc_record.append(correct / n)

    def test(self, X_df, y_df):
        '''X_df: test set, return a acc rate'''
        X = X_df[feature_cols].values
        y = y_df["Survived"].values
        n = X.shape[0]

        y_pred = self.predict(X)
        correct = np.sum(np.round(y_pred) == y)
        return correct / n


model = LogisticRegression(learning_rate=0.005, n_epochs=300, batch_size=64)
model.fit(X_train)
plt.plot(model.epoch_record, model.acc_record)
plt.title("Acc curve")
plt.xlabel("Epochs")
plt.ylabel("Acc rate")
plt.show()
```