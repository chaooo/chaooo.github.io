---
title: 「Java教程」常用设计模式
date: 2017-05-30 08:34:55
tags: [Java, 后端开发]
categories: Java教程
---


设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。
<!-- more -->


### 1.常用的设计原则
- **开闭原则**：对扩展开放，对修改关闭
- **里氏代换原则**：任何父类出现的的地方，子类一定可以出现（多使用继承和多态）
- **依赖倒转原则**：尽量多依赖于抽象类或接口而不是具体实现类，对子类具有强制性和规范性
- **接口隔离原则**：尽量多依赖小接口而不是大接口
- **迪米特法则**（最少知道原则）：一个实体应当少与其他实体之间发生相互作用，使系统功能模块相对独立。高内聚，低耦合。
- **合成复用原则**：尽量多使用合成/聚合的方式，而不是继承的方式。


### 2.设计模式分类
#### 2.1 基本概念
- 设计模式是一套被反复使用多数人知晓，经过分类编目，代码设计经验的总结。
- 设计模式用来解决某些特定场景下的某一类问题-->通用的解决方案。
- 设计模式可以让代码更容易被理解，确保了复用性、可靠性、可扩展性

#### 2.2 具体分类
1. **创建型模式**：*用于对象创建的过程*
  - **单例模式**、**工厂方法模式**、抽象工厂模式、建造者模式(生成器模式)、原型模式
2. **结构型模式**：*用于把类或对象通过某种形式结合在一起，构成某种复杂或合理的结构*
  - 适配器模式、装饰者模式、代理模式、外观模式、桥接模式、组合模式、享元模式(过滤器/标准模式)
3. **行为型模式**：*用于解决类或对象之间的交互，更合理的优化类或对象之间的关系*
  - 责任链模式、命令模式、迭代子模式(迭代器模式)、观察者模式、中介者模式、解析器模式、状态模式、空对象模式、策略模式、**模板模式**、访问者模式、备忘录模式、
4. JEE 设计模式
  - 数据访问对象模式 


### 3.单例模式（Singleton）
#### 3.1 实现流程：
1. 私有的构造方法
2. 私有的静态的当前类的对象作为属性
3. 共有的静态方法返回当前对象
#### 3.1 实现方式：
1. 饿汉式：立即加载，对象启动时就加载
2. 懒汉式：延迟加载，对象什么时候用到时才会加载
3. 生命周期托管：单例对象交给别人处理


### 4.模板模式
在模板模式中，父抽象类公开几个抽象方法供子类实现。在父抽象类中有另一个方法或几个方法使用抽象方法来实现业务逻辑。

- eg: 对于使用不同的软件，我们只需要从抽象类继承并提供详细的实现,模板模式是一种行为模式。

``` java
  // 抽象类
abstract class Software {
   abstract void initialize();
   abstract void start();
   abstract void end();
   public final void play(){
      initialize();
      start();
      end();
   }
}
  // 不同子类以不同方法实现抽象类的的方法
class Browser extends Software {
   @Override
   void end() {
      System.out.println("Browser Finished!");
   }
   @Override
   void initialize() {
      System.out.println("Browser Initialized!.");
   }
   @Override
   void start() {
      System.out.println("Browser Started.");
   }
}
class Editor extends Software {
   @Override
   void end() {
      System.out.println("Editor Finished!");
   }
   @Override
   void initialize() {
      System.out.println("Editor Initialized!");
   }
   @Override
   void start() {
      System.out.println("Editor Started!");
   }
}
// 使用
public class Main {
   public static void main(String[] args) {
      Software s1 = new Browser();
      s1.play();
      s1 = new Editor();
      s1.play();    
   }
}
```

#### 4.1 模板方法模式优缺点：
1. 优点
  - 模板方法模式通过把不变的行为搬移到超类，去除了子类中的重复代码。子类实现算法的某些细节，有助于算法的扩展。通过一个父类调用子类实现的操作，通过子类扩展增加新的行为，符合“开放-封闭原则”。
2. 缺点
  - 每个不同的实现都需要定义一个子类，这会导致类的个数的增加，设计更加抽象。
3. 适用场景
  - 在某些类的算法中，用了相同的方法，造成代码的重复。控制子类扩展，子类必须遵守算法规则。


### 5. 工厂模式
1. 简单工厂模式：一个工厂方法，依据传入的参数，生成对应的产品对象；
2. 工厂方法模式：将工厂提取成一个接口或抽象类，具体生产什么产品由子类决定；
3. 抽象工厂模式：为创建一组相关或者是相互依赖的对象提供的一个接口，而不需要指定它们的具体类。

