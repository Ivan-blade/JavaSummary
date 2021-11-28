### Kafka

#### 简介

+ kafka是什么
  + <a href="http://kafka.apache.org/">官网</a>
  + Kafka是最初由Linkedin公司开发，是一个分布式、分区的、多副本的、多订阅者，基于zookeeper协调的分布式日志系统（也可以当做 MQ系统），常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。
  + 主要设计目标
    + 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间的访问性能。
      + 算法复杂度：时间复杂度和空间复杂度
      + 以时间复杂度为O(1)的方式：常数时间运行和数据量的增长无关，假如操作一个链表，那么无论链表的大还是小，操作时间是一 样的
    + 高吞吐率。即使在**非常廉价的商用机器**上也能做到单机支持每秒100K条消息的传输。
      + 支持**普通服务器**每秒**百万级**写入请求
      + Memory mapped Files
    + 支持Kafka Server间的消息分区，及分布式消费，同时保证每个partition内的消息顺序传输。
    + 同时支持离线数据处理和实时数据处理。
    + Scale out:支持在线水平扩展
  + 消息传递模式
    + kafka有两种消息传递模式
      + 点对点
      + 发布-订阅
    + 对于消息中间件，消息分推拉两种模式。Kafka只有消息的拉取，没有推送，可以通过轮询实现消息的推送。
      + Kafka在一个或多个可以跨越多个数据中心的服务器上作为集群运行。
      + Kafka集群中按照主题分类管理，一个主题可以有多个分区，一个分区可以有多个副本分区。 
      + 每个记录由一个键，一个值和一个时间戳组成。
    + Kafka具有四个核心API:
      + Producer API:允许应用程序将记录流发布到一个或多个Kafka主题。
      + Consumer API:允许应用程序订阅一个或多个主题并处理为其生成的记录流。
      + Streams API:允许应用程序充当流处理器，使用一个或多个主题的输入流，并生成一个或多个输出主题的输出流，从而有效地将输入流转换为输出流。
      + Connector API:允许构建和运行将Kafka主题连接到现有应用程序或数据系统的可重用生产者或使用者。例如，关系数据库的连接器可能会捕获对表的所有更改。
  + 特点
    + 解耦。Kafka具备消息系统的优点，只要生产者和消费者数据两端遵循接口约束，就可以自行扩展或修改数据处理的业务过程。 
    + 高吞吐量、低延迟。即使在非常廉价的机器上，Kafka也能做到每秒处理几十万条消息，而它的延迟最低只有几毫秒。 
    + 持久性。Kafka可以将消息直接持久化在普通磁盘上并持久化到分区，从而防止数据丢失，且磁盘读写性能优异。
      + 零拷贝
      + 顺序读，顺序写
      + 利用Linux的页缓存
    + 分布式系统，扩展性：Kafka集群支持热扩展，Kaka集群启动运行后，用户可以直接向集群添加。
    + 可靠性：kafka是分布式，分区，复制和容错的 
    + 容错性：Kafka会将数据备份到多台服务器节点中，即使Kafka集群中的某一台加新的Kafka服务节点宕机，也不会影响整个系统的功 能。
    + 客户端状态维护：消息被处理的状态是consumer端维护，而不是server端维护，失败了能够自动平衡。 
    + 支持多种客户端语言。Kafka支持Java、.NET、PHP、Python等多种语言。
    + 支持多生产者和多消费者。
  + 主要应用场景
    + 消息处理（MQ）
      + KafKa可以代替传统的消息队列软件，使用KafKa来实现队列有如下优点
        + KafKa的append来实现消息的追加,保证消息都是有序的有先来后到的顺序,
        + 稳定性强队列在使用中最怕丢失数据，KafKa能做到理论上的写成功不丢失
        + 分布式容灾好
        + 容量大相对于内存队列,KafKa的容量受硬盘影响
        + 数据量不会影响到KafKa的速度
    + 分布式日志系统(Log)
      + 在很多时候我们需要对一些庞大的数据进行存留，日志存储这块会遇到巨大的问题，日志不能丢，日志存文件不好找，定位一条消息 成本高（遍历当天日志文件）,实时显示给用户难，这几类问题KafKa都能游刃有余
        + KafKa的集群备份机制能做到n/2的可用,当n/2以下的机器宕机时存储的日志不会丢失
        + KafKa可以对消息进行分组分片
        + KafKa非常容易做到实时日志查询
    + 流式处理
      + 流式处理就是指实时地处理一个或多个事件流。
        + 流式的处理框架(spark, storm , flink) 从主题中读取数据, 对其进行处理, 并将处理后的结果数据写入新的主题, 供用户和应用程序使用, kafka的强耐久性在流处理的上下文中也非常的有用
    + 用户活动跟踪
      + Kafka经常被用来记录Web用户或者App用户的各种活动，如浏览网⻚、搜索、点击等活动，这些 活动信息被各个服务器发布到Kafka的Topic中，然后消费者通过订阅这些Topic来做实时的监控分析，亦可保存到数据 库;
    + 运营指标
      + Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比 如报警和报告;

