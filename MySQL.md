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

##### 封锁类型

MySQL 的 InnoDB 存储引擎采用两段锁协议，会根据隔离级别在需要的时候自动加锁，并且所有的锁都是在同一时刻被释放，这被称为隐式锁定。

##### 隔离级别

1. 未提交读：事务中的修改，即使没有提交，对其他事务也是可见的。
2. 提交读：一个事物只能读取已经提交的事务所做的修改。
3. 可重复读：保证在同一个事务中多次读取同样的结果是一样的。
4. 可串行化：强制事务串行执行。需要加锁处理。

### 关系数据库设计理念

#### 函数依赖

记 A->B 表示 A 函数决定 B，也可以说 B 函数依赖于 A。

如果 {A1，A2，... ，An} 是关系的一个或多个属性的集合，该集合函数决定了关系的其它所有属性并且是最小的，那么该集合就称为键码。

对于 A->B，如果能找到 A 的真子集 A'，使得 A'-> B，那么 A->B 就是部分函数依赖，否则就是完全函数依赖。

对于 A->B，B->C，则 A->C 是一个传递函数依赖。

#### 范式

1. 第一范式：属性不可分
2. 第二范式：每个非主属性完全函数依赖于键码
3. 第三范式：不传递函数依赖

### SQL基本操作

<font color=green size=4px>创建表</font>	

```
CREATE Table student(
	id INT NOT NULL AUTO_INCREMENT,
	name VARCHAR(45) NOT NULL DEFAULT 2,
	PRIMARY KEY(`id`)
);
```

<font color=green size=4px>修改表</font>	

添加列

```mysql
ALTER TABLE student
add col char(20);
```

删除列

```mysql
ALTER TABLE student
DROP COLUMN col;
```

删除表

```
DROP TABLE student;

```

<font color=green size=4px>插入</font>

普通插入

```mysql
INSERT INTO student(id,name) VALUES(1,'小明');

```

插入检索出来的数据

```mysql
INSERT INTO mytable1(col1, col2)
SELECT col1, col2
FROM mytable2;

```

将一个表的内容插入到一个新表

```mysql
CREATE TABLE newtable AS
SELECT * FROM mytable;

```

<font color=green size=4px>更新表</font>	

```mysql
UPDATE student SET name = '小张' where id =1 

```

<font color=green size=4px>删除</font>	

```mysql
DELETE FROM student where id =1

```

TRUNCATE TABLE 可以清空表，也就是删除所有行

```mysql
TRUNCATE TABLE student

```

<font color=green size=4px>查询</font>	

DISTINCT：相同值只出现一次，作用于所有列

```mysql
SELECT DISTINCT col,col2
FROM student

```

LIMIT:限制返回的行数

```mysql
SELECT * FROM
student LIMIT 5;

```

要返回3~5行

```mysql
SELECT * FROM student LIMIT 2,3

```

<font color=green size=4px>排序</font>	

ASC:升序 

DESC:降序

```mysql
SELECT * FROM student 
ORDER BY col DESC,col2 ASC;

```

<font color=green size=4px>过滤</font>	

```mysql
SELECT *
FROM mytable
WHERE col IS NULL;

```

| 操作符  | 说明         |
| ------- | ------------ |
| =       |              |
| <       | 小于         |
| >       |              |
| <>!=    | 不等于       |
| <=!>    | 小于等于     |
| >=!<    | 大于等于     |
| BETWEEN | 在两个值之间 |
| IS NULL |              |

<font color=green size=4px>计算字段</font>

计算字段通常需要使用 **AS** 来取别名，否则输出的时候字段名为计算表达式。

```mysql
SELECT  id * `password` as try from `admin` where id = 1

```

```mysql
SELECT concat(TRIM(id),`password`) as demo from `admin`                                                                                                                                                                                                                                                                                                            

```

<font color=green size=4px>分组</font>

把具有相同的数据值的行放在同一组中。

可以对同一分组数据使用汇总函数进行处理，例如求分组数据的平均值等。

指定的分组字段除了能按该字段进行分组，也会自动按该字段进行排序。

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col;

```

WHERE 过滤行，HAVING 过滤分组，行过滤应当先于分组过滤。

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING num >= 2;

```

<font color=red size=6px>分组规定</font>

- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
- NULL 的行会单独分为一组；
- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

<font color=green size=4px>子查询</font>

```mysql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);

```

<font color=green size=4px>连接</font>

```mysql
SELECT e1.name
FROM employee AS e1 INNER JOIN employee AS e2
ON e1.department = e2.department
      AND e2.name = "Jim";

```

```mysql
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;

```

<font color=green size=4px>组合查询（UNION）</font>

```mysql
SELECT col
FROM mytable
WHERE col = 1
UNION
SELECT col
FROM mytable
WHERE col =2;

```

