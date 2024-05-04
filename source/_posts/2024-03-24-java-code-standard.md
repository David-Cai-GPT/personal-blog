---
layout: post
title: java 开发规范手册
tag: 技术笔记
date: 2024-3-24
category: Technology
---

## 编程规约

### 命名规范

【**强制**】**所有编程相关的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式**
说明：正确的英文拼写和语法可以让阅读者易于理解，避免歧义。注意，纯拼音命名方式更要避免采用。
正例：ali / alibaba / taobao / cainiao/ aliyun/ youku / hangzhou 等国际通用的名称，可视同英文。
反例：DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3

【**强制**】**常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长**
正例：MAX_STOCK_COUNT / CACHE_EXPIRED_TIME
反例：MAX_COUNT / EXPIRED_TIME

【**强制**】**POJO 类中的任何布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误**
说明：在本文 MySQL 规约中的建表约定第一条，表达是与否的值采用 is_xxx 的命名方式，所以，需要在
resultMap设置从 is_xxx 到 xxx 的映射关系。
反例：定义为基本数据类型 Boolean isDeleted 的属性，它的方法也是 isDeleted()，框架在反向解析的时
候，“误以为”对应的属性名称是 deleted，导致属性获取不到，进而抛出异常。

【**参考**】**各层命名规约**

1. Service/DAO 层方法命名规约

   - 获取单个对象的方法用 get 做前缀。
   - 获取多个对象的方法用 list 做前缀，复数结尾，如：listObjects。
   - 获取统计值的方法用 count 做前缀。
   - 插入的方法用 save/insert 做前缀。
   - 删除的方法用 remove/delete 做前缀。
   - 修改的方法用 update 做前缀。

2. 领域模型命名规约

   - 数据对象：xxxDO，xxx 即为数据表名。
   - 数据传输对象：xxxDTO，xxx 为业务领域相关的名称。
   - 展示对象：xxxVO，xxx 一般为网页名称。
   - POJO 是 DO/DTO/BO/VO 的统称，禁止命名成 xxxPOJO。

分层领域模型规约：

- DO（Data Object）：此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
- DTO（Data Transfer Object）：数据传输对象，Service 或 Manager 向外传输的对象。
- BO（Business Object）：业务对象，可以由 Service 层输出的封装业务逻辑的对象。
- Query：数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止使用 Map 类
来传输。
- VO（View Object）：显示层对象，通常是 Web 向模板渲染引擎层传输的对象。

### 常量定义

【**强制**】**不允许任何魔法值（即未经预先定义的常量）直接出现在代码中**

【**推荐**】**不要使用一个常量类维护所有常量，要按常量功能进行归类，分开维护**
说明：大而全的常量类，杂乱无章，使用查找功能才能定位到修改的常量，不利于理解，也不利于维护。
正例：缓存相关常量放在类 CacheConsts 下；系统配置相关常量放在类 ConfigConsts 下。

### 代码格式

【**强制**】**注释的双斜线与注释内容之间有且仅有一个空格**
正例：

```java
// 这是示例注释，请注意在双斜线之后有一个空格
String commentString = new String();
```

【**强制**】**IDE 的 text file encoding 设置为 UTF-8; IDE 中文件的换行符使用 Unix 格式，不要使用 Windows 格式**

### OOP规约

【**强制**】**Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals**

```java
正例："test".equals(object);
反例：object.equals("test");
说明：推荐使用 java.util.Objects#equals（JDK7 引入的工具类）
```

【**强制**】**任何货币金额，均以最小货币单位且整型类型来进行存储**

### 并发处理

【**强制**】**线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这
样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险**

说明：Executors 返回的线程池对象的弊端如下：
1） FixedThreadPool 和 SingleThreadPool：
允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
2） CachedThreadPool：
允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

【**强制**】**SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为 static必须加锁，或者使用 DateUtils 工具类**
正例：注意线程安全，使用 DateUtils。亦推荐如下处理：

```java
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() {
    @Override
    protected DateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd");
    }
};
```

说明：如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar，
DateTimeFormatter 代替 SimpleDateFormat，官方给出的解释：simple beautiful strong immutable thread-safe

【**参考**】**HashMap 在容量不够进行 resize 时由于高并发可能出现死链，导致 CPU 飙升，在
开发过程中注意规避此风险**

### 控制语句

