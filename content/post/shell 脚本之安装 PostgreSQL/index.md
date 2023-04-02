---
slug: 1a248b32
author: "昊色居士"
title: "Shell 脚本之安装 PostgreSQL"
description: 
date: 2022-10-01T17:29:20+08:00
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

# shell 脚本之安装 PostgreSQL

我们在开发过程中经常或会遇到在本地开发的时候使用数据库的场景，而且有时还要允许其他同事访问，今天给大家带来一个安装 pgsql 数据库的脚本

## 安装 PostgreSQL

### 添加源

由于我们准备使用`apt`去安装`PostgreSQL`，所以要先把软件源添加进去：

```shell
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main 13
" > /etc/apt/sources.list.d/pgdg.list'
```

大家看上面的这条命令，其实就是把字符串`deb http://apt.postgresql.org/pub/repos/apt buster-pgdg main`添加到文件`/etc/apt/sources.list.d/pgdg.list`中而已。但是为啥要这么写呢？下面让我们仔细的看看：

- `>` 符号。`>`符号的作用的把前面的结果覆盖到后面，如果文件不存在则会创建，与之类似的还有`>>`，不过这个是追加而已
- `echo`命令。`echo`命令是输出一个字符串
- `sh -c`。`sh -c ''`作用是执行字符串里面的命令
- `sudo`。作用是给后面是命令添加 root 权限

所以这个命令其实可以分两部分，一是：`echo "deb http://xxxx" > /etc/apt/sources.list.d/pgdg.list` ，目的是创建文件，二是：`sudo su -c ''`，之所以要加这个是因为我们要写入的文件一般用户的没有权限的，并且大家如果经过测试就会发现使用命令：`sudo echo "deb http://xxxx" > /etc/apt/sources.list.d/pgdg.list`是没有效果的，因为命令如果这个写的其中的 sudo 是给前面的 echo 添加权限，所以我们就把这条命令作为字符串使用`sh -c`的形式去执行，并且给`sh -c`添加 sudo，这样就可以了

### 添加签名

```shell
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add
```

### 使用 apt 安装

```shell
sudo apt update
sudo apt install postgresql-13 -y
```

这里使用了`-y`参数，其作用是不用再安装的过程中按[y/N]，直接静默安装

## 配置

### 删除`postgres`用户的默认密码

再安装过程中会自动创建用户`postgres`，也是后续执行`postgres`相关操作的建议用户，由于我们是开发环境自己使用的，所以为了方便，这里直接把密码删除：

```
sudo passwd -d postgres
```

如果需要设置密码的话，再重新设置密码就好

### 为默认用户设置默认密码

再安装`PostgreSQL`的时候，不仅会创建系统用户`postgres`，也会创建一个数据库用户`postgres`，但不会自己设置密码，我们这里给它添加一个默认密码：

```shell
su - -c "psql -c $'ALTER USER postgres WITH PASSWORD \'123456\';'" postgres
```

这里是使用了`su - -c '' postgres`的方式去执行的命令，这样写的目的是使用`postgres`用户去执行字符串里面的脚本，而字符串里面的脚本是`psql -c $''`，其作用是事业 pqsl 执行一段 sql 命令，这里是 sql 命令就是修改用户密码的命令：`ALTER USER postgres WITH PASSWORD '123456';`然后把这些组合到一起，并且为字符串添加转义

### 开启允许其他人访问

需要分别设置以下几个文件

客户端认证文件：/etc/postgresql/13/main/pg_hba.conf

其中有个字段是`address`，表示可以匹配的地址，默认是`127.0.0.1`，我们要把它修改为`0.0.0.0`:

```shell
sudo sed -i 's@127.0.0.1/32@0.0.0.0/0@g' /etc/postgresql/13/main/pg_hba.conf
```

这里使用了`sed -i`的替换命令，这里的`@`是分隔符，g 是全局的意思，就是全局替换

配置文件： /etc/postgresql/13/main/postgresql.conf

还需要再配置文件中设置一下监听的地址，所在行再 13 版本中为 60 行，内容为：

```
# listen_addresses = 'localhost'          # what IP address(es) to listen on;
```

默认是`localhost`，我们要把他改为`*`：

```shell
sudo sed -i '60s/^.//' /etc/postgresql/13/main/postgresql.conf
sudo sed -i '60s/localhost/*/' /etc/postgresql/13/main/postgresql.conf
```

这里用了两条`sed `语句去处理，第一条是删除第 60 行的开始两个字符，就是去掉注释，第二条则是把第 60 行的`localhost`替换成`*`，就是任何地址的意思

## 启动

这里启动用两条命令：

```shell
sudo pg_ctlcluster 13 main start
sudo /etc/init.d/postgresql restart postgresql
```

看到第二天命令里面的`restart`了吗？应该会猜到一系列的命令了吧？停止的`stop`，启动的`start`，重启的`start`

## 卸载

先卸载现有版本然后重新安装用的：

```shell
sudo apt-get --purge remove postgresql\*
sudo rm -rf /etc/postgresql* /var/lib/postgresql/
```

## 封装脚本

```shell
#!/bin/sh

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main 13
" > /etc/apt/sources.list.d/pgdg.list'
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
sudo apt install postgresql-13 -y

sudo passwd -d postgres
su - -c "psql -c $'ALTER USER postgres WITH PASSWORD \'123456\';'" postgres
sudo sed -i 's@127.0.0.1/32@0.0.0.0/0@g' /etc/postgresql/13/main/pg_hba.conf
sudo sed -i '60s/^.//' /etc/postgresql/13/main/postgresql.conf
sudo sed -i '60s/localhost/*/' /etc/postgresql/13/main/postgresql.conf

sudo pg_ctlcluster 13 main start
sudo /etc/init.d/postgresql restart postgresql

echo "\n\nPGSQL13 安装完毕!!! \n默认用户名：postgresql \n默认密码：123456 \n配置文件位置：/etc/postgresql/13/main/postgresql.conf"
```

## 结语

脚本本身不是很复杂，主要涉及到了以下的一些知识点。但是这个脚本还不是很完善，缺少容错机制，比如 pgsql 使用 apt 安装失败的问题？最后希望大家有所收获。

- sudo sh -c '' 执行命令
- psql -c '' 执行 sql
- su - -c '' user 其他用户执行命令
- sed -i 替换
- pgsql 的配置文件和客户端访问配置文件
