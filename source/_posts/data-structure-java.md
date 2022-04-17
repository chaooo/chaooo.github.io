---
title: 「数据结构」常见JAVA集合类的数据结构分析
date: 2019-07-28 22:53:06
tags: [java, 后端开发, 数据结构]
categories: 数据结构
---


### 集合(Collection/Map)
```
Collection接口
    |———— List接口
        |———— ArrayList类
        |———— Vector类
        |———— LinkedList类
        |———— Stack类
    |———— Set接口
        |———— HashSet类
        |———— TreeSet类
        |———— LinkedHashSet类
    |———— Queue接口
        |———— LinkedList类
Map接口
    |———— HashMap类
    |———— TreeMap类
    |———— LinkedHashMap类
    |———— Hashtable类
```

<!-- more -->

#### 0.1 List
- **Arraylist**： 动态数组
- Vector： 动态数组(线程安全)
- **LinkedList**： 双向链表(JDK1.6之前为循环链表，JDK1.7取消了循环)

#### 0.2 Set
- HashSet（无序，唯一）：基于 HashMap 实现的，底层采用 HashMap 来保存元素
- LinkedHashSet： LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
- TreeSet（有序，唯一）：红黑树(自平衡的排序二叉树)

#### 0.3 Map
- **HashMap**： JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为**红黑树**，以减少搜索时间
- LinkedHashMap： LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，**增加了一条双向链表**，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- Hashtable： 数组+链表(线程安全)，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
- TreeMap： 红黑树（自平衡的排序二叉树）

#### 0.4 如何选用集合
主要根据集合的特点来选用：
- 键值对就选用Map接口下的集合，需要排序时选择TreeMap，不需要排序时就选择HashMap,需要保证线程安全就选用ConcurrentHashMap.
- 只需要存放元素值时，就选用Collection接口下的集合，需要保证元素唯一时选择实现Set接口的集合（TreeSet或HashSet），不需要就选择实现List接口的ArrayList或LinkedList


#### 0.5 对数公式log 与 时空复杂度
- 若`a^n = b` (a>0,a≠1) 则 `n = log(a)b` , 如`log(2)8 = 3`; Java数据结构中log默认以2为底(个人理解,有待考证)
- 常用O(1), O(n), O(logn)表示对应算法的时间复杂度, 也用于表示空间复杂度。
    - **`O(1)`**: 最低的时空复杂度, 无论数据规模多大，都可以在一次计算后找到目标
    - **`O(n)`**: 数据量增大n倍时，耗时增大n倍; 比如常见的遍历算法
    - **`O(n^2)`**: 数据量增大n倍时，耗时增大n的平方倍; 比如冒泡排序，对n个数排序，需要扫描n×n次
    - **`o(logn)`**: 当数据增大n倍时，耗时增大logn倍; 二分查找就是O(logn)的算法，每找一次排除一半的可能，256个数据中查找只要找8次就可以找到目标(2^8=256)
    - **`O(nlogn)`**: 同理，就是n乘以logn，当数据增大256倍时，耗时增大256*8=2048倍。这个复杂度高于线性低于平方。归并排序就是O(nlogn)的时间复杂度


#### 0.6  移位运算符
按照平移的方向和填充数字的规则分为三种:<<(左移)、>>(带符号右移) 和 >>>(无符号右移)
- `左移 <<` : 丢弃最高位,0补最低位；左移n位就相当于乘以2的n次方
- `右移 >>` : 符号位不变,高位补上符号位(正数0, 负数1)；右移n位相当于除以2的n次方
- `无符号右移 >>>` : 忽略符号位，0补最高位(补码移位所得)

> - 正数的左移与右移，负数的无符号右移，就是相应的补码移位所得，在高位补0即可。
> - 负数的右移，就是补码高位补1,然后按位取反加1即可。

- 运算规则：
    + 左移：高位移出(舍弃)，低位的空位补零；int类型时，每移动1位它的第31位就要被移出并且丢弃；long类型时，每移动1位它的第63位就要被移出并且丢弃；byte和short类型时，将自动把这些类型扩大为int型。
    + 右移：低位移出(舍弃)，高位的空位补符号位，即正数补0，负数补1；当右移的运算数是byte 和short类型时，将自动把这些类型扩大为 int 型。
    + 无符号右移：补码移位，高位补0；正数和右移表现一致，负数变成了很大的正数；



