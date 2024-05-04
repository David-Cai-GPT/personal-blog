---
layout: post
title: 从源码开始手撸红黑树（构建与插入）！
tag: 数据结构
date: 2020-2-2
category: Technology blog
---

红黑树（Red Black Tree）是一种自平衡二叉查找树，是在计算机科学中用到的一种数据结构，是一个有限节点组成的一个具有层次关系的集合，数据就存在这些节点中。

### 树

首先我们先来了解一下树（Tree），root是根节点，在分支处有一个节点，指向多个方向，如果某节点下方没有任何分叉的话，就是叶子节点，从某个节点出发，到叶子节点位置，最长简单路径上边的条数，成为该节点的高度，从根节点出发，到某节点边的条数名称为该节点的深度。![img](https://img-blog.csdnimg.cn/20191231104336153.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 平衡二叉树

如下图，如果一个树的节点都在右边，这个也是树，但是虽说是“树”，但实际使用与“链表”没有区别，像这样以树的复杂结构来实现简单的链表功能，则是完全埋没了输的特点，因此，对于树的使用，还需要进行某种条件的约束，如图，让链表一样的树变得更有层次结构，平衡二叉树就呼之欲出了，而左右树的高度差便是一棵树是否为平衡二叉树的决定条件

![img](https://img-blog.csdnimg.cn/20191231110228871.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 二叉查找树

二叉查找树又叫二叉搜索树，即Binary Search Tree，其中Search也可以替换为Sort，也叫做二叉排序树，非常擅长数据查找，但在了解二叉查找树之前，我们先来了解一下二分查找，首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。

​           ![img](https://img-blog.csdnimg.cn/20191231111358437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对于二叉查找树来说，它的左子树上所有节点的值都小于它，而它的右子树上所有节点都大于它，查找过程便是从树的根节点开始，沿着简单的判断向下走，小于节点值的往左边走，大于的则往右边走，直到找到目标数据或者到达叶子节点还没有找到

### AVL树

AVL树是一种平衡二叉查找树，增加和删除节点后通过树形旋转重新达到平衡。

假设待左旋的结构中，P为父节点，S为孩子节点。左旋操作后，S节点代替P节点的位置，P节点成为S节点的左孩子，S节点的左孩子成为P节点的右孩子。

假设待右旋的结构中，P为父节点，S为孩子节点。右旋操作后，S节点代替P节点的位置，P节点成为S节点的右孩子，S节点的右孩子成为P节点的左孩子。

AVL树就是不断通过旋转来达到输的平衡的，口头上说明可能比较难以理解

![img](https://img-blog.csdnimg.cn/20191231113252621.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20191231113603937.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

通过不断地旋转从不平衡到平衡，从而达到最好的查询效率。

### 红黑树

红黑树是于1972年发明的，当时称为对称二叉B树，1978年得到优化，正式命名为红黑树。他的特征主要就是每个节点上增加一个属性来表示结点的颜色，可以是红色，也可以是黑色。

红黑树类似于AVL树，都是在进行插入和删除元素的时候，通过特定的而旋转来保持自身的平衡，获得较高的查找性能

红黑树的约束条件：

(1)节点只能是红色或黑色

(2)根节点必须是黑色的

(3)所有NIL节点都是黑色的（NIL：Nothing In Leaf即在叶子节点上不存在的两个虚拟节点）

(4)一条路径上不能出现相邻的两个红色节点

(5)在任何递归树内，根节点到叶子节点的所有路径上包含相同数目的黑色节点

下图为一红黑树图例，为了方便后续解释，我们先定义了节点以下几个叫法：

![img](https://img-blog.csdnimg.cn/20200103093531263.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们就直接上源码了

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        // 指向左子树的引用
        Entry<K,V> left;
        // 指向右子树的引用
        Entry<K,V> right;
        // 指向父节点的引用
        Entry<K,V> parent;
        // 节点的颜色信息，红黑树的精髓，默认为黑色
        boolean color = BLACK;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们通过put()和deleteEntry()方法实现红黑树节点的增加和删除，这里着重从源码介绍增加的方法，再插入新节点之前，需要明确三个前提条件：

(1)需要调整的新节点总是红色的

(2)如果插入新节点的父节点是黑色的，则无需调整

(3)如果插入新节点的父节点是红色的，因为红黑树不能出现相邻的两个红色节点，所以进入循环判断，或重新着色，或左右旋转，最终达到红黑树的五个约束条件，退出条件如下：

```java

while( x != null && x != root && x.parent.color == RED) {...}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果是根节点，直接设置为黑色退出即可，如果不是，并且其父节点为红色，会一直调整，知道退出循环

```java
public V put(K key, V value) {
        // t表示当前节点！先把TreeMap的根节点root引用复制给当前节点
        Entry<K,V> t = root;
        // 如果当前节点是null，即时空树，新增的KV形成的节点就是根节点
        if (t == null) {
            // 看似多此一举，其实预检了Key是否可以作比较
            compare(key, key); // type (and possibly null) check
            // 使用KV构造出新的Entry对象，其中第三个参数时parent，根节点没有父节点
            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        // cmp用来接收比较结果
        int cmp;
        Entry<K,V> parent;
        // 构造方法中置入外部比较器
        Comparator<? super K> cpr = comparator;
        // 根据二叉查找树的特性，找到新结点插入的合适位置
        if (cpr != null) {
        // 根据参数Key与当前节点的Key不断地进行比较
            do {
                // 当前节点赋值给父节点，故从根节点开始遍历比较
                parent = t;
                // 比较输入的参数Key和当前节点Key的大小
                cmp = cpr.compare(key, t.key);
                // 参数的Key更小，向左边走，把当前节点引用移动至它的左子节点上
                if (cmp < 0)
                    t = t.left;
                // 参数的Key更大，向右边走，把当前节点引用移动至它的右子节点上
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
                // 如果没有相等的Key，一直会遍历到NIL节点为止
            } while (t != null);
        }
        // 在没有指定比较器的情况下，调用自然排序的Cpmparable进行比较
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else  
                    return t.setValue(value);
            } while (t != null);
        }

        // 创建Entry对象，并把parent置入参数
        Entry<K,V> e = new Entry<>(key, value, parent);
        // 新节点找到自己的位置，原本以为可以安顿下来
        if (cmp < 0)
            // 如果比较结果小于0，则成为parent的左孩子
            parent.left = e;
        else
            // 如果比较结果大于0，则成为parent的右孩子
            parent.right = e;
        // 还需要对这个新节点进行重新找色和旋转操作，以达到平衡
        fixAfterInsertion(e);
        // 插入结束
        size++;
        modCount++;
        // 返回null
        return null;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

看完这个源码过后，大家想必想知道fixAfterInsertion()这个方法，首先红黑树插入的情况分为三种

插入节点的父节点和舅舅节点都是红色的，解决办法是将插入节点的父节点和舅舅结点都着为黑色，而将插入节点的祖父结点着为红色，然后从祖父结点继续向上判断是否破坏红黑树的性质。处理过程如下图所示：

![img](https://img-blog.csdnimg.cn/2020010310170964.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

插入节点的父节点是红色的和但舅舅节点是黑色的需要判断一下

父节点是祖父节点的**左孩子**，若插入节点是父节点的**左孩子**，祖父节点为轴右旋转即可，(父节点置黑，祖父节点置红)

![img](https://img-blog.csdnimg.cn/20200103103655505.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

父节点是祖父节点的**左孩子**，若插入节点是父节点的右**孩子**，那么父节点为轴先左旋转，然后再以祖父结点为轴**右旋转**(插入点置黑，祖父节点置红)

![img](https://img-blog.csdnimg.cn/20200103105315403.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

父节点是祖父节点的**右孩子**，若插入节点是父节点的右**孩子**，祖父节点为轴左旋转即可，(父节点置黑，祖父节点置红)

![img](https://img-blog.csdnimg.cn/20200103111713895.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

父节点是祖父节点的**右孩子**，若插入节点是父节点的左**孩子**，那么父节点为轴先右旋转，然后再以祖父结点为轴**左旋转**(插入点置黑，祖父节点置红)

![img](https://img-blog.csdnimg.cn/20200103123816182.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

接下来让我们看看代码

```java
private void fixAfterInsertion(Entry<K,V> x) {
        // 尽管Entry中默认为黑色，但新节点一律先赋值为红色
        x.color = RED;
        // 如果x为null或x为根结点或x的父节点为黑色则结束循环
        while (x != null && x != root && x.parent.color == RED) {
        // 如果父亲是其父亲（祖父节点）的左子节点
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                // 定义出y为祖父节点的右子节点(舅舅节点)
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                // 如果舅舅节点为红色
                if (colorOf(y) == RED) {
                    // 父亲节点置为黑色
                    setColor(parentOf(x), BLACK);
                    // 舅舅节点置为黑色
                    setColor(y, BLACK);
                    // 祖父节点置为红色
                    setColor(parentOf(parentOf(x)), RED);
                    // 爷爷成为新的节点，继续迭代
                    x = parentOf(parentOf(x));
                    // 如果其舅舅节点为黑色
                } else {
                    // 如果插入节点是父亲的右子节点，相对父亲进行一次左旋操作
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }
                    // 重新着色并对爷爷进行右旋操作
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateRight(parentOf(parentOf(x)));
                }
                    //和上面的代码是对称的。这里判断x是否是其右孩子
            } else {
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        //最后将root节点的颜色设置为红色，这里可以参考情况二如果，其爷爷节点是根节点则，循环结束，最后需要将根节点设置为黑色，都没有违反性质4和性质5
        root.color = BLACK;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这里再讲解一下左旋代码

```java
private void rotateLeft(Entry<K,V> p) {
    // 如果参数不是NIL节点
	if(p != null) {
        // 获取P的右子节点r
		Entry<K,V> r = p.right;
        // 将r的左子树设置为p的右子树
		p.right = r.left;
        // 若r的左子树不为空，则将p设置为r左子树的父亲
		if(r.left != null)
			r.left.parent = p;
        // p的父亲设置为r的父亲
		r.parent = p.parent;
		if(p.parent == null)
			root = r;
		else if(p.parent.left == p)
			p.parent.left = r;
		else
			p.parent.right = r;
	// 将p设置为r的左子树，将r设置为p的父亲
	r.left = p;
	p.parent = r;
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

右旋代码与左旋代码类似，这里就不再做讲解了。

参考学习：《码出高效Java开发手册》--杨冠宝 高慧沙著

​          blog：你真的了解二分查找吗？https://www.jianshu.com/p/cddfbdbc53c7