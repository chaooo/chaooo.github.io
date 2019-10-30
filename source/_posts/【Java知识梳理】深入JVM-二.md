---
title: 【Java知识梳理】深入JVM(二)-类文件结构 与 类加载机制
date: 2019-08-25 19:29:48
tags: [java, 后端开发]
categories: Java知识梳理
---


### 1. 类文件结构
Class文件是一组以 **8 位字节**为基础单位的二进制流，各个数据**严格按照顺序紧凑的排列**在 Class 文件中，中间无任何分隔符，这使得整个 Class 文件中存储的内容几乎全部都是程序运行的必要数据，没有空隙存在。当遇到需要占用 8 位字节以上空间的数据项时，会按照高位在前的方式分割成若干个 8 位字节进行存储。
Java 虚拟机规范规定 Class 文件格式采用一种类似与 C 语言结构体的伪结构体来存储数据，这种伪结构体中只有两种数据类型：**无符号数**和**表**。<!-- more -->
- **无符号数**：属于基本数据类型，以u1、u2、u4、u8来代表1个字节、2个字节、4个字节、8个字节的无符号数， 无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
- **表**：由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以「_info」结尾。表用于描述有层次关系的复合结构的数据，整个 Class 文件就是一张表。

根据 Java 虚拟机规范，类文件由单个 ClassFile 结构组成：
``` java
ClassFile {
    u4             magic;                //Class文件的标志(魔数)
    u2             minor_version;        //Class的小版本号
    u2             major_version;        //Class的大版本号
    u2             constant_pool_count;  //常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;         //Class的访问标记
    u2             this_class;           //当前类
    u2             super_class;          //父类
    u2             interfaces_count;     //接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;          //Class文件的字段属性
    field_info     fields[fields_count];  //一个类会可以有个字段
    u2             methods_count;         //Class文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;      //此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```

