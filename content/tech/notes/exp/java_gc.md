---
title: "JVM优化经验"
slug: ""
date: 2020-09-22T11:04:57+08:00
lastmod: 2020-09-22T11:04:57+08:00
author: ["路非非"]
tags: # 标签
- JVM
series:
-
description: ""
weight:
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

依据的原则是根据Java Performance里面的推荐公式来进行设置。
![rule](img.png)
>Java整个堆大小设置，Xmx 和 Xms设置为老年代存活对象的3-4倍，即FullGC之后的老年代内存占用的3-4倍。  
>永久代 PermSize和MaxPermSize(元空间)设置为老年代存活对象的1.2-1.5倍。  
>年轻代Xmn的设置为老年代存活对象的1-1.5倍。  
>老年代的内存大小设置为老年代存活对象的2-3倍。

------
# 理论参数
* `-Xms`  
初始堆的大小，也是堆大小的最小值，默认值是总共的物理内存/64（且小于1G），默认情况下，当堆中可用内存小于40%（这个值可以用-XX: MinHeapFreeRatio 调整，如-X:MinHeapFreeRatio=30）时，堆内存会开始增加，一直增加到-Xmx的大小
* `-Xmx`  
堆的最大值，默认值是总共的物理内存1/4，如果Xms和Xmx都不设置，则两者大小会相同，默认情况下，当堆中可用内存大于70%（这个值可以用-XX: MaxHeapFreeRatio 调整，如-X:MaxHeapFreeRatio=60）时，堆内存会开始减少，一直减小到-Xms的大小
* `-Xmn`  
用于设置新生代的大小。设置一个较大的新生代会减少老年代的大小，这个参数堆系统性能以及GC行为有很大的影响。新生代的大小一般设置为整个堆空间的1/3到1/4
* `-Xss`  
设置每个线程可使用的内存大小，即栈的大小。在相同物理内存下，减小这个值能生成更多的线程，当然操作系统对一个进程内的线程数还是有限制的，不能无限生成。线程栈的大小是个双刃剑，如果设置过小，可能会出现栈溢出，特别是在该线程内有递归、大的循环时出现溢出的可能性更大，如果该值设置过大，就有影响到创建栈的数量，如果是多线程的应用，就会出现内存溢出的错误。
* `--XX:SurvivorRatio`  
新生代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：为3，表示Eden：Survivor=3：2，一个Survivor区占整个新生代的1/5 
* `-XX:MaxTenuringThreshold`  
设置转入老年代的存活次数。如果是0，则直接跳过新生代进入老年代。  
将此值设置为一个较小值对于需要大量常驻内存的应用，这样做可以提高效率。  
如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代存活时间，增加对象在年轻代被垃圾回收的概率，减少Full GC的频率，这样做可以在某种程度上提高服务稳定性。

可以通过以下命令查看系统默认的InitialHeapSize和MaxHeapSize：  
```shell
java -XX:+PrintFlagsFinal -version | grep HeapSize
```
生产环境中设置-Xms与-Xmx一致以保证程序稳定  
开发测试环境设置一个比较小的-Xms，以支撑更多的测试程序运行  

# 实践步骤
## java应用
1、运行程序一段时间，查询系统各指标   
以开发集成环境为例  
```shell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]  
```

S0C: 年轻代第一个survivor的容量（字节）  
S1C：年轻代第二个survivor的容量（字节）  
S0U：年轻代第一个survivor已使用的容量（字节）  
S1U：年轻代第二个survivor已使用的容量（字节）  
EC：年轻代中Eden的空间（字节）  
EU：年代代中Eden已使用的空间（字节）  
OC：老年代的容量（字节）  
OU: 老年代中已使用的空间（字节）  
MC：方法区（元空间）的容量  
MU：方法区（元空间）已使用的容量  
CCSC: 压缩类的空间容量  
CCSU: 压缩类已使用的容量  
YGC：从应用程序启动到采样时年轻代中GC的次数  
YGCT: 从应用程序启动到采样时年轻代中GC所使用的时间（单位：S）  
FGC：从应用程序启动到采样时老年代中GC（FULL GC）的次数  
FGCT：从应用程序启动到采样时老年代中GC所使用的时间（单位：S）


得知：OU老年代所占用的内存为 **1215418（122M）**  

## 基础参数
**堆大小**  
`-Xms256m -Xmx512m`  

**新生代**  
`-Xmn128m`  

