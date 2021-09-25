#### hadoop集群安装

+ Stage3-Module1

  + 三台时间同步免密虚拟机

  + java8环境

  + hadoop2.9.2

  + 安装步骤

    + 解压hadoop

      ```shell
      tar -zxvf /opt/lagou/software/hadoop-2.9.2.tar.gz  -C /opt/lagou/software/
      ```

    + 添加hadoop环境变量

      ```shell
      vim  /etc/profile
      ##HADOOP_HOME
      export HADOOP_HOME=/opt/lagou/servers/hadoop-2.9.2
      export PATH=$PATH:$HADOOP_HOME/bin
      export PATH=$PATH:$HADOOP_HOME/sbin
      
      source /etc/profile
      ```

    + 解压后

      + bin
        + 操作hadoop的命令
      + sbin
        + 启动关闭hadoop的命令
      + etc
        + hadoop配置信息

#### 集群配置

+ 集群安排
  + linux121
    + HDFS NameNode
    + DataNode
    + YARN NodeManager
  + linux122
    + DataNode
    + NodeManager
  + linux123
    + SecondaryNameNode
    + DataNode
    + NodeManager
    + ResourceManager

+ Hadoop集群配置 = HDFS集群配置 + MapReduce集群配置 + Yarn集群配置

  + HDFS集群配置

    1. 将JDK路径明确配置给HDFS(修改hadoop-env.sh)
    2. 指定NameNode节点以及数据存储目录(修改core-site.xml)
    3.  指定SecondaryNameNode节点(修改hdfs-site.xml)

    4. 指定DataNode从节点(修改etc/hadoop/slaves文件，每个节点配置信息占一行)

  + MapReduce集群配置

    1. 将JDK路径明确配置给MapReduce(修改mapred-env.sh)
    2. 指定MapReduce计算框架运行Yarn资源调度框架(修改mapred-site.xml) 

  + Yarn集群配置
    1. 将JDK路径明确配置给Yarn(修改yarn-env.sh)
    2. 指定ResourceManager老大节点所在计算机节点(修改yarn-site.xml) 
    3. 指定NodeManager节点(会通过slaves文件内容确定)

+ 安装步骤

  + 编辑配置文件指定java路径

    ```shell
    cd /opt/lagou/servers/hadoop-2.9.2/etc/hadoop
    vim hadoop-env.sh
    export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
    ```

  + 指定NameNode节点以及数据存储目录(修改core-site.xml)

    ```shell
    vim core-site.xml
    
    <!-- 指定HDFS中NameNode的地址 -->
    <configuration>
      <!-- 指定HDFS中NameNode的地址 -->
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://linux121:9000</value>
      </property>
      <!-- 指定Hadoop运行时产生文件的存储目录 --> 
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/opt/lagou/servers/hadoop-2.9.2/data/tmp</value>
      </property>
    </configuration>
    ```

  + 指定secondarynamenode节点(修改hdfs-site.xml)

    ```shell
    vim hdfs-site.xml
    
    <configuration>
      <!-- 指定Hadoop辅助名称节点主机配置 -->
      <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>linux123:50090</value>
      </property>
      <!--副本数量 -->
      <property>
              <name>dfs.replication</name>
              <value>3</value>
      </property>
    </configuration>
    ```

  + 指定datanode节点

    ```shell
    vim slaves
    linux121
    linux122
    linux123
    ```

  + mapreduce指定jdk路径

    ```shell
    vim mapred-env.sh 
    export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
    ```

  + 指定mapreduce启动时调用的框架

    ```xml
    mv mapred-site.xml.template  mapred-site.xml
    
    vim mapred-site.xml
    <configuration>
      <!-- 指定MR运行在Yarn上 --> 
      <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    	</property>
    </configuration>
    
    ```

  + 指定yarn的jdk路径

    ```shell
    vim yarn-env.sh
    export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
    ```

    

  + 指定ResourceMnager的master节点信息(修改yarn-site.xml)

    ```xml
    vim yarn-site.xml
    
    <configuration>
      <!-- 指定YARN的ResourceManager的地址 --> 
      <property>
              <name>yarn.resourcemanager.hostname</name>
              <value>linux123</value>
      </property>
      <!-- Reducer获取数据的方式 --> 
      <property>
              <name>yarn.nodemanager.aux-services</name>
              <value>mapreduce_shuffle</value>
      </property>
    </configuration>
    ```

  + 编写差异远程传输脚本

    ```shell
    yum install -y rsync
    cd /usr/local/bin
    touch rsync-script
    vim rsync-script
    
    #!/bin/bash
    #1 获取命令输入参数的个数，如果个数为0，直接退出命令 
    paramnum=$#
    if((paramnum==0)); then
    echo no params;
    exit;
    fi
    #2 根据传入参数获取文件名称 
    p1=$1 
    file_name=`basename $p1` 
    echo fname=$file_name
    #3 获取输入参数的绝对路径
    pdir=`cd -P $(dirname $p1); pwd`
    echo pdir=$pdir 
    #4 获取用户名称
    user=`whoami`
    #5 循环执行rsync
    for((host=121; host<124; host++)); do
    echo ------------------- linux$host --------------
     rsync -rvl $pdir/$file_name $user@linux$host:$pdir
    done
    ```
  
  + 配置历史服务器
  
    ```shell
    vi mapred-site.xml
    
    <!-- 历史服务器端地址 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>linux121:10020</value>
    </property>
    <!-- 历史服务器web端地址 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>linux121:19888</value>
    </property>
    
    # 分发到其他服务
    rsync-script mapred-site.xml
    
    # 启动服务
    sbin/mr-jobhistory-daemon.sh start historyserver
    # 查看是否启动
    jps
    # 访问
    http://linux121:19888/jobhistory
    ```
  
    

#### 启动hadoop集群(命令都在sbin目录下)

+ 单节点

  + 第一次启动集群需要在namenode节点(linux121)初始化

    ```shell
    hadoop namenode -format
    ```

  + Linux121启动namenode

    ```shell
    hadoop-daemon.sh start namenode
    ```

  + 在linux121、linux122以及linux123上分别启动DataNode

    ```shell
    hadoop-daemon.sh start datanode
    ```

  + 外部访问

    ```shell
    http://linux121:50070/dfshealth.html#tab-overview
    ```

  + yarn集群单节点启动
  
    ```shell
    # linux123
    yarn-daemon.sh start resourcemanager
    
    #linux122,linux121
    yarn-daemon.sh start nodemanager
    ```
  
+ 集群

  + 第一次启动需要在namenode节点格式化

    ```shell
    hadoop namenode -format
    ```

  + 启动/停止hdfs

    ```shell
    # linux121
    ./start-dfs.sh
    ./stop-dfs.sh
    ```

  + 启动/停止yarn

    ```shell
    # linux123
    ./start-yarn.sh
    ./stop-yarn.sh
    ```

    

