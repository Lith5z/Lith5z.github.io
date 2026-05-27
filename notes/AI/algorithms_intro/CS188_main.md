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
- Search 搜索
  - 无信息搜索
    - DFS/BFS
    - UCS
    - IDDFS
  - 有信息搜索
    - Greedy
    - A-star
- Constraint Satisfaction Problems/CSP 约束满足问题
  - Backtracking 回溯搜索
    - filter, ordering, structure
  - LocalSearch 局部搜素
    - 爬山法
    - 模拟退火
- Games 博弈搜索
  - Minimax 极小化极大算法
    - Alpha-Beta剪枝 深度受限搜素
  - Expectimax 期望最大化算法
- Markov Decision Problems/MDP 马尔可夫决策过程
  - Value Iteration 价值迭代
  - Policy Iteration 策略迭代
- Reinforcement Learning/RL 强化学习
  - 基于模型
  - 无模型学习
    - Value-Learning
    - Q-Learning 精确/近似
    - Policy Search

**Part II: Uncertainty and Learning**
- Bayes'nets/BN 贝叶斯网络
  - D-Sepreration D-分离/有向分离
    - 三元组
    - 独立性分析
  - Inference 推理
    - Enumeration 枚举推理
    - Variable Elimination 变量消元法
  - Sampling 采样
    - Prior Sampling 先验采样
    - Rejection Sampling 拒绝采样
    - Likelihood Weight 似然加权
    - Gibbs Sampling 吉布斯采样
- Decision Network 决策网络
  - BN + Expectimax
  - Value of Perfect Information/VPI 完美信息价值
- Markov Model/MM 马尔可夫模型
  - BN + ~MDP + Time
  - Staionary Distribution 稳态分布
- Hidden Markov Model/HMM 隐马尔可夫模型
  - Forward Algorithm 前向算法
    - Time Elapse Update 时间流逝更新
    - Observation Update 观测更新
  - Particle Filtering 粒子滤波