# Redis

## 什么是 Redis

Redis 是一种支持是一种支持 Key-Value 等**多种数据结构**的存储系统。

## Redis 单线程问题

**单线程指的是网络请求模块使用了一个线程**（所以不需考虑并发安全性），即一个线程处理所有网络请求，**其他模块仍用了多个线程**。

## 使用 Redis 的原因

解决两个问题： **高性能、高并发**。

## 使用缓存带来的问题：

* 缓存与数据库双写不一致
* 缓存雪崩、缓存穿透
* 缓存并发竞争

下文详细解释。

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

## Redis 操作命令以及时间复杂度

| 类型       | 指令                                             | 功能                                                         | 时间复杂度        |
| ---------- | ------------------------------------------------ | ------------------------------------------------------------ | ----------------- |
| **String** | SET                                              | 为一个key设置value，可以配合EX/PX参数指定key的有效期，通过NX/XX参数针对key是否存在的情况进行区别操作 | O(1)              |
|            | GET                                              | 获取某个key对应的value                                       | O(1)              |
|            | GETSET                                           | 为一个key设置value，并返回该key的原value                     | O(1)              |
|            | MSET                                             | 为多个 key 设置 value                                        | O(N)              |
|            | MSETNX                                           | 同MSET，如果指定的key中有**任意一个**已存在，则不进行任何操作 | O(N)              |
|            | MGET                                             | 获取多个key对应的value                                       | O(N)              |
|            | INCR                                             | 将key对应的value值自增1，并返回自增后的值。只对可以转换为整型的String数据起作用。 | O(1)              |
|            | INCRBY                                           | 将key对应的value值自增指定的整型数值，并返回自增后的值。只对可以转换为整型的String数据起作用。 | O(1)              |
|            | DECR/DECRBY                                      | 同INCR/INCRBY，自增改为自减。                                | O(1)              |
| **List**   | LPUSH                                            | 向指定List的左侧（头部）插入一个或者多个元素，返回插入后的 List 长度，此时 N 为插入元素的数量。 | O(N)              |
|            | RPUSH                                            | 同LPUSH，向指定List的右侧（即尾部）插入1或多个元素           | O(N)              |
|            | LPOP                                             | 从指定List的左侧（即头部）移除一个元素并返回                 | O(1)              |
|            | RPOP                                             | 从指定List的左侧（即头部）移除一个元素并返回                 | O(1)              |
|            | LPUSHX/RPUSHX                                    | 与LPUSH/RPUSH类似，区别在于，LPUSHX/RPUSHX**操作的key如果不存在，则不会进行任何操作** | O(N)              |
|            | LLEN                                             | 返回List的长度                                               | O(1)              |
|            | LRANGE                                           | 返回指定List中指定范围的元素（双端包含，即`LRANGE key 0 10`会返回11个元素）<br />应尽可能控制一次获取的元素数量，一次获取过大范围的List元素会导致延迟，同时对长度不可预知的List，避免使用`LRANGE key 0 -1`这样的完整遍历操作。 | O(N)              |
|            | LINDEX**（谨慎使用，遍历整个List）**             | 返回指定List指定index上的元素，如果index越界，返回nil。index数值是回环的，即-1代表List最后一个位置，-2代表List倒数第二个位置. | **O(N)**          |
|            | LSET**（谨慎使用，遍历整个List）**               | 将指定List指定index上的元素设置为value，如果index越界则返回错误，时间复杂度O(N)，如果操作的是头/尾部的元素，则**时间复杂度为O(1)** | **O(N)**/**O(1)** |
|            | LINSERT**（谨慎使用）遍历整个List**              | 向指定List中指定元素之前/之后插入一个新元素，并返回操作后的List长度。如果指定的元素不存在，返回-1。如果指定key不存在，不会进行任何操作 | **O(N)**          |
| Hash       | HSET                                             | 将key对应的Hash中的field设置为value。如果该Hash不存在，会自动创建一个 | O(1)              |
|            | HGET                                             | 返回指定Hash中的filed字段的值                                | O(1)              |
|            | HMSET/HMGET                                      | 同HSET与HGET，可以批量操作同一个key下的多个filed，此时N为操作filed的数量 | O(N)              |
|            | HSETNX                                           | 同HSET，但如field已经存在，HSETNX不会进行任何操作            | O(1)              |
|            | HEXISTS                                          | 判断指定Hash中field是否存在，存在返回1，不存在返回0          | O(1)              |
|            | HDEL                                             | 删除指定Hash中的field（1个或多个），N为操作的field数量       | O(N)              |
|            | HINCRBY                                          | 同INCRBY命令，对指定Hash中的一个field进行INCRBY              | O(1)              |
|            | HGETALL**（谨慎使用，完整遍历对应Hash）**        | 返回指定Hash中所有的field-value对。返回结果为数组，数组中field和value交替出现 | **O(N)**          |
|            | HKEYS/HVALS**（谨慎使用，完整遍历对应Hash）**    | 返回指定Hash中所有的field/value                              | **O(N)**          |
| Set        | SADD                                             | 向指定Set中添加1个或多个member，如果指定Set不存在，会自动创建一个。N为添加的member个数 | O(N)              |
|            | SREM                                             | 指定Set中移除1个或多个member，N为移除的member个数            | O(N)              |
|            | SRANDMENBER                                      | 从指定Set中随机返回1个或多个member，N为返回的member个数      | O(N)              |
|            | SPOP                                             | 从指定Set中随机移除并返回count个member，N为移除的member个数  | O(N)              |
|            | SCARD                                            | 返回指定Set中的member个数                                    | O(1)              |
|            | SISMEMBER                                        | 判断指定的value是否存在于指定Set中                           | O(1)              |
|            | SMOVE                                            | 将指定member从一个Set移至另一个Set                           | O(1)              |
|            | SMEMBERS**（谨慎使用）**                         | 返回指定SET中所有的member                                    | **O(N)**          |
|            | SUNION/SUNIONSTORE**（谨慎使用）**               | 计算多个Set的**并集**并返回/存储至另一个Set中，N为参与计算的所有集合的总member数 | **O(N)**          |
|            | SINTER/SINTERSTORE**（谨慎使用）**               | 计算多个Set的**交集**并返回/存储至另一个Set中，N为参与计算的所有集合的总member数 | O(N)              |
|            | SDIFF/SDIFFSTORE**（谨慎使用）**                 | 计算1个Set与1或多个Set的**差集**并返回/存储至另一个Set中，N为参与计算的所有集合的总member数 | O(N)              |
| Sorted Set | ZADD                                             | 向指定Sorted Set中添加1个或多个member，M为添加的member数量，N为Sorted Set中的member数量 | O(Mlog(N))        |
|            | ZREM                                             | 从指定Sorted Set中删除1个或多个member，M为删除的member数量，N为Sorted Set中的member数量 | O(Mlog(N))        |
|            | ZCOUNT                                           | 返回指定Sorted Set中指定score范围内的member数量              | O(log(N))         |
|            | ZCARD                                            | 返回指定Sorted Set中的member数量                             | O(1)              |
|            | 返回指定Sorted Set中指定member的score            | O(1)                                                         |                   |
|            | ZRANK/ZREVRANK                                   | 返回指定member在Sorted Set中的排名，ZRANK返回按**升序排序**的排名，ZREVRANK则返回按**降序排序**的排名。 | O(log(N))         |
|            | ZINCREBY                                         | 同INCRBY，对指定Sorted Set中的指定member的score进行自增      | O(log(N))         |
|            | ZRANGE/ZREVRANGE**（谨慎使用）**                 | 返回指定Sorted Set中指定排名范围内的所有member，ZRANGE为按score升序排序，ZREVRANGE为按score降序排序，M为本次返回的member数 | O(log(N) + M)     |
|            | ZRANGEBYSCORE/ZREVRANGEBYSOCRE**（谨慎使用）**   | 返回指定Sorted Set中指定score范围内的所有member，返回结果以升序/降序排序，min和max可以指定为-inf和+inf，代表返回所有的member。 | O(log(N) + M)     |
|            | ZREMRANGEBYRANK/ZREMRANGEBYSCORE**（谨慎使用）** | 移除Sorted Set中指定排名范围/指定score范围内的所有member     | O(log(N) + M)     |

**总结：谨慎使用下述命令**

> **List**： `lindex`、`lset`、`linsert` （上述的三个命令的算法效率较低，需要对List进行遍历，命令的耗时无法预估，在List长度大的情况下耗时会明显增加，应谨慎使用。）
> **Hash**： `hgetall`、`hkeys`、`hvals`（都会对Hash进行完整遍历，Hash中的field数量与命令的耗时线性相关，对于尺寸不可预知的Hash，
> 应严格避免使用上面三个命令，而改为使用HSCAN命令进行游标式的遍历）
> **Set**： `smembers`、`sunion`、`sunionstore`、`sinter`、`sinterstore`、`sdiff`、`sdiffstore`（应谨慎使用，特别是在参与计算的Set尺寸不可知的情况下，应严格避免使用。
> 可以考虑通过SSCAN命令遍历获取相关Set的全部member）
> **Sorted Set**： `zrange`、`zrevrange`、`zrangebyscore`、`zrevrangebyscore`、`zremrangebyrank`、`zremrangebyscore`

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
	
	Redis 支持 String、Hash、List、Set、ZSet 结构。 Redis 内部使用一个 redisObject 对象来表示所有的 key 和 value

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

	Redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型，它采用 IO 多路复用机制 同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。

文件事件处理器包含 4 个部分：

* 多个 socket

* IO 多路复用程序

* 文件事件分派器

* 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

  	多个 socket 可能会并发不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器处理。

    	客户端与 Redis 的一次通信过程：

  ![](./image/Redis/redis-single-thread-model.png)

  	客户端 socket01 向 Redis 的 sever socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，IO多路复用程序监听到 sever socket 产生的事件后，将该事件压入队列中。文件事件分派器从队列中获取该事件，交给连接应答处理器。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令处理器关联。
		
  	假设此时客户端发送了一个 `set key value` 请求，此时Redis 中的 socket01 会产生`AE_READABLE` 事件，IO 多路复用程序将事件压入队列，此时事件分派器从队列中获取到该事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器获取 socket01 的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与命令回复处理器关联。
		
  	如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件与命令回复处理器的关联。

## Redis 过期策略

**Redis 的过期策略是： 定期删除+惰性删除**

### 定期删除

	定期删除是指 Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查是否过期，如果过期就删除。
	
	特殊情况：若一次性向 Redis 存储 10w 个key，都设置了过期时间，每隔几百毫秒，这样需要检查10w个key，性能消耗巨大，可能导致服务宕机。
	
	**注意：**这里不是每隔100ms 就遍历所有设置过期时间的 key，实际上***Redis每隔 100ms 随机抽取一些 key 来检查和删除的。***
	
	定期删除可能会导致很多过期的 key 到了时间并没有被删除掉，此时就需要依赖惰性删除了。

### 惰性删除

	在获取某个 key 时，redis 会检查一下，这个 key 如果设置了过期时间，判断是否过期，如果过期则会删除，不会返回任何东西。

> 获取可以 key 的时候，如果此时 key 已经过期，则删除 key， 不会返回任何东西。

	即使是 定期删除+惰性删除，也会有漏洞，假设此时定期删除漏掉了很多过期的 key，调用者在获取也没有检查，也没走惰性删除，此时如果有大量 key 堆积在内存，也有可能会使 Redis 内存块耗尽。所以此时需要 Redis 使用 **内存淘汰机制**

### 内存淘汰机制

#### 配置

我们通过配置redis.conf中的maxmemory这个值来开启内存淘汰功能。

```
maxmemory
```

值得注意的是，maxmemory为0的时候表示我们对Redis的内存使用没有限制。

```
maxmemory-policy noeviction
```

#### 动态改配置命令

设置最大内存

```
config set maxmemory 100000
```

设置淘汰策略

```
config set maxmemory-policy noeviction
```

#### 内存淘汰策略

根据应用场景，选择淘汰策略

	Redis 内存淘汰机制分为以下几种：（假设此时当内存不足以容纳新写数据时）

| 方式            | 方式原理                                                     | 是否常用               |
| --------------- | ------------------------------------------------------------ | ---------------------- |
| noeviction      | 新的写入操作会报错                                           | 不建议使用             |
| **allkeys-lru** | 在**键空间**中，移除最近最少未使用的 key                     | **这种方式时最常用的** |
| allkeys-random  | 在**键空间**中，随机移除某个 key。                           | 不常用                 |
| volatile-lru    | 在**设置了过期时间的键空间**中，移除最近最少未使用的 key     | 不常用                 |
| volatile-random | 在**设置了过期时间的键空间**中，随机移除某个 key             | 不常用                 |
| volatile-ttl    | 在**设置了过期时间的键空间**中，有**更早过期时间** 的key 优先移除 | 不常用                 |

### 手写 LRU 算法

手写一个简单的 LRU 算法（利用已有JDK数据结构构建 Java 版的 LRU 算法）

```java
class LRUCache<K, V> entends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;
    
    /**
     * 传递进来最多能缓存多少数据
     *
     * @param cacheSize 缓存大小
     */
    public LRUCache(int cacheSize) {
        // true 表示让 LinkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int)Math.ceil(cacheSize / 0.75) + 1, 0.75f, true)
    	CACHE_SIZE = cacheSize;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map 中的数据量大于指定的缓存个数时，就自动删除最老的数据。
        return size() > CACHE_SIZE
    }
}
```



## Redis 的高并发和高可用

* Redis 实现**高并发**主要依赖**主从架构**，一主多从，单个主服务器（master）用来写数据，多个从服务器（salve）用来查询数据。多个从实例可以提供**每秒 10w 的 QPS**。
* Redis **高可用**，如果是做主从架构，那么加上哨兵即可，就可以实现，任何一个实例宕机，可以进行主备切换。

## Redis 主从架构

单机的 Redis，可以承载 **上万~几万的 QPS**。

对于缓存来说，一般都是用来 **支撑*读高并发***的。所以此时可以设计成主从（master-slave）架构，一主多从，并且将数据复制到其他的 slave 节点，从节点负责读。所有的 **读请求全部走从节点**。这样也可以很轻松的实现水平扩容，**支撑读高并发**。

![](./image/Redis/redis-master-slave.png)

redis replication -> 主从架构 -> 读写分离 -> 水平扩容支撑读高并发。

### Redis replication(复制) 的核心机制

* Redis 采用一步方式复制数据到 slave 节点。

  **注意：**从 Redis 2.8 开始，slave node 会周期性地确认自己每次复制的数据量；

* 一个 master node 是可以配置多个 slave node 的；

* slave node 也可以连接其他的 slave node；

* slave node 做复制的时候，不会 block master node 的正产工作；

* slave node 在做复制的时候，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧的数据集，加载新的数据集，**此时会暂停对外服务**；

* slave node  主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。

**注意：**

* 如果采用了主从架构，**必须开启 master node 的持久化**，不建议用 slave node 作为 master node 的数据热备。因为存在一种情况，如果你关掉了 master 的持久化，可能在 master 宕机重启的时候，数据是空的，然后可能已经过复制，slave node 的数据也丢了。

* 对 master 需要做周全的备份方案。解决在本地所有文件丢了的情况下，可以从备份中挑选一份 rdb 去恢复 master，这样才能**确保Redis启动的时候，是有数据的**。**即使采用了高可用机制，slave node 可以自动接管 master node**，但是还是存在 slave node 还没检测到 master failure ，master node 就自动重启的情况，那么也有可能导致上面的 slave node 数据被清空。

### Redis 主从复制的核心原理

> 当启动一个 slave node 的时候，它会发送一个 `PSYNC` 命令给 master node。
>
> 如果 slave node 是初次连接到 master node，那么会触发一次 `full resynchronization` **全量复制**。此时 master node 会启动一个后台线程，开始生成一份 `RDB` 快照文件，同时还会**将从客户端 client 新收到的所有的写命令缓存在内存中**。`RDB` 文件生成完毕后，master 会将这个 `RDB` 发送给 slave，slave 会先**写入本地磁盘，然后再从本地磁盘加载到内存**中，接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。slave node 如果跟 master node 有网络故障，断开了连接，连接之后 master node 仅会复制给 slave 部分缺少的数据。

![](./image/Redis/redis-master-slave-replication.png)

#### 主从复制的端点续传

	从 Redis 2.8 开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制复制下去，而不是从头开始复制一份。
	
	master node 会在内存中维护一个 backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset	就保存在 backlog 中的。如果 master 和 slave 网络连接断掉，slave 会让 master 从上次 replica offset 开始继续复制，如果又找到对应的 offset，那么就会执行一次 `resynchronization`。

> 如果根据 host+ip 定位 master node，是不靠谱，如果 master node 重启或者数据出现了变化，那么 slave node 应该根据不同的 run id 区分。

#### 无磁盘化复制

	master 在内存中直接创建 `RDB`，然后发送给 slave，不会再自己本地落地磁盘了。只需要在配置文件中开启 `repl-diskless-sync yes` 即可。

```shell
repl-diskless-sync yes

