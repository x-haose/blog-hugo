---
slug: 59928d06
author: "昊色居士"
title: " Python处理Float精度问题方案"
description: 
date: 2022-09-30T17:36:56+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "Python", "Float精度"
]
categories: [
    "Python"
]
series:  [

]
---

# Python处理Float精度问题方案

## 前言

在开发时，大家可能或多或少的在各个语言中遇到过`float`类型精度的问题，看着加减是没问题的，但是结果却在小数点后面出现了一堆，像下面这样

![](https://images.haose.pro/2022/10/11/1665470401366_14%3A40%3A01_nzhumlz2aq.png)

## 原因

简单来说就是计算机在处理浮点型的时候无法准确的计算。

展开点说的话就是因为计算机在计算数字的时候是以二进制的形式进行的计算，而计算机是没办法把所以小数都转化为二进制的数字，所以在进行加减操作的时候出现了无法用二进制表示的小数，就会出错。

更加详细的原因，如果有兴趣请自行查阅资料

## 解决方案

这里会`python`为例写解决方案，当然其他语言也是类似的，稍加改动即可

### 转为整形

既然小数的加减计算容易出错，乘除不容易出错，就可以把小数经过乘来转化为整数后在除一下，如下所示：

```python
>>> (0.1*100 + 0.2*100)/100
0.3
>>> 
```

这个操作也是存在弊端的，就是麻烦，很多涉及计算时都要这样去转一下，而且如果是中途发现存在这种问题维护起来是很不方便的，容易存在遗漏

### 四舍五入

也是可以直接采用四舍五入的方法进行操作，在大部分的情况是适用的，如下：

```python
>>> round(0.1+0.2, 2)
0.3
```

### 高精度计算

在大部分语言中都存在高精度计算类型：`decimal`，就是专门处理需要高精度计算的情况，下面来简单的介绍一下：

```py
>>> from decimal import Decimal
>>> Decimal(0.1) + Decimal(0.2)
Decimal('0.3000000000000000166533453694')
>>> 
```

再也不对啊，咋还是这个结果呢，上面的错误的打开方式，重新来：

```python
>>> from decimal import Decimal
>>> Decimal("0.1") + Decimal("0.2")
Decimal('0.3')
>>> 
```

哎，这就对了嘛，是要把float类型专为字符串在传到Decimal中在计算就好了，当然返回的结果依旧是`Decimal`类型的，所以就要在转成float类型：

```python
>>> from decimal import Decimal
>>> f_1 = 0.1
>>> f_2 = 0.2
>>> d = Decimal(str(f_1)) + Decimal(str(f_2))
>>> d.to_eng_string()
'0.3'
>>> f_3 = float(d.to_eng_string())
>>> f_3
0.3
>>> 

```

由于`Decimal`类型自身的没有提供向`float`类型转换的方法，所以就要先转成字符串类型，在转成float类型，而`to_eng_string()`就是把`Decimal`专为字符串的方法

### 封装简化

如果数值从计算到存储一直用都都是decimal，那当然很好，但如果还是中途转换的话，就有点麻烦了，所以我们封装一下。

在写封装之前，要知道我们想要达成什么效果，我们要的是现有代码修改最小的形式来进行兼容，现有代码最常见的操作有：

* 0.1+0.2
* 0.3-0.1
* sum([0.1, 0.1, 0.1])

上面的加、减、累加是代码中出现最频繁的浮点数操作，我们就是实现这三种，前两个加和减我们使用Python里面的`__add__`和`__sub__`，实现这两个方法的类，就可以对类就是加和减，和其他语言里面的运算符重载很像

对于累加方法`sum()`不存在固定的主体，而且存在一个参数，目标参数是float的序列，我们可以以类的静态方法的方式去实现：`@staticmethod`

详细代码如下：

```python
from decimal import Decimal
from typing import Iterator


class DecimalFloat(float):
    """
    使用Decimal计算Float类型
    """

    def __sub__(self, other):
        """
        相减
        Args:
            other: 另一个数

        Returns:
            返回float类型的结果
        """
        decimal_self = Decimal(str(self))
        decimal_other = Decimal(str(other))
        decimal_result = decimal_self - decimal_other
        return float(decimal_result.to_eng_string())

    def __add__(self, other):
        """
        相加
        Args:
            other: 另一个数

        Returns:
            返回float类型的结果
        """
        decimal_self = Decimal(str(self))
        decimal_other = Decimal(str(other))
        decimal_result = decimal_self + decimal_other
        return float(decimal_result.to_eng_string())

    def __mul__(self, other):
        """
        相乘
        Args:
            other: 另一个数

        Returns:
            返回float类型的结果
        """
        decimal_self = Decimal(str(self))
        decimal_other = Decimal(str(other))
        decimal_result = decimal_self * decimal_other
        return float(decimal_result.to_eng_string())

    @staticmethod
    def sum(number_list: Iterator[float]):
        """
        对一组float数字进行累加操作
        Args:
            number_list: 一组Float类型的数字

        Returns:
            返回float类型的结果
        """
        decimal_number_list = [Decimal(str(number)) for number in number_list]
        return float(sum(decimal_number_list).to_eng_string()) if decimal_number_list else 0
```

对于加和减，代码是很简单的，就是和上面操作的那样，把float转换为str，在把str转换为Decimal计算

对于sum方法则是，把float的序列转为Decimal的序列，然后在进行sum运算后，在把结果转为float类型

使用方法：

```python
>>> DecimalFloat(0.1) + DecimalFloat(0.2)
0.3
>>> DecimalFloat(0.3) - DecimalFloat(0.1)
0.2
>>> l = [0.1, 0.1, 0.1]
>>> DecimalFloat.sum(l)
0.3
>>> 
```

## 总结

介绍了在Python中对于小数的精度问题的几种解决方案，虽然是以Python语言写的，但其他语言也是一样的，根据情况选择对应的方案。