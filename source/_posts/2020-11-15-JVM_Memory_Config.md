---
layout: post
title: JVM启动参数配置学习
tag: 技术笔记
date: 2020-11-15
category: Technology blog
---
### **例如tomcat下的启动配置：**

JAVA_OPTS="-Xms640m -Xmx800m -XX:PermSize=128M -XX:MaxNewSize=256m -XX:MaxPermSize=384m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/log -XX:OnOutOfMemoryError=\"/opt/tomcat/bin/stopTomcat.sh\" -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/log/heap_trace.log -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Duser.language=zh -Duser.region=CN -Duser.contry=CN -Dhttps.protocols=TLSv1.1,TLSv1.2"

-Xms500m  初始堆内存大小
-Xmx1024m 最大堆内存大小
-XX:PermSize=128M 永久区空间大小，JVM初始分配的非堆内存, 不会被回收
-XX:MaxPermSize=384m 永久空间最大大小，JVM最大允许分配的非堆内存
-XX:MaxNewSize=256m JVM堆区域新生代内存的最大可分配大小
-XX:+HeapDumpOnOutOfMemoryError 当内存溢出时触发java.lang.OutOfMemo: Java heap space
-XX:HeapDumpPath=/log 内存溢出时，保存内存快照文件在指定的文件目录
-XX:OnOutOfMemoryError=\"/opt/tomcat/bin/stopTomcat.sh\" 内存溢出执行stopTomcat.sh
-XX:+PrintGCDetails 打印GC详细信息

```java
5.617（时间戳）: [GC（Young GC） 5.617（时间戳）: [ParNew（GC的区域）: 43296K（垃圾回收前的大小）->7006K（垃圾回收以后的大小）(47808K)（该区域总大小）, 0.0136826 secs（回收时间）] 44992K（堆区垃圾回收前的大小）->8702K（堆区垃圾回收后的大小）(252608K)（堆区总大小）, 0.0137904 secs（回收时间）] [Times: user=0.03（GC用户耗时） sys=0.00（GC系统耗时）, real=0.02 secs（GC实际耗时）]
```

-XX:+PrintGCTimeStamps 打印GC的时间戳
-Xloggc:/log/heap_trace.log 指定GC日志的位置
-Dfile.encoding=UTF-8 强行设置系统文件编码格式为utf-8
-Dsun.jnu.encoding=UTF-8 强行设置系统文件命名编码为utf-8
-Duser.language=zh -Duser.region=CN -Duser.contry=CN 用户默认语言为中文，默认地区为中国，默认国家为中国
-Dhttps.protocols=TLSv1.1,TLSv1.2 默认浏览器协议

### **扩展**

-Xss 每个线程的堆栈大小
-XX:NewRatio 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)  -XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5 Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。
-XX:SurvivorRatio Eden区与Survivor区的大小比值 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10

#### **springboot一些自动项优化：**

1. 使用-server模式
设置JVM使用server模式，64位JDK默认启动该模式
java -server -jar springboot-1.0.jar

2. 指定堆参数
同上

3. 远程Debug
java -Djavax.net.debug=ssl -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket.
server=y,suspend=n,address=8888 -jar springboot-1.0.jar

JVM工具远程连接 （Jconsole JvisualVm）
java -jar -Djava.rmi.server.hostname={服务器地址} -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port={端口} -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false jantent-1.0-SNAPSHOT.

-Dcom.sun.management.jmxremote.ssl=false 不启用ssl安全连接
-Dcom.sun.management.jmxremote.authenticate=false 不需要用户进行安全验证