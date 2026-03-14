---
layout: default
title: "向量"
date: 2026-03-14
tags: [数学, 线性代数, AI]
---

有两种理解向量的视角——几何视角里，向量是一个有方向和大小的量；代数视角里，向量是一组有序的数的列表。
线性空间的核心发生在这两个视角的互相转换里。

### 线性组合
线性代数紧紧围绕向量**加法和数乘**。
两个数乘向量的和成为这两个向量的线性组合。

### 基向量
事实上，刚刚所说的代数视角，一般默认了直角坐标系。在这个坐标系里，任意的向量都可以表示为 $$\vec{i}$$ 和 $$\vec{j}$$ 的（唯一的）一个**线性组合**，我们把这样的向量称为基向量。
每当我们用数字表示向量的时候，这些数字都依赖于某组基。

### 向量张成的空间
如果我们默认向量的起点是原点，用向量的终点代替整个向量，那么空间中的每个点都对应一个向量，这种对应关系是一个双射。
每个向量全部线性组合构成的向量集合成为“张成的**空间**”。也就是仅通过向量加法和数乘两种**运算**，你可以得到的所有可能向量。
这正体现了代数和几何的转换。

### 线性相关与线性无关
一个向量可以表示为其他向量的线性组合，那么这个向量和其他向量线性相关。
”多余“的理解：向量组中存在这样一个向量，删掉它，其他所有向量依旧可以由一组选定的基线性表出。
从几何的角度来看，这其实是这个向量对于张成的空间没用贡献，它本身就在这个空间内，移除它空间的维数不会减小。

1. 线性表示的定义
> 向量 $$\vec{\beta}$$ 可以由向量组 $$A = \{ \vec{\alpha_1}, \vec{\alpha_2}, ..., \vec{\alpha_n}\}$$ 线性表示，则 $$\vec{\beta} = x_1\vec{\alpha_1} + x_2\vec{\alpha_2} + ... + x_n\vec{\alpha_n}$$ 有非零解。

几何角度，说明 $$\vec{\beta}$$ 落在了这些向量张成的空间里，“则”前面和后面的部分是张成的空间的几种不同定义。

2. 


### 点乘
（以下部分默认向量是二维的）
点乘的几何意义是投影向量和另一个向量的积，代数上是两向量对应元素的乘积的和。

代数上，两个向量点乘和一个 $$1 \times 2$$ 矩阵与向量相乘的结果是一样的。
这个 $$1 \times 2$$ 矩阵对应的线性变换是把**二维向量映射到一维数轴**上，变换后的两个基向量都是一维的。
> $$\begin{pmatrix} a \\ b \end{pmatrix} \cdot \begin{pmatrix} c \\ d \end{pmatrix} = \begin{pmatrix} a & b \end{pmatrix} \begin{pmatrix} c \\ d \end{pmatrix} = ac + bd$$.

在 07-点积和对偶性 中有对代数和几何意义转换的证明。
一句话说，与某个向量做**点乘**，其实就是施加了它**转置后对应的线性变换**。这就是一个**对偶性**。

### 叉乘
对于**二维向量**，其实没有严格的叉乘。一般定义的二维向量的叉乘就是它们围成的平行四边形的有向面积，符合右手定则为正。

> 若 $$\vec{\alpha} = \begin{pmatrix} a \\ b \end{pmatrix} , \vec{\beta} = \begin{pmatrix} c \\ d \end{pmatrix}$$，那么它们叉乘的大小是 $$\begin{vmatrix} a & c \\ b & d \end{vmatrix} = ad-bc$$.
> 因为二维向量叉乘的几何意义和二阶行列式几何意义是一样的。

**严格的叉乘是对于三维向量**的，两个三维向量叉乘得到一个方向符合右手定则，大小正好为围成的平行四边形面积的向量。