#### 5.1 简单工厂模式的实现：
``` java
  // 产品接口
public interface Fruit { void whatIm(); }
  // 具体类
public class Apple implements Fruit {
  @Override
  public void whatIm() { /*苹果*/}
}
public class Pear implements Fruit {
    @Override
    public void whatIm() { /* 梨 */ }
}
  // 工厂
public class FruitFactory {
    public Fruit createFruit(String type) {
        if (type.equals("apple")) {//生产苹果
            return new Apple();
        } else if (type.equals("pear")) {//生产梨
            return new Pear();
        }
        return null;
    }
}
  // 使用
FruitFactory mFactory = new FruitFactory();
Apple apple = (Apple) mFactory.createFruit("apple");//获得苹果
Pear pear = (Pear) mFactory.createFruit("pear");//获得梨
```
> 简单工厂只适合于产品对象较少，且产品固定的需求


#### 5.2 工厂方法模式实现：
``` java
  // 工厂接口
public interface FruitFactory {
    Fruit createFruit();//生产水果
}
  // 具体工厂
public class AppleFactory implements FruitFactory {
    @Override
    public Fruit createFruit() {
        return new Apple();
    }
}
public class PearFactory implements FruitFactory {
    @Override
    public Fruit createFruit() {
        return new Pear();
    }
}
  // 使用
AppleFactory appleFactory = new AppleFactory();
PearFactory pearFactory = new PearFactory();
Apple apple = (Apple) appleFactory.createFruit();//获得苹果
Pear pear = (Pear) pearFactory.createFruit();//获得梨
```
> 工厂方法模式虽然遵循了开闭原则，但如果产品很多的话，需要创建非常多的工厂

#### 5.3 抽象工厂模式实现：
  - 抽象工厂和工厂方法的模式基本一样，区别在于，工厂方法是生产一个具体的产品，而抽象工厂可以用来生产一组相同，有相对关系的产品；重点在于一组，一批，一系列；
  - eg：假如生产小米手机，小米手机有很多系列，小米note、红米note等；假如小米note生产需要的配件有825的处理器，6英寸屏幕，而红米只需要650的处理器和5寸的屏幕就可以了；用抽象工厂来实现：

``` java
 // cpu接口和实现类
public interface Cpu {
    void run();
    class Cpu650 implements Cpu {
        @Override
        public void run() {/* 625 也厉害 */ }
    }
    class Cpu825 implements Cpu {
        @Override
        public void run() { /* 825 处理更强劲 */ }
    }
}
  // 屏幕接口和实现类
public interface Screen {
    void size();
    class Screen5 implements Screen {
        @Override
        public void size() {/* 5寸 */}
    }
    class Screen6 implements Screen {
        @Override
        public void size() { /* 6寸 */ }
    }
}
  // 工厂接口
public interface PhoneFactory {
    Cpu getCpu();//使用的cpu
    Screen getScreen();//使用的屏幕
}
  // 具体工厂实现类
public class XiaoMiFactory implements PhoneFactory {
    @Override
    public Cpu getCpu() {
        return new Cpu.Cpu825();//高性能处理器
    }
    @Override
    public Screen getScreen() {
        return new Screen.Screen6();//6寸大屏
    }
}
public class HongMiFactory implements PhoneFactory {

    @Override
    public Cpu getCpu() {
        return new Cpu.Cpu650();//高效处理器
    }
    @Override
    public Screen getScreen() {
        return new Screen.Screen5();//小屏手机
    }
}
```
> 对于大批量，多系列的产品，用抽象工厂可以更好的管理和扩展；

#### 5.4 三种工厂方式总结：
1. 对于简单工厂和工厂方法来说，两者的使用方式实际上是一样的，如果对于产品的分类和名称是确定的，数量是相对固定的，推荐使用简单工厂模式；
2. 抽象工厂用来解决相对复杂的问题，适用于一系列、大批量的对象生产；


### 6.适配器模式（Adapter）
- 适配器模式Adapter是结构型模式的一种，分为**类适配器模式**，**对象适配器模式**，**缺省适配器模式**。
  * 类的适配器模式把适配的类的API转换成为目标类的API。使用对象继承的方式，是静态的定义方式；
  * 对象的适配器模式把被适配的类的API转换成为目标类的API，与类的适配器模式不同的是，对象的适配器模式不是使用继承关系，而是使用委派关系。一个适配器可以把多种不同的源适配到同一个目标。

> 适配器模式的缺点
<br>过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

#### 6.1 缺省适配器模式
- 缺省适配(Default Adapter)模式为一个接口提供缺省实现，这样子类型可以从这个缺省实现进行扩展，而不必从原有接口进行扩展。作为适配器模式的一个特例，缺省是适配模式在JAVA语言中有着特殊的应用。
- 缺省适配模式是一种“平庸”化的适配器模式。(实现类不必实现接口所有方法或留空的方法，可以有选择性了)
- 适配器(通常是一个抽象类)添加某些具体实现(需要缺省的方法内部抛出异常)。

