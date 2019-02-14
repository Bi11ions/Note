![](.\image\Zookeeper\Zookeeper.jpg)

# Zookeeper

## 什么是 Zookeeper

Zookeeper 是一种提供配置管理、分布式协同以及命名的中心化服务，这些提供的功能都是分布式系统中非常底层而且必不可少的基本功能。

**抽象模型：** Zookeeper 提供一个多层节点命名空间（**节点称为：znode**），每个节点都用一个以斜杠（**/**）分隔的路径标识，而且每个节点的父节点（根节点除外），都类似于文件系统。

例如：/foo/doo 这个标识一个 znode，它的父节点为 /foo，它的父父节点为 /，而 / 为根节点没有父节点。

与文件系统不同的是，这些节点都可以设置关联的数据，二文件系统中只有文件节点可以存放数据而目录节点不行。

Zookeeper 为了保证高吞吐和低延迟，在内存中维护了这个树状的目录结构，这种特性使得 Zookeeper 不能存放大量的数据，每个节点的存放数据的上线为 1M。

## Zookeeper 的节点的特性：

* **有序节点**：假如当前有一个父节点为 /lock，我们可以在这个父节点下面创建子节点；zookeeper 提供了一个可选的有序特性，例如我们可以在创建子节点“/lock/node-”并且指定有序，那么 Zookeeper 在生成子节点时会根据当前的子节点的数量自动添加整数序号，也就是说如果第一个创建的子节点，那么生成的子节点为“/lock/node-0000000000”，下一个节点则为“/lock/node-0000000001”，以此类推。
* **临时节点**：客户端可以建立一个临时节点，在会话结束或者会话超时后，Zookeeper 会自动删除该节点。
* **事件监听**：在读取数据时，我们可以同时对节点设置时间监听，当节点数据或者结构改变时，Zookeeper 会通知客户端。当前 Zookeeper 有如下四种事件监听：
  1. 节点创建；
  2. 节点删除；
  3. 节点数据修改；
  4. 子节点变更。

## Zookeeper 的使用场景：

* 分布式协调
* 分布式锁
* 元数据/配置信息管理
* HA——高可用性

### 分布式协调

**需求：**A系统发送请求到mq，然后B系统进行消费之后处理。A系统如何知道B系统的处理结果？

**解决方案：**使用Zookeeper 进行解决 —— A系统在发送请求之后可以在 Zookeeper 上**对某个节点的值注册个监听器**，一旦 B系统处理完成之后就修改 Zookeeper 中 A系统标记的节点的值，A立马就可以收到通知。

![](.\image\Zookeeper\zookeeper-distributed-coordination.png)

### 分布式锁

**需求：**在分布式需求中，数据一致性的问题不可避免，要求使用 Zookeeper 实现分布式锁。

	例如：对于某一数据连续发出修改操作，两台机器同时收到了请求，但是只能一台机器先执行玩另外一个机器再执行。

**解决方案：**一个机器接收到了请求之后先获取 Zookeeper 上的一把分布式锁——可以创建一个 Znode，接着执行操作，另外一个机器也去尝试创建那个 Znode，结果发现创建不了，因为被别人创建了，所以只能等待，等另外一台机器执行完了再执行。

![](.\image\Zookeeper\zookeeper-distributed-lock-demo.png)

#### 实现方式

**分布式锁优化方式一：**利用 Zookeeper 的节点特性，假设锁空间的根节点为 /lock，

1. 客户端连接 Zookeeper，并在 /lock 下创建**临时且有序的**子节点，第一个客户端对应的子节点为 /lock/lock-0000000000，第二个为 /lock/lock-0000000001，以此类推。
2. 客户端获取 /lock 下的子节点列表，判断自己创建的子节点是否为当前子节点列表中**序号最小的**子节点，如果是则认为获取锁，否则监听 /lock 的子节点的变更消息，获取子节点变更通知后除服此步骤直至获得锁；
3. 执行业务代码
4. 完成业务流程后，删除对应的子节点释放锁。

**问题：**步骤2中，获取子节点列表与设置监听这两个操作是否为原子操作？

	假设客户端A对应的子节点为 /lock/lock-0000000000，客户端B对应的子节点为/lock/lock-0000000001，在客户端B获取子节点列表时发现自己不是序号最小的，但是在设置监听器前，客户端A已经完成了业务流程并删除了子节点 /lock/lock-0000000000，客户端B此时再设置监听器从而造成了永远等待。

**解决方案：**Zookeeper 提供的API中，对节点设置监听器的操作与读操作是原子操作，保证不会丢失时间。

**分布式锁优化方式二：**

	对于优化方式一有一个极大的优化点：假设当前有1000个节点在等待锁，如果获得锁客户端释放锁时，这1000个客户端都会被唤醒，这种请求称之为“羊群效应”，在这种情况下，Zookeeper 需要通知1000个客户端，这会阻塞其他操作，**优化方案**：只唤醒新的最小节点对应的客户端。

**解决方案：**将优化方式一的步骤2中的设置监听操作，改为对自己之前以为的子节点设置监听。

### 元数据/配置信息管理

Zookeeper 可以用作很多系统的配置信息管理，例如 kafka、storm 等等很多分布式系统都会用 Zookeeper 来做元数据、配置信息的管理，包括 Dubbo 的注册中心也是支持 Zookeeper。

![](.\image\Zookeeper\zookeeper-meta-data-manage.png)

**原理**：利用Zookeeper节点的监控特性（watch功能），client 监控 Zookeeper 上的节点（znode），当节点变动时，client 会受到变动事件和变动后的内容，基于 watch 功能，我们可以给服务器集群中的所有机器（client）都注册 watch 事件，监控特定 znode，节点中存储部署代码的配置信息，需要更新代码时，修改znode中的值，服务器集群中的每一台 Sever 都会受到代码更新事件，然后触发调用，更新目标代码，也可以很容易的横向拓展，可以随意得增删机器，机器启动的时候注册监控注册节点时间即可。[具体可参考博客（Java）](https://blog.csdn.net/u011320740/article/details/78742625)，[Python 实现，通过 Git 修改配置](https://www.cnblogs.com/iforever/p/9095095.html)

#### 实际操作（模拟）

1. Zookeeper 配置

   * 创建三个文件：

     * `/path/to/zookeeper/conf/zoo1.cfg`，
     * `/path/to/zookeeper/conf/zoo2.cfg`，
     * `/path/to/zookeeper/conf/zoo3.cfg` ，配置分别如下：

     `zoo1.cfg`

     ```
     tickTime=2000
     initLimit=10
     syncLimit=5
     dataDir=/tmp/zk1/data
     dataLogDir=/tmp/zk1/log
     clientPort=2181
     server.1=localhost:2888:3888
     server.2=localhost:2899:3899
     server.3=localhost:2877:3877
     ```

     `zoo2.cfg`

     ```
     tickTime=2000
     initLimit=10
     syncLimit=5
     dataDir=/tmp/zk2/data
     dataLogDir=/tmp/zk2/log
     clientPort=2182
     server.1=localhost:2888:3888
     server.2=localhost:2899:3899
     server.3=localhost:2877:3877
     ```

     `zoo3.cfg`

     ```
     tickTime=2000
     initLimit=10
     syncLimit=5
     dataDir=/tmp/zk3/data
     dataLogDir=/tmp/zk3/log
     clientPort=2183
     server.1=localhost:2888:3888
     server.2=localhost:2899:3899
     server.3=localhost:2877:3877
     ```

     配置文件中 `dataDir`，`dataLogDir`，`clientPort` 这三个配置是有区别的。

     分别在 3 个节点对应的 dataDir 中建立 myid 文件，里面输入服务器标识号

     ```
     echo 1 > /tmp/zk1/data/myid
     echo 2 > /tmp/zk2/data/myid
     echo 3 > /tmp/zk3/data/myid
     ```

     启动三个节点

     ```
     bin/zkServer.sh start conf/zoo1.cfg
     bin/zkServer.sh start conf/zoo2.cfg
     bin/zkServer.sh start conf/zoo3.cfg
     ```

     查看三个节点，可以看到1、3号节点是 follower 节点，2号节点是 leader 节点

     ```
     ➜  zookeeper bin/zkServer.sh status conf/zoo3.cfg
     ZooKeeper JMX enabled by default
     Using config: conf/zoo3.cfg
     Mode: follower
     ➜  zookeeper bin/zkServer.sh status conf/zoo2.cfg
     ZooKeeper JMX enabled by default
     Using config: conf/zoo2.cfg
     Mode: leader
     ➜  zookeeper bin/zkServer.sh status conf/zoo1.cfg
     ZooKeeper JMX enabled by default
     Using config: conf/zoo1.cfg
     Mode: follower
     ```

   * 客户端代码模拟：使用到 《ZKClient》

     * 配置文件Config

       ```java
       package com.cwh.zk.util;
        
       import java.io.Serializable;
        
       public class Config implements Serializable{
       	private static final long serialVersionUID = 1L;
       	private String userNm;
       	private String userPw;
       	
       	public Config() {
       	}
       	public Config(String userNm, String userPw) {
       		this.userNm = userNm;
       		this.userPw = userPw;
       	}
       	public String getUserNm() {
       		return userNm;
       	}
       	public void setUserNm(String userNm) {
       		this.userNm = userNm;
       	}
       	public String getUserPw() {
       		return userPw;
       	}
       	public void setUserPw(String userPw) {
       		this.userPw = userPw;
       	}
       	@Override
       	public String toString() {
       		return "Config [userNm=" + userNm + ", userPw=" + userPw + "]";
           }	
       }
       ```

     * 配置管理中心ZkConfigMag

       ```java
       package com.cwh.zk.util;
        
       import org.I0Itec.zkclient.ZkClient;
        
       public class ZkConfigMag {
        
       	private Config config;
       	/**
       	 * 从数据库加载配置(模拟)
       	 */
       	public Config downLoadConfigFromDB(){
       		//getDB
       		config = new Config("nm", "pw");
       		return config;
       	}
       	
       	/**
       	 * 配置文件上传到数据库(模拟)
       	 */
       	public void upLoadConfigToDB(String nm, String pw){
       		if(config==null)config = new Config();
       		config.setUserNm(nm);
       		config.setUserPw(pw);
       		//updateDB
       	}
       	
       	/**
       	 * 配置文件同步到zookeeper
       	 */
       	public void syncConfigToZk(){
       		ZkClient zk = new ZkClient("10.222.76.148:2181", "10.222.76.148:2182", "10.222.76.148:2183");
       		if(!zk.exists("/zkConfig")){
       			zk.createPersistent("/zkConfig",true);
       		}
       		zk.writeData("/zkConfig", config);
       		zk.close();
       	}
       }
       ```

     * **应用监听实现ZkGetConfigClient**

       ```java
       package com.cwh.zk.util;
        
       import org.I0Itec.zkclient.IZkDataListener;
       import org.I0Itec.zkclient.ZkClient;
        
       public class ZkGetConfigClient {
       	private Config config;
        
       	public Config getConfig() {
       		ZkClient zk = new ZkClient("10.222.76.148:2181", "10.222.76.148:2182", "10.222.76.148:2183");
       		config = (Config)zk.readData("/zkConfig");
       		System.out.println("加载到配置："+config.toString());
       		
       		//监听配置文件修改
       		zk.subscribeDataChanges("/zkConfig", new IZkDataListener(){
       			@Override
       			public void handleDataChange(String arg0, Object arg1)
       					throws Exception {
       				config = (Config) arg1;
       				System.out.println("监听到配置文件被修改："+config.toString());
       			}
        
       			@Override
       			public void handleDataDeleted(String arg0) throws Exception {
       				config = null;
       				System.out.println("监听到配置文件被删除");
       			}
       			
       		});
       		return config;
       	}
           
       	public static void main(String[] args) {
       		ZkGetConfigClient client = new ZkGetConfigClient();
       		client.getConfig();
       		System.out.println(client.config.toString());
       		for(int i = 0;i<10;i++){
       			System.out.println(client.config.toString());
       			try {
       				Thread.sleep(1000);
       			} catch (InterruptedException e) {
       				// TODO Auto-generated catch block
       				e.printStackTrace();
       			}
       		}
       		
       	}
       }
       ```

     * 测试，启动配置管理中心

       ```java
       package com.cwh.zkConfig.test;
        
       import com.cwh.zk.util.Config;
       import com.cwh.zk.util.ZkConfigMag;
        
       public class ZkConfigTest {
        
       	public static void main(String[] args) {
       		ZkConfigMag mag = new ZkConfigMag();
       		Config config = mag.downLoadConfigFromDB();
       		System.out.println("....加载数据库配置...."+config.toString());
       		mag.syncConfigToZk();
       		System.out.println("....同步配置文件到zookeeper....");
       		
       		//歇会，这样看比较清晰
       		try {
       			Thread.sleep(10000);
       		} catch (InterruptedException e) {
       			// TODO Auto-generated catch block
       			e.printStackTrace();
       		}
       		
       		mag.upLoadConfigToDB("cwhcc", "passwordcc");
       		System.out.println("....修改配置文件...."+config.toString());
       		mag.syncConfigToZk();
       		System.out.println("....同步配置文件到zookeeper....");	
       	}
       }
       ```

     * 测试结果：

       配置管理中心打印：

       ![](.\image\Zookeeper\配置管理测试结果.png)

     * 应用监听：

       ![](.\image\Zookeeper\配置管理测试监听结果.png)

### HA 高可用性

hadoop、hdfs、yarn 等很多大数据系统，都选择基于 zookeeper 来开发 HA 高可用机制，就是一个**重要进程一般会做主备**两个，主进程挂了立马通过 zookeeper 感知到切换到备用进程。

![](.\image\Zookeeper\zookeeper-active-standby.png)

**单机搭建**：

1. 准备 Java 运行环境**

   安装 Java 1.6 或更高版本的 JDK，并配置好 Java 相关的环境变量 $JAVA_HOME 。

2. 下载 ZooKeeper 安装包

   下载地址：<http://zookeeper.apache.org/releases.html>。选择最新的 stable 版本并解压到指定目录，我们用 $ZK_HOME 表示该目录。

3. 配置 zoo.cfg

   首次使用 ZooKeeper，需要将 $ZK_HOME 下的 zoo_sample.cfg 文件重命名为 zoo.cfg，并进行以下配置

   ```
   tickTime=2000    ##Zookeeper最小时间单元，单位毫秒(ms)，默认值为3000
   dataDir=/var/lib/zookeeper    ##Zookeeper服务器存储快照文件的目录，必须配置
   dataLogDir=/var/lib/log     ##Zookeeper服务器存储事务日志的目录，默认为dataDir
   clientPort=2181    ##服务器对外服务端口，一般设置为2181
   initLimit=5    ##Leader服务器等待Follower启动并完成数据同步的时间，默认值10，表示tickTime的10倍
   syncLimit=2    ##Leader服务器和Follower之间进行心跳检测的最大延时时间，默认值5，表示tickTime的5倍
   ```

4. 启动服务

   使用 $ZK_HOME/bin 目录下的 zkServer.sh 脚本进行服务的启动。

**集群搭建**

只需修改多个 Zookeeper 的 zoo.cfg 配置文件，在其中加入 ` server.id=host:port1:port2 ` 配置

例如：

```
tickTime=2000
dataDir=/var/lib/zookeeper
dataLogDir=/var/lib/log
clientPort=2181
initLimit=5
syncLimit=2
server.1=IP1:2888:3888 ## 在这里指定本机和其他的主机上的IP和端口号
server.2=IP2:2888:3888
server.3=IP3:2888:3888
```

