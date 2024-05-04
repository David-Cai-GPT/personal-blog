---
layout: post
title: Redis持久化方式
tag: 技术笔记
date: 2021-01-16
category: Technology blog
---
## Redis持久化方式

Redis是一个内存数据库，数据保存在内存中，相对于数据库来说，Redis提供了更快的读写服务以满足更高的并发要求，但是既然是存在内存中， 那么数据是非常容易发生丢失的（例如服务器异常宕机），为此Redis为我们提供了持久化的机制，分别为RDB(Redis DataBase)与AOF(Append Only File)。


#### 持久化流程


首先来介绍一下持久化流程## Redis持久化方式


Redis是一个内存数据库，数据保存在内存中，相对于数据库来说，Redis提供了更快的读写服务以满足更高的并发要求，但是既然是存在内存中， 那么数据是非常容易发生丢失的（例如服务器异常宕机），为此Redis为我们提供了持久化的机制，分别为RDB(Redis DataBase)与AOF(Append Only File)。


#### 持久化流程


首先来介绍一下持久化流程


1. 客户端向服务端发送写操作（数据再客户端的内存中）
2. 数据库服务端接收到写请求的数据（数据再服务端的内存中）
3. 服务端调用write这个系统调用，将数据往磁盘上写(数据在系统内存的缓冲区中)。
4. 操作系统将缓冲区中的数据转移到磁盘控制器上(数据在磁盘缓存中)。
5. 磁盘控制器将数据写到磁盘的物理介质中(数据真正落到磁盘上)。


这5个过程是在理想条件下一个正常的保存流程，但是在大多数情况下，我们的机器等等都会有各种各样的故障，这里划分了两种情况：


- Redis数据库发生故障，只要在上面的第三步执行完毕，那么就可以持久化保存，剩下的两步由操作系统替我们完成。
- 操作系统发生故障，必须上面5步都完成才可以。


在这里只考虑了保存的过程可能发生的故障，其实保存的数据也有可能发生损坏，需要一定的恢复机制，不过在这里就不再延伸了。现在主要考虑的是redis如何来实现上面5个保存磁盘的步骤。它提供了两种策略机制，也就是RDB和AOF。


#### RDB


##### 简介


RDB是Redis用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。


##### 手动触发


手动触发Redis进行RDB持久化的命令有两种：


1.save


该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，知道RDB过程完成为止。


显然该命令对于内存比较大的实例会造成长时间的阻塞，这是致命的错误，为了解决此问题，Redis提供了第二种方式


2.bgsave


执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以相应客户端请求，具体操作失Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束，阻塞只发生在fork阶段，一般时间很短。


##### 优劣势


**优势**


1. RDB是一个非常紧凑(compact)的文件，它保存了redis 在某个时间点上的数据集。这种文件非常适合用于进行备份和灾难恢复。
2. 生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。
3. RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。


生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作


RDB在恢复大数据集时的速度要比


**劣势**


1. RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，如果不采用压缩算法(内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑)，频繁执行成本过高(影响性能)
2. RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题(版本不兼容)
3. 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改(数据有丢失)


1. 客户端向服务端发送写操作（数据再客户端的内存中）
2. 数据库服务端接收到写请求的数据（数据再服务端的内存中）
3. 服务端调用write这个系统调用，将数据往磁盘上写(数据在系统内存的缓冲区中)。
4. 操作系统将缓冲区中的数据转移到磁盘控制器上(数据在磁盘缓存中)。
5. 磁盘控制器将数据写到磁盘的物理介质中(数据真正落到磁盘上)。


这5个过程是在理想条件下一个正常的保存流程，但是在大多数情况下，我们的机器等等都会有各种各样的故障，这里划分了两种情况：


- Redis数据库发生故障，只要在上面的第三步执行完毕，那么就可以持久化保存，剩下的两步由操作系统替我们完成。
- 操作系统发生故障，必须上面5步都完成才可以。


在这里只考虑了保存的过程可能发生的故障，其实保存的数据也有可能发生损坏，需要一定的恢复机制，不过在这里就不再延伸了。现在主要考虑的是redis如何来实现上面5个保存磁盘的步骤。它提供了两种策略机制，也就是RDB和AOF。


#### RDB


##### 简介


RDB是Redis用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。


##### 手动触发


手动触发Redis进行RDB持久化的命令有两种：


1.save


该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，知道RDB过程完成为止。


