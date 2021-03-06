# MySql

## MySQL有哪些数据类型

| **分类**             | **类型名称** | **说明**                                                     |
| -------------------- | ------------ | ------------------------------------------------------------ |
| **整数类型**         | tinyInt      | 很小的整数(8位二进制)                                        |
|                      | smallint     | 小的整数(16位二进制)                                         |
|                      | mediumint    | 中等大小的整数(24位二进制)                                   |
|                      | int(integer) | 普通大小的整数(32位二进制)                                   |
| **小数类型**         | float        | 单精度浮点数                                                 |
|                      | double       | 双精度浮点数                                                 |
|                      | decimal(m,d) | 压缩严格的定点数                                             |
| **日期类型**         | year         | YYYY 1901~2155                                               |
|                      | time         | HH:MM:SS -838:59:59~838:59:59                                |
|                      | date         | YYYY-MM-DD 1000-01-01~9999-12-3                              |
|                      | datetime     | YYYY-MM-DD HH:MM:SS 1000-01-01 00:00:00~ 9999-12-31 23:59:59 |
|                      | timestamp    | YYYY-MM-DD HH:MM:SS 19700101 00:00:01 UTC~2038-01-19 03:14:07UTC |
| **文本、二进制类型** | CHAR(M)      | M为0~255之间的整数                                           |
|                      | VARCHAR(M)   | M为0~65535之间的整数                                         |
|                      | TINYBLOB     | 允许长度0~255字节                                            |
|                      | BLOB         | 允许长度0~65535字节                                          |
|                      | MEDIUMBLOB   | 允许长度0~167772150字节                                      |
|                      | LONGBLOB     | 允许长度0~4294967295字节                                     |
|                      | TINYTEXT     | 允许长度0~255字节                                            |
|                      | TEXT         | 允许长度0~65535字节                                          |
|                      | MEDIUMTEXT   | 允许长度0~167772150字节                                      |
|                      | LONGTEXT     | 允许长度0~4294967295字节                                     |
|                      | VARBINARY(M) | 允许长度0~M个字节的变长字节字符串                            |
|                      | BINARY(M)    | 允许长度0~M个字节的定长字节字符串                            |

## MySQL主从复制工作原理

![img](../Fig/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC85LzIxLzE2NWZiNjgzMjIyMDViMmU)

- 主库把数据的变更记录到二进制日志中（binlog）
- 从库将主库的日志复制到自己的中继日志中（relaylog）
- 从库读取中继日志的时间，将其重放到从库的数据当中

线程之间的关联：

**主**：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；

**从**：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进自己的relay log中；

**从**：sql执行线程——执行relay log中的语句；

## 读写分离的解决方案

使用**AbstractRoutingDataSource+aop+annotation**在dao层决定数据源。
本项目采用了mybatis， 可以将读写分离放在ORM层，比如mybatis可以通过mybatis plugin拦截sql语句，所有的insert/update/delete都访问master库，所有的select 都访问salve库，这样对于dao层都是透明。 plugin实现时可以通过注解或者分析语句是读写方法来选定主从库。不过这样依然有一个问题， 也就是不支持事务， 所以我们还需要重写一下DataSourceTransactionManager， 将read-only的事务扔进读库， 其余的有读有写的扔进写库。

### 读写分离对事务是否有影响？

-  对于写操作包括开启事务和提交事务或回滚要在一台机器上执行，分离开到多台master执行后数据库原生的单机事务就失效了。
-  对于事务中同时包含读写操作，与事务的隔离级别设置有关，如果事务隔离界别为read-committed或者read-uncommitted，读写分离没有影响，但是如果隔离界别为repeatable-read或者seiralizable，读写分离就会有影响，因为会在slave上看到新数据，而正在事务中的master就看不到新数据。

## 数据库优化

sql语句优化：

- 1.避免全表扫描，应考虑在 where 及 order by 涉及的列上建立索引

  而索引的建立也要遵循一定的原则：

  - 对于较频繁的作为查询条件的字段创建索引；
  - 唯一性太差的字段不适合单独创建索引；
  - 更新非常频繁的数据不适合创建索引；

- 4.尽量避免在 where 子句中使用!=或<>操作符，

- 5.遵循最左原则，在where子句中写查询条件时把索引字段放在前面

- 7.不需要获取全表数据的时候，不要查询全表数据，使用LIMIT来限制数据。

- 在进行表设计的时候，可以适度增加冗余字段，减少JOIN操作；
- 选择恰当的数据类型，如char和varchar的选择；
- 对于强调快速读取的数据，可以考虑使用MyISAM数据库引擎；
- 使用慢查询工具找出效率低下的SQL语句进行优化；
- 构建缓存，减少数据库的磁盘操作；
- 可以考虑结合使用内在型数据库，如Redis，进行混合存储。