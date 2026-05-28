---
layout: default
title: "隐马尔可夫模型 前向算法"
date: 2026-05-27
tags: [AI, BN]
parent: "笔记 CS188"
nav_order: 12
---

1. 目录
{:toc}

# 隐马尔可夫模型 Hidden Markov Model/HMM

## 马尔可夫模型 MM

引入时间维度

<figure>
  <img src="{{ '/assets/images/algorithms_intro/MM_pic.webp' | absolute_url }}" alt="MM" />
</figure>

**和MDP的比较：**
- 每个节点是$$X_t$$这样和时间有关的状态，类似数列
- 转移概率是$$P(X_{t+1} \mid X_t)$$
- 需要提供初始概率$$P(X_1)$$
- 暂时不考虑行动action
- **稳定性假设**Stationary Assumption：在任意时间，转移函数总是相同的

**和BN的比较：**
- 条件独立性：假设 $$ \text{Past} \perp \!\!\! \perp \text{Future} \mid \text{Present} $$
（或者说当前状态只和上一状态有关。称为一阶马尔可夫性，和D分离有类似之处）
- 实际上就是一个可以无限扩展的BN

<figure>
  <img src="{{ '/assets/images/algorithms_intro/MM_example_pic.webp' | absolute_url }}" alt="MM_example" />
</figure>

递推公式

$$ P(X_t) = \sum_{X_{t-1}} P(X_t \mid X_{t-1}) P(X_{t-1}) $$

这里的转移概率是已知的，所以可以通过数学方法写出通项

> 这其实也是合并两个节点的BN的变量消元法。

### 稳态分布 Staionary Distribution

对于大多数的链状MM，随着时间无限推移，最后的概率会趋于稳定，而且这个收敛的值和初始状态概率无关，只和转移概率有关 <br>
这种情况可以通过求解（不动点法）

$$ \begin{cases} P(X_{t}) = P(X_{t+1}) \\ \sum{P(X_{t})} = 1 \end{cases} $$

得到最后收敛情况的概率分布

这和MDP中的策略迭代算法类似，策略迭代因为有确定的策略，贝尔曼方程里没有max项，变成了线性方程组，求解最后收敛的结果即可

> BN采样中提到的Gibbs Sampling也是类似的方法。

> Google推出的PageRank算法实际上就是构建了个马尔可夫模型，根据网页之间超链接的情况作为转移概率，计算出稳态分布的情况，作为网页排名的依据。

## 隐马尔可夫模型 HMM

<figure>
  <img src="{{ '/assets/images/algorithms_intro/HMM_pic.webp' | absolute_url }}" alt="HMM" />
</figure>

对比MM：
- 没法直接观察到$$X_t$$，只能观测与$$X_t$$有关的证据$$E_t$$
- 因此多了一个概率分布$$P(E_t \mid X_t)$$，这个分布也是稳定的，在任意时间都相同
- 条件独立性增加：当前观测的结果只取决于当前的状态
- 条件独立性注意：证据之间无法保证独立，可以由D分离common cause+chain推理得到

总的来说，HMM的属性是：
- 无法确定$$X_t$$，只能得到$$E_t$$
- 已知的概率信息：
  - 初始概率$$P(X_0)$$
  - 转移概率$$P(X_{t+1} \mid X_t)$$
  - 观测概率$$P(E_t \mid X_t)$$
- 稳定性假设：在任意时间，
  - 转移概率总是相同的
  - 观测概率总是相同的
- 条件独立性：
  - 当前状态只取决于上一状态
  - 当前观测的结果只取决于当前的状态

下面先介绍两个基本的计算过程，HMM的前向算法和粒子滤波都是建立在这两个基本过程上的

### 时间流逝更新 Time Elapse Update

先考虑一个简单的MM，不包含证据 <br>
假设我们知道当前状态的概率分布$$P(X_t)$$，想要知道下一状态的，只需要

$$ P(x_{t+1}) = \sum_{x_t} P(x_{t+1},x_t) $$

$$ = \sum_{x_t} P(x_{t+1} \mid x_t) P(x_t) $$

这个结果很符合直觉，到达下一个状态$$X_{t+1}$$，需要先经历所有可能的$$x_t$$

下面考虑HMM，方法其实也类似，只是多了个证据$$e_t$$作为条件

假设我们根据当前所有的证据（用$$e_{1:t}$$表示），有一个关于当前隐藏状态的推测/信念

$$ B(X_t) = P(X_t \mid e_{1:t}) $$

推进一个时间步，则下一个根据当前所有证据，下一个状态的概率是

$$ P(X_{t+1} \mid e_{1:t})  = \sum_{x_t} P(X_{t+1},x_t \mid e_{1:t}) $$

$$ = \sum_{x_t} P(X_{t+1} \mid x_t e_{1:t}) P(x_t \mid e_{1:t}) \quad \text{(条件概率公式)} $$

$$ = \sum_{x_t} P(X_{t+1} \mid x_t) P(x_t \mid e_{1:t}) \quad \text{(条件独立性：只和上一状态有关)} $$

这个结果很符合直觉，到达下一个状态$$X_{t+1}$$，需要先经历所有可能的$$x_t$$，再通过转移概率移动

如果按照信念概率，写成

$$ B'(X_{t+1}) = \sum_{x_t} P(X' \mid x_t) B(x_t) $$

这个结果就像信念也跟着转移在传递，由$$B(X_t)$$推测下一个时间步的$$B'(X_{t+1})$$

### 观测更新 Observation Update

当获得下一个时间步的观测结果$$e_{t+1}$$，可以更新推测的信念$$B'(X_{t+1})$$

$$ P(X_{t+1} \mid e_{1:t+1}) = \frac{P(X_{t+1},e_{t+1} \mid e_{1:t})}{P(e_{t+1} \mid e_{1:t})} $$

$$ \propto_{X_{t+1}} P(X_{t+1},e_{t+1} \mid e_{1:t}) $$

$$ =P(e_{t+1} \mid X_{t+1},e_{1:t}) P(X_{t+1} \mid e_{1:t}) $$

$$ = P(e_{t+1} \mid X_{t+1}) P(X_{t+1} \mid e_{1:t})  \quad \text{(条件独立性：只和当前状态有关)}$$

也就是

$$ B(X_{t+1}) \propto_{X_{t+1}} P(e_{t+1} \mid X_{t+1}) B'(X_{t+1}) $$

这里是成比例，因为所有的$$x_{t+1}$$都有一项相同的因子$$P(e_{t+1} \mid e_{1:t})$$，最后可以归一化消掉 <br>
所以成比例不影响结果

### 结合：前向算法

<figure>
  <img src="{{ '/assets/images/algorithms_intro/HMM_forward_pic.webp' | absolute_url }}" alt="HMM_forward" />
</figure>

先根据已有的证据推测下一步隐藏变量的情况；然后进行新的观测，更新之前的推测。结合两个更新，就是前向算法