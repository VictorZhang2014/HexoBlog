---
title: 你不得不知的SQLite
date: 2017-10-20 19:08:31
tags: SQLite, 优化
categories: iOS
---

很多时候，随着业务越来越好，系统所牵涉到的数据数量也是越来越多、越来越大，这时，许多系统的瓶颈就在数据的存储上了，所以我们就不得不考虑对数据库进行优化了。

### 从服务器的角度来讲，
伸缩（scale）就是一种方式，它分为两种方法：
- 1.`向上伸缩`（`scale up`），它的意思是通过使用更好的硬件来提高系统的性能参数。
- 2.`向外伸缩`（`scale out`），它的意思是通过额外的硬件（如：服务器）来达到同样的效果。

### 读写分离
`读写分离`的意思是，一个`主数据库（master）`负责`写`数据，一个`从服务器（slave）`的数据库负责读数据；master负责将写操作的数据同步到各个`节点`，读写分离来提供系统性能的有点：
- 物理服务器增加，机器处理能力提升，那硬件换性能
- 主从只负责各自的写和读，极大程度缓解了X锁和S锁争用

它们之间最大的问题就是同步，因为当一条数据写到主服务器后，从服务器如何同步？
这几个主流数据库都是良好支持主从同步的：
- MySQL Replication
- MongoDB Master Slave Replication
- Oracle Replication
- SQL Server Replication


### 数据分片（`data sharding`）
是将整体数据分开存放，如：服务器端，可以是将数据存放在多台服务器之间；客户端，可以是将数据库/数据表分开存放，以满足大数据量的需要。
但是值得注意的是：本来是一张表或一个数据库的数据被分开存放了，那么对于`select`/`insert`/`update`/`delete`应该如何处理了？

### 策略如下：
- 1.根据取模，例如：先要对计算的key进行哈希计算，然后再对分开的表的数量或者设备进行取模运行，得到的结果是几，那么这条记录就放在编号为几的数据分区中
- 2.根据时间范围，例如：前50万条数据放在第一个分区中，第二个50万条数据放在第二个分区，依次类推
- 3.基于索引表：例如：新建立一个索引表，然后根据ID先去一个表内找到它所在的分区，然后再去目标分区进行查找

数据分区虽然好处多多，尤其是对系统的性能（performance）和伸缩性(scalability)，但是同时也带来是开发的复杂度。

