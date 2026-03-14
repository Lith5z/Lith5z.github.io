---
layout: default
title: "使用记录 Latex"
date: 2026-03-14
tags: [教程]
---



一对`$$`插入公式，两对则会强制换到空行
凡是代表同一层级的，用 `{}` 括起来，特别是显示出问题的时候，往往是层级关系搞错了，比如 `e^(-x)` $e^(-x)$ 圆括号不能看作一个整体，要用 `e^{(-x)}` $e^{(-x)}$



### 最基本的
分数 `\frac{分子}{分母}` $f(x) = \frac{1}{1+e^{-x}}$
上标 `a^0` $a^0$
下标 `a_0` $a_0$
希腊字母 `\sigma` $\sigma$
空格 从小到大 `\,` `\quad` 因为所有空格都会被忽略
括号 简单区间直接用 凡是有分数的 `\left( \right)` $\left[ \frac{1}{3}, \frac{1}{2} \right]$
所有字母都会斜体，为了写不斜体 `\text{}` $\text{ReLU}$



### 运算符
加减 直接用
乘法 `\times` $\times$
点乘 `\cdot` $a \cdot b$
除法 `\div` $\div$
方根 `\sqrt[n]{}` $\sqrt{x}$
无穷 `\infty` $\infty$

绝对值 `\lvert \rvert` $\lvert A \rvert = －1$
求和 `\sum` $\sum$ 配合上下标
积分 `\int f(x) \, dx` $\int f(x) \, dx$ 配合上下标
多重积分 `\iint` `\iiint` $\iint_S$ $\iiint_V$
曲线积分 `\oint` $\oint$
极限 `\lim` $\lim_{x \to 0} f(x)$
偏导 `\partial` $\partial$

等于 直接用
约等于 `\approx` $\approx$
不等于 `\neq` $\neq$
属于 `\in` $\in$
不属于 `\notin` $\notin$
包含于 `\subseteq` $\subseteq$
真包含于 `\subsetneq` $\subsetneq$

排列组合 `\binom{}{}` $\binom{a}{b}$



### 函数相关
三角函数、对数函数等 `\cos` $\cos(x)$ 否则会斜体
最大值函数也同理 `\max`
分段函数举例
`f(x) = \begin{cases} x, & \text{if } x > 0 \\ 0, & \text{otherwise} \end{cases}` $\text{ReLU}(x) = \begin{cases} x, & \text{if } x > 0 \\ 0, & \text{otherwise} \end{cases}$



### 线性代数相关
向量 `\vec` $\vec{x}$
如果要长箭头 `\overrightarrow{Easy}` $\overrightarrow{Easy}$
也可以用斜粗体 `\boldsymbol{}` $\boldsymbol{\nabla}$

矩阵 `\begin{pmatrix} a & b \\ c & d \end{pmatrix}` $\begin{pmatrix} a & b \\ c & d \end{pmatrix}$
`&` 用于分隔 `\\` 换行
`pmatrix` 圆括号 `vmatrix` 竖线 `bmatrix` 方括号

省略号
`\cdots` $\cdots$
`\vdots` $\vdots$
`\ddots` $\ddots$

直立粗体 `\mathbf{}` $\mathbf{F}$ 写矩阵



### 其他
映射箭头 `\to` 或 `\rightarrow` $\to$
逻辑蕴含 `\Rightarrow` $\Rightarrow$
双箭头 `\leftrightarrow` $\leftrightarrow$ `\Leftrightarrow` $\Leftrightarrow$
如果箭头上下方还有内容 用 `\xrightarrow[below]{above}` $\xrightarrow[\delta]{\sigma(x)}$