### 1. Arraylist
ArrayList的底层是数组队列，相当于**动态数组**。与数组相比，它的容量能动态增长。在添加大量元素前，应用程序使用ensureCapacity操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。
它继承于 AbstractList，实现了 List, RandomAccess, Cloneable, java.io.Serializable 这些接口。
- **数组**时间复杂度: **插入/删除:O(n)**，**增加(末尾)/随机访问: O(1)**
- ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的**添加、删除、修改、遍历**等功能
- ArrayList 实现了RandomAccess 接口， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持快速随机访问的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是**快速随机访问**
- ArrayList 实现了Cloneable 接口，即覆盖了函数 clone()，**能被克隆**。
- ArrayList 实现java.io.Serializable 接口，这意味着ArrayList**支持序列化**，能通过序列化去传输。
- 和 Vector 不同，ArrayList 中的操作**不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者 CopyOnWriteArrayList。


#### 1.1 ArrayList扩容机制*（重点）
- 以无参数构造方法创建ArrayList时，实际上初始化赋值的是一个空数组；当add第一个元素时，才真正分配容量(**默认10**)
- ArrayList在每次增加元素(1个或一组)时，都要调用`ensureCapacityInternal()`方法来确保足够的容量
- 当容量不足以容纳当前的元素个数时，进入`grow()`方法进行扩容，首先设置新的容量为旧容量的**1.5倍**
- 若设置后的新容量还不够，则设置新容量为`minCapacity`(所需最小容量)
- 比较新容量是否大于`MAX_ARRAY_SIZE`(Integer最大值减8)，若大于，再比较`minCapacity`是否大于`MAX_ARRAY_SIZE`，若大于，设置新的容量为`Integer.MAX_VALUE`(Integer最大值)，否则设置新的容量为`MAX_ARRAY_SIZE`(Integer最大值减8)
- 最后用`Arrays.copyof()`方法将元素拷贝到新的数组
- (第`Integer.MAX_VALUE+1`次添加元素时，抛出`OutOfMemoryError`异常)

> **System.arraycopy()和Arrays.copyOf()方法**
> 通过源码发现这两个实现数组复制的方法被广泛使用, 比如插入操作add(int index, E element)方法就很巧妙的用到了 System.arraycopy()方法让数组自己复制自己实现让index开始之后的所有成员后移一个位置
> - Arrays.copyOf()内部也是调用了System.arraycopy()方法 
> - Arrays.copyOf()是系统自动在内部新建一个数组，并返回该数组
> - System.arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置


#### 1.2 ensureCapacity
ArrayList对外提供了一个`ensureCapacity(int n)`方法
- 最好在`add`大量元素之前用ensureCapacity方法，以**减少增量重新分配的次数**
- ensureCapacity一次性扩容到位，否则在添加大量元素的过程中，一点一点的进行扩容


#### 1.3 内部类
``` java
private class Itr implements Iterator<E>{...}
private class ListItr extends Itr implements ListIterator<E>{...}
private class SubList extends AbstractList<E> implements RandomAccess{...}
static final class ArrayListSpliterator<E> implements Spliterator<E>{...}
```
ArrayList有四个内部类
- Itr 实现了Iterator接口，同时重写了里面的hasNext()， next()， remove() 等方法；
- ListItr 继承 Itr，实现了ListIterator接口，同时重写了hasPrevious()， nextIndex()， previousIndex()， previous()， set(E e)， add(E e) 等方法
- Iterator和ListIterator的区别: 
    - ListIterator在Iterator的基础上增加了添加对象，修改对象，逆向遍历等方法，这些是Iterator不能实现的。



### 2. LinkedList
LinkedList是基于**双向链表**实现的, 可以在任何位置进行高效地插入和移除操作的有序序列。
- 复杂度: **增加(末尾)/删除:O(1)**，**插入/获取: O(n)**
- LinkedList 继承AbstractSequentialList的**双向链表**。它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行**队列操作**。
- LinkedList 实现 Deque 接口，即能将LinkedList当作**双端队列**使用。
- LinkedList 实现了Cloneable接口，即覆盖了函数clone()，**能克隆**。
- LinkedList 实现java.io.Serializable接口，这意味着LinkedList**支持序列化**，能通过序列化去传输。
- LinkedList **不是线程安全的**，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法

