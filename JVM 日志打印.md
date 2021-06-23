###  JVM 日志打印

#### 最佳实践

* 日常打印：

  ```shell
  -Xloggc:'gc.log' -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
  ```

* 排查打印

  在日常打印基础上添加下列参数

  ```shell
  -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCApplicationStoppedTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1
  -XX:+PrintHeapAtGC -XX:+PrintTLAB -XX:+PrintReferenceGC -XX:+PrintTenuringDistribution
  ```

#### GC 日志模式

一般只要开启 GC 日志打印，都会默认开启简单的日志模式，生产环境强烈建议开启详细 GC 日志模式，两种模式互斥，同时开启为详细GC日志模式

```shel
// 二选一
-XX:+PrintGC  		// 简单gc日志模式 
-XX:+PrintGCDetails // 详细gc日志模式
```

#### GC 日志时间

只要开启GC日志打印，都会默认开启打印距 JVM 启动时间间距差值时间，生产环境建议打印当前系统时间

```shell
//二选一
-XX:+PrintGCTimeStamps  //打印距jvm启动时间间距差值时间
-XX:+PrintGCDateStamps  //打印当前系统时间
```

* **打印 GC Case**

  ```shell
  -XX:+PrintGCCause
  [Full GC (Heap Inspection Initiated GC) //jmap -histo:live <pid>触发
  [Full GC (Heap Dump Initiated GC)       //jmap -dump:live <pid> 触发
  ```

* **在日志中输出每次垃圾回收前，应用未中断的执行时间**

  ```shell
  -XX:+PrintGCApplicationConcurrentTime
  ```

  输出形式：

  ```java
  Application time: 0.6862714 seconds
  ```

* **在日志中输出程序STW的暂停时间**

  在日志中输出垃圾垃圾回收期间应用**STW（Stop-The-World机制）**的暂停时间。

  > 可定位其他的 STW 操作，如 JIT 活动、偏向锁反擦除、特定的JVMTI操作等场景。

  ```shell
  -XX:+PrintGCApplicationStoppedTime 
  ```

  输出形式：

  ```java
  Total time for which application threads were stopped: 0.0468229 seconds。
  ```

* 在日志中输出线程到达安全点时间

  ```shell
  //需要配合-XX:+PrintGCApplicationStoppedTime一起使用
  -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1
  ```

* 在日志中输出堆中各代的内存大小分布

  ```shell
  -XX:+PrintHeapAtGC
  ```

* 在日志中输出答应 TLAB 相关信息

  ```shell
  -XX:+PrintTLAB
  ```

* 在日志里输出 Reference 相关内容

  ```
  -XX:+PrintReferenceGC
  ```

  输出形式：

  ```java
  [2019-07-12T20:59:22.184+0800: 1965.254: [SoftReference, 0 refs, 0.0011870 secs]
  2019-07-12T20:59:22.185+0800: 1965.255: [WeakReference, 4259 refs, 0.0007310 secs]
  2019-07-12T20:59:22.186+0800: 1965.256: [FinalReference, 11956 refs, 0.0029340 secs]
  2019-07-12T20:59:22.189+0800: 1965.259: [PhantomReference, 0 refs, 16 refs, 0.0039560 secs]
  2019-07-12T20:59:22.193+0800: 1965.263: [JNI Weak Reference, 0.0002220 secs]
   
  ```

* 在日志中输出对象年龄分布

  ```shell
  -XX:+PrintTenuringDistribution
  ```

  输出形式：

  ```java
  Desired survivor size 190119936 bytes, new threshold 15 (max 15)
  - age   1:   47865096 bytes,   47865096 total
  - age   2:    1662912 bytes,   49528008 total
  - age   3:    2637304 bytes,   52165312 total
  - age   4:    4456792 bytes,   56622104 total
  - age   5:    3278536 bytes,   59900640 total
  - age   6:    6639664 bytes,   66540304 total
  - age   7:    5271808 bytes,   71812112 total
  - age   8:    1220384 bytes,   73032496 total
  - age   9:     945152 bytes,   73977648 total
  - age  10:    1770400 bytes,   75748048 total
  - age  11:     165816 bytes,   75913864 total
  - age  12:     561376 bytes,   76475240 total
  - age  13:     607024 bytes,   77082264 total
  - age  14:     459776 bytes,   77542040 total
  - age  15:     313296 bytes,   77855336 total
  ```

  