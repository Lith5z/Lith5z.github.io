---
title: "语法 pandas"
date: 2026-07-18
tags: [编程]
---

# pandas

pandas可以在numpy数组的基础之上，提供标签数据，类似从列表到字典

`import pandas as pd`

## 类型

根据标签维度的数量，分为一维对象Series（和字典的功能一致）和多维对象DataFrame（类似表格）

pandas中标签values、行标签index、列标签columns都是通过numpy数组储存的
使用`.values` `.index` `.columns`获取标签和索引，类型都是np数组

## 创建

### 一维对象

类似numpy通过列表创建数组，pandas可以通过字典创建一维Series对象
```py
sr1 = pd.Series({"Apple" : 8, "Pear" : 10, "Cherry" : 12}) # 传入字典
# Apple      8
# Pear      10
# Cherry    12
# dtype: int64
```

或者分别传入numpy数组/py列表/tensor张量，作为值和行标签创建
```py
idx = ["Apple", "Pear", "Cherry"]
val_price = [8, 10, 12]
sr2 = pd.Series(val_price, idx)
# 传入索引可以为空，则默认从0开始
```

### 二维对象

可以通过把多个Series对象拼接为一个字典的方式创建DataFrame对象
```py
idx = ["Apple", "Pear", "Cherry"]
val_price = [8, 10, 12]
val_taste = ["Normal", "Good", "Bad"]
sr1 = pd.Series(val_taste, idx)
sr2 = pd.Series(val_price, idx)

df1 = pd.DataFrame({"taste": sr1, "price": sr2}) # 传入字典
#         taste  price
#Apple   Normal      8
#Pear      Good     10
#Cherry     Bad     12
```

或者分别传入值values（np矩阵）、行标签index、列标签columns创建
```py
idx = ["Apple", "Pear", "Cherry"]
val_price = np.array([8, 10, 12]).reshape(-1,1)
val_taste = np.array(["Normal", "Good", "Bad"]).reshape(-1,1)
val = np.concatenate([val_price, val_taste],1) # 拼接成一个矩阵
col = ["price", "taste"]
df1 = pd.DataFrame(val, idx, col)
print(df1)
```
上述代码的问题是，通过拼接为一个np数组val，两个列都被自动转换成了字符串类型，导致后续无法进行数值运算。上述代码仅作为一个示例，实际上逐列添加更好（避免了混合类型）

## 访问

numpy支持的所有索引的功能，pandas都支持：花哨索引、切片是视图等

pandas的对象既可以通过行列索引（显式索引 `.loc[]`）来访问，也可以通过数组本身从0开始的索引（隐式索引 `.iloc[]`）访问
对于一维对象，在不会混淆的情况下，可以直接用`[]`访问；二维对象必须使用索引器`df.loc['a', 'b']`

注意，使用显式索引进行切片，最后一个位置是包括的：
```py
print(df1.loc['Apple':'Pear']) # 如果只对行/列提取，可以省略另一部分
#        taste  price
#Apple  Normal      8
#Pear     Good     10
```
特别的，提取一个列，可以不写索引器，结果退化为一个一维对象
这个写法很常用
```py
print(df2['price'])
#Apple      8
#Pear      10
#Cherry    12
```

## 变形

`.T` 转置
`df = df.iloc[ : ，: : -1 ]` 左右翻转（行全保留，列使用-1为step相当于倒序）
`df = df.iloc[ : : -1 , : ]` 上下翻转

添加单个列
用访问里提到的提取列的方法，可以对二维对象添加一个列
`df2['number'] = pd.Series(val_num, idx)`

添加单个行
`df2.loc['Orange'] = [Good, 8]`

合并对象
`sr3 = pd.concat([sr1, sr2])` 语法和numpy一致
注意如果索引重复，会保留
可以使用`.index.is_unique`检查

## 运算

对每个元素逐个运算，只有数值类型的列才能运算

如果有类型不匹配无法运算的列，要单独提取出来运算
`df['number'] = df['number'] * 2`

二维对象的运算，维度不匹配时的结果是缺失值NaN

np中提供的函数，在pd中也有，只不过变为了对象的方法（并且忽略缺失值）
np.sum(arr) -> sr.sum()

## 缺失值

`.isnull()`
结果以布尔型存储

`.dropna()` 剔除缺失值
对于二维对象来说，默认是去掉含缺失值的行（去掉这一个样本）；如果需要去掉列，添加参数`axis=1`
如果想要这一行全是NaN才删除，添加参数`how="all"`

`.fillna(val)` 以val填充缺失值
注意返回的是一个副本，要`X = X["Age"].fillna(X["Age"].median)`
也可以不指定val，添加参数`method="ffill"`/`"bfill"`用前/后一个值填充

## 读写CSV

1. excel表格（必须有行列索引）导出为csv
2. 使用`df = pd.read_csv('filename.csv', index_col = 0)`读入文件

这里的index_col是要不要自动补充一个编号列，如果数据集中已经有了编号的列，就传入0

`df.to_csv('filename.csv')`

## 简单数据分析

具体操作可见 DL概述.md

print的时候可以`print(df.head())`，只显示开头一部分数据

除了np提供的几个函数之外，可以使用`.describe()`显示基本信息

`.pivot_table('title', index=..., column=..., aggfunc=mean)`
对给定的index，进行给定函数（默认是mean均值）来数据透视分析

比如泰坦尼克的例子：
`df.pivot_table("是否生还", index='gender')`
得到的就是表格
```
        是否生还
gender
female 0.742038
male   0.188908
```

对于连续而非分类用途的值，比如年龄、费用，需要先通过`pd.cut()`函数得到类别型数据，手动给出区间的界限；或者`pd.qcut()`给定区间数量自动分类
比如：
`age = pd.cut( df['age'], [0,25,120] )`
然后作为一个行索引传入
`df.pivot_table('是否生还', index=['性别',age], columns='船舱等级')`