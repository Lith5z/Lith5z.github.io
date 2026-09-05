---
title: "矩阵微积分"
date: 2026-09-05
tags: [数学, 线性代数, 矩阵分析, 微积分, AI]
---

# 矩阵相关的微积分

## nabla算子

读作nabla/del，又称哈密顿算子

首先区分两种函数，**数量值函数和向量值函数**，分别把由单个或多个自变量构成的输入向量

$$\vec x = \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_n \end{bmatrix} \in \R^n$$

映射到一个数和一个向量

$$f : \R^n \rightarrow \R$$

$$\vec f : \R^n \rightarrow \R^m$$

然后定义nabla算子（函数到函数的映射）

$$\nabla = [\frac{\partial}{\partial x_1} , \frac{\partial}{\partial x_2} , \dots , \frac{\partial}{\partial x_n}]^T$$

### 梯度

当nabla算子作用在数量值函数时，得到一个向量值函数 $\nabla f$，这个函数就是原函数的梯度

$$\nabla f = \begin{bmatrix} \frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2} \\ \vdots \\ \frac{\partial f}{\partial x_n} \end{bmatrix} = \text{grad} \, f$$

这一操作在形式上可以看作向量的数量乘法

梯度向量指向原函数增长最快的方向，其大小反映了陡峭程度

### 散度

当nabla算子与向量值函数做内积时，得到一个数量值函数 $\nabla \cdot \vec f$，这个函数是原函数的散度（第四声）

$$\nabla \cdot \vec f = \frac{\partial f_1}{\partial x_1} + \frac{\partial f_2}{\partial x_2} + \dots + \frac{\partial f_n}{\partial x_n} = \text{div} \, \vec f$$

散度是通量的体密度，也就是高斯Gauss公式

$$\oiint_{\partial V} \vec f \cdot \vec n \, dS = \iiint_V \nabla \cdot \vec f \, dV$$

其中 $\partial V$ 代表区域 $V$ 的边界，等式左边是流过该边界的通量，用散度计算，就是右边在区域上 $V$ 的三重积分

### 旋度

当nabla算子和向量值函数都为三维时

$$\nabla = [\frac{\partial}{\partial x}, \frac{\partial}{\partial y}, \frac{\partial}{\partial z}]^T , \quad \vec f = [f_1, f_2, f_3]^T$$

两者做叉乘得到一个新的向量值函数 $\nabla \times \vec f$，这个函数就是原函数的旋度

$$\nabla \times \vec f = \begin{vmatrix} \vec i & \vec j & \vec k \\ \frac{\partial}{\partial x} & \frac{\partial}{\partial y} & \frac{\partial}{\partial z} \\ f_1 & f_2 & f_3 \end{vmatrix} = \begin{bmatrix} \frac{\partial f_3}{\partial y} - \frac{\partial f_2}{\partial z} \\ \frac{\partial f_1}{\partial z} - \frac{\partial f_3}{\partial x} \\ \frac{\partial f_2}{\partial x} - \frac{\partial f_1}{\partial y} \end{bmatrix} = \text{curl} \, \vec f$$

旋度是环量的面密度，也就是斯托克斯Stokes公式

$$\oint_{\partial S} \vec f \cdot d\vec r = \iint_S \nabla \times \vec f \cdot d\vec S$$

左边是围绕边界 $\partial S$ 的环量，右边是在曲面 $S$ 上旋度的贡献

### nabla方

nabla算子和自己做内积

$$\nabla^T \nabla = \frac{\partial^2}{\partial x_1^2} + \dots + \frac{\partial^2}{\partial x_n^2}$$

得到的算子将数量值函数变为数量值函数

nabla算子和自己做外积

$$\nabla \nabla^T = \begin{bmatrix} \frac{\partial^2}{\partial x_1^2} & \cdots & \frac{\partial^2}{\partial x_1 \partial x_n} \\ \vdots & \ddots & \vdots \\ \frac{\partial^2}{\partial x_n \partial x_1} & \cdots & \frac{\partial^2}{\partial x_n^2} \end{bmatrix}$$

这是多元函数的二阶导构成的矩阵，也就是黑塞Hessian矩阵。若 $f$ 二阶连续可微，则混合偏导相等，该矩阵为对称矩阵

上面两种有时都简写成 $\nabla^2$，一般来说 $\nabla^2$ 默认指的是内积，而黑塞矩阵用 $H$ 或 $\nabla \nabla^T$ 单独表示

为了区分，定义拉普拉斯Laplace算子 $\Delta = \nabla \cdot \nabla$，用于nabla算子的内积

## 矩阵和向量的求导

### 定义

下面约定所有向量都是列向量

- 标量 $y$ 对向量 $\mathbf{x}\in\mathbb{R}^n$ 的导数
  $$\frac{\partial y}{\partial \mathbf{x}} = \begin{bmatrix} \frac{\partial y}{\partial x_1} \\ \vdots \\ \frac{\partial y}{\partial x_n} \end{bmatrix} \in \mathbb{R}^n$$

- 向量 $\mathbf{y}\in\mathbb{R}^m$ 对向量 $\mathbf{x}\in\mathbb{R}^n$ 的导数：雅可比矩阵
  $$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y_1}{\partial x_1} & \cdots & \frac{\partial y_1}{\partial x_n} \\ \vdots & \ddots & \vdots \\ \frac{\partial y_m}{\partial x_1} & \cdots & \frac{\partial y_m}{\partial x_n} \end{bmatrix} \in \mathbb{R}^{m\times n} = \begin{bmatrix} (\frac{\partial y_1}{\partial \mathbf{x}})^T \\ \vdots \\ (\frac{\partial y_m}{\partial \mathbf{x}})^T \end{bmatrix}$$

