---
layout: default
title: "感知器"
date: 2026-06-14
tags: [ML, 感知器, AI]
parent: "笔记 机器学习"
nav_order: 1
---

# 感知器 Perceptron

<figure>
  <img src="{{ '/assets/images/machine_learning/perceptron_pic.webp' | absolute_url }}" alt="perceptron" />
</figure>

> 感知器由 **Frank Rosenblatt** 于 1957 年在 Cornell 航空实验室提出，并在 1958 年正式发表论文 *"The Perceptron — A Perceiving and Recognizing Automaton"*。这是第一个具有学习能力的人工神经元模型，在当时的 AI 社区引起了巨大轰动。
> Rosenblatt 甚至为感知器设计了专门的硬件 **Mark I Perceptron**，能够通过模拟电路实时学习识别简单的视觉模式。它被视为连接主义（Connectionism）的开端，也是现代神经网络的雏形。
> ——DeepSeek v4

借鉴了生物神经元的结构，通过多个树突接收特征信息，权重模拟神经元之间连接的强弱，进行加权求和后判断是否激活

如果没有特别说明，下面讨论的都是二元的感知器

## 输出

感知器最后输出的结果是一个二元变量，用于分类问题

$$ \hat{y} = g(w \cdot x + b) $$

- $$w$$为权重向量
- $$x$$为输入的特征向量，做内积
- $$b$$为偏置，一个标量
- $$g$$为激活函数，这里一般是阶跃函数（区别神经网络），例如：

$$g(x) = \begin{cases} 1, & x > 0 \\ -1, & x \leq 0 \end{cases}$$

> 如果偏置项为0，激活函数以0为界，那么决定输出的分界点就是w和x点积为0，即正交的时候。所以感知器找到的分类界限，是与权重向量正交的一个分界面。
> 偏置项会影响wx=0时的结果，相当于平移这个分界面。
> 权重向量其实代表某类典型的输入，做点积的过程就是在衡量两个向量的相似度。

## 权重更新

一开始所有参数初始化为0

如果预测结果符合真实结果，不做变动；如果有误，更新

$$ w \leftarrow w + \eta \cdot (y - \hat{y}) \cdot x $$

$$ b \leftarrow b + \eta \cdot (y - \hat{y}) $$

其中$$\eta$$是学习率

**从公式的角度理解** <br>
假设$$g(x)$$就是上面的那个阶跃函数，期望的$$y=1$$，而模型预测$$\hat{y}=0$$，$$x \gt 0$$。此时权重会被增大，让$$g(x)$$的输入变大，更可能得到预期的+1

**从线性变换的角度理解** <br>
如果$$x$$和$$w$$在分界线的两侧且预测错误，这一项会逐渐把特征向量拉到$$w$$的一侧

<figure>
  <img src="{{ '/assets/images/machine_learning/perceptron_update_pic.webp' | absolute_url }}" alt="perceptron_update" />
</figure>

## 多分类感知器

如果要进行多个类别的分类，输出的就不能只是简单的二元数值

<figure>
  <img src="{{ '/assets/images/machine_learning/perceptron_multi_pic.webp' | absolute_url }}" alt="perceptron_multi" />
</figure>

这时候对于每个类别都要提供一个特征向量$$w_y$$，输出的结果是产生最大点积的那个类别：

$$ y = \arg \max_y w_y \cdot x $$

更新过程也类似，对于分类出错的样本，只更新相关的两个特征向量：对于正确类别的特征向量，点积要增大；目前预测类别的特征向量，点积要减小。其他类别的权重向量保持不变

设真实类别为 $$y^*$$，模型预测类别为 $$\hat{y}$$，当 $$y^* \neq \hat{y}$$ 时：

$$
\begin{aligned}
w_{y^*} &\leftarrow w_{y^*} + \eta \cdot x \\
w_{\hat{y}} &\leftarrow w_{\hat{y}} - \eta \cdot x
\end{aligned}
$$

每次犯错，就把真实类别的“模板”向量拉向当前样本，把错误类别的“模板”向量推离当前样本，从而在特征空间中划出更清晰的决策边界。

## 缺陷

对于可分Separable的数据，上面的训练过程一定会收敛（感知器收敛定理，Perceptron Convergence Theorem） <br>
但如果不具备可分性，比如XOR（异或）的功能，训练过程不会收敛，也就是说单层感知器无法解决这一问题

> 1969 年，**Marvin Minsky** 和 **Seymour Papert** 在著作 *"Perceptrons"* 中系统分析了感知器的数学局限性，严格证明了单层感知器无法处理线性不可分问题（如 XOR）。
> 这一结论对当时的神经网络研究造成了沉重打击，加之计算资源匮乏，政府资助大幅转向符号主义 AI，导致神经网络研究进入了长达十余年的 **"AI 寒冬"**（First AI Winter）。
> 直到 1986 年，**Rumelhart、Hinton 和 Williams** 在 *"Learning representations by back-propagating errors"* 中系统阐述了 **反向传播算法（Backpropagation）**，为**多层感知器MLP**的训练提供了可行的梯度下降方案。多层感知器通过引入隐藏层和非线性激活函数（如 Sigmoid），克服了单层感知器无法解决非线性问题的缺陷，开启了神经网络的复兴之路。
> ——DeepSeek v4

可见[[3B1B 多层感知器]]

## 代码实现

我自己写了个，基于numpy
```py
import numpy as np

# 下面约定，大写字母为向量/矩阵等数据集，小写字母是从中抽取的单条样本

X = np.array([[0,0], [0,1], [1,0], [1,1]]) # one row is one input
Y_and = np.array([0,0,0,1])
Y_or = np.array([0,1,1,1])
Y_xor = np.array([0,1,1,0])
# 不过一般数据集X是大写，标签集y是小写..

class Perceptron:
    def __init__(self, learning_rate=0.01, n_epochs=100):
        self.lr = learning_rate
        self.epochs = n_epochs
        self.w = None
        self.b = None

    def activation(self, z):
        return 1 if z >= 0 else 0

    def predict(self, x):
        '''x is one sample'''
        z = x @ self.w + self.b
        return self.activation(z)
    
    def fit(self, X, Y):
        self.w = np.zeros(X.shape[1])
        self.b = 0
        for i in range(self.epochs):
            correct_nums = 0
            for idx, sample in enumerate(X):
                # 计算预测值和误差
                y_pred = self.predict(sample)
                error = Y[idx] - y_pred

                # 更新权重和偏置
                self.w += self.lr * error * sample
                self.b += self.lr * error

                if error == 0:
                    correct_nums += 1
            acc = correct_nums / X.shape[0]
            print(f"Epoch: {i+1} / {self.epochs} - Acc: {acc}")

            if correct_nums == X.shape[0]:
                print(f"Finish at epoch {i+1}. Final Acc: {acc}")
                return

        print(f"Finish at epoch {i+1}. Final Acc: {acc}")

model = Perceptron(n_epochs=10)
print("Example 1 - and func")
model.fit(X,Y_and)
print("\n-------------------\n")
print("Example 2 - or func")
model.fit(X,Y_or)
print("\n-------------------\n")
print("Example 3 - xor func")
model2 = Perceptron(n_epochs=50)
model2.fit(X,Y_xor)
```

最后xor的例子里，50轮正确率一直是0，这与之前的理论相吻合

## 扩展阅读

- "The Perceptron — A Perceiving and Recognizing Automaton" (Rosenblatt 1957)
- "Perceptrons" (Marvin Minsky / Seymour A. Papert)
- [语雀文档 感知器](https://www.yuque.com/qx2io/blz6hd/kuqdhoh4p4wgirey)