【**强制**】**在一个 switch 块内，每个 case 要么通过 continue/break/return 等来终止，要么
注释说明程序将继续执行到哪一个 case 为止；在一个 switch 块内，都必须包含一个 default
语句并且放在最后，即使它什么代码也没有**

【**强制**】**在 if/else/for/while/do 语句中必须使用大括号**
说明：即使只有一行代码，禁止不采用大括号的编码方式

```java
if (condition) statements
```

【**强制**】**在高并发场景中，避免使用”等于”判断作为中断或退出的条件**
说明：如果并发控制没有处理好，容易产生等值判断被“击穿”的情况，使用大于或小于的区间判断条件
来代替
反例：判断剩余奖品数量等于 0 时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，
这样的话，活动无法终止

### 注释规约

【**强制**】**类、类属性、类方法的注释必须使用 Javadoc 规范，使用** **/******内容*****/ 格式，不得使用// xxx 方式**

说明：在 IDE 编辑窗口中，Javadoc 方式会提示相关注释，生成 Javadoc 可以正确输出相应注释；在 IDE
中，工程调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率。

【**强制**】**所有的类都必须添加创建者和创建日期**
说明：在设置模板时，注意 IDEA 的@author 为`${USER}`，而 eclipse 的@author 为`${user}`，大小写有
区别，而日期的设置统一为 yyyy/MM/dd 的格式。

```java
正例：
/**
* @author yangguanbao
* @date 2016/10/31
*/
```

【**强制**】**方法内部单行注释，在被注释语句上方另起一行，使用//注释。方法内部多行注释使用/* */注释，注意与代码对齐**

【**参考**】**特殊注释标记，请注明标记人与标记时间。注意及时处理这些标记，通过标记扫描，经常清理此类标记。线上故障有时候就是来源于这些标记处的代码**

1） 待办事宜（TODO）:（标记人，标记时间，[预计处理时间]）
表示需要实现，但目前还未实现的功能。这实际上是一个 Javadoc 的标签，目前的 Javadoc 还没
有实现，但已经被广泛使用。只能应用于类，接口和方法（因为它是一个 Javadoc 标签）。

2） 错误，不能工作（FIXME）:（标记人，标记时间，[预计处理时间]）
在注释中用 FIXME 标记某代码是错误的，而且不能工作，需要及时纠正的情况。

### 异常日志（错误码）

【**强制**】**错误码的制定原则：快速溯源、简单易记、沟通标准化**

说明： 错误码想得过于完美和复杂，就像康熙字典中的生僻字一样，用词似乎精准，但是字典不容易随身
携带并且简单易懂。
正例：错误码回答的问题是谁的错？错在哪？

1）错误码必须能够快速知晓错误来源，可快速判断是谁的问题。

2）错误码易于记忆和比对（代码中容易 equals）。3）错误码能够脱离文档和系统平台达到线下轻量
化地自由沟通的目的。

### 异常处理

【**强制**】**异常不要用来做流程控制，条件控制**

说明：异常设计的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多。

【**推荐**】**方法的返回值可以为 null，不强制返回空集合，或者空对象等，必须添加注释充分说明什么情况下会返回 null 值**

说明：本手册明确防止 NPE 是调用者的责任。即使被调用方法返回空集合或者空对象，对调用者来说，也
并非高枕无忧，必须考虑到远程调用失败、序列化失败、运行时异常等场景返回 null 的情况。

【**推荐**】**防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：**

1） 返回类型为基本数据类型，return 包装数据类型的对象时，自动拆箱有可能产生 NPE。反例：public int f() { return Integer 对象}， 如果为 null，自动解箱抛 NPE。

2） 数据库的查询结果可能为 null。

3） 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。

4） 远程调用返回对象时，一律要求进行空指针判断，防止 NPE。

5） 对于 Session 中获取的数据，建议进行 NPE 检查，避免空指针。

6） 级联调用 obj.getA().getB().getC()；一连串调用，易产生 NPE。
正例：使用 JDK8 的 Optional 类来防止 NPE 问题。

### 日志规约

【**强制**】**应用中不可直接使用日志系统（Log4j、Logback）中的 API，而应依赖使用日志框架（SLF4J、JCL--Jakarta Commons Logging）中的 API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一**

说明：日志框架（SLF4J、JCL--Jakarta Commons Logging）的使用方式（推荐使用 SLF4J）
使用 SLF4J：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger logger = LoggerFactory.getLogger(Test.class);
```

使用 JCL：

```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

