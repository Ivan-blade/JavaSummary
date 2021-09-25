### Impala

#### 安装

+ Impala的安装需要提前装好Hadoop，Hive这两个框架

+ hive需要在所有的Impala安装的节点上⾯面都要有，因为Impala需要引⽤用Hive的依赖包

+ hadoop的框架需要⽀持C程序访问接⼝，查看下图，如果有该路路径有.so结尾⽂件，就证明⽀支持C 接⼝。

+ 配置本地yum源（默认yum源没有impala包）

  + linux121

    ```shell
     #yum⽅方式安装httpd服务器器 
    yum install httpd -y
    #启动httpd服务器器 
    systemctl start httpd
    #验证httpd⼯工作是否正常,默认端⼝口是80，可以省略略 
    http://linux121
    ```

+ 下载相关rpm包（这边是cdh），移动到/opt/lagou/software中并解压/opt/lagou/server

  ```shell
  tar -zxvf cdh5.7.6-centos7.tar.gz -C /opt/lagou/server
  ```

+ http服务器默认暴露/var/www/html目录下的文件，使用软连接方式将解压目录暴露到该目录下（直接复制也可以）

  ```shell
  ln -s /opt/lagou/software/cdh/5.7.6 /var/www/html/cdh57
  
  # 验证访问
  http://linux121/cdh57/
  # 删除软连接不加/，否则会删除链接的文件，这边是cdh57而不是cdh57/
  rm -rf cdh57
  ```

+ 修改yum源配置文件

  ```shell
   cd /etc/yum.repos.d 
   #创建⼀一个新的配置⽂文件 
   vim local.repo 
   #添加如下内容
  [local]
  name=local
  baseurl=http://linux121/cdh57/
  gpgcheck=0
  enabled=1
  ```

+ 分发到其他节点

  ```shell
  rsync-script local.repo
  ```

+ linux123

  ```shell
  yum  install  impala -y
  yum install impala-server -y
  yum install impala-state-store  -y
  yum install impala-catalog  -y
  yum  install  impala-shell -y
  ```

+ Linux121,linux122

  ```shell
  yum install impala-server -y
  yum  install  impala-shell -y
  ```

+ Linux123配置impala

  + 修改hive-site.xml

    ```xml
     vim hive-site.xml 
    <!--指定metastore地址，之前添加过可以不不⽤用添加 --> 
    <property>
      <name>hive.metastore.uris</name>
    	<value>thrift://linux121:9083,thrift://linux123:9083</value>
    </property>
    <property>
      <name>hive.metastore.client.socket.timeout</name>
    	<value>3600</value>
    </property>
    ```

  + 分发hive安装包到集群

    ```shell
    rsync-script /opt/lagou/servers/hive-2.3.7/
    ```

  + linux123启动metastore和hiveserver2

    ```shell
    nohup hive --service metastore &
    nohup hive --service hiveserver2 &
    ```

  + 修改hdfs配置设置短路读取

    ```shell
    cd /opt/lagou/servers/hadoop-2.9.2/lib/native
    ls
    # 查看是否有libhadoop.so，如果没有凉凉
    ```

  + 创建短路读取本地中转战

    ```shell
    #所有节点创建⼀一下⽬目录
    mkdir -p /var/lib/hadoop-hdfs
    ```

  + 修改hdfs-site.xml

    ```xml
     
    <!--添加如下内容 --> 
    <!--打开短路路读取开关 --> 
    <!-- 打开短路路读取配置--> 
    <property>
        <name>dfs.client.read.shortcircuit</name>
        <value>true</value>
      </property>
    <!--这是⼀一个UNIX域套接字的路路径，将⽤用于DataNode和本地HDFS客户机之间的通信 --> 
    <property>
        <name>dfs.domain.socket.path</name>
        <value>/var/lib/hadoop-hdfs/dn_socket</value>
      </property>
    <!--block存储元数据信息开发开关 --> 
    <property>
        <name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.client.file-block-storage-locations.timeout</name>
        <value>30000</value>
    </property>
    ```

  + 分发到集群其它节点。重启Hadoop集群

    ```shell
    rsync-script hdfs-site.xml
     
    #停⽌止集群 
    stop-dfs.sh 
    start-dfs.sh 
    #启动集群 
    stop-yarn.sh
    start-yarn.sh
    ```

  + 所有节点执行命令

    ```shell
    ln -s /opt/lagou/servers/hadoop-2.9.2/etc/hadoop/core-site.xml /etc/impala/conf/core-site.xml
    ln -s /opt/lagou/servers/hadoop-2.9.2/etc/hadoop/hdfs-site.xml /etc/impala/conf/hdfs-site.xml
    ln -s /opt/lagou/servers/hive-2.3.7/conf/hive-site.xml /etc/impala/conf/hive-site.xml
    
     
    vim /etc/default/impala
    <!--更更新如下内容 -->
    IMPALA_CATALOG_SERVICE_HOST=linux123
    IMPALA_STATE_STORE_HOST=linux123
    
    #创建节点
    mkdir -p /usr/share/java
    ln -s /opt/lagou/servers/hive-2.3.7/lib/mysql-connector-java-5.1.46.jar /usr/share/java/mysql-connector-java.jar
    ```

  + 修改bigtop参数

    ```shell
    vim /etc/default/bigtop-utils
    export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
    ```

  + 启动

    ```shell
     #linux123启动如下⻆角⾊色
    service impala-state-store start 
    service impala-catalog start 
    service impala-server start
    #其余节点启动如下⻆角⾊色
    service impala-server start
    ```

    