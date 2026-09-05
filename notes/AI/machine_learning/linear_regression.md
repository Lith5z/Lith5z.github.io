---
title: "线性回归"
date: 2026-09-05
tags: [ML, 线性回归, AI]
---

# 线性回归 Linear Regression

最简单的机器学习模型，用于回归问题，下面以单特征的线性回归为例

## 输出

$$\hat y = w^T x + b$$

- $w$和$x$都是向量
- 做点积得到的结果是标量

## 计算损失

和[[感知器]]中的用于分类的离散的输出不同，线性回归的输出是连续的。所以前者我们用正确的比例来衡量模型的准确性，后者需要一个损失函数

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

> 其实不完全是，因为损失函数最小值对应的权重是可以通过解析方法直接得到的，只是对于大数据量，解析方法的开销可能会很大，而优化迭代方法可行性更高。

最通用的优化方法是梯度下降，计算损失函数的负梯度得到最小值的方向

$$ \text{GD:}\quad \theta \leftarrow \theta - \eta \nabla_{\theta} L(\theta) $$

$\theta$ 是模型的所有参数，$\eta$ 是学习率 / learning_rate，$L$ 是损失函数。

如果损失函数是 MSE，可以直接写出解析梯度，比数值方法（有限差分）快得多，而且没有误差（注意梯度是向量）

$$\nabla_w L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i) \, x_i$$

$$\nabla_b L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i)$$

参数的具体更新公式是

$$w \leftarrow w - \eta \cdot \nabla_w L$$

$$b \leftarrow b - \eta \cdot \nabla_b L$$

## 多项式回归

现实中很多关系不是直线，可以把原始输入向量 $x$ 映射到一组新特征

$$\phi(x) = [1, x, x^2, ..., x^n]$$

然后仍然做线性回归

$$ \hat y = w^T \phi(x) + b = b + w_1 x + w_2 x^2 + \cdots + w_n x^n $$

## 多特征

约定样本矩阵 $\mathbf{X}\in\mathbb{R}^{N\times d}$，标签 $\mathbf{y}\in\mathbb{R}^N$，权重 $\mathbf{w}\in\mathbb{R}^d$
 
$$L = \|\mathbf{X}\mathbf{w} - \mathbf{y}\|^2 = (\mathbf{X}\mathbf{w} - \mathbf{y})^\mathsf{T}(\mathbf{X}\mathbf{w} - \mathbf{y})$$

逐元素写出

$$L = \sum_{i=1}^N\left(\sum_{j=1}^d X_{ij}w_j - y_i\right)^2$$

对分量 $w_k$ 求偏导

$$\frac{\partial L}{\partial w_k} = 2\sum_{i=1}^N \left(\sum_j X_{ij}w_j - y_i\right) X_{ik}$$

把所有偏导堆成列向量

$$\frac{\partial L}{\partial \mathbf{w}} = 2\mathbf{X}^\mathsf{T}(\mathbf{X}\mathbf{w} - \mathbf{y}) \in \mathbb{R}^d$$

## 扩展话题

### 评估模型 R²

在数据标签的量纲不同时，MSE值不能作为评估模型准确性的依据

这时需要用 $R^2$ 来评估

$$ R^2 = 1 - \frac{\sum (\hat y_i - y_i)^2}{\sum (y_i - \bar y)^2} $$

$R^2 = 1$ 表示完美拟合，$R^2 = 0$ 表示和猜均值一样差，$R^2 < 0$ 表示比猜均值还差

### 闭式解

对于线性回归，损失函数MSE是凸函数，可以直接解出全局最优解

$$ w = (X^T X)^{-1} X^T y $$

在数据量小时（$n < 10^4$ 左右），直接使用上述公式，不需要迭代训练

### 特征标准化

梯度下降要求各特征的数值范围相近，但是得到的数据，各个特征的尺度很可能不同

比如在泰坦尼克数据集里，船舱等级 `Pclass` (1–3) 和 年龄 `Age` (0–80) 相差几十倍，而根据MSE损失下梯度的解析式

$$\nabla_w L = \frac{2}{n} \sum_{i=1}^{n} (\hat y_i - y_i) \, x_i$$

可以看到梯度大小和特征的值是正相关的

假设学习率$\eta$是固定的，如果$\eta$对于 `Pclass` 来说合适，更新的幅度对于 `Age` 来说就会非常大；如果对于 `Age` 合适，更新的幅度对于 `Pclass` 来说就会非常小