#### kafka架构（缺图）

+ Kafka Cluster：
  + 由多个服务器组成。每个服务器单独的名字broker（掮客）。

+ kafka broker：
  + kafka集群中单个服务称为broker，broker接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保 存。broker为消费者提供服务，对读取分区的请求做出响应，返回已经提交到磁盘上的消息。单个**broker**可以轻松处理数千个分区以及每秒百万级的消息量。

  + broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。

    + 如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的一个partition。

    + 如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的一个partition，

    剩下的M个broker不存储该topic的partition数据。

    + 如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个

    partition。在实际生产环境中，尽量避免这种情况的发生，这种情况容易导致Kafka集群数据不均衡。

  + 每个集群都有一个broker是集群控制器(自动从集群的活跃成员中选举出来)。

    + 控制器负责管理工作:
      + 将分区分配给broker 
      + 监控broker
    + 集群中一个分区属于一个**broker**，该broker称为分区首领。 
    + 一个分区可以分配给多个**broker**，此时会发生分区复制。 
    + 分区的复制提供了消息冗余，高可用。副本分区不负责处理消息的读写。只负责同步数据，在主分区不可用时切换为主分区

+ Kafka Producer：
  + 消息生产者、发布消息到 kafka 集群的终端或服务。 
  + 生产者创建消息。消费者消费消息。
    + 一个消息被发布到一个特定的主题上。
    + 生产者在默认情况下把消息均衡地分布到主题的所有分区上:
      + 默认情况下通过轮询把消息均衡地分布到主题的所有分区上。
      + 在某些情况下，生产者会把消息直接写到指定的分区。这通常是通过消息键和分区器来实现的，分区器为键生成一个散列值，并将其映射到指定的分区上。这样可以保证包含同一个键的消息会被写到同一个分区上
      + 生产者也可以使用自定义的分区器，根据不同的业务规则将消息映射到分区。

+ Kafka consumer：
  + 消息消费者、负责消费数据。 
  + 消费者订阅一个或多个主题，并按照消息生成的顺序读取它们。
  + 消费者通过检查消息的偏移量来区分已经读取过的消息。偏移量是另一种元数据，它是一个不断递增的整数值，在创建消息时，Kafka 会把它添加到消息里。在给定的分区里，每个消息的偏移量都是唯一的。消费者 把每个分区最后读取的消息偏移量保存在Zookeeper 或**Kafka** 上，如果消费者关闭或重启，它的读取状态不会丢失。
  + 消费者是消费组的一部分。群组保证每个分区只能被一个消费者使用。
  + 如果一个消费者失效，消费组里的其他消费者可以接管失效消费者的工作，再平衡，分区重新分配。

+ Kafka Topic: 
  + 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。 
  + 物理上不同Topic的消息分开存储。 
  + 主题就好比数据库的表，尤其是分库分表之后的逻辑表。

+ Kafka Partition

  + 主题可以被分为若干个分区，一个分区就是一个提交日志。
  + 消息以追加的方式写入分区，然后以先入先出的顺序读取。
  + 无法在整个主题范围内保证消息的顺序，但可以保证消息在单个分区内的顺序。
  + Kafka 通过分区来实现数据冗余和伸缩性。
  + 在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

+ Kafka Replices

  + Kafka 使用主题来组织数据，每个主题被分为若干个分区，每个分区有多个副本。那些副本被保存在broker 上， 每个broker 可以保存成百上千个属于不同主题和分区的副本。
  + 副本有以下两种类型
    + 首领副本:  每个分区都有一个首领副本。为了保证一致性，所有生产者请求和消费者请求都会经过这个副本。
    + 跟随者副本: 首领以外的副本都是跟随者副本。跟随者副本不处理来自客户端的请求，它们唯一的任务就是从首领那里复制消
      息，保持与首领一致的状态。如果首领发生崩溃，其中的一个跟随者会被提升为新首领。

  + **AR**
    + 分区中的所有副本统称为**AR**(Assigned Repllicas)：**AR=ISR+OSR**
  + **ISR**
    + 所有与leader副本保持一定程度同步的副本(包括Leader)组成**ISR**(In-Sync Replicas)，ISR集合是AR集合中 的一个子集。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步，同步期间内 follower副本相对于leader副本而言会有一定程度的滞后。前面所说的“一定程度”是指可以忍受的滞后范围，这个范围 可以通过参数进行配置。

  + **OSR**
    + 与leader副本同步滞后过多的副本(不包括leader)副本，组成**OSR**(Out-Sync Relipcas)。在正常情况下，所有 的follower副本都应该与leader副本保持一定程度的同步，即AR=ISR,OSR集合为空。

  + **HW**
    + HW是High Watermak的缩写， 俗称高水位，它表示了一个特定消息的偏移量(offset)，消费之只能拉取到这 个offset之前的消息。

  + **LEO**
    + LEO是Log End Offset的缩写，它表示了当前日志文件中下一条待写入消息的offset。

