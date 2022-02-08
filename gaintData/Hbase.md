### Hbase

#### 简介

##### Hbase是什么

##### Hbase特点

##### Hbase应用

##### 数据模型

##### 整体架构

##### 集群安装部署

+ 安装hadoop环境，zookeeper环境

+ <a href="http://archive.apache.org/dist/hbase/1.3.1/">hbase-1.3.1-bin.tar.gz</a>

+ 解压压缩包

  ```shell
  tar -zxvf hbase-1.3.1-bin.tar.gz -C /opt/lagou/servers
  ```

+ 拷贝配置文件

  ```shell
  ln -s /opt/lagou/servers/hadoop-2.9.2/etc/hadoop/core-site.xml /opt/lagou/servers/hbase-1.3.1/conf/core-site.xml
  ln -s /opt/lagou/servers/hadoop-2.9.2/etc/hadoop/hdfs-site.xml /opt/lagou/servers/hbase-1.3.1/conf/hdfs-site.xml
  ```

+ 修改conf目录下配置文件

  + 修改 hbase-env.sh

    ```shell
    #添加java环境变量
    export JAVA_HOME=/opt/module/jdk1.8.0_231 
    #指定使用外部的zk集群
    export HBASE_MANAGES_ZK=FALSE
    
  # 指定id存放目录，否则会导致删除脚本找不到相关文件无法执行,这边找到对应行取消注释之后创建指定文件夹即可
    export HBASE_PID_DIR=/var/hadoop/pids
  
    mkdir -p /var/hadoop/pids
    
    ```
  
  + 修改 hbase-site.xml
  
    ```xml
    <configuration>
    <!-- 指定hbase在HDFS上存储的路径 -->
            <property>
                    <name>hbase.rootdir</name>
                    <value>hdfs://linux121:9000/hbase</value>
            </property>
    <!-- 指定hbase是分布式的 --> 
      			<property>
                    <name>hbase.cluster.distributed</name>
                    <value>true</value>
            </property>
    <!-- 指定zk的地址，多个用“,”分割 --> 
    			<property>
                    <name>hbase.zookeeper.quorum</name>
                  <value>linux121:2181,linux122:2181,linux123:2181</value>
            </property>
    </configuration>
    ```
  
  + 修改regionservers文件
  
  ```shell
    #指定regionserver节点 
  linux121
    linux122
    linux123
    ```

  + hbase的conf目录下创建文件backup-masters (Standby Master)

    ```txt
    linux122
    ```
  
+ 配置hbase的环境变量
  
  ```shell
    export HBASE_HOME=/opt/lagou/servers/hbase-1.3.1
    export PATH=$PATH:$HBASE_HOME/bin
    ```

  + 分发hbase目录和环境变量到其他节点

    ```shell
    rsync-script hbase-1.3.1
    ```

  + 让所有节点的hbase环境变量生效

    ```shell
    source /etc/profile
  ```
  
  + 前提条件:先启动hadoop和zk集群 
  
    + 启动HBase:start-hbase.sh
    + 停止HBase:stop-hbase.sh
  
  + web界面：HMaster的主机名:16010

##### shell基本操作