#### 2.1 LinkedList底层分析:
LinkedList的底层是一个双向链表，链表中挂载着一个个的Node元素；可以从LinkedList的Node内部类看出奥秘：
``` java
transient Node<E> first; //头指针
transient Node<E> last; //尾指针
//内部类
private static class Node<E> {
    E item; // 数据域（当前节点的值）
    Node<E> next; // 后继（指向当前一个节点的后一个节点）
    Node<E> prev; // 前驱（指向当前节点的前一个节点）
    // 构造函数，赋值前驱后继
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
- LinkedList 是基于链表结构实现，所以在类中包含了 first 和 last 两个指针(Node)。
- Node 中包含了上一个节点和下一个节点的引用，这样就构成了双向的链表。


#### 2.2 LinkedList增删改查
+ 链表批量增加，是靠for循环遍历原数组，依次执行插入节点操作。增加一定会修改modCount。
+ 通过下标获取某个node的时候(add select)，会根据index处于前半段还是后半段**进行一个折半**，以提升查询效率
+ 删也一定会修改modCount。 
    + 按下标删，也是先根据index找到Node，然后去链表上unlink掉这个Node。 
    + 按元素删，会先去遍历链表寻找是否有该Node，如果有，去链表上unlink掉这个Node。
+ 改也是先根据index找到Node，然后替换值。不修改modCount。
+ CRUD操作里，都涉及到根据index去找到Node的操作。


#### 2.2 unlink原理
- 先判断该节点是否存在上一个节点，即是否有前驱节点。
    - 无前驱节点则说明要删除的节点为链表的第一节点，那么只需要把该节点的下一个节点设置为链表的第一个节点。
    - 有前驱节点则需要把前驱节点的尾部引用指向该节点的下一个节点。
- 再判断该节点是否存在下一个节点，即是否有后继节点。
    - 无后继节点则说明该节点是链表的最后一个节点，那么只需要把该节点前驱节点设置成链表的最后一个节点即可。
    - 有后继节点则需要把后继节点的头部引用指向该节点的上一个节点。
- 核心就是在于将要删除的节点的前驱节点尾部指向该节点的后继节点，将要删除的节点的后继节点的头部指向该节点的前驱节点。这样便完成了链表的删除操作。

> 删除和新增方法的实现基本是对该节点的上一个节点和下一个节点的引用设置，不需要操作其他节点，效率相对较高


#### 2.3 offer与add的区别
- offer属于 offer in interface **Deque**。
- add 属于 add in interface **Collection**。
- 当队列为空时候，使用add方法会报错，而offer方法会返回false。
- 作为List使用时,一般采用add / get方法来 压入/获取对象。
- 作为Queue使用时,才会采用 offer/poll/take等方法作为链表对象时,offer等方法相对来说没有什么意义这些方法是用于支持队列应用的。


#### 2.2 对比Vector、ArrayList、LinkedList有何区别
这三者都是实现集合框架中的 List，也就是所谓的有序集合，因此具体功能也比较近似，比如都提供按照位置进行定位、添加或者删除的操作，都提供迭代器以遍历其内容等。但因为具体的设计区别，在行为、性能、线程安全等方面，表现又有很大不同。
- Vector 是 Java 早期提供的**线程安全**的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector 内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据扩容为旧容量的**2倍**。
- ArrayList 是应用更加广泛的动态数组实现，它本身不是线程安全的，所以性能要好很多。ArrayList 也是可以根据需要调整容量，在扩容为旧容量的**1.5倍**。
- LinkedList 顾名思义是 Java 提供的双向链表，**不需要扩容**，它也不是线程安全的。LinkedList不支持高效的随机元素访问。




### 3. HashMap
HashMap是**数组+链表+红黑树**（JDK1.8增加了红黑树部分）实现的, 用于存储Key-Value键值对的集合，每一个键值对也叫做一个Entry。这些Entry分散存储在一个数组当中，这个数组就是HashMap的主干。
- HashMap继承了AbstractMap类，实现了Map，Cloneable，Serializable接口
- 继承 abstractMap，也就是用来减轻实现Map接口的编写负担。
- 实现 Cloneable：能够使用Clone()方法，在HashMap中，实现的是**浅层次拷贝**，即对拷贝对象的改变会影响被拷贝的对象。
- 实现 Serializable：能够使之**序列化**，即可以将HashMap对象保存至本地，之后可以恢复状态。

> JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）.JDK1.8 以后在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（默认为 8）时，将链表转化为红黑树**，以实现O(logn)时间复杂读查找。

HashMap类中有一个非常重要的字段，就是 Node[] table，即**哈希桶数组**，明显它是一个Node的数组。
- HashMap的实例有两个参数影响其性能:
    + 初始容量(默认16)：哈希表中桶的数量
    + 加载因子(默认0.75)：哈希表在其容量自动增加之前可以达到多满的一种尺度
- 当哈希表中条目数超出了当前容量*加载因子(其实就是HashMap的实际容量)时，则对该哈希表进行rehash操作，将哈希表扩充至两倍的桶数。


#### 3.1 HashMap的 put 方法过程*（重点）
put方法内部是一个 `putVal` 的调用：
1. 对 Key 求 Hash 值，然后再计算下标。
2. 如果没有碰撞，直接放入桶中，
3. 如果碰撞了，若是树节点，就`putTreeVal`添加元素，若不是就遍历链表插入。
4. 如果链表长度超过阀值（TREEIFY_THRESHOLD==8），就把链表转成红黑树。
5. 如果节点已经存在就替换旧值，若未找到则继续
6. 如果桶满了（容量 * 加载因子），就需要 resize(扩容为原来2倍并重新散列,元素的下标要么不变，要么变为「原下标+原容量」)。

#### 3.2 HashMap 桶下标计算
- **下标**：`hash(key) & (table.length - 1)`
- 扰动函数**hash(key)**：(key==null) ? 0 : `(key.hashCode()^(key.hashCode() >>> 16))`
- 低16位 和 高 16位 做了一个**异或**得到 hash值 与 (容器长度-1)进行**取模(%)**运算,得到下标。
    + 利用位运算代替取模运算，提高程序的计算效率：（当 b=2^n 时，a%b = a & (b-1) ），也是因此，HashMap 才将初始长度设置为 16，且扩容只能是以 2 的倍数（2^n）扩容。
- 有些数据计算出的哈希值差异主要在高位，而HashMap里的哈希寻址是忽略容量以上的高位的，那么这种处理就可以尽可能有效的避免哈希碰撞。

> HashMap 的性能表现非常依赖于哈希码的有效性: equals相等，hashCode一定要相等。重写了 hashCode 也要重写 equals。hashCode 需要保持一致性，状态改变返回的哈希值仍然要一致。


#### 3.3 HashMap 容量、负载因子和树化
- 容量和负载系数决定了可用的桶的数量，空桶太多会浪费空间，如果使用的太满则会严重影响操作的性能。
- 如果能够知道 HashMap 要存取的键值对数量，可以考虑预先设置合适的容量大小。
- 计算条件：**负载因子 * 容量 > 元素数量**；所以，预先设置的容量需要满足，大于“预估元素数量/负载因子”，同时它是**2的幂数**
- 容量理论最大极限由 MAXIMUM_CAPACITY 指定，数值为 **1<<30**，也就是2的30次方

##### 3.3.1 HashMap 负载因子loadFactor
+ loadFactor加载因子是控制数组存放数据的疏密程度，越大越密，越小越稀疏。
+ loadFactor太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor的**默认值为0.75f**是官方给出的一个比较好的临界值。
+ 给定的默认容量为16，负载因子为0.75。当数量达到了 16*0.75 = 12 就需要将当前16的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
+ 而对于负载因子，建议：
    - 如果没有特别需求，不要轻易进行更改，因为 JDK 自身的默认负载因子是非常符合通用场景的需求的。
    - 如果确实需要调整，建议不要设置超过 0.75 的数值，因为会显著增加冲突，降低 HashMap 的性能。
    - 如果使用太小的负载因子，按照上面的公式，预设容量值也进行调整，否则可能会导致更加频繁的扩容，增加无谓的开销，本身访问性能也会受影响。

##### 3.3.2 HashMap 门限值threshold
`threshold = capacity * loadFactor`，当`Size>=threshold`的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 衡量数组是否需要扩增的一个标准。
- 门限值等于(负载因子 x 容量)，如果构建 HashMap 的时候没有指定它们，那么就是依据相应的默认常量值。
- 门限通常是以倍数进行调整 （newThr = oldThr << 1），根据 putVal 中的逻辑，当元素个数超过门限大小时，则调整 Map 大小。
- 扩容后，需要将老的数组中的元素重新放置到新的数组，这是扩容的一个主要开销来源。

##### 3.3.2 HashMap 树化改造
树化改造逻辑主要在 putVal 和 `treeifyBin` 中。
- 链表结构（这里叫 bin）的数量大于 `TREEIFY_THRESHOLD`(默认为8) 时：
    + 如果容量小于 `MIN_TREEIFY_CAPACITY`(默认为64) ，只会进行简单的扩容。
    + 如果容量大于 `MIN_TREEIFY_CAPACITY`(默认为64)，则会进行树化改造。


#### 3.4 HashMap 扩容resize
- HashMap扩容条件：
    + 元素个数超出了加载因子与当前容量的乘积，并且发生了Hash碰撞
- HashMap扩容步骤：
    1. 创建一个新的Entry空数组，长度是原来的2倍。
    2. 遍历原Entry数组，把所有的Entry重新Hash到新数组里。
    3. 重新散列的元素下标要么「不变」，要么变为「原下标+原容量」，取决于位运算((n - 1) & hash)

> 经过一次扩容处理后，元素会更加均匀的分布在各个桶中，会提升访问效率。
> 但会遍历所有的元素，时间复杂度很高；遍历元素所带来的坏处大于元素在桶中均匀分布所带来的好处。
> 尽量避免进行扩容处理。


#### 3.5 常见的hash算法及冲突的解决
hash函数，即散列函数。它可以将不定长的输入，通过散列算法转换成一个定长的输出，这个输出就是散列值(不保证唯一)。
- 常见Hash算法：
    1. 直接定址法：直接以关键字k或者k加上某个常数（k+c）作为哈希地址（H(k)=ak+b）。
    2. 数字分析法：提取关键字中取值比较均匀的数字作为哈希地址（如一组出生日期，相较于年-月，月-日的差别要大得多，可以降低冲突概率）
    3. 分段叠加法：按照哈希表地址位数将关键字分成位数相等的几部分，其中最后一部分可以比较短。然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。
    4. 平方取中法：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。
    5. 伪随机数法：选择一随机函数，取关键字的随机值作为散列地址，通常用于关键字长度不同的场合。
    6. 除留余数法：用关键字k除以某个不大于哈希表长度m的数p，将所得余数作为哈希表地址（H(k)=k%p, p<=m; p一般取m或素数）。
- 常见解决hash冲突的方法
    1. 链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为 i 的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。
    2. 开放定址法：即发生冲突时，去寻找下一个空的哈希地址。只要哈希表足够大，总能找到空的哈希地址。
    3. 再哈希法：即发生冲突时，由其他的函数再计算一次哈希值。
    4. 建立公共溢出区：将哈希表分为基本表和溢出表，发生冲突时，将冲突的元素放入溢出表。

> HashMap就是使用链地址法来解决冲突的（JDK1.8增加了红黑树）


#### 3.6 对比Hashtable、HashMap、TreeMap有什么不同
Hashtable、HashMap、TreeMap 都是最常见的一些 Map 实现，是以键值对的形式存储和操作数据的容器类型。
- Hashtable 是早期 Java 类库提供的一个哈希表实现，本身是同步的，不支持 null 键和值，由于同步导致的性能开销，所以已经很少被推荐使用。
- HashMap 是应用更加广泛的哈希表实现，行为上大致上与 HashTable 一致，主要区别在于 HashMap 不是同步的，支持 null 键和值等。通常情况下，HashMap 进行 put 或者 get 操作，可以达到常数时间的性能，所以它是绝大部分利用键值对存取场景的首选，比如，实现一个用户 ID 和用户信息对应的运行时存储结构。
- TreeMap 则是基于红黑树的一种提供顺序访问的 Map，和 HashMap 不同，它的 get、put、remove 之类操作都是 O(log(n))的时间复杂度，具体顺序可以由指定的 Comparator 来决定，或者根据键的自然顺序来判断。


