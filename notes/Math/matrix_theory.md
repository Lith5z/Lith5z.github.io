---
title: "矩阵理论"
date: 2026-09-05
tags: [数学, 线性代数, 矩阵分析]
---

# 矩阵理论

先简单介绍一下几个从实数域到复数域上不同的概念

$A^H = \overline{A^T}$ 共轭转置，对于实矩阵 $A^H=A^T$

对称矩阵 $A^T=A$；厄密矩阵Hermitian Matrix $A^H=A$

正交矩阵 $A^T A=I$；酉矩阵 $A^H A=I$

## 奇异值

奇异值定义为

$$\sigma_i = \sqrt{\lambda_i(A^H A)} = \sqrt{\lambda_i(A A^H)}$$

即矩阵 $A^H A$ 的特征值开根

行列式绝对值等于奇异值的乘积，即

$$|\text{det}(A)| = \prod_i \sigma_i$$

上式对 $A^H A$ 取行列式，再利用 $\text{det} = \prod_i |\lambda_i|$ 即得

注意，当 $A$ 是对称矩阵时，才有 $\sigma_i = |\lambda_i|$

## 向量范数

范数是用于度量大小的概念

除了向量的模（实际上就是L2范数）外，还有其他的向量范数

设 $x = (x_1,x_2,\cdots,x_n)^T \in C^n$，规定p范数为 $||x||_p = (\sum_k |x_k|^p)^{1/p} \quad (1 \leq p \leq \infty)$，常用的范数有

$$\text{L1范数：} \, ||x||_1 = \sum_k |x_k|$$

$$\text{L2范数：} \, ||x||_2 = \sqrt{\sum_k |x_k|^2}$$

$$\text{L} \infty \text{范数：} \, ||x||_{\infty} = \max_k |x_k|$$

### 用途

[[正则化]]中，在损失函数添加 $||\omega||_1$ 或 $||\omega||_2^2$ 项，称为L1、L2正则化。这里的 $\omega$ 是参数矩阵展平后得到的向量

## 矩阵范数

$$\text{F范数：} \, ||A||_F = \sqrt{\sum_i \sum_j |x_{ij}|^2}$$

相当于把矩阵视作一个长向量，然后按照L2范数计算

定义诱导范数，通过向量范数导出矩阵范数的定义

$$||A|| = \max_{||x||=1} ||Ax||$$

其中 $||\cdot||$ 为向量范数

诱导范数度量矩阵对向量拉伸的最大倍数，分别由向量的L1、L2、L无穷范数，得到矩阵的1、2、无穷范数如下

$$\text{1-范数/极大列和范数：} \, ||A||_1 = \max_j \sum_i |a_{ij}|$$

$$\text{2-范数/谱范数：} \, ||A||_2 = \sqrt{\lambda_{\max}(A^H A)}$$

H为共轭转置（因为在复数域中讨论），2-范数也即最大奇异值

$$\infty\text{-范数/极大行和范数：} \, ||A||_{\infty} = \max_i \sum_j |a_{ij}|$$

> 和[[行列式]]的区别：行列式衡量线性变换对体积的缩放，诱导范数只关注缩放程度最大的某个方向。用公式表达，$|\det(A)| = \prod \sigma_i$，而 $||A||_2 = \max \sigma_i$ 只取最大。
>
> 比如 $A = \begin{bmatrix} 100 & 0 \\ 0 & 0.01 \end{bmatrix}$ 的谱半径是100，行列式是1。体积不缩放，但是一个方向上缩放剧烈。

## 矩阵的谱半径

$$\rho(A) = \max_i |\lambda_i|$$

描述线性变换沿不变方向（特征向量方向）的最大缩放效应，谱半径不大于任意一种矩阵范数

$A \in C^{n \times n}$，若满足 $\lim_{k \to \infty} \, A^k = 0$，则 $A$ 为收敛矩阵

收敛矩阵的充要条件为 $\rho(A) \lt 1$

### 用途

根据谱半径和1的大小关系，可以估计梯度在传播过程中的行为

## 矩阵的条件数

$$\text{cond}(A) = ||A|| \cdot ||A^{-1}||$$

不同范数定义不同的条件数，条件数总是大于等于1

条件数很大时，A被称作病态矩阵；反之为良态矩阵

### 定义推导

求解线性方程组 $Ax = b$ 其中 $b$ 存在小误差（噪声）$\Delta b$，导致求解产生误差 $\Delta x$，即

$$A(x + \Delta x) = b + \Delta b$$

代入 $Ax = b$ 得

$$\Delta x = A^{-1} \Delta b$$

两边取范数，得绝对误差的不等式

$$\text{1:} \quad ||\Delta x|| \leq ||A^{-1}|| \cdot ||\Delta b||$$

现在推导相对误差，由 $b = Ax$ 得

$$||b|| \leq ||A|| \cdot ||x|| \quad \Rightarrow \quad \text{2:} \quad \frac{1}{||x||} \leq \frac{||A||}{||b||}$$

一二两式相乘，整理即得定义

$$\frac{||\Delta x||}{||x||} \leq \left( ||A|| \cdot ||A^{-1}|| \right) \cdot \frac{||\Delta b||}{||b||}$$

因而 $||A|| \cdot ||A^{-1}||$ 恰好就是输入相对误差 $\frac{||\Delta b||}{||b||}$ 被放大到输出相对误差 $\frac{||\Delta x||}{||x||}$ 的最坏情况倍数