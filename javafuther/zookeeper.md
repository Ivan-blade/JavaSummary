### zookeeper

#### 简介

+ 基本概念
  + Zookeeper是一个开源的分布式协调服务，其设计目标是将那些复杂的且容易出错的分布式一致性服务 封装起来，构成一个高效可靠的原语集，并以一些简单的接口提供给用户使用。
  + zookeeper是一个典型 的分布式数据一致性的解决方案，分布式应用程序可以基于它实现诸如数据订阅/发布、负载均衡、命名 服务、集群管理、分布式锁和分布式队列等功能
+ 构成
  + 集群角色
    + 通常在分布式系统中，构成一个集群的每一台机器都有自己的⻆色，最典型的集群就是Master/Slave模 式(主备模式)，此情况下把所有能够处理写操作的机器称为Master机器，把所有通过异步复制方式获 取最新数据，并提供读服务的机器为Slave机器。
    + 而在Zookeeper中，这些概念被颠覆了。它没有沿用传递的Master/Slave概念，而是引入了Leader、 Follower、Observer三种⻆色。Zookeeper集群中的所有机器通过Leader选举来选定一台被称为 Leader的机器，Leader服务器为客户端提供读和写服务，除Leader外，其他机器包括Follower和 Observer,Follower和Observer都能提供读服务，唯一的区别在于**Observer**不参与**Leader**选举过程， 不参与写操作的过半写成功策略，因此Observer可以在不影响写性能的情况下提升集群的性能。
  + 会话
    + Session指客户端会话，一个客户端连接是指客户端和服务端之间的一个**TCP**⻓连接，Zookeeper对外的 服务端口默认为2181，客户端启动的时候，首先会与服务器建立一个TCP连接，从第一次连接建立开 始，客户端会话的生命周期也开始了，通过这个连接，客户端能够心跳检测与服务器保持有效的会话， 也能够向Zookeeper服务器发送请求并接受响应，同时还能够通过该连接接受来自服务器的Watch事件 通知。
  + 数据节点
    + 在谈到分布式的时候，我们通常说的“节点”是指组成集群的每一台机器。然而，在ZooKeeper中，“节 点”分为两类，第一类同样是指构成集群的机器，我们称之为机器节点;第二类则是指数据模型中的数据 单元，我们称之为数据节点——ZNode。ZooKeeper将所有数据存储在内存中，数据模型是一棵树 (ZNode Tree)，由斜杠(/)进行分割的路径，就是一个Znode，例如/app/path1。每个ZNode上都 会保存自己的数据内容，同时还会保存一系列属性信息。
  + 版本
    + 刚刚我们提到，Zookeeper的每个Znode上都会存储数据，对于每个ZNode，Zookeeper都会为其维护 一个叫作Stat的数据结构，Stat记录了这个ZNode的三个数据版本，分别是version(当前ZNode的版 本)、cversion(当前ZNode子节点的版本)、aversion(当前ZNode的ACL版本)。
  + 事件监听器
    + Wathcer(事件监听器)，是Zookeeper中一个很重要的特性，Zookeeper允许用户在指定节点上注册 一些Watcher，并且在一些特定事件触发的时候，Zookeeper服务端会将事件通知到感兴趣的客户端， 该机制是Zookeeper实现分布式协调服务的重要特性
  + acl
    + Zookeeper采用ACL(Access Control Lists)策略来进行权限控制，其定义了如下五种权限:
      + CREATE:创建子节点的权限。
      + READ:获取节点数据和子节点列表的权限。
      + WRITE:更新节点数据的权限。
      + DELETE:删除子节点的权限。
      + ADMIN:设置节点ACL的权限。
      + CREATE和DELETE这两种权限都是针对子节点的权限控制

#### 	环境搭建

+ 单机模式

  + 下载压缩包 http://zookeeper.apache.org/releases.html，获取zookeeper-3.4.14.tar.gz

  + 上传到linux

  + 解压

    ```shell
    tar -zxvf zookeeper-3.4.14.tar.gz
    ```

  + 进入 **zookeeper-3.4.14** 目录，创建 **data** 文件夹

    ```shell
    cd zookeeper-3.4.14
    mkdir data
    ```

  + 修改配置文件名称

    ```shell
    cd conf
    mv zoo_sample.cfg zoo.cfg
    ```

  + 修改**zoo.cfg**中的**data**属性

    ```shell
    vim zoo.cfg
    dataDir=/opt/lagou/servers/zookeeper-3.4.14/data
    ```

  + **zookeeper**服务启动,关闭，状态查询命令

    ```shell
    ./zkServer.sh start
    ./zkServer.sh stop
    ./zkServer.sh status
    ```

