---
layout: default
title: "笔记 CS188"
date: 2026-03-25
tags: [AI, Algorithms]
---

学校开设的人工智能导论课程很一般，而且是由伯克利CS188修改而来的，那我为什么不直接看CS188的录课？

课程在B站上就有，建议看2018版

参考用书：Artificial Intelligence: A Modern Approach/AIMA

**Part I: Search and Planning**
- Search
  - 无信息搜索
    - DFS/BFS
    - UCS
    - IDDFS
  - 有信息搜索
    - Greedy
    - A-star
- Constraint Satisfaction Problems/CSP
  - Backtracking 回溯搜索
    - filter, ordering, structure
  - LocalSearch 局部搜素
    - 爬山法
    - 模拟退火
- Games
  - Minimax 极小化极大算法
    - Alpha-Beta剪枝 深度受限搜素
  - Expectimax 期望最大化算法
- Markov Decision Problems/MDP
  - Value Iteration 价值迭代
  - Policy Iteration 策略迭代
- Reinforcement Learning/RL
  - 基于模型
  - 无模型学习
    - Value-Learning
    - Q-Learning 精确/近似
    - Policy Search

**Part II: Uncertainty and Learning**