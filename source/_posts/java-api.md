---
title: 「Java教程」核心工具类
date: 2017-04-12 16:42:47
tags: [Java, 后端开发]
categories: Java教程
---


API (Application Programming Interface) 应用程序编程接口，Java中的API，就是JDK提供的各种功能的Java类。
<!-- more -->

### 常用的包
- java.lang包：是Java最核心的包，JVM(Java虚拟机)启动时自动加载lang包的所有类和接口，无需import。如：System类、String类、Object类、Class类...
- java.util包：是Java工具包，包括很多工具类和集合。如：Scanner类、Random类...
- java.io包：是输入输出包，包括读写各种设备。
- java.net包：是网络编程的包，包括各种网络编程。
- java.sql包：是操作数据库的所有类和接口。


### 1. Object类与其常用方法
#### 1.1 Object类
- java.lang.Object类在Java类继承结构中位于顶端(根类)，任何类都是该类的直接或间接子类。
- Object定义了“对象”的基本行为，被子类默认继承。

#### 1.2 equals() 和 hashCode()
- boolean equals()方法用于非空对象的“相等”逻辑，默认比较两个对象的地址，返回布尔值。
- equals()方法要求：自反性/对称性/传递性/一致性/非空性。
- Java类可以根据需要重写继承自Object的equals()方法。

> 注意：当equals()方法被重写时，必须重写hashCode方法，以维护hashCode方法的常规协定，该协定声明相等对象必须具有相等的哈希码。

- int hashCode():返回对象的哈希码值，对应一个内存。
- hashCode规范要求：
    * 一致性，同一对象，若没有改变属性值，多次调用其hashCode应该时一致的
    * 如果两个对象判定相等，它们的hashCode应该时同一个值
    * 如果两个对象不相等，它们的hashCode可以相同，但最好不相同而可以提高哈希表的性能。

- hashCode()方法和equals()方法的判断条件必须保持一致，如果重写一个，另一个也必须重写。

#### 1.3 toString()
- String toString()：用于获取调用对象的字符串形式，返回"包名.类名@hashCode值的16进制"。
- Java类可以根据需要重写toString方法返回更有意义的信息。
- Java在使用System.out.println()打印对象时或者`+`连接字符串时，默认调用toString()方法。


### 2. 包装类
#### 2.1 包装类
- 由于某些特殊场合(集合)中要求所有数据内容都必须是对象，而对于基本数据类型的变量来说不满足该要求，为了使得该变量也能够使用就需要对变量打包处理变成对象，此时就需要借助包装类。
- Java语言8种基本类型分别对应了8中“包装类”，每一种包装类都封装了一个对应的基本类型成员变量，还提供了一些针对该数据类型的实用方法。

|基本类型 |对应包装类         |
|--------|------------------|
|byte    |java.lang.Byte    |
|short   |java.lang.Short   |
|int     |java.lang.Integer |
|long    |java.lang.Long    |
|float   |java.lang.Float   |
|double  |java.lang.Double  |
|boolean |java.lang.Boolean |
|char    |java.lang.Character|

1. 八个包装类都在同一个包下（java.lang包），不需要import导包直接使用
2. 八个包装类中有六个是与数字相关，都默认继承父类Number
3. 八个包装类都实现了Serializable, Comparable
4. 八个包装类都有带自己对应类型参数的构造方法，其中有七个(除了Character)还有构造方法重载，带String类型
5. 八个包装类都提供了各自对应的拆包方法，如intValue,floatValue,将包装类对象拆成基本类型

#### 2.2 Integer类
- java.lang.Integer类是int类型的包装类，该类型对象中包含一个int类型的成员变量。该类由final关键字修饰表示不能被继承。
- Integer类重写了**equals()**方法（重写后比较的是数值）、hashCode()以及toString()方法。

|Integer类的常用方法|                  |
|-----------------|------------------|
|Integer(int i)               |根据参数指定整数来构造对象|
|Integer(String s)            |根据参数指定的字符串来构造对象|
|int intValue()               |获取调用对象中整数值并返回|
|static Integer valueOf(int i)|根据参数指定整数值得到Integer类型对象|
|static int parseInt(String s)|将字符串类型转换为int类型并返回|

#### 2.3 装箱和拆箱

