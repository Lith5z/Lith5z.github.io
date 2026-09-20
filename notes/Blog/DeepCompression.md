---
title: "论文阅读 DeepCompression"
date: 2026-09-20
tags: [剪枝, 轻量化, 论文]
---

# DEEP COMPRESSION

Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding

ICLR 2016

神经网络因为储存和计算的高开销，难以在资源受限的嵌入式设备上运行。作者使用了剪枝+量化+霍夫曼编码的方法，缩小了35-49倍的储存要求，大大提高了运算速度和能耗表现，同时不影响准确率。并且剪枝和量化不会相互影响

> Our main insight is that, pruning and trained quantization are able to compress the network without interfering each other, thus lead to surprisingly high compression rate.

模型压缩的核心目的不仅是减小存储，更是为了将模型完全放进芯片上的SRAM中，从而避免高昂的片外DRAM访存能耗

## 剪枝

关于剪枝的具体操作，本文一笔带过了，有些内容在作者的上一篇论文[Learning both Weights and Connections for Efficient Neural Networks](https://arxiv.org/abs/1506.02626)里，重点是conv层对剪枝的敏感度大于FC层

- 常规训练
- 每层确定剪枝阈值 $\text{Thershold}_l = \lambda_l \times \sigma(W_l)$，其中 $\lambda_l$ 是每层的敏感度系数，$\sigma(W_l)$ 是该层权重的标准差
  - 带有L2正则化训练后神经网络的权重分布近似于均值为0的正态分布，落在 $[-\lambda \sigma, \lambda \sigma]$ 区间的概率由高斯误差函数确定，只与 $\lambda$ 有关，确定 $\lambda$ 的过程就与层的权重分布情况解耦了
  - 具体见上一篇论文
- 迭代进行剪枝和微调
  - 全模型剪枝+微调效果很差
  - conv1剪枝->微调->conv2剪枝->微调...，开销很大，且层之间可能相互耦合，逐层操作破坏了整体性
  - 采用分组迭代的方式：FC层剪枝->FC微调->conv剪枝->conv微调

剪枝的结果可见Table 4和5，对于AlexNet和VGG16的实验结果，重点看第一第二列，即参数量和保留率，可以看出

<figure>
  <img src="/assets/images/blog/deepcompression_table45_pic.webp" alt="deepcompression_table45" />
</figure>

- conv层的保留率普遍高于FC层
- conv层中，第一层的保留率往往是最高的
- FC层中，最后一层的保留率最高

这需要结合模型架构来分析

- CNN网络里，卷积层可以看作特征提取器，顶层的FC层负责映射到分类。但是这两个早期CNN网络都采取了大FC的模式（FC层占了模型80-90%的参数量），FC层的冗余度很大，受剪枝的影响小
- conv1直接处理图像输入，通道数只有3，所以通道间的冗余小
- 顶层FC负责分类输出，每个神经元都与上一层全连接，对应一个类别

## 稀疏存储

根据之前的图表，剪枝后的矩阵大部分参数都为0，只需要存储非零值

- CSR/CSC格式存储稀疏矩阵，只需要 2a+n+1 个元素即可存储
  - 以CSR格式为例，对于一个(n,n)共a个非零元素的矩阵
  - `values`数组，长度为a，按照行主序存所有非零元素的值，FP32
  - `col_idx`数组，长度a，存每个非零元素对应的列号，INT
  - `row_ptr`数组，长度n+1，存原矩阵每行第一个非零元素在`values`中的起始位置，且`row_ptr[n]=a`，INT
- 对于CSR下的`col_idx`数组（或者CSC下是`row_idx`），再使用index diff储存方式进一步压缩
  - 在上一篇论文中，conv层仅需8bits/fc层5bits来存储diff
  - 此时`values`数组还是FP32
  - 如果diff超出上限，填充0

一个CSR的例子：

$$A = \begin{bmatrix} 1 & 0 & 2 \\ 0 & 0 & 3 \\ 4 & 5 & 0 \end{bmatrix}$$

结果是

```text
values      = [1, 2, 3, 4, 5]
col_indices = [0, 2, 2, 0, 1]
row_ptr     = [0, 2, 3, 5]
```

## 量化和权重共享

对于**每一层剪枝后**的权重矩阵（或者是`values`向量），将相似值分组cluster，每组得到本组的centroid作为共享权重，再使用一个低位数的`cluster index`矩阵即可让原权重矩阵映射到（少得多的）centroids数组（也叫codebook）上

对于剪枝后的AlexNet，每个conv层，只需要8bit存储`cluster index`，也就是256个共享权重/centroids；对于FC层则是5bit；注意共享的权重centroids本身依旧是FP32

再将梯度也进行相同分组，把每组的累积梯度作为centroid的梯度，对共享权重进行微调

逐层独立进行的，不同层之间不共享权重

例如下图Figure3是4×4 fp32 矩阵 -> 分成4组，只需要 4个fp32的centroid数组 和 4×4 2bit index矩阵

<figure>
  <img src="/assets/images/blog/deepcompression_figure3_pic.webp" alt="deepcompression_figure3" />
</figure>

### 如何聚类

假设从n个权重分出k类，先通过某个方法初始化得到centroid，再通过k-means迭代优化，得到微调前的centroid

初始化方法

- Forgy/random：随机从n中观测k个
- Density-based：根据累积分布函数CDF选取
- Linear：在权重的[min,max]之间等距取k个点

实际中大权重比小权重更加重要，但是数量也更少。random和density方法都会受到权重分布的影响，从而更容易选取小权重作为centroid。而linear方法没有这个问题

下图是Figure4，可以看出剪枝之后，0附近的权重很少，权重分布在两个峰的附近，random和density采样点也在这两个峰附近

<figure>
  <img src="/assets/images/blog/deepcompression_figure4_pic.webp" alt="deepcompression_figure4" />
</figure>

再看table4和5的中间4列，先忽略(P+Q+H)的部分，这是后面的huffmax coding

<figure>
  <img src="/assets/images/blog/deepcompression_table45_pic.webp" alt="deepcompression_table45" />
</figure>

现在模型的参数已经由如下数据存储，以AlexNet为例：

- `value`被拆分成两部分：
  - 权重共享的索引`cluster index`，也就是table中的weight bits，conv为8bit，FC为5bit
  - codebook内，共享的权重centroid还是FP32
- `col_idx`用index diff存储，也就是table中的index bits，实际上用的是4bit
  - 我们之前在剪枝部分说是conv为8bit，FC为5bit
  - 实际上conv稀疏度没这么低，使用8bit作为diff是浪费了
  - 5.2节也说*The relative sparse index is encoded with 4 bits.*
- `row_ptr`，INT

### 如何微调

使用聚类后的centroid值来进行正向和反向传播的计算

每组的**累积**梯度作为centroid的梯度，对centroid进行最后微调

## 霍夫曼编码

<figure>
  <img src="/assets/images/blog/deepcompression_figure5_pic.webp" alt="deepcompression_figure5" />
</figure>

经过上述操作之后，权重的大小分布在两个峰值附近，权重diff index的分布高度偏向0的一侧，索引极少超过20，对两者分别使用huffman coding进一步减小20%-30%的空间

这部分的讲解略去

## Discussion

这部分难度不大，我摘录了一些内容

- 剪枝和量化同时使用效果比单独使用更好，figure6
- conv层比FC层更敏感，figure7里，FC层只需要2bit的cluster index，而conv需要4-5
  > The first two plots in Figure 7 show that CONV layers require more bits of precision than FC layers. For CONV layers, accuracy drops significantly below 4 bits, while FC layer is more robust: not until 2 bits did the accuracy drop significantly
- 聚类的linear初始化是最优的

## 局限性

1. 使用非结构化剪枝，需要硬件和计算库的支持
2. 大batch推理下速度更慢，密集矩阵有缓存优势，而稀疏网络的索引存储方式有额外开销（见table 8）
3. 涉及到多步微调，麻烦

其他扩展思考：

1. 为什么不对bias进行剪枝？（没必要）
2. 结合近年论文的趋势，半/结构化剪枝才是更可行的
3. 为什么现代开源大模型，在hugging face往往有量化版本而没有剪枝版本