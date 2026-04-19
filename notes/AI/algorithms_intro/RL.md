---
layout: default
title: "强化学习 RL"
date: 2026-04-18
tags: [AI, RL]
parent: "笔记 CS188"
nav_order: 7
---

1. 目录
{:toc}

# 强化学习 RL

```
假设两台老虎机，一台100%获得1元；另一台75%获得2元，25%获得0元
这当然可以当作一个MDP来解决，简单计算就知道另一台更优，到了实际游戏时，只需要一直拉另一台

但是不知道如果第二台的概率，就需要在实际游戏里测试，得到数据再反过来解决MDP
这就不是一个规划Planning，而是一个学习Learning过程，具体的说，是强化学习
```

依旧假设问题是一个MDP：
- 状态state 动作action
- 转移函数 T(s,a,s')
- 奖励函数 R(s,a,s')
- 目标是找到策略pi(s)

只不过，**T和R是未知的**，需要通过实际测试才能得到一定数据样本

## 基于模型的学习 Model-Based Learning

针对MDP，模型指的就是T和R函数。但是在RL中，T和R是未知的

但是能通过一定的探索获得信息，比如：
- 计数(s,a,s') 统计得到T(s,a,s')
- 根据到达状态(s,a,s')后的反馈 统计得到R(s,a,s') （注意这里的反馈是人类预先设定好的，agent并不知道具体的函数）

得到T和R，就可以转换为之前所说的MDP问题，通过价值迭代等方法来求策略pi <br>
可以根据这个策略继续进行探索，反复进行

<figure>
  <img src="{{ '/assets/images/model_based_learning_pic.webp' | absolute_url }}" alt="modelBased" />
</figure>

## 无模型学习 Model-Free Learning

**区别** <br>
假设要求一个班级中学生年龄的期望 <br>
两种学习方法都需要采样 $$ a_1, a_2, ... ,a_n $$ （$$a$$是年龄，下标为一种索引）

基于模型的学习： <br>
根据采样估计出某个年龄的概率 $$ P(a) = nums(a) / n $$ <br>
然后对各个年龄a加权求和 $$ E(a) = \sum_a P(a) \cdot a$$

无模型学习： <br>
直接根据采样计算 $$ E(a) = \frac{1}{n} \sum_i a_i $$

这里的概率P就相当于之前的T R

两种方法：
- Value-Learning
- Q-Learning

### 价值学习 Value Learning/被动强化学习

在**固定的策略**下，不断进行游戏，统计(state, value)，目标是得到**状态的价值**，而非求出T R然后再求解MDP（基于模型） <br>
但是单通过价值得不到最优策略（结尾部分会解释） <br>
这不是规划，因为agent实际上进行了游戏

根据统计到的状态，有两种计算价值的办法

**直接评估 Direct Evaluation**
<figure>
  <img src="{{ '/assets/images/direct_evaluation_pic.webp' | absolute_url }}" alt="direct_evaluation" />
</figure>

```
以状态C为例，总共有4个样本，价值总计为三个[-1, 10]=9 一个[-1, -10]=-11，取平均得到V_c = 4
注意，价值是从一个状态开始未来的预期效用值，也就是把后续所有reward相加

这里因为样本少出现一个问题：if B and E both go to C under this policy, how can their values be different?
```

这样做的缺点是：
- 只用到了state和对应的reward，浪费了状态转移的信息
- 需要很长的时间，但是只要样本足够，最后可以给出正确的结果



**时间差分学习 Temporal Difference Learning/间接评估**

既然有固定的策略，为什么不用策略评估来计算状态的价值？

$$ V^{\pi}_{k+1}(s) = \sum_{s'} T(s,\pi(s),s') [R(s,\pi(s),s') + \gamma V^{\pi}_k(s')] $$

因为不知道具体的T() R()

时间差分学习是一种可以在不知道T R的情况下，用类似价值评估得到价值的方法 <br>
核心想法是每次行动后就更新V(s)，使用一种动态求平均值的方法

首先初始化所有的$$V^\pi(s) = 0$$ <br>
每次行动获得一个样本(state, action, state', R)，比如(B, east, C, -2)，计算

$$ \text{sample (of V(s))} = R(s,\pi(s),s') + \gamma V^{\pi}(s') $$

（一开始$$ V^{\pi}(s') $$为0，直接带入；之后发生了更新就带入更新后的值） <br>
这个样本代表V(s)的一种可能，我们想要的是平均的结果，故这样动态更新$$ V^{\pi}(s) $$

$$ V^{\pi}(s) \leftarrow (1-\alpha)V^{\pi}(s) + \alpha \cdot \text{sample} $$

或者写成

$$ V^{\pi}(s) \leftarrow V^{\pi}(s) + \alpha \cdot (\text{sample} - V^{\pi}(s)) $$

也就是部分保留了原先的值，也进行了部分的更新，随着alpha的下调，最后$$ V^{\pi}(s') $$可以得到一个收敛的值

这里的alpha叫学习率 <br>
$$(1 - \alpha) < 1$$，越早得到的sample占最后$$ V^{\pi}(s') $$的权重逐渐缩小，越后面获得的样本越重要 <br>
这也是说得通的，因为越后面获得的样本，带入计算的$$ V^{\pi}(s') $$也是多次更新的结果，也就更接近正确结果

<figure>
  <img src="{{ '/assets/images/temporal_difference_pic.webp' | absolute_url }}" alt="temporal_difference" />
</figure>

```
这个例子里，假设gamma=1, alpha=0.5
一开始V^pi(D)有初始值

(B, east, C, -2)计算略，这里展示(C, east, D, -2)的计算过程
sample = -2 + 1 * 8 = 6
更新V^pi(C) = (1-0.5) * V^pi(C) + 0.5 * sample = 3

可以看到，尽管这一步得到的R=-2 < 0，但是更新的V(C)变大了
这依旧是价值的定义：从一个状态开始未来的预期效用值，而这里D有较大的价值
```

**得不到策略** <br>
然而，有了价值，我们还是无法进行策略提取

$$ \pi^{*}(s)=\arg\max_{a}\sum_{s'}T(s,a,s')[R(s,a,s')+\gamma V^{*}(s')]$$

依旧是因为T未知

所以这个方法是不完整的，只能得到价值而得不到策略 <br>
（作为对比，策略迭代算法同时进行了策略评估和提取） <br>
这时候就需要学习Q-Value，因为知道了所有的Q-Value也就知道了最优策略。又因为不同Q-Value需要在同一个状态下做不同的行动（Q-State），不能按照固定的策略进行，所以这是一个主动学习

### Q-Learning/主动强化学习

最终可以得到最优的**策略和价值**

需要权衡
- 探索Exploration 尝试未知的行动以获取信息
- 利用Exploitation 信息足够后，最终要运用已掌握的知识做出决策

**Q-Value Iteration** 基于模型的利用 <br>
类似价值迭代，从Q价值出发，需要已知转移模型T和奖励模型R <br>
通常设初始化$$Q_0(s,a) = 0$$

$$ Q_{k+1}(s,a) = \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma \max_{a'} Q_k(s',a')] $$

直到$$Q_{k+1}(s,a)$$稳定后，就是$$Q(s,a)$$

公式的推导其实就是把迭代形式的贝尔曼方程反过来带入：

$$ Q_{k+1}(s,a) = \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V_{k}(s')] $$

其中

$$ V_{k}(s') = \max_{a'} {Q_{k}(s',a')} $$

这是最核心的替换

**Q-Learning** 无模型学习 <br>
在探索中收集样本(s,a,s',r)，依旧通过**时序差分**进行更新

$$ \text{sample} = R(s,a,s') + \gamma \max_{a'} Q(s',a') $$

$$ Q(s,a) \leftarrow (1 - \alpha)Q(s,a) + \alpha \cdot \text{sample} $$

这里的更新目标（max）是在利用已有知识，而数据采集过程包含探索

最终，**最优策略**由收敛后的Q值导出：$$\pi^*(s) = \arg\max_a Q(s,a)$$ <br>
> 特别的，如果知道是Q*，那么可以直接知道策略，因为qstate就是action后的价值（MDP笔记）

另外，公式中的max保证了无论本次行动是什么，Q值都是根据目前为止最优的行动更新的，所以Q-Learning是不需要策略的 off-policy learning <br>
只要探索的足够多，学习率一开始设置的足够小，且逐步减少（让更新量变小，保证收敛），这样子无论怎么选择探索的行动都可以得到最优策略

**强烈建议去看BV1HcqpYwEw6 最后20分钟的演示，讲解非常清楚**

## 如何权衡探索和利用

### Epsilon-Greedy

每一步都进行先进行一次判定，有$$\varepsilon$$的可能随机行动，剩下的可能按着（目前已经知道的）最优策略行动

$$\varepsilon$$在一开始应该设置的比较大，从而多进行探索；一段时间后应当调小，才能反映出学习的成果（不然可能策略已经收敛，还在随机行动）

### Exploration Function

显而易见更好的做法是，对于已经探索过足够次数的地方，就不在选择随机探索 <br>
用探索函数$$ f(u,n) = u + \frac{k}{n} $$代替

$$ \text{sample} = R(s,a,s') + \gamma \max_{a'} f(Q(s',a'),N(s',a')) $$

其中u是价值，n是被探索的次数，k是一个常量

这个函数相当于给价值附带了一项奖励，当没怎么被探索时，奖励更大，更新的Q-Value就更大（准确的说这里的Q-Value多包含了一部分）

## Approximate Q-Learning 近似Q学习

之前提到的都是精确的方法，进行Q-Learning的时候，要维护所有的Q-State，总数量是状态数×行动数，这可能完全无法容纳 <br>
近似方法的想法是，先学习一部分状态，遇到相似的新状态时根据已有经验来推断策略（泛化Generalize），而不是认为这是个完全不同的状态，这其实就是机器学习Machine Learning的思想

<figure>
  <img src="{{ '/assets/images/naiveQLearning_pic.webp' | absolute_url }}" alt="naiveQLearning" />
</figure>

引入一个类似评估函数的方法，设置一系列的特征feature/property，对于吃豆人，可能是：
- Distance to closest ghost
- Distance to closest dot
- Number of ghosts
- 1/(dist to dot)²
- ...

这些特征是把state映射到$$R:[0,1]$$上的函数，然后对这些特征加权求和，作为Q-Value

$$ Q(s,a) = w_1f_1(s,a) + w_2f_2(s,a) + \dots + w_nf_n(s,a) $$

这些权重通过类似Q-Learning的方法自动调整 <br>
对于一个样本(s,a,s',r)，预测的特征值Q(s,a)，计算与样本给出的Q之间的差异

$$ \text{difference} = [r + \gamma \max_{a'} Q(s',a')] - Q(s,a) $$

然后遍历更新

$$ w_i \leftarrow w_i + \alpha \cdot \text{difference} f_i(s,a) $$

当$$\text{diff}<0$$，说明预测的Q更大，权重会被调小 <br>
$$f_i(s,a)$$的大小影响改变量，这个样本下越活跃的特征，会主导权重的改变

这个公式的推导类似最小二乘法，此处略去。另外这个更新的方法就是一种机器学习

<figure>
  <img src="{{ '/assets/images/RLapproximate_pic.webp' | absolute_url }}" alt="RLapproximate_pic" />
</figure>

这样就通过少量的特征权重数据概括了大量的Q-State，学习速度会快很多 <br>
但是如果特征太少或者不恰当，很可能让完全不相似的情况有一样的特征值，就没办法区分；如果特征太多，样本又不够多，可能会发生过拟合

## Policy Search 策略搜索

之前针对Q价值进行学习，但其实数值本身不重要，重要的是Q价值的排序反映的action <br>
比如吃豆人项目里，近似Q值可能非常离谱（比如在绝境给出+100），但只要它在危险动作上给的分数更低，在吃豆动作上给的分数更高，那给出的策略就是对的 <br>
为何不直接去学那个最大化reward的策略本身？这就是策略搜素

最简单的策略搜索，先从一个已经学习好的Q函数和策略出发，爬山法微调权重，再直接执行策略，如果效果更好就保留 <br>
但是分辨策略更好与否可能需要大量测试，在样本效率上比Q-Learning低，而且如果权重很多，这可能就不可能了

# 总结

已知的MDP：
- 求解V* Q* pi*：价值迭代/策略迭代
- 根据固定的pi评估价值：策略评估

未知的MDP：
- 基于模型：通过样本平均值求出T R，转换为已知情形
- 无模型：
  - 求解V* Q* pi*：Q-Learning（精确/近似）
  - 根据固定的pi评估价值：Value-Learning（直接评估/时间差分学习）
  - 求解pi*：Policy Search