``` java
int i = 100;
Integer it = Integer.valueOf(i); //实现了int类型到Integer类型的转换，这个过程叫做装箱
int ia = it.intValue();//实现了Integer类型到int类型的转换，这个过程叫做拆箱
//jdk5增加了自动拆箱和装箱功能（编译器预处理）:
Integer i = 100;//自动装箱
int ia = i;//自动拆箱
```

- 笔试考点：
> * 在Integer类部提供了自动装箱池技术，将**-128~127间的整数已经装箱完毕**，当使用该范围整数时直接取池中的对象即可，从而提高效率。
> * Integer类加载的时候，自己有一个静态的空间立即加载Integer类型的数组，存储256个Integer对象（-128 ~ 127），当使用该范围整数时，直接取静态区中找对应的对象；如果我们用的对象范围会帮我们创建一个新的Integer对象。

``` java
Integer it1 = 128;
Integer it2 = 128;
Integer it3 = new Integer(128);
Integer it4 = new Integer(128);
System.out.println(it1.equals(it2));//比较内容 true
System.out.println(it1 == it2);//比较地址 false
System.out.println(it3.equals(it4));//比较内容 true
System.out.println(it3 == it4);//比较地址 false

Integer it5 = 127;
Integer it6 = 127;
Integer it7 = new Integer(127);
Integer it8 = new Integer(127);
System.out.println(it5.equals(it6));//比较内容 true
System.out.println(it5 == it6);//比较地址 true, 自动装箱池范围-128~127。
System.out.println(it7.equals(it8));//比较内容 true
System.out.println(it7 == it8);//比较地址 false
```


### 3. 数学处理类
- java.lang.Math构造方法是私有的，我们不能直接调用创建对象；由于Math中提供的属性及方法都是static  不需要创建对象。

| 常用的方法             |返回值类型|  |
|-----------------------|---------|---|
|Math.abs()             |      |返回给定数字的绝对值(参数 int long float double)|
|Math.ceil()            |double| 向上取整|
|Math.floor()           |double| 向下取整|
|Math.rint()            |double| 临近的整数 如果两边距离一样 则返回偶数|
|Math.round()           |int   | 四舍五入的整数|
|Math.max(a,b)/min(a,b) |      | (参数int  long  float  double)|
|Math.pow(a,b)          |double| a的b次方  (参数double 返回值double)|
|Math.sqrt(double a)    |      |获取给定参数的平方根|
|Math.random()          |double|随机产生一个[0.0--1.0)|

- 0-9之间的随机整数：int value = (int)**(Math.random()*10**);
- Math.random()计算小数的时候精确程度可能有些损失

#### 3.1 Random类
- java.util.Random，在java.util包中的类，需要import导入，没有任何继承关系  默认继承Object类

| 常用的方法                |Random r = new Random();  |
|--------------------------|--------|
| r.nextInt();             |随机产生 int取值范围的整数 有正有负(`-2^31`\~`2^31-1`即`正负21亿`之间)|
| r.nextInt(int bound);    |随机产生一个[0--bound)整数；注意bound必须为正数，否则会出现如下的运行时异常：IllegalArgumentException|
| r.nextFloat()            |随机产生一个 [0.0---1.0)|
| r.nextBoolean()          |随机产生一个boolean值   true  false|

#### 3.2 UUID类
- java.util.UUID，在java.util包中的类，需要import导入，没有任何继承关系  默认继承Object类
- 只有有参构造方法，我们通常不会创建对象
- UUID uuid = UUID.randomUUID();//通常用于数据库表格主键 primary key
- 产生一个32位的随机元素 每一个位置是一个16进制的数字

#### 3.3 BigDecimal
- java.math.BigDecimal类处理大浮点数，需要import导入，继承自Number
- Java浮点数据类型(float和double)在运算时会有舍入误差，如果希望得到精确运算结果，可以使用java.math.BigDecimal类。
- 提供的构造方法全部都是带参数的
    * 通常利用带String参数的构造方法创建这个类的对象：BigDecimal  bi = new BigDecimal("1.23");

