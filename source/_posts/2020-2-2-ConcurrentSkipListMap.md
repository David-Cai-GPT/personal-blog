---
layout: post
title: 查找/插入/删除的速度如何趋于线性且支持高并发——ConcurrentSkipListMap!
tag: 数据结构
date: 2020-2-2
category: Technology blog
---

ConcurrentSkipListMap，属于并发集合类，来源于大名鼎鼎的J.U.C，集合并发类的要求是执行速度快，提取数据准，最著名的类便是之前有接触到的ConcurrentHashMap类，通过不断的优化，由刚开始的锁分段到后来的CAS，不断地去提高自身的并发性能，其他便是ConcurrentSkipListMap，CopyOnWriteArrayList，BlockingQueue，虽然使用的频率没有AVL或者红黑树那么高，但是它的实现相对于树来说更加简单。

ConcurrentSkipListMap可以看成是并发版本的TreeMap，和TreeMap不同是，TreeMap不是线程安全的，ConcurrentSkipListMap是线程安全有序的哈希表，且不是基于红黑树实现的，其底层是一种类似**跳表（Skip List）**的结构

### 跳表（Skip List）

**Skip List**（以下简称跳表），是一种类似链表的数据结构，其查询/插入/删除的时间复杂度都是`O(logn)`。

我们知道，通常意义上的链表是不能支持随机访问的（通过索引快速定位），其查找的时间复杂度是`O(n)`，而数组这一可支持随机访问的数据结构，虽然查找很快，但是插入/删除元素却需要移动插入点后的所有元素，时间复杂度为`O(n)`。

首先我们来看下传统链表

![img](https://img-blog.csdnimg.cn/20200106144855852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果我想查询15这个元素，那么我需要从头开始遍历，知道遍历到我想要的数据停止，查找的时间复杂度为O(n)

来看下Skip List的数据结构是什么样的：

![img](https://img-blog.csdnimg.cn/20200106144831928.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



上图便为跳表的一种可能的数据结构，如果说我要查询15这个元素

![img](https://img-blog.csdnimg.cn/20200106144747745.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

有种类似于二分法查找的感觉，事实上没错，跳表的查询也是基于二分法查询的概念

所以跳表总结下来的性质有以下几点

(1) 由很多层结构组成，层数是通过一定的概率随机产生的；
(2) 每一层都是一个有序的链表，默认是升序 ；
(3) 最底层的链表包含所有元素；
(4) 如果一个元素出现在i层的链表中，则它在i层之下的链表也都会出现； 
(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。

接下来我们从源码来看

doput()获取方法

```java
private V doPut(K kkey, V value, boolean onlyIfAbsent) {
    Comparable<? super K> key = comparable(kkey);
    for (;;) {
        // 找到key的前继节点
        Node<K,V> b = findPredecessor(key);
        // 设置n为“key的前继节点的后继节点”，即n应该是“插入节点”的“后继节点”
        Node<K,V> n = b.next;
        for (;;) {
            if (n != null) {
                Node<K,V> f = n.next;
                // 如果两次获得的b.next不是相同的Node，就跳转到”外层for循环“，重新获得b和n后再遍历。
                if (n != b.next)
                    break;
                // v是“n的值”
                Object v = n.value;
                // 当n的值为null(意味着其它线程删除了n)；此时删除b的下一个节点，然后跳转到”外层for循环“，重新获得b和n后再遍历。
                if (v == null) {               // n is deleted
                    n.helpDelete(b, f);
                    break;
                }
                // 如果其它线程删除了b；则跳转到”外层for循环“，重新获得b和n后再遍历。
                if (v == n || b.value == null) // b is deleted
                    break;
                // 比较key和n.key
                int c = key.compareTo(n.key);
                if (c > 0) {
                    b = n;
                    n = f;
                    continue;
                }
                if (c == 0) {
                    if (onlyIfAbsent || n.casValue(v, value))
                        return (V)v;
                    else
                        break; // restart if lost race to replace value
                }
                // else c < 0; fall through
            }

            // 新建节点(对应是“要插入的键值对”)
            Node<K,V> z = new Node<K,V>(kkey, value, n);
            // 设置“b的后继节点”为z
            if (!b.casNext(n, z))
                break;         // 多线程情况下，break才可能发生(其它线程对b进行了操作)
            // 随机获取一个level
            // 然后在“第1层”到“第level层”的链表中都插入新建节点
            int level = randomLevel();
            if (level > 0)
                insertIndex(z, level);
            return null;
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

doRemove()方法

```java
final V doRemove(Object okey, Object value) {
    Comparable<? super K> key = comparable(okey);
    for (;;) {
        // 找到“key的前继节点”
        Node<K,V> b = findPredecessor(key);
        // 设置n为“b的后继节点”(即若key存在于“跳表中”，n就是key对应的节点)
        Node<K,V> n = b.next;
        for (;;) {
            if (n == null)
                return null;
            // f是“当前节点n的后继节点”
            Node<K,V> f = n.next;
            // 如果两次读取到的“b的后继节点”不同(其它线程操作了该跳表)，则返回到“外层for循环”重新遍历。
            if (n != b.next)                    // inconsistent read
                break;
            // 如果“当前节点n的值”变为null(其它线程操作了该跳表)，则返回到“外层for循环”重新遍历。
            Object v = n.value;
            if (v == null) {                    // n is deleted
                n.helpDelete(b, f);
                break;
            }
            // 如果“前继节点b”被删除(其它线程操作了该跳表)，则返回到“外层for循环”重新遍历。
            if (v == n || b.value == null)      // b is deleted
                break;
            int c = key.compareTo(n.key);
            if (c < 0)
                return null;
            if (c > 0) {
                b = n;
                n = f;
                continue;
            }

            // 以下是c=0的情况
            if (value != null && !value.equals(v))
                return null;
            // 设置“当前节点n”的值为null
            if (!n.casValue(v, null))
                break;
            // 设置“b的后继节点”为f
            if (!n.appendMarker(f) || !b.casNext(n, f))
                findNode(key);                  // Retry via findNode
            else {
                // 清除“跳表”中每一层的key节点
                findPredecessor(key);           // Clean index
                // 如果“表头的右索引为空”，则将“跳表的层次”-1。
                if (head.right == null)
                    tryReduceLevel();
            }
            return (V)v;
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 在有序的情况下

有序的情况下：

- 在非多线程的情况下，应当尽量使用TreeMap（红黑树实现）。
- 对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。但是对于高并发程序，应当使用ConcurrentSkipListMap。

无序情况下：

- 并发程度低，数据量大时，ConcurrentHashMap 存取远大于ConcurrentSkipListMap。
- 数据量一定，并发程度高时，ConcurrentSkipListMap比ConcurrentHashMap效率更高。

参考

原文:https://www.cnblogs.com/java-zzl/p/9767255.html

原文:https://www.jianshu.com/p/60d2561b821c