<br/>
## 那SQLite如何优化了？
[FMDB](https://github.com/ccgus/fmdb)是基于SQLite封装的。早期的[FMDB](https://github.com/ccgus/fmdb)是不支持并发的，但是后来的版本中，FMDB已经解决了在并发问题。
`SQLite共识`：
- `四种锁`：
    - `共享锁`（shared lock）
    - `预留锁`（reserved lock）
    - `未决锁`（pending lock）
    - `排他锁`（exclusive lock）
- `SQLite读操作`（如：`select`），可以并发的读取数据库，如果有一个读存在，那么就不允许写
- `SQLite写操作`（如：`insert/update/delete`），
    - 1.它首先会申请一个预留锁（reserved lock），在启用预留锁后，如果已存在的读操作可以继续，新的读请求也可以有；
    - 2.然后，它会把需要更新的数据写到缓冲区中；
    - 3.需要写到缓冲区的更新写完以后，就需要将更新刷到硬盘DB了，但是此时它会申请未决锁（pending lock），申请了该锁后，就不能再有新的共享锁被申请了，也就是阻止了新的读操作。但是已经存在的读操作还是可以继续读的。然后它就等待读操作全部完毕后，它就会申请排他锁（exclusive lock），此时不能再有其他的锁存在了，然后就进行commit操作，最后，将缓冲区的数据写到DB中。
- `事务`，就是一组SQL语句操作让它们顺序执行，要么都成功了，要么只要有一个错误，全部操作回滚；它们的特点如下，一般都是四个命令来控制，`begin transaction`，`commit`或者`end transaction`，`rollback`：
    - `原子性`（Atomicity）：确保工作单位内的所有操作都成功完成，否则，事务会在出现故障的地方终止，并且回滚到以前的状态
    - `一致性`（Consistency）：确保数据库在成功提交的事务上正确的改变状态
    - `隔离性`（Isolation）：使事务操作相互独立和透明
    - `持久性`（Durability）：确保已提交事务的结果或效果在系统发生故障的情况下仍然存在

<br/>
## SQLite优化要点
- 1.设计数据库时遵循三范式
    - `第一范式`：每个字段（每一列）都不可再分，意思就是说：一列就是一个数据类型并且存储一种类型的值
    - `第二范式`：满足第一范式的情况下，我们还需要保证一条数据在表中有一个主键（唯一标识），不一定是id，也可以是身份证号（只要表示唯一性就行）
    - `第三范式`：满足第二范式的情况下，我们要确保数据表中的每一列数据都和主键有直接关系，而不是简介关系
- 2.数据量比较大的话，可以对数据表（data table）进行`分片`（sharding）
- 3.从业务上可以分离，且数据量也较大的话，也可以`分库`（sub-database）
- 4.查询数据太频繁的地方，不建议使用`连表`查询

对数据表进行分片
对数据库也分子库
1. https://stackoverflow.com/questions/15778716/sqlite-insert-speed-slows-as-number-of-records-increases-due-to-an-index/17110004#17110004
2. https://stackoverflow.com/questions/128919/extreme-sharding-one-sqlite-database-per-user

<br/>
** 读写死锁案例 **
所谓死锁，就是两个互相等待对方释放资源，在SQLite这里一样存在该问题。如：
伪码
```
读操作A                                                写操作B
sqlite> select * from table;                        sqlite> insert into table values('a','b');
sqlite> insert into table values('a','b');          sqlite> commit;
//SQL error: database is locked                      //SQL error: database is locked
```
有`读操作A`，`写操作B`；
- 1.`写操作B`申请了预留锁；然后`读操作A`申请了共享锁（有预留锁时，是允许申请读操作（共享锁）的）；
- 2.然后`读操作A`又同时想进行`写操作`（未释放共享锁的情况），此时申请了预留锁（因为已经有预留锁存在了）失败；
- 3.`写操作B`写完缓存，想commit时，申请了未决锁，但是无法从未决锁提升到排他锁（因为有共享锁存在）；
- 4.此时发生死锁，因为`读操作A`和`写操作B`都在互相等待对方释放锁。
- 5.在四种锁中，保留锁和共享锁是可以同时存在的，而且出发这些锁的机制都取决于你的数据的操作

**解决方法**，就是采用`事务`（transaction）
伪码
```
读操作A                                                写操作B
sqlite> begin transaction;                          sqlite> begin transaction;
sqlite> select * from table;                        sqlite> insert into table values('a','b');
sqlite> insert into table values('a','b');          sqlite> commit;
sqlite> commit;                                     sqlite> end transaction;
sqlite> end transaction;
```

事务的另一个`优点`就是：可以`大批量`插入数据，我们都知道正常情况下，sql语句每执行一条，就会打开和关闭一次数据库，这样的速度显然是很慢的，那么如果我们需要大批量插入数据，而且一次打开和关闭数据了？那就是使用事务，因为所有的`insert`语句会在`commit`之前被缓存到内存中。

<br/>
<br/>

## 关系型数据库
[WCDB 由微信开发团队开源](https://github.com/Tencent/wcdb)
[SQLCiper 加密](https://github.com/sqlcipher/sqlcipher)
[FMDB 包含SQLCiper，多线程安全的](https://github.com/ccgus/fmdb)
[SQLite中文教程](http://www.runoob.com/sqlite/sqlite-tutorial.html)
[SQLite Official](https://www.sqlite.org/index.html)

## Key-Value数据库
代表有<a href="https://realm.io/" target="_blank">Realm</a>，<a href="https://github.com/google/leveldb" target="_blank">LevelDB</a>，<a href="https://github.com/facebook/rocksdb" target="_blank">RocksDB</a>等
对于开发者而言，key-value的实现直接易懂，可以像使用NSDictionary一样使用Realm。并且ORM彻底，省去了拼装Object的过程。但其对代码侵入性很强，Realm要求类继承RLMObject的基类。这对于单继承的ObjC，意味着不能再继承其他自定义的子类。同时，key-value数据库对较为复杂的查询场景也比较无力。


