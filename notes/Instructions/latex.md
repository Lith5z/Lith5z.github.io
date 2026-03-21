---
layout: default
title: "Latex使用笔记"
date: 2026-03-14
tags: [教程]
parent: "使用笔记"
---



一对`$$ $$`插入公式，两对则会强制换到空行 <br>
凡是代表同一层级的，用 `{}` 括起来，特别是显示出问题的时候，往往是层级关系搞错了，比如 `e^(-x)` $$e^(-x)$$ 圆括号不能看作一个整体，要用 `e^{(-x)}` $$e^{(-x)}$$



### 最基本的
分数 `\frac{分子}{分母}` $$f(x) = \frac{1}{1+e^{-x}}$$ <br>
上标 `a^0` $$a^0$$ <br>
下标 `a_0` $$a_0$$ <br>
希腊字母 `\sigma` $$\sigma$$ <br>
空格 从小到大 `\,` `\quad` 因为所有空格都会被忽略 <br>
括号 简单区间直接用 凡是有分数的 `\left( \right)` $$\left[ \frac{1}{3}, \frac{1}{2} \right]$$ <br>
所有字母都会斜体，为了写不斜体 `\text{}` $$\text{ReLU}$$ <br>



### 运算符
加减 直接用 <br>
乘法 `\times` $$\times$$ <br>
点乘 `\cdot` $$a \cdot b$$ <br>
除法 `\div` $$\div$$ <br>
方根 `\sqrt[n]{}` $$\sqrt{x}$$ <br>
无穷 `\infty` $$\infty$$ <br>



绝对值 `\lvert \rvert` $$\lvert A \rvert = -1$$ <br>
求和 `\sum` $$\sum$$ 配合上下标 <br>
积分 `\int f(x) \, dx` $$\int f(x) \, dx$$ 配合上下标 <br>
多重积分 `\iint` `\iiint` $$\iint_S$$ $$\iiint_V$$ <br>
曲线积分 `\oint` $$\oint$$ <br>
极限 `\lim` $$\lim_{x \to 0} f(x)$$ <br>
偏导 `\partial` $$\partial$$ <br>



等于 直接用 <br>
约等于 `\approx` $$\approx$$ <br>
不等于 `\neq` $$\neq$$ <br>
属于 `\in` $$\in$$ <br>
不属于 `\notin` $$\notin$$ <br>
包含于 `\subseteq` $$\subseteq$$ <br>
真包含于 `\subsetneq` $$\subsetneq$$ <br>



排列组合 `\binom{}{}` $$\binom{a}{b}$$ <br>



### 函数相关
三角函数、对数函数等 `\cos` $$\cos(x)$$ 否则会斜体 <br>
最大值函数也同理 `\max` <br>
分段函数举例
`f(x) = \begin{cases} x, & \text{if } x > 0 \\ 0, & \text{otherwise} \end{cases}` $$\text{ReLU}(x) = \begin{cases} x, & \text{if } x > 0 \\ 0, & \text{otherwise} \end{cases}$$



### 线性代数相关
向量 `\vec` $$\vec{x}$$ <br>
如果要长箭头 `\overrightarrow{Easy}` $$\overrightarrow{Easy}$$ <br>
也可以用斜粗体 `\boldsymbol{}` $$\boldsymbol{\nabla}$$ <br>



矩阵
```
\begin{pmatrix} a & b \\ c & d \end{pmatrix}
``` 
$$\begin{pmatrix} a & b \\ c & d \end{pmatrix}$$ <br>
`&` 用于分隔 `\\` 换行 <br>
`pmatrix` 圆括号 `vmatrix` 竖线 `bmatrix` 方括号 <br>



省略号
`\cdots` $$\cdots$$ <br>
`\vdots` $$\vdots$$ <br>
`\ddots` $$\ddots$$ <br>



直立粗体 `\mathbf{}` $$\mathbf{F}$$ 写矩阵 <br>



### 其他
映射箭头 `\to` 或 `\rightarrow` $$\to$$ <br>
逻辑蕴含 `\Rightarrow` $$\Rightarrow$$ <br>
双箭头 `\leftrightarrow` $$\leftrightarrow$$ `\Leftrightarrow` $$\Leftrightarrow$$ <br>
如果箭头上下方还有内容 用 `\xrightarrow[below]{above}` $$\xrightarrow[\delta]{\sigma(x)}$$ <br>