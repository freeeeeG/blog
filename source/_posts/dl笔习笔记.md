---
title: dl笔习笔记
date: 2021-12-15 17:47:22
tags: 学习
---


# pytourch的基础使用
## 万物皆数之张量
张量的概念比较复杂，高度抽象了我们平时里的常用单位。
简单的说。
0阶的张量就是标量，一个变量足够描述，像温度。
1阶的张量就是矢，两个变量（空间中两个点）足够描述，像方向。
2阶的张量就是，3个变量（两个点和力的大小）足够描述，像力。
所以以此类推假如我们要识别一张600*800的图片，那么我们的张量阶数就是420000-1阶。
这非常方便我们把万物抽象成数字
他在py中也表示的很简单

```python
x = torch.rand(5, 5, requires_grad=True)
```
这里的参数比较简单，前面是定义这个张量的大小，这里是5*5，第三个变量的TRUE
表示其能否求导（前向传播）
这个变量写的还是很有意思的。而且有些许bug。
就是但其为True时，我们不能对其进行一些“常规操作”

```python
#  requires_grad=False时
x[2][2] = 1
#  requires_grad=True时
上述操作不可行
```
那我们现在对其试着求导
```python
x = torch.rand(5, 5, requires_grad=True)
y = torch.rand(5, 5, requires_grad=True)
z = x*2 + y*3
z.backward(torch.ones_like(x))
```
其值为
tensor(\[[2., 2., 2., 2., 2.],
        [2., 2., 2., 2., 2.],
        [2., 2., 2., 2., 2.],
        [2., 2., 2., 2., 2.],
        [2., 2., 2., 2., 2.]])
虽然确实是对每个值求了偏导(总的就是梯度)，但是其实我不解为什么这个地方是2，其实应该是1。
我试过z = x输出其导数。发现他们的偏导也全为1。我并不知道这是为什么，理论上应该是0啊QAQ。

### 各种乘法
```python

x = torch.rand(5, 5, requires_grad=True)
y = torch.rand(5, 5, requires_grad=True)

```
![1](dl1.png)
定义x，y两个变量。
接下来我们比较不同的乘法
```python

z = x.mm(y)
z.backward(torch.ones_like(x))
print(x.grad)
print(z)
z = x.mul(y)
z.backward(torch.ones_like(x))
print(x.grad)
print(z)
z = x*y
z.backward(torch.ones_like(x))
print(x.grad)
print(z)

```
叉乘的偏导和z的值
![2](dl2.png)


mm是叉乘，mul和*是点乘

## 线性回归 （Linear Regreesion）


```python
# 引用
# 注意，这里我们使用了一个新库叫 seaborn 如果报错找不到包的话请使用pip install seaborn 来进行安装
import torch
from torch.nn import Linear, Module, MSELoss
from torch.optim import SGD
# SGD（随机最速下降）这个玩意好难看啊QAQ
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
```