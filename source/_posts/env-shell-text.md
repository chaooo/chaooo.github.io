---
title: 「Shell」Shell 文本处理
date: 2020-06-09 15:12:46
tags: [环境配置, Linux, Shell]
categories: 环境配置
---


### 1. 文件查找命令
#### 1.1 find命令
语法格式：`find 搜索路径 [选项] 搜索内容`<!-- more -->

|  选项  |  解析  |
|  ----  | ----  |
|  -name  |  按照文件名搜索  |
|  -iname  |  按照文件名搜索，不区分文件名大小  |
|  -inum  |  按照 inode 号搜索  |
|  -type  |  根据文件类型(f:文件,d:目录,c:字符设备文件,b:块设备文件,l:链接文件,p:管道文件)搜索  |
|  -size  |  根据文件大小(单位:ckMGTP)搜索，`-`小于，`+`大于，例如查找/etc目录下大于1M的文件：find /etc -size +1M  |
|  -mtime  |  根据修改时间(单位:smhdw)搜索  |
|  -ctime  |  根据创建时间(单位:smhdw)搜索  |
|  -atime  |  根据被访问时的时间间隔(单位:smhdw)搜索  |
|  -mmin  |   n分钟以(-n:内，+n:外)内修改的文件  |
|  -mindepth   |  从n级子目录开始搜索,最多搜索到n-1级子目录  |
|  -depth   |  检索深度为 n 的文件，即位于指定目录以下 n 层的文件  |
|  -empty  | 检索空文件或空目录  |
|  -perm  | 根据文件权限搜索  |
|  -ls  | 打印搜索到的文件的详细信息  |
|  -delete  | 删除检索到的文件  |
|  -exec  |  对搜索的文件常用操作("-exec"和"-ok"相似，对文件执行特定的操作，"-ok"得到确认命令后，才会执行；-print打印输出)  |


#### 1.2 locate命令
2. `locate`命令，不同于find命令是在整块磁盘中搜索，locate命令是在数据库文件中查找
    + find是默认全局匹配，locate则是默认部分匹配
    + 文件更新后，用updatedb命令把文件更新到数据库(默认是第二天系统才会自动更新到数据库)，否则locate查找不到

#### 1.3 whereis命令
3. `whereis`命令，只能用于程序名的搜索
    + 命令参数：二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

#### 1.4 which命令
4. `which`命令，仅查找二进制程序文件


### 2. Linux文本处理三剑客
文本处理三剑客工具`grep`，`sed`和`awk`都是基于行处理的，它们会一行行读入数据，处理完一行之后再处理下一行。
#### 2.1 文件处理三剑客之grep
`grep`命令，用于查找文件里符合条件的字符串。
* 语法格式：
    + 语法1：`grep [option] [ pattern] [file1, file2..]`
    + 语法2：`command | grep [option] [pattern]`

|  选项(option)  |  解析  |
|  ----  | ----  |
|-v		|不显示匹配行信息							|
|-i		|搜索时忽略大小写							|
|-n		|显示行号								|
|-r		|递归搜索								|
|-E		|支持扩展正则表达式						|
|-F		|不按正则表达式匹配，按照字符串字面意思匹配	|
|-c		|只显示匹配行总数							|
|-w		|匹配整词								|
|-x		|匹配整行								|
|-l		|只显示文件名，不显示内容					|
|-s		|不显示错误信息							|

* `grep`和`egrep`
    + `grep`默认不支持扩展正则表达式，只支持基础正则表达式
    + 使用`grep-E`可以支持扩展正则表达式使用
    + `egrep`可以支持扩展正则表达式，与`grep-E`等价


#### 2.2 文件处理三剑客之sed
`sed`（Stream Editor），流编辑器。对标准输出或文件逐行进行处理
* 语法格式：
    + 语法1：`stdout |sed [option] "pattern command"`
    + 语法2：`sed [option] "pattern command" file`


##### 2.2.1 sed选项

|  选项(option)  |  解析  |
|  ----  | ----  |
|-n		|只打印模式匹配行					|
|-e		|直接在命令行进行sed编辑，默认选项	|
|-f		|编辑动作保存在文件中，指定文件执行	|
|-r		|支持扩展正则表达式				|
|-i		|直接修改文件内容					|


##### 2.2.2 sed匹配模式

