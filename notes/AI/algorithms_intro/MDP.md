---
layout: default
title: "马尔可夫决策过程 MDP"
date: 2026-04-09
tags: [AI, MDP]
parent: "笔记 CS188"
nav_order: 6
---

# 马尔可夫决策过程 MDP

和上一章介绍的确定性决策过程不同，这一章介绍的是不确定的决策过程

一个马尔可夫决策过程 markov decision processes包含：
- 状态state 动作action
- 转移函数 T(s,a,s') 实际上是一个条件概率，P(s'/s,a)，意味着动作的结果不是确定的
- 奖励函数 R(s,a,s')或者也可能是R(s) R(s') 对**每一个**(s,a)给出评价 而所有的奖励总和为**效用值**

在确定性的搜索问题，我们要找一个plan 一个动作序列；在MDP里，需要一个策略policy $$\pi$$ S -> A，描述不同状态下要完成什么action <br>
最优的policy记作pi*

一个马尔科夫链可以转换为搜索树
<figure>
  <img src="{{ '/assets/images/MDPchain_pic.webp' | absolute_url }}" alt="MDPchain_pic" />
</figure>
<figure>
  <img src="{{ '/assets/images/MDPtree_pic.webp' | absolute_url }}" alt="MDPtree_pic" />
</figure>

为了方便处理，搜索树中除了状态节点，还有一个q-state(s,a)，代表从s采取动作后，还未确定结果s'的节点
<figure>
  <img src="{{ '/assets/images/qstate_pic.webp' | absolute_url }}" alt="qstate_pic" />
</figure>

**state节点相当于一个max节点 qstate相当于chance节点**，在后续会阐明

解决MDP的两种算法：
- 价值迭代算法
- 策略迭代算法

## 奖励函数和折扣 Reward func & Discount

获得奖励的先后顺序可能会有影响，所以引入一个折扣因子gamma，之后获得的奖励都乘gamma <br>
比如gamma=0.5 奖励序列[1,2,3] = 1 + 2 * 0.5 + 3 * 0.25 = 2.75 不如[3,2,1]

<figure>
  <img src="{{ '/assets/images/discount_pic.webp' | absolute_url }}" alt="discount_pic" />
</figure>

```
假设初始位置是D
gamma = 1的情况下，向左的奖励序列[0,0,0,10]，总效用值为10；向右[1]，因而向左更佳
gamma = 0.1的情况下，向左收到折扣[0,0,0,0.0001]，向右[0.1]，向右的效用值更大
```

在上面的开车的例子，游戏可能会无限继续，不同的策略可能都会导致无穷大的效用值，无法判断谁更优有三种解决方法：
- 设置action的数量上限，类似深度受限搜索
- 设置discount，这样总和不会是无穷大，折扣也有助于算法收敛
- 保证对于任何策略，都会最终导致终止状态

## 节点价值value的计算 贝尔曼方程

<figure>
  <img src="{{ '/assets/images/qstate_pic.webp' | absolute_url }}" alt="qstate_pic" />
</figure>

s状态的最大价值，是从这个状态开始，可能的最大效用值。用递推的方法，也就是采取所有行动后所有可能的q状态的最大值，这里的s节点相当于一个max节点

$$ V^*(s) = \max_a {Q^*(s,a)} $$

q状态的最大价值，对可能发生的状态s'进行加权（权重是转移函数，也就是概率），价值包含到达s'状态的奖励函数，也包含s'这个状态未来可能的价值。这里的q节点相当于一个chance节点