显然该命令对于内存比较大的实例会造成长时间的阻塞，这是致命的错误，为了解决此问题，Redis提供了第二种方式


2.bgsave


执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以相应客户端请求，具体操作失Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束，阻塞只发生在fork阶段，一般时间很短。


##### 优劣势


**优势**


1. RDB是一个非常紧凑(compact)的文件，它保存了redis 在某个时间点上的数据集。这种文件非常适合用于进行备份和灾难恢复。
2. 生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。
3. RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。


生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作


RDB在恢复大数据集时的速度要比


**劣势**


1. RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，如果不采用压缩算法(内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑)，频繁执行成本过高(影响性能)
2. RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题(版本不兼容)
3. 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改(数据有丢失)

##### 配置参数详解

在redis.conf配置文件中，找到如下的配置项，文档已经描述的非常详细，以下这段配置表示的含义如下：

save 900 1：900秒之内，对数据库至少修改了1次。
save 300 10：300秒之内，对数据库至少修改了10次。
save 60 10000：60秒之内，对数据库至少修改了10000次。

只要满足三个条件中任意一个，就会触发bgsave。

```python
################################ SNAPSHOTTING ################################
#
# Save the DB on disk:
#
# save <seconds> <changes>
#
# Will save the DB if both the given number of seconds and the given
# number of write operations against the DB occurred.
#
# In the example below the behaviour will be to save:
# after 900 sec (15 min) if at least 1 key changed
# after 300 sec (5 min) if at least 10 keys changed
# after 60 sec if at least 10000 keys changed
#
# Note: you can disable saving completely by commenting out all "save" lines.
#
# It is also possible to remove all the previously configured save
# points by adding a save directive with a single empty string argument
# like in the following example:
#

#save ""
save 900 1
save 300 10
save 60 10000

```

回收触发的机制可以看日志输出

```java
:M ** Jun 09:47:37.057 * 1 changes in 900 seconds. Saving...
:M ** Jun 09:47:37.058 * Background saving started by pid 12789
:C ** Jun 09:47:39.023 * DB saved on disk

:M ** Dec 10:55:11.963 * 10000 changes in 60 seconds. Saving...
:M ** Dec 10:55:11.965 * Background saving started by pid 27865
:C ** Dec 10:55:13.268 * DB saved on disk
```

如果需要关闭RDB，则需要配置参数

```Python
save ""
#save 900 1
#save 300 10
#save 60 10000
```

一些其他参数

```python
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
#(default) dir ./
dir /opt/redis/rdb

```

**stop-writes-on-bgsave-error**

默认情况下，如果bgsave失败，Redis将停止接受写操作。以此让用户意识到bgsave出了问题，否则可能没有人会注意到，并将发生一些灾难。

如果bgsave再次开始工作，Redis将自动再次允许写入。

然而，如果你已经设置了适当的监测Redis服务器和持久性，你可以禁用这个功能，这样即使生成rdb文件时出现磁盘、权限等出现问题，redis也会照常工作。

**rdbcompression**

默认情况下使用LZF压缩，如果你不想消耗CPU，可以设置为false，但是文件可能会更大。

**rdbchecksum**

使用CRC64算法对生成的rdb文件进行检验，但会有大约10%的性能消耗。

**dbfilename**

生成rdb文件的名称

**dir**

生成rdb文件的目录

#### AOF

##### 简介

本质：把用户执行的每个 ”写“ 指令（增加、修改、删除）都备份到文件中，还原数据的时候就是执行具体写指令。

##### 优劣势

**优势**

1. 使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 fsync 策略，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。 AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。

**劣势**

1. 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。

##### 配置参数详解

根据redis.conf中的配置项，redis提供了三种同步的策略。

```Python
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# 默认配置为 appendfsync everysec
# appendfsync always
appendfsync everysec
# appendfsync no
```

**appendfsync everysec**

默认配置，表示每隔1秒钟，调用fsync()函数，将aof_buf缓冲区中的数据写入磁盘，这是在速度与数据安全两者之间的折中选项，就算出现了故障停机，也只丢失一秒钟的操作数据。

**appendfsync always**

这是3种同步中最慢的方式，不过也是最安全的方式，每次调用flushAppendOnlyFile函数都会执行fsync操作。所以当出现了故障停机时，丢失的是一个事件循环过程中产生的命令数据。

**appendfsync no**

这是3种同步中最不安全的方式，缓冲区中的数据何时写入磁盘完全后操作系统决定，但也因为这种方式不需要调用flushAppendOnlyFile函数，所以也是3种方式中最快的。