|  匹配模式(pattern)  |  解析  |
|  ----  | ----  |
|10command						|匹配到第10行								|
|10,20command					|匹配从第10行开始，到第20行结束10，			|
|10,+5command					|匹配从第10行开始，到第16行结束				|
|/pattern1/command				|匹配到pattern1的行							|
|/pattern1/,/pattern2/command	|匹配到pattern1的行开始，到匹配到patern2的行结束|
|10,/pattern1/command			|匹配从第10行开始，到匹配到pettern1的行结束		|
|/pattern1/,10command			|匹配到pattern1的行开始，到第10行匹配结束		|

``` shell
# 打印test.txt文件的第17行
sed -n "17p" test.txt

# 打印文件的10到20行
sed -n "10,20p" test.txt

# 打印test.txt文件中从第10行开始，往后面加5行
sed -n "10,+5p" test.txt

# 打印test.txt文件中以root开头的行
sed -n "/^root/p" test.txt

# 打印test.txt文件中第一个匹配到以ftp开头的行，到mail开头的行结束
sed -n "/^ftp/,/^mail/p" test.txt

# 打印test.txt文件中从第4行开始匹配，直到以hdfs开头的行结束
sed -n "4,/^hdfs/p" test.txt

# 打印test.txt文件中匹配root的行，直到第10行结束
sed -n "/root/,10p"test.txt
```


##### 2.2.3 sed中的编辑命令

|  编辑命令  |  类别  |  含义  |
|  ----  |  ----  |  ----  |
|p				|查询|打印						|
|=				|查询|只显示行号					|
|a				|增加|行后追加					|
|i				|增加|行前追加					|
|r				|增加|外部文件读入，行后追加		|
|w				|增加|匹配行写入外部文件			|
|d				|删除|删除						|
|s/old/new		|修改|将行内第一个old替换为new		|
|s/old/new/g	|修改|将行内全部的old替换为new						|
|s/old/new/2g	|修改|将行内从第2个old开始到剩下所有的old替换为new	|
|s/old/new/ig	|修改|将行内全部的old替换为new，忽略大小写			|

**反向引用**

``` shell
[cmuser@localhost test]$ cat abc.txt
First Line
Second haha
[cmuser@localhost test]$ sed -i 's/Sec..d/&s/g' abc.txt
[cmuser@localhost test]$ cat abc.txt
First Line
Seconds haha
[cmuser@localhost test]$ sed -i 's/\(Sec..ds\)/\10/g' abc.txt
[cmuser@localhost test]$ cat abc.txt
First Line
Seconds0 haha
[cmuser@localhost test]$ sed -i 's/\(Sec\)...../\1FFFFFFFFFF/g' abc.txt
[cmuser@localhost test]$ cat abc.txt
First Line
SecFFFFFFFFFF haha
```
* `&`和`\1`引用模式匹配到的整个串
    + 两者区别在于只能表示匹配到的完整字符串，只能引用整个字符串；而`\1`可以使用`()`匹配到的字符

* sed中引用变量时注意事项：
    1. 匹配模式中存在变量，则建议使用双引号
    2. sed中需要引入自定义变量时，如果外面使用单引号，则自定义变量也必须使用单引号

``` shell
[cmuser@localhost test]$ old_str=First
[cmuser@localhost test]$ new_str=One
[cmuser@localhost test]$ sed -i "s/$old_str"/"$new_str/g" abc.txt
[cmuser@localhost test]$ cat abc.txt
One Line
SecFFFFFFFFFF haha
```

##### 2.2.4 sed实例
1. sed查找文件内容（处理一个MySQL配置文件my.cnf的文本，示例如下；编写脚本实现以下功能：输出文件有几个段，并且针对每个段可以统计配置参数总个数）
``` shell
[jenkins@caimeidev1 test]$ vim test1.sh
#!/bin/bash

FILE_NAME=/etc/my.cnf

function get_all_segments {
    echo "`sed -n '/\[.*\]/p' $FILE_NAME |sed -e 's/\[//g' -e 's/\]//g'`"
}

function count_items_in_segment {
    echo "`sed -n '/\['$1'\]/,/\[.*\]/p' $FILE_NAME  | grep -v ^# |grep -v ^$ |grep -v "\[.*\]" |wc -l `"
}

num=0
for seg in `get_all_segments`
do
    num=`expr $num + 1`
    items_count=`count_items_in_segment $seg`
    echo "$num:$seg  $items_count"