**新生代中Eden区与两个Survivor区**  
`-XX:+PrintTenuringDistribution`表示输出GC之后，survivor区域各年龄段的内存分布情况
![img_1.png](img_1.png)
Desired survivor size表示survivor区域允许容纳的最大空间大小为34078720bytes。   
> 这里Desired survivor size这行下面并没有各个age对象的分布，那就表示此次gc之后，当前survivor区域并没有age小于max threshold的存活对象。而这里一个都没有输出，表示此次gc回收对象之后，没有存活的对象可以拷贝到新的survivor区。

`-XX:SurvivorRatio=8`   
默认值8。含义：eden/suvivor=8 ,其中survivor包含from、to。
>新生代设定128m，那么,from =128m/(8+2)=12.8m, eden=102.4m。也就是说新生代会被分成10分，eden区占8分，survivor占2份，这也能看出比值越大survivor被稀释的越严重。比值越小survivor分到的空间更大。

`-XX:+HeapDumpOnOutOfMemoryError`  
`-XX:HeapDumpPath=./dump.log`   
在Java程序运行过程中，如果堆内存不足，则有可能抛出内存溢出错误（OutOfMemory）。一旦发生这类问题，系统将被迫退出，引起严重的业务中断。为了能不断改善系统，需要在发生错误时，保留尽可能多的现场信息，使用该参数可以在内存溢出时导出整个堆信息。

**方法区配置**  
`-XX:MetaspaceSize=256M` `-XX:MaxMetaspaceSize=256M`  
方法区大小决定系统可以保存多少个类，如果使用动态代理，那设置该值需要更加合理，以确保方法区不发生内存溢出。  

**栈配置**    
`-Xss128K`  
这个参数决定了函数调用的最大深度，通过jclasslib工具可以查看函数的局部变量信息。  
>设置每个线程的堆栈大小。JDK5.0以后每个线程堆 栈大小为1M，以前每个线程堆栈大小为256K。根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。该应用tomcat线程为500。

**直接内存配置**  
`-XX:MaxDirectMemorySize不设置`  
默认值为最大堆空间-Xmx，达到最大量时同样触发GC  
>直接内存跳过了Java堆，使Java程序可以直接访问原生堆空间，因此从一定程度上加快了内存空间的访问速度。  
> 但是直接内存的申请速度远远大于堆空间，因此直接内存适合于申请次数较少，读写频繁的场合。  
> 
> 从Java 1.4开始引入了直接内存。新的I / O（NIO）类引入了一种基于通道和缓冲区执行I / O的新方法。 NIO添加了对直接ByteBuffers的支持，可以直接传递给本机内存而不是Java堆。  
> 在某些情况下，由于它们可以避免在Java堆和本机堆之间复制数据，因此可以大大提高它们的速度。  
> 
> 可以通过调用 allocateDirect $ b创建直接字节缓冲区。 $ b此类的工厂方法。与非直接缓冲区相比，用此方法返回的缓冲区通常具有更高的分配和释放成本。
> 直接缓冲区的内容可能驻留在正常垃圾回收堆之外的中，因此它们对应用程序内存占用的影响可能并不明显。  
> 因此，建议主要为直接受基础系统本机 I/O 操作约束的大型大型缓冲区分配直接缓区。通常，最好仅在直接缓冲区在程序性能中产生可衡量的收益时才分配它们。

## 垃圾回收器选择
jdk1.7 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）  
jdk1.8 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）  
jdk1.9 默认垃圾收 2020/09/21 18:05 集器G1  

`-XX:+PrintCommandLineFlagsjvm`参数  
可查看默认设置收集器类型  

`-XX:+PrintGCDetails`  
亦可通过打印的GC日志的新生代、老年代名称判断  

串行回收器（小数据量）不采用  
`-XX:+UseSerialGC`  
设置串行回收器。虚拟机在Client模式下运行时，它是默认的垃圾回收器。

并行回收器（吞吐量优先）不采用  
`-XX:+UseParallelGC`  
设置为并行回收器。此配置仅对年轻代有效。即年轻代使用并行回收，而年老代仍使用串行回收。  

`-XX:+UseParallelOldGC`  
配置年老代垃圾回收方式为并行回收。JDK6.0开始支持对年老代并行回收。  

`-XX:ParallelGCThreads=20`  
配置并行回收器的线程数，即：同时有多少个线程一起进行垃圾回收。此值建议配置与CPU数目相等。CMS默认启动的并发线程数是(ParallelGCThreads+3)/4。

