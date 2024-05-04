---
layout: post
title: 单例模式
tag: 技术笔记
date: 2022-8-7
category: Technology
---

当我们提到单例模式，首先想到的是什么

**指在内存中只会创建且仅创建一次对象的设计模式**。在程序中多次使用同一个对象且作用相同的时候，为了防止频繁地创建对象使得内存飙升，**单例模式可以让程序尽在内存中创建一个对象，所有需要调用的地方都共享这一单例对象**。

单例模式创建的方式有两种

**懒汉式**：在需要的时候才去创建该单例类对象

```java
public class Singleton {

    /**
     * 单例对象
     */
    private static Singleton singleton;

    /**
     * 私有无参构造函数
     */
    private Singleton(){}

    /**
     * 静态创建方法
     * @return 实例
     */
    public static Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

**饿汉式**：在类加载的时候就以及创建好单例对象，等待被程序使用

```java
public class Singleton {

    /**
     * 单例对象
     */
    private static final Singleton singleton = new Singleton();

    /**
     * 私有无参构造函数
     */
    private Singleton(){}

    /**
     * 静态创建方法
     * @return 实例
     */
    public static Singleton getInstance(){
        return singleton;
    }
}
```

**懒汉式**创建方法有一个问题，就是多线程并发的时候有可能会创建多个对象，举个简单例子，A线程和B线程基本同时经过判空判断，这个时候就会创建两个实例，这里我们需要解决一下**线程安全**问题。

可以使用synchronized修饰获取实例方法或者当前类对象。

```java
public static synchronized Singleton getInstance(){
    if(singleton == null){
        singleton = new Singleton();
    }
    return singleton;
}


public static Singleton getInstance() {
    synchronized (Singleton.class) {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

这样每次获取对象的时候都要先获取锁，开销比较大，我们应该在判断没有实例的情况下，创建新的实例对象操作加锁是最理想的。

```java
public class Singleton {

    private static Singleton singleton;

    private Singleton(){}

    public static Singleton getInstance() {
        if(singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

这里有个双重校验的过程，线程A在获取锁后需要再次校验，应为有可能上个获取锁的线程以及实例化了对象，所以需要两次判空。

上面的代码基本上在逻辑层面已经没有什么问题了，但是还存在最后一个问题：**指令重排**

**JVM在保证最终结果正确的情况下，可以不按照程序编码的顺序执行语句，尽可能的提高程序的性能**

```java
singleton = new Singleton();
```

会被编译成如下JVM命令

```c
memory = allocate();   //分配对象的内存空间
ctorInstance(memory);  //初始化对象
instance = memory;  //设置instance指向刚分配的内存地址

#但这些指令指令顺序并不是一成不变的，有可能经过优化的情况下，可指令重排成下面的排序

memory = allocate();   //分配对象的内存空间
instance = memory;  //设置instance指向刚分配的内存地址
ctorInstance(memory);  //初始化对象
```

这里就会出现一个问题，当A线程执行完第一步和第二步时，虽然对象还没有初始化，但是它已不再指向null，这个时候线程B抢占到CPU资源，执行判空判断就会false，从而返回一个没有初始化完成的singleton对象。

这里我们使用volatile关键字可以防止指令重排序，**使用volatile关键字修饰的变量，可以保证其指令执行的顺序与程序指明的顺序一致，不会发生顺序变换**，当然volatile作用不只于此，这里不再展开。

```java
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton(){}

    public static Singleton getInstance() {
        if(singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

使用静态内部类实现单例模式，会更加巧妙

```java
public class Singleton {

    private static class LazyHolder {
        private static final Singleton singleton = new Singleton();
    }

    private Singleton(){};

    public static Singleton getInstance(){
        return LazyHolder.singleton;
    }
}
```

从外部无法访问静态内部类LazyHolder，只有当调用Singleton.getInstance方法的时候，才能得到单例对象singleton，singleton实例的初始化时机也不是在Singleton被加载的时候，而是在调用getInstance()方法，使用的静态内部类LazyHolder被加载的时候，**利用classLoader的加载机制来实现懒加载，并且保证了单例的线程安全问题。**

但是单例模式不管怎么实现，还存在着一个通病，就是**无法防止利用反射来重复构建对象**

```java
public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    //获取构造器
    Constructor constructor = Singleton.class.getDeclaredConstructor();
    //设置为可访问
    constructor.setAccessible(true);

    Singleton singleton1 = (Singleton) constructor.newInstance();
    Singleton singleton2 = (Singleton) constructor.newInstance();

    System.out.println(singleton1.equals(singleton2));
}

#结果：flase
```

所以怎么写出简介无懈可击的单例模式呢，用枚举来实现

```java
public enum SingletonEnum {
	SINGLETON;
}	
```

我们再次运行刚刚的代码

```plain
Exception in thread "main" java.lang.NoSuchMethodException: Example.Singleton.Singleton.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.getDeclaredConstructor(Class.java:2178)
	at Example.Singleton.testsingleton.main(testsingleton.java:9)
```

使用枚举实现的单例模式，不但可以防止利用反射强行构建单例对象，而且可以在枚举类对象被**反序列化**的时候，保证反序列的返回结果是同一对象。而对于其他方式实现的单例模式，则必须实现readResolve方法



引用

> https://mp.weixin.qq.com/s/1fQkkdtzYh_OikbYJnmZWg