> 若 $$\vec{\alpha} = \begin{pmatrix} a \\ b \\ c \end{pmatrix} , \vec{\beta} = \begin{pmatrix} e \\ f \\ g \end{pmatrix}$$，那么它们叉乘的结果可由这个行列式计算： $$\begin{vmatrix} \vec{i} & a & e \\ \vec{j} & b & f \\ \vec{k} & c & g \end{vmatrix}$$.
> 国内的教材常常写成转置的形式，结果是一样的，这样写方便我们后续的论述。

为什么在行列式里插入基向量，并且把基向量当作一个数，最后能得到叉乘向量的结果呢？
类似于点乘和多维到一维线性变换的**对偶性**，我们可以这样解释：
如果行列式第一列不取基向量 $$\vec{i},\vec{j},\vec{k}$$ 而是取一个任意的三维向量 $$\begin{pmatrix} x \\ y \\ z \end{pmatrix}$$，并把这个任意三维向量看作自变量，那么这个行列式就是个把三维向量映射到一维数轴的函数： $$$$f \begin{pmatrix} x \\ y \\ z \end{pmatrix} = \begin{vmatrix} x & a & e \\ y & b & f \\ z & c & g \end{vmatrix}$$$$其几何意义是这三个向量围成的平行六面体的体积。
由于行列式关于每一列都是线性的，所以这个函数也是线性的。那么这个从三维映射到一维的函数，可以理解成向量与某个 $$1 \times 3$$ 矩阵相乘，即$$$$f \begin{pmatrix} x \\ y \\ z \end{pmatrix} = \begin{pmatrix} u & m & n \end{pmatrix} \begin{pmatrix} x \\ y \\ z \end{pmatrix}$$$$用对偶性的思想，这个矩阵是从三维映射到一维的线性变换，那么这个变换必然对应一个三维向量 $$\vec{p} = \begin{pmatrix} u \\ m \\ n \end{pmatrix}$$ 使得$$$$f \begin{pmatrix} x \\ y \\ z \end{pmatrix} = \begin{vmatrix} x & a & e \\ y & b & f \\ z & c & g \end{vmatrix} = \begin{pmatrix} u & m & n \end{pmatrix} \begin{pmatrix} x \\ y \\ z \end{pmatrix} = \begin{pmatrix} u \\ m \\ n \end{pmatrix} \cdot \begin{pmatrix} x \\ y \\ z \end{pmatrix}$$$$现在的问题就是找到 $$\vec{p} = (u,m,n)^T$$ 具体的值，让这些等式成立。
关注$$$$\begin{vmatrix} x & a & e \\ y & b & f \\ z & c & g \end{vmatrix} = x\begin{vmatrix} b & f \\ c & g \end{vmatrix} - y\begin{vmatrix} a & e \\ c & g \end{vmatrix} + z\begin{vmatrix} a & e \\ b & f \end{vmatrix}$$$$ 和$$$$ \begin{pmatrix} u \\ m \\ n \end{pmatrix} \cdot \begin{pmatrix} x \\ y \\ z \end{pmatrix} = ux + my + nz$$$$对比系数就可以得到 $$\vec{p} = (u,m,n)^T$$ 具体的值。

刚刚是代数的方法，从几何的角度思考，我们还可以说明这个向量 $$\vec{p}$$ 的大小就是平行四边形的面积。
因为找这个向量 $$\vec{p}$$ 就是在问：
与哪个向量做点乘，结果是行列式 $$\begin{vmatrix} x & a & e \\ y & b & f \\ z & c & g \end{vmatrix}$$ ？
（最好在脑中想象一个三维空间，并且带有四个向量：已知向量 $$(a,b,c)^T$$ 和 $$(e,f,g)^T$$ ，可以自由移动的自变量向量 $$(x,y,z)^T$$ ，要找的对偶向量 $$\vec{p}$$ ）
从行列式的几何意义上看，这是平行六面体的体积。而点乘把 $$(x,y,z)^T$$ 投影到自变量向量 $$\vec{p}$$ 上。可以断言， $$\vec{p}$$  的大小是两个已知向量围成的平行四边形的面积，方向垂直于两已知向量且符合右手定则。这样 $$(x,y,z)^T$$ 投影的长度得到的就是高，乘积就是体积。且右手定则也保证了体积的有向性。