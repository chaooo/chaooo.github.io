---
title: 「Java教程」集合框架
date: 2017-04-17 22:10:57
tags: [Java, 后端开发]
categories: Java教程
---


为了方便对多个对象的操作，对对象进行存储，集合就是存储对象最常用的一种方式。
<!-- more -->


### 1. Collection集合
- Collection集合框架，字面意思容器；与数组类似，集合的长度存储之后还能改变，集合框架中包含了一系列不同数据结构的实现类。

> 数组与集合的比较
> - 数组的特点：
    1. 数组本质上就是一段连续的存储单元，用于存放多个类型相同的数据类容；
    2. 支持下标访问，实现随机访问非常方便；
    3. 增删操作不方便，可能会移动大量元素；
    4. 数组一旦声明长度固定无法更改；
    5. 数组支持基本数据类型，也支持引用数据类型；
> - 集合的特点：
    1. 集合的存储单元可以不连续，数据类容可以不相同；
    2. 集合部分支持下标访问，部分不支持；
    3. 集合中增删元素可以不移动大量元素；
    4. 集合大小可以随时动态调整；
    5. 集合中的元素必须是引用数据类型（基本数据类型可用包装类）；

```
-Collection接口
    |————List接口
        |————ArrayList类
        |————LinkedList类
        |————Stack类
        |————Vector类
    |————Queue接口
        |————LinkedList类
    |————Set接口
        |————HashSet类
        |————TreeSet类
-Map接口
    |————HashMap类
    |————TreeMap类
```

> - Collection存储的都是value,其中List有序可重复，Set无序无重复
> - Map存储的是以key-value形式,key无序无重复 value无序可重复
> - 序 : 顺序--添加进去的元素，取得元素的顺序一致；注意指的不是集合自己的顺序

|Collection集合的常用方法||
|----------|------------|
|boolean add(E e); | 向集合中添加对象|
|boolean contains(Object o); | 判断是否包含指定对象|
|boolean remove(Object o); | 从集合中删除对象|
|void clear(); | 清空集合 |
|int size(); | 返回包含对象的个数 |
|boolean isEmpty(); | 判断是否为空 |

``` java
Collection c2 = new ArrayList(); //多态
boolean b1 = c2.add(new String("one")); //true
boolean b2 = c2.add(new Integer(2)); //true
System.out.println("c2 = " + c2); //[one, 2]

boolean b3 = c2.contains(new Integer(2));//true
//contains方法工作原理：(o==null ? e==null : o.equals(e));
```


### 2. List集合
- java.util.List集合是Collection集合的子集合。
- List集合中元素有先后放入次序并且元素可以重复；实现类有：ArrayList类、LinkedList类、Stack类以及Vector类。
    - ArrayList类的底层使用**数组**进行数据管理，访问元素方便，增删不方便。
    - LinkedList类的底层使用**链表**进行数据管理，访问不方便，增删方便。
    - Stark类的底层使用数组进行数据管理，该类主要描述具有**后进先出**的特征的数据结构，叫做**栈**。
    - Vector类的底层使用数组进行数据管理，与ArrayList类似，与之比线程安全的类，因此效率低。
- List类除了继承Collection定义的方法外，还根据线性表的数据结构定义了一系列方法，其中最常用的是基于下标的get()，set()方法。

|List类常用方法|   |
|----|---|
|void add(int index, E element)|向集合指定位置添加元素|
|boolean addAll(int index, Collection<?extends E> c)|向集合中添加所有元素|
|E get(int index)|从集合中获取指定位置的元素|
|E set(int index, E element)|修改指定位置的元素|
|E remove(int index)|删除指定位置的元素|
|int indexOf(Object o)|在集合中检索某个对象，判断逻辑(o==null?get(i)==null:o.equals(get(i)))|
|<T> T[] toArray(T[] a)|将集合中的对象序列化以对象数组的形式返回。|
|List<E> subList(int fromIndex, int toIndex)|获取List从fromIndex(包括)和 toIndex(不包括)之间的部分视图|



### 3. 泛型机制
- 集合可以存放不同的对象，本质上都看作Object类型放入，此时从集合中取出也是Object类型，为了表达该元素真实类型需要强制类型转换，而强制类型转换可能发生类型转换异常。
- 从jdk1.5开始推出泛型机制，在集合名称后面使用<数据类型>的方式明确要求该集合中可以存放的数据类型。如：`List<String> lt = new LinkedList<String>();`。
- 从jdk1.7开始可省略后面<>的数据类型，叫做`菱形特性`，如：`List<String> lt = new ArrayList<>();`。
- 泛型本质就是参数化类型，让数据类型作为参数传递，`public interface List<E>{}`其中`E`是占位形参，由于实参可以支持各种广泛的类型，因此得名`泛型`。
- 泛型可以用在哪里：
    1. 泛型类：类定义的时候描述某种数据类型，集合的使用就是这样
    2. 泛型接口：与泛型类的使用基本一致，子类实现接口时必须添加泛型
    3. 泛型方法：方法调用时传参数，方法的泛型与类无关，带有泛型的方法可以不放在带有泛型的类中
    4. 方法参数泛型限制，高级泛型，规范边界，extends，super


