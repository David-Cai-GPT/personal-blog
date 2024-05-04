---
layout: post
title: 继承，多态，装饰者模式
tag: 技术笔记
date: 2020-8-23
category: Technology blog
---

### 继承，多态与装饰者模式
在Java开发中，继承、多态、封装与抽象作为面向对象的四个基本特征，而装饰者模式则是属于开发的设计模式之一，在实际的代码实现当中，可以发现其实它们有很多相似的地方，将它们灵活运用在软件的开发当中，能够使项目开发效率更高且更易于维护，所以单在这里写一篇总结来探讨一下它们各自的优势与区别。
#### 继承
继承是java面向对象编程技术的一块基石，因为它允许创建分等级层次的类。
继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

在继承关系中，父类更通用，子类则更加具体，父类具有更加一般的特征和行为，而子类除了具有父类的特征和行为，还具有一些自己的特征和行为

![批注 2020-08-23 091733](批注 2020-08-23 091733.png)

这里我们举个例子，如上图，兔和山羊属于食草类动物，而老虎和狮子则属于食肉类动物，食草类动物和食肉类动物又都属于动物类，他们属于一层一层归属的关系，虽然都是归属于动物类，但是食草动物与食肉动物在属性和行为上是有区别的，所有子类会具有父类的一般特性同时也会具有自身的特性。

* 子类通过extends来继承父类，extends的意思是扩展，意思是子类是父类的扩展
* Java中的类只能单继承，不能多继承，（这里是和implement实现的区别），支持多重继承
* 继承是类和类之间的一种关系，除此之外，类和类之间的关系还有依赖，组合，聚合等等
* 子类和父类之间，从意义上讲应该具有“is a”的关系

接下来我们来看下具体代码的实现
```java
//动物基类
public class animal {
    private String name;
    private String gender;

    public animal(String myName,String myGender){
        name = myName;
        gender = myGender;
    }

    public String eat(){
        return name + "正在吃";
    }

    public String sleep(){
        return name + "正在睡";
    }

    public String introduction(){
        return "大家好，我是" + name + "我的性别是：" + gender;
    }

    public String desc(){
        return "这是子类没有重写的父类方法";
    }
}
```
```java
//山羊类
public class goal extends animal{
    public goal(String myName, String myGender) {
        super(myName, myGender);
    }

    @Override
    public String eat() {
        //相比动物类，山羊类便更加详细，所以我们这里重写父类方法
        return super.eat() + "草";
    }

    @Override
    public String sleep() {
        return super.sleep() + "在内蒙古大草原";
    }

    @Override
    public String introduction() {
        return super.introduction() + "，我是草食动物";
    }
}
```
```java
//狮子类
public class lion extends animal {
    public lion(String myName, String myGender) {
        super(myName, myGender);
    }

    @Override
    public String eat() {
        return super.eat() + "肉";
    }

    @Override
    public String sleep() {
        return super.sleep() + "在非洲热带雨林";
    }

    @Override
    public String introduction() {
        return super.introduction() + "，我是肉食动物";
    }
    //子类独有的特性
    public String kingOfAnimal(){
        return "我是万兽之王";
    }
}
```
#### 多态

多态是用一个行为具有多个不同表现形式或形态的能力，多态就是用一个接口，使用不同的实例而执行不同操作

写一个方法，它只接收父类作为参数，编写的代码也只与父类打交道，调用这个方法时，实例化不同的子类对象

1. 子类重写父类的方法，是子类具有不同的方法实现
2. 把父类类型作为参数类型，该弗雷及其子类作为参数转入
3. 运行时，根据实际创建的对象类型形态觉得使用那个方法

我们来看下代码


```java
public class main {
    public static void main(String[] args) {
        goal g = new goal("山羊", "公");
        lion l = new lion("狮子", "母");
        System.out.println(g.eat());
        System.out.println(g.sleep());
        System.out.println(g.introduction());
        System.out.println(l.eat());
        System.out.println(l.sleep());
        System.out.println(l.introduction());
        System.out.println(l.kingOfAnimal());//调用子类特有方法
        animal l1 = new lion("狮子", "母"); //向上造型
        System.out.println(l1.desc()); //调用子类没有重写的方法默认调用父类
    }
}

public class main {

    public static void main(String[] args) {
        show(new Goal());
        show(new Lion());
        Animal a = new Goal(); //向上转型
        System.out.println(a.eat());
        Goal g = (Goal)a;//向下转型
        System.out.println(g.desc());
    }


    //首先建立抽象类与抽象方法
    static abstract class Animal {
        abstract String eat();
    }

     static class Goal extends Animal{
        @Override
        public String eat(){
            return "吃草";
        }

        public String desc(){
            return "山羊";
        }
    }

    static class Lion extends Animal{

        @Override
        public String eat() {
            return "吃肉";
        }

        public String desc(){
            return "狮子";
        }
    }

    public static void show(Animal a)  {
        System.out.println(a.eat());
        // 类型判断
        if (a instanceof Goal)  {  // 类型为山羊
            Goal g = (Goal)a;
            System.out.println(g.desc());
        } else if (a instanceof Lion) { // 类型为狮子
            Lion l = (Lion)a;
            System.out.println(l.desc());
        }
    }

}

输出：
吃草
山羊
吃肉
狮子
吃草
山羊
```

#### 装饰者模式