# 等待 5s 后再开始复制，因为要等更多 slave 重新链接过来
repl-diskless-sync-delay 5
```

#### 过期 key 处理

	slave 不会过期 key，只会等待 master 过期 key。如果 master 过期了一个 key，或者通过 LRU 淘汰了一个 key，那条 del 命令发送给 slave。

### 复制的完整流程

	slave node 启动时，会在自己本地保存 master node 的信息，包括 master node 的 `host` 和 `ip`，但是复制流程没开始。
	
	slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现，就跟 master node 建立 socket 网络连接。然后 slave node 发送 `ping` 命令给 master node。如果 master 设置了 requirepass，那么 slave node 必须发送 masterauth 的口令进行认证。 master node **第一次执行全量复制**，将所有数据发送给 slave node。而在后续，master node 持续将写命令**异步复制**给 slave node。

![](./image/Redis/redis-master-slave-replication-detail.png)

#### 全量复制

* master 执行 `bgsave`，在本地生成一份 `RDB` 快照文件。
* master node6 将 `RDB` 快照文件发送给 slave node，如果 `RDB` 复制时间超过 60 秒（repl-timeout），那么 slave node 就会认为复制失败，可以适当调大这个参数（对于千兆网卡的机器，一般每秒传输 100MB，6G文件，很可能超过 60s）
* master node 再生成 `RDB` 时，会将所有新的写命令缓存在内存中，在 slave node 保存了 `RDB` 之后，再将新的写命令复制给 slave node。
* 如果复制期间，内存缓存区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败。

```shell
client-output-buffer-limit slave 256MB 64MB 60
```

* slave node 接收到 `RDB` 之后，清空自己的旧数据，然后重新加载 `RDB` 到自己的内存中，同时**基于旧的数据版本**对外提供服务。
* 如果 slave node 开启了 `AOF`，那么会立即执行 `BGREWRITEAOF`，重写 AOF。

### 增量复制

* 如果全量复制过程中，master-slave 网络连接断掉了，那么 slave 重新连接 master 时，会触发增量复制。
* master 直接从自己的 backlog 中获取部分丢失的数据，发送给 slave node，默认 backlog 就是 1MB。
* master 就是根据 slave 发送的 psync 中的 offset 来从 backlog 中获取 数据的。

#### Heartbeat —— 心跳机制

* 主从节点互相都会发送 heartbeat 信息。
* master 默认**每隔 10 秒**发送一次 heartbeat，slave node **每隔 1 秒** 发送一个heartbeat。

#### 异步复制

	master 每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node。

## Redis 如何才能做到高可用

**高可用：**如果系统在**99.99%**的时间里都能够持续对外提供服务，那么可以说该系统是高可用的。

如果一个 slave 挂掉了，是不会影响可用性的，还有其他的 slave 在提供相同数据下的相同的对外查询服务。

但是如果 master node 挂掉了，此时没有解决方案的话，slave node 没有 master node 给它们复制数据，系统相当于不可用了。

**Redis 的高可用架构：** `failover` **故障转移**，也可以叫做主备切换。

主备切换：master node 在故障时，自动检测，并且将每个 slave node 自动切换为 master node 的过程，称之为主备切换。这个过程实现了 Redis 在主从架构下的高可用。

### Redis 哨兵集群实现高可用

#### 哨兵的介绍

sentinel，中文名为哨兵。哨兵是 Redis 集群机构中非常重要的一个组件，主要有以下功能：

* **集群监控**：负责监控 Redis master 和 slave 进程是否正常工作。
* **消息通知**：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
* **故障转移**：如果 master node 挂掉了，会自动转移到 slave node 上。
* **配置中心**：如果故障转移发生了，通知 client 客户端新的 master 地址。

哨兵用于实现 Redis 集群的高可用，本身也是分布式的，作为哨兵集群去运行，互相协同工作。

* 故障转移时，判断一个 master node 是否宕机了，需要大部分哨兵都同意了才通过，涉及到了分布式选举的问题。
* 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的，因为如果一个作为高可用机制的重要组成部分的故障转移系统本身是单点的，那么是很荒唐的。

#### 哨兵的核心知识

* 哨兵至少需要 3 个实例，来保证自己的健壮性。
* 哨兵 + Redis 主从的部署架构，是**不能保证数据领丢失**的，只能保证 Redis 集群的高可用性。
* 对于哨兵 + Redis 主从这种复杂的部署架构，尽量都在测试环境和生产环境，都进行充足的测试和演练。

哨兵集群必须部署 >2 个节点，如果哨兵集群只部署了 2 个哨兵实例，`quorum = 1`。

```shell
+----+         +----+
| M1 |---------| R1 |
| S1 |         | S2 |
+----+         +----+
```

如果 `quorum = 1` ，如果 master 宕机， s1 和 s2 中只有一个哨兵认为 master 宕机了，就可以进行切换，同时 s1 和 s2 会选举出一个 哨兵来执行故障转移。但是同时这个时候，需要 `majority`，也就是大多数哨兵都是运行的

```
2 个哨兵，majority=2
3 个哨兵，majority=2
4 个哨兵，majority=2
5 个哨兵，majority=3
...
```

如果此时仅仅是 M1 进程宕机了，哨兵 s1 正常运行，那么故障转移是 OK 的。但是如果整个 M1 和 s1 运行的机器宕机了，那么哨兵只有一个，此时就没有 `majority` 来允许执行故障转移，虽然另外一台机器上还有一个 R1，但是故障转移不会执行。

经典的 3 节点哨兵集群：

```
       +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+   
