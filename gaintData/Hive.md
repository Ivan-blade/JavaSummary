### Hive

+ 安装与配置（Linux123）

  + hive元数据默认存储在derby数据库中，生产多用mysql

    ```shell
    # 查询是否安装了mariadb 
    # 删除mariadb。-e 删除指定的套件;--nodeps 不验证套件的相互关联性 
    rpm -aq | grep mariadb
    rpm -e --nodeps mariadb-libs
    
    # 安装依赖
    yum install perl -y
    yum install net-tools -y
    
    # 安装mysql
    # 接压缩
    tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
    # 依次运行以下命令
    rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
    rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
    rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
    rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm
    
    # 启动数据库
    # 查找root密码
    systemctl start mysqld
    grep password /var/log/mysqld.log
    
    # 进入MySQL，使用前面查询到的口令 
    mysql -u root -p
    # 设置口令强度;将root口令设置为L1178594290;刷新
    set global validate_password_policy=0;
    set password for 'root'@'localhost' =password('L1178594290'); 
    flush privileges;
    
    # 创建hive用户
    -- 创建用户设置口令、授权、刷新
    CREATE USER 'hive'@'%' IDENTIFIED BY 'L1178594290'; 
    GRANT ALL ON *.* TO 'hive'@'%';
    FLUSH PRIVILEGES;
    ```

  + hive安装

    + 安装步骤: 

      + 下载、上传、解压缩 

        ```shell
        cd /opt/lagou/software
        tar zxvf apache-hive-2.3.7-bin.tar.gz -C ../servers/
        cd ../servers
        mv apache-hive-2.3.7-bin hive-2.3.7
        ```

      + 修改环境变量 

        ```shell
        # 在 /etc/profile 文件中增加环境变量
        export HIVE_HOME=/opt/lagou/servers/hive-2.3.7
        export PATH=$PATH:$HIVE_HOME/bin
        # 执行并生效
        source /etc/profile
        ```

      + 修改hive配置 

        ```xml
        cd $HIVE_HOME/conf
        # 新建hive-site.xml
        vi hive-site.xml 
        
        <?xml version="1.0" encoding="UTF-8" standalone="no"?>
        <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
        <configuration>
        	<!-- hive元数据的存储位置 --> 
          <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://linux123:3306/hivemetadata?createDatabaseIfNotExist=true&amp;useSSL=false</value>
                <description>JDBC connect string for a JDBC metastore</description>
        	</property>
          <!-- 指定驱动程序 --> 
          <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
                <description>Driver class name for a JDBC metastore</description>
           </property>
        	 <!-- 连接数据库的用户名 --> 
           <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>hive</value>
                <description>username to use against metastore database</description>
           </property>
          <!-- 连接数据库的口令 --> 
           <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>L1178594290</value>
                <description>password to use against metastore database</description>
           </property>
          <!-- 数据默认的存储位置(HDFS) -->
          <property>
        		<name>hive.metastore.warehouse.dir</name> 
            <value>/user/hive/warehouse</value> 
            <description>location of default database for the warehouse</description>
        	</property>
        	<!-- 在命令行中，显示当前操作的数据库 --> 
          <property>
            <name>hive.cli.print.current.db</name>
        		<value>true</value>
        		<description>Whether to include the current database in the Hive prompt.</description>
        	</property>
        	<!-- 在命令行中，显示数据的表头 --> 
          <property>
            <name>hive.cli.print.header</name> 
            <value>true</value>
        	</property>
          <!-- 操作小规模数据时，使用本地模式，提高效率 --> 
          <property>
        		<name>hive.exec.mode.local.auto</name> 
            <value>true</value>
        		<description>Let Hive determine whether to run in local mode automatically</description>
        	</property>
        </configuration>
        ```

      + 拷贝JDBC的驱动程序 

        ```shell
        mv mysql-connector-java-5.1.46.jar $HIVE_HOME/lib
        ```

      + 初始化元数据库

        ```shell
        schematool -dbType mysql -initSchema
        ```

      + hive

        ```shell
        hive
        ```

        