---
layout: post
title:  "Java集合框架之HashMap源码解析"
subtitle:   "Java集合框架之HashMap源码解析"
author:     "李航"
date:   2017-12-06 21:35:59 +0800
catalog: true
tags:	Java
---

# Java集合框架之HashMap源码解析
> HashMap是通过key-value键值对的方式来存储数据的，通过put、get方法实现键值对的快速存取，这是HashMap最基本的用法。HashMap实现了Map接口，允许放入null元素，除该类未实现同步外，其余跟Hashtable大致相同，跟TreeMap不同，该容器不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个HashMap的顺序可能会不同。

##概述

有两个参数可以影响HashMap的性能：初始容量（inital capacity）和负载系数（load factor）。初始容量指定了初始table的大小，负载系数用来指定自动扩容的临界值。当entry的数量超过capacity*load_factor时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。

![HashMap结构图](http://img.blog.csdn.net/20171206200800108?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGloYW5nNjU2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##方法详解：
###put()

```
  public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //初始化操作
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果tab[i]为空则将Node放入tab[i]
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //如果新加入元素与tab[i]的元素key相同，则做替换操作
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //该节点为TreeNode的情况
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //遍历tab[i]的链表
                for (int binCount = 0; ; ++binCount) {
                    //如果该链表没有元素的key与新加元素冲突则将新的Node对象加到链表末尾
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果新加入元素与该Node的key相同，则做替换操作
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //真正执行替换操作
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //操作次数加1
        ++modCount;
        //如果将要超出容量则重新调整大小
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
###get()

```
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

```
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //当前table不为空,根据hash值查找相应数组位置
        if ((tab = table) != null && (n = tab.length) > 0 &&(first = tab[(n - 1) & hash]) != null) {
            //优先检查第一个node
            if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //向后查找
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    //如果Node对象的hash值跟上面计算得到的hash值相等，并且key也相等，那么就返回此Node对象
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //当前table为空，直接返回null
        return null;
    }
```

###remove()

```
public boolean remove(Object key, Object value) {
        return removeNode(hash(key), key, value, true, true) != null;
    }
```

```
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //根据hash值和数组长度计算数组下标值i，而且该位置不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //如果此单链表上Node p的hash值与计算出的hash值相等并且其key也跟传入的key相等，则从单链表上移除p
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            //遍历链表后续位置
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        //如果此单链表上Node e的hash值与计算出的hash值相等并且其key也跟传入的key相等，则从单链表上移除e
                        if (e.hash == hash &&
                                ((k = e.key) == key ||
                                        (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //真正移除节点的操作
            if (node != null && (!matchValue || (v = node.value) == value ||
                    (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                //操作次数+1
                ++modCount;
                //容量-1
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```
##总结
 从以上源码的分析中我们知道了HashMap底层维护的是数组加链表的混合结构，其本质是对数组和链表的操作。要注意的是HashMap不是线程安全的，我们可以使用Collections.synchoronizedMap方法来获得线程安全的HashMap。例如：

```
Map map = Collections.sychronizedMap(new HashMap());
```
 以上是我个人对HashMap底层原理的一点理解，不妥的地方欢迎指正！