```

配置 `quorum = 2`，如果  M1 所在机器宕机了，那么三个哨兵还剩下2个，s2 和 s3 可以一致认为 master 宕机了，然后选举出一个来执行故障转移，同时 3 个哨兵的 `majority` 是 2，所以还剩下的 2 个哨兵运行着，就可以允许执行故障转移。



#### Redis 哨兵主备切换的数据丢失问题

##### 两种情况会导致数据丢失

	主备切换的过程中，可能会导致数据的丢失：

* **异步复制导致的数据丢失：**因为 master -> slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时就会导致数据的丢失。

![](./image/Redis/async-replication-data-lose-case.png)

* **脑裂导致的数据丢失：** 

  	在脑裂发生时，某个 slave 被切换成 master，但是可能 client 还没来切换到新的 master，还继续向旧的master 写数据。因此**旧的 master 恢复时，会被作为一个新的 master 上去**，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。

    	***脑裂*** ：某个 master 所在的机器突然**脱离了正常的网络**，跟其他的 slave 机器不能连接，但是实际上 master 还在运行着。此时哨兵可能就会 **认为** master 宕机了，然后开始选举，将其它的 slave 选举为新的 master。 这个时候，整个分布式系统中，就存在了两个 master，称之为脑裂。

![](./image/Redis/redis-cluster-split-brain.png)

##### 数据丢失问题的解决方案

进行如下配置：

```shell
min-slaves-to-write 1
min-slaves-max-lag 10
```

上述表示：要求至少有一个 slave，数据复制和同步的延迟不能超过 10 秒。

如果说一旦所有的 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。

* **减少异步复制数据的丢失：**有了 `min-slaves-max-log` 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延迟太长了，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失损失降低到可控范围内。
* **减少脑裂的数据丢失：**如果一个 master 出现了脑裂，跟其它 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒数据。

#### sdown 和 odown 转换机制

* **sdown ** 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机
* **odown** 是客观宕机，如果 `quorum` 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机。

sdown 达成的条件很简单，如果一个哨兵 ping 一个 master，超过了 `is-master-down-after-milliseconds` 指定的毫秒数之后，就主观认为 master 宕机了；

如果 一个哨兵在指定时间内，收到了 `quorum` 数量的其他哨兵也认为那个 master 是 sdown 的消息，那么就认为是 odown 了。

#### 哨兵集群的自动发现机制

哨兵互相之间的发现，是通过 Redis 的 **pub/sub** 系统实现的，每个哨兵都会往 `_sentinel_:hello` 这个 channel 里发送消息，这时候所有的其他哨兵都可以消费到这个消息，并感知到其他哨兵的存在。

每隔 2s，每个哨兵都会往自己监控的某个 master+slaves 对应的 `_sentienl_:hello` channel里 **发送一个消息**，内容是自己的 **host、ip 和 runid** 还有对这个 master 的监控配置。

每个哨兵也会去 **监听** 自己监控的每个 master+slaves 对应的 `sentinel_:hello` channel，然后感知到同样监听这个 master+slaves 的其他哨兵的存在。

每个哨兵还会跟其他哨兵交换对 master 的监控配置，互相进行监控配置的同步。

#### Slave 配置的自动纠正

哨兵会负责自动纠正 slave 的一些配置，比如 slave 如果要成为潜在 master 候选人，哨兵会确保 slave 复制到现有 master 的数据；如果 slave 连接到了一个错误的 master 上，比如故障转移之后，那么哨兵会确保它们连接到了正确的 master 上。

#### Slave -> Master 选举算法

如果一个 master 被认为 down 了，而且 `majority` 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时先要选举一个 slave 来，会考虑 slave 的一些信息：

* 跟 master 断开连接的时长
* slave 的优先级
* 复制 offset 
* run id

如果一个 slave 跟 master 断开连接的时间超过了 `down-after-milliseconds` 的 10 倍，外加 master 宕机的时长，那么认为 slave 就被认为不适合选举为 master

```shell
(down-after-milliseconds * 10) + milliseonds_since_master_is_in_SDOWN_state
```

接下来会对 slave 进行排序：

* 按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
* 如果 slave priority 相同，那么看

* 如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave

#### quorum 和 majority

**quorum** : 确认 master **odown** 的最少的**哨兵数量**

**majority：**授权进行主从切换的最少的哨兵数量

每次一个哨兵要做主备切换，首先需要 `quonum` 数量的哨兵认为 odown，然后选举出一个哨兵来做切换，这个哨兵还得得到 `majority` 哨兵的授权，才能正式执行切换。

* `quonum` < `majority`，比如 5 个哨兵，`majority` 就是 3，`quorum` 设置为 2，那么最终共有 3 个哨兵授权就可以执行切换。
* `quonum` >= `majority`，那么必须 `quonum` 数量的哨兵都授权，比如 5 个哨兵，`quonum` 是 5，那么必须 5 个哨兵都同意授权，才能执行切换

#### configuration epoch

哨兵会对一套 Redis cluster + slaves 进行监控，有相应的监控的配置。

执行切换的那个哨兵，会从要切换到的新的 master（slave -> master） 那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。

如果第一个选举出的哨兵切换失败了，那么其他的哨兵，会等待 `failover-timeout` 事件，然后接替继续执行切换，此时会重新获取一个新的 configuration epoch，作为新的 version 号。

#### configuration 传播

哨兵完成切换之后，会在做自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过上文提到的 `pub/sub` 消息机制。

所以在之前的 version 号就很重要了，因为各种消息都是通过一个 **channel** 去发布和监听的，所以一个哨兵完成一次新的切换之后，新的 master 配置是跟着新的 version 号的。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。

## Redis集群的几种实现方式如下：

- 客户端分片，如redis的Java客户端jedis也是支持的，使用一致性hash
- 基于代理的分片，如codis和Twemproxy
- 路由查询， redis-cluster

### 客户端分片

Redis Sharding是Redis Cluster出来之前，业界普遍使用的多Redis实例集群方法。其主要思想是采用**哈希算法**将Redis数据的key进行散列，通过hash函数，特定的key会映射到特定的Redis节点上。java redis客户端驱动jedis，支持Redis Sharding功能，即ShardedJedis以及结合缓存池的ShardedJedisPool。

Redis Sentinel提供了主备模式下Redis监控、故障转移功能达到系统的高可用性。在主Redis宕机时，备Redis接管过来，上升为主Redis，继续提供服务。主备共同组成一个Redis节点，通过自动故障转移，保证了节点的高可用性。

客户端sharding技术其优势在于非常简单，服务端的Redis实例彼此独立，相互无关联，每个Redis实例像单服务器一样运行，非常容易线性扩展，系统的灵活性很强。

客户端sharding的劣势也是很明显的。由于sharding处理放到客户端，规模进一步扩大时给运维带来挑战。客户端sharding不支持动态增删节点。服务端Redis实例群拓扑结构有变化时，每个客户端都需要更新调整。连接不能共享，当应用规模增大时，资源浪费制约优化。

### 基于代理的分片

客户端发送请求到一个代理组件，代理解析客户端的数据，并将请求转发至正确的节点，最后将结果回复给客户端。

该模式的特性如下：

- 透明接入，业务程序不用关心后端Redis实例，切换成本低。
- Proxy 的逻辑和存储的逻辑是隔离的。
- 代理层多了一次转发，性能有所损耗。

### 路由查询

Redis Cluster是一种服务器Sharding技术，3.0版本开始正式提供。Redis Cluster并没有使用一致性hash，而是采用slot(槽)的概念，一共分成16384个槽。将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行。当客户端操作的key没有分配到该node上时，就像操作单一Redis实例一样，当客户端操作的key没有分配到该node上时，Redis会返回转向指令，指向正确的node，这有点儿像浏览器页面的302 redirect跳转。

Redis集群，要保证16384个槽对应的node都正常工作，如果某个node发生故障，那它负责的slots也就失效，整个集群将不能工作。为了增加集群的可访问性，官方推荐的方案是将node配置成主从结构，即一个master主节点，挂n个slave从节点。这时，如果主节点失效，Redis Cluster会根据选举算法从slave节点中选择一个上升为主节点，整个集群继续对外提供服务。

特点：

- 无中心架构，支持动态扩容，对业务透明
- 具备Sentinel的监控和自动Failover能力
- 客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
- 高性能，客户端直连redis服务，免去了proxy代理的损耗

缺点：

* 运维也很复杂，数据迁移需要人工干预
* 只能使用0号数据库
* 不支持批量操作
* 分布式逻辑和存储模块耦合等。

## Redis Cluster

- 自动将数据进行分片，每个 master 上放一部分数据
- 提供内置的高可用支持，部分 master 不可用时，还是可以继续工作的

在 redis cluster 架构下，每个 redis 要放开两个端口号，比如一个是 6379，另外一个就是 加1w 的端口号，比如 16379。

16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议，`gossip` 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

#### 安装与配置

官方推荐集群至少需要六个节点，即三主三从。六个节点的配置文件基本相同，只需要修改端口号。

```conf
port 7000
cluster-enabled yes  #开启集群模式
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