|BigDecimal类的常用方法|                  |
|-----------------|------------------|
|BigDecimal(String val)                       | 根据参数指定的字符串来构造对象|
|BigDecimal    setScale(int newScale, RoundingMode roundingMode)|两个参数前面是保留小数点之后的位数，后面参数是设置的模式(向上取整或向下等)|
|BigDecimal **add**(BigDecimal augend)            | 用于实现**加法**运算 |
|BigDecimal **subtract**(BigDecimal subtrahend)   | 用于实现**减法**运算 |
|BigDecimal **multiply**(BigDecimal multiplicand) | 用于实现**乘法**运算 |
|BigDecimal **divide**(BigDecimal divisor)        | 用于实现**除法**运算，也可传入更多参数设置保留小数点位数和取值模式 |

``` java
BigDecimal d3 = new BigDecimal("3.0");
BigDecimal d4 = new BigDecimal("2.9");
System.out.println(d3.add(d4));//加：5.9
System.out.println(d3.subtract(d4));//减：0.1
System.out.println(d3.multiply(d4));//乘：8.70
System.out.println(d3.divide(d4, 8, BigDecimal.ROUND_HALF_UP));//除：1.03448276
```

对于divide方法，通常需要制定**精度和舍入模式**，否则当遇到无限小数时，除法会一直进行下去直至抛出异常。

#### 3.4 BigInteger
- java.math.BigInteger类处理大整数，需要import导入，继承自Number
- java提供的整数类型(int\long)的存储范围有限，当需要进行很大整数运算时可以使用java.math.BigInteger类，理论上其储值范围只受内存容量限制。 
- 如何创建对象，提供的构造方法全部都是带参数的
    * 通常利用带String参数的构造方法创建这个类的对象：BigInteger  bi = new BigInteger("123");
- 和BigDecimal类似，BigInteger也提供add()、substract()、multiply()、divide()等方法。

