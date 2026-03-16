---
layout: default
title: "嵌入"
date: 2026-03-16
tags: [深度学习, AI]
parent: "笔记 3B1B深度学习"
nav_order: 4
---

## 嵌入 Embedding
用户输入的句子被分割成多个单词片段和标点token，这些token组成一个一维向量，该过程也称作tokenization

在模型中有已经训练好的$$W_E$$ embedding matrix，用矩阵向量乘法将每个token转化成对应的向量，这时候输入数据就变成了矩阵

（经过训练）在这些向量组成的线性空间里，每个方向都代表某种语义，比如视频中的有趣例子

$$\overrightarrow{\text{Hitler}} + \overrightarrow{\text{Italy}} - \overrightarrow{\text{Germany}} \approx \overrightarrow{\text{Mussolini}}$$

好像某个方向定义了“从德国到意大利”和“二战轴心国领导人”

此外每个词的在句中的位置信息也被编码到向量中

然而单个向量仅仅是一个单词的信息，要结合具体上下文语境，还需要经过Attention和MLP层的处理

### 解嵌入 Unembedding
最后输出的矩阵，再与训练好的$$W_U$$相乘，得到一维向量。其中每个数都代表一个单词，这个向量也被称作预测下一个词的logits

logits再经过softmax归一化就得到输出单词的概率

### 归一化 softmax
把一列范围是 $$R$$ 的数据转换为概率分布，需要满足：
1. $$p_i \in [0,1]$$
2. $$\sum p_i = 1$$
3. 转换函数是可导d的

结果就是这个函数

$$p_i = \text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{n=0}^{N-1}e^{x_n}}$$

实际中，还有个温度temperature变量会影响softman的结果，温度越高，模型会给低概率的结果更高权重，从而让分布更均匀。T=0的时候，所有的概率都给予原本的最大值

$$\text{(with temperature) }p_i = \text{softmax}(x_i/T) = \frac{e^{x_i/T}}{\sum_{n=0}^{N-1}e^{x_n/T}}$$