**配置 JedisClusterConfig**

```java
@Configuration
public class JedisClusterConfig {
    private static Logger logger = LoggerFactory.getLogger(JedisClusterConfig.class);

    @Value("${redis.cluster.nodes}")
    private String clusterNodes;

    @Value("${redis.cluster.timeout}")
    private int timeout;

    @Value("${redis.cluster.max-redirects}")
    private int redirects;
    
    @Autowired
    private JedisPoolConfig jedisPoolConfig;

    @Bean
    public RedisClusterConfiguration getClusterConfiguration() {
        Map<String, Object> source = new HashMap();

        source.put("spring.redis.cluster.nodes", clusterNodes);
        logger.info("clusterNodes: {}", clusterNodes);
        source.put("spring.redis.cluster.max-redirects", redirects);
        return new RedisClusterConfiguration(new MapPropertySource("RedisClusterConfiguration", source));
    }

    @Bean
    public JedisConnectionFactory getConnectionFactory() {
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(getClusterConfiguration());
        jedisConnectionFactory.setTimeout(timeout);
        return jedisConnectionFactory;
    }

    @Bean
    public JedisClusterConnection getJedisClusterConnection() {
        return (JedisClusterConnection) getConnectionFactory().getConnection();
    }

    @Bean
    public RedisTemplate getRedisTemplate() {
        RedisTemplate clusterTemplate = new RedisTemplate();
        clusterTemplate.setConnectionFactory(getConnectionFactory());
        clusterTemplate.setKeySerializer(new StringRedisSerializer());
        clusterTemplate.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return clusterTemplate;

    }

}
```

