### redis

#### redis快速实战

##### 缓存原理与设计

+ 缓存基本思想
+ 缓存的优势和代价
+ 缓存的读写模式
+ 缓存架构的设计思路

##### redis简介与安装

+ 简介

+ 安装

  + 修改redis配置文件关闭保护模式

  + 进入bin目录

    ```shell
    ./redis-server ../redis.conf 
    
    nohup ./redis-server ../redis.conf &
    ```

    


##### redis客户端访问

+ java访问
+ spring访问
+ springboot访问

##### redis数据类型选择与应用场景

+ string字符串
+ list列表
+ set集合
+ sortedSet
+ hash
+ bitmap
+ geo

#### redis扩展功能

##### 发布与订阅

+ 频道模式的订阅与退订
+ 检验发送的信息
+ 使用场景的哨兵模式，单框架使用，弱化框架集成

##### 事务

+ ACID回顾
+ 事务使用

##### Lua脚本

+ 创建修改lua环境
+ lua环境协作组件
+ EVAL、EVALSHA命令实现
+ 脚本管理命令实现
+ 脚本复制

##### 慢查询日志

##### 监听器

#### redis核心原理

##### redis持久化

+ RDB
  + bgsave触发方式
  + bgsave执行流程
  + RDB文件结构
  + 分析RDB文件
+ AOF
  + AOF持久化实现
  + AOF重写，触发方式，混合持久化
  + AOF文件的载入与数据还原
+ 对比
+ RDB与AOF过期键的处理
+ 应用场景

##### 底层数据结构

+ redisDB结构
  + dict
  + expires
  + id
+ RedisObject结构
  + 结构信息概览
    + 4位type
    + 4位encodeing
    + 24位LRU（高16位和低8位作用）
    + refcount
    + ptr
  + 7种type
    + 字符串对象
    + 列表对象
    + 集合对象
    + 有序集合对象
    + 散列表对象
    + 模块对象
    + 流对象
  + 10种encoding

##### 通讯协议及事件处理机制

#### redis企业实战

##### 架构设计

##### 缓存过期与淘汰策略

##### 缓存与数据库一致性

##### 缓存问题

##### 分布式锁

##### 分布式集群架构中的session分离

##### 阿里redis使用手册

#### redis高可用方案

##### 主从复制

##### 哨兵模式

##### 	集群与分区