`-XX:MaxGCPauseMillis=100`  
设置每次年轻代垃圾回收的最长时间（单位毫秒）。如果无法满足此时间，JVM会自动调整年轻代大小，以满足此时间。该值越小，停顿时间越短，触发GC次数越频繁、总回收时间越长、吞吐量越低。

`-XX:+UseAdaptiveSizePolicy`   
设置此选项后，并行收集器会自动调整年轻代Eden区大小和Survivor区大小的比例，以达成目标系统规定的最低响应时间或者收集频率等指标。此参数建议在使用并行收集器时，一直打开。

并发收集器（响应时间优先）
`-XX:+UseConcMarkSweepGC`  
即CMS收集，设置年老代为并发收集。CMS收集是JDK1.4后期版本开始引入的新GC算法。它的主要适合场景是对响应时间的重要性需求大于对吞吐量的需求，能够承受垃圾回收线程和应用线程共享CPU资源，并且应用中存在比较多的长生命周期对象。CMS收集的目标是尽量减少应用的暂停时间，减少Full GC发生的几率，利用和应用程序线程并发的垃圾回收线程来标记清除年老代内存。

`-XX:+UseParNewGC`  
设置年轻代为并发收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此参数。

`-XX:+UseCMSCompactAtFullCollection`  
打开内存空间的压缩和整理，在Full GC后执行。可能会影响性能，但可以消除内存碎片。

`-XX:CMSFullGCsBeforeCompaction=0 默认0`  
由于并发收集器不对内存空间进行压缩和整理，所以运行一段时间并行收集以后会产生内存碎片，内存使用效率降低。此参数设置运行0次Full GC后对内存空间进行压缩和整理，即每次Full GC后立刻开始压缩和整理内存。

`-XX:+CMSIncrementalMode`  
设置为增量收集模式。一般适用于单CPU情况。

`-XX:CMSInitiatingOccupancyFraction=70`  
默认值-1  
当老年代的空间使用率达到68%时，会执行一次CMS回收。如果内存使用率增长过快，在CMS的执行过程中，已经出现了内存不足的情况，此时，CMS回收就会失败，虚拟机将启用老年代串行回收器进行垃圾回收，应用程序也将完全中断，知道垃圾回收完成。  
因此可以根据应用程序的特点，如果内存增长缓慢，则可以设置一个较大的值，有效降低CMS的触发频率，反之若内存增长很快，则应该降低这个值，避免频繁触发老年代串行回收器。

## 其它垃圾回收参数
`-XX:+ScavengeBeforeFullGC`  
年轻代GC优于Full GC执行。

`-XX:-DisableExplicitGC`  
不响应 System.gc() 代码。

`-XX:+UseThreadPriorities`  
启用本地线程优先级API。即使`java.lang.Thread.setPriority()`生效，不启用则无效。

`-XX:SoftRefLRUPolicyMSPerMB=0`  
软引用对象在最后一次被访问后能存活0毫秒（JVM默认为1000毫秒）。

`-XX:TargetSurvivorRatio=90`  
允许90%的Survivor区被占用（JVM默认为50%）。提高对于Survivor区的使用率。

## GC日志
`-XX:+PrintGC`  
参数-XX:+PrintGC（或者-verbose:gc）开启了简单GC日志模式，为每一次新生代（young generation）的GC和每一次的Full GC打印一行信息。

`-XX:+PrintGCDetails`  
开启了详细GC日志模式。在这种模式下，日志格式和所使用的GC算法有关。

`-XX:+PrintGCTimeStamps`  
`-XX:+PrintGCDateStamps`  
将时间和日期加到GC日志中。

`-Xloggc:./gclogs.log`  
缺省的GC日志时输出到终端的，使用-Xloggc:也可以输出到指定的文件。需要注意这个参数隐式的设置了参数-XX:+PrintGC和-XX:+PrintGCTimeStamps。

`-XX:+PrintHeapAtGC`  
打印GC前后的详细堆栈信息。

`-XX:+PrintCommandLineFlags`  
打印虚拟机默认参数。

`-XX:+PrintTenuringDistribution`  
用于显示每次Minor GC（年轻代GC）时Survivor区中各个年龄段的对象的大小。

`-XX:+PrintCompilation`  
打印Java HotSpot VM动态运行时编译器的详细输出。

# 最终JVM参数
```shell
-Xms256m -Xmx512m -Xmn128m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./dump.log -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -Xss128K -XX:+UseConcMarkSweepGC -XX:PrintHeapAtGC -XX:+PrintGCDateStamps -Xloggc:./gclogs.log
```
