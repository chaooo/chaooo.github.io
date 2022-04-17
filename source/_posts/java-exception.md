---
title: 「Java教程」异常处理机制
date: 2017-04-25 08:34:31
tags: [Java, 后端开发]
categories: Java教程
---


Java语言提供了完善的异常处理机制。正确运用这套机制，有助于提高程序的健壮性。
<!-- more -->

### 1. 基本概念
- 异常用于在Java语言中描述运行阶段发生的错误。
- 在Java中有一个定义好的规则Throwable（可以抛出的）
- java.lang.Throwable类是所有错误(Error)和异常(Exception)的超类。
    * Error类主要用于描述比较严重无法编码解决的错误，如：JVM内存资源耗尽等。
    * Exception类主要用于描述比较轻微可以编码解决的错误，如：文件损坏、非法输入等。
- java.lang.Exception类是所有异常的超类，主要分为两大类：
    * RuntimeException - 运行时异常，也叫非检测性异常
    * IOException和其他异常 - 其他异常也叫做检测性异常

> 注意：当程序运行过程中发生异常而又没有手动处理时，则由java虚拟机采用默认方式处理，即打印异常名称、原因、发生位置并终止程序。在开发中尽量使用条件判断避免异常的发生。

```
Throwable类
    |————Exception类
        |————RuntimeException异常
            |————ArithmeticException类
            |————ArrayIndexOutOfBoundsException类
            |————NullPointerException类
            |————ClassCastException类
            |————NumberFormatException类
        |————IOException和其他异常
    |————Error类
```


### 2. 异常的分支结构
#### 2.1 运行时异常（非检查异常）
1. Error和RuntimeException都算作运行时异常
2. javac编译的时候，不会提示和发现的，
3. 在程序编写时不要求必须做处理，如果我们愿意可以添加处理手段(try throws)
4. 要求大家出现这样异常的时候 知道怎么产生及如何修改
    + InputMisMatchException 输入不匹配
        - int value = input.nextInt();//   abc
    + *NumberFormatException 数字格式化
        - int value = Integer.parseInt("123.45");
    + NegativeArraySizeException 数组长度负数
        - int[] array = new int[-2];
    + *ArrayIndexOutOfBoundsException 数组索引越界
        - int[] array = {1,2,3};
        - array[5];
    + *5NullPointerException 空指针异常
        - int[][] array = new int[3][];
        - array[0][0] =10;
        - Person p = null;
        - p.getName();
    + ArithmeticException 数字异常
        - 10/0    整数不允许除以0    Infinity小数除以0会产生无穷
    + *ClassCastException 造型异常
        - Person p = new Teacher();
        - Student s = (Student)p;
    + *StringIndexOutOfBoundsException 字符串越界
        - String str = "abc";
        - str.charAt(5);
    + *IndexOutOfBoundsException 集合越界
        - List家族
        - ArrayList  list = new ArrayList();
        - list.add(); list.add(); list.add();
        - list.get(5);
    + IllegalArgumentException 非法参数异常
        - ArrayList  list = new ArrayList(-1);

#### 2.2 编译时异常(检查异常)
- 除了Error和RuntimeException以外其他的异常
- javac编译的时候，强制要求我们必须为这样的异常做处理(try或throws)
- 因为这样的异常在程序运行过程中极有可能产生问题的
- 异常产生后后续的所有执行就停止

``` java
//eg: InterruptException
try{
    Thread.sleep(5000);
}catch(Exception e){
    //...
}
```


### 3. 添加处理异常的手段
- 处理异常不是 异常消失了
- 处理异常指的是：处理掉异常之后，后续的代码不会因为此异常而终止执行
- 两种手段：
    * 异常的捕获：try{}catch(){}[ finally{} ]
    * throws抛出
- final，finally，finalize区别
    * final：特征修饰符，修饰变量，属性，方法，类
        + 修饰变量：基本类型:值不能改变；引用类型:地址不能改变(如果变量没有初值,给一次机会赋值)
        + 修饰属性：特点与修饰变量类似(要求必须给属性赋初始值,否则编译报错)
        + 修饰方法：不能被子类重写
        + 修饰类：不能被其他的子类继承
    * finally：处理异常手段的一部分
        + try{}catch(){}后面的一个部分
        + 这个部分可有可无，如果有只能含有一份，且必须执行
    * finalize：是Object类中的一个protected方法
        + 对象没有任何引用指向的时候 -- 会被GC回收
        + 当对象回收的时候 默认调用finalize方法
        + 若想要看到对象回收的效果，可以重写 public void finalize(){}


### 4. 异常的捕获
``` java
try{
    可能发生异常的代码;
}catch(异常类型 引用变量){
    针对该异常的处理代码;
}catch ...
finally{
    无论是否发生异常都要执行的代码;
}
```

- 处理异常放在方法内部 可能会出现的小问题
    * 如果在方法内部含有返回值，不管返回值return关键字在哪里，finally一定会执行完毕，返回值的具体结果得看情况。

```
public String test() {
    try {
        //...可能产生异常的的代码
        return "值1";//事先约定好 返回值
    }catch(Exception e){
        e.printStackTrace();//打印输出异常的名字
    }finally {
        System.out.println("finally块执行啦");
    }
    return "值2";
}
```

- 上述执行结果：若try中代码块产生异常return返回 **值2**，若try中无异常则return返回 **值1**，无论return在哪finally都会执行。

> 异常捕获的注意事项：
> - 当需要多分catch分子时，切记小类型应该放在大类型的前面；
> - 懒人写法：catch(Exception e){...}
> - finally通常用于善后处理，如：关闭已经打开的文件等。


### 5. 异常的抛出
- 当程序中发生异常又不方便直接处理时，可以将异常转移给方法调用者进行处理，这个过程叫做异常的抛出。
- 语法格式：访问权限 返回值类型 方法名(形参列表) throws 异常类型1,异常类型2,...{} ，
<br>如：`public void show() throw Exception {}`
- 重写方法的抛出规则：
    * 不抛出异常
    * 抛出父类异常中的子类异常
    * 抛出和父类一样的异常
    * 不能抛出同级不一样的异常
    * 不能抛出更大的异常


### 6. 自定义异常
- 可以根据需要自定义异常类。
- 自定义异常的方式：
    * 继承Exception或者异常的子类。
    * 提供两个构造，无参构造和String做参数的构造。 
- 异常的手段
    * 如果继承是RuntimeException---->运行时异常(不需要必须添加处理手段)
    * 如果继承是Exception----->编译时异常(必须添加处理手段)
- 类中可以写带String参数的构造方法，可以做细致的说明
- 通过throw关键字，new一个异常的对象
- 主动产生异常：`throw new 异常类型();`


### 7. 总结
- 1.在开发中尽量使用条件判断避免异常的发生;
- 2.若实在避免不了，则进行异常捕获；
- 3.若实在捕获不了，则进行异常抛出；
- 4.若需要使用针对性异常，则自定义异常。

