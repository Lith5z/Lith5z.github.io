---
layout: default
title: "矩阵和线性变换"
date: 2026-03-14
tags: [数学, 线性代数, AI]
parent: "笔记 线性代数"
nav_order: 3
---

### 线性变换
线性的两个要求：
1. 直线依旧是直线（坐标网格线保持平行且等距）
2. 原点保持固定
实际教材中，线性变换满足的性质是保持**加法和数乘**运算。

只要**确定了基向量**的改变，这个线性变化对其他所有向量的作用就完全确定了。
线性变换的性质保证了，变换后的向量在新基下的**坐标**，和变换前的向量在旧基下的坐标是一致的。也就是说：
> 对于任意的 $$\vec{x} = a\vec{i} + b\vec{j}$$ ，经过线性变换 $$T$$ ， $$\vec{i}$$ 变为 $$\vec{i'}$$ ， $$\vec{j}$$ 变为 $$\vec{j'}$$ ，那么这个新的基下 $$\vec{x'} = a\vec{i'} + b\vec{j'}$$ .

### 矩阵
如果 $$\vec{i'} = \begin{pmatrix} a \\ b \end{pmatrix}$$ 和 $$\vec{j'} = \begin{pmatrix} c \\ d \end{pmatrix}$$ ，这个线性变化就完全由这4个坐标数字确定，我们可以用 $$\begin{pmatrix} a & c \\ b & d \end{pmatrix}$$ 矩阵的形式来表示这个线性变换。

这样，矩阵对向量的乘法就可以这样理解：
> $$\begin{pmatrix} a & c \\ b & d \end{pmatrix} \begin{pmatrix} x \\ y \end{pmatrix} = x \begin{pmatrix} a \\ b \end{pmatrix} + y \begin{pmatrix} c \\ d \end{pmatrix} = \begin{pmatrix} ax+cy \\ bx+dy \end{pmatrix}$$.
> 向量原来的坐标，代表着在原来的基下线性组合的系数。这个线性变化作用在向量上，组合系数不变，基向量改变。
> 也可以看作对新的基向量进行伸缩再相加。

这像是把矩阵写成列向量再做乘法。
在这个视角下，可以快速写出你想要的线性变化对应的矩阵，从而利用这个乘法得到坐标的变化公式。

矩阵之间的乘法则是求两个线性变换的**复合**，找到等效于先后进行这两个线性变换的矩阵。
形式化的说：
> 若矩阵 $$A = \begin{pmatrix} a & c \\ b & d \end{pmatrix},B = \begin{pmatrix} e & g \\ f & h \end{pmatrix}$$ ，对向量 $$\begin{pmatrix} x \\ y \end{pmatrix}$$ ，如果矩阵 $$C$$ 使得 $$AB \begin{pmatrix} x \\ y \end{pmatrix} = C\begin{pmatrix}x \\ y \end{pmatrix}$$，那么定义 $$C = AB$$.
> 这个复合变化是先进行 $$B$$ 的线性变化，然后进行 $$A$$ 的。类似于复合函数的读法，也是从内向外。

那么矩阵乘法可以这样理解：
> $$AB = \begin{pmatrix} a & c \\ b & d \end{pmatrix} \begin{pmatrix} e & g \\ f & h \end{pmatrix} = \begin{pmatrix} \vec{i''} & \vec{j''} \end{pmatrix} = C$$.
> 其中 $$\vec{i''} = \begin{pmatrix} a & c \\ b & d \end{pmatrix} \begin{pmatrix} e \\ f \end{pmatrix} = \begin{pmatrix} ae+cf \\ be+df \end{pmatrix}$$. $$\vec{j''}$$ 与之同理.
> 先对直角坐标系的基进行 $$B$$ 的线性变换，得到的新基就是 $$B$$ 的两个列向量。这两个新的基向量再进行 $$A$$ 线性变换的结果，可以运用上面的结论。

从线性变化的角度看，矩阵乘法的一些性质就是显然的了：
1. 不满足交换律
> $$AB \neq BA$$

比如先旋转再剪切，和先剪切再旋转的效果是不一样的。

2. 结合率
> $$(AB)C = A(BC)$$

乘法是个等效于所有进行的线性变换的线性变换。

### 非方阵
$$A\vec{\beta} = \begin{pmatrix} a & c \\ b & d \\ e & f \end{pmatrix} \begin{pmatrix} x \\ y \end{pmatrix} = \begin{pmatrix} m \\ n \\ u \end{pmatrix}$$.
这个三行两列的矩阵其实是把二维的向量投射到三维去。

也就是说，无论如何，$$A$$ 的列向量依旧代表变换后的基向量，而 $$A$$ 的行数是变换后的基向量的维数。

### 基变换、坐标变换、矩阵变换
> 从线性空间的一组基底 $$\vec{i},\vec{j}$$ 到另一组基底 $$\vec{a},\vec{b}$$ 的过渡矩阵/基变换矩阵为 $$P$$ ，已知向量 $$\vec{p}$$ 在 $$\vec{i},\vec{j}$$ 下的坐标为 $$(x_1,y_1)$$ ，在 $$\vec{a},\vec{b}$$ 下的坐标为 $$(x_2,y_2)$$ ，那么有$$$$\begin{pmatrix} \vec{a} & \vec{b} \end{pmatrix} = \begin{pmatrix} \vec{i} & \vec{j} \end{pmatrix} P$$$$和$$$$\begin{pmatrix} x_1 \\ y_1 \end{pmatrix} = P^{-1} \begin{pmatrix} x_2 \\ y_2 \end{pmatrix}$$$$

对于坐标变换，课本的证明方法是，对于同一个向量 $$\vec{p}$$ ，有 $$\begin{pmatrix} \vec{a} & \vec{b} \end{pmatrix} \begin{pmatrix} x_1 \\ y_1 \end{pmatrix} = \begin{pmatrix} \vec{i} & \vec{j} \end{pmatrix} \begin{pmatrix} x_2 \\ y_2 \end{pmatrix}$$ ，代入过渡矩阵即可。

同一个线性变换，在基 $$\vec{i},\vec{j}$$ 下为 $$A$$ ，那么在基 $$\vec{a},\vec{b}$$ 下为 $$P^{-1}AP$$ ，这就是不同基下的矩阵变换。
这部分没看懂...