done
```

输出结果：
``` shell
[jenkins@caimeidev1 test]$ ./test1.sh
1:mysqld  9
2:mysqld_safe  2
3:mysql  1
4:client  1
```

2. sed删除和修改文件内容

``` shell
# 删除文件中的所有注释行和空行
sed -i '/[:blank:]*#/d;/^$/d' nginx.conf
# 在配置文件中所有不以#开头的行前面添加*符号，注意：以#开头的行不添加
sed -i 's/^[^#]/\*&/g' nginx.conf

# 修改/etc/passwd中第1行中第1个root为ROOT	
sed -i '1s/root/ROOT/' passwd
# 修改/etc/passwd中第5行到第10行中所有的/sbin/nologin为/bin/bash
sed -i '5,10s/\/sbin\/nologin/\/bin\/bash/g' passwd
# 修改/etc/passwd中匹配到/sbin/nologin的行，将匹配到行中的login改为大写的LOGIN
sed -i '/\/sbin\/nologin/s/login/LOGIN/g' passwd
# 修改/etc/passwd中从匹配到以root开头的行，到匹配到行中包含mail的所有行。修改内为将这些所有匹配到的行中的bin改为HADOOP
sed -i '/^root/,/mail/s/bin/HADOOP/g' passwd
# 修改/etc/passwd中从匹配到以root开头的行，到第15行中的所有行，修改内容为将这些行中的nologin修改为SPARK
sed -i '/^root/,15s/nologin/SPARK/g' passwd
# 修改/etc/passwd中从第15行开始，到匹配到以yarn开头的所有行，修改内容为将这些行中的bin换位BIN
sed -i '15,/^yarn/s/bin/BIN/g' passwd 
```

3. sed追加文件内容（a:在匹配行后面追加，i:在匹配行前面追加，r:将文件内容追加到匹配行后面，w:将匹配行写入指定文件）

``` shell
# a:在匹配行后面追加
# (1)、passwd文件第10行后面追加"Add Line Behind"		
sed -i '10a Add Line Begind' passwd
# (2)、passwd文件第10行到第20行，每一行后面都追加"Test Line Behind"
sed -i '10,20a Test Line Behind' passwd
# (3)、passwd文件匹配到/bin/bash的行后面追加"Insert Line For /bin/bash Behind"
sed -i '/\/bin\/bash/a Insert Line For /bin/bash Behind' passwd

# i:在匹配行前面追加
# (1)、passwd文件匹配到以yarn开头的行，在匹配行前面追加"Add Line Before"
sed -i '/^yarn/i Add Line Before' passwd
# (2)、passwd文件每一行前面都追加"Insert Line Before Every Line"
sed -i 'i Insert Line Before Every Line' passwd

# r:将文件内容追加到匹配行后面
# (1)、将/etc/fstab文件的内容追加到passwd文件的第20行后面
sed -i '20r /etc/fstab' passwd
# (2)、将/etc/inittab文件内容追加到passwd文件匹配/bin/bash行的后面
sed -i '/\/bin\/bash/r /etc/inittab' passwd
# (3)、将/etc/vconsole.conf文件内容追加到passwd文件中特定行后面，匹配以ftp开头的行，到第18行的所有行
sed -i '//,18r /etc/vconsole.conf' passwd

