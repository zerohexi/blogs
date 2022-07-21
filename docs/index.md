## 1，基础架构

![img](https://img2020.cnblogs.com/blog/1316091/202005/1316091-20200526004551727-1177978077.png)

### 1.1，基础分层

​	SQL层：权限判断，SQL解析，查询缓存,优化器

​	存储引擎层：负责底层数据库数据存储操作

### 1.2，逻辑模块

1. 初始化模块

   在数据库启动时的一些初始化操作，例如 环境变量初始化，各种缓存，存储引擎初始化

   > mysql --verbose --help  查看系统参数设置

2. 核心API

3. 网络交互模块

4. 服务器客户端交互协议模块

5. 用户模块

6. 访问控制模块

7. 连接管理，连接线程和线程管理

8. 转发模块

9. 缓存模块

10. 优化器

11. 表变更管理模块

12. 表维护模块

13. 系统状态管理模块

14. 表管理器

15. 日志记录模块

16. 复制模块

17. 存储引擎接口模块

### 1.3，存储引擎

| 特点         | InnoDB | MyISAM | MEMORY |
| ------------ | ------ | ------ | ------ |
| 存储限制     | 64TB   | 有     | 有     |
| 事务安全     | 支持   |        |        |
| 锁机制       | 行锁   | 表锁   | 表锁   |
| B数索引      | 支持   | 支持   | 支持   |
| 哈希索引     |        |        | 支持   |
| 全文索引     |        | 支持   |        |
| 集群索引     | 支持   |        |        |
| 数据缓存     | 支持   | 支持   | 支持   |
| 数据可压缩   |        | 支持   |        |
| 空间使用     | 高     | 低     |        |
| 内存使用     | 高     | 低     | 中等   |
| 批量插入速度 | 低     | 高     | 高     |
| 支持外键     | 支持   |        |        |

## 2，权限与安全

### 2.1，权限表

user表 ： 存储允许连接到服务器的账号信息

db表：存储用户对莫格数据库的操作权限

host表：存储某个主机对数据库的操作权限

tables_priv：对表设置操作权限

columns：对表的某一列设置 权限

procs_priv：对存储过程和存储函数设置操作权限

### 2.2，账号管理

```shell
mysql -h localhost -u root -p test  #登录本地mysql服务器
#新建普通用户
create user 'xxxxxx'@'localhost' identified by 'pwd'
#删除用户
drop user 'xxxx'@'localhost'
#root管理员修改密码
mysqladmin -u username -h localhost -p password 'newpwd'
#root 修改普通用户密码
set password for 'user'@'localhost' = password('newpwd')
```

## 3，数据备份与还原

### 3.1，数据备份

```shell
#使用mysqldump 命令备份  filename.sql 为备份文件位置
mysqldump -u user -h host -p password dbname '数据库名称' '表名' > filename.sql 
```

### 3.2，数据还原

```shell
# 根据备份文件还原
mysql -u user -p password dbname < filename.sql
# 导出数据
select * from test.tablename into outfile "file.txt"
# 导入数据
load data infile 'file.txt' into table tablename
```

## 4，高级特性

### 4.1，查询缓存

```shell
#开启缓存
set session query_cache_type=on;
#查询缓存是否开启
select @@query_cache_type;
#查询缓存是否可用
show variables like 'have_query_cache';
#查询缓存大小
select @@global.query_cache_size;
#设置缓存大小
set @@global.query_cache_size = 1000;
#设置缓存最大值
set @@global.query_chache_limit = 1024;
#查询缓存的使用情况
show status like 'Qcache%'
```

> 缓存不会缓存引用了用户自定义函数，存储过程，用户自定义变量，临时表的查询

### 4.2 合并表和分区表

合并表： 就是使用union 把多个相同结果合并成一张表

分区表主要目的是为了让某些特定的查询操作减少响应时间，同时对于应用来讲完全透明，主要分为**垂直分区**和**水平分区**

#### range分区

```mysql
partition by range(分区字段)
(
partition p1 values less than (1000),
partition p1 values less than (2000),
partition p1 values less than (3000),
partition p4 values less than maxvalue
)
```

#### list分区

```mysql
partition by list(分区字段)
(
partition p1 values in (0,1000),
partition p1 values in (1000,3000),
partition p1 values in(3000)
)
```

#### hash分区

```mysql
partition by hash(year(分区字段) #year 是一个表达式  可以是系统函数或者用户自定义
partitions 4
```

#### 线性hash分区

```mysql
partition by linear hash(year(分区字段) #year 是一个表达式  可以是系统函数或者用户自定义
partitions 4
```

#### key分区

```mysql
partition by key(分区字段) 
partitions 4
```

#### 复合分区

分区表中每个分区的再次分割

> 如果一个分区中创建了复合分区，其他分区也要由复合分区
>
> 如果创建了复合分区，每个分区中的复合分区数必有相同
>
> 同一个分区内的复合分区，名字不相同，不同分区内的复合分区名字可以相同

```mysql
# List-key 复合分区
partition by list(分区字段)
subpartition by hash(year(分区字段)),
(
partition p1 values in (1000,3000),
partition p1 values in(3000)
)
```

### 4.3，事务

本地事务

分布式事务

## 5，锁机制

> 行锁：锁定粒度到一行数据，性能开销大，发生锁冲突概率低
>
> 表锁：整张表的锁定，性能开销小，所冲突概率高
>
> 页锁：粒度介于行锁和表锁之间

MyIsAm存储引擎支持表锁，InnoDB支持行锁

### 5.1，隔离级别

**ACID：原子性，一致性，隔离性，持久性**

> 更新丢失： 两个事务更新同一行数据，但是第二个事务却中途失败退出，导致对两个修改都失效，这时系统没有执行任务锁操作，因此并发事务没有隔离
>
> 脏读： 一个事务读了某行数据，但是另一个事务已经更新了这行数据
>
> 不可重复读：一个事务对一行数据重复读取两次，可是得到了不同的结果
>
> 幻读：事务在操作过程中进行两次查询，第二次查询结果包含了第一次没有出现的数据

| 隔离级别                  | 读一致性                                 | 脏读 | 不可重复读 | 幻读 |
| ------------------------- | ---------------------------------------- | ---- | ---------- | ---- |
| 未提交读 Read uncommitted | 最低级别，只能保证不读取物理上损坏的数据 | 是   | 是         | 是   |
| 已提交读 Read committed   | 语句级                                   | 否   | 是         | 是   |
| 可重复读 Repeatable Read  | 事务级                                   | 否   | 否         | 是   |
| 可序列化 serializable     | 最高级别                                 | 否   | 否         | 否   |

### 5.2，日志详解

**binlog：**是mysql服务层产生的日志，常用来进行数据恢复、数据库复制

**redolog：**记录了数据操作在物理层面的修改，mysql中使用了大量缓存，缓存存在于内存中，修改操作时会直接修改内存，而不是立刻修改磁盘，当内存和磁盘的数据不一致时，称内存中的数据为脏页(dirty page)。为了保证数据的安全性，事务进行中时会不断的产生redo log，在事务提交时进行一次flush操作，保存到磁盘中, redo log是按照顺序写入的，磁盘的顺序读写的速度远大于随机读写。当数据库或主机失效重启时，会根据redo log进行数据的恢复，如果redo log中有事务提交，则进行事务提交修改数据。这样实现了事务的原子性、一致性和持久性。

**undolog**： 当进行数据修改时还会记录undo log，undo log用于数据的撤回操作，它记录了修改的反向操作

### 5.3，MVCC

为什么需要MVCC呢？数据库通常使用锁来实现隔离性。使用了一种读写锁的方法，读锁和读锁之间不互斥，而写锁和写锁、读锁都互斥。这样就很大提升了系统的并发能力，就是读取数据时通过一种类似快照的方式将数据保存下来，这样读锁就和写锁不冲突了，不同的事务session会看到自己特定版本的数据。

MVCC只在 READ COMMITTED 和 REPEATABLE READ 两个隔离级别下工作。

InnoDB行记录中除了刚才提到的rowid外，还有trx_id和db_roll_ptr, trx_id表示最近修改的事务的id,db_roll_ptr指向undo segment中的undo log。

新增一个事务时事务id会增加，trx_id能够表示事务开始的先后顺序。

Undo log分为Insert和Update两种，delete可以看做是一种特殊的update，即在记录上修改删除标记。

update undo log记录了数据之前的数据信息，通过这些信息可以还原到之前版本的状态。

当进行插入操作时，生成的Insert undo log在事务提交后即可删除，因为其他事务不需要这个undo log。

进行删除修改操作时，会生成对应的undo log，并将当前数据记录中的db_roll_ptr指向新的undo log。

ReadView中主要就是有个列表来存储我们系统中当前活跃着的读写事务，也就是begin了还未提交的事务。通过这个列表来判断记录的某个版本是否对当前事务可见。其中最主要的与可见性相关的属性如下：

**up_limit_id**：当前已经提交的事务号 + 1，事务号 < up_limit_id ，对于当前Read View都是可见的。理解起来就是创建Read View视图的时候，之前已经提交的事务对于该事务肯定是可见的。

**low_limit_id**：当前最大的事务号 + 1，事务号 >= low_limit_id，对于当前Read View都是不可见的。理解起来就是在创建Read View视图之后创建的事务对于该事务肯定是不可见的。

**trx_ids**：为活跃事务id列表，即Read View初始化时当前未提交的事务列表。所以当进行RR读的时候，trx_ids中的事务对于本事务是不可见的（除了自身事务，自身事务对于表的修改对于自己当然是可见的）。理解起来就是创建RV时，将当前活跃事务ID记录下来，后续即使他们提交对于本事务也是不可见的。

也就是说已提交读隔离级别下的事务在每次查询的开始都会生成一个独立的ReadView,而可重复读隔离级别则在第一次读的时候生成一个ReadView，之后的读都复用之前的ReadView。

这就是Mysql的MVCC,通过版本链，实现多版本，可并发读-写，写-读。通过ReadView生成策略的不同实现不同的隔离级别。

## 6，SQL性能优化

### 执行计划 explain

**id**：select标识符，id值越大优先级越高，越先被执行，相同id则由上至下执行

**select_type**

- ​	simple: 简单查询，不包括连接查询和子查询
- ​	primary:  主查询或者最外层的查询
- ​	union：连接查询的第二个或者后面的查询，不依赖外部查询的结果集
- ​	union result：表示union查询的结果集
- ​	dependent union:  连接查询中的第二个或者后面的select语句，取决于外面的查询
- ​	subquery：子查询中第一个select语句
- ​	dependent subquery：子查询中的第一个select，取决于外面的查询
- ​	derived：用于from 字句里有子查询的情况，mysql会递归执行这些子查询，把结果放在临时表

**table**: 显示这一行的数据是关于哪张表的，有可能看到的不是真实的表名。而是derivedX(x是数字)

**type**:  连接类型

- system:  该表是仅有一行的系统表
- const: 数据表最多只有一个匹配行，他将在查询开始时被读取，并在余下的查询优化中作为常量对待
- eq_ref: 对于每个来自前面的表的行组合，从该表中读取一行，当一个索引的所有部分都在查询中使用并且索引时unique或者primary key时，即可使用这种类型
- ref： 对于来自前面的表的任务行组合，将从该表中读取所有匹配的行，这种类型用于索引不是unique也不是primary key 的情况，或者查询中使用了索引列的左子集，即索引中左边的部分列组合
- ref_or_null：该连接类型如同ref,但是添加了mysql 可以专门搜索包含null 值的行
- index_merge：该连接类型表示使用索引合并优化方法，在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素
- uni_que_subquery: 是一个索引查找函数，可以完全替换子查询，效率高
- index_subquery：类似unique_subquery 可以替换in 子查询，但只适合子查询中非唯一索引
- range：只检索给定范围的行
- index: 只扫描索引时，
- all: 全表扫描

**possible_keys**：支持mysql 能使用哪些索引在该表中找到行

**key**： 表示查询实际使用到的索引

**key_len**: 表示mysql选择索引字段按字节计算的长度

**ref**：表示使用哪一个列或者常熟与索引一起来查询

**rows**：显示mysql在表中进行查询时必须检查的行数

### 利用profiling分析查询语句

可以获取一条查询在整个执行过程中多种资源的消耗情况，例如 内存消耗，IO，CPU等

### 合理使用索引

like ‘abc%’ 左起不匹配才能启用索引

启用多列索引，只有在查询中使用第一个字段时 索引才生效

or 前后两个条件列都是索引时才生效

order by + limit 索引优化 在排序字段建立索引

where + order by + limit 索引优化  使用where字段+排序字段建立联合索引

### mysql索引

> InnoDB是聚集索引，使用B+Tree作为索引结构，数据文件是和（主键）索引绑在一起的（表数据文件本身就是按B+Tree组织的一个索引结构），必须要有主键，通过主键索引效率很高。
>
> MyISAM是非聚集索引，也是使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针

### B+Tree

![img](https://img-blog.csdnimg.cn/20200325224440101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvbmdsb25nNjY4Mg==,size_16,color_FFFFFF,t_70)

> - 在B+Tree中，所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储key值信息，这样可以大大加大每个节点存储的key值数量，降低B+Tree的高度。

### mysql 执行顺序

![img](https://img2020.cnblogs.com/blog/1316091/202005/1316091-20200526005203493-1023561420.png)

### 聚合函数

| 函数名 | 说明                   |      |      |
| ------ | ---------------------- | ---- | ---- |
| max    | 查询指定列的最大值     |      |      |
| min    | 查询指定列的最小值     |      |      |
| count  | 统计查询结果的行数     |      |      |
| sum    | 返回指定列的总和       |      |      |
| avg    | 返回指定列的数据平均值 |      |      |

## 主从复制

![img](https://img2020.cnblogs.com/blog/1279412/202112/1279412-20211218095651536-62334417.png)

1，每个事务更新数据完成之前，主服务将这些操作的信息记录在二进制日志里面，在事务写入二进制日志完成后，主服务通知存储引擎提交事务。

2，slave上的IO进程连接上master, 并发出日志请求，master接受到来自slave的IO日志请求，通过负责复制的IO进程根据请求信息读取指定位置之后的日志信息，返回给slave的IO进程，返回信息中除了日志所包含的信息外，还包括本次返回的信息已经到master端的bin-log文件名和位置

3，slave进程收到信息后，将日志内容一次添加到slave端的relay-log文件的最末端，并将读取到master端的bin-log文件名和位置记录到master-info文件中

4，slave的SQL进程检测到relay-log中新增加内容后，会马上解析relay-log 未执行语句去执行。

### 主从复制的三种方式

1，同步复制

当主库提交事务后，binlog已经通过dump线程传到从库的中继日志，主库需要一直等待从库的提交确认，从库重放完成之后，回复一个ACK给主库，主库这才结束等待，执行后续操作，注意:如果这个时候有多个从节点，那么主库等待的时间就越久，所以需要设置一个超时等待时间

2，异步复制

主库提交完事务后，不需要等待从库提交确认，就直接执行后续操作，返回客户端； 但是这个就会造成这样一个问题:当主机提交完事务后挂了，但是这个时候binlog还没有同步到从库，如果强制切换主从的话，就会造成新的主库数据不完整

3，半同步复制

同样主库还是需要等待从库的确认后才执行后续操作，但是不同的是这次不是等待从库提交完事务后，才发一个确认通知给主库，而是当从库将binlog写到relaylog后，就会给主库发送确认通知，这个不仅缩短了等待时间而且还维护了数据的安全性

只有当收到至少一个从库返回的relay log写入确认后，才提交事务，也就是说提交事务在收到确认之后

### 主从复制三种模式

**STATEMENT模式（SBR）**

- 每一条会修改数据的sql语句会记录到binlog中。优点是并不需要记录每一条 sql语句和每一行的数据变化，减少了binlog日志量，节约IO，提高性能。
- 缺点是在某些情况下会导致 master-slave中的数据不一致(如sleep()函数， last_insert_id()，以及user-defined functions(udf)等会出现问题)

**ROW模式（RBR）**

- 不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了，修改成什么样了。而且不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题。
- 缺点是会产生大量的日志，尤其是altertable的时候会让日志暴涨。

**MIXED模式（MBR）**

- 以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。

## Mysql高可用架构

1，Mysql+DRBD+HA

2，Lvs+Keepalived+Mysql 单点写入 主主同步方案

3，MMM 高可用Mysql方案

