# Redis

## 什么是 Redis

Redis 是一种支持是一种支持 Key-Value 等**多种数据结构**的存储系统。

## 应用场景 

* **缓存**
* **消息的发布或订阅**（消息通知）
* **消息队列**，例如：支付，活动排行榜、计数等
* 商品列表、评论列表

## 支持的数据类型

* 字符串 String
* 哈希 Hash
* 列表 List
* 集合 Set
* 有序集合 ZSet

### 1. 字符串 String <key, value>

Redis 最基本的数据类型，一个键对应一个值，需要注意的是**一个键值最大存储512MB**

![](.\image\Redis\StringOperation.png)

### 2. 哈希 Hash <key, field, value>

Redis Hash 是一个键值对的集合，是一个 String 类型的 field 和 value 的映射表，适合存储对象

![](./image/Redis/HashOperation.png)

### 3. 列表 List <key, value>

Redis 的简单的字符串列表，部分功能类似于双向链表，按照插入的顺序排序

可存储一些列表性的数据结构，例如：粉丝列表、文章的评论列表等

还可以做一个简单的消息队列，从 List 的头放入消息，List 的尾巴消费。

```shell
lpush mylist 1
lpush mylist 2
lpush mylist 3 4 5

# 1
rpop mylist
```

基本操作：

![](./image/Redis/ListOperation.png)

### 4. 集合 Set <key, value>

![](./image/Redis/SetStructure.png)

Redis 中字符串类型的**无序集合**，**不可重复**。

* 可以使用 Set 做去重，在分布式情况下，可以使用 Redis 进行全局的 Set 去重。

* 还可以基于 Set 做交集、并集、查集的操作。例如找出两人的共同好友：

  ```
  #-------操作一个set-------
  # 添加元素
  sadd mySet 1
  
  # 查看全部元素
  smembers mySet
  
  # 判断是否包含某个值
  sismember mySet 3
  
  # 删除某个/些元素
  srem mySet 1
  srem mySet 2 4
  
  # 查看元素个数
  scard mySet
  
  # 随机删除一个元素
  spop mySet
  
  #-------操作多个set-------
  # 将一个set的元素移动到另外一个set
  smove yourSet mySet 2
  
  # 求两set的交集
  sinter yourSet mySet
  
  # 求两set的并集
  sunion yourSet mySet
  
  # 求在yourSet中而不在mySet中的元素
  sdiff yourSet mySet
  ```


* 基本操作：

![](./image/Redis/SetOperation.png)

### 5. 有序集合 ZSet <key, value>

![](./image/Redis/ZSetStructure.png)

Redis 中 String 类型的有序集合，不可重复。

有序集合中的每个元素都需要制定一个分数（**权值**），根据分数对元素进行**升序排序**（**默认是升序，所以需要 rev 改为降序`zrevrange `**），如果多个元素有相同的分数，则按照字典序进行升序排序，适合实现排名的场景。

```shell
zadd board 85 zhangsan
zadd board 72 lisi
zadd board 96 wangwu
zadd board 63 zhaoliu

# 获取排名前三的用户（默认是升序，所以需要 rev 改为降序）
zrevrange board 0 3

# 获取某用户的排名
zrank board zhaoliu
# 获取某用户的排名
zrank board zhaoliu
```



* 基本操作：

![](./image/Redis/ZSetOperation.png)

## Redis 服务相关的命令

```shell
select # 选择数据库(数据库编号0-15)
info # Redis 的信息
config get # 获取数据库的配置
flushdb # 删除当前数据库的所有 key
flushall # 删除当前 Redis 下所有数据库的 key
```