<!--视图-->

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。

对视图的操作和对普通表的操作一样。

视图具有如下好处：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一部分数据；
- 通过只给用户访问视图的权限，保证数据的安全性；
- 更改数据格式和表示。

<!--存储过程-->

- 代码封装，保证了一定的安全性；
- 代码复用；
- 由于是预先编译，因此具有很高的性能

```mysql
delimiter //

create procedure myprocedure( out ret int )
    begin
        declare y int;
        select sum(col1)
        from mytable
        into y;
        select y*y into ret;
    end //

delimiter ;

```

==触发器==

触发器会在某个表执行以下语句时而自动执行：DELETE、INSERT、UPDATE。

触发器必须指定在语句执行之前还是之后自动执行，之前执行使用 BEFORE 关键字，之后执行使用 AFTER 关键字。BEFORE 用于数据验证和净化，AFTER 用于审计跟踪，将修改记录到另外一张表中。

```mysql
CREATE TRIGGER mytrigger AFTER INSERT ON mytable
FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果

```

### 索引

<!--B+ Tree 原理-->

==1.数据结构==

B Tree指的是Balance Tree，也就是平衡树。平衡树是一颗查找树，并且所有叶子节点位于同一层。

B+ Tree是基于B Tree和叶子节点顺序访问指针进行实现，它具有B Tree的平衡性，并且通过顺序访问指针来提高区间查询的性能。

在 B+ Tree 中，一个节点中的 key 从左到右非递减排列，如果某个指针的左右相邻 key 分别是 keyi 和 keyi+1，且不为 null，则该指针指向节点的所有 key 大于等于 keyi 且小于等于 keyi+1。

==2.操作==

进行查找操作，首先在根节点通过二分法找到叶子节点，然后在叶子节点进行二分查找，找出key所对应的data。

插入删除操作会破坏平衡树的平衡性，因此在插入删除操作后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。

==3.与红黑树的比较==

1. 更少的查找次数（B+ Tree出度非常大）
2. 利用磁盘预读特性

#### MySQL索引

1.B+ Tree索引

对树进行搜索，所以查找速度很快，由于B+ Tree的有序性，所以除了用于查找，还可以用于排序和分组。 

可以指定多个列作为索引列，多个索引组成键。

2.哈希索引

哈希索引能以O(1)时间进行查找，但是失去了有序性

InnoDB存储引擎可以进行“自适应哈希索引”

3.全文索引

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

4.空间数据索引

MyISAM 存储引擎支持空间数据索引（R-Tree），可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。

<!--索引的优点-->

- 大大减少了服务器需要扫描的数据行数。
- 帮助服务器避免进行排序和分组，以及避免创建临时表（B+Tree 索引是有序的，可以用于 ORDER BY 和 GROUP BY 操作。临时表主要是在排序和分组过程中创建，不需要排序和分组，也就不需要创建临时表）。
- 将随机 I/O 变为顺序 I/O（B+Tree 索引是有序的，会将相邻的数据都存储在一起

==索引的使用条件：==

- 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效；
- 对于中到大型的表，索引就非常有效；
- 但是对于特大型的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以使用分区技术。

### ==查询性能优化：==

1. 减少请求的数据量

   - 只返回必要的列：最好不要使用 SELECT * 语句。
   - 只返回必要的行：使用 LIMIT 语句来限制返回的数据。
   - 缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

2. 减少服务器扫描的行数

   最有效的方式是使用索引来覆盖查询。

==重构查询方式==

1. 切分大查询：一个大查询如果一次性执行的话，可能一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。
2. 分解大连接查询
   - 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其它表的查询缓存依然可以使用。
   - 分解成多个单表查询，这些单表查询的缓存结果更可能被其它查询使用到，从而减少冗余记录的查询。
   - 减少锁竞争；
   - 在应用层进行连接，可以更容易对数据库进行拆分，从而更容易做到高性能和可伸缩。
   - 查询本身效率也可能会有所提升。例如下面的例子中，使用 IN() 代替连接查询，可以让 MySQL 按照 ID 顺序进行查询，这可能比随机的连接要更高效。

<!--存储引擎-->

==InnoDB==

是 MySQL 默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+ 间隙锁（Next-Key Locking）防止幻影读。

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。

==MyISAM==

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。

提供了大量的特性，包括压缩表、空间数据索引等。

不支持事务。

不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。

可以手工或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。

如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。

==比较==

事务：InnoDB支持事务

并发：MyISAM 只支持表级锁，而InnoDB还支持行级锁。

外键：InnoDB支持外键

备份：InnoDB支持在线热备份

崩溃恢复：MyISAM崩溃后容易发生损坏，而且恢复速递慢

其他特性：MyISAM支持压缩表和空间数据索引。

### 数据类型

整型、浮点型、字符串、DATETIME、TIMESTAMP