$$ Q^*(s,a) = \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V^*(s')] $$

我们可以联立两个式子，建立一个方程，把某个状态的V\*与其他状态的V\*联系起来，这个式子叫**贝尔曼方程**

$$ V^*(s) = \max_a \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V^*(s')] $$

- 效用值 utility （折扣后）奖励的总和
- 价值 value 从一个状态开始未来的预期效用值
- q-value 从一个q状态开始未来的的预期效用值

上面只是用递推的方法重新给出了价值的具体定义

## 价值迭代算法 Value Iteration

<figure>
  <img src="{{ '/assets/images/MDPtree_pic.webp' | absolute_url }}" alt="MDPtree_pic" />
</figure>

我们可以使用期望最大化算法，但是有两个明显问题：
- 这个树是无限的（虽然可以用深度受限的方法截断）
- 树实际上只有三个状态，有很多子树是完全相同的，但是在重复出现（而且gamma小于1时，过深的部分影响很小）

价值迭代算法通过记录限制深度的子树的价值$$ V_k(s) $$，解决子树重复的问题。 <br>
其中的k是一个时间步数，代表从这个状态s开始，还能做几次行动 <br>
与期望最大化不同的是，这个算法从（树被截断）底部开始往上计算，首先令k=0，这样$$ V_0(s) = 0 $$，接着开始考虑$$ V_1(s) $$，代表这个状态做一次行动可以得到的最大价值，然后$$ V_2(s) $$.. <br>
其实过程就类似深度受限的expectimax，但是这个过程是可以**复用/迭代**的。比如算$$ V_2(s) $$时无需一直到根节点再返回，进行一次操作后，就相当于$$ V_1(s) $$，所以根据贝尔曼方程：

$$ V_{k+1}(s) = \max_a \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V_k(s')] $$

因而计算每一个$$ V_{k+1}(s) $$，其实就是深度为1的expectimax

举个例子：
<figure>
  <img src="{{ '/assets/images/valueIteration_pic.webp' | absolute_url }}" alt="valueIteration_pic" />
</figure>

```
总共有三种状态：Cool Warm Overheated
过热状态只有一种行动，所以V_0(Overheated) = V_1(Overheated) = V_2(Overheated) = 0

当k=0，三种状态的V_0都为0，因为无法采取行动
因此 V_0(Cool) = V_0(Warm) = 0

当k=1，Cool和Warm状态可以有两种action，slow(R=1)或者fast(R=2) -> 这步相当于计算max_a
对于Cool状态，两种action中，fast能得到的最大R=2
对于Warm状态，两种action，fast会导致过热(R=-10)，故选择slow(R=1)
因此 V_1(Cool) = 2; V_1(Warm) = 1

重点是k=2的时候！
对于Cool状态
如果选择slow，这一步得到+1奖励，还剩一步就相当于V_1(cool) = 2，因而选择slow的总价值为1+2=3
如果选择fast，这一步得到+2奖励，还剩一步有0.5的概率转移到V_1(cool) = 2，有0.5概率转移到V_1(warm) = 1，因此选择fast的总价值为2 + 0.5*2 + 0.5*1 = 3.5
-> 这一步是计算\sum_{s'} T(s,a,s') 之前没有出现过这一项，而且状态可以复用！不需要一路算到V_0
因此 V_2(Cool) = 3.5 -> max_action
同理可以得到V_2(Warm) = 2.5
```
这个例子比较特殊，折扣因子是1，所以结果不会收敛；而且奖励函数是在状态转移时生效的

**当迭代一定次数后，只要gamma<1，v_k就会收敛到v\***

复杂度是O(s^2 a)

## 策略评估和策略提取

为了评估不同策略的优劣，定义一个固定的策略下的效用值：

$$ V^{\pi}(s) = \sum_{s'} T(s,\pi(s),s') [R(s,\pi(s),s') + \gamma V^{\pi}(s')] $$

遵守一个特定的策略pi时，V是相似的，只不过没有了max_a，因为a由pi(s)决定，这实际上已经是一个线性方程

**策略评估 Policy Evaluation** <br>
有了上面固定策略下的效用值迭代式子，就可以通过类似价值迭代的方法得到某个$$V^{\pi}$$：

$$\text{令} V^{\pi}_0(s) = 0$$

$$ V^{\pi}_{k+1}(s) = \sum_{s'} T(s,\pi(s),s') [R(s,\pi(s),s') + \gamma V^{\pi}_k(s')] $$

因为少了对所有action遍历求最大，复杂度是O(s^2)

**策略提取 Policy Extraction** <br>
如果已经知道各个状态下的V*，该怎么得到最优的策略？
<figure>
  <img src="{{ '/assets/images/MDP_value_pic.webp' | absolute_url }}" alt="MDP_value_pic" />
</figure>

这并不是显然的向着value大的地方移动就行，而要遍历所有可能的action，得到哪个action能得到最大的V：

$$ \pi^{*}(s)=\arg\max_{a}\sum_{s'}T(s,a,s')[R(s,a,s')+\gamma V^{*}(s')] $$

（argmax 返回取到最大值的参数a） <br>
只不过因为各个状态下的V\*已知，公式里的V\*(s')是已知的，相当于一步expectimax <br>
特别的，如果知道是Q*，那么可以直接知道策略，因为qstate就是action后的价值

## 策略迭代算法 Policy Iteration

价值迭代存在以下问题（建议看BV1HcqpYwEw6 55min处的讲解）：
- O(S^2 A)的复杂度太高，尤其是State多的时候
- 往往在价值收敛之前，策略就已经得到了收敛

所以给出策略迭代算法，该算法重复策略评估和策略提取两步，直到策略收敛：

$$ V^{\pi_i}_{k+1}(s) = \sum_{s'} T(s,\pi_i(s),s') [R(s,\pi_i(s),s') + \gamma V^{\pi_i}_k(s')] $$

上式进行到V收敛为止

$$ \pi_{i+1}^{*}(s) = \arg \max_{a} \sum_{s'} T(s,a,s') [R(s,a,s') + \gamma V^{\pi_i}(s')] $$

上式根据第一步给出的value提取出最优的策略

- 最优的
- 特定条件下收敛快很多

## 总结

上面提到的内容其实都是贝尔曼方程的变体，也都是一层的expectimax（迭代，V_k已经得知），区别只是遍历了action还是有个确定的策略