所以大尺度特征会主导梯度，导致收敛缓慢甚至发散

这时候需要对数据进行标准化处理，把每个特征缩放到均值 0、标准差 1

$$ x' = \frac{x - \mu}{\sigma} $$

## 代码实现

下面以泰坦尼克号幸存者数据集为例，我用pandas手写了一个简单的线性回归，大概能到80%左右的准确率

特别注意：预测生存与否是个分类问题，应该使用逻辑回归，我这里强行用线性回归，结果只能说能跑...还有很多地方都处理的很烂，仅供参考

```py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

DEBUG = False

# PassengerId Survived Pclass Name Sex Age SibSp Parch Ticket Fare Cabin Embarked
X_train = pd.read_csv("D:\\dataset\\titanic\\train.csv") # 891 samples
X_test = pd.read_csv("D:\\dataset\\titanic\\test.csv") # 418 samples
y_test = pd.read_csv("D:\\dataset\\titanic\\gender_submission.csv")

# 打算只对Pclass(int) Sex(str) Age(int)进行拟合，需要对Sex处理一下
# female -> -1, male -> 1
X_train["Sex"] = [ -1 if string == "female" else 1 for string in X_train["Sex"] ]
X_test["Sex"] = [ -1 if string == "female" else 1 for string in X_test["Sex"] ]

# Age有大约20%缺失项，做一下填充
X_train["Age"] = X_train["Age"].fillna(X_train["Age"].median())

# 特征标准化（三个参数的范围差距很大，Age的范围最大，但是没有理由确定Age能主导梯度下降）
feature_cols = ["Pclass", "Sex", "Age"]
mean = X_train[feature_cols].mean()
std = X_train[feature_cols].std()
X_train[feature_cols] = (X_train[feature_cols] - mean) / std

class LinearRegression:
    def __init__(self, learning_rate=0.01, n_epochs=100, batch_size = 20 ,delta=0.0001):
        self.lr = learning_rate
        self.epochs = n_epochs
        self.batch_size = batch_size
        self.delta = delta
        self.w = None
        self.b = None
        self.epoch_record = []
        self.acc_record = []

    def predict(self, x):
        '''x is one sample, returns raw continuous value'''
        x_extracted = np.array([x["Pclass"], x["Sex"], x["Age"]])
        return x_extracted @ self.w + self.b
    
    def loss(self, X : pd.DataFrame):
        '''MSE loss'''
        result = 0
        for i in range(X["PassengerId"].size):
            result += (self.predict(X.iloc[i]) - X.iloc[i]["Survived"]) ** 2
        return result / X["PassengerId"].size

    # 这里是纯粹数值计算的梯度，但是对于MSE这样简单的损失函数，可以直接写出解析式的
    def w_grad(self, X : pd.DataFrame):
        result = np.zeros(3) # w_Pclass w_Sex w_Age
        for i in range(3):
            self.w[i] += self.delta
            changed = self.loss(X)
            self.w[i] -= self.delta
            result[i] = (changed - self.loss(X)) / self.delta
        return result
    def b_grad(self, X):
        self.b += self.delta
        changed = self.loss(X)
        self.b -= self.delta
        return (changed - self.loss(X)) / self.delta

    def fit(self, X):
        self.w = np.zeros(3) # w_Pclass w_Sex w_Age
        self.b = 0
        size = X["PassengerId"].size
        for i in range(self.epochs):
            if DEBUG: print(f"In epochs {i+1}")
            self.epoch_record.append(i+1)
            correct_nums = 0
            batch = 0
            for p in range(int(size / self.batch_size) + 1):
                if batch + self.batch_size <= size:
                    upper = batch + self.batch_size
                else:
                    upper = size

                # 统计参数更新前的正确个数
                for j in range(batch, upper):
                    y_pred = self.predict(X.iloc[j])
                    if X.iloc[j]["Survived"] == np.round(y_pred):
                        correct_nums += 1

                # 更新参数
                w_grad = self.w_grad(X.loc[batch : upper])
                b_grad = self.b_grad(X.loc[batch : upper])
                self.w -= self.lr * w_grad
                self.b -= self.lr * b_grad

                batch += self.batch_size

            self.acc_record.append(correct_nums / size)

    def test(self, X, y):
        pass

model = LinearRegression(n_epochs=100)
model.fit(X_train)
fig1 = plt.figure()
plt.plot(model.epoch_record, model.acc_record)
plt.show()
```