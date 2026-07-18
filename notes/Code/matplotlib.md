---
title: "语法 Matplotlib"
date: 2026-07-18
tags: [编程]
---

# Matplotlib

`import matplotlib.pyplot as plt`

## 绘制

plt的行为类似栈，会保存当前图形的情况

画一个图，只需要直接调用plt
```py
x = [1,2,3,4,5,6]
y = [2,3,5,7,11,13]
plt.plot(x,y)
plt.show()
```

如果画多个图，要分别保存为一个变量
```py
fig1 = plt.figure() # 把当前图形的情况保存在fig1
x = [1,2,3,4,5,6]
y = [2,3,5,7,11,13]
plt.plot(x,y) # plot 折线图
plt.show() # 展示 此操作会清空当前图形，最后使用
fig1.savefig(r"D:\Codes\Python-All\test.png") # 保存，写绝对路径/默认是基于运行的相对路径
```

## 图形分类

### plot 折线图

传入x和y两个向量

其他参数：

- `color="#808080"`
- `linestyle='-'` 默认参数，还有`-`为虚线、`:`为点虚线、`-.`为线和点、` `为隐藏（结合标记画散点图）
- `linewidth=2`
- `marker='.'` 数据点处的标记，这个是圆点，还有`o`为大圆，`^`为上三角、`s`为正方形，`D`为菱形
- `markersize=6`

### imshow 网格图

传入一个矩阵，可以画出图像
```py
fig1 = plt.figure()
matrix = np.random.normal(0,10,(64,64))
plt.imshow(matrix)
plt.colorbar() # 显示颜色条
plt.show()
fig1.savefig("test.png")
```

### hist 统计图

传入一个向量
```py
fig1 = plt.figure()
matrix = np.random.randn(1000) # 生成含1000个元素正态分布的数组
plt.hist(matrix)
plt.show()
```

其他参数：

- `bins=20` 区间划分的数量
- `alpha=1` 透明度

## 图窗属性

通过`plt.xlim(start,end)`设置坐标轴的上下限

`plt.title("title")`

`plt.xlabel('x label')`

### 图例

所有的类型的图形，在绘制的时候可以添加参数
`plt.plot(x,y,label='1')`
给线条一个标签，然后通过
`plt.legend()`
绘制图例

legend 还有三个常用的关键字参数：

- loc 用于表示图例位置，该关键字在upper、center、lower 中选一个，在left、center、right 中选一个，用法如 `loc='upper right'`，也可以`loc='best'`
- frameon 用于表示图例边框，去边框是 `frameon=False`
- ncol用于表示图例的列数，默认是1列，也可以通过`ncol=2`调为2列

### 网格

`plt.grid()`