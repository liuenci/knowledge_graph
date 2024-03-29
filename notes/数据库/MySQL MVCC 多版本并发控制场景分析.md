# 数据库并发场景
在介绍 MVCC 多版本并发控制之前，先要讲一讲数据库的几种并发场景，它们分别是

- 读读：不存在任何问题，无需并发版本控制。
- 读写：存在线程安全问题，可能出现脏读、不可重复读、幻读等场景。
- 写写：存在线程安全问题，可能出现第二类丢失更新问题。



# MVCC 的应用场景
针对以上三种数据库的并发场景，MVCC 解决的问题主要是读写产生的线程安全问题。他是一种用来解决读写并发冲突的无锁并发控制。下面来详细进行讲解：
​

MySQL 的多版本并发控制是通过 undolog 来处理的。
​

## 当前读与快照读
### 当前读
像 select lock in share mode（共享锁），select for update，update，insert，delete （排他锁）这些操作都是一种当前读，为什么叫做当前读？就是它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。
### 快照读
不加锁的 select 查询语句就是快照读，即不加锁的非阻塞读；快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读；之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于多版本并发控制，即 MVCC，可以认为 MVCC 是行锁的一个变种，但它在很多情况下，避免了加锁操作，降低了开销；既然是基于多版本，即快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本。
​

具体的底层实现：在非串行化的隔离级别下，MySQL 会对每一个修改保存一个版本号，这个版本号是单向增长的时间戳，这个版本和事务的时间戳关联，读操作只读该事务开始前的数据库的快照。
​

### 两种读类型总结
对于当前读来说，在读的时候加锁，是一种悲观锁的实现，而快照读，会带一个版本号从 undolog 里面读，读取的可能不是最新的版本。
