---
layout: default
title: "贝叶斯网络 采样"
date: 2026-05-15
tags: [AI, BN]
parent: "笔记 CS188"
nav_order: 10
---

1. 目录
{:toc}

# 采样 Sampling in BN

之前在强化学习中已经接触过采样sampling的方法，但是当时的目的是学习Learning <br>
在贝叶斯网络中，节点的概率信息是已知的，采样的目的是比直接计算更快地得到概率推理的结果，虽然这个结果不一定准确

假设一个节点A，有60%概率得到+a，40%概率得到-a <br>
我们可以用一个0-1之间的随机数模拟这个节点，当随机数在0-0.6之间，视为采样得到+a；0.6-1之间视为-a

如果一个采样方法是一致的consistent，意味着当采样次数足够大，频率能趋近于正确的概率

## 先验采样 Prior Sampling

沿着BN从上到下进行抽取样本，根据每个节点的概率分布来模拟抽取的概率

<figure>
  <img src="{{ '/assets/images/algorithms_intro/BN_priorSampling_pic.webp' | absolute_url }}" alt="BN_priorSampling" />
</figure>

```
下图中，从C节点开始，按照0.5/0.5的概率抽取，得到+c
向下遍历，S节点，按照P(S|+c)的模拟..
依此类推，最后得到一个完整的样本
```

然后根据样本的频率，就可以推理概率，例如

$$P(+c) = N(+c) / N$$

$$P(+c \mid -s) = N(+c,-s) / N(-s)$$

伪代码：
```py
#generate one sample
def PriorSampling():
    for i in (1,2,...,n):
        sample x_i from P(X_i | Parents(X_i))
    return (x_1,x_2,...,x_n)
```

## 拒绝采样 Rejection Sampling

和先验采样的流程基本一致 <br>
唯一的区别是，对于带有条件概率的推理，不保留所有样本最后再计算，而是直接丢弃不符合条件evidence的样本

伪代码：
```py
def RejectionSampling():
    for i in (1,2,...,n):
        sample x_i from P(X_i | Parents(X_i))
        if x_i not consistent with evidence:
            return
            #reject - no sample is generated in this try
    return (x_1,x_2,...,x_n)
```

抽取到符合条件的样本的可能性就是P(condition)，所以很多样本都被抛弃了，没有得到有效利用

## 似然加权 Likelihood Weight

<figure>
  <img src="{{ '/assets/images/algorithms_intro/BN_priorSampling_pic.webp' | absolute_url }}" alt="BN_priorSampling" />
</figure>

在拒绝采样中，会抛弃很多不符合证据的样本；而似然加权对于包含证据的节点，不按照概率去生成，而是直接固定，概率用来调整样本的权重

```
比如计算P(+s,+w)，一开始权重为1
抽取C和R节点的过程和之前相同，假设抽取到了+c +r
在S节点，不按照P(S|+c)去生成样本，而是直接固定为证据+s，权重乘以P(+s|+c) = 0.1
在W节点，直接固定+w，然后权重乘以P(+w|+s,+r) = 0.99

最后该样本的权重为0.099
```

推理概率的时候，按照加权的频率计算

伪代码：
```py
def LikelihoodWeight():
    weight = 1
    for i in (1,2,...,n):
        if X_i is an evidence variable:
            x_i = evidence x_i for X_i
            weight *= P(x_i | Parents(X_i))
        else:
            sample x_i from P(X_i | Parents(X_i))
    return (x_1,x_2,...,x_n),weight
```

当证据概率极低，而且还在BN的下游时，似然加权就没有很有效了
```
比如 Fire -> Alarm，推理概率P(+a)
按照现实情况设定概率，+f概率很低，警报也非常准确
那么首先对F节点，绝大多数样本都抽取到-f，然后固定+a，但是权重P(+a|-f)也将非常小
极少数+f样本，才有较大的权重
```
虽然通过似然加权，样本都满足证据不会被抛弃，但是这种情况下多数样本的权重趋近于零，大权重的有效样本数量极少，采样的效率很低

## Gibbs Sampling

<figure>
  <img src="{{ '/assets/images/algorithms_intro/BN_gibbsSampling_pic.webp' | absolute_url }}" alt="BN_gibbsSampling_pic" />
</figure>

- 先固定所有证据，其他变量随机赋值
- 然后进行迭代：每一次选择一个变量，重新赋值，赋值的概率依据是 $$P(X_i \mid \text{其余全部确定的变量} )$$
- 大量重复这个过程，得到一条样本链

最后进行概率推理：丢弃开头的一部分样本（烧入期，让链稳定），然后每隔几步取一个样本（减少相关性），这些样本就近似服从目标概率

**如何得到概率$$P(X_i \mid \text{其余全部确定的变量} )$$** <br>
根据BN的特性进行计算，与$$X_i$$无关的项都会抵消掉 <br>
比如在上图中：

$$
\begin{aligned}
P(S|+c,+r,-w) & =\frac{P(S,+c,+r,-w)}{P(+c,+r,-w)} \\
 & =\frac{P(S,+c,+r,-w)}{\sum_{s}P(s,+c,+r,-w)} \\
 & =\frac{P(+c)P(S|+c)P(+r|+c)P(-w|S,+r)}{\sum_{s}P(+c)P(s|+c)P(+r|+c)P(-w|s,+r)} \\
 & =\frac{P(S|+c)P(-w|S,+r)}{\sum_{s}P(s|+c)P(-w|s,+r)}
\end{aligned}
$$

**相关性** <br>
样本链中相邻的样本高度相似，具有相关性