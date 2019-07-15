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

   

### 事务性

事务性：满足ACID，可以通过commit提交一个事务，也可以使用rollback进行回滚。

1. 原子性：要么全部提交成功。要么全部失败回滚。

2. 一致性：数据库在事务执行前后都保持一致性状态。

3. 隔离性：一个事务所做的修改在最终提交以前，对其他事务是不可见的。

4. 持久性：一旦事务提交，则其所做的修改将会永远保存到数据库中，即使系统发生崩溃，事务执行的结果也不能丢失。

### 并发一致性问题

在并发环境下，事务的隔离性很难保证，因此会出现很多并发一致性问题。

##### 丢失修改

T1、T2对同一个数据进行修改，T1先修改，T2随后修改，T2的修改覆盖了T1的修改。

##### 读脏数据

T1修改一个数据，T2随后读取这个数据，如果T1撤销了这次更改，T2读取的是脏数据。

##### 不可重复读

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

##### 幻影读

T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

产生并发不一致性问题主要原因是破坏了事务的隔离性，解决方法是通过并发控制来保证隔离性

### 封锁

##### 封锁粒度

MySQL中提供了两种封锁粒度：行级锁以及表级锁。

应该尽量只锁定需要修改的那部分数据，而不是所有的资源。锁定的数据量越少，发生锁争用的可能就越小，系统的并发程度就越高。

但是加锁需要消耗资源，锁的各种操作（包括获取锁、释放锁、以及检查锁状态）都会增加系统开销。因此封锁粒度越小，系统开销就越大。

在选择封锁粒度时，需要在锁开销和并发程度之间做一个权衡。