#### 3.5 DecimalFormat类
- 所属的包 java.text，import导入才能使用
- 通过带String参数的构造方法创建一个格式化对象(0:未满会补齐，#：未满不补）

``` java
    //调用format方法将一个小数格式化成一个字符串
DecimalFormat df = new DecimalFormat("000.000");
System.out.println(df.format(12.45)); //012.450
System.out.println(df.format(12345.6789)); //12345.679

DecimalFormat df2 = new DecimalFormat("###.###");
System.out.println(df2.format(12.45)); //12.45
System.out.println(df2.format(12345.6789)); //12345.679

DecimalFormat df3 = new DecimalFormat("000.###");
System.out.println(df3.format(12.45)); //012.45
System.out.println(df3.format(12345.6789)); //12345.679    
```


### 4. Scanner类和System类
#### 4.1 Scanner类
1. 所属的包java.util包  需要import导包
2. 通过一个带输入流的构造方法创建对象
3. 常用方法    nextInt()  nextFloat()   next()   nextLine()

#### 4.1 System类
1. 所属的包java.lang包 不需要导入
2. 不需要创建对象  通过类名就可以访问
3. 有三个属性及若干的方法
    * 三个属性out   in   err
    * 方法：gc()  exit(0);  currentTimeMillis()获取系统当前时间毫秒;



### 5. 日期类
#### 5.1 Date类
- java.util.Date类表示特定的瞬间，精确到毫秒。
- 通常使用无参数的构造方法，或者带long构造方法
- Date类中常用的方法
    * before();  after();
    * setTime()  getTime();----->long
    * compareTo();   //-1  1  0
- Date类大多数用于进行时间分量计算的方法已经被Calender取代。

``` java
Date date = new Date();//当前日期信息
    //Date类重写了toString方法，输出格式如：Sun Jan 06 11:52:55 CST 2019
long time = date.getTime();//1970年1月1日距今毫秒数。
date.setTime(time + 24\*60\*60\*1000);//通过毫秒数设置时间
```

#### 5.2 SimpleDateFormat类
- java.text.SimpleDateFormat类主要用于实现日期和文本类型之间的转换。是DateFormat(抽象类)的子类
- 其构造方法 SimpleDateFormat(String pattern)

``` java
Date date = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日");
String dateStr = sdf.format(date);
// format用于将日期按指定格式转换为字符串

String str = "2013-01-06";
SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd");
Date date2 = sdf2.parse(str);//如果字符串格式不匹配将抛出异常
```

|常用格式字符串| 含义        |  示例  |
|----------|------------|---------|
|y|年      |yyyy年——2013年；yy——13年 |
|M|月      |MM月——01月；M月——1月|
|d|日      |dd日——01日；d日——1日|
|H|小时(24)|HH:mm:ss—12:46:33|
|h|小时(12)|hh(a):mm:ss—12(下午):47:48|
|m|分钟    |--|
|s|秒      |--|

#### 5.3 Calendar类
- java.util.Calendar类是一个抽象类,主要用于取代Date类中过时的方法来描述年月日时分秒信息。
- 有构造方法，用protected修饰的，通常访问不到，通常会调用默认的getInstance();
- 通常使用Calendar的静态方法getInstance获得Calendar对象；getInstance方法将根据系统地域信息返回不同的Calendar类的实现

``` java
Calendar c1 = Calendar.getInstance();
c1.set(2008,9-1,20,8,8,8);
System.out.println(c1.getTime());
```

- 常用方法
    * after()  before()
    * setTime()  getTime()---->Date
    * getTimeInMillis()----time
    * getTimeZone()---TimeZone
    * Calendar里面包含一个date属性  可以操作date的某一个局部信息
    * set   get
        * calendar.set(Calendar.YEAR,2015);
        *int year = calendar.get(Calendar.YEAR);

- TimeZone
    1. java.util包
    2. 可以通过calendar对象.getTimeZone()获取 或 TimeZone.getDefault();
    3. 常用方法
        - tz.getID()       ---->    Asia/Shanghai
        - tz.getDisplayName()    ---->  中国标准时间



### 6. String类
#### 6.1 基本概念
- String类 ---> 引用类型  ---> java.lang包
- 没有任何继承关系，实现三个接口Serializable, CharSequence, Comparable<String>
- java.lang.String类用于描述字符串数据，java程序中所有的字符串字面值都可以使用String类的实例(对象)加以描述，如"abc"等，任何一个字符对应2字节定长编码。
- String类由final关键字修饰表示该类不能被继承，该类描述的字符串内容是常量，一旦创建无法更改，因此可以被共享。对字符串重新赋值不是改变其内容，而是改变引用的指向。

``` java
//如何构建对象
String str1 = "abc"; //直接将字符串常量赋值给str   (字符串常量池)
String str2 = new String();//无参数构造方法创建空的对象
String str3 = new String("abc");//带string参数的构造方法创建对象
byte[] bArr = {97, 98, 99, 100, 101};//a:97，b:98，c:99，d:100
String str4 = new String(bArr);//将数组中的每一个元素转化成对应的char 组合成String
char[] cArr = {'h', 'e', 'l', 'l', 'o'};
String str5 = new String(cArr);//将数组中的每一个char元素拼接成最终的String
String str6 = String(char[], index, count);//使用char数组中下标从index位置开始的count个字符来构造对象
String str7 = String(byte[], index, length);//使用byte数组下标从index位置开始length个字节来构造对象
```


#### 6.2 字符串常量池
- 由于String类型对象描述的字符串内容是个常量，若多个相同的内容单独存储会造成时间和空间的浪费。
- 出于性能考虑，Java虚拟机(JVM)将**字符串字面量对象**缓存在常量池中；对于重复出现的字符串直接量，JVM会首先在缓存池中查找，如果存在即返回该对象。

``` java
String str1 = "Hello";
String str2 = "Hello";
String str3 = new String("Hello");
System.out.println(str1.equals(str2));//比较内容 true
System.out.println(str1==str2);//比较地址 true，不会重新创建
System.out.println(str1.equals(str3));//比较内容 true
System.out.println(str1==str3);//比较地址 false，使用new会重新创建新的String对象
    //1.下面的代码中创建了几个对象并分别存放在什么位置？
String s1 = "hello"; //1个对象，常量池。
String s2 = new String("world"); //2个对象，1个在常量池，1个new后在堆区(内容为常量池里的副本)
```

#### 6.3 String类常用方法
1. 第一梯队(重写): equals  hashCode  compareTo  toString
2. 第二梯队(常用):charAt()，codePointAt()，indexOf()，lastIndexOf()，substring()，split()，replace()，length()，concat()，contains()， trim()，getBytes()， toCharArray()，matches()。
3. 第三梯队(一般):toUpperCase()，toLowerCase()，startsWith()，endsWith()，isEmpty()。

- 重写了equals(obj)，hashCode()，toString()方法，compareTo(str)方法实现自Comparable接口
    1. boolean = equals(Object obj);
        - 继承自Object类中的方法，重写后改变了规则，比较字符串中的字面值（==与equals()区别）;
    2. int = hashCode();
        - 继承自Object类中的方法，重写了：31*h+和...
    3. int = compareTo();
        - 实现自Comparable接口，实现方法：结果按照字典排布(unicode编码)顺序，按照两个字符串的长度较小的那个(次数)来进行循环，若每次的字符不一致 则直接返回code之差，若比较之后都一致  则直接返回长度之差
    4. String = toString()
        - Object类中返回类名@hashCode(16进制形式)
        - String类重写后返回的是String对象的字面值

>忽略大小写比较：equalsIgnoreCase(), compareToIgnoreCase();

|String类的成员方法         |                  |
|--------------------------|------------------|
|char charAt(int index)    |返回字符串指定位置|
|int codePointAt(int index)|"abc"0-->97，返回给定index对应位置的那个char所对应的code码|
|String concat(String)     |将给定的字符串拼接在当前字符串之后|
|int length()              |返回字符串序列的长度|
> 注意：区别数组的length是属性，String的length()是方法，集合是size()方法

``` java
String str6 = new String("hello");
System.out.println("下标为0的字符是："+str6.charAt(0));// h
System.out.println("字符串长度是："+str6.length());// 5

    //将字符串"12345"转换为整数类型
String str = new String("123456");
    //方式一：Integer类中的pareseInt方法
int ia = Integer.parseInt(str);
System.out.println("转换出来结果是："+ ia);//123456
    //方式二：利用ASCII数值进行转换'1'-'0'=1，'2'-'0'=2，...
int res = 0;
for(int i=0; i<str.length(); i++){
    res = res*10 + (str.charAt(i)-'0');
}
System.out.println("转换出来结果是："+ res);//123456
```

|String类的常用基本方法|               |
|-----------------|------------------|
|boolean contains(CharSequence s)|判断当前字符串是否包含参数指定的内容|
|String toLowerCase()|返回小写形式|
|String toUpperCase()|返回大写形式|
|String trim()|返回去掉前后空格的字符串|
|boolean startsWith(String prefix)|判断是否以参数字符开头|
|boolean endsWith(String suffix)|判断是否以参数字符结尾|
|boolean equals(Object anObject)|比较字符串内容是否相等，String类已重写|
|boolean equalsIgnoreCase(String anotherString)|同上，并且忽略大小写|
|int indexOf(String str)|返回第一次出现str位置，找不到返回-1|
|int indexOf(String str, int fromIndex)|同上，从fromIndex开始检索|
|String substring(int beginIndex, int endIndex)|截取字符串，beginIndex开始，endIndex结束|
|String substring(int beginIndex)|截取字符串，beginIndex开始到结尾|

#### 6.4 正则相关方法
- 正则表达式本质就是一个字符串，用于对用户输入数据的格式进行验证。

|正则相关方法|                  |
|-----------------|------------------|
|boolean matches(String regex)|用于判断是否匹配正则表达式规则。|
|String[] split(String regx)|以正则为分割符，将字符串拆分成字符串数组|
|String replaceAll(String regex, String replacement)|正则替换|



### 7. StringBuilder类/StringBuffer类
#### 7.1 基本概念
1. java.lang.StringBuilder类和java.lang.StringBuffer类描述的字符串内容是个可以改变的字符串序列。
2. StringBuffer和StringBuilder继承AbstractStringBuilder间接继承 Object，实现接口Serializable,CharSequence,Appendable
    - StringBuffer/StringBuilder没有compareTo方法
    - StringBuffer/StringBuilder含有一个String没有的方法 append();拼接

#### 7.2 特性
可变字符串，char[] value;  动态扩容
#### 7.3 对象的构建

``` java
    //无参数构造方法  构建一个默认长度16个空间的对象  char[]
StringBuilder builder = new StringBuilder();
    //利用给定的参数 构建一个自定义长度空间的对象 char[]
StringBuilder builder = new StringBuilder(20);
    //利用带String参数的构造方法  默认数组长度字符串长度+16个
StringBuilder builder = new StringBuilder("abc");
```


#### 7.4 StringBuilder中常用的方法
- 最主要的方法 **append()** 频繁的拼接字符串的时候使用此方法 提高性能
- ensureCapacity(int minimumCapacity)  确保底层数组容量够用
- capacity();//字符串底层char[]的容量
- length();//字符串有效元素个数(长度)
- setLength();//设置字符串的有效元素个数
- char = charAt(int index);
- int = codePointAt(int index);
- String = substring(int start [,int end]);//注意需要接受返回值 看见截取出来的新字符串效果
- StringBuilder = delete(int start [,int end]);//StringBuilder类中独有的方法String类没有，将start到end之间的字符串删掉  不用接受返回值就看到效果啦
- StringBuilder = deleteCharAt(int index);//String类中没有的方法，将给定index位置的某一个字符删除掉啦
- int = indexOf(String str [,int fromIndex]);
- int = lastIndexOf(String str [,int fromIndex]);//找寻给定的str在字符串中第一次出现的索引位置  带重载 则从某一个位置开始找
- insert(int index,value);//将给定的value插入在index位置之上
- replace(int start,int end,String str);//将start和end之间的部分替换成str, builder.replace(2,5,"zzt");
- setCharAt(int index,char value);//将index位置的字符改成给定的value
- toString();//将StringBuilder对象 构建成一个string对象 返回
- trimToSize();//将数组中无用的容量去掉  变成length长度的数组

#### 7.5 总结
1. StringBuilder类不一定需要，是为了避免String频繁拼接修改字符串信息的时候才用的，底层数组是可变的，提高了性能；
2. 常用方法
    * 与String类不同的独有方法：append()，insert()，delete()，deleteCharAt()，reverse()；
    * 与String类相同的方法：length()，charAt()，codePointAt()，indexOf()，lastIndexOf()，substring()，replace()；名字相同 用法不一致
    * 不是很常用的方法：ensureCapacity()，capacity()，setLength()，trimToSize()，setCharAt();
7. String家族笔试中经常容易考察的知识点
1. String所属的包 继承关系 实现接口
    * java.lang 继承Object 接口Serializable,CharSequence,Comparable
2. String构建方式
    * 常量  构造方法  
3. String对象内存结构
    * 字符串常量区  new堆内存对象
    * ==  equals()区别
    * "a"+"b"+"c"
4. String不可变特性
    * 长度及内容
5. String中的常用方法
    * concat();  toUpperCase();
6. String和StringBuilder区别   |   String和StringBuffer区别
    * String不可变字符串
        - JDK1.0
        - 有一个接口Comparable
        - 不可变体现在长度及内容
        - 有一些方法StringBuilder没有 concat  compareTo  toUpperCase
    * StringBuilder可变字符串
        - JDK1.5
        - 有一个接口Appendable
        - 可变字符串  没有final修饰  底层可以进行数组扩容
        - 有一些方法String没有  append() insert() delete() reverse()
7. StringBuffer和StringBuilder的不同
    * 它们方法基本相同
    * StringBuffer早期版本1.0，早期版本，线程同步，安全性比较高，执行效率相对较低
    * StringBuilder后来的版本1.5，后期版本，线程非同步，安全性比较低，执行效率相对较高



### 8. Optional类
- 可能包含或不包含非空值的容器对象。 如果一个值存在， isPresent()将返回true和get()将返回值。
- 获取字符串长度：
  1. 方式1：if(null==str){return 0;}else{return str.length();}
  2. 方式2：return Optional.ofNullable(str).map(String::length).orElse(0);

``` java
// 获取两个字符串长度和
String str1 = "zhangsan";
String str2 = null;
int str1Length = Optional.ofNullable(str1).map(String::length).orElse(0);
int str2Length = Optional.ofNullable(str2).map(String::length).orElse(0);
System.out.println(str1Length + str2Length);//8，8+0
//步骤分解:
//构建Optional对象
Optional<String> op1 = Optional.ofNullable(str1);
//将str1的长度的结果构建成Optional对象
Optional<Integer> op2 = op1.map(String::length);
//如果长度不为空，则获取长度值，否则返回默认值
int len = op2.orElse(0);
System.out.println(len);//8
```