**[常用命令](http://www.cnblogs.com/themost/p/8464490.html)**

## Redis 的发布与订阅

Redis 的发布与订阅（发布/订阅）是它的一种消息通信模式，一方发送消息，一方接收消息。如下图所示：

![](./image/Redis/publish.png)

如果有新的消息发送给频道1时，就会将消息发送给订阅它的三个客户端：

![](./image/Redis/publishNewMsg.png)

### Redis 订阅与发布的 5 个命令：

| 命令         | 用例与描述                                                   |
| ------------ | ------------------------------------------------------------ |
| SUBSCRIBE    | subscribe channel [channel ...] —— 订阅给定的一个或者多个频道 |
| UBSUBSCRIBE  | unsubscribe channel [channel ...] —— 退订一个或多个频道      |
| PUBLISH      | publish channel  message —— 向给定频道发送消息               |
| PSUBSCRIBE   | psubscribe pattern [pattern...]  —— 订阅给定模式匹配的所有频道 |
| PUNSUBSCRIBE | punsubscribe pattern [pattern ...] —— 退订一个或多个给定模式的频道，如果执行时没有给定任何模式，则退订所有模式 |

## Redis 的持久化

Redis 持久化的有两种方式：

* 快照（RDB）：

  可以将存在于某一时刻的所有数据都写入硬盘中。

  默认存放于 dump.rdb

* 仅附加文件（AOF）

  在执行**写命令**时，将被执行写命令复制到硬盘里。

这两种持久化方式既可以单独使用，也可以同时使用。

**实际操作**

* RDB 方式：（**配置文件中默认开启**）

  * 客户端向 Redis 发送 **`BGSAVE`** 命令，创建快照。

    Redis 会调用 **`FORK`** 创建一个子进程，然后子进程负责将快照写入硬盘，而父进程则继续处理命令请求

  * 客户端还可以向 Redis 发送 **`SAVE`** 命令，**若 Redis 接收到 SAVE 命令，那么 Redis 服务器在快照创建完毕之前将不再响应任何其他命令**，**`SAVE`**命令并不常用，只有在没有足够内存去执行 BGSAVE 命令的情况下，或者在无所谓等待时间的情况下使用。

    ```shell
    save 60 1000
    ```

    设置Redis从最近一次创建快照之后算起，每当**60 秒内有1000 个写入命令**，这个条件满足时，Redis 会自动触发快照的 BGSAVE 命令，若用户设置了多个 save 配置，则只要满族任意一个配置，就会触发 BGSAVE 命令。

  * Redis 通过 **`SHUTDOWN`**命令接收到关闭服务器命令时，或者接收到 **`TERM`**信号时，会执行一个 SAVE 命令，阻塞所有客户端，不再执行客户端发送的任何命令，在SAVE 命令结束后关闭服务器。

  * 在 Redis 集群配置的情况下，某一服务器向其他服务器发送的 **`SYNC`** 命令时，如果主服务器目前没有执行 **`BGSAVE`** 操作，或者主服务器并非刚刚执行完 **`BGSAVE`** 操作，那么主服务器就会执行 **`BGSAVE`**  命令。 

* AOF 方式：

  需要在配置文件中打开配置：

  ```shell
  appendonly yes
  ```

  并且可以设置 AOF 的同步频率进行设置

  | 选项     | 同步频率                                                     |
  | -------- | ------------------------------------------------------------ |
  | always   | 每个 Redis 写命令都要写入硬盘，这样会严重降低 Redis 的速度。 |
  | everysec | 每秒执行一次同步，显式地将多个写命令同步到硬盘**<兼顾数据安全和写入性能>** |
  | no       | 让操作系统来决定何时进行同步                                 |



## Redis 与 Memcached 的比较

|              | Memcached                                          | Redis                                                        |
| ------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| 线程         | 多线程                                             | 单线程                                                       |
| 性能         | 在 100K 以上的数据中，Memcached 的性能要高于 Redis | Redis 为单线程工作模式，只能使用单核，而 Memcached 支持多线程，可以使用多核。 |
| 数据结构     | key-value 结构                                     | String、Hash、List、Set、ZSet                                |
| 内存使用效率 | key-value 存储效率上，Memcached 高                 | Hash 结构存储时，Redis 效率高                                |
| 数据备份恢复 | 服务宕机后，数据不可恢复                           | 1. 服务宕机后，可通过 AOF 的方式恢复；2. 支持数据备份，即 Master-Slave 主从模式数据备份 |
| 数据存储     | 内存，不可持久化，不可超过内存大小                 | 内存，可通过 RDB/AOF 的方式持久化                            |
| 集群         | 本身不支持集群，依赖客户端集群                     | 支持集群                                                     |

### 一、性能

	Redis 为单线程工作模式，只能使用单核，而 Memcached 支持多线程，可以使用多核。

	Redis 平均在每一个核存储小数据时比 Memcached 性能高；

	在 100K 以上的数据中，Memcached 的性能要高于 Redis。

### 二、数据结构

	Memcached 只支持简单的 key-value 结构的数据，

	Redis 支持 String、Hash、List、Set、ZSet 结构。 **Redis 内部使用一个 redisObject 对象来表示所有的 key 和 value**

### 三、内存使用效率

	key-value 时，Memcached 更胜一筹，

        Redis 使用 Hash 结构时，则 Redis 效率更高

### 四、Redis 支持服务端的操作

	Redis 相比 Memcached，拥有更多的数据结构，并支持丰富的数据操作。

	在 Memcached 中，需要将数据拿到客户端进行类似修改再 Set 回去，**序列化再反序列化**，这样大大增加了网络 IO 的次数和数据体积。

### 五、数据备份和恢复

	Memcached 挂掉后，数据不可恢复；

	Redis 数据丢失后，可以通过 AOF 的方式恢复，并且 Redis 支持数据的备份 —— Master-Slave 主从模式的数据备份。

### 六、数据存储

	Redis 与 Memcached 都是将数据存储放在内存中。

 * Memcached 还可以用于缓存图片、视频等；挂掉后数据丢失，数据不能超过内存大小

 * Redis 有部分存放在硬盘上，这样能够保证数据的持久性，**Memcached 不支持化**；

   当系统物理内存用完时：

   * Redis 可以将很久没用到的value 交换到硬盘上；
   * Memcached 会抹掉前面的数据。

### 七、内存管理机制

	传统C语言的 **malloc/free 函数**是最常用的分配和释放内存的方法，但是这种方法存在很大缺陷：

* **首先，对于开发人员来说不匹配的 malloc和free 容易造成内存的泄露；**

* **其次，频繁的调用会造成大量的内存的碎片无法重新利用，降低内存利用；**

* **最后最为系统调用，其系统开销远远大于一般函数调用**

  所以，为了提高内存的管理效率，高效的内存管理方案都不会直接使用 malloc/free 调用

#### Memcached 默认使用 **Slab Allocation** 机制管理内存

	主要思想是 **按照预先规定的大小，将分配内存分割成特定长度的块一存储响应长度的 key-value 数据机构，以完全解决内存碎片问题**。

![](./image/Redis/MemcachedStructure.png)

	如图所示，这种机制首先会**从操作系统申请一大块区域，并将其分割成各种尺寸的块 Chunk，并把尺寸相同的块分成组 Slab Class**。**Chunk 就是用来存储 key-value数据的最小单位**。

	当Memcached 接收到客户端发送过来的数据首先会根据收到数据的大小，选择一个最合适的 Slab Class，然后通过查询 Memcached 保存着的该 Slab Class 内空闲 Chunk 的列表就可以找到一个以用于存储数据的 Chunk。当一条数据过期或丢弃时，该记录所占用的 Chunk 就可以回收，重新添加到空闲列表中。

	这种方式的效率高，不会造成内存碎片，但是最大的缺点就是**会导致空间浪费，因为每个Chunk 都分配了特定长度的内存空间，所以边长数据无法充分利用这些空间。**

#### Redis 采用的是包装的malloc/free

	Redis 通过源码中的 zmalloc.h 和 zmalloc.c 两个文件实现，Redis 为了方便内存的管理，在分配一块内存后，会将这块内存的大小存入内存块的头部。

![](./image/Redis/RedisStructure.png)

	如图所示，real_ptr 是 Redis 调用 malloc 后返回的指针，redis 将内存块的大小 size 存入头部，size 所占据的内存大小是一直的，为 size_t 类型的长度，然后返回 ret_ptr。当需要释放内存的时候，ret_ptr 被传给内存管理程序。 通过 ret_ptr，程序可以很容易算出 real_ptr 的值，然后将 real_ptr 传给 free 释放内存。

## 为什么 Redis 单线程模型也能保持高效率？

* 纯内存操作；
* 核心是基于非阻塞的 IO 多路复用机制；
* 单线程反而避免了多线程的频繁上下文切换问题。

## Redis 的线程模型

	Redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型，它采用 **IO 多路复用机制** 同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。

	文件事件处理器包含 4 个部分：

* 多个 socket

* IO 多路复用程序

* 文件事件分派器

* 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

  	多个 socket 可能会并发不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器处理。

  	客户端与 Redis 的一次通信过程：

  ![](./image/Redis/redis-single-thread-model.png)

  	客户端 socket01 向 Redis 的 sever socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，IO多路复用程序监听到 sever socket 产生的事件后，将该事件压入队列中。文件事件分派器从队列中获取该事件，交给**连接应答处理器**。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令处理器关联。

  	假设此时客户端发送了一个 `set key value` 请求，此时Redis 中的 socket01 会产生`AE_READABLE` 事件，IO 多路复用程序将事件压入队列，此时事件分派器从队列中获取到该事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器获取 socket01 的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与命令回复处理器关联。

  	如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件与命令回复处理器的关联。