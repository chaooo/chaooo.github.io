---
title: 「Shell」Shell 变量的用法与数学运算
date: 2020-06-07 15:12:46
tags: [环境配置, Linux, Shell]
categories: 环境配置
---


### 1. 变量替换

|  语法  |  说明  |
|  ----  | ----  |
|`${变量#匹配规则}`          |从变量**开头**进行规则匹配，将符合**最短**的数据删除|
|`${变量##匹配规则}`         |从变量**开头**进行规则匹配，将符合**最长**的数据删除【贪婪模式】|
|`${变量%匹配规则}`          |从变量**尾部**进行规则匹配，将符合**最短**的数据删除|
|`${变量%%匹配规则}`         |从变量**尾部**进行规则匹配，将符合**最长**的数据删除【贪婪模式】|
|`${变量/旧字符串/新字符串}`  |变量内容符合旧字符串，则**第一个**旧字符串会被新字符串取代|
|`${变量//旧字符串/新字符串}` |变量内容符合旧字符串，则**全部的**旧字符串会被新字符串取代|

<!-- more -->

``` shell
[chao@localhost ~]$ var1="I love you, Do you love me"
[chao@localhost ~]$ echo $var1
I love you, Do you love me
# 头部匹配删除
[chao@localhost ~]$ var2=${var1#*ov}
[chao@localhost ~]$ echo $var2
e you, Do you love me
[chao@localhost ~]$ var3=${var1##*ov}  # 贪婪模式
[chao@localhost ~]$ echo $var3
e me
# 尾部匹配删除
[chao@localhost ~]$ var4=${var1%ov*}
[chao@localhost ~]$ echo $var4
I love you, Do you l
[chao@localhost ~]$ var5=${var1%%ov*}  # 贪婪模式
[chao@localhost ~]$ echo $var5
I l
# 替换
[chao@localhost ~]$ var6=${var1/love/hate}
[chao@localhost ~]$ echo $var6
I hate you, Do you love me
[chao@localhost ~]$ var7=${var1//love/hate}
[chao@localhost ~]$ echo $var7
I hate you, Do you hate me
```


### 2. 字符串相关操作
1. 计算字符串的长度

|  语法  |  说明  |
|  ----  | ----  |
|`${#变量}`	|-		|
|`expr length "$变量"`|若字符串变量有空格，则必须加双引号|

``` shell
[chao@localhost ~]$ echo ${#var1}
21
[chao@localhost ~]$ echo `expr length "$var1"`
21
```

2. 字符串其他操作

|  语法  |  说明  |
|  ----  | ----  |
|`expr index $变量 $子串(变量或值)`|获取字符索引的位置，从字串的第一个字符开始匹配，只要有一个字符匹配到了就返回对应的坐标|
|`expr match $变量 $子串`	       |计算子串长度，从头开始匹配，若从头匹配不上，从中间任何一个位置匹配返回都是0，也就是未匹配到|

``` shell
[chao@localhost ~]$ echo `expr index "$var1" love`
3
[chao@localhost ~]$ echo `expr match "$var1" "I love"`
6
```

3. 抽取子串

|  语法  |  说明  |
|  ----  | ----  |
|`${变量:下标}`                     |从变量中的指定下标开始|
|`${变量:下标:length}`              |从指定下标开始，匹配长度为length|
|`${变量: -下标}`                   |从右边指定下标开始匹配 注意冒号和负号中间有空格|
|`${变量:(下标)}`                   |从右边指定下标开始匹配|
|`expr substr $变量 $下标 $length`  |从指定下标开始，匹配长度为length|

``` shell
[chao@localhost ~]$ echo ${var1:10}
you love me
[chao@localhost ~]$ echo ${var1:10:5}
you l
[chao@localhost ~]$ echo ${var1: -4}
e me
[chao@localhost ~]$ echo ${var1:(-4)}
e me
[chao@localhost ~]$ echo ${var1:(-4):2}
e
[chao@localhost ~]$ echo `expr substr "$var1" 10 5`
you
```


### 3. 命令替换
Shell命令替换是指将命令的输出结果赋值给某个变量，有两种方式：一种是反引号`` ` ``，一种是$()。

|  语法  |  说明  |
|  ----  | ----  |
|`` 变量=`命令` ``|可以只有一个命令，也可以有多个命令，多个命令之间以分号;分隔|
|` 变量=$(命令) ` |同上|

``` shell
# 根据系统时间计算今年或明年
[chao@localhost ~]$ echo "This is $(date +%Y) year"
This is 2020 year
[chao@localhost ~]$ echo "This is $(($(date +%Y) + 1)) year"
This is 2021 year
# 注意：$()表示命令替换，仅有()表示计算 加减乘除
```

``` shell
# 判定nginx进程是否存在，若不存在则自动拉起该进程
ps -ef | grep nginx | grep -v grep | wc -l
# 对应脚本
# !/bin/bash
#
nginx_process_num=$(ps -ef | grep nginx | grep -v grep | wc -l)
if [$nginx_process_num -eq 0 ];then
	systemctl start nginx