- 标量 $L$ 对矩阵 $\mathbf{W}\in\mathbb{R}^{m\times n}$ 的导数
  $$\frac{\partial L}{\partial \mathbf{W}} \in \mathbb{R}^{m\times n},\quad \left(\frac{\partial L}{\partial \mathbf{W}}\right)_{ij} = \frac{\partial L}{\partial w_{ij}}$$

### 如何计算

在实际计算时，可以根据定义，逐元素写出标量的表达式，然后对向量/矩阵的每个分量求导，再拼成同形状的向量/矩阵。但是这样破坏了向量/矩阵的整体性，而且实操起来拆解表达式非常麻烦

下面简单摘录基于微分的求导方法，并给出几个法则和公式。强烈建议阅读扩展部分的[2]，讲的太好以至于再记更多笔记都只会是复制粘贴

该方法的来源是导数和微分之间的关系：

- 一元微积分中的导数（标量对标量的导数）与微分有：$df = f'(x)dx$
- 多元微积分中的梯度（标量对向量的导数）与微分有：$df = \sum_{i=1}^n \frac{\partial f}{\partial x_i}dx_i = \frac{\partial f}{\partial \boldsymbol{x}}^T \cdot d\boldsymbol{x}$

从而将矩阵导数与微分建立联系：$df = \sum_{i=1}^m \sum_{j=1}^n \frac{\partial f}{\partial X_{ij}}dX_{ij} = \text{tr}\left(\frac{\partial f}{\partial X}^T dX\right)$

其中tr代表迹trace，为方阵对角线元素之和，满足性质：对同形状的矩阵 $\text{tr}(A^TB) = \sum_{i,j}A_{ij}B_{ij}$，即$\text{tr}(A^TB)$是矩阵A,B的**内积**

所以求数量值函数对向量/矩阵的导数，只需要先求微分，然后两侧取迹，再做一些变换得到 $df = \text{tr}\left(\frac{\partial f}{\partial X}^T dX\right)$ 的形式，就可以得到 $\frac{df}{dX}$

然后给出几个矩阵微分运算法则，类似高中学习的导数公式表

1. 加减法：$d(X\pm Y) = dX \pm dY$；矩阵乘法：$d(XY) = (dX)Y + X dY $；转置：$d(X^T) = (dX)^T$；迹：$d\text{tr}(X) = \text{tr}(dX)$
2. 逆：$dX^{-1} = -X^{-1}dX X^{-1}$。此式可在 $XX^{-1}=I$ 两侧求微分来证明
3. 行列式：$d|X| = \text{tr}(X^{*}dX)$，其中 $X^{*}$ 表示X的伴随矩阵，在X可逆时又可以写作 $d|X|= |X|\text{tr}(X^{-1}dX)$
4. 逐元素乘法：$d(X\odot Y) = dX\odot Y + X\odot dY$，$\odot$ 表示同形状的矩阵X,Y逐元素相乘
5. 逐元素函数：$d\sigma(X) = \sigma'(X)\odot dX $，$\sigma(X) = \left[\sigma(X_{ij})\right]$ 是逐元素标量函数运算，$\sigma'(X)=[\sigma'(X_{ij})]$ 是逐元素求导数。例如 $X=\left[\begin{matrix}X_{11} & X_{12} \\ X_{21} & X_{22}\end{matrix}\right], d \sin(X) = \left[\begin{matrix}\cos X_{11} dX_{11} & \cos X_{12} d X_{12}\\ \cos X_{21} d X_{21}& \cos X_{22} dX_{22}\end{matrix}\right] = \cos(X)\odot dX$

迹技巧(trace trick)：

1. 标量的迹是它本身：$a = \text{tr}(a)$
2. 转置：$\mathrm{tr}(A^T) = \mathrm{tr}(A)$
3. 线性：$\text{tr}(A\pm B) = \text{tr}(A)\pm \text{tr}(B)$
4. 矩阵乘法交换：$\text{tr}(AB) = \text{tr}(BA)$，其中 $A$ 与 $B^T$ 同形状。两侧都等于 $\sum_{i,j}A_{ij}B_{ji}$
5. 矩阵乘法/逐元素乘法交换：$\text{tr}(A^T(B\odot C)) = \text{tr}((A\odot B)^TC)$，其中 $A, B, C$ 同形状。两侧都等于 $\sum_{i,j}A_{ij}B_{ij}C_{ij}$

再介绍链式法则用于复合函数求导：

先写出 $df = \text{tr}\left(\frac{\partial f}{\partial Y}^T dY\right)$，再将 $dY$ 用 $dX$ 表示出来代入，并使用迹技巧将其他项交换至 $dX$ 左侧，即可得到 $\frac{\partial f}{\partial X}$

比如 $Y = AXB$，此时

$$df = \text{tr}\left(\frac{\partial f}{\partial Y}^T dY\right) = \text{tr}\left(\frac{\partial f}{\partial Y}^T AdXB\right) = \text{tr}\left(B\frac{\partial f}{\partial Y}^T AdX\right) = \text{tr}\left((A^T\frac{\partial f}{\partial Y}B^T)^T dX\right)$$

可得到 $\frac{\partial f}{\partial X}=A^T\frac{\partial f}{\partial Y}B^T$。注意常量的微分 $dA=0,dB=0$

综合上述的运算法则，可以解决数量值函数对向量/矩阵的导数问题

## 扩展阅读

1. [nabla算子 与梯度、散度、旋度](https://www.bilibili.com/video/BV1a541127cX)
2. [知乎 矩阵求导术](https://zhuanlan.zhihu.com/p/24709748)
3. [Matrix Cookbook](https://www.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf)