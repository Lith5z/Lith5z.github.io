---
layout: default
title: "对抗搜索 Expectimax"
date: 2026-04-07
tags: [AI, GraphSearch, Expectimax]
parent: "笔记 CS188"
nav_order: 5
---

# 期望最大算法 Exceptimax

之前的minimax假设两个玩家都是optimal的，但是如果对手有一定犯错的**概率**，就无法计算出可能的更好结果，比如：

```
        Max
       /   \
     Min   Min
    / \     / \
   +3  +6 +2  +100

右侧的min节点，如果玩家有一半的概率犯错，max就可以得到+100，就算没犯错也是+2，没有比+3差太多
但是minimax算法总会让max选择+3的这条路径
```

为了解决带有随机性的博弈问题，需要对minimax做出这下调整：
- max节点的表现完全相同
- 对手的chance节点不会选择最小效用值的后继节点，而是按着一定概率选择。因此计算这个节点的值，要用加权平均的方法计算期望

从而expectimax的结果是
```
        Max
       /   \
   chance  chance
    / \     / \
   +3  +6 +2  +100
如果概率均等，左chance节点+4.5，右+51，max会选择右作为下一步
```

## 不能用alpha-beta剪枝

min节点获取到一个+3的后继之后，就可以确定min节点的值肯定不比+3大；但是chance节点没有生成所有子节点之前不能确定范围，所以不能用alpha-beta剪枝

## 和minimax

minimax适用optimal博弈的对手，而expectimax适用带有概率性的对手
<figure>
  <img src="{{ '/assets/images/expectimaxCompare_pic.webp' | absolute_url }}" alt="compare" />
</figure>
minimax只看节点效用值的排序顺序，而不关心具体大小；expectimax则相反。这个例子里，minimax的决策不会因为效用函数的变化而改变