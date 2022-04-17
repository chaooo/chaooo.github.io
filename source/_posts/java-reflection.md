---
title: 「Java教程」反射机制
date: 2017-05-18 15:47:15
tags: [Java, 后端开发]
categories: Java教程
---

反射(Reflection)是Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。
多数情况下反射是为了提高程序的灵活性，运行时动态加载需要加载的对象。
<!-- more -->


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


### 7. 注解(Annotation)
#### 7.1 注解相关概念
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
- 注解的分类
    1. 按运行机制分：源码注解，编译时注解，运行时注解
    2. 按照来源分：来自JDK的注解，来自第三方的注解，自定义注解
    

#### 7.2 自定义注解类型的语法要求：
1. 使用@interface关键字定义注解
2. 成员以**无参无异常**方式声明
3. 可以用default为成员指定一个默认值
4. 成员类型是受限的，合法类型包括原始类型及String,Class,Annotation,Enumeration
5. 如果注解只有一个成员，则成员名必须取名**value()**,在使用时可以忽略成员名和赋值号(=)
6. 注解类可以没有成员，没有成员的注解称为标识注解
7. 需要元注解来描述说明
    + @Target：当前注解的放置(CONSTRUCTOR，FIELD，LOCAL_VARIABLE，METHOD，PACKAGE，PARAMETER，TYPE)
    + @Retention：当前注解的生命周期作用域(SOURCE，CLASS，RUNTIME)，源代码文件(SOURCE)--->编译--->字节码文件(CLASS)--->加载--->内存执行(RUNTIME)
    + @Inherited：允许子类继承
    + @Document：当前注解是否能被文档(javadoc)所记录

``` java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Description{
    String desc();
    String author();
    int age() default 18;
}
```

#### 7.3 使用自定义注解：
* @<注解名>(<成员名1>=<成员值1>,<成员名2>=<成员值2>,...)

``` java
@Description(desc="I am eyeColor", author="Chao", age=18)
public String eyeColor(){
    return "red";
}
```

> 如果自定义注解只有一个value成员，在使用的时候就可以省略方法名，如果方法是两个以上，每一个方法必须写名字

``` java
@Description("I am class annotation")
public class Child implements Person{
    @Override
    @Description("I am method annotation")
    public String name(){
        return null;
    }
    @Override
    public int age(){
        return 0;
    }
    @Override
    public void sing(){ }
}
```

#### 7.4 解析注解
通过反射获取类、函数或成员上的运行时注解信息，从而实现动态控制程序运行的逻辑。

1. 使用类加载器加载类
    * `Class c=Class.forName（"com.ann.test.Child")`
2. 找到类上面的注解
    * `isAnnotationPresent（类类型）`：Class对象的方法，判断当前类类型是否存在某个类类型的注解，返回类型为boolean。
3. 拿到注解实例，需要强制类型转换。
    * `Description d=（Description）c.getAnnotation(Description.class);`
4. 找到方法上的注解，首先，遍历所有方法，通过方法对象的isAnnotation查看是否有自定义注解。

``` java
public class ParseAnn{
  public static void main(String[]){
    try{//1. 使用类加载器加载类
      Class c=Class.forName（"com.ann.test.Child")
      //2. 找到类上面的注解
      boolean isExist = c.isAnnotationPresent(Description.class);
      if(isExist){
        //3. 拿到注解实例
        Description d=（Description）c.getAnnotation(Description.class);
        System.out.println(d.value());
      }
      //4.找到方法上的注解
      Method[] ms = c.getMethods();
      for(Method m:ms){
        boolean isMExist = m.isAnnotationPresent(Description.class);
        if(isMExist){
          Description md=（Description）c.getAnnotation(Description.class);
          System.out.println(md.value());
        }
      }
    }catch(ClassNotFoundException e){
      e.printStackTrace();
    }
  }
}
```

* 另一种解析方法上的注解:
    + 获取这个方法的所有注解，`Annotation [] as=m.getAnnotations();`然后遍历该注解，如果遍历的注解是Description类型，则把遍历的注解强转为Description类型，并进行输出value()信息。

``` java
for(Method m:ms){
  Annotation [] as=m.getAnnotations();
  for(Annotation a:as){
    if(a instanceof Description){
      Description md = (Description)a;
      System.out.println(md.value());
    }
  }
}
```

> @Inherited:当自定义注解上使用了该注解，如果在父类上标识该注解，解析一个子类，子类也可以获取该注解的信息。 