```shell
# 进入客户端命令操作界面
hbase shell
# 查看帮助命令
help
# 查看当前数据库有哪些表
list
# 创建一张lagou表，base_info,extra_info(hbase建表必须指定列族)
create 'lagou', 'base_info', 'extra_info'
# 或者
create 'lagou', {NAME => 'base_info', VERSIONS => '3'},{NAME =>'extra_info',VERSIONS => '3'}
# VERSIONS 是指此单元格内的数据可以保留最近的 3 个版本

# 添加数据操作
# 向lagou表中插入信息，row key为 rk1，列族base_info中添加name列标示符，值为wang
put 'lagou', 'rk1', 'base_info:name', 'wang'
# 向lagou表中插入信息，row key为rk1，列族base_info中添加age列标示符，值为30
hbase(main):001:0>  put 'lagou', 'rk1', 'base_info:age', 30
# 向lagou表中插入信息，row key为rk1，列族extra_info中添加address列标示符，值为shanghai
put 'lagou', 'rk1', 'extra_info:address', 'shanghai'

# 查询数据
# 通过rowkey进行查询
# 获取表中rowkey为rk1的所有信息
get 'lagou', 'rk1'
# 查看rowkey下面的某个列族的信息
# 获取lagou表中rowkey为rk1，base_info列族的所有信息
get 'lagou', 'rk1', 'base_info'
# 查看rowkey指定列族指定字段的值
# 获取表中rowkey为rk1，base_info列族的name、age列标示符的信息
get 'lagou', 'rk1', 'base_info:name', 'base_info:age'

# 查看rowkey指定多个列族的信息
# 获取lagou表中row key为rk1，base_info、extra_info列族的信息
get 'lagou', 'rk1', 'base_info', 'extra_info'
get 'lagou', 'rk1', {COLUMN => ['base_info', 'extra_info']}
get 'lagou', 'rk1', {COLUMN => ['base_info:name','extra_info:address']}

# 指定rowkey与列值查询
# 获取表中row key为rk1，cell的值为wang的信息
get 'lagou', 'rk1', {FILTER => "ValueFilter(=,'binary:wang')"}

# 指定rowkey与列值模糊查询
# 获取表中row key为rk1，列标示符中含有a的信息
get 'lagou', 'rk1', {FILTER => "(QualifierFilter(=,'substring:a'))"}

# 查询所有数据
# 查询lagou表中的所有信息
scan 'lagou'

# 列族查询
# 查询表中列族为 base_info 的信息
scan 'lagou', {COLUMNS => 'base_info'}
can 'lagou', {COLUMNS => 'base_info', RAW => true, VERSIONS => 3}
## Scan时可以设置是否开启Raw模式,开启Raw模式会返回包括已添加删除标记但是未实际删除的数据 
## VERSIONS指定查询的最大版本数

# 指定多个列族与按照数据值模糊查询
# 查询lagou表中列族为 base_info 和 extra_info且列标示符中含有a字符的信息
scan 'lagou', {COLUMNS => ['base_info', 'extra_info'], FILTER => "(QualifierFilter(=,'substring:a'))"}

# rowkey的范围值查询(非常重要)
# 查询lagou表中列族为base_info，rk范围是[rk1, rk3)的数据(rowkey底层存储是字典序)
# 按rowkey顺序存储。
scan 'lagou', {COLUMNS => 'base_info', STARTROW => 'rk1',ENDROW => 'rk3'}

# 指定rowkey模糊查询
# 查询lagou表中row key以rk字符开头的
scan 'lagou',{FILTER=>"PrefixFilter('rk')"}

# 更新数据
# 更新操作同插入操作一模一样，只不过有数据就更新，没数据就添加
# 更新数据值
# 把lagou表中rowkey为rk1的base_info列族下的列name修改为liang
put 'lagou', 'rk1', 'base_info:name', 'liang'


# 删除数据和表
# 指定rowkey以及列名进行删除
# 删除lagou表row key为rk1，列标示符为 base_info:name 的数据
delete 'lagou', 'rk1', 'base_info:name'

# 指定rowkey，列名以及时间戳信息进行删除
# 删除lagou表row key为rk1，列标示符为base_info:name的数据
### 待补充

# 删除列族
# 删除 base_info 列族
# alter 'lagou', 'delete' => 'base_info'

# 清空表数据
# 删除lagou表数据
truncate 'lagou'

# 删除表
# #先disable 再drop hbase
disable 'lagou'
drop 'lagou'
#如果不进行disable，直接drop会报错
ERROR: Table user is enabled. Disable it first.
```



#### Hbase原理

##### Hbase读数据流程

##### HBase写数据流程

##### Hbase的flush（刷写）以及compact（合并）机制

##### Region拆分机制

##### HBase表的预分区

##### Region合并

#### HbaseAPI应用和优化

+ <a href="https://github.com/Ivan-blade/JavaNewDemo/tree/master/helloKafka">点击跳转demo</a>

##### HBase API客户端操作

##### Hbase协处理器

##### Hbase表的RowKey设计

##### Hbase的二级索引

##### 布隆过滤器在hbase的应用