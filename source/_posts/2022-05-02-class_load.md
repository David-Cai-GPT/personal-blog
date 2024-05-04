---
layout: post
title: 类加载机制
tag: 技术笔记
date: 2022-5-2
category: Technology
---

#### 类的生命周期

首先我们来了解一下类的整个生命周期

加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载

![](类加载流程.PNG)

其中前五步为类的加载过程



##### 加载

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的**静态存储结构**转换为方法区的**运行时数据结构**
3. 在内存中生成一个代表该类的Class对象，作为方法区数据的访问入口

一个**非数组类的加载阶段**（加载阶段获取类的二进制字节流的动作）是**可控性最强的阶**段，这一步我们可以去完成还可以**自定义类加载器**去控制字节流的获取方式（重写一个类加载器的 `loadClass()` 方法）。

数组类型不通过类加载器创建，它由 Java 虚拟机直接创建。

加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。



##### 验证

验证主要为了保证类不会对java虚拟机运行造成破环

主要包含了：

**文件格式验证**：验证字节流是否符合Class文件格式的规范

**元数据验证**：对字节码描述的信息进行语义分析

**字节码验证**：通过数据流和控制流分析，确定程序语义是合法的，符合逻辑的

**符号引用验证**：确保解析动作能正常进行



##### 准备

准备阶段是正式为类变量**分配内存**并**设置类变量初始值**的阶段，这些内存都将在方法区中分配。

1. 这时候进行内存分配的仅包括类变量（ Class Variables ，即静态变量，被 `static` 关键字修饰的变量，只与类相关，因此被称为类变量），而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在 Java 堆中。
2. 从概念上讲，类变量所使用的内存都应当在方法区中进行分配。不过有一点需要注意的是：JDK 7 之前，HotSpot 使用永久代来实现方法区的时候，实现是完全符合这种逻辑概念的。 而在 JDK 7 及之后，HotSpot 已经把原本放在永久代的字符串常量池、静态变量等移动到堆中，这个时候类变量则会随着 Class 对象一起存放在 Java 堆中。



##### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接应用的过程

解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行



##### 初始化

初始化阶段是执行初始化方法<client>()方法的过程，是类加载的最后一步，这一步JVM才开始真正执行类中定义的Java程序代码

1. 当遇到 `new` 、 `getstatic`、`putstatic` 或 `invokestatic` 这 4 条直接码指令时，比如 `new` 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。

   当 jvm 执行 `new` 指令时会初始化类。即当程序创建一个类的实例对象。

   当 jvm 执行 `getstatic` 指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。

   当 jvm 执行 `putstatic` 指令时会初始化类。即程序给类的静态变量赋值。

   当 jvm 执行 `invokestatic` 指令时会初始化类。即程序调用类的静态方法。
   
2. 使用 `java.lang.reflect` 包的方法对类进行反射调用时如 `Class.forname("...")`, `newInstance()` 等等。如果类没初始化，需要触发其初始化。

3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。



##### 卸载

卸载类即该类的Class对象被GC

1. 该类的所有的实例对象都已被 GC，也就是说堆不存在该类的实例对象。
2. 该类没有在其他任何地方被引用
3. 该类的类加载器的实例已被 GC

所以，在 JVM 生命周期内，由 jvm 自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

只要想通一点就好了，jdk 自带的 `BootstrapClassLoader`, `ExtClassLoader`, `AppClassLoader` 负责加载 jdk 提供的类，所以它们(类加载器的实例)肯定不会被回收。

而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的。



#### 类加载顺序

##### 双亲委派机制

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序，自顶向下尝试加载类

**Bootstrap ClassLoader**：Load JRE\lib\rt.jar包

**Extension ClassLoader**：Load JRE\LIB\ext\*.jar包

**App ClassLoader**：Load CLASSPATH或-Djava.class.path所指定的目录下的类和jar包

**Custom ClassLoader**：通过java.lang.ClassLoader的子类自定义加载class