# w:将匹配行写入指定文件	
# (1)、将passwd文件匹配到/bin/bash的行追加到/tmp/sed.txt文件中
sed -i '/\/bin\/bash/w /tmp/sed.txt' passwd
# (2)、将passwd文件从第10行开始，到匹配到hdfs开头的所有行内容追加到/tmp/sed-1.txt
sed -i '10,/^hdfs/w /tmp/sed-1.txt' passwd
```


#### 2.3 文件处理三剑客之awk
`awk`是一个文本处理工具，通常用于处理数据并生成结果报告

``` shell
awk 'BEGIN{} pattern{commands}END{}' file_name
standard output | awk 'BEGIN{}pattern{commands}END{}'
```

|  语法格式  |  解析  |
|  ----  | ----  |
|BEGIN{}	|正式处理数据之前执行		|
|pattern	|匹配模式				|
|{commands}	|处理命令，可能多行		|
|END{}		|处理完所有匹配数据后执行	|

##### 2.3.1 awk内置变量

|  内置变量  |  解析  |
|  ----  | ----  |
|$0			|打印行所有信息													|
|$1~$n		|打印行的第1到n个字段的信息										|
|NF			|Number Field 处理行的字段个数									|
|NR			|Number Row 处理行的行号，从1开始计数								|
|FNR		|File Number Row 多文件处理时，每个文件单独记录行号，都是从0康凯斯	|
|FS			|Field Separator 字段分割符，不指定时默认以空格或tab键分割			|
|RS			|Row Separator 行分隔符，不指定时以回车分割\n						|
|OFS		|Output Filed Separator 输出字段分隔符。							|
|ORS		|Output Row Separator 输出行分隔符								|
|FILENAME	|处理文件的文件名													|
|ARGC		|命令行参数个数													|
|ARGV		|命令行参数数组													|

``` shell
[cmuser@localhost test]$ awk '{print $0}' abc.txt
One Line
SecFFFFFFFFFF haha
[cmuser@localhost test]$ awk '{print $1}' abc.txt
One
SecFFFFFFFFFF
[cmuser@localhost test]$ awk '{print NR}' abc.txt
1
2
```


##### 2.3.2 awk格式化输出之printf

|  格式符  |  解析  |
|  ----  | ----  |
|%s		|打印字符串				|
|%d		|打印10进制数			|
|%f		|打印浮点数				|
|%x		|打印16进制数			|
|%o		|打印8进制数				|
|%e		|打印数字的科学计数法格式	|
|%c		|打印单个字符的ASCII码	|

|  修饰符  |  解析  |
|  ----  | ----  |
|-		|左对齐									|
|+		|右对齐									|
|#		|显示8进制在前面加o，显示16进制在前面加0x	|

``` shell
# 1、以字符串格式打印/etc/passwd中的第7个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%s\n",$7}' /etc/passwd
# 2、以10进制格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%d\n",$3}' /etc/passwd
# 3、以浮点数格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%0.3f\n",$3}' /etc/passwd
# 4、以16进制数格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%#x\n",$3}' /etc/passwd
# 5、以8进制数格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%#o\n",$3}' /etc/passwd
# 6、以科学计数法格式打印/etc/passwd中的第3个字段，以":"作为分隔符
awk 'BEGIN{FS=":"} {printf "%e\n",$3}' /etc/passwd
```


##### 2.3.3 awk模式匹配的两种用法
* `awk`模式匹配：
    1. `RegExp`：按正则表达式匹配
    2. `关系运算匹配`：按关系运算匹配

1. RegExp(正则表达式匹配)

``` shell
# 匹配/etc/passwd文件行中含有root字符串的所有行
awk 'BEGIN{FS=":"}/root/{print $0}' /etc/passwd
# 匹配/etc/passwd文件行中以yarn开头的所有行
awk 'BEGIN{FS=":"}/^yarn/{print $0}' /etc/passwd
```

2. 运算符匹配(`<`:小于，`>`:大于，`<=`:小于等于，`>=`:大于等于，`==`:等于，`!=`:不等于，`~`:匹配正则表达式，`!~`:不匹配正则表达式)

``` shell
# 以:为分隔符，匹配/etc/passwd文件中第3个字段小于50的所有行信息
awk 'BEGIN{FS=":"}$3<50{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段大于50的所有行信息
awk 'BEGIN{FS=":"}$3>50{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第7个字段为/bin/bash的所有行信息
awk 'BEGIN{FS=":"}$7=="/bin/bash"{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第7个字段不为/bin/bash的所有行信息
awk 'BEGIN{FS=":"}$7!="/bin/bash"{print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd中第3个字段包含3个以上数字的所有行信息
awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{print $0}' /etc/passwd
```

3. 布尔运算符匹配(`||`:或，`&&`:与，`!`:非)

``` shell
# 以:为分隔符，匹配/etc/passwd文件中包含hdfs或yarn的所有行信息
awk 'BEGIN{FS=":"}$1=="hdfs" || $1=="yarn" {print $0}' /etc/passwd
# 以:为分隔符，匹配/etc/passwd文件中第3个字段小于50并且第4个字段大于50的所有行信息
awk 'BEGIN{FS=":"}$3<50 && $4>50 {print $0}' /etc/passwd
```


##### 2.3.4 awk表达式用法

|  运算符  |  解析  |
|  ----  | ----  |
|`+`		|加							|
|`/`		|除							|
|`%`		|模							|
|`^`或`**`	|乘方						|
|`++x`	|在返回x变量之前，×变量加1	|
|`X++`	|在返回x变量之后，×变量加1	|

``` shell
# 使用awk计算/etc/services中的空白行数量
awk '/^$/{sum++}END{print sum}' /etc/services
```


##### 2.3.5 awk动作中的条件及循环语句

``` shell
# 以:为分隔符，只打印/etc/passwd中第3个字段的数值在50-100范围内的行信息
awk 'BEGIN{FS=":"}{
    if($3<50){ 
        printf "%-20s%-20s%-5d\n","小于50的UID",$1,$3
    } else if($3>100) {
        printf "%-20s%-20s%-5d\n","大 于100的UID",$1,$3
    } else{
        printf "%-20s%-20s%-5d\n","大于50 小于100的UID",$1,$3
    }
}' /etc/passwd
```


##### 2.3.6 awk中的字符串函数

|  函数名  |  解析  |
|  ----  | ----  |
|length(str)		|计算长度														|
|index(str1,str2)	|返回在str1中查询到的str2的位置									|
|tolower(str)		|小写转换														|
|toupper(str)		|大写转换														|
|split(str,arr,fs)	|分隔字符串，并保存到数组中										|
|match(str,RE)		|返回正则表达式匹配到的子串的位置									|
|substr(str,m,n)	|截取子串，从m个字符开始，截取n位。n若不指定，则默认截取到字符串尾	|
|sub(RE,RepStr,str)	|替换查找到的第一个子串											|
|gsub(RE,RepStr,str)|替换查找到的所有子串											|

``` shell
# 搜索字符串"I have a dream"中出现"ea"子串的位置
awk 'BEGIN{str="I hava a dream";location=index(str,"ea");print location}
awk 'BEGIN{str="I hava a dream";location=match(str,"ea");print location}'