![](http://cdn.chaooo.top/Java/JVM-class.png)


#### 1.1 魔数 (Magic Number)
- Class文件的**0~3字节**(前四个字节: ca fe ba be)
- 作用: 确定这个文件是否为一个能被虚拟机接收的Class文件


#### 1.2 Class文件版本
- **4~7字节**, 其中4~5次版本号,6~7主版本号(如jdk8主版本号是: 00 34)


#### 1.3 常量池
- **8~9字节**表示16进制常量池数量,其后紧跟具体常量池, 常量池的数量是 `constant_pool_count-1`（**常量池计数器是从1开始计数的，将第0项常量空出来是有特殊考虑的，索引值为0代表“不引用任何一个常量池项”**）
- 常量池主要存放两大常量: **字面量**和**符号引用**
    + 字面量: Java语言层面的常量概念(String,final等)
    + 符号引用: 编译原理方面的概念(类和接口的全限定名\字段的名称和描述符\方法的名称和描述符)
- 常量池中每一项常量都是一个表，这**14种表**有一个共同的特点：**开始的第一位是一个 u1 类型的标志位 -tag 来标识常量的类型，代表当前这个常量属于哪种常量类型**
- .class文件可以通过`javap -v class类名` 指令来看一下其常量池中的信息(`javap -v class类名-> temp.txt` ：将结果输出到 temp.txt 文件)


#### 1.4 类的访问标志与继承信息
- 在常量池结束之后，紧接着的**两个字节**代表访问标志，这个标志用于识别一些类或者接口层次的访问信息，包括：这个 Class 是类还是接口，是否为 public 或者 abstract 类型，如果是类的话是否声明为 final 等等.
- access_flags中一共有16个标志位可以使用，当前只定义了其中的8个，没有使用到的标志位要求一律为0。


#### 1.5 当前类索引(`this`),父类索引(`super`)与接口索引集合(`interfaces`)
- 类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于 Java 语言的单继承，所以父类索引只有一个，除了 java.lang.Object 之外，所有的 java 类都有父类，因此除了 java.lang.Object 外，所有 Java 类的父类索引都不为 0。
- 接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按implents(如果这个类本身是接口的话则是extends) 后的接口顺序从左到右排列在接口索引集合中。


#### 1.6 成员变量信息(Feild)
- 字段表（field info）用于描述接口或类中声明的变量。字段包括类级变量以及实例变量，但不包括在方法内部声明的局部变量。
- 字段信息包括：字段的作用域（public、private、protected修饰符）、是实例变量还是类变量（static修饰符）、可变性（final）、并发可见性（volatile修饰符，是否强制从主内存读写）、可否被序列化（transient修饰符）、字段数据类型（基本类型、对象、数组）、字段名称，以上修饰符都是布尔类型。
- 方法和字段的描述符作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。
- 根描述规则，基本数据类型（byte、char、double、float、int、long、short、boolean）以及代表无返回值的void类型都用一个大写字符来表示，对象类型使用字符L加对象的全限定名来表示。
    + B: 基本类型byte
    + C: 基本类型char
    + D: 基本类型double
    + F: 基本类型float
    + I: 基本类型
    + J: 基本类型long
    + S: 基本类型short
    + Z: 基本类型boolean
    + V: 特殊类型void
    + L: 对象类型，如Ljava/lang/Object


#### 1.7 方法信息(Method)
- `methods_count` 表示方法的数量，而 `method_info` 表示的方法表。
- Class 文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式。方法表的结构如同字段表一样，依次包括了访问标志、名称索引、描述符索引、属性表集合几项。


#### 1.8 附加属性信息
- `attributes_count`表示属性表中的属性个数, `attribute_info` 表示属性表
- 在 Class 文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与 Class 文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写 入自己定义的属性信息，Java 虚拟机运行时会忽略掉它不认识的属性。



### 2. 字节码指令
Java字节码指令就是Java虚拟机能够识别、可执行的指令，可以说是Jvm的最小执行单元。javac命令会将Java源文件编译成字节码文件，即.class文件，其中就包含了大量的字节码指令，javap命令可以解析字节码(.class文件)，将字节码内部逻辑以可读的方式呈现出来 (`javap -v -p HelloWorld`)。
+ 按指令的功能分为如下几类：
    1. **存储和加载类指令**：主要包括load系列(将一个局部变量加载到操作数栈)、store系列(将一个数值从操作数栈存储到局部变量表)和ldc/push/const系列(将一个常量加载到操作数栈)，主要用于在**局部变量表**、**操作数栈**和**常量池**三者之间进行**数据调度**；
        + 例如: `iload_0`表示从当前栈帧局部变量表中0号位置取int类型的数值加载到操作数栈
    2. **对象操作指令**（创建与读写访问）：比如我们刚刚的putfield和getfield就属于读写访问的指令，此外还有putstatic/getstatic，还有new系列指令，以及instanceof等指令。
    3. **操作数栈管理指令**：如pop和dup，他们只对操作数栈进行操作。
    4. 类型转换指令和运算指令：如add(加)/sub(减)/mul(乘)/div(除)/l2i/d2f等系列指令，实际上这类指令一般也只对操作数栈进行操作。
    5. 控制跳转指令：这类里包含常用的if系列指令以及goto类指令。
    6. **方法调用和返回指令**：主要包括invoke系列指令和return系列指令。这类指令也意味这一个方法空间的开辟和结束，即invoke会唤醒一个新的java方法小宇宙（新的栈和局部变量表），而return则意味着这个宇宙的结束回收。
+ 从指令操作的数据类型来讲：指令开头或尾部的一些字母，就往往表明了它所能操作的数据类型：
    - a对应对象，表示指令操作对象性数据，比如aload和astore、areturn等等。
    - i对应整形。也就有iload，istore等i系列指令。
    - f对应浮点型，l对应long，b对应byte，d对应double，c对应char。
    - ia对应int array，aa对应object array，da对应double array。


### 3. 编译期处理(语法糖)
**语法糖**: 指Java编译器把.java源码编译为.class字节码过程中,自动生成和转换的一些代码. 如:默认构造器,自动拆装箱等.
1. **默认构造器**: `public class Candy{}` 编译后为: `public class Candy{public Candy(){super();}}` 
2. **自动拆装箱**: `Integer x=1;int y=x;` 编译后为: `Integer x=Integer.valueOf(1);int y=x.intValue();`
3. **泛型擦除**: 擦除的是字节码上的泛型信息.
4. **泛型反射**: 通过反射获得泛型信息
5. **可变参数**: `String... args` 可以是一个`String[] args`
6. **foreach**: 集合相当于获取迭代器Iterator
7. **switch**: Jdk7开始可以配合String和枚举
    + switch-String: 执行了两遍switch,第一遍根据字符串的hashCode和equals将字符串转换为相应的byte类型,第二遍利用byte执行比较.
    + switch-枚举: 会为当前类生成一个静态内部类(合成类,仅JVM使用,对我们不可见),用来映射枚举类的枚举编号(从0开始)与数组元素的关系,数组大小即为枚举元素的个数,里面存储case用来对比的数字,根据这个数字执行switch
8. **枚举类**: 继承Enum并且用final修饰类,构造方法私有,枚举量被编译成本类的final类变量,定义私有静态枚举量数组$VALUES,静态方法values()用来返回定义的枚举量数组的clone(),静态方法valueOf()调用父类valueOf(本类.class,名称)根据类型和名称得到相应实例
9. **try-with-resources**: 无论try块的异常还是关闭资源时的异常都不会丢。可以在 try-with-resources 语句中同时处理多个资源。
    + 在 Java 7/8 ，try-with-resources 语句中必须声明要关闭的资源。通过这种方式声明的资源属于隐式 final。
    + Java 9 中甚至能使用预先创建的资源，只要所引用的资源声明为 final 或者是 effective final。
    + 在幕后施展魔法的是 AutoCloseable 或者 Closeable 接口，它们与 try-with-resources 语句协同工作。
10. **重写桥接**: 子类重写方法返回值可以是父类返回值的子类,JVM内部使用了桥接方法(synthetic bridge修饰)重写父类方法并返回子类重写的同名方法,并且没有命名冲突,仅对jvm可见.
11. **匿名内部类**: 内部创建了final修饰的实现类, 匿名内部类引用局部变量时,局部变量必须是final的:因为内部创建实现类时,将值赋给其对象的valx属性,valx属性没有机会再跟着一起变化.



### 4. 类加载阶段
1. 隐式加载：new
2. 显式加载：loadClass、forName等(需要调用Class的newInstance方法获取实例)
3. 类的装载阶段：**`加载 --> 链接 --> 初始化`**
    + 加载：通过Classloader加载class文件字节码，生成class对象
    + 链接：校验-->准备-->解析
        - 校验：检查加载的Class的正确性和安全性
        - 准备：为变量分配存储空间并设置类变量初始值
        - 解析：JVM将常量池内的符号引用转换为直接引用
    + 初始化：执行类变量赋值和静态代码块

#### 4.1 加载
- 将类的字节码载入方法区中,内部采用C++的instanceKlass描述java类, 它的重要field有:
    + `_java_mirror`:Java类的镜像, `_super`:父类, `_field`:成员变量, `_methods`:方法, `_constants`:常量池, `_class_loader`:类加载器, `_vtable`:虚方法表, `_itable`:接口方法表
- 如果这个类还有父类没加载,先加载父类
- **加载和链接可能是交替运行的**

> `instanceKlass`这样的元数据是存储在方法区(元空间),但`_java_mirror`存储在堆中; 可通过HSDB工具查看.

#### 4.2 链接
1. 验证: 验证类是否符合JVM规范,安全性检查
2. 准备: 为static变量分配空间,设置默认值
    + jdk7开始, static变量存储于`_java_mirror`末尾, jdk7之前是instanceKlass末尾.
    + static变量分配空间和赋值是两个步骤, 分配空间在准备阶段完成,赋值在初始化阶段完成
    + 如果static变量是final的**基本类型或字符串常量**,那么编译阶段值就确定了,赋值在准备阶段完成
    + 如果static变量是final的**引用类型**,那么赋值还是会在初始化阶段完成
3. 解析: 将常量池中的符号引用解析为直接引用(确切知道类,方法,属性在内存中的位置)

#### 4.3 初始化
- 初始化即调用`<cinit>()V`方法,虚拟机会**保证**这个类的[**构造方法**]**线程安全**
- **发生的时机**: 概括的说,类初始化是[**懒惰的**]
    + main方法所在的类的,总会被首先初始化
    + 首次访问这个类的静态变量或静态方法时
    + 子类初始化, 如果父类没有初始化,会引发
    + 子类访问父类静态变量, 只会触发父类的初始化
    + Class.forName 和 new操作 导致初始化
- **不会**导致类初始化的情况
    + 访问类的static final 静态常量(基本类型和字符串常量)**不会**触发初始化
    + 类对象.class 不会
    + 创建该类的数组 不会
    + 类加载器的loadClass方法 不会
    + Class.forName的第二个参数为false时 不会

#### 4.4 应用实例-懒惰初始化单例模式(线程安全)
``` java
class Singleton{
    private Singleton(){}
    // 内部类中保存单例
    private static class LazyHolder{
        private static final Singleton SINGLETON = new Singleton();
    }
    // 第一次调用getInstance,才会导致内部类加载和初始化其静态成员
    public static Singleton getInstance(){
        return LazyHolder.SINGLETON;
    }
}
```


### 5. 类加载器
以JDK8为例:

| 名称 | 加载哪的类 | 说明 |
|------|----------|------|
|Bootstrap ClassLoader  |JAVA_HOME/jre/lib    |启动类加载器, 最顶层, 打印显示为null|
|Extension ClassLoader  |JAVA_HOME/jre/lib/ext|扩展类加载器, 第二级, 打印显示为$ExtClassLoader|
|Application ClassLoader|classpath            |应用程序类加载器, 第三级, 打印显示为$AppClassLoader|
|自定义类加载器           |自定义               |上级为Application|

#### 5.1 类加载器-双亲委派机制
- 类加载器在接到加载类的请求时，首先将加载任务**委托给上级加载器**，依次递归，如果上级加载器可以完成类加载任务，就成功返回；只有上级加载器无法完成此加载任务时，才自己去加载。
- 这种双亲委派模式的好处，一个可以避免类的重复加载，另外也避免了java的核心API被篡改。

``` java
/**
 * loadClass方法的实现方式
 */
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        //【1】 检查该类是否已经加载
        Class c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //【2】 有上级的话,委派上级 loadClass
                    c = parent.loadClass(name, false);
                } else {
                    //【3】 如果没有上级了(ExtClassLoader),则委派BootstrapClassLoader
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order to find the class.
                long t1 = System.nanoTime();
                //【4】 每一级都找不到,调用findClass(每个类加载器自己扩展)来加载
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```



#### 5.2 线程上下文类加载器
- Java 提供了很多服务提供者接口(Service Provider Interface，SPI),允许第三方为这些接口提供实现(常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等)。
- SPI接口中的代码经常需要加载具体的实现类; SPI的接口由Java核心库来提供，实现类可能是作为Java应用所依赖的jar包被包含进来，可以通过类路径（CLASSPATH）来找到。
- SPI的接口是Java核心库的一部分，是由引导类加载器来加载的；引导类加载器是无法找到SPI的实现类的,这时候需要抛弃双亲委派加载链模式，使用线程上下文里的类加载器加载类。
- 类 java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。
- Java默认的 线程上下文类加载器 是 应用程序类加载器(AppClassLoader)。


#### 5.3 何时使用Thread.getContextClassLoader()?
- 总的说来动态加载资源时，一般只有两种选择，当前类加载器和线程上下文类加载器。当前类加载器是指当前方法所在类的加载器。这个类加载器是运行时类解析使用的加载器，Class.forName(String)和Class.getResource(String)也使用该类加载器。代码中X.class的写法使用的类加载器也是这个类加载器。
- 该如何选择类加载器？
    + 如若代码是限于某些特定框架，这些框架有着特定加载规则，则不要做任何改动，让框架开发者来保证其工作（比如应用服务器提供商，尽管他们并不能总是做对）。如在Web应用和EJB中，要使用Class.gerResource来加载资源。
    + 在其他情况下，我们可以自己来选择最合适的类加载器。可以使用策略模式来设计选择机制。其思想是将“总是使用上下文类加载器”或者“总是使用当前类加载器”的决策同具体实现逻辑分离开。往往设计之初是很难预测何种类加载策略是合适的，该设计能够让你可以后来修改类加载策略。
    + 一般来说，上下文类加载器要比当前类加载器更适合于框架编程，而当前类加载器则更适合于业务逻辑编程。


#### 5.4 类加载器与Web容器
以 Apache Tomcat 来说，每个 Web 应用都有一个对应的类加载器实例。**该类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是使得 Web 应用自己的类的优先级高于 Web 容器提供的类**。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。
- 绝大多数情况下，Web 应用的开发人员不需要考虑与类加载器相关的细节。下面给出几条简单的原则：
    1. 每个 Web 应用自己的 Java 类文件和使用的库的 jar 包，分别放在 WEB-INF/classes和 WEB-INF/lib目录下面。
    2. 多个应用共享的 Java 类文件和 jar 包，分别放在 Web 容器指定的由所有 Web 应用共享的目录下面。
    3. 当出现找不到类的错误时，检查当前类的类加载器和当前线程的上下文类加载器是否正确。


#### 5.5 自定义类加载器
1. 什么时候需要自定义类加载器
    + 加载非classpath路径的任意路径类文件
    + 都是通过接口来使用实现,希望解耦时,常用于框架设计
    + 这些类希望予以隔离,不同应用的同名类都可以加载,不冲突,常见于tomcat容器
2. 如何自定义类加载器
    1. 继承ClassLoader类
    2. 重写findClass(String className)方法
    3. 读取(加载)类文件的字节码。
    4. 调用ClassLoader超类的defineClass方法，向虚拟机提供字节码。
    5. 使用者调用该自定义类加载器的loadClass方法

``` java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
public class MyClassLoader extends ClassLoader {
    /**
     * @param name 类名称
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            String cname = "E:\\myclasspath\\" + name.replace('.', '/') + ".class";
            byte[] classBytes = Files.readAllBytes(Paths.get(cname));
            Class<?> cl = defineClass(name, classBytes, 0, classBytes.length);
            if (cl == null) {
                throw new ClassNotFoundException(name);
            }
            return cl;
        } catch (IOException e) {
            System.out.print(e);
            throw new ClassNotFoundException(name);
        }
    }
}
```



### 6. 运行期JVM自动优化
Java程序最初是通过解释器进行解释执行的，当程序需要迅速启动和执行时，解释器可以首先发挥作用，省去编译时间，立即执行；当程序运行后，随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码，获得更高的执行效率。**解释执行节约内存，编译执行提升效率**。 同时，解释器可以作为编译器激进优化时的一个“逃生门”，让编译器根据概率选择一些大多数时候都能提升运行速度的优化手段，当激进优化的假设不成立，则通过逆优化退回到解释状态继续执行。
HotSpot虚拟机中内置了两个即时编译器，分别称为**Client Compiler(C1编译器)**和**Server Compiler(C2编译器)**，默认采用Interpreter(解释器)与其中一个编译器直接配合的方式工作，使用哪个编译器取决于虚拟机运行的模式，也可以自己去指定。
+ 分层编译策略, JVM将执行状态分成了5个层次:
    1. 0层, 解释执行
    2. 1层, 使用C1即时编译器编译执行(不带profiling)
    3. 2层, 使用C1即时编译器编译执行(带基本的profiling)
    4. 3层, 使用C1即时编译器编译执行(带完全的profiling)
    5. 4层, 使用C2即时编译器编译执行
> profiling是指在运行过程中收集一些程序执行状态的数据,例如[方法的调用次数],[循环的回边次数]等

+ 即时编译器(JIT)与解释器的区别
    - 解释器是将字节码解释为机器码,下次即便遇到相同的字节码,仍会执行重复的解释
    - JIT是将一些字节码编译为机器码并存入CodeCache,下次遇到相同的代码,直接执行,无需再编译
    - 解释器是将字节码解释为针对所有平台都通用的机器码
    - JIT会根据平台类型,生成平台特定的机器码

+ 对于占据大部分的不常用的代码,我们无需耗费时间将其编译成机器码,而是采用解释执行的方式运行;另一方面,对于占据小部分的热点代码,我们则可以将其编译成机器码,以达到理想的运行速度;
+ 执行效率: `Interpreter < C1 < C2`, 总的目标是发现热点代码(hotpot名称的由来)优化之.

#### 6.1 公共子表达式消除
如果一个表达式E已经计算过了，并且先前的计算到现在E中所有变量的值都没有发生变化，那么E的这次出现就成为了公共表达式，可以直接用之前的结果替换。
例：int d = (c * b) * 12 + a + (a + b * c) => int d = E * 12 + a + (a + E)

#### 6.2 数组边界检查消除
Java语言中访问数组元素都要进行上下界的范围检查，每次读写都有一次条件判定操作，这无疑是一种负担。编译器只要通过数据流分析就可以判定循环变量的取值范围永远在数组长度以内，那么整个循环中就可以把上下界检查消除，这样可以省很多次的条件判断操作。

#### 6.3 方法内联
方法内联能去除方法调用的成本，同时也为其他优化建立了良好的基础，因此各种编译器一般会把内联优化放在优化序列的最靠前位置，然而由于Java对象的方法默认都是虚方法，在编译期无法确定方法版本，就无法内联。
- 因此方法调用都需要在运行时进行多态选择，为了解决虚方法的内联问题，Java虚拟机团队引入了“类型继承关系分析(CHA)”的技术。
    1. 在内联时，若是非虚方法，则可以直接内联  
    2. 遇到虚方法，首先根据CHA判断此方法是否有多个目标版本，若只有一个，可以直接内联，但是需要预留一个“逃生门”，称为守护内联，若在程序的后续执行过程中，加载了导致继承关系发生变化的新类，就需要抛弃已经编译的代码，退回到解释状态执行，或者重新编译。
    3. 若CHA判断此方法有多个目标版本，则编译器会使用“内联缓存”，第一次调用缓存记录下方法接收者的版本信息，并且每次调用都比较版本，若一致则可以一直使用，若不一致则取消内联，查找虚方法表进行方法分派。

#### 6.4 逃逸分析
分析对象动态作用域，当一个方法被定以后，它可能被外部方法所引用，称为方法逃逸，甚至还有可能被外部线程访问到，称为线程逃逸。
+ 若能证明一个对象不会逃逸到方法或线程之外，这可以通过栈上分配、同步消除、标量替换来进行优化。
    1. 栈上分配：如果确定一个对象不会逃逸，则可以让它分配在栈上，对象所占用的内存空间就可以随栈帧出栈而销毁。这样可以减小垃圾收集系统的压力。  
    2. 同步消除：线程同步相对耗时，如果确定一个变量不会逃逸出线程，那这个变量的读写不会有竞争，则对这个变量实施的同步措施也就可以消除掉。  
    3. 标量替换：如果逃逸分析证明一个对象不会被外部访问，并且这个对象可以被拆散的话，那么程序真正执行的时候可以不创建这个对象，改为直接创建它的成员变量，这样就可以在栈上分配。


### 7. 反射机制
简单说，反射机制是程序在运行时能够获取自身的信息。在java中，只要给定类的名字，那么就可以通过反射机制来获得类的所有信息。
Class反射对象描述的是类的语义结构，通过class对象，可以获取构造器，成员变量，方法等类元素的反射对象，并且可以用编程的方法通过这些反射对象对目标对象进行操作。
这些反射类在java.lang.reflect包中定义，下面是最主要的三个类：
1. Constructor：类的构造函数反射类：
    + 通过Class#getConstructors()方法可以获得类的所有构造函数的反射对象数组。
    + 其中最主要的方法是newInstance(Object[] args),通过该方法可以创建一个对象类的实例，功能和new一样。在jdk5.0之后，提供了newInstance(Object...args)更为灵活。
2. Method：类方法的反射类。
    + 通过Class#getDeclaredMethods()方法可以获取所有方法的反射类对象数组Method[].其中最主要的方法是:
    + invoke(String name,class parameterTypes),和invoke(Object obj,Object...args)。同时也还有很多其他方法
    + Class getReturnType（）：获取方法的返回值类型
    + Class[] getParameterTypes（）：获取方法的参数数组
3. Field：类成员变量的反射类，
    + 通过Class#getDeclareFields（）可以获取类成员变量反射的数组。
    + Class#getDeclareField（String  name）获取某特定名称的反射对象。
    + 最主要的方法是：set(Object obj,Object value),为目标对象的成员变量赋值。如果是基础类型还可以这样赋值setInt(),setString()...

+ java还提供了包的反射类和注解的反射类。
+ 总结:java反射体系保证了通过程序化的方式访问目标对象的所有元素，对于private 和protected成员变量或者方法，也是可以访问的。


#### 7.1 反射中，Class.forName和classloader的区别
+ Class.forName()得到的Class是完成初始化的
+ 而ClassLoader.loadClass()得到的Class是还没有链接的。
+ Spring IoC为了加快初始化速度，因此大量使用了延时加载技术。而使用classloader不需要执行类中的初始化代码，可以加快加载速度，把类的初始化工作留到实际使用到这个类的时候。

#### 7.2 哪里用到反射机制？
+ JDBC中，利用反射动态加载了数据库驱动程序。
+ Web服务器中利用反射调用了Sevlet的服务方法。
+ Eclispe等开发工具利用反射动态刨析对象的类型与结构，动态提示对象的属性和方法。
+ 很多框架都用到反射机制，注入属性，调用方法，如Spring。

#### 7.3 反射机制的优缺点？
优点：可以动态执行，在运行期间根据业务功能动态执行方法、访问属性，最大限度发挥了java的灵活性。
缺点：对性能有影响，这类操作总是慢于直接执行java代码。