+ 伪集群模式(见pdf)

  + 在一台机器上启动多个zookeeper，需要保证端口，输出日志路径等等信息有所区别，生产环境没什么用

+ 集群模式

  + 在每个zookeeper的 data 目录下创建一个 myid 文件，内容分别是1、2、3 。这个文件就是记录 每个服务器的ID

    ```shell
    vim myid
    1
    ```

  + 在每一个zookeeper 的 zoo.cfg配置客户端访问端口(clientPort)和集群服务器IP列表。

    ```shell
    server.1=192.168.8.101:2881:3881
    server.2=192.168.8.102:2881:3881
    server.3=192.168.8.103:2881:3881
    #server.服务器ID=服务器IP地址:服务器之间通信端口:服务器之间投票选举端口
    ```

  + 分别启动三个服务即可，通过status查看leader和follower节点

#### 基本使用

+ zookeeper系统模型

  + **ZooKeeper**数据模型**Znode**（缺图）

    + 在ZooKeeper中，数据信息被保存在一个个数据节点上，这些节点被称为znode。ZNode 是 Zookeeper 中最小数据单位，在 ZNode 下面又可以再挂 ZNode，这样一层层下去就形成了一个层次化 命名空间 ZNode 树，我们称为 ZNode Tree，它采用了类似文件系统的层级树状结构进行管理。⻅下图 示例:

    + 在 Zookeeper 中，每一个数据节点都是一个 ZNode，上图根目录下有两个节点，分别是:app1 和 app2，其中 app1 下面又有三个子节点,所有ZNode按层次化进行组织，形成这么一颗树，ZNode的节 点路径标识方式和Unix文件系统路径非常相似，都是由一系列使用斜杠(/)进行分割的路径表示，开 发人员可以向这个节点写入数据，也可以在这个节点下面创建子节点。

    + **ZNode** 的类型（Zookeeper 节点类型可以分为三大类:）

      + 持久性节点(Persistent)
      + 临时性节点(Ephemeral)
      + 顺序性节点(Sequential)
      + 在开发中在创建节点的时候通过组合可以生成以下四种节点类型:持久节点、持久顺序节点、临时节点、临时顺序节点。不同类型的节点则会有不同的生命周期
        + 持久节点:是Zookeeper中最常⻅的一种节点类型，所谓持久节点，就是指节点被创建后会一直存在服 务器，直到删除操作主动清除
        + 持久顺序节点:就是有顺序的持久节点，节点特性和持久节点是一样的，只是额外特性表现在顺序上。 顺序特性实质是在创建节点的时候，会在节点名后面加上一个数字后缀，来表示其顺序。
        + 临时节点:就是会被自动清理掉的节点，它的生命周期和客户端会话绑在一起，客户端会话结束，节点 会被删除掉。与持久性节点不同的是，临时节点不能创建子节点。
        + 临时顺序节点:就是有顺序的临时节点，和持久顺序节点相同，在其创建的时候会在名字后面加上数字 后缀。
    
    + 事务**ID**
    
      + 首先，先了解，事务是对物理和抽象的应用状态上的操作集合。往往在现在的概念中，􏰁义上的事务通 常指的是数据库事务，一般包含了一系列对数据库有序的读写操作，这些数据库事务具有所谓的ACID特 性，即原子性(Atomic)、一致性(Consistency)、隔离性(Isolation)和持久性(Durability)。
      + 而在ZooKeeper中，事务是指能够改变ZooKeeper服务器状态的操作，我们也称之为事务操作或更新操 作，一般包括数据节点创建与删除、数据节点内容更新等操作。对于每一个事务请求，ZooKeeper都会 为其分配一个全局唯一的事务ID，用 **ZXID** 来表示，通常是一个 64 位的数字。每一个 ZXID 对应一次更 新操作，从这些ZXID中可以间接地识别出ZooKeeper处理这些更新操作请求的全局顺序
    
    + **ZNode** 的状态信息
    
      + 整个 ZNode 节点内容包括两部分:节点数据内容和节点状态信息。
    
        ```shell
         cZxid 就是 Create ZXID，表示节点被创建时的事务ID。
        ctime 就是 Create Time，表示节点创建时间。
        mZxid 就是 Modified ZXID，表示节点最后一次被修改时的事务ID。
        mtime 就是 Modified Time，表示节点最后一次被修改的时间。
        pZxid 表示该节点的子节点列表最后一次被修改时的事务 ID。只有子节点列表变更才会更新 pZxid，
        子节点内容变更不会更新。
        cversion 表示子节点的版本号。
        dataVersion 表示内容版本号。
        aclVersion 标识acl版本
        ephemeralOwner 表示创建该临时节点时的会话 sessionID，如果是持久性节点那么值为 0 dataLength 表示数据⻓度。
        numChildren 表示直系子节点数。
        ```
    
  + **Watcher--**数据变更通知
  
    + Zookeeper使用Watcher机制实现分布式数据的发布/订阅功能
      + 一个典型的发布/订阅模型系统定义了一种 一对多的订阅关系，能够让多个订阅者同时监听某一个主题 对象，当这个主题对象自身状态变化时，会通知所有订阅者，使它们能够做出相应的处理。
      + 在 ZooKeeper 中，引入了 Watcher 机制来实现这种分布式的通知功能。ZooKeeper 允许客户端向服务 端注册一个 Watcher 监听，当服务端的一些指定事件触发了这个 Watcher，那么就会向指定客户端发 送一个事件通知来实现分布式的通知功能。
      + 整个Watcher注册与通知过程如图所示。(暂时无图)
      + Zookeeper的Watcher机制主要包括客户端线程、客户端**WatcherManager**、**Zookeeper**服务器三部 分。
      + 具体工作流程为:客户端在向Zookeeper服务器注册的同时，会将Watcher对象存储在客户端的 WatcherManager当中。当Zookeeper服务器触发Watcher事件后，会向客户端发送通知，客户端线程 从WatcherManager中取出对应的Watcher对象来执行回调逻辑。
  
  + **ACL--**保障数据的安全
  
    + Zookeeper作为一个分布式协调框架，其内部存储了分布式系统运行时状态的元数据，这些元数据会直 接影响基于Zookeeper进行构造的分布式系统的运行状态，因此，如何保障系统中数据的安全，从而避 免因误操作所带来的数据随意变更而导致的数据库异常十分重要，在Zookeeper中，提供了一套完善的 ACL(Access Control List)权限控制机制来保障数据的安全。
  
    + 我们可以从三个方面来理解ACL机制:权限模式(**Scheme**)、授权对象(**ID**)、权限 (**Permission**)，通常使用"**scheme: id : permission**"来标识一个有效的ACL信息。
  
      + 权限模式:**Scheme**
  
        + 权限模式用来确定权限验证过程中使用的检验策略，有如下四种模式:
  
          + **IP**
  
            + IP模式就是通过IP地址粒度来进行权限控制，如"ip:192.168.0.110"表示权限控制针对该IP地址， 同时IP模式可以支持按照网段方式进行配置，如"ip:192.168.0.1/24"表示针对192.168.0.*这个网段 进行权限控制
  
          + **Digest**
  
            + Digest是最常用的权限控制模式，要更符合我们对权限控制的认识，其使用"username:password"形式的权限标识来进行权限配置，便于区分不同应用来进行权限控制。 当我们通过“username:password”形式配置了权限标识后，Zookeeper会先后对其进行SHA-1加密和BASE64编码。
  
          + **World**
  
            + World是一种最开放的权限控制模式，这种权限控制方式几乎没有任何作用，数据节点的访问权限 对所有用户开放，即所有用户都可以在不进行任何权限校验的情况下操作ZooKeeper上的数据。 另外，World模式也可以看作是一种特殊的Digest模式，它只有一个权限标识，即“world: anyone”。
  
          + **Super**
  
            + Super模式，顾名思义就是超级用户的意思，也是一种特殊的Digest模式。在Super模式下，超级
  
              用户可以对任意ZooKeeper上的数据节点进行任何操作。
  
    + 授权对象:**ID**
  
      + 授权对象指的是权限赋予的用户或一个指定实体，例如 IP 地址或是机器等。在不同的权限模式下，授 权对象是不同的，表中列出了各个权限模式和授权对象之间的对应关系。（缺图）
  
    + 权限
  
      + 权限就是指那些通过权限检查后可以被允许执行的操作。在ZooKeeper中，所有对数据的操作权限分为 以下五大类:
        + · CREATE(C):数据节点的创建权限，允许授权对象在该数据节点下创建子节点。 
        + · DELETE(D): 子节点的删除权限，允许授权对象删除该数据节点的子节点。 
        + · READ(R):数据节点的读取权限，允 许授权对象访问该数据节点并读取其数据内容或子节点列表等。
        +  · WRITE(W):数据节点的更新权 限，允许授权对象对该数据节点进行更新操作。 
        + · ADMIN(A):数据节点的管理权限，允许授权对象 对该数据节点进行 ACL 相关的设置操作。