+ Kafka Offset

  + 生产者offset
    + 消息写入的时候，每一个分区都有一个offset，这个offset就是生产者的offset，同时也是这个分区的最新最大的 offset。
    + 有些时候没有指定某一个分区的offset，这个工作kafka帮我们完成。

  + 消费者offset
    + 例如：生产者写入的offset是最新最大的值是12，Consumer A进行消费时，从0开始消费，一直消费到了9，消费者的offset就记录在9，Consumer B就纪录在了11。等下一次他们再来消费时，他们可以选择接着上一次的位置消费，当然也可以选择从头消费，或者跳到最近的记录并从“现在”开始消费。

+ 注意：Kafka的元数据都是存放在zookeeper中。

#### 架构剖析（缺图）

+ 说明：kafka支持消息持久化，消费端为拉模型来拉取数据，消费状态和订阅关系有客户端负责维护，消息消费完 后，不会立即删除，会保 留历史消息。因此支持多订阅时，消息只会存储一份就可以了。
+ Topic：每条发布到kafka集群的消息都有一个类别，这个类别就叫做topic
+ Topic Partition：分区，物理上的概念，每个topic包含一个或多个partition，一个partition对应一个文件夹，这个文件夹下存储partition的 数据和索引文件，每个partition内部是有序的

#### 关系解释

+ topic&partition
  + Topic 就是数据主题，是数据记录发布的地方,可以用来区分业务系统。 
  + Kafka中的Topics总是多订阅者模式，一个topic可以拥有一个或者多个消费者来订阅它的数据。 
  + 一个topic为一类消息，每条消息必须指定一个topic。 
  + 对于每一个topic， Kafka集群都会维持一个分区日志。
  + 每个分区都是有序且顺序不可变的记录集，并且不断地追加到结构化的commit log文件。 
  + 分区中的每一个记录都会分配一个id号来表示顺序，称之为offset，offset用来唯一的标识分区中每一条记录。
  + 在每一个消费者中唯一保存的元数据是offset（偏移量）即消费在log中的位置，偏移量由消费者所控制：通常在读取记录后，消费者会以线 性的方式增加偏移量 ，但是实际上，由于这个位置由消费者控制，所以消费者可以采用任何顺序来消费记录。例如，一个消费者可以重置到一个旧的偏移量，从 而重新处理过去的数 据；也可以跳过最近的记录，从"现在"开始消费。 这些细节说明Kafka 消费者是非常廉价的—消费者的增加和减少，对集群或者其他消费者没有多大的影响。

#### 消息和批次

+ Kafka的数据单元称为消息。可以把消息看成是数据库里的一个“数据行”或一条“记录”。消息由字节数组组成。
+ 消息有键，键也是一个字节数组。当消息以一种可控的方式写入不同的分区时，会用到键。
+ 为了提高效率，消息被分批写入Kafka。批次就是一组消息，这些消息属于同一个主题和分区。

+ 把消息分成批次可以减少网络开销。批次越大，单位时间内处理的消息就越多，单个消息的传输时间就越⻓。批
次数据会被压缩，这样可以提升数据的传输和存储能力，但是需要更多的计算处理。

#### 环境搭建

+ ZooKeeper 作为给分布式系统提供协调服务的工具被 kafka 所依赖。在分布式系统中，消费者需要知道有哪些生产者是可用的，而如果 每次消费者都需要和生产者建立连接并测试是否成功连接，那效率也太低了，显然是不可取的。而通过使用 ZooKeeper 协调服务，Kafka 就能将 Producer，Consumer，Broker 等结合在一起，同时借助 ZooKeeper，Kafka 就能够将所有组件在无状态的条件下建立起生产者和 消费者的订阅关系，实现负载均衡。

+ 安装kafka实例的机器上需要安装zookeeper，在linux121-123三台机器上安装kafka集群，ip这边分别为（192.168.8.101-103）

+ 三台机器准备好jdk1.8环境，静态ip网络配置，做好host映射

+ zookeeper集群搭建并启动集群


