---
layout: post
title: 桥接模式与JDBC
tag: 技术笔记
date: 2022-11-12
category: Technology
---

### 桥接模式

#### 定义与特点

桥接模式，是将抽象和实现解耦，将抽象和实现分离，使得两者可以独立地变化。它是用组合\聚合关系代替继承关系来实现，从而降低了抽象和实现的两个可变维度的耦合度；又或者说是一个类存在两个或多个独立变化的维度，我们通过组合的方式，让这两个或多个维度可以独立进行扩展。例如针对一个图形，我们可以设计颜色和形状两个变化维度。

由于抽象与实现分离，所以扩展能力很强， 实现细节对使用者来说是不需要过多关注的。

#### 几个关键的基本结构

**Abstraction**

抽象化角色：主要职责是定义出角色的行为，同时包含一个对实现化角色的引用

**Implementor**

实现化角色：它是接口或者抽象类，定义角色必需的具体行为和具体属性，供扩展化抽象角色使用

**Refined Abstraction**

扩展抽象化角色：抽象化角色的子类，实现父类的业务方法，并通过组合\聚合调用实现化角色中的业务方法

**Concrete Implemetor**

具体实现化角色：实现化角色的具体实现

#### 应用场景

- 当一个类存在两个独立变化的维度，且这两个维度都需要进行扩展时
- 当一个系统不希望使用集成或因为多层次集成导致系统类的个数急剧增加时
- 当一个系统需要在构件的抽象化角色和具体化角色之间增加过多的灵活性时

#### CODE DEMO

```java
/**
 * 实现化类
 */
public interface Implementor {

    /**
     * 定义一个方法
     */
    void doSomething();

}
```

```java
/**
 * 具体化实现类1
 */
public class ConcreteImplementor1 implements Implementor {

    public void doSomething() {
        //具体业务逻辑
        System.out.println("This is ConcreteImplementor1");
    }

}

/**
 * 具体化实现类2
 */
public class ConcreteImplementor2 implements Implementor{

    public void doSomething() {
        //具体业务逻辑
        System.out.println("This is ConcreteImplementor2");
    }

}
```

```java
/**
 * 抽象化角色
 */
public abstract class Abstraction {

    /**
     * 定义对实现化角色的引用
     */
    private Implementor implementor;

    public Abstraction(Implementor implementor){
        this.implementor = implementor;
    }

    /**
     * 自身的行为和属性
     */
    public void request(){
        this.implementor.doSomething();
    }

    /**
     * 获取实现化角色
     * @return
     */
    public Implementor getImplementor(){
        return implementor;
    }

}

```

```java
/**
 * 修正抽象化角色
 */
public class RefinedAbstraction extends Abstraction {

    /**
     * 覆写构造函数
     *
     * @param implementor
     */
    public RefinedAbstraction(Implementor implementor) {
        super(implementor);
    }

    /**
     * 修正父类的行为
     */
    @Override
    public void request() {
        super.request();
    }
}

```

```java
public class Main {
    public static void main(String[] args) {
        // 定义一个实现化角色
        Implementor implementor1 = new ConcreteImplementor1();
        Implementor implementor2 = new ConcreteImplementor2();

        // 定义一个抽象化角色
        Abstraction abstraction1 = new RefinedAbstraction(implementor1);
        // 执行方法
        abstraction1.request();

        // 定义一个抽象化角色
        Abstraction abstraction2 = new RefinedAbstraction(implementor2);
        // 执行方法
        abstraction2.request();
    }
}

运行结果：
This is ConcreteImplementor1
This is ConcreteImplementor2
```

#### JDBC

通过原生JDBC API连接MySQL数据库，则有如下的示例代码

如果我们想要把 MySQL 数据库换成 Oracle 数据库，只需要把第一行代码中的 com.mysql.cj.jdbc.Driver 换成oracle.jdbc.driver.OracleDriver 就可以了。

```java
//要求JVM查找并加载指定的Driver类
Class.forName("com.mysql.cj.jdbc.Driver");
Connection conn = DriverManager.getConnection("jdbc:mysql://<host>:<port>/<database>");
```

**Class.forName()方法**

该方法将返回与给定字符串名的类或接口相关联的java.lang.Class类对象，用于在程序运行时的某个时刻，由客户端调用，动态加载该类或该接口到当前线程中

