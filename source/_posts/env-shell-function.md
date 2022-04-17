---
title: 「Shell」Shell 函数的用法
date: 2020-06-08 15:12:46
tags: [环境配置, Linux, Shell]
categories: 环境配置
---


### 1. Shell 函数定义
``` shell
function name() {
    # 函数要执行的代码
    [return value]
}
```

函数的简化写法可以省略`function`或`()`之一，不过还是推荐标准写法，才能做到做到“见名知意”。<!-- more -->


### 2. Shell函数调用
调用 Shell 函数时可以给它传递参数，也可以不传递。如果不传递参数，直接给出函数名字即可：
``` shell
name
```

如果传递参数，那么多个参数之间以空格分隔：
``` shell
name param1 param2 param3
```

* 不管是哪种形式，函数名字后面都不需要带括号。
* 和其它编程语言不同的是，Shell 函数在定义时不能指明参数，但是在调用时却可以传递参数，并且给它传递什么参数它就接收什么参数。
* Shell 也不限制定义和调用的顺序，你可以将定义放在调用的前面，也可以反过来，将定义放在调用的后面。

定义一个函数，计算所有参数的和：
``` shell
[cmuser@localhost ~]$ #!/bin/bash
[cmuser@localhost ~]$ function getsum(){
>     local sum=0
>     for n in $@
>     do
>         ((sum+=n))
>     done
>     return $sum
> }
[cmuser@localhost ~]$ getsum 10 20 55 15
[cmuser@localhost ~]$ echo $?
100
```


### 3. Shell函数参数
函数参数是 Shell 位置参数的一种，在函数内部可以使用`$n`来接收，例如，`$1`表示第一个参数，`$2`表示第二个参数，依次类推。

除了`$n`，还有另外三个比较重要的变量：
* `$#`可以获取传递的参数的个数；
* `$@`或者`$*`可以一次性获取所有的参数。

使用 $n 来接收函数参数:
``` shell
[cmuser@localhost ~]$ #!/bin/bash
# 定义函数
[cmuser@localhost ~]$ function show(){
>     echo "Tutorial: $1"
>     echo "URL: $2"
>     echo "Author: "$3
>     echo "Total $# parameters"
> }
# 调用函数
[cmuser@localhost ~]$ show test www.test.com Tom
Tutorial: test
URL: www.test.com
Author: Tom
Total 3 parameters
```

使用 $@ 来遍历函数参数，计算所有参数的和：
``` shell
[cmuser@localhost ~]$ #!/bin/bash
# 定义函数
[cmuser@localhost ~]$ function getsum(){
>     local sum=0
>     for n in $@
>     do
>         ((sum+=n))
>     done
>     echo $sum
>     return 0
> }
# 调用函数
[cmuser@localhost ~]$ echo $(getsum 10 20 55 15)
100
```


### 4. Shell函数返回值
Shell中的`return`返回值表示的是函数的退出状态：返回值为` 0 `表示函数执行成功了，返回值为`非 0 `表示函数执行失败（出错）了。if、while、for 等语句都是根据函数的退出状态来判断条件是否成立。

Shell 函数的返回值只能是一个介于` 0~255 `之间的整数，其中只有` 0 `表示成功，其它值都表示失败。

如果函数体中没有return语句，那么使用默认的退出状态，也就是最后一条命令的退出状态。更加严谨的写法为：`return $?`。
$?是一个特殊变量，用来获取上一个命令的退出状态，或者上一个函数的返回值。

* 如何得到函数的处理结果？
    1. 借助全局变量，将得到的结果赋值给全局变量；
    3. 在函数内部使用 `echo`、`printf` 命令将结果输出，在函数外部使用`$()`或者`` ` ``捕获结果。

具体来定义一个函数 getsum，计算从 m 加到 n 的和，并使用以上两种解决方案。
1. 【实例1】将函数处理结果赋值给一个全局变量。

``` shell
[cmuser@localhost ~]$ #!/bin/bash
[cmuser@localhost ~]$ sum=0                # 全局变量
[cmuser@localhost ~]$ function getsum(){
>     for((i=$1; i<=$2; i++)); do
>         ((sum+=i))
>     done
>     return $?                            # 返回上一条命令的退出状态
> }
[cmuser@localhost ~]$ getsum 1 100
[cmuser@localhost ~]$ echo "The sum is $sum" # 输出全局变量
The sum is 5050
```

2. 【实例2】在函数内部使用 echo 输出结果。
``` shell
[cmuser@localhost ~]$ function getsum(){
>     local sum=0                         # 局部变量
>     for((i=$1; i<=$2; i++)); do
>         ((sum+=i))
>     done
>     echo $sum
>     return $?
> }
[cmuser@localhost ~]$ echo "The sum is "$(getsum 1 100)
The sum is 5050
```
代码中总共执行了两次 `echo` 命令，但是却只输出一次，这是因为`$()`捕获了第一个 `echo` 的输出结果，它并没有真正输出到终端上。除了`$()`，你也可以使用`` ` ``来捕获 `echo` 的输出结果。


### 5. Shell函数库的使用
shell函数库实质为一个脚本，脚本内包含了多个函数（函数具有普遍适用性），经常使用的重复代码封装成库函数文件。

库函数一般不直接执行，而是由其他脚本调用，库函数文件名的后缀是任意的，但一般使用`.lib`，库文件通常没有可执行权限。

第一行一般使用#!/bin/echo，输出警告信息，避免用户执行。

``` shell
[cmuser@localhost test]$ vim base.lib 
#!/usr/bin/echo

add(){
    echo "$(expr $1 + $2)"
}
reduce(){
    echo "$(expr $1 - $2)"
}
multiple(){
    echo "$(expr $1 \* $2)"
}
divide(){
    echo "$(expr $1 / $2)"
}
```

``` shell
[cmuser@localhost test]$ vim calculate.sh
#!/usr/bin/bash

# 加载函数库文件
source ./base

# 调用函数，传入参数
add $1 $2
reduce $1 $2
multiple $1 $2
divide $1 $2
```

``` shell
[cmuser@localhost test]$ sh calculate.sh 40 5
45
35
200
8
```
