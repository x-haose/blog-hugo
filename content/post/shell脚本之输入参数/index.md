---
slug: 9f81cb07
author: "昊色居士"
title: "Shell脚本之输入参数"
description: 
date: 2022-10-02T23:30:43+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "shell", "Linux", "脚本"
]
categories: [
    "Linux", "shell"
]
series:  [

]
---

# shell脚本之输入参数

## 前言

我们再使用c、c++、python或其他语言编写脚本的时候，都会提供很多的参数供大家选择，其中包括位置参数和关键字参数，其实我们在编写shell脚本的时候也是有这种操作的，今天我们就来简单的介绍一下。

## 位置参数

在shell 脚本中获取位置变量是通过: `$1`、 `$2`、 `$3`、 `$4`的方式依次获取的，其中`$0`则是脚本本身的名字，如下示例脚本：

```shell
echo $1, $2
echo $0
```

我们直接在终端去执行它，结果为：

```shell
 sh test.sh 1 2
 
 1, 2
test.sh
```

还有一个叫做`$@`的，这个可以把所有的参数取出来，如下：

```shell
# test.sh:
echo $1, $2
echo $0
echo $@

# 执行
root➜~» sh test.sh 1 22 3
1, 22
test.sh
1 22 3
```

## 关键字参数

### shift 关键字

再说关键字之前，先介绍一个`shift`，我们上面再介绍位置参数的时候，就是1、2、3依次获取的，而`shift`的作用就是把参数的索引向左移动，举例如下：

```shell
# 执行脚本
sh test.sh 1 2 3 4

# 输出内容
1
2
3
4
```

看着上面的这个输出，大家肯定会以为脚本内容是类似这样的：

```shell
echo $1
echo $2
echo $3
echo $4
```

其实不然，这里如果使用了`shift`的话，脚本大可以这么写：

```shell
echo $1
shift 1

echo $1
shift 1

echo $1
shift 1

echo $1
```

每输出一次，则把参数向做移动一次，这样就可以一直使用`$1`来获取后面的参数，上面的例子是每次移动一位，我们也可以移动两位，示例脚本如下：

```shell
a=$1
b=$2

shift 2

echo $1

shift 1
echo $1

shift 1
echo $1

shift 1
echo $1

echo $a, $b
```

允许结果：

```shell
# 命令
sh test.sh aaa bbb 1 2 3 4
 
# 结果
1
2
3
4
aaa, bbb
```

我们先把前两个参数给提取了出来，然后把参数向做移动了两位之后，每获取一次第一个参数就向左移动一次

### 循环

我们上面的通过手动调用了四次`shift`和四次`echo`来达成的操作，那么如果是40个呢？所以这里可以使用循环，在shell脚本中也是有`while`循环的，大概是这样：

```shell
while [ 循环条件 ]; do
	# 循环体
done
```

我们来把上面的一段脚本改为循环来试试：

```shell
a=$1
b=$2

shift 2

while [ "$1" != "" ];do
        echo $1
        shift
done

echo $a, $b
```

运行结果：

```shell
root➜~» sh test.sh aaa bbb 1 2 3 4        

# 结果
1
2
3
4
aaa, bbb
```

> 要注意的是：while循环中两个[]里面一定要有空格，而且还要是左右各一个空格！！！

这样我们就把上面的脚本改成使用while循环的方式了

### case

这个case关键字就和我们c系、js等语言常见的case是一样的，就是条件语句，用法大概是这样的：

```shell
a=$1
b=$2

shift 2

while [ "$1" != "" ];do
    case $1 in
        1)
            echo "等于1"
            ;;
        2)
            echo "等于2"
            ;;
        *)
            echo "等于其他的：$1"
            ;;
        esac
        shift
done

echo $a, $b
```

输出如下：

```shell
root➜~» sh test.sh aaa bbb 1 2 3 1000                                                       

# 运行结果
等于1
等于2
等于其他的：3
等于其他的：1000
aaa, bbb
```

达到了我们根据参数的值来运行不同的代码的效果，效果是和一般编程语言中差不多的

### 获取关键字参数

上面的基础知识了解了之后，只要组合一下就能达到获取关键字参数的效果了，示例：

```shell
a=$1
b=$2

shift 2

while [ "$1" != "" ];do
    case $1 in
        -h | --host )
            shift
            echo "host: $1"
            ;;
        -p | --port )
            shift
            echo "port: $1"
            ;;
        *)
            echo "其他参数：$1"
            ;;
        esac
        shift
done

echo $a, $b
```

运行效果：

```shell
root➜~» sh test.sh aaa bbb --host 127.0.0.1 -p 2000 3 1000                                                                                                                       
# 运行结果
host: 127.0.0.1
port: 2000
其他参数：3
其他参数：1000
aaa, bbb
```

### 多个参数

有时会出现这样的参数：`-f 1.txt -f 2.txt -f s.mp4`在参数里面指定多个文件的情况，我们也来模拟下这个情况：

```shell
#!/bin/bash

a=$1
b=$2
files=()

shift 2

while [ "$1" != "" ];do
    case $1 in
        -h | --host )
            shift
            echo "host: $1"
            ;;
        -p | --port )
            shift
            echo "port: $1"
            ;;
        -f | --file )
            shift
            echo "文件路径: $1"
            files+=("$1")
            ;;
        *)
            shift
            echo "未知参数：$1"
            ;;
        esac
        shift
done

echo $a, $b
echo ${files[@]}
```

运行结果如下：

```shell
root➜~» bash test.sh aaa bbb --host 127.0.0.1 -p 2000 -f 1.txt -f 2.avi -f 3.jpg                                                                                          
# 运行结果
host: 127.0.0.1
port: 2000
文件路径: 1.txt
文件路径: 2.avi
文件路径: 3.jpg
aaa, bbb
1.txt 2.avi 3.jpg
```

大家可能会发现我们运行的方式由`sh`改为了`bash`，因为`bash`的功能更强大一点，才支持了数组的这种数据结构

## 结语

好了，到目前为止，我们已经在shell脚本中实现了传参的大部分情况，其他shell的学习还是很方便的，因为有很多安装的脚本就是shell的，好奇的话直接点进去看看，然后观察是怎么实现的就好，希望大家能有所收获！！！