```java

    /**
     * Returns the {@code Class} object associated with the class or
     * interface with the given string name. Given the fully qualified
     * name for a class or interface this method attempts to locate, load,
     * and link the class or interface.
     */
    @CallerSensitive
    public static Class<?> forName(String className)
                throws ClassNotFoundException {
        Class<?> caller = Reflection.getCallerClass();
        return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
    }
```

**com.mysql.cj.jdbc.Driver类**

MySQL将具体的java.sql.Driver接口的实现放到了NonRegisteringDriver中，com.mysql.cj.jdbc.Driver类仅包含一段静态代码，具体类图如下：

```java
package com.mysql.cj.jdbc;

import java.sql.DriverManager;
import java.sql.SQLException;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    //静态代码，将MySQL Driver注册到DriverManager类中
    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}

```

```java
public static synchronized void registerDriver(java.sql.Driver driver,
            DriverAction da)
        throws SQLException {

        /* Register the driver if it has not already been added to our list */
        if(driver != null) {
            //registeredDrivers静态字段的类型是实现了List接口的CopyOnWriteArrayList类，它能够保存进一步封装java.sql.Driver接口的DriverInfo类实例
            registeredDrivers.addIfAbsent(new DriverInfo(driver, da));
        } else {
            // This is for compatibility with the original DriverManager
            throw new NullPointerException();
        }

        println("registerDriver: " + driver);
    }
```

```java
class DriverInfo {

    final Driver driver;
    DriverAction da;
    DriverInfo(Driver driver, DriverAction action) {
        this.driver = driver;
        da = action;
    }

    @Override
    public boolean equals(Object other) {
        return (other instanceof DriverInfo)
                && this.driver == ((DriverInfo) other).driver;
    }

    @Override
    public int hashCode() {
        return driver.hashCode();
    }

    @Override
    public String toString() {
        return ("driver[className="  + driver + "]");
    }

    DriverAction action() {
        return da;
    }
}
由上面的分析可得，Class.forName()方法调用后，com.mysql.cj.jdbc.Driver类被加载，并执行static { } 静态代码段，将com.mysql.cj.jdbc.Driver类实例注册到DriverManager中。然后，客户端会调用DriverManager.getConnection()方法获取一个Connection数据库连接实例，该方法的部分源码如下：
```

```java
private static Connection getConnection(String url, java.util.Properties info, Class<?> caller) throws SQLException {
  // ……
  //遍历registeredDrivers静态字段，获取字段内保存的每一个Driver来尝试响应客户端的数据库连接请求，若所有Driver都连接数据库失败，则提示连接失败信息
  for(DriverInfo aDriver : registeredDrivers) {
    // If the caller does not have permission to load the driver then
    // skip it.
    if(isDriverAllowed(aDriver.driver, callerCL)) {
      try {
        println(" trying " + aDriver.driver.getClass().getName());
        Connection con = aDriver.driver.connect(url, info);
        if (con != null) {
          // Success!
          println("getConnection returning " + aDriver.driver.getClass().getName());
          return (con);
        }
      } catch (SQLException ex) {
        if (reason == null) {
          reason = ex;
        }
      }
    } else {
      println(" skipping: " + aDriver.getClass().getName());
    }
  }
  // ……
}
```

桥接模式中的实现化(Implementor)角色对应Driver接口，

```java
public interface Driver {

   Connection connect(String url, java.util.Properties info)
        throws SQLException;

   boolean acceptsURL(String url) throws SQLException;

   DriverPropertyInfo[] getPropertyInfo(String url, java.util.Properties info)
                         throws SQLException;

   int getMajorVersion();

   int getMinorVersion();
    
   boolean jdbcCompliant();

    //------------------------- JDBC 4.1 -----------------------------------
   public Logger getParentLogger() throws SQLFeatureNotSupportedException;
```

具体实现化(Concrete Implementor)角色对应MysqlDriver、OracleDriver和MariadbDriver，扩展抽象化 (Refined Abstraction)角色对应DriverManager，不具有抽象化(Abstraction)角色作为扩展抽象化角色的父类

桥接模式的主要应用场景是某个类存在两个独立变化的维度，且这两个维度都需要进行扩展，而现在仅有Driver一个变化维度，DriverManager没有抽象化父类，它本身也没有任何子类，因此，在JDBC中，是一种**简化的桥接模式**。



> 引用

https://www.cnblogs.com/kuluo/p/13038076.html

https://jishuin.proginn.com/p/763bfbd68968