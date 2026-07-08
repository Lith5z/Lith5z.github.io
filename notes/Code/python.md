---
title: "语法 Python"
date: 2026-07-08
tags: [编程]
---

# Python 速通

建立在熟悉c/cpp的基础之上，面向ML/DL的需要学习Python

来自于[爆肝杰哥 Python基础](https://www.bilibili.com/video/BV1K14y1c75e)

## 基本变量类型

str int float bool
所有的基本变量可以通过`str(123)`这样的写法进行类型转换
转换成bool为`False`的条件是数字为0，或者其他类型为空

另外还有`None`，表示"空"（类似c/cpp的`NULL`/`nullptr`），判断用`is None`

### 字符串

用单双三引号包裹

**f字符串**：允许插入`{}`作为参数
```py
str1 = "i am a man"
str2 = f"do you think {str1}"
```
最常用在`print()`中

**分割和连接**
```py
str1 = "hello world"
str1.split()        # ['hello', 'world'] - 分词，非常常用
str1.strip()        # 去除首尾空白
",".join(["a","b"]) # 'a,b' 拼接
```

转义字符`\n` `\t`也同样支持

### 数字

int和float

除法4/3的结果和c/cpp不一样，会返回浮点数
`//`整除 `%`取余

### 布尔类型

首字母大写，是`True`和`False`而不是c/cpp的`true`

`and` `or` `not` 对应 c/cpp 的 `&&` `||` `!`
`&` `|` 在py中仍是按位与/或（和c/cpp一样），不要混用于逻辑判断

## 高级变量类型

set tuple dict list
作为容器，内部可以储存任意类型的数据

这几个类型可以任意转换
转换成字典需要用`dict1 = dict(zip(keys, vals))`通过`zip`将键列表和值列表配对

`len(a)`获取长度

### set 集合

`a = {1,2,3}`
内部的元素无序、不可重复

由于字典的使用频率比集合高很多，所以`a = {}`这样的创建默认是字典，需要的话使用`a = set()`创建空集合

### tuple 元组

`a = (1,2,3)` 或者省略`a = 1,2,3`

可以使用下标访问

**元组的解包**
`a,b = 1,2` 可以实现快速赋值（1,2被视作一个元组）
`a,b = b,a` 交换变量 （b,a临时创建一个元组然后再解包）
`a,b,_ = (1,2,3)` 最后一个值被抛弃，个数要匹配
`a,b,*c = (1,2,3,4)` c为一个列表，储存剩余的值

### list 列表

`a = [1,2,3,4,5,6]`
类似cpp的vector动态数组而非链表，但其中储存的实际上是指针

可以使用下标访问，另外python的下标支持负数

`+`被重载为添加元素，`*`是复制自己添加到结尾

**切片**
`b = a[start : end : step]`
左闭右开，从下标start处开始，每隔step选取一个元素，一直到end下标之前一个元素，得到切片后的新列表（拷贝）

**常用方法**
```py
a = [1,2,3]
a.append(4)          # [1,2,3,4] - 末尾添加（常用）
a.extend([5,6])      # [1,2,3,4,5,6] - 合并列表
a.insert(0, 0)       # [0,1,2,3,4,5,6] - 指定位置插入
a.pop()              # 弹出并返回末尾元素
a.pop(0)             # 弹出并返回指定位置元素
a.remove(3)          # 删除第一个匹配的值
```

**注意引用问题**：列表是可变对象，直接赋值只是引用拷贝
```py
b = a                # b和a指向同一个列表
b = a.copy()         # 拷贝得到独立的新列表
```

**列表推导式**
```py
a = [1,2,3,4]
b = [i**2 for i in a if i < 3]
#b = [1,4]
```

### dict 字典

`a = {'a':1, 'b':10}`
和cpp的`unordered_map`基本一致，哈希表

使用`del a['a']`删除元素
用`.values()` `.keys()` `.items()`得到值 键 键值对（打包成元组），用于循环遍历或者类型转换

**字典/集合推导式**（和列表推导式类似）
```py
a = [1,2,2,3,4]
d = {x: x**2 for x in a}     # {1:1, 2:4, 3:9, 4:16} - 字典推导
s = {x**2 for x in a}        # {1, 4, 9, 16} - 集合推导（自动去重）
```

### 高级类型的通用语法

**`in` 运算符**：检查成员是否存在，可用于list/dict/set/str
```py
3 in [1,2,3]          # True
'a' in {'a':1}        # True（检查键）
'e' not in "hello"        # False
```

**`any()` / `all()`**：检查可迭代对象中是否有任意/全部元素为True
```py
any([True, False, False])   # True
all([True, True, True])     # True
# 结合推导式使用很常见
any(x > 5 for x in [1,3,5]) # False
all(x > 0 for x in [1,3,5]) # True
```

### `is` vs `==`

`==`比较值，`is`比较内存地址（是否是同一个对象）

```py
a = [1,2,3]
b = [1,2,3]
a == b              # True（值一样）
a is b              # False（不同对象）

# 特别要注意 None 的判断：用 is
x = None
x is None           # 推荐写法
```

## for 遍历循环

**`range()`**：生成整数序列，常用于指定次数的循环
```py
for i in range(5):       # 0~4 共5次
for i in range(1, 6):    # 1~5 左闭右开
for i in range(0, 10, 2):# 0,2,4,6,8
```

**`enumerate()`**：同时获取下标和值
```py
a = ['a','b','c']
for idx, val in enumerate(a):
    print(idx, val)  # 0 a / 1 b / 2 c
```

**`zip()`**：并行遍历多个列表
```py
names = ['a','b','c']
scores = [90, 85, 95]
for n, s in zip(names, scores):
    print(f"{n}: {s}")
```

如果需要循环的是字典，用`.values()` `.keys()` `.items()`遍历值 键 键值对（打包成元组）
```py
a = {"a":1, "b":2, "c":3}

for tmp in a.items():
    print(tmp) #tmp是元组
for key,val in a.items():
    print(f"{key} = {val}")
```

## 函数

```py
def func(para):
   '''文档字符串：解释函数，使用func.__doc__获取文档字符串'''
   return None
   #不写return就默认返回None
```

可以返回多个值，会打包成一个元组再返回
可以输入任意数量的参数，写成`(*para)`即可，会将传入的任意数量参数打包成元组；这一项必须在函数参数列表的最后
可以传入任意数量的键值对参数：
```py
def func2(val1, val2, **kwargs):
    #调用 func2(1, 2, c=3, d=4) 时 kwargs = {'c':3, 'd':4}
    return kwargs
```

### 可变/不可变对象

- **不可变**：int float str tuple（函数内修改会创建新对象）
- **可变**：list dict set（函数内的修改在原对象上进行）

```py
# 经典坑：用可变对象作为默认参数
def bad_append(x, lst=[]):  # 默认列表只创建一次！
    lst.append(x)
    return lst
bad_append(1)  # [1]
bad_append(2)  # [1,2] ← 不是[2]！
# 正确做法
def good_append(x, lst=None):
    if lst is None:
        lst = []
    lst.append(x)
    return lst
```

## 类

类名默认是首字母大写的

### 成员

`__init__`类似于c/cpp的构造函数，但是要写出`self`作为第一个参数，这个类似于c/cpp的this指针
不同于c/cpp，python对大部分成员的声明是在`__init__`里完成的
```py
class Dog:
    def __init__(self, name, age):
        #实例变量（每个对象独有）
        self.name = name
        self.age = age
        self.speed = 10 
        
        #私有变量（名称修饰）
        self._secret = "secret"
    
    #类变量（静态成员）
    species = "Canis familiaris"
```

python没有访问限制相关的关键字，而是用命名`_val`约定为私有，但是不限制访问

### 继承

```py
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"
    def move(self):
        return f"{self.name} moves"

class Dog(Animal):  #继承
    def __init__(self, name, breed):
        super().__init__(name)  #调用父类构造函数
        self.breed = breed
    
    def speak(self):  #自动覆盖
        return "Woof!"
    def fetch(self):
        return f"{self.name} fetches"
```

## 导入模块

```py
import numpy               # 导入整个模块，用 numpy.array()
import numpy as np         # 起别名（ML/DL中最常见），用np.array()
from numpy import array    # 只导入特定函数，直接 array()
```

## with 文件读写

```py
# 自动管理资源释放，无需手动close
with open("data.txt", "r") as f:   # r读 w写 a追加
    content = f.read()             # 全部读取
    lines = f.readlines()          # 按行读取为列表

# ML中常用with读数据
with open("train.txt", "r", encoding="utf-8") as f:
    for line in f:
        process(line.strip())
```

## main

让一个 `.py` 文件既能被导入（复用函数/类），又能直接运行测试代码：

```py
# my_utils.py
def add(a, b):
    return a + b

if __name__ == '__main__':
    # 只有直接运行 python my_utils.py 时才会执行
    print(add(3, 5))  # 测试代码
```

## 实践相关
传入参数，如果是自定义类型，自行声明参数才有自动补全：
```py
class Solution:
    def __init__(self,N,K):
        self.N = N
        self.K = K
        self.start_state = (N, N, 0)

    def getStartState(self):
        return self.start_state

    def isGoalState(self, state):
        return state == (0,0,1)

    def getSuccessors(self, state):
        return get_successors(state, self.N, self.K)
    
def DFS(problem: Solution):
    #这样子就打problem就可以按Tab补全
```