装饰者模式，允许向一个现有的对象添加新的功能，同时又不改变其结构，这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种模式创建了一个装饰类，用来包装原有的类，并在保持类的方法签名完整性的前提下，提供了额外的功能。能够动态的给一个对象添加一些额外的职责，就增加功能来说的话，装饰者模式相比生成子类更加灵活。

这里用咖啡店来举个例子

咖啡馆为了吸引更多的顾客，需要在订单系统中允许顾客选择加入不同调料的咖啡，例如：蒸奶（Steamed Milk）、豆浆（Soy）、摩卡（Mocha，也就是巧克力风味）或覆盖奶泡。会根据所加入的调料收取不同的费用。所以订单系统必须考虑到这些调料部分。如果每种饮料搭配都单独创建一个类，那么必定会有类的数量十分庞大的情况出现，这种设计肯定是不行的

所以在这里我们引入装饰者模式

```java
//这里先定义一个饮料的抽象类
public abstract class Beverage {
    //描述是未知饮料
    String desc = "Unknown Beverage";

    public String getDesc(){
        return desc;
    }
    //这里定义一个得到价格的抽象方法，具体类中自己来实现
    public abstract BigDecimal cost();
}
```

```java
//深焙咖啡类，一种具体的饮料
public class DarkRoast extends Beverage{
    public DarkRoast(){
        desc = "DarkRoast";
    }

    @Override
    public BigDecimal cost() {
        return new BigDecimal("3.00");
    }
}
```

```java
//低咖啡因咖啡类，一种具体的饮料
public class Decaf extends Beverage{
    public Decaf(){
        desc = "Decaf";
    }

    @Override
    public BigDecimal cost() {
        return new BigDecimal("4.00");
    }
}
```

```java
//浓缩咖啡类，一种具体的饮料
public class Espresso extends Beverage{
    public Espresso(){
        desc = "Espresso";
    }

    @Override
    public BigDecimal cost() {
        return new BigDecimal("2.00");
    }
}
```

```java
//这里实现调料装饰者的抽象类
public abstract class CondimentDecoractor extends Beverage{
    //所有的调料装饰者都必须重新实现getDescription()方法
    //这样才能够用递归的方式来得到所选饮料的具体描述
    public abstract String getDesc();
}
```

```java
//摩卡调料类（继承自CondimentDecorator）
public class Mocha extends CondimentDecoractor{
    //用一个实例变量记录饮料，也就是被装饰者
    Beverage beverage;

    public Mocha(Beverage beverage){
        this.beverage = beverage;
    }
    //在原来饮料的基础上添加上Mocha描述（原来的饮料加入Mocha调料，被Mocha调料装饰）
    @Override
    public String getDesc() {
        return beverage.getDesc() + ",Mocha";
    }
    //在原来饮料的基础上加上Mocha的价格（原来的饮料加入Mocha调料，被Mocha调料装饰）
    @Override
    public BigDecimal cost() {
        return new BigDecimal("0.2").add(beverage.cost());
    }
}
```

```java
//豆浆调料类（继承自CondimentDecorator）
public class Soy extends CondimentDecoractor {
    //用一个实例变量记录饮料，也就是被装饰者
    Beverage beverage;

    public Soy(Beverage beverage){
        this.beverage = beverage;
    }

    @Override
    public String getDesc() {
        return beverage.getDesc() + ",Soy";
    }

    @Override
    public BigDecimal cost() {
        return new BigDecimal("0.3").add(beverage.cost());
    }
}
```

```java
//奶泡调料类（继承自CondimentDecorator）
public class Whip extends CondimentDecoractor{
    //用一个实例变量记录饮料，也就是被装饰者
    Beverage beverage;

    public Whip(Beverage beverage){
        this.beverage = beverage;
    }

    @Override
    public String getDesc() {
        return beverage.getDesc() + ",Whip";
    }

    @Override
    public BigDecimal cost() {
        return new BigDecimal("0.4").add(beverage.cost());
    }
}
```

```java
public class MAIN {
    public static void main(String[] args) {
        Beverage beverage = new Espresso();
        System.out.println("Description: " + beverage.getDesc() + " $" + beverage.cost());

        //制造出一个DarkRoast(3.00)对象,用Mocha(0.2)装饰它,用第二个Mocha(0.2)装饰它,用Whip(0.4)装饰它，打印出它的描述与价钱。
        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println("Description: " + beverage2.getDesc() + " $" + beverage2.cost());

        //再来一杯调料为豆浆(Soy 0.3)、摩卡(Mocha 0.2)、奶泡(Whip 0.4)的Decaf（低咖啡因咖啡 4.00），打印出它的描述与价钱。
        Beverage beverage3 = new Decaf();
        beverage3 = new Soy(beverage3);
        beverage3 = new Mocha(beverage3);
        beverage3 = new Whip(beverage3);
        System.out.println("Description: " + beverage3.getDesc() + " $" + beverage3.cost());
    }
}

运行结果：
Description: Espresso $2.00
Description: DarkRoast,Mocha,Mocha,Whip $3.80
Description: Decaf,Soy,Mocha,Whip $4.90
```

> 参考
> [菜鸟教程](https://www.runoob.com/java/java-polymorphism.html)
> [设计模式之装饰者模式](https://www.cnblogs.com/of-fanruice/p/11565679.html)