+ kafka集群搭建


  + <a href="http://kafka.apache.org/">官网下载tgz包，建议2.11</a>

  + 上传至服务器并解压

    ```shell
    tar -xvf kafka_2.11-1.0.0.tgz -C /opt/lagou/servers/
    cd /opt/lagou/servers/
    # 重命名，看着舒服
    mv kafka_2.11-1.0.0 kafka
    ```

  + 修改kafka相关配置文件(linux121)

    ```shell
    cd /opt/lagou/servers/kafka/config/
    vi server.properties
    主要修改一下6个地方:
    1) broker.id 需要保证每一台kafka都有一个独立的broker
    2) log.dirs 数据存放的目录
    3) zookeeper.connect zookeeper的连接地址信息
    4) delete.topic.enable 是否直接删除topic
    5) host.name 主机的名称
    6) 修改: listeners=PLAINTEXT://linux121:9092
    #broker.id 标识了kafka集群中一个唯一broker。
    broker.id=0
    num.network.threads=3
    num.io.threads=8
    socket.send.buffer.bytes=102400
    socket.receive.buffer.bytes=102400
    socket.request.max.bytes=104857600
    # 存放生产者生产的数据 数据一般以topic的方式存放
    log.dirs=/opt/lagou/data/kafka
    num.partitions=1
    num.recovery.threads.per.data.dir=1
    offsets.topic.replication.factor=1
    transaction.state.log.replication.factor=1
    transaction.state.log.min.isr=1
    log.retention.hours=168
    log.segment.bytes=1073741824
    log.retention.check.interval.ms=300000
    # zk的信息
    zookeeper.connect=linux121:2181,linux122:2181,linux123:2181
    zookeeper.connection.timeout.ms=6000
    group.initial.rebalance.delay.ms=0
    delete.topic.enable=true
    host.name=linux121
    ```

  + 编写完成之后将该文件发送到linux122和linux123上，做以下修改

    ```shell
    ip为101的服务器: broker.id=0 , host.name=linux121 listeners=PLAINTEXT://linux121:9092
    ip为102的服务器: broker.id=1 , host.name=linux122 listeners=PLAINTEXT://linux122:9092
    ip为103的服务器: broker.id=2 , host.name=linux123 listeners=PLAINTEXT://linux123:9092
    ```

  + 在三台机器上创建日志输出目录

    ```shell
    mkdir -p /opt/lagou/data/kafka
    ```

  + 分别启动三台主机以启动集群

    ```shell
    cd /opt/lagou/servers/kafka/bin
    #前台启动
    ./kafka-server-start.sh /opt/lagou/servers/kafka/config/server.properties
    #后台启动
    nohup ./kafka-server-start.sh /opt/lagou/servers/kafka/config/server.properties 2>&1 &
    注意：可以启动一台broker，单机版。也可以同时启动三台broker，组成一个kafka集群版
    #kafka停止
    ./kafka-server-stop.sh
    # jps查询相关进程
    jps
    ```

  + 启动报错

    ```shell
    # There is insufficient memory for the Java Runtime Environment to continue. 
    # Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
    # kafka默认需要1G运存，内存不够在bin目录下编辑启动指令修改内存大小即可，我这边改为500M
    vim kafka-server-start.sh 
    export KAFKA_HEAP_OPTS="-Xmx500M -Xms500M"
    ```

    

  + 在docker中部署kafka集群


    暂时省略

#### kafka基本操作

+ linux中进行shell操作

  + kafka-topics.sh 用于管理主题。

    ```shell
    # 列出现有的主题
    kafka-topics.sh --list --zookeeper localhost:2181/myKafka
    # 创建主题，该主题包含一个分区，该分区为Leader分区，它没有Follower分区副本。
    kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_1 --partitions 1 --replication-factor 1
    # 查看分区信息
    kafka-topics.sh --zookeeper localhost:2181/myKafka --list
    # 查看指定主题的详细信息
    kafka-topics.sh --zookeeper localhost:2181/myKafka --describe --topic topic_1
    # 删除指定主题
    kafka-topics.sh --zookeeper localhost:2181/myKafka --delete --topic topic_1
    ```
  
  + kafka-console-producer.sh用于生产消息:

    ```shel
    kafka-console-producer.sh--topictopic_1--broker-listlocalhost:9020
    ```
  
  + kafka-console-consumer.sh用于消费消息:

    ```shell
    # 开启消费者
    kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic_1
    # 开启消费者方式二，从头消费，不按照偏移量消费
    kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic_1 --from-beginning
    ```
  