### 4. Queue集合
- java.util.Queue集合是Collection集合的子集合。
- Queue集合主要描述具有**先进先出**特性的数据结构，叫做**队列**(FIFO:First Input First Output)。
- Queue集合主要实现类是`LinkedList类`，因为该类在增删方面有一定优势。

|Queue接口中主要方法| |
|----------|--------|
|boolean offer(E e)| 将一个对象添加至队尾，若添加成功则返回true|
|E poll()|从队首删除并返回一个元素|
|E peek()|返回队首的元素（但并不删除）|

``` java
Queue<Integer> q1 = new LinkedList<Integer>();
//将数据11、22、33、44、55依次入队
for(int i=1; i<=5; i++) {
    q1.offer(i*11);
}
```


### 5. *ArrayList类
1. 底层是利用(动态)数组形式实现，jdk1.5，所属的包 java.util
2. ArrayList特点适合遍历轮询，不适合插入删除
3. 如何构建一个ArrayList对象
    - 无参数构造方法，带默认容量构造方法，带collection参数的构造方法
4. ArrayList中常用的方法
    - 增删改查：add(E e)，remove(index)，set(index value)，get(index)，size()
5. 类中其他常用的方法
    - addAll并集，removeAll差集，ratainAll交集;
    - indexOf()，lastIndexOf()，contains()，List=subList();
    - isEmpty()，clear()，ensureCapacity()，iterator();迭代器
    - toArray(T[] x)，trimToSize();


### 6. Vector类
1. 是ArrayList集合的早期版本，所属的包 java.util
    - Vector底层也是利用(动态)数组的形式存储
    - Vector是线程同步的(synchronized)，安全性高，效率较低
2. 扩容方式与ArrayList不同
    - 默认是扩容2倍，可以通过构造方法创建对象时修改这一机制
3. 构造方法和常用方法与ArrayList类似



### 7. Stack类
1. Stack类，栈，java.util包
2. 构造方法只有一个无参数
3. 除了继承自Vacton类的方法外还有特殊的方法
    - push(E e)将某一个元素压入栈顶(add())
    - E = pop()将某一个元素从栈顶取出并删掉(E = remove())
    - E = peek()查看栈顶的一个元素 不删除(get())
    - boolean = empty()判断栈内元素是否为空(isEmpty())
    - int = search()查找给定的元素在占中的位置(indexOf())
4. 应用场景
    - 中国象棋，悔棋
    - 栈中存储每一次操作的步骤
    - 撤销功能



### 8. *LinkedList类
1. LinkedList类，java.util包
2. 底层使用**双向链表**的数据结构形式来存储
    - 适合于插入或删除  不适合遍历轮询
3. 构建对象
    - 无参数构造方法，带参数的构造方法(collection)
4. 常用的方法
    - 增删改查：add()，remove()，set()，get()，size()，offer，poll，peek
    - 手册中提供的其他常用方法：addAll，addFist，addLast()，clear()，contains()，element()，getFirst()，getLast()，indexOf()，lastIndex()
5. 插入删除的特性是否像想的那样
    - 对比ArrayList  Linked



### 9. Set集合
- java.util.Set集合是Collection集合的子集合。
- Set集合没有先后放入次序，并且不允许有重复关系，实现类有`HashSet类`和`TreeSet`类。
- 其中`HashSet类`底层是采用哈希表进行数据管理的。
- 其中`TreeSet类`的底层是采用二叉树进行数据管理的。

``` java
//方法和Collection集合基本一样
Set<String> set1 = new HashSet<String>();
set1.add("one");
System.out.println("s1="+s1);
```

- set集合的无重复特性
    * HashSet，无重复原则有两个方法同时起作用
        - equals    hashCode
        - 默认比较的是两个对象的地址  若第二个对象地址与之前的一致  不再存入
        - 如果想要改变其比较的规则  可以重写上述两个方法
    * TreeSet，无重复原则有一个方法起作用
        - compareTo
        - 上述这个方法不是每一个对象都有的
        - 若想要将某一个对象存入TreeSet集合中，需要让对象所属的类实现接口Comparable
        - 实现接口后将compareTo方法重写，返回值int，负数靠前排布，整数排列靠后

#### 9.1 Set集合的遍历
- 所有Collection的实现类都实现了其iterator方法，该方法返回Iterator接口类型对象，用于实现对集合元素的迭代遍历。

