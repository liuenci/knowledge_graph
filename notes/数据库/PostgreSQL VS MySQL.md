#### 性能
- PostgreSQL 适合读写速度要求很高的大型系统中使用。
- MySQL 主要用于 Web 应用程序，该 Web 应用程序仅需要数据库。

​

#### ACID

- PostgreSQL 从头到尾都遵循 ACID 原则，并确保满足要求。
- MySQL 只有在使用 InnoDB 和 NDB 存储引擎时才有 ACID 原则。

​

#### SQL 兼容性

- PostgreSQL 兼容大部分 SQL 语法，语法结构更加完善。
- MySQL 正在努力达到 SQL 标准。

​

#### 物化视图

- PostgreSQL 支持物化视图。
- MySQL 不支持物化视图。

​

#### 数据备份

- PostgreSQL 支持主备复制，并且还可以通过实现第三方扩展来处理其他类型的复制。
- MySQL 支持主备复制，其中每个节点都是主节点，并且有权更新数据。

​

#### 数据性能

- MySQL 数据量超过 2000w 之后性能骤降。
- PostgreSQL 最大表大小 32 TB，最大行大小 1.6 TB，最大字段大小 1 GB，每个表的最大行数无限制

​

#### 参考地址

- [https://www.zhihu.com/question/20010554](https://www.zhihu.com/question/20010554)
