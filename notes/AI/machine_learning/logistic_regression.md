---
title: "逻辑回归"
date: 2026-09-05
tags: [ML, 逻辑回归, AI]
---

# 逻辑回归 Logistic Regression

在线性回归的基础之上增加一个sigmoid函数，并换用交叉熵作为损失函数

用于解决分类问题，下面先介绍二分类

## 输出

$$z = w^T x + b$$

$$\hat y = \sigma(z)$$

其中 $\sigma$ 是sigmoid，表达式为

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

图像是

<figure>
  <img src="/assets/images/machine_learning/sigmoid_pic.webp" alt="sigmoid_pic" />
</figure>

因为目的是分类，需要最后的结果是二元变量01，或者分为某类的概率

这就需要把线性回归的原始输出 $z$ 映射到集合 $\{0,1\}$ 或者区间 $[0,1]$ 上，其中前者的函数不连续，没法通过梯度下降进行训练

sigmoid作为激活函数，有下面的好处：

- 输出范围是 $(0,1)$，符合概率分布
- $z=0$ 是分界线，分出正类和负类
- 处处可导，而且导数比较简单

## 损失

与线性回归不同，逻辑回归的损失函数是交叉熵损失

在线性回归里，MSE+线性输出得到的损失函数，如果以参数为自变量，是凸函数，这确保能收敛到全局最优

但是MSE+线性输出+sigmoid得到的函数是非凸的，把MSE换为交叉熵损失，又回到了凸函数

$$J(\theta) = -\frac{1}{n} \sum_{i=1}^n [y_i \log(\hat y) + (1-y_i) \log(1 - \hat y)] $$

交叉熵损失的推导可见[[似然估计]]

被求和的部分有两项，在二分类问题里，标签 $y_i$ 只有0/1两个取值，所以对每个样本而言，只剩下一项对数项

假设 $y_i=1$，那么只剩下 $\log(\hat y)$，预测标签 $\hat y$ 越接近1，损失越接近0；如果越接近0，损失会趋于无穷大

## 梯度

接下来推导 $\nabla_w J$

sigmoid的导数性质是显然的

$$\sigma'(z) = \sigma(z)(1-\sigma(z)) = \hat y (1 - \hat y)$$

对单个样本 $(x_i, y_i)$，用链式法则

$$\frac{\partial J_i}{\partial w}
= \frac{\partial J_i}{\partial \hat y_i} \cdot \frac{\partial \hat y_i}{\partial z_i} \cdot \frac{\partial z_i}{\partial w}$$

逐项计算

$$\frac{\partial J_i}{\partial \hat y_i}
= -\left( \frac{y_i}{\hat y_i} - \frac{1-y_i}{1-\hat y_i} \right)
= \frac{\hat y_i - y_i}{\hat y_i (1-\hat y_i)}$$

$$\frac{\partial \hat y_i}{\partial z_i} = \sigma'(z_i) = \hat y_i (1-\hat y_i)$$

$$\frac{\partial z_i}{\partial w} = x_i$$

三项相乘，中间的 $\hat y_i (1-\hat y_i)$ 恰好约掉：

$$\frac{\partial J_i}{\partial w}
= \frac{\hat y_i - y_i}{\cancel{\hat y_i (1-\hat y_i)}} \cdot \cancel{\hat y_i (1-\hat y_i)} \cdot x_i
= (\hat y_i - y_i)\, x_i$$

对所有样本取平均，最后的结果是

$$\nabla_w J = \frac{1}{n} \sum_{i=1}^n (\hat y_i - y_i)\, x_i$$

同理可得偏置的梯度：

$$\nabla_b J = \frac{1}{n} \sum_{i=1}^n (\hat y_i - y_i)$$

如果线性回归的MSE多乘一个1/2，那么两者的梯度在形式上就是完全一样的了

## 参数更新

和线性回归一样

$$w \leftarrow w - \eta \cdot \nabla_w L$$

$$b \leftarrow b - \eta \cdot \nabla_b L$$

## 多分类的梯度

上面都是二分类问题，如果需要对 $K$ 个分类输出概率，需要做一些改动

$z \in R^K$ 每个元素 $z_i$ 都是对应类别 $i$ 的预测logits，然后代入softmax

$$P(y=k) = \hat y_k = \frac{e^{z_k}}{\sum_{j=1}^K e^{z_j}}$$

交叉熵损失做相应扩展，从两项变成 K 项

$$J(\theta) = -\frac{1}{n} \sum_{i=1}^n \sum_{k=1}^K y_{ik} \log(\hat y_{ik})$$

其中 $y_{ik}$ 是第 $i$ 个样本的 one-hot 标签，属于类别 $k$ 时为 1，否则为 0

最后每个样本的求和里其实也只剩下一项，和二分类的情况一样

### 逐元素形式

接下来推导 $\nabla_{w_k} J$

对单个样本 $(x_i, y_i)$，用链式法则。注意损失中每个 $\hat y_j$ 都有贡献，而每个 $\hat y_j$ 又都依赖 $z_k$，所以要加上对 $j$ 的求和：

$$\frac{\partial J_i}{\partial w_k}
= \sum_{j=1}^K \frac{\partial J_i}{\partial \hat y_j} \cdot \frac{\partial \hat y_j}{\partial z_k} \cdot \frac{\partial z_k}{\partial w_k}$$