+ docker环境

  + 创建topic

    ```shell
    #登录到Kafka容器
    docker exec -it 9218e985e160 /bin/bash
    #切换到bin目录
    cd opt/kafka/bin/
    #执行创建
    kafka-topics.sh --create --zookeeper zoo1:2181 --replication-factor 3 --partitions 1 --topic test
    ```

    + --create：新建命令 
    + --zookeeper：Zookeeper节点，一个或多个
    + --replication-factor：指定副本，每个分区有三个副本。 
    + --partitions：1
    + --topic: 主题名

  + 查看主题

    ```shell
    kafka-topics.sh --list --zookeeper zoo1:2181,zoo2:2181,zoo3:2181
    ```

    + __consumer_offsets 这个topic是由kafka自动创建的，默认50个分区，存储消费位移信息（offset），老版本架构中是存储在Zookeeper 中。

  + 生产者生产数据

    + 模拟生产者来生产数据： Kafka自带一个命令行客户端，它从文件或标准输入中获取输入，并将其作为message（消息）发送到Kafka集群。 默认情况下，每行将作为单独的message发送。 运行 producer，然后在控制台输入一些消息以发送到服务器

    + 在使用的时候会用到bootstrap与broker.list其实是实现一个功能，broker.list是旧版本命令。

      ```shell
      kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9093,kafka3:9094 --topic testThis is a messageThis is another message
      ```

    + 消费者消费数据

      ```shell
      kafka-console-consumer.sh --bootstrap-server kafka1:9092, kafka2:9093, kafka3:9094 --topic test --frombeginning
      ```

      + 确保消费者消费的消息是顺序的，需要把消息存放在同一个topic的同一个分区
      + 一个主题多个分区，分区内消息有序。

    + 查看topic相关详细信息

      + 运行describe查看topic的相关详细信息

        ```shell
        #查看topic主题详情，Zookeeper节点写一个和全部写，效果一致
        kafka-topics.sh --describe --zookeeper zoo1:2181,zoo2:2181,zoo3:2181 --topic test
        ```
    
    + 增加topic分区数
    
      + 执行以下命令
    
        ```shell
        kafka-topics.sh --zookeeper zkhost:port --alter --topic test --partitions 8
        ```
    
    + 增加集群配置
    
      + flush.messages：
    
        + 此项配置指定时间间隔：强制进行fsync日志，默认值为None。 例如，如果这个选项设置为1，那么每条消息之后都需要进行fsync，如果设置为5，则每5条消息就需要进行一次fsync。
    
        + 一般来说，建议你不要设置这个值。此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡。 如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞)。 如果此值过小,将会导致"fsync"的次数较多，这也意味着整体的client请求有一定的延迟，物理server故障，将会导致没有fsync的消息丢失。
    
          ```shell
          kafka-topics.sh --zookeeper zoo1:2181 --alter --topic test --config flush.messages=1
          ```
    
    + 删除集群配置
    
      ```shell
      kafka-topics.sh --zookeeper zoo1:2181 --alter --topic test --delete-config flush.messages
      ```
    
    + 删除topic
    
      + 目前删除topic在默认情况只是打上一个删除的标记，在重新启动kafka后才删除。
    
      + 如果需要立即删除，则需要在 server.properties中配置： delete.topic.enable=true（集群中的所有实例节点），一个主题会在不同的kafka节点中分配分组信息和副本信息 然后执行以下命令进行删除topic
    
        ```shell
        kafka-topics.sh --zookeeper zoo1:2181 --delete --topic test
        ```
    
    #### java API操作kafka
    
    + 引入依赖
    
      ```xml
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.5.2</version>
      </parent>
      
      
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <!-- KAFKA -->
          <dependency>
              <groupId>org.springframework.kafka</groupId>
              <artifactId>spring-kafka</artifactId>
          </dependency>
      </dependencies>
      ```
    
    + 启动类加上注解
    
      ```java
      @SpringBootApplication
      @EnableKafka
      public class Application {
          public static void main(String[] args) {
              SpringApplication.run(Application.class,args);
          }
      }
      ```
    
    + 编写相关配置
    
      ```yml
      spring:
        kafka:    # super kafka servers
          bootstrap-servers: 192.168.8.101:9092,192.168.8.102:9092,192.168.8.103:9092
          zookeeper-connect: 192.168.8.101:2182,192.168.8.102:2182,192.168.8.103:2182
          producer:
            acks: 1
            retries: 1
            batch.size: 16384
            buffer-memory: 104857600
            max-request-size: 104857600
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          consumer:
            # 配置是否进行消费
            autoStartup: true
            concurrency: 3
            # 一个消息只能被一个groupId消费一次，如果多个业务需要消费同一个消息，可以使用不同的groupId
            groupId: xxxxxx
            # 配置监听topic
            topics: test
            autoOffsetReset: latest
            enableAutoCommit: false
            maxPollRecords: 100
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            allow.auto.create.topics: false
      ```

  

    + 编写配置读取类

      + KafkaOtherProperties

        ```java
        @Configuration
        public class KafkaOtherProperties {
            
            @Value("${spring.kafka.consumer.concurrency:3}")
            private int concurrency;
        
            @Value("${spring.kafka.consumer.bootstrap-servers:3}")
            private List<String> bootstrapServers;
        
            public int getConcurrency() {
                return concurrency;
            }
        
            public void setConcurrency(int concurrency) {
                this.concurrency = concurrency;
            }
        
            public List<String> getBootstrapServers() {
                return bootstrapServers;
            }
        
            public void setBootstrapServers(List<String> bootstrapServers) {
                this.bootstrapServers = bootstrapServers;
            }
        }
        ```

      + KafkaListenerConfig

        ```java
        @Configuration
        @EnableKafka
        public class KafkaListenerConfig {
        
          @Autowired 
          private KafkaProperties kafkaProperties;
          
          @Autowired 
          private KafkaOtherProperties otherProperties;
        
          @Bean
          KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>>
              kafkaListenerContainerFactory() {
            ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
            factory.setConsumerFactory(consumerFactory());
            factory.setConcurrency(otherProperties.getConcurrency());
            factory.getContainerProperties().setPollTimeout(30000L);
            factory.setBatchListener(true);
            return factory;
          }
        
          @Bean
          public ConsumerFactory<String, String> consumerFactory() {
            return new DefaultKafkaConsumerFactory<>(consumerConfigs());
          }
        
          @Bean
          public Map<String, Object> consumerConfigs() {
            Map<String, Object> props = new HashMap<>();
            props.put(ConsumerConfig.GROUP_ID_CONFIG, kafkaProperties.getConsumer().getGroupId());
            props.put(
                ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,
                kafkaProperties.getConsumer().getEnableAutoCommit());
            props.put(
                ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,
                kafkaProperties.getConsumer().getAutoOffsetReset());
            props.put(
                ConsumerConfig.MAX_POLL_RECORDS_CONFIG, kafkaProperties.getConsumer().getMaxPollRecords());
            props.put(
                ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                kafkaProperties.getConsumer().getKeyDeserializer());
            props.put(
                ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                kafkaProperties.getConsumer().getValueDeserializer());
            props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, otherProperties.getBootstrapServers());
            props.putAll(kafkaProperties.getProperties());
            return props;
          }
        }
        ```

  + 编写消费者监听类

    ```java
    @Component
    public class KafkaListener {
    
        // id可以设置kafka消费者线程名，不设置也没事
    	@KafkaListener(
              id = "kafkaListener",
          topics = "#{'${spring.kafka.consumer.topics}'.split(',')}",
          autoStartup = "${spring.kafka.consumer.autoStartup}",
          containerFactory = "kafkaListenerContainerFactory")
      public void listen(List<ConsumerRecord<String, String>> records, Consumer<?, ?> consumer) {
          for (ConsumerRecord<String, String> record : records) {
            String value = record.value();
            System.out.println(value);
          }
          consumer.commitSync();
      }
    }
    ```

  + 编写kafka生产者

    ```java
    @Component
    public class KafkaSend {
    
      @Autowired
      private KafkaTemplate<String, String> kafkaTemplate;
    
      public void send(String topic, List<String> messages) {
        for (String message : messages) {
          send(topic, message);
        }
      }
    
      public boolean send(String topic, String message) {
        boolean success = true;
        try {
          kafkaTemplate.send(topic, message).get(10, TimeUnit.SECONDS);
        } catch (Exception e) {
          success = false;
          logger.error("发送kafka失败, topic: {}, message: {}", topic, message, e);
        }
        return success;
      }
    }
    ```
    
  + <a href="https://gitee.com/ivanblade/JavaNewDemo/tree/master/helloKafka">完整项目</a>

