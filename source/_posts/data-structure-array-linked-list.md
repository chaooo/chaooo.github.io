---
title: 「数据结构与算法」数组与链表
date: 2022-02-28 21:05:24
tags: [算法, 数据结构]
categories: 数据结构
---

线性表（Linear List）。顾名思义，线性表就是数据排成像一条线一样的结构。每个线性表上的数据最多只有前和后两个方向。比如数组，链表、队列、栈等也是线性表结构。
而与它相对立的概念是非线性表，比如二叉树、堆、图等。之所以叫非线性，是因为，在非线性表中，数据之间并不是简单的前后关系。
<!-- more -->

### 1. 数组
数组（`Array`）是一种线性表数据结构。它用一组连续的内存空间，来存储一组具有相同类型的数据。

数组寻址
```
a[i]_address = base_address + i * data_type_size
```

以长度为10的int数组（`int[] a = new int[10]`）为例：分配了一块连续内存空间 `1000～1039`，其中，内存块的首地址为 `base_address = 1000`，`int`类型的`data_type_size`为`4`个字节，即`a[0]`地址为`1000`，`a[1]`地址为`1004`...

![](array_address.webp)

容器是否完全替代数组？
- 容器的优势：对于Java语言，容器封装了数组插入、删除等操作的细节，并且支持动态扩容。
- 对于Java，一些更适合用数组的场景：
    1. Java的`ArrayList`无法存储基本类型，需要进行装箱操作，而装箱与拆箱操作都会有一定的性能消耗，如果特别注意性能，或者希望使用基本类型，就可以选用数组。
    2. 若数组大小事先已知，并且对数组只有非常简单的操作，不需要使用到`ArrayList`提供的大部分方法，则可以直接使用数组。
    3. 多维数组时，使用数组会更加直观。

### 2. 链表
链表（`Linked list`）也是一种线性表数据结构。它的内存结构是不连续的内存空间，是将一组零散的内存块串联起来，从而进行数据存储。
链表中每一个内存块称为节点（`Node`），节点除了存储数据外，还需存储下一个节点的地址。

1. 单链表：每个节点只包含一个指针，即后继指针（`next`）；首节点地址表示整条链表，尾节点的后继指针指向空地址null。
2. 循环链表：尾节点的后继指针指向首节点，其他与单链表一致。
3. 双向链表：每个节点包含两个指针，前驱指针（`prev`）和后继指针（`next`），首节点的前驱指针和尾节点后继指针都指向空地址null。
4. 双向循环链表：首节点前驱指针指向尾节点，尾节点后继指针指向首节点，其他与双链表一致。


### 3. 数组vs链表
1. 数组中的元素存在一个连续的内存空间中，若数组申请空间足够但不连续也会失败；而链表中的元素可以存在于不连续的内存空间，不过需要额外的内存空间存储指针信息。
2. 数组支持随机访问，根据下标随机访问的时间复杂度是`O(1)`；链表随机访问的时间复杂度是`O(n)`。
3. 链表适合插入、删除操作，时间复杂度为`O(1)`；数组插入、删除操作，时间复杂度为`O(n)`。
4. 数组大小固定，若存储空间不足，需要进行扩容，扩容就需要数据复制，这非常耗时；链表进行频繁的插入删除操作会导致内存频繁的内存申请和释放，容易造成内存碎片，Java环境还可能造成频繁的`GC`(自动垃圾回收)操作。
5. 数组在实现上使用连续的内存空间，可以借助CPU的缓冲机制预读数组中的数据，所以访问效率更高，而链表在内存中并不是连续存储，所以对CPU缓存不友好，没办法预读。


### 附：双向链表的Java实现
``` java
/**
 * 双向链表
 */
public class LinkedList<E> {
    int size = 0;
    /**
     * 头节点指针
     */
    Node<E> first;
    /**
     * 尾节点指针
     */
    Node<E> last;
    /**
     * 构造函数
     */
    public LinkedList() {}
    /**
     * 结点类（私有静态内部类）
     */
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    /**
     * 返回size
     */
    public int size() {
        return size;
    }

    /**
     * 添加元素（默认尾部添加）
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    /**
     * 添加为头元素
     */
    public boolean addFirst(E e) {
        linkFirst(e);
        return true;
    }

    /**
     * 删除元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 根据索引获取元素
     */
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    /**
     * 根据索引设置元素
     */
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }

    /**
     * e元素链接为尾部
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<E>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
    }

    /**
     * e元素链接为头部
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<E>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
    }

    /**
     * 取消非空节点 x 的链接
     */
    void unlink(Node<E> x) {
        // assert x != null;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
        x.item = null;
        size--;
    }

    /**
     * 根据指定索引检查非空
     */
    private void checkElementIndex(int index) {
        if (!(index >= 0 && index < size))
            throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
    }
    /**
     * 返回指定索引的（非空）节点。
     */
    Node<E> node(int index) {
        Node<E> x;
        if (index < (size >> 1)) {
            x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
        } else {
            x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
        }
        return x;
    }
}
```
