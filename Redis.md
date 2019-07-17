## ==Redis学习笔记==

### 一、概述

Redis是速度非常快的非关系型内存键值数据库，可以存储字符串、列表、集合、散列表、有序集合。Redis支持将内存中的数据持久化到硬盘中。

### 二、数据类型

| 数据类型 | 可以存储的             | 操作                                                         |
| :------- | :--------------------- | :----------------------------------------------------------- |
| STRING   | 字符串、整数、或浮点数 | 对整个字符串或者字符串的其中一部分执行操作<br/>对整数和浮点数执行自增或者自减操作 |
| LIST     | 列表                   | 从两端压入或者弹出元素 <br/>对单个或者多个元素进行修剪，<br/>只保留一个范围内的元素 |
| SET      | 无序集合               | 添加、获取、移除单个元素<br/>检查一个元素是否存在于集合中<br/>计算交集、并集、差集<br/>从集合里面随机获取元素 |
| HASH     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对<br/>获取所有键值对<br/>检查某个键是否存在 |
| ZSET     | 有序集合               | 添加、获取、删除元素<br/>根据分值范围或者成员来获取元素<br/>计算一个键的排名 |

1. STRING

   ```
   set hello world
   get hello
   del hello
   get hello
   ```

2. LIST

   ```
   rpush list-key item
   rpush list_key item1
   
   
   lindex list-key 1
   lrange list-key 0 -1
   ```

   

3. SET

```
sadd set-key item
sadd set-key item2

smembers set-key
sismember set-key item2
srem set-key item2

```

4. HASH

```
hset hash-key sub-key1 value1
hset hash-key sub-key2 value2

hgetall hash-key
hdel hash-key
```

5. ZSET

   ```
   zadd zset-key 728 member1
   zrange zset-key 0 -1 withscores
   
   ```

   