private static final Log log = LogFactory.getLog(Test.class);
```

【**强制**】**在日志输出时，字符串变量之间的拼接使用占位符的方式**
说明：因为 String 字符串的拼接会使用 StringBuilder 的 append()方式，有一定的性能损耗。使用占位符仅
是替换动作，可以有效提升性能。

正例：

```java
logger.debug("Processing trade with id: {} and symbol: {}", id, symbol);
```

【**强制**】**日志打印时禁止直接用 JSON 工具将对象转换成 String**

说明：如果对象里某些 get 方法被重写，存在抛出异常的情况，则可能会因为打印日志而影响正常业务流程的执行。

正例：打印日志时仅打印出业务相关属性值或者调用其对象的 toString()方法。

【**推荐**】**可以使用 warn 日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。如非必要，请不要在此场景打出 error 级别，避免频繁报警**

说明：注意日志输出的级别，error 级别只记录系统逻辑出错、异常或者重要的错误信息。

### 单元测试

【**强制**】**好的单元测试必须遵守 AIR 原则**

说明：单元测试在线上运行时，感觉像空气（AIR）一样并不存在，但在测试质量的保障上，却是非常关键
的。好的单元测试宏观上来说，具有自动化、独立性、可重复执行的特点。

- A：Automatic（自动化）
- I：Independent（独立性）
- R：Repeatable（可重复）

【**推荐**】**编写单元测试代码遵守 BCDE 原则，以保证被测试模块的交付质量**

- B：Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。
- C：Correct，正确的输入，并得到预期的结果。
- D：Design，与设计文档相结合，来编写单元测试。
- E：Error，强制错误信息输入（如：非法数据、异常流程、业务允许外等），并得到预期的结果。

### 安全规约

【**强制**】**隶属于用户个人的页面或者功能必须进行权限控制校验**
说明：防止没有做水平权限校验就可随意访问、修改、删除别人的数据，比如查看他人的私信内容。

【**强制**】**用户敏感数据禁止直接展示，必须对展示数据进行脱敏**
说明：中国大陆个人手机号码显示为:137****0969，隐藏中间 4 位，防止隐私泄露。

【**强制**】**用户请求传入的任何参数必须做有效性验证**
说明：忽略参数校验可能导致：

- page size 过大导致内存溢出
- 恶意 order by 导致数据库慢查询
- 缓存击穿
- SSRF
- 任意重定向
- SQL 注入，Shell 注入，反序列化注入
- 正则输入源串拒绝服务 ReDoS

Java 代码用正则来验证客户端的输入，有些正则写法验证普通用户输入没有问题，但是如果攻击人员使用的是特殊构造的字符串来验证，有可能导致死循环的结果。

### 二方库依赖

【**强制**】**二方库版本号命名方式：主版本号.次版本号.修订号**

1）主版本号：产品方向改变，或者大规模 API 不兼容，或者架构不兼容升级。

2） 次版本号：保持相对兼容性，增加主要功能特性，影响范围极小的 API 不兼容修改。

3） 修订号：保持完全兼容性，修复 BUG、新增次要功能特性等。

说明：注意起始版本号必须为：1.0.0，而不是 0.0.1。

反例：仓库内某二方库版本号从 1.0.0.0 开始，一直默默“升级”成 1.0.0.64，完全失去版本的语义信息。

【**强制**】**二方库里可以定义枚举类型，参数可以使用枚举类型，但是接口返回值不允许使用枚举类型或者包含枚举类型的 POJO 对象**

### 服务器

【**推荐**】**调大服务器所支持的最大文件句柄数（File Descriptor，简写为 fd）**

说明：主流操作系统的设计是将 TCP/UDP 连接采用与文件一样的方式去管理，即一个连接对应于一个 fd。

主流的linux服务器默认所支持最大fd数量为1024，当并发连接数很大时很容易因为 fd不足而出现“open too many files”错误，导致新的连接无法建立。建议将 linux 服务器所支持的最大句柄数调高数倍（与服
务器的内存数量相关）。

### 其他

【**强制**】**避免用 Apache Beanutils 进行属性的 copy**

说明：Apache BeanUtils 性能较差，可以使用其他方案比如 Spring BeanUtils, Cglib BeanCopier，注意
均是浅拷贝。

参考：阿里巴巴Java开发手册（泰山版）