|迭代器`Iterator<E> iterator()`，主要方法有||
|---------------|-------------------------|
|boolean hasNext() | 判断集合中是否有可以迭代/访问的元素 |
|E next() | 用于取出一个元素并指向下一个元素|
|void remove() | 用于删除访问到的最后一个元素|

``` java
Iterator<String> it = set1.iterator();//获取当前集合的迭代器对象
while(it.hasNext()) {//判断是否有可以访问的元素
    String temp = it.next();//取出一个并指向下一个
    System.out.println( temp );
    if("two".equals(temp)){
        it.remove();//删除set1中该元素
    }
}
```

- 增强for循环(for each结构)
- 语法格式：`for(元素类型 变量名:集合/数组){ 循环体; }`。
- 执行流程：不断从集合/数组中取出一个元素赋值给变量名后执行循环体，直到取出所有元素。

``` java
//遍历集合
for(String ts : s1) {
    System.out.println(ts);
}
//遍历数组
int[] arr = {11,22,33,44,55};
for(int ti : arr) {
    System.out.println(ti);
}
```



### 10. HashSet类
1. HashSet集合底层采用HashMap（数组+链表-->散列表），java.util包。
2. 它不保证set 的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用null元素。 
3. 创建对象：无参数，有参数
4. 集合容器的基本使用
    - 增删改查：boolean = add(value)，addAll(collection c)，retainAll，removeAll，boolean = remove(Object)
    - 没有修改方法
    - iterator()  获取一个迭代器对象
    - size()
5. 无重复的原则
    - 在HashSet中，元素都存到HashMap键值对的Key上面，而Value时有一个统一的值private static final Object PRESENT = new Object();，(定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。)



### 11. TreeSet类
1. TreeSet类，无序无重复，java.util包。(底层TreeMap 二叉树 利用Node(left item right))
2. 创建对象： 无参数构造方法 ，带Collection构造方法
3. 基本常用方法：add(E e)，iterator()，remove(E e)，没有修改，size()
4. 二叉树主要指每个节点最多只有两个子节点的树形结构。
5. 满足以下三个特征的二叉树叫做**有序二叉树**：
    * 左子树中的任意节点元素都小于根节点元素；
    * 右子树中的任意节点元素都大于根节点元素；
    * 左子树和右子树内部也遵守上述规则；
6. 无序无重复：treeSet集合本身有顺序，我们指的无序存入的和取出来的不一致。

7. 元素放入TreeSet集合过程：
由于TreeSet集合底层采用**有序二叉树**进行数据的管理，当有新元素插入到TreeSet集合时，需要使用新元素与集合中已有的元素依次比较来确定存放合理位置，而比较元素大小规则有两种方式：
    1. 使用元素的**自然排序**规则进行比较并排序，让元素类型实现java.lang.Comparable接口；
    2. 使用**比较器规则**进行比较并排序，构造TreeSet集合时传入java.util.Comparable接口；

> 注意：
    1. 自然排序的规则比较单一，而比较强的规则比较多元化，而且比较器优先于自然排序；
    2. 可以使用Collections工具类对集合中的元素进行操作；



### 12. Map集合
- java.util.Map<K, V>集合存取元素的基本单位是：单对元素（键值对key-value）。
- Map：映射，通过某一个key可以直接定位到一个value值
- key无序无重复   value无序可重复
    * key无序还是一样，指的是存入顺序与取得顺序不一致，key无重复当然指的是，元素不能一致
- 主要有两个实现类：`HashMap类`和`TreeMap类`。
- Map基本使用：HashMap，TreeMap，Properties
- Map集合常用方法：
    * 增改：put(key,value)，删：remove(key)，查：get(key),containsKey(key),containsValue(value)
- Map集合的遍历方式：a.迭代Key，b.迭代Entry
- Map集合的性能调优：
    - 加载因子较小时散列查找性能会提高，同时也浪费了散列桶空间容量。0.75是性能和空间相对平衡的结果，在常见散列表时指定合理容量，减少rehash提高性能。（Capacity:容量，Initial capacity:初始容量，Size:数据大小，Load factor:加载因子(size/capacity),默认0.75）



### 13. HashMap类
1. 包:java.util，底层散列表的形式（数组+链表）
2. 构造方法创建对象   无参数  带默认容量的  带map参数的构造方法
3. 特点:(数组+链表)底层散列表形式存储，key无序无重复,value无序可重复
    - 找寻某一个唯一元素的时候建议使用map，更适合于查找唯一元素，Map$Entry
4. 基本方法：
    - 增 put(key,value)，存放一组映射关系key-value
        1. key存储的顺序与取得顺序不同
        2. 不同的key可以存储相同的value
        3. key若有相同的 则将 原有的value覆盖而不是拒绝存入(跟set刚好相反)
    - 删 E = remove(key);
    - 改 replace(key,newValue)，put(key,value2)
    - 查 E = get(key)；
    - Set<Key> = keySet()获取全部的key
    - Set<Entry> = entrySet();
    - size();

