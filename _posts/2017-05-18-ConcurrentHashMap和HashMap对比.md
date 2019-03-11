---
title: ConcurrentHashMap和HashMap对比
categories:
 - Java
tags: Java
---

### ConcurrentHashMap
两种数据结构最大的区别就是线程安全了。
ConcurrentHashMap是线程安全的，采用的是**锁分段技术**，
首先将数据分成一段一段的存储，然后给每一段数据配一把锁，
当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。
Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，
HashEntry则用于存储键值对数据。
一个ConcurrentHashMap里包含一个Segment数组，
Segment的结构和HashMap类似，是一种数组和链表结构， 
一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 
每个Segment守护者一个HashEntry数组里的元素,
当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

### HashMap工作原理
HashMap允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。
在HashMap中有两个重要参数，capacity(容量)和load factor(负载因子)。
Capacity就是buckets的数目，Load factor就是buckets填满程度的最大比例。
如果对迭代性能要求很高的话不要把capacity设置过大，也不要把load factor设置过小。
当bucket填充的数目（即hashmap中元素的个数）大于capacity*load factor时，
就需要调整buckets的数目为当前的2倍。
#### put
put函数大致的思路为：
+ 对key的hashCode()做hash，然后再计算index;
+ 如果没碰撞直接放到bucket里；
+ 如果碰撞了，以链表的形式存在buckets后；
+ 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
+ 如果节点已经存在就替换old value(保证key的唯一性)
+ 如果bucket满了(超过load factor*current capacity)，就要resize。
#### get
+ bucket里的第一个节点，直接命中；
+ 如果有冲突，则通过key.equals(k)去查找对应的entry
    若为树，则在树中通过key.equals(k)查找，O(logn)；
    若为链表，则在链表中通过key.equals(k)查找，O(n)。
#### hash
在get和put的过程中，计算下标时，先对hashCode进行hash操作，然后再通过hash值进一步计算下标。
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
异或一下，既减少了系统的开销，也不会造成的因为高位没有参与下标的计算(table长度比较小时)，从而引起的碰撞。
如果还是产生了频繁的碰撞，使用树来处理频繁的碰撞。
之前已经提过，在获取HashMap的元素时，基本分两步：
首先根据hashCode()做hash，然后确定bucket的index；
如果bucket的节点的key不是我们需要的，则通过keys.equals()在链中找。
在Java 8之前的实现中是用链表解决冲突的，在产生碰撞的情况下，进行get时，两步的时间复杂度是O(1)+O(n)。
因此，当碰撞很厉害的时候n很大，O(n)的速度显然是影响速度的。
因此在Java 8中，利用红黑树替换链表，这样复杂度就变成了O(1)+O(logn)了，这样在n很大的时候，能够比较理想的解决这个问题
#### resize
当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。
当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，
所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
我们在扩充HashMap的时候，不需要重新计算hash，
只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。

#### 总结
1. 什么时候会使用HashMap？他有什么特点？
是基于Map接口的实现，存储键值对时，它可以接收null的键值，是非同步的，HashMap存储着Entry(hash, key, value, next)对象。

2. 你知道HashMap的工作原理吗？
通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Facotr则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

3. 你知道get和put的原理吗？equals()和hashCode()的都有什么作用？
通过对key的hashCode()进行hashing，并计算下标( n-1 & hash)，从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中去查找对应的节点

4. 你知道hash的实现吗？为什么要这样实现？
在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

5. 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？
如果超过了负载因子(默认0.75)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。

原文链接<http://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/>