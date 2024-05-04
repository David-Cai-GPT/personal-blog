---
layout: post
title: spring事务控制
tag: 技术笔记
date: 2023-1-8
category: Technology
---

#### 简介

事务是恢复和并发控制的基本单位，其具备4个属性：原子性(atomicity)、一致性(consistency)、隔离性(isolation)、隔离性(durability)、持久性。这四个属性通常称为ACID特性。

原子性：一个事务是一个不可分割的工作单位，事务中包括的操作要么都做，要么都不做。

一致性：事务必须使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。

隔离性：一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

持久性：持久性也称为永久性，只一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。

#### **事务并发问题**

丢失更新：撤销一个事务时，把其他事务已提交的更新的数据覆盖了。

脏读：一个事务读到另一个事务未提交的更新数据。

不可重复读：一个事务两次都同一行数据，可是这两次读到的数据不一样。

幻读：一个事务执行两次查询，但第二次查询比第一次查询多出了一些数据行。

#### 隔离级别

Read Uncommitted:  读未提交数据，隔离级别最差，存在脏读、不可重复读、幻读的问题。

Read Committed:  读已提交数据，存在不可重复读，幻读的问题。

Repeatable Read:  可重复读，存在幻读的问题。

Serializable：串行化，问题都不存在。

#### 关键接口

PlatformTransactionManager:  事务管理器；

TransactionDefinition：事务的一些的基础信息，如超时时间、隔离级别、传播属性等；

TransactionStatus:  事务的一些状态信息，如是否是一个新的事务、是否已被标记为回滚。

#### 事务在Spring中是如何运作的

了解嵌套事务之前，可以先看下单个事务在spring中的处理流程，以便后面可以更清晰地认识嵌套事务的逻辑。

Spring事务使用AOP的机制实现，会在@Transactional注解修饰的方法前后分别织入开启事务的逻辑，以及提交和回滚的逻辑。

当出现多个事务嵌套的场景发生时，Spring事务的处理会变得复杂一下，需要考虑事务下的提交顺序，以及回滚顺序，对此，Spring提供了多种传播机制，每种传播机制的效果都不尽相同，以便应对各种复杂的业务场景。

嵌套事务处理流程如下：

主事务开启->主事务执行->子事务开启->子事务执行->子事务提交或回滚->主事务提交或回滚（根据事务传播机制配置来判断）。

#### Spring事务传播机制

事务的传播机制，主要是决定业务方法之间调用，事务应该如何处理。

required: 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务，默认值。

required_new: 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

supports: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

not_supports: 以非事务方式运行，如果但钱存在事务，则把当前事务挂起。

mandatory：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

never: 以非事务方式运行，如果当前存在事务，则抛出异常。

NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则等价于required。

#### 事务的几种实现方式

**基于xml配置方式的事务管理**

**基于注解方式的事务管理(*声明式事务*)**

　@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

​     虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

​    默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。

关于@Transactional注解中的默认值

只读属性为false，传播行为是TransactionDefinition.PROPAGATION_REQUIRED，隔离级别是TransactionDefinition.ISOLATION_DEFAULT，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。所以注解的默认值足够我们项目中使用，我们只用在类前面打上注解就可以了。

**基于编程式事务管理**

编程式事务使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。
 对于编程式事务管理，spring推荐使用TransactionTemplate。



引用

> https://www.cnblogs.com/qlqwjy/p/7296493.html
>
> https://blog.csdn.net/qq_18478183/article/details/121966561
>
> https://blog.csdn.net/szy350/article/details/122466050