#### kafka原理

+ 分区副本机制
  + kafka有三层结构：kafka有多个主题，每个主题有多个分区，每个分区又有多条消息。
    + 分区机制（一份数据分为n个分区保存）：
      + 主要解决了单台服务器存储容量有限和单台服务器并发数限制的问题 一个分片的不同副本不能放到同一个broker上。 当主题数据量非常大的时候，一个服务器存放不了，就将数据分成两个或者多个部分，存放在多台服务器上。每个服务器上的数据，叫做一个分片
    + 分区优势：
      + 分区对于 Kafka 集群的好处是：实现负载均衡，高存储能力、高伸缩性。分区对于消费者来说，可以提高并发度，提高效率。
    + 副本（一份数据的拷贝）：
      + 副本备份机制解决了数据存储的高可用问题 当数据只保存一份的时候，有丢失的风险。为了更好的容错和容灾，将数据拷贝几份，保存到不同的机器上。
    + 多个follower副本通常存放在和leader副本不同的broker中。通过这样的机制实现了高可用，当某台机器挂掉后，其他follower副本也能迅 速”转正“，开始对外提供服务。
    + 副本作用
      + 在kafka中，实现副本的目的就是冗余备份，且仅仅是冗余备份，所有的读写请求都是由leader副本进行处理的。follower副本仅有一 个功能，那就是从leader副本拉取消息，尽量让自己跟leader副本的内容一致。
    + 为什么副本不对外提供服务
      + 这个问题本质上是对性能和一致性的取舍。试想一下，如果follower副本也对外提供服务那会怎么样呢？首先，性能是肯定会有所提升 的。但同时，会出现一系列问题。类似数据库事务中的幻读，脏读。

+ kafka如何保证消息不丢失

  + 从Kafka的大体角度上可以分为数据生产者，Kafka集群，还有就是消费者，而要保证数据的不丢失也要从这三个角度去考虑。

  + 消息生产者保证数据不丢失：消息确认机制（ACK机制）,参考值有三个：0,1，-1

    ```java
    //producer无需等待来自broker的确认而继续发送下一批消息。
    //这种情况下数据传输效率最高，但是数据可靠性确是最低的。
    properties.put(ProducerConfig.ACKS_CONFIG,"0");
    //producer只要收到一个分区副本成功写入的通知就认为推送消息成功了。
    //这里有一个地方需要注意，这个副本必须是leader副本。
    //只有leader副本成功写入了，producer才会认为消息发送成功。
    properties.put(ProducerConfig.ACKS_CONFIG,"1");
    //ack=-1，简单来说就是，producer只有收到分区内所有副本的成功写入的通知才认为推送消息成功了。
    properties.put(ProducerConfig.ACKS_CONFIG,"-1");
    ```

  + 消费者消费机制
  
    + kafka通过offset记录读取消息的位置
  
    + 什么时候消费者丢失数据呢？
  
      + 由于Kafka consumer默认是自动提交位移的（先更新位移，再消费消息），如果消费程序出现故障，没消费完毕，则丢失了消息，此时，broker并不知道。
  
    + 处理策略
  
      + 关闭自动提交位移改,在编写业务逻辑时判断消息被完整消费后执行手动提交
  
        ```properties
        enableAutoCommit: false 关闭自动提交位移
        ```