**AOF文件的重写**

AOF持久化是通过保存被执行的写命令来记录数据库当前状态的，所以随着服务器运行时间的流逝，AOF文件中的内容会越来越多，文件的体积也会越来越大，所以为了解决这个问题，Redis就提供了AOF文件重写的功能。

```Python
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

**auto-aof-rewrite-percentage 100**

**auto-aof-rewrite-min-size 64mb**

这个两个参数的表示：要重写的AOF文件最小大小为64mb，并且以上一次重写后的文件大小与当前文件大小进行比较，如果超过100%，则触发重写。

```Python
# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no
```

实际上本质还是磁盘I/O的问题，Redis作者认为在AOF中如果你配置了every甚至always，那么就会产生大量的磁盘写入操作(调用sync)，假设此时又遇到了bgsave或bgwriteaof，这将会导致主进程阻塞的时间更长，所以提供了这样一个配置，当为no时，表示不考虑磁盘写入的问题，依旧会调用fsync刷盘操作，这样数据可靠性得到保证，但可能会产生较高的阻塞延时问题，如果不能接受阻塞的问题，你可以配置成yes，那么就意味着在重写期间，即使你配置的是every或always，那么也只会调用write函数写入到操作系统缓存中，什么时候刷盘由操作系统自身决定（默认为30秒），这样可以有效减少阻塞问题，但数据的可靠性就无法保证了，最差情况下就会丢失30秒的数据。

#### 出现问题排查方案

此次问题为Redis持久化阻塞问题

看一下问题日志

```python
:M ** Jan 16:27:51.008 * Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.
:M ** Jan 16:30:20.037 * 10 changes in 300 seconds. Saving...
:M ** Jan 16:30:20.038 * Background saving started by pid 15524
:C ** Jan 16:30:27.370 * DB saved on disk
:C ** Jan 16:30:27.371 * RDB: 2 MB of memory used by copy-on-write
:M ** Jan 16:30:27.449 * Background saving terminated with success
:M ** Jan 16:32:03.085 * Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.
:M ** Jan 16:34:28.506 * 10000 changes in 60 seconds. Saving...
:M ** Jan 16:34:28.507 * Background saving started by pid 29110
:C ** Jan 16:35:20.481 * DB saved on disk
```

Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.

AOF fsync阻塞问题，AOF fsync超过两秒阻塞了Redis主线程，导致写入出现问题

**持久化阻塞**

当我们开启AOF持久化功能时， 文件刷盘的方式一般采用每秒一次， 后台线程每秒对AOF文件做fsync操作。 当硬盘压力过大时， fsync操作需要等待， 直到写入完成。 如果主线程发现距离上一次的fsync成功超过2秒， 为了数据安全性它会阻塞直到后台线程执行fsync操作完成。 这种阻塞行为主要是硬盘压力引起\， 可以查看Redis日志识别出这种情况， 当发生这种阻塞行为时， 会打印如上日志：

阻塞流程分析：
1） 主线程负责写入AOF缓冲区。
2） AOF线程负责每秒执行一次同步磁盘操作，并记录最近一次同步时间(AOF线程复制统计时间，主线程负责对比和上次时间差)
3） 主线程负责对比上次AOF同步时间：
·如果距上次同步成功时间在2秒内， 主线程直接返回。
·如果距上次同步成功时间超过2秒， 主线程将会阻塞， 直到同步操作完成。

通过对AOF阻塞流程可以发现两个问题：
1） everysec配置最多可能丢失2秒数据， 不是1秒。
2） 如果系统fsync缓慢， 将会导致Redis主线程阻塞影响效率。

排查

1. 检查最近fork操作耗时：执行 `info status` 获取到 latest_fork_usec 指标
2. 检查AOF刷盘最近成功时间：查看日志

解决：

1. 若fork操作耗时超过1秒，避免使用过大的内存实例和规避fork缓慢的操作系统
2. 若AOF刷盘fsync成功操作超过2秒，降低其他进程对硬盘的压力


[详解Redis中两种持久化机制RDB和AOF（面试常问，工作常用)](https://baijiahao.baidu.com/s?id=1654694618189745916&wfr=spider&for=pc)
[Redis RDB持久化流程分析与参数配置详解](https://blog.csdn.net/csdn_wyl2016/article/details/109445556)
[B持久化流程分析与参数配置详解](https://blog.csdn.net/qq_34556414/article/details/106292161)