+ zookeeper命令行操作

  + 连接

    ```shell
    ./zkcli.sh 连接本地的zookeeper服务器 
    ./zkCli.sh -server ip:port 连接指定的服务器
    help 显示可用命令
    ```

  + 创建节点

    ```shell
    create [-s][-e] path data acl 其中，-s或-e分别指定节点特性，顺序或临时节点，若不指定，则创建持久节点;acl用来进行权限控制。
    create -s /zk-test 123 命令创建zk-test顺序节点,数据内容为123
    create -e /zk-temp 123 命令创建zk-temp临时节点,临时节点在客户端会话结束后，就会自动删除
    create /zk-permanent 123 命令创建zk-permanent永久节点
    quit                   命令退出客户端
    ```

  + 读取节点

    + 与读取相关的命令有ls 命令和get 命令
      + ls命令可以列出Zookeeper指定节点下的所有子节点，但只能查看指定节点下的第一级的所有子节点;
      + get命令可以获取Zookeeper指定节点的数据内容和属性信息。

    ```shell
    ls path
    get path
    # 其中，path表示的是指定数据节点的节点路径
    ```

  + 更新

    + 使用set命令，可以更新指定节点的数据内容，成功更新后dataversion字段会自增

      + 其中，data就是要更新的新内容，version表示数据版本，在zookeeper中，节点的数据是有版本概 念的，这个参数用于指定本次更新操作是基于Znode的哪一个数据版本进行的，如将/zk-permanent节 点的数据更新为456，可以使用如下命令:**set /zk-permanent 456**

      ```shell
      set path data [version]
      ```

  + 删除节点

    + 使用delete命令可以删除Zookeeper上的指定节点，用法如下

      ```shell
       delete path [version]
       delete /zk-permanent # 命令即可删除/zk-permanent节点
       # 值得注意的是，若删除节点存在子节点，那么无法删除 该节点，必须先删除子节点，再删除父节点
      ```

