---
slug: 26e83edc
author: "昊色居士"
title: "打卡软件破解（〇）前言"
description: 
date: 2022-07-29T23:50:52+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "逆向", "破解"
]
categories: [
    "逆向"
]
series:  [
    "打卡软件破解"
]
---

# 打卡软件破解（〇）前言

## 为啥要破
有个朋友，他们公司是使用的自研APP来打卡，所以想要以`纯技术`的角度来看看能不能破解，他和我说`绝不是因为想要作弊`，我也信了，所以就研究了一下，也就有了这个系列的教程。

>君子声明: 本系列教程是以纯技术的角度来探讨研究，严禁使用本系列教程中提到的技术去破解其他本类型APP，损坏他人及公司利益。

## 想想咋破
那个朋友啊，他说这个软件主要是使用`wifi热点`、`定位`两种方式来分别进行打卡的校验。所以下面就可以针对这两种方式在配合软件和硬件这两个方向去操作。

## 有啥方法

### 软件-客户端方向

软件方向针对软件本身就做一些操作和修改，比如校验了`wifi热点`和`定位`，同时服务器肯定无法知道客户端本身的信息，需要接口去传递信息，这样就可以针对这两种去修改软件包本身，使之调用接口使获取的信息就是服务器想要的，而不是真实的
* 此方案优点：客户端不强制更新，获取更改相关逻辑就能一直用
* 从方案缺点：每次更新都需要重新反编译破解，如果软件进行了加壳及代码混淆会增加难度

### 软件-服务端方向

上面提到了，既然打卡肯定是通过调用服务端接口，那么反正都要调用接口，为啥不直接调用呢？只要知道了打卡的接口等信息，就可以绕过客户端直接调用模拟请求来打卡，甚至可以定时全自动打卡。
* 此方案优点：没有客户端更新限制，随时随地打卡
* 此方案缺点：
  * 需要对接口进行模拟测试。增加暴露风险
  * 服务器代码可以随时更新。增加暴露风险
  * 如接口参数进行了加密，还需要进行反编译软件。增加技术难度

### 软件--第三方软件

同样，既然是定位，那么使用第三方软件把手机的定位，改成规定的位置不就好啦。既然是wifi热点，也同样使用第三方的软件把手机wifi热点的`mac地址`改掉就好了。
* 此方案优点：技术难度小，可随时更新客户端，隐蔽性强
* 此方案缺点：依赖于第三方软件，随时会有失效的风险，同时此类软件属于对系统层面上的修改，通常需要安卓系统的`Root`和IOS系统的`越狱`

### 硬件--模拟WiFi热点
wifi热点打卡原理是获取当前连接wifi的mac地址在与目标mac地址进行对比。所以这里就可以通过修改无线网卡的mac地址和wif名称的方法来与目标wifi达成一致，这里的无线网卡可以使用：

* 自行购买的无线网卡
* 笔记本电脑的无线网卡
* 台式机的外接wifi网卡
* 两部手机。其中一部手机进行 ROOT或 越狱后修改自身的mac地址

优缺点同样很明显，就是硬件上不方便携带的问题

### 远控的方式
也可以采用使用PC设置或手机设置来远控在目标位置的另一部手机来进行打卡的方法来解决。

## 总结一下
方法多种多样，重要的是思路。技术上来讲就是一个攻和受，啊不，是攻与防的过程。没有不透风的墙，同样也没有破不了的软件。只要锄头挥的好，就算墙挖不倒，也会刨个洞。祝大家学有所成！