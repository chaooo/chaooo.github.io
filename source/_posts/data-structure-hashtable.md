---
title: 「数据结构与算法」散列表（Hash Table）
date: 2022-03-18 17:46:12
tags: [算法, 数据结构]
categories: 数据结构
---

散列表（`Hash Table`），也叫“哈希表”或者“Hash表”。是能够通过给定的关键字的值直接访问到具体对应的值的一个数据结构。

通常，我们把这个关键字称为 `Key`，把对应的记录称为 `Value`，所以也可以说是通过 `Key` 访问一个映射表来得到 `Value` 的地址。而这个映射表，也叫作散列函数或者哈希函数，存放记录的数组叫作散列表。

散列表用的是**数组支持按照下标随机访问数据**的特性，所以散列表其实就是**数组的一种扩展**，由数组演化而来。
<!-- more -->

`Hash`，一般翻译做“散列”/“哈希”，就是把任意长度的输入，通过散列算法，变换成固定长度的输出，该输出就是散列值。
(这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，这种现象叫作碰撞（Collision）。)
简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

对散列表中的键(`key`)进行`Hash`的函数就是散列函数（`Hash(key)`）。函数的计算结果就是散列值,就是`Hash(key)`的值。

### 1. 几种常见哈希函数：
+ 直接寻址法：取关键字或关键字的某个线性函数值为散列地址；如`Hash(key)=key`或者`Hash(key)=a*key+b`，`a`和`b`都为常数。
+ 数字分析法：如果关键字由多位字符或者数字组成，就可以考虑抽取其中的`2`位或者多位作为该关键字对应的散列地址，在取法上尽量选择变化较多的位，避免冲突发生。
+ 平方取中法：对关键字做平方操作，取平方值的中间几位作为散列地址。此方法也是比较常用的构造哈希函数的方法。计算平方之后的中间几位和关键字中的每一位都相关，所以不同的关键字会以较高的概率产生不同的散列地址。
+ 折叠法：将关键字分割成位数相同的几部分（最后一部分的位数可以不同），然后取这几部分的叠加和（舍去进位）作为哈希地址。此方法适合关键字位数较多的情况。
+ 取随机数法：使用一个随机函数，取关键字的随机值作为散列地址，这种方式通常用于关键字长度不同的场合。
+ 除留余数法：若已知整个哈希表的最大长度`m`，可以取一个不大于`m`的数`p`，然后对该关键字`key`做取余运算，即：`Hash(key)=key%p`。
+ 随机数法：取关键字的一个随机函数值作为它的哈希地址，即：`Hash(key)=random(key)`，此方法适用于关键字长度不等的情况。

构建哈希函数，需要根据实际的查找表的情况采取适当的方法。通常考虑的因素有以下几方面：
1. 关键字的长度。如果长度不等，就选用随机数法。如果关键字位数较多，就选用折叠法或者数字分析法；反之如果位数较短，可以考虑平方取中法；
2. 哈希表的大小。如果大小已知，可以选用除留余数法；
3. 关键字的分布情况；
4. 查找表的查找频率；
5. 计算哈希函数所需的时间（包括硬件指令的因素）


### ２. 几种哈希冲突的处理方式：
+ 开放寻址法：`Hash(key) = (Hash(key) + d) MOD m`；（其中`m`为哈希表的表长，`d`为一个增量）。当得出的哈希地址产生冲突时，选取一种探测方法获取`d`的值，然后继续计算，直到计算出的哈希地址不在冲突为止。三种探测方法：
    - 线性探测法：d=1，2，3，…，m-1 （每次 +1，向右探测，直到有空闲的位置为止）
    - 二次探测法：d=12，-12，22，-22，32，… （按照 +12，-12，+22，…如此探测，直到有空闲的位置）
    - 伪随机数探测法：d=伪随机数 （每次加上一个随机数，直到探测到空闲位置结束）
+ 再哈希法：当通过哈希函数求得的哈希地址同其他关键字产生冲突时，使用另一个哈希函数计算，直到冲突不再发生。
+ 链地址法：链地址法其实就是对Key通过哈希之后落在同一个地址上的值，做一个链表。
+ 建立一个公共溢出区：建立两张表，一张为基本表，另一张为溢出表。基本表存储没有发生冲突的数据，当关键字由哈希函数生成的哈希地址产生冲突时，就将数据填入溢出表。


### ３. 散列表的特点
1. 访问速度很快
    + 由于散列表有散列函数，可以将指定的 Key 都映射到一个地址上，所以在访问一个 Key（键）对应的 Value（值）时，根本不需要一个一个地进行查找，可以直接跳到那个地址。所以我们在对散列表进行添加、删除、修改、查找等任何操作时，速度都很快。
2. 需要额外的空间
    + 首先，散列表实际上是存不满的，如果一个散列表刚好能够存满，那么肯定是个巧合。而且当散列表中元素的使用率越来越高时，性能会下降，所以一般会选择扩容来解决这个问题。另外，如果有冲突的话，则也是需要额外的空间去存储的，比如链地址法，不但需要额外的空间，甚至需要使用其他数据结构。
    + 这个特点有个很常用的词可以表达，叫作“空间换时间”，在大多数时候，对于算法的实现，为了能够有更好的性能，往往会考虑牺牲些空间，让算法能够更快些。
3. 无序
    + 散列表还有一个非常明显的特点，那就是无序。为了能够更快地访问元素，散列表是根据散列函数直接找到存储地址的，这样我们的访问速度就能够更快，但是对于有序访问却没有办法应对。
4. 可能会产生碰撞
    + 没有完美的散列函数，无论如何总会产生冲突，这时就需要采用冲突解决方案，这也使散列表更加复杂。通常在不同的高级语言的实现中，对于冲突的解决方案不一定一样。

