# JVM 性能调优点

### 一、内存调优

如果项目为war包且为 Tomcat 启动，可将 Tomcat 的配置修改一下：

找到Tomcat根目录下的bin目录，设置catalina.sh文件中JAVA_OPTS变量即可，因为后面的启动参数会把JAVA_OPTS作为JVM的启动参数来处理。

> JAVA_OPTS= "$JAVA_OPTS  -Xmx512m -Xms512m -Xmn170m -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0"

* `-Xmx512m`：设置Java虚拟机的堆的最大可用内存大小，单位：兆(m)，整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m。堆的不同分布情况，对系统会产生一定的影响。尽可能将对象预留在新生代，减少老年代GC的次数（通常老年回收起来比较慢）。实际工作中，**通常将堆的初始值(-Xms)和最大值(-Xmx)设置相等，这样可以减少程序运行时进行的垃圾回收次数和空间扩展，从而提高程序性能**。

* `-Xms512m`：设置Java虚拟机的堆的初始值内存大小，单位：兆(m)，此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

* `-Xmn170m`：设置年轻代内存大小，单位：兆(m)，此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。一般在增大年轻代内存后，也会将会减小年老代大小。

* `-Xss128k`：设置每个线程的栈大小。JDK5.0以后每个线程栈大小为1M，以前每个线程栈大小为256K。更具应用的线程所需内存大小进行调整。

  在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

* `-XX:NewRatio=4`：设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5 。

* `-XX:SurvivorRatio=4`：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6。

* `-XX:MaxPermSize=16m`：设置持久代大小为16m，上面也说了，持久代一般固定的内存大小为64m。

* `-XX:MaxTenuringThreshold=0`：设置垃圾最大年龄。
  如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。
  如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。



### 二、垃圾回收策略调优

找到Tomcat根目录下的bin目录，也是设置catalina.sh文件中JAVA_OPTS变量即可。我们都知道Java虚拟机都有默认的垃圾回收机制，但是不同的垃圾回收机制的效率是不同的，正是因为这点我们才经常对Java虚拟机的垃圾回收策略进行相应的调整。

> Java虚拟机的垃圾回收策略一般分为：串行收集器、并行收集器和并发收集器。

* ### 串行收集器：

  * `-XX:+UseSerialGC`：代表垃圾回收策略为串行收集器，即在整个扫描和复制过程采用单线程的方式来进行，适用于单CPU、新生代空间较小及对暂停时间要求不是非常高的应用上，是client级别默认的GC方式，主要在JDK1.5之前的垃圾回收方式。

* ### 并行收集器：

  * `-XX:+UseParallelGC`：代表垃圾回收策略为并行收集器(吞吐量优先)，即在整个扫描和复制过程采用多线程的方式来进行，适用于多CPU、对暂停时间要求较短的应用上，是server级别默认采用的GC方式。此配置仅对年轻代有效。该配置只能让年轻代使用并发收集，而年老代仍旧使用串行收集。
  * `-XX:ParallelGCThreads=4`：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
  * -XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集 。
  * `-XX:MaxGCPauseMillis=100`：设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
  * `-XX:+UseAdaptiveSizePolicy`：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

* ### 并发收集器：

  * `-XX:+UseConcMarkSweepGC`:代表垃圾回收策略为并发收集器。

* ![](./image\public\GC内存分配策略组合.png)

青谷服务：

	>pkill -f saas-uc-1.0.0-SNAPSHOT.jar && nohup java -jar saas-uc-1.0.0-SNAPSHOT.jar --spring.profiles.active=test > /dev/null 2>&1 &	
	>
	>nohup java -jar -Xms200m -Xmx200m -Xmn200m qgyun-service-workflow-1.0.3-SNAPSHOT.jar --spring.profiles.active=test > /dev/null 2>&1 &