fi
```


### 4. 设置变量类型
declare 和 typeset 都是 Shell 内建命令，它们的用法相同，都用来设置变量的属性。不过 typeset 已经被弃用了，建议使用 declare 代替。
declare 命令的用法：`declare [+/-] [选项] [变量名=变量值]`，其中，`-`表示设置属性，`+`表示取消属性，其具体含义如下表：

|  选项  |  含义  |
|  ----  | ----  |
|-f [name]		|列出之前由用户在脚本中定义的函数名称和函数体。				|
|-F [name]		|仅列出自定义函数名称。									|
|-g name		|在 Shell 函数内部创建全局变量。							|
|-p [name]		|显示指定变量的属性和值。									|
|-a name		|声明变量为普通数组。										|
|-A name		|声明变量为关联数组（支持索引下标为字符串）。				|
|-i name		|将变量定义为整数型。										|
|-r name[=value]|将变量定义为只读（不可修改和删除），等价于 readonly name。	|
|-x name[=value]|将变量设置为环境变量，等价于 export name[=value]。			|

``` shell
# declare -r 将变量设为只读，再赋值就会报错
[chao@localhost ~]$ declare -r var="hello shell"
[chao@localhost ~]$ var="hello world"
-bash: var: 只读变量

# declare -a 将变量定义为数组
[chao@localhost ~]$ declare -a array
[chao@localhost ~]$ array=("mike" "lilei" "hanmeimei")
[chao@localhost ~]$ echo ${array[0]}
mike

# 分片访问
# 从下标为 0 的位置开始，向后取 2个元素，忽略中间的空元素，直到取够 2个元素。如果元素不足2个，则输出后面的所有元素即可。
[chao@localhost ~]$ echo ${array[@]:0:2}
mike lilei

# 内容替换
[chao@localhost ~]$ array1=${array[@]/ke/KE}
[chao@localhost ~]$ echo $array1
miKE lilei hanmeimei

# 数组遍历
[chao@localhost ~]$ 
> for arr in ${array[@]}
> do
> echo $arr
> done
mike
lilei
hanmeimei

# 给数组某个下标赋值
[chao@localhost ~]$ array[2]="zhangsan"
# 删除元素
[chao@localhost ~]$ unset array[1]
# 清空整个数组
[chao@localhost ~]$ unset array
```


### 5. 数学运算
|  算术运算符  |  说明/含义  |
|  ----  | ----  |
| +、- 				|加法（或正号）、减法（或负号）|
| *、/、% 				|乘法、除法、取余（取模）|
| ** 					|幂运算|
| ++、-- 				|自增和自减，可以放在变量的前面也可以放在变量的后|
| !、&&、&#124;&#124; 	|逻辑非（取反）、逻辑与（and）、逻辑或（or）|
| <、<=、>、>= 			|比较符号（小于、小于等于、大于、大于等于）|
| ==、!=、= 				|比较符号（相等、不相等；对于字符串，= 也可以表示相当于）|
| <<、>> 				|向左移位、向右移位|
| ~、&#124;、 &、^ 		|按位取反、按位或、按位与、按位异或|
| =、+=、-=、*=、/=、%= 	|赋值运算符，例如 a+=1 相当于 a=a+1，a-=1 相当于 a=a-1|

在Shell中，如果不特别指明，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。
``` shell
[chao@localhost ~]$ echo 2+8
2+8
[chao@localhost ~]$ a=23
[chao@localhost ~]$ echo $a + 55
23 + 55
```

#### 5.1 数学运算命令
Shell中常用的六种数学计算方式:

|运算操作符/运算命令|说明|
|  ----  | ----  |
|`(( ))`		|用于整数运算，效率很高，推荐使用。|
|`let`			|用于整数运算，和 (()) 类似。|
|`$[]`			|用于整数运算，不如 (()) 灵活。|
|`expr`			|可用于整数运算，也可以处理字符串。|
|`bc`			|Linux下的一个计算器程序，可以处理整数和小数。Shell 本身只支持整数运算，想计算小数就得使用 bc 这个外部的计算器。|
|`declare -i`	|将变量定义为整数，然后再进行数学运算时就不会被当做字符串了。功能有限，仅支持最基本的数学运算（加减乘除和取余），不支持逻辑运算、自增自减等，所以在实际开发中很少使用。|

``` shell
# expr 计算，注意>,*等运算需要转义
[chao@localhost ~]$ num1=305
[chao@localhost ~]$ num2=50
[chao@localhost ~]$ expr $num1 + $num2
355
[chao@localhost ~]$ expr $num1 \* $num2
15250

# 判断一个变量是一个整数
[chao@localhost ~]$ expr $num1 + 1
306
[chao@localhost ~]$ expr $num3 + 1
expr: 非整数参数
```

* `bc`是bash内建的运算器，支持浮点数运算，内建变量scale可以设置精确度(默认为0)，操作符`^`用于指数运算。

``` shell
[chao@localhost ~]$ echo "3*8"|bc
24
[c.biancheng.net]$ echo "scale=4;3*8/7"|bc
3.4285
#十六进制转十进制
[chao@localhost ~]$ m=1E
[chao@localhost ~]$ n=$(echo "obase=10;ibase=16;$m"|bc)
[chao@localhost ~]$ echo $n
30
```
