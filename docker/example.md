### docker下的mysql主从复制  
- 主从复制的原理  
  MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；
  主库推送二进制日志文件Binlog中的事件到从库的中继日志Relay Log中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库的数据一致性；
  MySql通过三个线程来完成主从库间的数据复制，其中Binlog Dump线程跑在主库上，I/O线程和SQL线程跑着从库上；
  当在从库上启动复制时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给I/O线程，I/O线程获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的SQL线程读取中继日志Relay Log中更新的数据库事件并应用
- 搭建示例
- 运行mysql实例  
  docker run -p 3307:3306 --name mysql-master \
  -v /mydata/mysql-master/log:/var/log/mysql \
  -v /mydata/mysql-master/data:/var/lib/mysql \
  -v /mydata/mysql-master/conf:/etc/mysql \
  -e MYSQL_ROOT_PASSWORD=root  \
  -d mysql:latest
- 在mysql的配置文件夹/mydata/mysql-master/conf中创建一个配置文件my.cnf  
    ```
    [mysqld]
    ## 设置server_id，同一局域网中需要唯一
    server_id=101 
    ## 指定不需要同步的数据库名称
    binlog-ignore-db=mysql
    ## 开启二进制日志功能
    log-bin=mall-mysql-bin
    ## 设置二进制日志使用内存大小（事务）
    binlog_cache_size=1M
    ## 设置使用的二进制日志格式（mixed,statement,row）
    binlog_format=mixed
    ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
    binlog_expire_logs_seconds=604800
    ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
    ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
    slave_skip_errors=1062
    ```
- 重启容器，进入容器，创建用户。
  CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
  GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
- 从库搭建  
  docker run -p 3308:3306 --name mysql-slave \
  -v /mydata/mysql-slave/log:/var/log/mysql \
  -v /mydata/mysql-slave/data:/var/lib/mysql \
  -v /mydata/mysql-slave/conf:/etc/mysql \
  -e MYSQL_ROOT_PASSWORD=root  \
  -d mysql:latest
- 在mysql的配置文件夹/mydata/mysql-slave/conf中创建一个配置文件my.cnf：
    ```
    [mysqld]
    ## 设置server_id，同一局域网中需要唯一
    server_id=102
    ## 指定不需要同步的数据库名称
    binlog-ignore-db=mysql
    ## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
    log-bin=mall-mysql-slave1-bin
    ## 设置二进制日志使用内存大小（事务）
    binlog_cache_size=1M
    ## 设置使用的二进制日志格式（mixed,statement,row）
    binlog_format=mixed
    ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
    binlog_expire_logs_seconds=604800
    ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
    ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
    slave_skip_errors=1062
    ## relay_log配置中继日志
    relay_log=mall-mysql-relay-bin
    ## log_slave_updates表示slave将复制事件写进自己的二进制日志
    log_slave_updates=1
    ## slave设置为只读（具有super权限的用户除外）
    read_only=1
    ```
- 进行主从链接
  show master status;查看主库状态
  change master to master_host='内网ip', master_user='slave', master_password='123456', master_port=3307, master_log_file=blog名称', master_log_pos=xxx, master_connect_retry=30;进入从库配置
  show slave status \G;查看从库同步状态
  start slave 开启同步。


