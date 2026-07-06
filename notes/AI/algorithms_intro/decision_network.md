---
title: "决策网络"
date: 2026-05-21
tags: [AI, BN]
---

# 决策网络 Decision Network

## 决策网络

把贝叶斯网络和Exceptimax结合

<figure>
  <img src="/assets/images/algorithms_intro/DN_example_pic.webp" alt="DN_example" />
</figure>

三种节点：

- Chance Node：类似BN中的节点，带有一个条件概率表
- 动作节点 Action Node：由agent决定进行什么行动，没有父节点
- 效用节点 Utility Node：由父节点决定效用值

决定决策的方式就是期望最大化，选择在给定的证据下，能得到最大期望效用值 Maximum Expected Utility/MEU的行动

## 效用值计算

- 实例化证据
- 遍历所有可能的action
    - 通过给定证据，推理得到需要的概率
    - 结合目前选定的action计算效用值
- 选择使效用值最大的action

$$ \text{MEU}(e) = \max_a \text{EU}(a) = \max_a \sum_s P(s \mid e) U(s,a) $$

上图比较特殊，公式里的概率$P(s)$直接给出，一般来说要通过BN的特性推理得到；而且没有已知的证据（MEU的空集符号表示）

<figure>
  <img src="/assets/images/algorithms_intro/DN_example2_pic.webp" alt="DN_example2" />
</figure>

这里，给定的证据Evidence是Forecast = bad，要根据这个证据计算出$P(w \mid \text{bad})$

可以看出，已知的证据改变，最后的MEU也发生了改变。在这里例子里，最佳的动作leave变成了take <br>
这说明，获得信息Evidence会改变效用值，有时候还会改变决策行动

## 完美信息价值 VPI/Value of Perfect Information

在已知某个随机变量E=e的时候，MEU是

$$ \text{MEU}(e) = \max_a \sum_s P(s \mid e) U(s,a) $$

对于其他某个随机变量E'，如果我们知道它的**具体**取值E'=e'，此时MEU变成

$$ \text{MEU}(e,e') = \max_a \sum_s P(s \mid e,e') U(s,a) $$

但是对于这个随机变量E'，获得信息只是确定E'到底取哪个e'的过程，但在此之前，没法获得信息会得到哪个e'，要进行加权平均

$$ \text{MEU}(e,E') = \sum_{e'} P(e' \mid e) \text{MEU}(e,e') $$

所以获得信息后，改变的效用值是

$$ \text{VPI}(E' \mid e) = \text{MEU}(e,E') - \text{MEU}(e) $$

<figure>
  <img src="/assets/images/algorithms_intro/DN_VPI_pic.webp" alt="DN_VPI" />
</figure>

在之前的例子里，E'就是机会节点Forecast，e'可能是bad/good <br>
得到bad会降低MEU，得到good会提升，但是揭示这个E'的信息到底能得到bad/good是不确定的，要加权平均 <br>
最后的差是7.8，也就是如果花7.8的效用值去揭示Forecast节点的信息，是不亏不赚的

性质：

- 非负性： $ \forall E^{\prime},e:VPI(E^{\prime} \mid e)\geq0 $
- 不可加性： $ VPI(E_j,E_k \mid e)\neq VPI(E_j \mid e)+VPI(E_k \mid e) $
- 顺序独立性：$ VPI(E_j,E_k \mid e)=VPI(E_j \mid e)+VPI(E_k \mid e,E_j)=VPI(E_k \mid e)+VPI(E_j \mid e,E_k) $

### 和BN的关系

如果获得某个信息不改变决策，那么这个信息的VPI为0

可以通过BN的D分离，定性分析获得的信息有多大价值/没有价值

<figure>
  <img src="/assets/images/algorithms_intro/DN_VPI2_pic.webp" alt="DN_VPI2" />
</figure>

```
上图里，假设OilLoc只有左右两个均等的可能，DrillLoc也只有左右两个行动
没有信息的时候，记MEU是K/2

得到OilLoc的信息，MEU直接变成K，所以这个信息的价值是K/2
如果得到的是勘探人员的水平Scout的信息，在这个BN里，Scout和OilLoc条件独立，价值为0

如果在Scout基础上得到了共同原因ScoutingReport的信息
此时变为相关的，价值又不是0了
```

一般来说，如果要获得的信息，在已知证据下，和效用节点的父节点条件独立，那么这个信息的VPI为0