#### 消息存储以及查询机制

+ 简介
  + kafka 使用日志文件的方式来保存生产者消息，每条消息都有一个 offset 值来表示它在分区中的偏移量
  + Kafka 中存储的一般都是海量的消息数据，为了避免日志文件过大，一个分片并不是直接对应在一个磁盘上的日志文件，而是对应磁盘上 的一个目录，这个目录的命名规则是topicName_partitionId(kafka容器数据目录：/kafka/kafka-logs-kafka1)。

#### 消息存储机制

+ kafka一个分区内如果出现大量数据如何存储
  + 对文件切分多文件存储
+ 为什么对文件切分保存到多个文件中
  + kafka作为消息中间件只会临时存储数据不是永久保存，需要删除过期数据，如果所有的数据都在一个文件中，数据删除比较麻烦，如果分成多个文件，删除过期数据只需要按日期属性删除即可，默认保留168小时，也就是7天的数据
+ log分段
  + 每一个分片目录中，kafka通过分段的方式将数据分为多个logsegment，一个segment对应一个log文件和一个index文件，log文件保存日志，index文件保存索引
  + 每个LogSegment 的大小可以在server.properties 中log.segment.bytes=107370 （设置分段大小,默认是1gb）选项进行设置。当log文件等于1G时，新的会写入到下一个segment中。
  + timeindex文件，是kafka的具体时间日志

#### 通过offset查找message

+ 存储结构：一个主题 --> 多个分区 ----> 多个日志段（多个文件）

  + 查询segment file：

    + segment file命名规则跟offset有关，根据segment file可以知道它的起始偏移量，因为Segment file的命名规则是上一个segment文件 最后一条消息的offset值。所以只要根据offset 二分查找文件列表，就可以快速定位到具体文件。

      ```txt
      比如
      第一个segment file是00000000000000000000.index表示最开始的文件，起始偏移量(offset)为0。
      第二个是00000000000000091932.index：代表消息量起始偏移量为91933 = 91932 + 1。那么offset=5000时应该定位
      00000000000000000000.index
      ```

  + 通过segment file查找message：

    + 通过第一步定位到segment file，当offset=5000时，依次定位到00000000000000000000.index的元数据物理位置和 00000000000000000000.log的物理偏移地址，然后再通过00000000000000000000.log顺序查找直到offset=5000为止。

#### 生产者消息分发策略

+ kafka在数据生产的时候，有一个数据分发策略。默认使用DefaultPartitioner.class类。

  + 如果是用户指定了partition，生产就不会调用DefaultPartitioner.partition()方法 数据分发策略的时候，可以指定数据发往哪个partition。 当ProducerRecord 的构造参数中有partition的时候，就可以发送到对应partition上。

  + partition()方法

    + 如果指定key，是取决于key的hash值对分区数取余 如果不指定key，轮询分发

  + 指定区分示例

    + <a herf="https://blog.csdn.net/Lu_Xiao_Yue/article/details/84864123">参考博客</a>
    + application.properties指定分区配置类

    ```properties
    #此处对应的是kafka定义分区的类
    partitioner.class=com.node.demo.kafka.Partitions
    ```

    + 编写指定分区配置类

      ```java
      设置分区
      package com.node.demo.kafka;
      
      import org.apache.kafka.clients.producer.Partitioner;
      import org.apache.kafka.common.Cluster;
      
      import java.util.Map;
      
      public class Partitions implements Partitioner{
          @Override
          public int partition(String topic, Object key, byte[] keyBytes, Object o1, byte[] valueBytes, Cluster cluster) {
              //这里返回几就代表是生产者生产数据到那个分区中
              final String value = new String(valueBytes);
              //final int intvalues = Integer.parseInt(value);
              return 1;
          }
      
          @Override
          public void close() {
      
          }
          @Override
          public void configure(Map<String, ?> map) {
      
          }
      }
      
      
      ```

      

#### 消费者负载均衡机制

##### 如何保证消息顺序

+ 发送消息时指定分区
+ 携带key值，kafka会计算key值hash并取模决定消息存储分区，相当于间接指定分区

##### 如何保证消费只被一个消费者中的多个消费者消费一次

+ 同一个分区中的数据，只能被一个消费者组中的一个消费者所消费

  + 例如 P0分区中的数据不能被Consumer Group A中C1与C2同时消费

