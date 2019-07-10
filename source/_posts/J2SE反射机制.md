---
title: J2SE反射机制
date: 2019-07-10 15:47:15
tags: [javaSE, 后端开发]
categories: javaSE知识梳理
---

##  八、反射机制
1. [基本概念](#id1)
2. [Class类](#id2)
3. [Constructor类](#id3)
4. [Field类](#id4)
5. [Method类](#id5)
6. [原始方式与反射方式构造对象实例](#id6)
7. [注解(Annotation)](#id7)
8. [Properties类的使用](#id8)


<span id="id1"><span>
### 1. 基本概念
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

反射（reflect）就是把java类中的各种成分映射成一个个的Java对象；
类是用来描述一组对象，反射机制可以理解为是用来描述一组类

通俗来讲，反射机制就是用于动态创建对象并且动态调用方法的机制；目前主流的框架底层都采用反射机制实现的。

#### 1.1 相关类及描述
- Class：用来描述类和接口；该类没有公共构造方法，由虚拟机和类加载器自动构造完成
- Package：用来描述类所属的包
- Field：用来描述类中的属性
- Method：用来描述类中的方法
- Constructor：用来描述类中的构造方法
- Annotation：用来描述类中的注解


<span id="id2"><span>
### 2. Class类
java.lang.Class：用来描述类和接口；该类没有公共构造方法，由虚拟机和类加载器自动构造完成

#### 2.1 获取Class类型对象的三种方式
``` java
Class clazz = Class.forName("包名.类名");//用的最多，但可能抛出ClassNotFoundException异常
Class clazz = 类名.class;//任何类都有一个隐含的静态成员变量class
Class clazz = 对象.getClass();//Object类中的方法
Class clazz = 包装类.TYPE;//获取对应基本数据类型的class对象
```

#### 2.2 常用方法
- static Class<?> forName(String className)
	* 用于获取参数指定对应的Class对象并返回
- T newInstance()
	* 默认调用无参数构造方法创建对象，若类中不存在无参数构造方法抛出异常NoSuchMethodException
- Constructor<T> getConstructor(Class<?>... parameterTypes)
	* 用于获取此Class对象所表示类型中参数指定的公共构造方法。
- Constructor<?>[] getConstructors()
	* 用于获取此Class对象所表示类型中所有的公共构造方法
- Field getDeclaredField(String name)
	* 用于获取此Class对象所表示类中参数指定的单个成员变量信息
- Field[] fs = getDeclaredFields()
	* 用于获取此Class对象所表示类中所有成员变量信息
- Method getMethod(String name, Class<?>... parameterTypes)
	* 用于获取该Class对象所表示类型中名字为name参数为parameterTypes的指定公共成员方法
- Method[] getMethods()
	* 用于获取该Class对象表示类中所有公共成员方法。
- 获取私有相关方法
	* getDeclaredConstructor(Class<?>... parameterTypes)；获取该类对象表示的类或接口的指定构造函数(包括私有)
	* getDeclaredConstructors()；获取该类对象所表示的类声明的所有构造函数(包括私有)
	* getDeclaredMethod(String name, Class<?>... parameterTypes) 获取一个方法(自己类 公有 私有)
	* getDeclaredMethods(); 获取全部的方法(自己类 公有 私有)


#### 2.3 其他方法
1. int result = getModifiers(); 获取类的修饰符(权限+特征)
	* 每一个修饰符 用一个整数来进行表示：0--默认不写，1--public，2--private，4--protected，-static， 16--final，32--synchronized，64volatile，128--transient，256--native，512--interface，1024--abstract
2. String name = getName(); 获取类的全名(包名.类名)
3. String name = getSimpleName(); 获取类简单名(只有类名 缺少包)
4. Package p = getPackage(); 获取当前类所属的包
	* p.getName(); 获取包名(Package类中的方法)
5. Class sclazz = getSuperClass(); 获取超类(父类)对应Class
6. Class[] classes = getInterface(); 获取当前类父亲接口
7. Class[] classes = getClasses(); 获取类中的内部类
8. Object obj = **newInstance()**; 默认调用无参数构造方法创建对象，若类中不存在无参数构造方法抛出异常NoSuchMethodException
9. Field f = getField("属性名"); 获取类中的属性(公有的 自己类+父类)
10. Field[] fs = getFields(); 获取类中的全部属性(公有的 自己类+父类)
11. getDeclaredField("属性"); 获取当前类中的属性(公有+私有 自己类)
12. Field[] fs = getDeclaredFields(); 获取当前类中全部的属性(公有+私有 自己类)


<span id="id3"><span>
### 3. Constructor类
java.lang.reflect.Constructor类主要用于描述获取到的构造方法信息

#### 3.1 Constructor类中的常用方法
- T newInstance(Object... initargs)
	* 使用此Constructor对象描述的构造方法来构造Class对象代表类型的新实例；该方法的参数用于给新实例中的成员变量进行初始化操作。

#### 3.2 其他方法
- con.getModifiers();
- con.getName();
- con.getParameterTypes();
- con.getExceptionTypes();
- 如何操作构造方法
	* 执行一次,创建对象
	* Object = newInstance(执行构造方法时的所有参数);
	* con.setAccessible(true);


<span id="id4"><span>
### 4. Field类
java.lang.reflect.Field类主要用于描述获取到的单个成员变量信息。

#### 4.1 Field类中的常用方法
- Object get(Object obj)
	* 调用该方法的意义就是获取参数对象obj中此Field对象所表示成员变量的数值。
- Object set(Object obj, Object value)
	* 将参数对象obj中此Field对象表示成员变量的数值修改为参数value的数值。
- void setAccessible(boolean flag)
	* 当实参传递true时，则反射的对象在使用时应该取消java语言访问检查


#### 4.2 其他方法
1. int = getModifiers(); 获取属性修饰符(权限+特征)
2. Class = getType(); 获取属性的类型对应的那个class
3. String = getName(); 获取属性的名字
4. 操作属性: set(对象,值); Object = get(对象);
	* 如果是私有属性不能直接操作的，需设置一个使用权setAccessable(true);准入


<span id="id5"><span>
### 5. Method类
java.lang.reflect.Method类主要用于描述获取到的单个成员方法信息。

#### 5.1 Method类中的常用方法
- Object invoke(Object obj, Object... args)
	* 使用对象obj来调用此Method对象所表示的成员方法，实参传递args。

#### 5.2 其他方法
- int mm = m.getModifiers(); 获取方法的修饰符(权限+特征)
- Class mrt = m.getReturnType(); 获取返回值数据类型
- String mn = m.getName(); 获取方法的名字
- Class[] mpts = m.getParameterTypes(); 获取方法参数列表的类型
- Class[] mets = m.getExceptionTypes(); 获取方法抛出异常的类型
- 如何操作方法
- 调用方法   让他执行一次
- Object result = invoke(对象,执行方法需要传递的所有参数...);
- 若方法是私有的方法  不允许操作
- 可以设置setAccessable(true)   设置方法使用权  准入


<span id="id6"><span>
### 6. 原始方式与反射方式构造对象实例
1. 使用原始方式来构造对象

``` java
	//1.采用无参的方式构造Person对象并打印
Person p = new Person();
System.out.println(p); //null 0
	//2.使用有参方式来构造Person对象
Person p2 = new Person("zhangfei", 30);
System.out.println(p2); //zhangfei 30
	//3.修改与获取属性(成员变量)，调用get,set方法
p2.setName("guanyu");
System.out.println("修改后的姓名是：" + p2.getName()); //guanyu
```

2. 使用反射机制来构造对象

``` java
	//1.使用获取到的Class对象来构造Person对象并打印
Class c1 = Class.forName("myproject.Person");//不可省略包名
System.out.println(c1.newInstance());//null 0
	//2.使用有参方式来构造对象
Class c2 = Class.forName("myproject.Person");
Constructor ct2 = c2.getConstructor(String.class, int.class);
Object obj = ct2.newInstance("zhangfei", 30);
System.out.println(obj);//zhangfei 30
	//3.修改与获取属性(成员变量)
Field f2 = c2.getDeclaredField("name");
f2.setAccessible(true);//暴力反射，设置使用权
f2.set(obj, "guanyu");
System.out.println("修改后的姓名是：" + f2.get(obj)); //guanyu
	//4.获取成员方法getName，使用获取到的成员方法来获取姓名并打印出来
Method m1 = c2.getMethod("getName");
System.out.println("获取到的姓名是：" + m1.invoke(obj)); //zhangfei
	//5.成员方法setName，调用getMethod方法来修改姓名并打印出来
Method m2 = c2.getMethod("setName", String.class);
Object res = m2.invoke(obj, "guanyu");
System.out.println("方法调用的返回值是：" + res); //null
System.out.println("修改后的姓名是：" + m1.invoke(obj)); //guanyu
```



<span id="id7"><span>
### 7. 注解(Annotation)
- 注释
	* 单行注释：`//`
	* 多行注释：`/*   */`
	* 文档注释：`/**   */`
- 注解的写法
	* `@XXX [(一些信息)]`
- 注解位置
	* 类的上面，属性上面，方法上面，构造方法上面，参数前面
- 注解的作用
	1. 用来充当注释的作用(仅仅是一个文字的说明)，@Deprecated
	2. 用来做代码的检测(验证)，@Override
	3. *可以携带一些信息(内容)，文件.properties/.xml，注解
- 常用的注解
	* @Deprecated：用来说明方法是废弃的
	* @Override：用来做代码检测   检测此方法是否是一个重写
	* @SuppressWarnings(String[])：{""}，如果数组内的元素只有一个长度，可以省略{}
		+ unused：变量定义后未被使用
		+ serial：类实现了序列化接口  不添加序列化ID号
		+ rawtypes：集合没有定义泛型
		+ deprecation：方法以废弃    
		+ *unchecked：出现了泛型的问题  可以不检测
		+ all：包含了以上所有(不推荐)
- 注解中可以携带信息，可以不携带；信息不能随意写，信息的类型只能是如下的类型：
	1. 基本数据类型
	2. String类型
	3. 枚举类型enum
	4. 注解类型@
	5. 数组类型[]，数组的内部需要是如上的四种类型

#### 7.1 自己描述一个注解类型
1. 通过@interface 定义一个新的注解类型
2. 发现写法与接口非常相似(可以利用接口的特点来记忆注解)
	* 可以描述public static final的属性，比较少见
	* 可以描述public abstract的方法，方法要求返回值必须有，返回值类型是如上那些
3. 我们自己定义的注解如果想要拿来使用
	* 光定义还不够，还需要做很多细致的说明(需要利用Java提供好的注解来说明)
	* 元注解(也是注解，不是拿来使用的，是用来说明注解)
		+ @Target：描述当前的这个注解可以放置在哪里写的
		+ @Retention：描述当前的这个注解存在什么作用域中的(SOURCE，CLASS，RUNTIME)，源代码文件(SOURCE)--->编译--->字节码文件(CLASS)--->加载--->内存执行(RUNTIME)
		+ @Inherited：描述当前这个注解是否能被子类对象继承
		+ @Document：描述这个注解是否能被文档所记录
4. 自己使用自己描述的注解
	* 问题1：在注解里面描述了一个方法，方法没有参数，方法是有返回值String[]
		+ 使用注解的时候   让我们传递参数
		+ 理解为  注解的方法做事   将我们传递给他的参数  搬运走了  给了别人
	* 问题2：使用别人写好的注解不用写方法名  我们自己定义的方法必须写名字
		+ 如果我们自己定义的注解  只有一个方法  方法名字叫value，在使用的时候就可以省略方法名	
		+ 如果传递的信息是一个数组   数组内只有一个元素  可以省略{}
		+ 如果方法是两个以上  每一个方法必须写名字


<span id="id8"><span>
### 8. Properties类的使用
- Java.util.Properties，主要用于读取Java的配置文件。
- Properties类继承自Hashtable
- 配置文件：在Java中，其配置文件常为.properties文件，格式为文本文件，文件的内容的格式是“键=值”的格式，文本注释信息可以用"#"来注释。
- Properties类的主要方法：
	1. getProperty ( String key)，用指定的键在此属性列表中搜索属性。也就是通过参数 key ，得到 key 所对应的 value。
	2. load ( InputStream inStream)，从输入流中读取属性列表（键和元素对）。通过对指定的文件（比如说上面的 test.properties 文件）进行装载来获取该文件中的所有键 - 值对。以供 getProperty ( String key) 来搜索。
	3. setProperty ( String key, String value) ，调用 Hashtable 的方法 put 。他通过调用基类的put方法来设置 键 - 值对。
	4. store ( OutputStream out, String comments)，以适合使用 load 方法加载到 Properties 表中的格式，将此 Properties 表中的属性列表（键和元素对）写入输出流。与 load 方法相反，该方法将键 - 值对写入到指定的文件中去。
	5. clear ()，清除所有装载的 键 - 值对。该方法在基类中提供。