``` java
Set<Entry<Integer,String>> entrys = map.entrySet();//获取集合中全部的entry对象
Iterator<Entry<Integer,String>> it = entrys.iterator();
while(it.hasNext()){
    Entry<Integer,String> entry = it.next();//entry  key value
    Integer key = entry.getKey();
    String value = entry.getValue();
    System.out.println(key+"--"+value);
}
```

5. 除了上述几个常用的方法外  其他API中提供的方法
    - clear，containsKey(key)，containsValue(value)
    - getOrDefault(key,defaultValue);如果key存在就返回对应的value 若没有找到则返回默认值
    - isEmpty()
    - putAll(map)
    - putIfAbsent(key,value);//如果key不存在才向集合内添加  如果key存在就不添加啦
6. map集合在什么情形下用?
    1. 想要存储一组元素
        - 数组  or  集合，如果存储的元素以后长度不变 用数组，如果长度以后不确定 用集合
    2. 如果发现长度以后不确定--->集合

| list | Set | Map |
|:------:|:----:|:------:|
| List家族有序的 | Set家族无重复 | Map家族k-v |
|存储有顺序用这个|存储元素希望自动去掉重复元素用这个|通过唯一的k快速找寻v用这个|
|ArrayList:更适合遍历轮询|HashSet:性能更高|HashMap:性能更高|
|LinkedList:更适合插入和删除|TreeSet:希望存进去的元素自动去重复,同时还能自动排序|Tree:希望存进去的元素key自动排序|
|Stack:LIFO|-|-|



### 14. TreeMap类
1. java.util包
2. 构造方法：无参数，带map参数
3. 常用方法：put， get，remove，replace，size
4. 底层数据结构的存储：红黑二叉树（层级多余2层可能会左旋或右旋）
5. 自然有序，按照Unicode编码自然有序
    - ap集合中的key需要可比较的   key的对象需要实现Comparable接口



### 15. Lambda表达式
- java8支持的新的语法格式，Lambda允许`把函数作为一个方法的参数`(函数作为参数传递进方法中)，使用lambda表达式可以`使代码变得更加简洁紧凑`。
- 函数式编程：一种抽象程度很高的编程范式。函数也可以跟变量、对象一样使用，可以作为参数，也可以作为返回值，大大简化了代码的开发。
- lambda表达式语法由**参数列表**、**箭头函数`->`**和**函数体**组成，函数体即可以是一个表达式，也可以是一个语句块。

``` lambda
(int a, int b) -> a+b
() -> 42
(String s) -> {System.out.println(s);}
```

- 函数式接口：指仅仅只包含一个抽象方法的接口，每一个该类型的lambda表达式大都会被匹配到这个抽象方法。
- jdk1.8提供了一个@FunctionalInterface注解来定义函数式接口，如果我们定义的接口不符合函数式的规范便会报错。

#### 15.1 Lambda表达式-方法引用
- 方法引用：只需要使用方法的名字，而具体调用交给函数式接口，需要和Lambda表达式配合使用。
- 方法引用和lambda表达式拥有相同的特性，我们并不需要为方法引用提供方法体，我们可以直接通过方法名称引用已有的方法。



### 16. Stream API
- Stream(流)借助lambda表达式来进行集合数据处理,分为中间操作和最终操作两种；最终操作返回一特定类型的计算结果，而中间操作返回Stream本身，这样就可以将多个操作依次串起。
- 虽然大部分情况下stream是容器调用Collection.stream()方法得到的，但stream和collections有以下不同：
    * **无存储**。stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
    * **为函数式编程而生**。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
    * **惰式执行**。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
    * **可消费性**。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

- 对stream的操作分为为两类，中间操作和结束操作，二者特点是：
    * **中间操作**总是会惰式执行，调用中间操作只会生成一个标记了该操作的新stream，仅此而已。
    * **结束操作**会触发实际计算，计算发生时会把所有中间操作积攒的操作以pipeline的方式执行，这样可以减少迭代次数。计算完成之后stream就会失效。

#### 16.1 stream方法使用
- stream跟**函数接口**关系非常紧密，没有函数接口stream就无法工作（通常函数接口出现的地方都可以使用Lambda表达式，所以不必记忆函数接口的名字)。

``` java
// 找出最长的单词
Stream<String> stream = Stream.of("I", "love", "you", "too");
Optional<String> longest = stream.reduce((s1, s2) -> s1.length()>=s2.length() ? s1 : s2);
//Optional<String> longest = stream.max((s1, s2) -> s1.length()-s2.length());
System.out.println(longest.get());
```