逐项计算，其中

$$\frac{\partial J_i}{\partial \hat y_j} = -\frac{y_j}{\hat y_j}$$

第二项比较复杂，二分类里 sigmoid 的导数是标量 $\hat y(1-\hat y)$，但 softmax 的每个输出 $\hat y_j$ 都通过分母依赖**所有** $z_1,\dots,z_K$，所以导数要分 $j=k$ 和 $j \neq k$ 两种情况：

$$\frac{\partial \hat y_j}{\partial z_k} = \begin{cases} \hat y_k (1 - \hat y_k) & j = k \\ -\hat y_j \hat y_k & j \neq k \end{cases}$$

$$\frac{\partial z_k}{\partial w_k} = x_i$$

代入导数，把求和拆成 $j=k$ 的一项和 $j \neq k$ 的其余项：

$$\frac{\partial J_i}{\partial w_k}
= -\left[ \frac{y_k}{\hat y_k} \cdot \hat y_k (1-\hat y_k) + \sum_{j \neq k} \frac{y_j}{\hat y_j} \cdot (-\hat y_j \hat y_k) \right] x_i \\ = -\left[ y_k (1-\hat y_k) - \hat y_k \sum_{j \neq k} y_j \right] x_i$$

用 one-hot 和为1的性质 $\sum_{j \neq k} y_j + y_k = 1$ 整理括号里的项：

$$= -\left[ y_k - \hat y_k \left( y_k + \sum_{j \neq k} y_j \right) \right] x_i = (\hat y_k - y_k)\, x_i$$

对所有样本取平均，最后的结果是

$$\nabla_{w_k} J = \frac{1}{n} \sum_{i=1}^n (\hat y_{ik} - y_{ik})\, x_i$$

同理可得偏置的梯度：

$$\nabla_{b_k} J = \frac{1}{n} \sum_{i=1}^n (\hat y_{ik} - y_{ik})$$

和二分类的形式完全一样

### 矩阵形式

下面的内容改编自[知乎-矩阵求导术](https://zhuanlan.zhihu.com/p/24709748)，非常值得一看；或者笔记[[矩阵微积分]]中也要大致摘要

$l = -\boldsymbol{y}^T\log\text{softmax}(W\boldsymbol{x})$，求 $\frac{\partial l}{\partial W}$

其中 $\boldsymbol{y}$ 是除一个元素为1外其它元素为0的 $m×1$ 列向量，$W$ 是 $m\times n$ 矩阵，$\boldsymbol{x}$ 是 $n×1$ 列向量，$l$ 是标量；log表示自然对数，$\text{softmax}(\boldsymbol{a}) = \frac{\exp(\boldsymbol{a})}{\boldsymbol{1}^T\exp(\boldsymbol{a})}$，其中 $\exp(\boldsymbol{a})$ 表示逐元素求指数，$\boldsymbol{1}$ 代表全1向量

代入softmax

$$l = -\boldsymbol{y}^T \left(\log (\exp(W\boldsymbol{x}))-\boldsymbol{1}\log(\boldsymbol{1}^T\exp(W\boldsymbol{x}))\right) = -\boldsymbol{y}^TW\boldsymbol{x} + \log(\boldsymbol{1}^T\exp(W\boldsymbol{x}))$$

注意逐元素log满足等式 $\log(\boldsymbol{u}/c) = \log(\boldsymbol{u}) - \boldsymbol{1}\log(c)$，以及 $\boldsymbol{y}$ 满足 $\boldsymbol{y}^T \boldsymbol{1} = 1$

求微分

$$dl =- \boldsymbol{y}^TdW\boldsymbol{x}+\frac{\boldsymbol{1}^T\left(\exp(W\boldsymbol{x})\odot(dW\boldsymbol{x})\right)}{\boldsymbol{1}^T\exp(W\boldsymbol{x})}$$

再套上迹并做交换，注意可化简 $\boldsymbol{1}^T\left(\exp(W\boldsymbol{x})\odot(dW\boldsymbol{x})\right) = \exp(W\boldsymbol{x})^TdW\boldsymbol{x}$，这是根据等式 $\boldsymbol{1}^T (\boldsymbol{u}\odot \boldsymbol{v}) = \boldsymbol{u}^T \boldsymbol{v}$

$$dl = \text{tr}\left(-\boldsymbol{y}^TdW\boldsymbol{x}+\frac{\exp(W\boldsymbol{x})^TdW\boldsymbol{x}}{\boldsymbol{1}^T\exp(W\boldsymbol{x})}\right) =\text{tr}(-\boldsymbol{y}^TdW\boldsymbol{x}+\text{softmax}(W\boldsymbol{x})^TdW\boldsymbol{x}) = \text{tr}(\boldsymbol{x}(\text{softmax}(W\boldsymbol{x})-\boldsymbol{y})^TdW)$$

所以 $\frac{\partial l}{\partial W}= (\text{softmax}(W\boldsymbol{x})-\boldsymbol{y})\boldsymbol{x}^T = (\hat{\boldsymbol{y}} - \boldsymbol{y})\boldsymbol{x}^T$ 这一公式符合上面的逐元素推导

## 代码实现

优化了[[线性回归]]里关于泰坦尼克的代码

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
        y = X_df["Survived"].values # (n,1)
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