# 将字符串"Hadoop is a bigdata Framawork"全部转换为小写
awk 'BEGIN{str="Hadoop is a bigdata Framework";print tolower(str)}'

# 将字符串"Hadoop is a bigdata Framawork"全部转换为大写
awk 'BEGIN{str="Hadoop is a bigdata Framework";print toupper(str)}'

# 将字符串"Hadoop Kafka Spark Storm HDFS YARN Zookeeper"，按照空格为分隔符，分隔每部分保存到数组array中
awk 'BEGIN{str="Hadoop Kafka Spark Storm HDFS YARN Zookeeper";split(str,arr," ");for(a in arr) print arr[a]}'

# 搜索字符串"Tranction 2345 Start:Select * from master"第一个数字出现的位置
awk 'BEGIN{str="Tranction 2345 Start:Select * from master";location=match(str,/[0-9]/);print location}'

# 截取字符串"transaction start"的子串，截取条件从第4个字符开始，截取5位
awk 'BEGIN{str="transaction start";print substr(str,4,5)}'

# 替换字符串"Tranction 243 Start,Event ID:9002"中第一个匹配到的数字串为$符号
awk 'BEGIN{str="Tranction 243 Start,Event ID:9002";count=sub(/[0-9]+/,"$",str);print count,str}'
awk 'BEGIN{str="Tranction 243 Start,Event ID:9002";count=gsub(/[0-9]+/,"$",str);print count,str}'
```


##### 2.3.7 awk的常用选项

|  选项  |  解析  |
|  ----  | ----  |
|-v		|参数传递		|
|-f		|指定脚本文件	|
|-F		|指定分隔符		|
|-V		|查看awk的版本号|


##### 2.3.8 awk中数组的用法
在awk中，使用数组时，不仅可以使用1.2…n作为数组下标，也可以使用字符串作为数组下标。

``` shell
# 打印元素：
echo ${array[2]}

# 打印元素个数：
echo ${#array[@]}

# 打印元素长度：
echo ${#array[3]}

# 给元素赋值：
array[3]="Li"

# 删除元素：
unset array[2];unset array

# 分片访问：
echo ${array[@]:1:3}

# 元素内容替换：
${array[@]/e/E}只替换第一个e；
$tarray[@]//e/E}替换所有的e

# 数组的遍历：
for a in array 
do
	echo $a 
done 

# 使用字符串作为数组下标
array["var1"]="Jin"
array["var2"]="Hao"
array["var3"]="Fang"
for(a in array)
	print array[a]
```

``` shell
# 统计主机上所有的TCP连接状态数，按照每个TCP状态分类
netstat -an | grep tcp | awk '{array[$6]++} END{for(a in array) print a,array[a]}'
```