可以配置密码，cluster对密码支持不太友好，如果对集群设置密码，那么requirepass和masterauth都需要设置，否则发生主从切换时，就会遇到授权问题。

**yml 配置**

```yaml
redis:
  cluster:
    enabled: true
    timeout: 2000
    max-redirects: 8
    nodes: 127.0.0.1:7000,127.0.0.1:7001
```

主要配置了redis cluster的节点、超时时间等。

**使用 RedisTemplate**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisConfigTest {
    @Autowired
    RedisTemplate redisTemplate;
    
    @Test
    public void clusterTest() {
      redisTemplate.opsForValue().set("foo", "bar");
      System.out.println(redisTemplate.opsForValue().get("foo"));
    }
}
```

#### Redis Cluster 内部通信机制

集群元数据维护方式：

* 集中式

  集中式是将集群元数据（节点信息、故障等等）几种存储在某个节点上。集中式元数据集中存储的一个典型代表，就是大数据领域的 `storm`。它是分布式的大数据实时计算引擎，是集中式的元数据存储的结构，底层基于 zookeeper（分布式协调的中间件）对所有元数据进行存储维护。

  **好处**：

  * 元数据的读取和更新，时效性非常好，一旦元数据出现了变更，就立即更新到集中式的存储中，其它节点读取的时候就可以感知到；

  **缺点**：

  * 所有的元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力。

  ![](./image/Redis/元数据维护集中式.png)

* gossip 协议

  redis 维护集群元数据采用另一个方式， `gossip` 协议，所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。

  **优点**：

  * 元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续，打到所有节点上去更新，降低了压力；

  **缺点**：

  * 元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。

  ![](./image/Redis/元数据维护gossip协议.png)

**基本通信原理**

Redis Cluster 节点采用 gossip 协议进行通信。

#### redis-gossip

- 10000 端口
  每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如 7001，那么用于节点间通信的就是 17001 端口。每个节点每隔一段时间都会往另外几个节点发送 `ping` 消息，同时其它几个节点接收到 `ping` 之后返回 `pong`。
- 交换的信息
  信息包括故障信息，节点的增加和删除，hash slot 信息 等等。

#### gossip 协议

gossip 协议包含多种消息，包含 `ping`,`pong`,`meet`,`fail` 等等。

- meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。

```conf
redis-trib.rb add-node
```

其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集群。

- ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
- pong：返回 ping 和 meeet，包含自己的状态和其它信息，也用于信息广播和更新。
- fail：某个节点判断另一个节点 fail 之后，就发送 fail 给其它节点，通知其它节点说，某个节点宕机啦。

**ping 消息深入**

ping 时要携带一些元数据，如果很频繁，可能会加重网络负担。

每个节点每秒会执行 10 次 ping，每次会选择 5 个最久没有通信的其它节点。当然如果发现某个节点通信延时达到了 `cluster_node_timeout / 2`，那么立即发送 ping，避免数据交换延时过长，落后的时间太长了。比如说，两个节点之间都 10 分钟没有交换数据了，那么整个集群处于严重的元数据不一致的情况，就会有问题。所以 `cluster_node_timeout` 可以调节，如果调得比较大，那么会降低 ping 的频率。

每次 ping，会带上自己节点的信息，还有就是带上 1/10 其它节点的信息，发送出去，进行交换。至少包含 `3` 个其它节点的信息，最多包含`总结点-2` 个其它节点的信息。

#### 分布式寻址算法

- hash 算法（大量缓存重建）
- 一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）
- redis cluster 的 hash slot 算法

**hash 算法**

来了一个 key，首先计算 hash 值，然后对节点数取模。然后打在不同的 master 节点上。

**缺点**：一旦某一个 master 节点宕机，所有请求过来，都会基于最新的剩余 master 节点数去取模，尝试去取数据。这会导致**大部分的请求过来，全部无法拿到有效的缓存**，导致大量的流量涌入数据库。

**一致性 hash 算法**

一致性 hash 算法将整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织，下一步将各个 master 节点（使用服务器的 ip 或主机名）进行 hash。这样就能确定每个节点在其哈希环上的位置。

来了一个 key，首先计算 hash 值，并确定此数据在环上的位置，从此位置沿环**顺时针“行走”**，遇到的第一个 master 节点就是 key 所在位置。

在一致性哈希算法中，如果一个节点挂了，受影响的数据仅仅是此节点到环空间前一个节点（沿着逆时针方向行走遇到的第一个节点）之间的数据，其它不受影响。增加一个节点也同理。

**缺点**：一致性哈希算法在节点太少时，容易因为节点分布不均匀而造成**缓存热点**的问题。

为了解决这种热点问题，一致性 hash 算法引入了**虚拟节点机制**，即对每一个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点。这样就实现了数据的均匀分布，负载均衡。

<img src="./image/Redis/一致性hash算法.png" style="zoom:50%;" />

**redis cluster 的 hash slot 算法**

edis cluster 有固定的 `16384` 个 hash slot，对每个 `key` 计算 `CRC16` 值，然后对 `16384` 取模，可以获取 key 对应的 hash slot。

redis cluster 中每个 master 都会持有部分 slot，比如有 3 个 master，那么可能每个 master 持有 5000 多个 hash slot。

**hash slot 让 node 的增加和移除很简单**：

* 增加一个 master，就将其他 master 的 hash slot 移动部分过去
* 减少一个 master，就将它的 hash slot 移动到其他 master 上去。

移动 hash slot 的成本是非常低的。客户端的 api，可以对指定的数据，让他们走同一个 hash slot，通过 `hash tag` 来实现。

任何一台机器宕机，另外两个节点，不影响的。因为 key 找的是 hash slot，不是机器。

![](./image/Redis/RedisHashSlot.png)

#### Redis cluster 的高可用与主备切换原理

redis cluster 的高可用的原理，几乎跟哨兵是类似的

**判断节点宕机**

如果一个节点认为另外一个节点宕机，那么就是 `pfail`，**主观宕机**。如果多个节点都认为另外一个节点宕机了，那么就是 `fail`，**客观宕机**，跟哨兵的原理几乎一样，sdown，odown。

在 `cluster-node-timeout` 内，某个节点一直没有返回 `pong`，那么就被认为 `pfail`。

如果一个节点认为某个节点 `pfail` 了，那么会在 `gossip ping` 消息中，`ping` 给其他节点，如果**超过半数**的节点都认为 `pfail` 了，那么就会变成 `fail`。

**从节点过滤**

对宕机的 master node，从其所有的 slave node 中，选择一个切换成 master node。

检查每个 slave node 与 master node 断开连接的时间，如果超过了 `cluster-node-timeout * cluster-slave-validity-factor`，那么就**没有资格**切换成 `master`。

**从节点选举**

每个从节点，都根据自己对 master 复制数据的 offset，来设置一个选举时间，offset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。

所有的 master node 开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node`（N/2 + 1）`都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。

从节点执行主备切换，从节点切换为主节点。

**与哨兵比较**

整个流程跟哨兵相比，非常类似，所以说，redis cluster 功能强大，直接集成了 replication 和 sentinel 的功能。