1. 加载过程中会先检查类是否被已加载，**检查顺序是自底向上**，从Custom ClassLoader到BootStrap ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。而**加载的顺序是自顶向下**，也就是由上层来逐层尝试加载此类。 
2. 在加载类时，每个类加载器会将加载任务上交给其父，如果其父找不到，再由自己去加载。 
3. Bootstrap Loader（启动类加载器）是最顶级的类加载器了，其父加载器为null。

![](类加载顺序.PNG)

**但双亲委派机制有一个缺陷**

通过双亲委派机制的原理可以得出一下结论：由于BootstrapClassloader是顶级类加载器，BootstrapClassloader无法委派AppClassLoader来加载类，也就是说BootstrapClassloader中加载的类中无法使用由AppClassLoader加载的类。通常情况下，启动类加载器中的类为系统核心类，包括一些重要的系统接口，而在应用类加载器中，为应用类。按照这种模式，`应用类访问系统类自然是没有问题，但是系统类访问应用类就会出问题。`比如在系统类中提供了一个接口，该接口需要在应用中得以实现，该接口还绑定一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，就会出现该工厂方法无法创建由应用类加载器加载的应用实例的问题。可能绝大部分情况这个不算是问题，因为BootstrapClassloader加载的都是基础类，供AppClassLoader加载的类调用的类。但是万事万物都不是绝对的比如经典的JAVA SPI机制。

Tomcat的类加载机制也是违反了双亲委托原则的，对于一些未加载的非基础类(Object,String等)，各个web应用自己的类加载器(WebAppClassLoader)会优先加载，加载不到时再交给Common ClassLoader走双亲委托。

![](tomcat类加载流程.PNG)

当 Tomcat 使用 WebAppClassLoader 进行类加载时，具体过程如下

（1）先在本地 cache 缓存中查找该类是否已经加载过，看看 Tomcat 有没有加载过这个类

（2）如果 Tomcat 没有加载过这个类，则从系统类加载器的 cache 缓存中查找是否加载过

（3）如果没有，则使用 ExtClassLoader 类加载器类加载，重点来了，Tomcat 的 WebAppClassLoader 并没有先使用 AppClassLoader 来加载类，而是直接使用了 ExtClassLoader 来加载类。不过 ExtClassLoader 依然遵循双亲委派，它会使用 Bootstrap ClassLoader 来对类进行加载，保证了 Jre 里面的核心类不会被重复加载。

比如在 Web 中加载一个 Object 类。WebAppClassLoader → ExtClassLoader → Bootstrap ClassLoader，这个加载链，就保证了 Object 不会被重复加载。

（4）如果没有加载成功，WebAppClassLoader 就会调用自己的 findClass() 方法由自己来对类进行加载，先在 WEB-INF/classes 中加载，再从 WEB-INF/lib 中加载。

（5）如果仍然未加载成功，WebAppclassLoader 会委派给 SharedClassLoader，SharedClassLoad 再委派给 CommonClassLoader，CommonClassLoader 委派给 AppClassLoader，直到最终委派给 BootstrapClassLoader，最后再一层一层地在自己目录下对类进行加载。

（6）都没有加载成功的话，抛出异常。

​    **WebAppClassLoader 加载类的时候，故意打破了JVM 双亲委派机制，绕开了 AppClassLoader，直接先使用 ExtClassLoader 来加载类。最主要原因是保证部署在同一个 Web 容器上的不同 Web 应用程序所使用的类库可以实现相互隔离，避免不同项目的相互影响**



> [类加载过程](https://www.cnblogs.com/Vincent-yuan/p/15478341.html)
>
> [Java中类的加载顺序介绍(ClassLoader)](https://blog.csdn.net/weixin_37766296/article/details/80545283)
>
> [双亲委派机制的优势与劣势](https://www.jianshu.com/p/7243edfece1a)
>
> [tomcat类加载机制](https://blog.csdn.net/a745233700/article/details/120802616)