+ Zookeeeper API使用

  + 普通api

  + zkcliapi

    + 引入依赖

    ```xml
    <dependency>
       <groupId>com.101tec</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.2</version>
    </dependency>
    ```

    + 使用

      ```java
      // 创建会话
      ZkClient zkClient = new ZkClient("192.168.8.101:2181");
      String nodePath = "/node/node1";
      
      
      // 判断节点是否存在
      if (!zkClient.exists(nodePath)) {
          // 创建节点，true表示是否递归创建节点
          zkClient.createPersistent(nodePath, true);
          System.out.println(nodePath+"节点注册完成");
        	// 删除节点
        	zkClient.delete(nodePath);
      }
      
      
      // 获取子节点
      String path = "/127.0.0.1";
      List<String> children = zkClient.getChildren(path);
      // 接听节点下是否存在子节点变化
      zkClient.subscribeChildChanges(path, new IZkChildListener() {
          /**
           * @param s 监听节点路径
           * @param list 变化后节点列表
           * @throws Exception
           */
          @Override
          public void handleChildChange(String s, List<String> list) throws Exception {
              System.out.println(list);
          }
      });
      
      // 节点数据相关
      String path = "/node/node1";
      //判断节点是否存在
      boolean exists = zkClient.exists(path);
      if (!exists){
          zkClient.createEphemeral(path, "123");
      }
      //注册监听
      zkClient.subscribeDataChanges(path, new IZkDataListener() {
      public void handleDataChange(String path, Object data) throws Exception { System.out.println(path+"该节点内容被更新，更新后的内容"+data);}
      public void handleDataDeleted(String s) throws Exception { System.out.println(s+" 该节点被删除");} 
      });
      
      //获取节点内容
      Object o = zkClient.readData(path); 
      System.out.println(o);
      //更新 
      zkClient.writeData(path,"4567");
      //删除 
      zkClient.delete(path);
      ```

      

  + curator api

#### 使用场景

#### 深入进阶

#### 源码分析