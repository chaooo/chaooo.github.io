---
title: 「Shell」Shell 编程入门
date: 2020-06-06 15:12:46
tags: [环境配置, Linux, Shell]
categories: 环境配置
---


### 1. Shell概述
1. Shell简介
    * shell是操作系统提供给我们用户来访问系统资源的一个**接口**。
    * shell同时还是一个Linux下的**命令行解释器**，类似Windows下的cmd。
    * shell 同时还是解释型的脚本语言：运行时翻译，执行一条语句翻译一条，每次执行程序都需要进行解释。<!-- more -->
2. Shell的发展
    * shell有多个版本：Bourne Shell，C Shell，Korn Shell，Bash Shell。现在广泛使用的是 Bash Shell，也就是Linux中默认内嵌的Shell。
3. Shell脚本
    * `Shell脚本`(shell script)，是一种为Shell编写的脚本程序。

### 2. Shell变量基础
#### 2.1 Shell变量的分类
1. 用户自定义变量：由用户自己定义，修改和使用的变量
2. Shell环境变量：用于**设置shell的运行环境**，只要少数的环境变量可以修改其值。环境变量也是可以自定义的。
3. 位置参数变量：通过命令行给脚本传递参数，**变量名已经固定**，不能自定义。
4. 内部参数变量：是**bash中已经定义好的变量**，变量名不能自定义，变量作用也是固定的。

#### 2.2 变量赋值与设置
``` shell
# 赋值有两种格式
var1=value
var2=`command`      # 注意：command是可以执行的命令
# 清除变量 与 设置只读
unset var1          # 清除var1变量
readonly var2       # 设置var2变量为只读
# 变量的引用有两种方式（使用时加美元符号）
$var1               # $变量名
${var1}             # ${变量名}
```

#### 2.3 位置参数变量
位置参数变量是一种特殊的shell变量，用于从命令行向脚本中传递参数。
$0表示脚本的名称，$1表示第一个参数，$2表示第二个参数，依次下去代表第几个参数，
但是从第十个参数位开始表示方法有所改变，需要加大括号，例如：${10}，${11}

``` shell
[chao@localhost ~]$ ls anaconda-ks.cfg install.log install.log.syslog
```

> `$0`的值就是`ls`命令本身，`$1`的值就是`anaconda-ks.cfg`这个文件，`$2`是`install.log`文件，`$3`是`install.log.syslog`文件

|  位置参数变量  |  作 用  |
|  ----  | ----  |
|$n	        |n 为数字，$0 代表命令本身，$1〜$9 代表第 1〜9 个参数，10 以上的参数需要用大括号包含， 如${10}	|
|$*			|这个变量代表命令行中所有的参数，把所有的参数看成一个整体										|
|$@			|这个变量也代表命令行中所有的参数，不过 $@ 把每个参数区别对待									|
|$#			|这个变量代表命令行中所有参数的个数															|

位置参数变量要用于向命令或程序脚本中传递信息，比如：
``` shell
[chao@localhost ~]$ vi count.sh
#!/bin/bash
num1=$1
num2=$2
sum=$(($num1 + $num2))
echo $sum

[chao@localhost ~]$ ./count.sh 11 22
33
```


#### 2.4 Shell环境变量
* Shell 变量的作用域可以分为三种：
    + 有的变量只能在函数内部使用，这叫做局部变量（local variable）；
    + 有的变量可以在当前 Shell 进程中使用，这叫做全局变量（global variable），在 Shell 中定义的变量，默认就是全局变量；
    + 而有的变量还可以在子进程中使用，这叫做环境变量（environment variable）。
全局变量只在当前 Shell 进程中有效，对其它 Shell 进程和子进程都无效。如果使用export命令将全局变量导出，那么它就在所有的子进程中也有效了，这称为“环境变量”。


#### 2.5 内置参数变量
内部参数分为两类：命令行参数，与进程相关的内部参数
1. 命令行参数
``` shell
$@   # 表示传递给脚本或函数的所有参数。被双引号引用时，与$*有所不同。
$*   # 表示传递给脚本或函数的所有参数
$0   # 表示命令行输入的脚本名称
$#   # 表示命令行上的参数个数
```
>  注意： $@和$*不加引号时,二者都是返回传入的参数,但加了引号后$*把参数作为一个字符串整体(单字符串)返回, $@把每个参数作为一个一个字符串返回


2. 与进程相关的内部参数
``` shell
$?   # 表示上一个命令执行的返回结果
$$   # 表示当前程序运行的 PID 
$!   # 表示获取上一个在后台工作进程的PID
$_   # 表示获取在此之前执行命令或脚本的最后一个参数
```


### 3. 退出状态码
`exit`是一个Shell内置命令，用来退出当前Shell进程，并返回一个退出状态；使用$?可以接收这个退出状态。
`exit`命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。
`exit`退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败，可以根据退出状态来判断具体出现了什么错误。

``` shell
[chao@localhost ~]$ vi test.sh
#!/bin/bash
echo "befor exit"
exit 8
echo "after exit"

[chao@localhost ~]$ bash ./test.sh
befor exit
[chao@localhost ~]$ echo $?
8
```