+ 消费组：一个消费组中可以包含多个消费者，properties.put(ConsumerConfig.GROUP_ID_CONFIG,"groupName")；如果该消费组有四个 消费者，主题有四个分区，那么每人一个。多个消费组可以重复消费消息，但是单个消费组中的所有成员消费的数据不会重复。

  ```xml
  如果有3个Partition, p0/p1/p2，同一个消费组有3个消费者，c0/c1/c2，则为一一对应关系；
  如果有3个Partition, p0/p1/p2，同一个消费组有2个消费者，c0/c1，则其中一个消费者消费2个分区的数据，另一个消费者消费一个
  分区的数据；
  如果有2个Partition, p0/p1，同一个消费组有3个消费者，c0/c1/c3，则其中有一个消费者空闲，另外2个消费者消费分别各自消费一个分区的数据；
  ```

#### kafka配置文件说明

+ server.properties

  + broker.id
    + kafka集群是由多个节点组成的，每个节点称为一个broker，中文翻译是代理。每个broker都有一个不同的brokerId，由broker.id指定， 是一个不小于0的整数，各brokerId必须不同，但不必连续。如果我们想扩展kafka集群，只需引入新节点，分配一个不同的broker.id即可。
    + 启动kafka集群时，每一个broker都会实例化并启动一个kafkaController，并将该broker的brokerId注册到zooKeeper的相应节点中。集 群各broker会根据选举机制选出其中一个broker作为leader，即leader kafkaController。leader kafkaController负责主题的创建与删除、 分区和副本的管理等。当leader kafkaController宕机后，其他broker会再次选举出新的leader kafkaController。

+ log.dir

  + broker持久化消息到哪里，数据目录

+ log.retention.hours = 168

  + log文件最小存活时间，默认是168h，即7天。相同作用的还有log.retention.minutes、log.retention.ms。retention是保存的意思。 数据存储的最大时间超过这个时间会根据log.cleanup.policy设置的策略处理数据，也就是消费端能够多久去消费数据。 log.retention.bytes和log.retention.hours任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖。

+ log.retention.check.interval.ms

  + 多长时间检查一次是否有log文件要删除。默认是300000ms，即5分钟。

+ log.retention.bytes

  + 限制单个分区的log文件的最大值，超过这个值，将删除旧的log，以满足log文件不超过这个值。默认是-1，即不限制。

+ log.roll.hours

  + 多少时间会生成一个新的log segment，默认是168h，即7天。相同作用的还有log.roll.ms、segment.ms。

+ log.segment.bytes

  + log segment多大之后会生成一个新的log segment，默认是1073741824，即1G。

+ log.flush.interval.messages

  + 指定broker每收到几个消息就把消息从内存刷到硬盘（刷盘）。默认是9223372036854775807 好大。 kafka官方不建议使用这个配置，建议使用副本机制和操作系统的后台刷新功能，因为这更高效。这个配置可以根据不同的topic设置不同的 值，即在创建topic的时候设置值。

  + 补充

    ```txt
    在Linux操作系统中，当我们把数据写入到文件系统之后，数据其实在操作系统的page cache里面，并没有刷到磁盘上去。如果此时操作系统
    挂了，其实数据就丢了。
    1、kafka是多副本的，当你配置了同步复制之后。多个副本的数据都在page cache里面，出现多个副本同时挂掉的概率比1个副本挂掉，概率
    就小很多了
    2、操作系统有后台线程，定期刷盘。如果应用程序每写入1次数据，都调用一次fsync，那性能损耗就很大，所以一般都会在性能和可靠性之间
    进行权衡。因为对应一个应用来说，虽然应用挂了，只要操作系统不挂，数据就不会丢。
    ```

+ delete.topic.enable

  + 是否允许从物理上删除topic

#### kafka监控与维护

+ kafka-eagle

  + <a href="http://download.kafka-eagle.org/">kafka-eagle官网下载1.3.2.tar.gz即可</a>

  + 解压压缩包在linux123上

  + 准备mysql数据库用于给eagle保存元信息
    ```sql
    create database eagle;
    ```

  +  修改kafka-eagle配置文件

    ```shell
    cd /opt/lagou/servers/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2/conf
    vim system-config.properties
    #内容如下:
    kafka.eagle.zk.cluster.alias=cluster1
    cluster1.zk.list=192.168.8.101:2181,192.168.8.102:2181,192.168.8.103:2181
    kafka.eagle.driver=com.mysql.jdbc.Driver
    kafka.eagle.url=jdbc:mysql://192.168.8.103:3306/eagle
    kafka.eagle.username=root
    kafka.eagle.password=xxxxxx
    ```

  + 默认情况下MySQL只允许本机连接到MYSQL实例中，所以如果要远程访问，必须开放权限：

    ```sql
    update user set host = '%' where user ='root'; //修改权限
    flush privileges; //刷新配置
    ```

  + 配置环境变量

    ```shell
    vi /etc/profile
    #内容如下:
    export KE_HOME=/opt/lagou/servers/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2
    export PATH=:$KE_HOME/bin:$PATH
    #让修改立即生效，执行
    source /etc/profile
    ```

  + 启动

    ```shell
    cd kafka-eagle-web-1.3.2/bin
    chmod u+x ke.sh
    ./ke.sh start
    ```

  + <a href="http://node03:8048/ke/account/signin?/ke/">访问主界面</a>

    + 用户名：admin
    + 密码：123456 

### 进阶

#### 