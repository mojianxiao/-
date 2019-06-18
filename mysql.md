# MySQL

### MySQL的读写分离

### 前言

<p>	解决大型网站的高并发，除了网站实现分布式负载均衡，利用主从数据库来实现读写分离，从而分担主数据库的压力也是一个好方法。
    在多个服务器上部署MySQL，将其中一台作为主数据库，而其他作为从数据库，实现主从同步。其中主数据负责主动写的操作，从数据库只负责主动读的操作，为了保持数据的一致性，从数据库仍然会被动的进行写操作。</p>
![img](https://img-blog.csdnimg.cn/20190304165712787.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDkyMDc5,size_16,color_FFFFFF,t_70)

### 配置

1. 在两台不同服务器上安装MySQL

2. 配置主服务器文件my.cnf(在/etc/my.c*nf)

   ```mysql
   server-id=1
   log_bin=master-bin
   log_bin_index=master-bin.index
   binlog_do_db=test
   #备注：
   #server-id 服务器唯一标识。
   #log_bin 启动MySQL二进制日志，即数据同步语句，从数据库会一条一条的执行这些语句。
   #binlog_do_db 指定记录二进制日志的数据库，即需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可。
   #binlog_ignore_db 指定不记录二进制日志的数据库，即不需要复制的数据库名，如果有多个数据库，重复设置这个选项即可。
   #其中需要注意的是，binlog_do_db和binlog_ignore_db为互斥选项，一般只需要一个即可！
   ```

   

3. 创建从服务器的用户和权限

   ```sql
   mysql> grant replication salve on *.* to masterbackup@'ip地址' identified by 'password';
   ```

   

4. 重启主服务器mysql服务

5. 配置从服务器文件my.cnf

   ```
   
   server-id=2
   relay-log=slave-relay-bin
   relay-log-index=slave-relay-bin.index
   重启主服务器mysql服务
   ```

   

6. 重启从服务器mysql服务

7. 连接主服务器

   ```sql
   change master to master_host='ip地址',master_port=3306,master_user='masterbackup',master_password='123456',master_log_file='master-bin.000001',master_log_pos=154;
   ```

8. 启动slave数据同步并检测

   ```
   mysql> start slave
   mysql> show slave status\G
   ```

   

