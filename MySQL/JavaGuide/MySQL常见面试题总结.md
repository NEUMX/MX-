# MySQL常见面试题总结

# MySQL常见面试题总结
## MySQL 基础
### 什么是关系型数据库？
顾名思义，关系型数据库（RDB，Relational Database）就是一种建立在关系模型的基础上的数据库。关系模型表明了数据库中所存储的数据之间的联系（一对一、一对多、多对多）。

关系型数据库中，我们的数据都被存放在了各种表中（比如用户表），表中的每一行就存放着一条数据（比如一个用户的信息）。

![1732497608875-4409b3c3-92b1-4550-bf03-42b8ee7d0d73.png](./img/QHmSnXBq8gIJZRqx/1732497608875-4409b3c3-92b1-4550-bf03-42b8ee7d0d73-810518.png)

关系型数据库表关系

大部分关系型数据库都使用 SQL 来操作数据库中的数据。并且，大部分关系型数据库都支持事务的四大特性(ACID)。

**有哪些常见的关系型数据库呢？**

MySQL、PostgreSQL、Oracle、SQL Server、SQLite（微信本地的聊天记录的存储就是用的 SQLite） ……。

### 什么是 SQL？
SQL 是一种结构化查询语言(Structured Query Language)，专门用来与数据库打交道，目的是提供一种从数据库中读写数据的简单有效的方法。

几乎所有的主流关系数据库都支持 SQL ，适用性非常强。并且，一些非关系型数据库也兼容 SQL 或者使用的是类似于 SQL 的查询语言。

SQL 可以帮助我们：

+ 新建数据库、数据表、字段；
+ 在数据库中增加，删除，修改，查询数据；
+ 新建视图、函数、存储过程；
+ 对数据库中的数据进行简单的数据分析；
+ 搭配 Hive，Spark SQL 做大数据；
+ 搭配 SQLFlow 做机器学习；
+ ……

### 什么是 MySQL？
![1732497608940-43e86c82-525b-400a-b912-3f1013514d19.png](./img/QHmSnXBq8gIJZRqx/1732497608940-43e86c82-525b-400a-b912-3f1013514d19-701012.png)

**MySQL 是一种关系型数据库，主要用于持久化存储我们的系统中的一些数据比如用户信息。**

由于 MySQL 是开源免费并且比较成熟的数据库，因此，MySQL 被大量使用在各种系统中。任何人都可以在 GPL(General Public License) 的许可下下载并根据个性化的需要对其进行修改。MySQL 的默认端口号是**3306**。

### MySQL 有什么优点？
这个问题本质上是在问 MySQL 如此流行的原因。

MySQL 主要具有下面这些优点：

1. 成熟稳定，功能完善。
2. 开源免费。
3. 文档丰富，既有详细的官方文档，又有非常多优质文章可供参考学习。
4. 开箱即用，操作简单，维护成本低。
5. 兼容性好，支持常见的操作系统，支持多种开发语言。
6. 社区活跃，生态完善。
7. 事务支持优秀， InnoDB 存储引擎默认使用 REPEATABLE-READ 并不会有任何性能损失，并且，InnoDB 实现的 REPEATABLE-READ 隔离级别其实是可以解决幻读问题发生的。
8. 支持分库分表、读写分离、高可用。

## MySQL 字段类型
MySQL 字段类型可以简单分为三大类：

+ **数值类型**：整型（TINYINT、SMALLINT、MEDIUMINT、INT 和 BIGINT）、浮点型（FLOAT 和 DOUBLE）、定点型（DECIMAL）
+ **字符串类型**：CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB 等，最常用的是 CHAR 和 VARCHAR。
+ **日期时间类型**：YEAR、TIME、DATE、DATETIME 和 TIMESTAMP 等。

下面这张图不是我画的，忘记是从哪里保存下来的了，总结的还蛮不错的。

![1732497609010-2ed47e87-42e1-4145-b5c7-06b0ecdb14e1.png](./img/QHmSnXBq8gIJZRqx/1732497609010-2ed47e87-42e1-4145-b5c7-06b0ecdb14e1-238957.png)

MySQL 常见字段类型总结

MySQL 字段类型比较多，我这里会挑选一些日常开发使用很频繁且面试常问的字段类型，以面试问题的形式来详细介绍。如无特殊说明，针对的都是 InnoDB 存储引擎。

另外，推荐阅读一下《高性能 MySQL(第三版)》的第四章，有详细介绍 MySQL 字段类型优化。

### 整数类型的 UNSIGNED 属性有什么用？
MySQL 中的整数类型可以使用可选的 **<font style="background-color:#d9dffc;">UNSIGNED 属性来表示不允许负值的无符号整数</font>**。使用 UNSIGNED 属性可以将正整数的上限提高一倍，因为它不需要存储负数值。

例如， TINYINT UNSIGNED 类型的取值范围是 0 ~ 255，而普通的 TINYINT 类型的值范围是 -128 ~ 127。INT UNSIGNED 类型的取值范围是 0 ~ 4,294,967,295，而普通的 INT 类型的值范围是 -2,147,483,648 ~ 2,147,483,647。

**<font style="background-color:#d9dffc;">对于从 0 开始递增的 ID 列，使用 UNSIGNED 属性</font>****可以非常适合，因为不允许负值并且可以拥有更大的上限范围，****<font style="background-color:#d9dffc;">提供了更多的 ID 值可用</font>**。

### CHAR 和 VARCHAR 的区别是什么？
CHAR 和 VARCHAR 都是关系型数据库中用于存储字符串数据的数据类型，但它们之间有以下主要区别：

1. **固定长度 vs 可变长度**： 
    - `CHAR` 是固定长度的字符数据类型。当你声明一个 CHAR(n) 字段时，无论实际存入的字符长度是多少，数据库都会为每个记录分配 n 个字节的空间来存储该字段，并在不足部分填充空格（对于某些数据库系统可能是其他字符）以确保总长度一致。
    - `VARCHAR` 是可变长度的字符数据类型。它仅占用足够的空间来存储实际插入的数据，然后加上额外的一到两个字节（具体取决于数据库引擎）来存储长度信息。
2. **存储效率和空间利用率**： 
    - CHAR 类型对于所有记录来说，其存储开销相对固定且可能造成一定的空间浪费，尤其是当大多数数据的实际长度远小于定义的最大长度时。
    - VARCHAR 类型则更加灵活，能更好地利用存储空间，特别是当字符串长度变化较大时。
3. **性能影响**： 
    - 在查询和索引操作上，由于 CHAR 类型的长度固定，在处理速度上可能稍微优于 VARCHAR，因为数据库可以预先知道每个值的确切位置和大小。
    - 对于频繁更新的场景，如果每次更新的数据长度接近或等于CHAR定义的长度，则 CHAR 可能具有更高的写入效率，因为无需改变已有数据的位置。
4. **最大长度限制**： 
    - 不同数据库系统对 CHAR 和 VARCHAR 的最大长度有不同的限制，例如 MySQL 中 CHAR 最大长度一般为 255 或 8000 字符，而 VARCHAR 的最大长度通常更大一些。
5. **排序与比较**： 
    - 对于 CHAR 类型，由于末尾填充了空格，因此在进行排序和比较时会考虑这些空格，可能导致意外的结果，除非明确使用 TRIM 函数去除空格。
    - VARCHAR 类型则按照实际字符内容进行排序和比较。

综上所述，选择 CHAR 还是 VARCHAR 主要取决于你的具体需求，包括存储的数据特征、对存储空间的有效利用以及对性能的需求等。如果数据长度几乎恒定并且不需要担心空间浪费，同时需要最佳的读取性能，可以选择 CHAR；如果数据长度变化较大或者希望节省空间，VARCHAR 通常是更好的选择。

### VARCHAR(100)和 VARCHAR(10)的区别是什么？
VARCHAR(100) 和 VARCHAR(10) 都是MySQL数据库中的变长字符串数据类型，它们的主要区别在于存储容量和对所存储数据长度的要求：

1. **存储容量**： 
    - `VARCHAR(10)` 表示该列可以存储最多10个字符（在UTF-8编码下可能是30字节，因为一个汉字通常占用3个字节）。
    - `VARCHAR(100)` 则允许存储最多100个字符，相应的，在UTF-8编码下，最大字节数为300字节。
2. **业务适应性**： 
    - 如果你预计存储的字符串长度不会超过10个字符，使用`VARCHAR(10)`可以节省存储空间。
    - 如果未来可能需要存储更长的字符串，则选择`VARCHAR(100)`能够提供更大的灵活性和扩展性，无需修改表结构就能存储更长的数据。
3. **内存和性能影响**： 
    - 虽然对于实际存储的相同长度字符串，两者在磁盘上占用的空间是一样的，但在内存中处理时，MySQL可能会为查询操作分配固定大小的内存块来处理行数据。因此，即使实际存储的字符串较短，`VARCHAR(100)`字段可能消耗更多的内存，特别是在涉及排序或临时表等操作时。
4. **索引效率**： 
    - 对于非常短的字段，创建索引时，`VARCHAR(10)`相对于`VARCHAR(100)`可能更为高效，因为它占用的索引空间更小。

综上所述，根据应用的实际需求合理选择字段长度至关重要，既要考虑当前数据需求，也要考虑未来的扩展性和性能优化的可能性。

### DECIMAL 和 FLOAT/DOUBLE 的区别是什么？
DECIMAL 和 FLOAT 的区别是：**DECIMAL 是定点数，FLOAT/DOUBLE 是浮点数。DECIMAL 可以存储精确的小数值，FLOAT/DOUBLE 只能存储近似的小数值。**

DECIMAL 用于存储具有精度要求的小数，例如与货币相关的数据，可以避免浮点数带来的精度损失。

在 Java 中，MySQL 的 DECIMAL 类型对应的是 Java 类java.math.BigDecimal。

### 为什么不推荐使用 TEXT 和 BLOB？
TEXT 类型类似于 CHAR（0-255 字节）和 VARCHAR（0-65,535 字节），但可以存储更长的字符串，即长文本数据，例如博客内容。

| 类型 | 可存储大小 | 用途 |
| :--- | :--- | :--- |
| TINYTEXT | 0-255 字节 | 一般文本字符串 |
| TEXT | 0-65,535 字节 | 长文本字符串 |
| MEDIUMTEXT | 0-16,772,150 字节 | 较大文本数据 |
| LONGTEXT | 0-4,294,967,295 字节 | 极大文本数据 |


BLOB 类型主要用于存储二进制大对象，例如图片、音视频等文件。

| 类型 | 可存储大小 | 用途 |
| :--- | :--- | :--- |
| TINYBLOB | 0-255 字节 | 短文本二进制字符串 |
| BLOB | 0-65KB | 二进制字符串 |
| MEDIUMBLOB | 0-16MB | 二进制形式的长文本数据 |
| LONGBLOB | 0-4GB | 二进制形式的极大文本数据 |


在日常开发中，很少使用 TEXT 类型，但偶尔会用到，而 BLOB 类型则基本不常用。如果预期长度范围可以通过 VARCHAR 来满足，建议避免使用 TEXT。

数据库规范通常不推荐使用 BLOB 和 TEXT 类型，这两种类型具有一些缺点和限制，例如：

+ 不能有默认值。
+ 在使用临时表时无法使用内存临时表，只能在磁盘上创建临时表（《高性能 MySQL》书中有提到）。
+ 检索效率较低。
+ 不能直接创建索引，需要指定前缀长度。
+ 可能会消耗大量的网络和 IO 带宽。
+ 可能导致表上的 DML 操作变慢。
+ ……

### DATETIME 和 TIMESTAMP 的区别是什么？
DATETIME 类型没有时区信息，TIMESTAMP 和时区有关。

TIMESTAMP 只需要使用 4 个字节的存储空间，但是 DATETIME 需要耗费 8 个字节的存储空间。但是，这样同样造成了一个问题，Timestamp 表示的时间范围更小。

+ DATETIME：1000-01-01 00:00:00 ~ 9999-12-31 23:59:59
+ Timestamp：1970-01-01 00:00:01 ~ 2037-12-31 23:59:59

关于两者的详细对比，请参考我写的[MySQL 时间类型数据存储建议](https://javaguide.cn/database/mysql/some-thoughts-on-database-storage-time.html)。

### NULL 和 '' 的区别是什么？
NULL跟''(空字符串)是两个完全不一样的值，区别如下：

+ NULL代表一个不确定的值,就算是两个NULL,它俩也不一定相等。例如，SELECT NULL=NULL的结果为 false，但是在我们使用DISTINCT,GROUP BY,ORDER BY时,NULL又被认为是相等的。
+ ''的长度是 0，是不占用空间的，而NULL是需要占用空间的。
+ NULL会影响聚合函数的结果。例如，SUM、AVG、MIN、MAX等聚合函数会忽略NULL值。COUNT的处理方式取决于参数的类型。如果参数是*(COUNT(*))，则会统计所有的记录数，包括NULL值；如果参数是某个字段名(COUNT(列名))，则会忽略NULL值，只统计非空值的个数。
+ 查询NULL值时，必须使用IS NULL或IS NOT NULLl来判断，而不能使用 =、!=、 <、> 之类的比较运算符。而''是可以使用这些比较运算符的。

看了上面的介绍之后，相信你对另外一个高频面试题：“为什么 MySQL 不建议使用NULL作为列默认值？”也有了答案。

### Boolean 类型如何表示？
MySQL 中没有专门的布尔类型，而是用 TINYINT(1) 类型来表示布尔值。TINYINT(1) 类型可以存储 0 或 1，分别对应 false 或 true。

## MySQL 基础架构
建议配合[SQL 语句在 MySQL 中的执行过程](https://javaguide.cn/database/mysql/how-sql-executed-in-mysql.html)这篇文章来理解 MySQL 基础架构。另外，“一个 SQL 语句在 MySQL 中的执行流程”也是面试中比较常问的一个问题。

下图是 MySQL 的一个简要架构图，从下图你可以很清晰的看到客户端的一条 SQL 语句在 MySQL 内部是如何执行的。

![1732497609237-11db5a6b-329e-400c-871e-7f8ddf4cb4a8.png](./img/QHmSnXBq8gIJZRqx/1732497609237-11db5a6b-329e-400c-871e-7f8ddf4cb4a8-697165.png)

从上图可以看出， MySQL 主要由下面几部分构成：

+ **连接器：**身份认证和权限相关(登录 MySQL 的时候)。
+ **查询缓存：**执行查询语句的时候，会先查询缓存（MySQL 8.0 版本后移除，因为这个功能不太实用）。
+ **分析器：**没有命中缓存的话，SQL 语句就会经过分析器，分析器说白了就是要先看你的 SQL 语句要干嘛，再检查你的 SQL 语句语法是否正确。
+ **优化器：**按照 MySQL 认为最优的方案去执行。
+ **执行器：**执行语句，然后从存储引擎返回数据。 执行语句之前会先判断是否有权限，如果没有权限的话，就会报错。
+ **插件式存储引擎**：主要负责数据的存储和读取，采用的是插件式架构，支持 InnoDB、MyISAM、Memory 等多种存储引擎。

## MySQL 存储引擎
MySQL 核心在于存储引擎，想要深入学习 MySQL，必定要深入研究 MySQL 存储引擎。

### MySQL 支持哪些存储引擎？默认使用哪个？
MySQL 支持多种存储引擎，你可以通过SHOW ENGINES命令来查看 MySQL 支持的所有存储引擎。

![1732497609353-a94d7d1c-3942-4530-87ed-1ed28f37c9fd.png](./img/QHmSnXBq8gIJZRqx/1732497609353-a94d7d1c-3942-4530-87ed-1ed28f37c9fd-116047.png)

查看 MySQL 提供的所有存储引擎

从上图我们可以查看出， MySQL 当前默认的存储引擎是 InnoDB。并且，所有的存储引擎中只有 InnoDB 是事务性存储引擎，也就是说只有 InnoDB 支持事务。

我这里使用的 MySQL 版本是 8.x，不同的 MySQL 版本之间可能会有差别。

MySQL 5.5.5 之前，MyISAM 是 MySQL 的默认存储引擎。5.5.5 版本之后，InnoDB 是 MySQL 的默认存储引擎。

你可以通过SELECT VERSION()命令查看你的 MySQL 版本。

```plsql
mysql> SELECT VERSION();
+-----------+
| VERSION() |
+-----------+
| 8.0.27    |
+-----------+
1 row in set (0.00 sec)
```

你也可以通过SHOW VARIABLES LIKE '%storage_engine%'命令直接查看 MySQL 当前默认的存储引擎。

```plsql
mysql> SHOW VARIABLES  LIKE '%storage_engine%';
+---------------------------------+-----------+
| Variable_name                   | Value     |
+---------------------------------+-----------+
| default_storage_engine          | InnoDB    |
| default_tmp_storage_engine      | InnoDB    |
| disabled_storage_engines        |           |
| internal_tmp_mem_storage_engine | TempTable |
+---------------------------------+-----------+
4 rows in set (0.00 sec)
```

如果你想要深入了解每个存储引擎以及它们之间的区别，推荐你去阅读以下 MySQL 官方文档对应的介绍(面试不会问这么细，了解即可)：

+ InnoDB 存储引擎详细介绍：[https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)[](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)。
+ 其他存储引擎详细介绍：[https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)[](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)。

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497609497-6303d4c1-18f5-4608-b48e-a90fc328d56f.png)

### MySQL 存储引擎架构了解吗？
MySQL 存储引擎采用的是**插件式架构**，支持多种存储引擎，我们甚至可以为不同的数据库表设置不同的存储引擎以适应不同场景的需要。**存储引擎是基于表的，而不是数据库。**

并且，你还可以根据 MySQL 定义的存储引擎实现标准接口来编写一个属于自己的存储引擎。这些非官方提供的存储引擎可以称为第三方存储引擎，区别于官方存储引擎。像目前最常用的 InnoDB 其实刚开始就是一个第三方存储引擎，后面由于过于优秀，其被 Oracle 直接收购了。

MySQL 官方文档也有介绍到如何编写一个自定义存储引擎，地址：[https://dev.mysql.com/doc/internals/en/custom-engine.html](https://dev.mysql.com/doc/internals/en/custom-engine.html)[](https://dev.mysql.com/doc/internals/en/custom-engine.html)。

### MyISAM 和 InnoDB 有什么区别？
MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎，可谓是风光一时。

虽然，MyISAM 的性能还行，各种特性也还不错（比如全文索引、压缩、空间函数等）。但是，MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。

MySQL 5.5 版本之后，InnoDB 是 MySQL 的默认存储引擎。

言归正传！咱们下面还是来简单对比一下两者：

**1.是否支持行级锁**

MyISAM 只有表级锁(table-level locking)，而 InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。

也就说，MyISAM 一锁就是锁住了整张表，这在并发写的情况下是多么滴憨憨啊！这也是为什么 InnoDB 在并发写的时候，性能更牛皮了！

**2.是否支持事务**

MyISAM 不提供事务支持。

InnoDB 提供事务支持，实现了 SQL 标准定义了四个隔离级别，具有提交(commit)和回滚(rollback)事务的能力。并且，InnoDB 默认使用的 REPEATABLE-READ（可重读）隔离级别是可以解决幻读问题发生的（基于 MVCC 和 Next-Key Lock）。

关于 MySQL 事务的详细介绍[MySQL事务隔离级别详解](https://www.yuque.com/vip6688/neho4x/scqtpadh3g5871tg)

**3.是否支持外键**

MyISAM 不支持，而 InnoDB 支持。

外键对于维护数据一致性非常有帮助，但是对性能有一定的损耗。因此，通常情况下，我们是不建议在实际生产项目中使用外键的，在业务代码中进行约束即可！

阿里的《Java 开发手册》也是明确规定禁止使用外键的。

![1732497609565-09da07b3-9bea-4bb6-8176-39f249ada90c.png](./img/QHmSnXBq8gIJZRqx/1732497609565-09da07b3-9bea-4bb6-8176-39f249ada90c-956121.png)

不过，在代码中进行约束的话，对程序员的能力要求更高，具体是否要采用外键还是要根据你的项目实际情况而定。

总结：一般我们也是不建议在数据库层面使用外键的，应用层面可以解决。不过，这样会对数据的一致性造成威胁。具体要不要使用外键还是要根据你的项目来决定。

**4.是否支持数据库异常崩溃后的安全恢复**

MyISAM 不支持，而 InnoDB 支持。

使用 InnoDB 的数据库在异常崩溃后，数据库重新启动的时候会保证数据库恢复到崩溃前的状态。这个恢复的过程依赖于redo log。

**5.是否支持 MVCC**

MyISAM 不支持，而 InnoDB 支持。

讲真，这个对比有点废话，毕竟 MyISAM 连行级锁都不支持。MVCC 可以看作是行级锁的一个升级，可以有效减少加锁操作，提高性能。

**6.索引实现不一样。**

虽然 MyISAM 引擎和 InnoDB 引擎都是使用 B+Tree 作为索引结构，但是两者的实现方式不太一样。

InnoDB 引擎中，其数据文件本身就是索引文件。相比 MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按 B+Tree 组织的一个索引结构，树的叶节点 data 域保存了完整的数据记录。

详细区别。[MySQL索引详解](https://www.yuque.com/vip6688/neho4x/cpwtqmfh71drcvbl)

**7.性能有差别。**

InnoDB 的性能比 MyISAM 更强大，不管是在读写混合模式下还是只读模式下，随着 CPU 核数的增加，InnoDB 的读写能力呈线性增长。MyISAM 因为读写不能并发，它的处理能力跟核数没关系。

![1732497609640-aadd821f-ee55-4d2a-9b69-02c5a9161959.png](./img/QHmSnXBq8gIJZRqx/1732497609640-aadd821f-ee55-4d2a-9b69-02c5a9161959-631093.png)

InnoDB 和 MyISAM 性能对比

**总结**：

+ InnoDB 支持行级别的锁粒度，MyISAM 不支持，只支持表级别的锁粒度。
+ MyISAM 不提供事务支持。InnoDB 提供事务支持，实现了 SQL 标准定义了四个隔离级别。
+ MyISAM 不支持外键，而 InnoDB 支持。
+ MyISAM 不支持 MVCC，而 InnoDB 支持。
+ 虽然 MyISAM 引擎和 InnoDB 引擎都是使用 B+Tree 作为索引结构，但是两者的实现方式不太一样。
+ MyISAM 不支持数据库异常崩溃后的安全恢复，而 InnoDB 支持。
+ InnoDB 的性能比 MyISAM 更强大。

最后，再分享一张图片给你，这张图片详细对比了常见的几种 MySQL 存储引擎。

![1732497609744-c0d36774-c538-4f83-8093-9584d82cb2f6.png](./img/QHmSnXBq8gIJZRqx/1732497609744-c0d36774-c538-4f83-8093-9584d82cb2f6-616201.png)

常见的几种 MySQL 存储引擎对比

### MyISAM 和 InnoDB 如何选择？
大多数时候我们使用的都是 InnoDB 存储引擎，在某些读密集的情况下，使用 MyISAM 也是合适的。不过，前提是你的项目不介意 MyISAM 不支持事务、崩溃恢复等缺点（可是~我们一般都会介意啊！）。

《MySQL 高性能》上面有一句话这样写到:

不要轻易相信“MyISAM 比 InnoDB 快”之类的经验之谈，这个结论往往不是绝对的。在很多我们已知场景中，InnoDB 的速度都可以让 MyISAM 望尘莫及，尤其是用到了聚簇索引，或者需要访问的数据都可以放入内存的应用。

一般情况下我们选择 InnoDB 都是没有问题的，但是某些情况下你并不在乎可扩展能力和并发能力，也不需要事务支持，也不在乎崩溃后的安全恢复问题的话，选择 MyISAM 也是一个不错的选择。但是一般情况下，我们都是需要考虑到这些问题的。

因此，对于咱们日常开发的业务系统来说，你几乎找不到什么理由再使用 MyISAM 作为自己的 MySQL 数据库的存储引擎。

## MySQL 索引
MySQL 索引相关的问题比较多，对于面试和工作都比较重要，MySQL 索引相关的知识点和问题：[MySQL索引详解](https://www.yuque.com/vip6688/neho4x/cpwtqmfh71drcvbl)

## MySQL 查询缓存
执行查询语句的时候，会先查询缓存。不过，MySQL 8.0 版本后移除，因为这个功能不太实用

my.cnf加入以下配置，重启 MySQL 开启查询缓存

```plsql
query_cache_type=1
query_cache_size=600000
```

MySQL 执行以下命令也可以开启查询缓存

```plsql
set global  query_cache_type=1;
set global  query_cache_size=600000;
```

如上，**开启查询缓存后在同样的查询条件以及数据情况下，会直接在缓存中返回结果**。这里的查询条件包括查询本身、当前要查询的数据库、客户端协议版本号等一些可能影响结果的信息。

**查询缓存不命中的情况：**

1. 任何两个查询在任何字符上的不同都会导致缓存不命中。
2. 如果查询中包含任何用户自定义函数、存储函数、用户变量、临时表、MySQL 库中的系统表，其查询结果也不会被缓存。
3. 缓存建立之后，MySQL 的查询缓存系统会跟踪查询中涉及的每张表，如果这些表（数据或结构）发生变化，那么和这张表相关的所有缓存数据都将失效。

*_缓存虽然能够提升数据库的查询性能，但是缓存同时也带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。*__因此，开启查询缓存要谨慎，尤其对于写密集的应用来说更是如此。如果开启，要注意合理控制缓存空间大小，一般来说其大小设置为几十 MB 比较合适。此外，__**还可以通过**__*sql_cache__**和**__sql_no_cache_***来控制某个查询语句是否需要缓存：**

```plsql
SELECT sql_no_cache COUNT(*) FROM usr;
```

## MySQL 日志
[MySQL 日志：常见的日志都有什么用？](https://www.yuque.com/vip6688/neho4x/gui0lcqtec9wgvsg)

MySQL 日志常见的面试题有：

+ MySQL 中常见的日志有哪些？
+ 慢查询日志有什么用？
+ binlog 主要记录了什么？
+ redo log 如何保证事务的持久性？
+ 页修改之后为什么不直接刷盘呢？
+ binlog 和 redolog 有什么区别？
+ undo log 如何保证事务的原子性？
+ ……

## MySQL 事务
### 何谓事务？
我们设想一个场景，这个场景中我们需要插入多条相关联的数据到数据库，不幸的是，这个过程可能会遇到下面这些问题：

+ 数据库中途突然因为某些原因挂掉了。
+ 客户端突然因为网络原因连接不上数据库了。
+ 并发访问数据库时，多个线程同时写入数据库，覆盖了彼此的更改。
+ ……

上面的任何一个问题都可能会导致数据的不一致性。为了保证数据的一致性，系统必须能够处理这些问题。事务就是我们抽象出来简化这些问题的首选机制。事务的概念起源于数据库，目前，已经成为一个比较广泛的概念。

**何为事务？****一言蔽之，****事务是逻辑上的****<font style="background-color:#d9dffc;">一组操作，要么都执行，要么都不执行</font>********。**

事务最经典也经常被拿出来说例子就是转账了。假如小明要给小红转账 1000 元，这个转账会涉及到两个关键操作，这两个操作必须都成功或者都失败。

1. 将小明的余额减少 1000 元
2. 将小红的余额增加 1000 元。

事务会把这两个操作就可以看成逻辑上的一个整体，这个整体包含的操作要么都成功，要么都要失败。这样就不会出现小明余额减少而小红的余额却并没有增加的情况。

![1732497609829-3300e612-eda7-4afd-8160-d2b26f3a6cb0.png](./img/QHmSnXBq8gIJZRqx/1732497609829-3300e612-eda7-4afd-8160-d2b26f3a6cb0-442951.png)

事务示意图

### 何谓数据库事务？
大多数情况下，我们在谈论事务的时候，如果没有特指**分布式事务**，往往指的就是**数据库事务**。

数据库事务在我们日常开发中接触的最多了。如果你的项目属于单体架构的话，你接触到的往往就是数据库事务了。

**那数据库事务有什么作用呢？**

简单来说，数据库事务可以保证多个对数据库的操作（也就是 SQL 语句）构成一个逻辑上的整体。构成这个逻辑上的整体的这些数据库操作遵循：**要么全部执行成功,要么全部不执行**。

```plsql
# 开启一个事务
START TRANSACTION;
# 多条 SQL 语句
SQL1,SQL2...
## 提交事务
COMMIT;
```

![1732497609912-3af5640a-b595-400f-9786-68fb01c9088f.png](./img/QHmSnXBq8gIJZRqx/1732497609912-3af5640a-b595-400f-9786-68fb01c9088f-542295.png)

数据库事务示意图

另外，关系型数据库（例如：MySQL、SQL Server、Oracle等）事务都有**ACID**特性：

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497609997-b4403298-3acc-46b1-9460-af45c0ef641b.png)

ACID

1. **原子性**（Atomicity）：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. **一致性**（Consistency）：执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
3. **隔离性**（Isolation）：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. **持久性**（Durability）：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

🌈 这里要额外补充一点：**只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。也就是说 A、I、D 是手段，C 是目的！**想必大家也和我一样，被 ACID 这个概念被误导了很久! 我也是看周志明老师的公开课[《周志明的软件架构课》](https://time.geekbang.org/opencourse/intro/100064201)[](https://time.geekbang.org/opencourse/intro/100064201)才搞清楚的（多看好书！！！）。

![1732497610081-2aee68d5-80e3-4077-a271-817a2e155cbe.png](./img/QHmSnXBq8gIJZRqx/1732497610081-2aee68d5-80e3-4077-a271-817a2e155cbe-011773.png)

AID->C

另外，DDIA 也就是[《Designing Data-Intensive Application（数据密集型应用系统设计）》](https://book.douban.com/subject/30329536/)[](https://book.douban.com/subject/30329536/)的作者在他的这本书中如是说：

Atomicity, isolation, and durability are properties of the database, whereas consis‐tency (in the ACID sense) is a property of the application. The application may relyon the database’s atomicity and isolation properties in order to achieve consistency,but it’s not up to the database alone.翻译过来的意思是：原子性，隔离性和持久性是数据库的属性，而一致性（在 ACID 意义上）是应用程序的属性。应用可能依赖数据库的原子性和隔离属性来实现一致性，但这并不仅取决于数据库。因此，字母 C 不属于 ACID 。

《Designing Data-Intensive Application（数据密集型应用系统设计）》这本书强推一波，值得读很多遍！豆瓣有接近 90% 的人看了这本书之后给了五星好评。另外，中文翻译版本已经在 GitHub 开源，地址：[https://github.com/Vonng/ddia](https://github.com/Vonng/ddia)[](https://github.com/Vonng/ddia)。

![1732497610160-a61c09a1-c6b8-4efd-8c60-d43eb159d192.png](./img/QHmSnXBq8gIJZRqx/1732497610160-a61c09a1-c6b8-4efd-8c60-d43eb159d192-236711.png)

### 并发事务带来了哪些问题?
在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对同一数据进行操作）。并发虽然是必须的，但可能会导致以下的问题。

#### 脏读（Dirty read）
一个事务读取数据并且对数据进行了修改，这个修改对其他事务来说是可见的，即使当前事务没有提交。这时另外一个事务**<font style="background-color:#d9dffc;">读取了这个还未提交的数据</font>**，但第一个事务突然回滚，导致数据并没有被提交到数据库，那第二个事务读取到的就是脏数据，这也就是脏读的由来。

例如：事务 1 读取某表中的数据 A=20，事务 1 修改 A=A-1，事务 2 读取到 A = 19,事务 1 回滚导致对 A 的修改并未提交到数据库， A 的值还是 20。

![1732497610227-72bde29d-1600-4596-83b5-7428b84d9e39.png](./img/QHmSnXBq8gIJZRqx/1732497610227-72bde29d-1600-4596-83b5-7428b84d9e39-702582.png)

脏读

#### 丢失修改（Lost to modify）
在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。

例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 先修改 A=A-1，事务 2 后来也修改 A=A-1，最终结果 A=19，事务 1 的修改被丢失。

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497610282-1ef4f95a-8533-435b-9dac-1e9deb63dd67.png)

丢失修改

#### 不可重复读（Unrepeatable read）
指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在**<font style="background-color:#d9dffc;">一个事务内两次读到的数据是不一样的情况</font>**，因此称为不可重复读。

例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 修改 A=A-1，事务 2 再次读取 A =19，此时读取的结果和第一次读取的结果不同。

![1732497610339-f4d44dca-d615-4fa4-8053-1837bec54cc9.png](./img/QHmSnXBq8gIJZRqx/1732497610339-f4d44dca-d615-4fa4-8053-1837bec54cc9-770399.png)

不可重复读

#### 幻读（Phantom read）
幻读与不可重复读类似。它发生在一个事务读取了几行数据，接着另一个并发事务插入了一些数据时。在随后的查询中，第一个事务就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

例如：事务 2 读取某个范围的数据，事务 1 在这个范围插入了新的数据，**<font style="background-color:#d9dffc;">事务 2 再次读取这个范围的数据发现相比于第一次读取的结果多了新的数据。</font>**

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497610411-d99cfce2-459c-4e23-85c3-84d7436ef63a.png)

幻读

### 不可重复读和幻读有什么区别？
+ 不可重复读的重点是**<font style="background-color:#d9dffc;">内容修改或者记录减少</font>**比如多次读取一条记录发现其中某些记录的值被修改；
+ 幻读的重点在于**<font style="background-color:#d9dffc;">记录新增</font>**比如多次执行同一条查询语句（DQL）时，发现查到的记录增加了。

幻读其实可以看作是不可重复读的一种特殊情况，单独把区分幻读的原因主要是解决幻读和不可重复读的方案不一样。

举个例子：

执行`**delete**`和`**update**`操作的时候，可以直接对记录加锁，保证事务安全。

而执行`**insert**`操作的时候，由于**<font style="background-color:#d9dffc;">记录锁（Record Lock）只能锁住已经存在的记录</font>**，为了避免插入新记录，需要依赖间隙锁（Gap Lock）。

也就是说执行insert操作的时候需要依赖 `**Next-Key Lock（Record Lock+Gap Lock）**` 进行加锁来保证不出现幻读。

### 并发事务的控制方式有哪些？
MySQL 中并发事务的控制方式无非就两种：**锁**和**MVCC**。锁可以看作是悲观控制的模式，多版本并发控制（MVCC，Multiversion concurrency control）可以看作是乐观控制的模式。

**锁**控制方式下会通过锁来显示控制共享资源而不是通过调度手段，MySQL 中主要是通过**读写锁**来实现并发控制。

+ **共享锁（S 锁）**：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
+ **排他锁（X 锁）**：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条记录加任何类型的锁（锁不兼容）。

读写锁可以做到**<font style="background-color:#d9dffc;">读读并行</font>**，但是**<font style="background-color:#d9dffc;">无法做到写读、写写并行</font>**。另外，根据根据锁粒度的不同，又被分为**表级锁(table-level locking)**和**行级锁(row-level locking)**。InnoDB 不光支持表级锁，还支持行级锁，默认为行级锁。**<font style="background-color:#d9dffc;">行级锁的粒度更小，仅对相关的记录上锁即可</font>**（对一行或者多行记录加锁），所以对于并发写入操作来说， InnoDB 的性能更高。不论是表级锁还是行级锁，都存在共享锁（Share Lock，S 锁）和排他锁（Exclusive Lock，X 锁）这两类。

**MVCC**是多版本并发控制方法，即对一份数据会存储多个版本，通过事务的可见性来保证事务能看到自己应该看到的版本。通常会有一个全局的版本分配器来为每一行数据设置版本号，版本号是唯一的。

MVCC 在 MySQL 中实现所依赖的手段主要是:**隐藏字段、read view、undo log**。

+ `undo log `: undo log 用于记录某行数据的多个版本的数据。
+ `read view `和 **隐藏字段** : 用来判断当前版本数据的可见性。

关于 InnoDB 对 MVCC 的具体实现可以看这篇文章：[InnoDB存储引擎对MVCC的实现](https://www.yuque.com/vip6688/neho4x/evp4n2ey4y6g8mft)

### SQL 标准定义了哪些事务隔离级别?
SQL 标准定义了四个隔离级别：

+ **READ-UNCOMMITTED(读取未提交)**：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
+ **READ-COMMITTED(读取已提交)**：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
+ **REPEATABLE-READ(可重复读)**：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
+ **SERIALIZABLE(可串行化)**：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

---

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | :---: | :---: | :---: |
| READ-UNCOMMITTED | √ | √ | √ |
| READ-COMMITTED | × | √ | √ |
| REPEATABLE-READ | × | × | √ |
| SERIALIZABLE | × | × | × |


### MySQL 的隔离级别是基于锁实现的吗？
MySQL 的隔离级别基于锁和 MVCC 机制共同实现的。

SERIALIZABLE 隔离级别是通过锁来实现的，READ-COMMITTED 和 REPEATABLE-READ 隔离级别是基于 MVCC 实现的。不过， SERIALIZABLE 之外的其他隔离级别可能也需要用到锁机制，就比如 REPEATABLE-READ 在当前读情况下需要使用加锁读来保证不会出现幻读。

### MySQL 的默认隔离级别是什么?
MySQL InnoDB 存储引擎的默认支持的隔离级别是**REPEATABLE-READ（可重读）**。我们可以通过SELECT @@tx_isolation;命令来查看，MySQL 8.0 该命令改为SELECT @@transaction_isolation;

```plsql
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

关于 MySQL 事务隔离级别的详细介绍[MySQL事务隔离级别详解](https://www.yuque.com/vip6688/neho4x/scqtpadh3g5871tg)

## MySQL 锁
锁是一种常见的并发事务的控制方式。

### 表级锁和行级锁了解吗？有什么区别？
MyISAM 仅仅支持表级锁(table-level locking)，一锁就锁整张表，这在并发写的情况下性非常差。InnoDB 不光支持表级锁(table-level locking)，还支持行级锁(row-level locking)，默认为行级锁。

行级锁的粒度更小，仅对相关的记录上锁即可（对一行或者多行记录加锁），所以对于并发写入操作来说， InnoDB 的性能更高。

**表级锁和行级锁对比**：

+ **表级锁：**MySQL 中锁定粒度最大的一种锁（全局锁除外），是针对非索引字段加的锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。不过，触发锁冲突的概率最高，高并发下效率极低。表级锁和存储引擎无关，MyISAM 和 InnoDB 引擎都支持表级锁。
+ **行级锁：****MySQL 中锁定粒度最小的一种锁，是****针对索引字段加的锁**，只针对当前操作的行记录进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。行级锁和存储引擎有关，是在存储引擎层面实现的。

### 行级锁的使用有什么注意事项？
InnoDB 的行锁是针对索引字段加的锁，表级锁是针对非索引字段加的锁。当我们执行UPDATE、DELETE语句时，如果WHERE条件中字段没有命中唯一索引或者索引失效的话，就会导致扫描全表对表中的所有行记录进行加锁。这个在我们日常工作开发中经常会遇到，一定要多多注意！！！

不过，很多时候即使用了索引也有可能会走全表扫描，这是因为 MySQL 优化器的原因。

### InnoDB 有哪几类行锁？
InnoDB 行锁是通过对索引数据页上的记录加锁实现的，MySQL InnoDB 支持三种行锁定方式：

+ **记录锁（Record Lock）**：也被称为记录锁，属于单个行记录上的锁。
+ **间隙锁（Gap Lock）**：锁定一个范围，不包括记录本身。
+ **临键锁（Next-Key Lock）**：Record Lock+Gap Lock，锁定一个范围，包含记录本身，主要目的是为了解决幻读问题（MySQL 事务部分提到过）。记录锁只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁。

**在 InnoDB 默认的隔离级别 REPEATABLE-READ 下，行锁默认使用的是 Next-Key Lock。但是，如果操作的索引是唯一索引或主键，InnoDB 会对 Next-Key Lock 进行优化，将其降级为 Record Lock，即仅锁住索引本身，而不是范围。**

一些大厂面试中可能会问到 Next-Key Lock 的加锁范围，这里推荐一篇文章：[MySQL next-key lock 加锁范围是什么？ - 程序员小航 - 2021](https://segmentfault.com/a/1190000040129107)[](https://segmentfault.com/a/1190000040129107)。

### 共享锁和排他锁呢？
不论是表级锁还是行级锁，都存在共享锁（Share Lock，S 锁）和排他锁（Exclusive Lock，X 锁）这两类：

+ **共享锁（S 锁）**：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
+ **排他锁（X 锁）**：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁（锁不兼容）。

排他锁与任何的锁都不兼容，共享锁仅和共享锁兼容。

|  | S 锁 | X 锁 |
| --- | :--- | :--- |
| S 锁 | 不冲突 | 冲突 |
| X 锁 | 冲突 | 冲突 |


由于 MVCC 的存在，对于一般的SELECT语句，InnoDB 不会加任何锁。不过， 你可以通过以下语句显式加共享锁或排他锁。

```plsql
# 共享锁 可以在 MySQL 5.7 和 MySQL 8.0 中使用
SELECT ... LOCK IN SHARE MODE;
# 共享锁 可以在 MySQL 8.0 中使用
SELECT ... FOR SHARE;
# 排他锁
SELECT ... FOR UPDATE;
```

### 意向锁有什么作用？
如果需要用到表锁的话，如何判断表中的记录没有行锁呢，一行一行遍历肯定是不行，性能太差。我们需要用到一个叫做意向锁的东东来快速判断是否可以对某个表使用表锁。

意向锁是表级锁，共有两种：

+ **意向共享锁（Intention Shared Lock，IS 锁）**：事务有意向对表中的某些记录加共享锁（S 锁），加共享锁前必须先取得该表的 IS 锁。
+ **意向排他锁（Intention Exclusive Lock，IX 锁）**：事务有意向对表中的某些记录加排他锁（X 锁），加排他锁之前必须先取得该表的 IX 锁。

**意向锁是由数据引擎自己维护的，用户无法手动操作意向锁，在为数据行加共享/排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。**

意向锁之间是互相兼容的。

|  | IS 锁 | IX 锁 |
| --- | :--- | :--- |
| IS 锁 | 兼容 | 兼容 |
| IX 锁 | 兼容 | 兼容 |


意向锁和共享锁和排它锁互斥（这里指的是表级别的共享锁和排他锁，意向锁不会与行级的共享锁和排他锁互斥）。

|  | IS 锁 | IX 锁 |
| --- | :--- | :--- |
| S 锁 | 兼容 | 互斥 |
| X 锁 | 互斥 | 互斥 |


《MySQL 技术内幕 InnoDB 存储引擎》这本书对应的描述应该是笔误了。

![](https://cdn.nlark.com/yuque/0/2024/png/45178513/1732497610472-459bc8f2-0f65-4fce-a0e0-57ae273e243a.png)

### 当前读和快照读有什么区别？
**快照读**（一致性非锁定读）就是单纯的SELECT语句，但不包括下面这两类SELECT语句：

```plsql
SELECT ... FOR UPDATE
# 共享锁 可以在 MySQL 5.7 和 MySQL 8.0 中使用
SELECT ... LOCK IN SHARE MODE;
# 共享锁 可以在 MySQL 8.0 中使用
SELECT ... FOR SHARE;
```

快照即记录的历史版本，每行记录可能存在多个历史版本（多版本技术）。

快照读的情况下，如果读取的记录正在执行 UPDATE/DELETE 操作，**<font style="background-color:#d9dffc;">读取操作不会因此去等待记录上 X 锁的释放，而是会去读取行的一个快照。</font>**

只有在事务隔离级别**<font style="background-color:#d9dffc;"> RC(读取已提交)</font>** 和 **<font style="background-color:#d9dffc;">RR（可重读）下</font>**，InnoDB 才会使用一致性非锁定读：

+ 在 RC 级别下，对于快照数据，一致性非锁定读总是读取被锁定行的**<font style="background-color:#d9dffc;">最新一份快照数据</font>**。
+ 在 RR 级别下，对于快照数据，一致性非锁定读总是读取本**<font style="background-color:#d9dffc;">事务开始时的行数据版本</font>**。

快照读比较适合对于数据一致性要求不是特别高且追求极致性能的业务场景。

**当前读**（一致性锁定读）就是给行记录加 X 锁或 S 锁。

当前读的一些常见 SQL 语句类型如下：

```plsql
# 对读的记录加一个X锁
SELECT...FOR UPDATE
# 对读的记录加一个S锁
SELECT...LOCK IN SHARE MODE
# 对读的记录加一个S锁
SELECT...FOR SHARE
# 对修改的记录加一个X锁
INSERT...
UPDATE...
DELETE...
```

### 自增锁有了解吗？
不太重要的一个知识点，简单了解即可。

关系型数据库设计表的时候，通常会有一列作为自增主键。InnoDB 中的自增主键会涉及一种比较特殊的表级锁—**自增锁（AUTO-INC Locks）**。

```plsql
CREATE TABLE `sequence_id` (
  `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
  `stub` CHAR(10) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `stub` (`stub`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

更准确点来说，不仅仅是自增主键，AUTO_INCREMENT的列都会涉及到自增锁，毕竟非主键也可以设置自增长。

如果一个事务正在插入数据到有自增列的表时，会先获取自增锁，拿不到就可能会被阻塞住。这里的阻塞行为只是自增锁行为的其中一种，可以理解为自增锁就是一个接口，其具体的实现有多种。具体的配置项为innodb_autoinc_lock_mode（MySQL 5.1.22 引入），可以选择的值如下：

| innodb_autoinc_lock_mode | 介绍 |
| :--- | :--- |
| 0 | 传统模式 |
| 1 | 连续模式（MySQL 8.0 之前默认） |
| 2 | 交错模式(MySQL 8.0 之后默认) |


交错模式下，所有的“INSERT-LIKE”语句（所有的插入语句，包括：INSERT、REPLACE、INSERT…SELECT、REPLACE…SELECT、LOAD DATA等）都不使用表级锁，使用的是轻量级互斥锁实现，多条插入语句可以并发执行，速度更快，扩展性也更好。

不过，如果你的 MySQL 数据库有主从同步需求并且 Binlog 存储格式为 Statement 的话，不要将 InnoDB 自增锁模式设置为交叉模式，不然会有数据不一致性问题。这是因为并发情况下插入语句的执行顺序就无法得到保障。

如果 MySQL 采用的格式为 Statement ，那么 MySQL 的主从同步实际上同步的就是一条一条的 SQL 语句。

最后，再推荐一篇文章：[为什么 MySQL 的自增主键不单调也不连续](https://draveness.me/whys-the-design-mysql-auto-increment/)[](https://draveness.me/whys-the-design-mysql-auto-increment/)。

## MySQL 性能优化
关于 MySQL 性能优化的建议总结，[MySQL高性能优化规范建议总结](https://www.yuque.com/vip6688/neho4x/lqovlxlqfn9iiatb)

### 能用 MySQL 直接存储文件（比如图片）吗？
可以是可以，直接存储文件对应的二进制数据即可。不过，还是建议不要在数据库中存储文件，会严重影响数据库性能，消耗过多存储空间。

可以选择使用云服务厂商提供的开箱即用的文件存储服务，成熟稳定，价格也比较低。

![1732497610542-88d0a90a-00e7-4f26-bed1-cd7c9f6ba247.png](./img/QHmSnXBq8gIJZRqx/1732497610542-88d0a90a-00e7-4f26-bed1-cd7c9f6ba247-772156.png)

也可以选择自建文件存储服务，实现起来也不难，基于 FastDFS、MinIO（推荐） 等开源项目就可以实现分布式文件服务。

**数据库只存储文件地址信息，文件由文件存储服务负责存储。**

相关阅读：[Spring Boot 整合 MinIO 实现分布式文件服务](https://www.51cto.com/article/716978.html)[](https://www.51cto.com/article/716978.html)。

### MySQL 如何存储 IP 地址？
可以将 IP 地址转换成整形数据存储，性能更好，占用空间也更小。

MySQL 提供了两个方法来处理 ip 地址

+ INET_ATON()：把 ip 转为无符号整型 (4-8 位)
+ INET_NTOA():把整型的 ip 转为地址

插入数据前，先用INET_ATON()把 ip 地址转为整型，显示数据时，使用INET_NTOA()把整型的 ip 地址转为地址显示即可。

### 有哪些常见的 SQL 优化手段？
[有哪些常见的 SQL 优化手段？](https://www.yuque.com/vip6688/neho4x/gt65s40gr728k6vb)



### 如何分析 SQL 的性能？
我们可以使用EXPLAIN命令来分析 SQL 的**执行计划**。执行计划是指一条 SQL 语句在经过 MySQL 查询优化器的优化会后，具体的执行方式。

EXPLAIN并不会真的去执行相关的语句，而是通过**查询优化器**对语句进行分析，找出最优的查询方案，并显示对应的信息。

EXPLAIN适用于SELECT,DELETE,INSERT,REPLACE, 和UPDATE语句，我们一般分析SELECT查询较多。

我们这里简单来演示一下EXPLAIN的使用。

EXPLAIN的输出格式如下：

```plsql
mysql> EXPLAIN SELECT `score`,`name` FROM `cus_order` ORDER BY `score` DESC;
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra          |
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
|  1 | SIMPLE      | cus_order | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 997572 |   100.00 | Using filesort |
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
1 row in set, 1 warning (0.00 sec)
```

各个字段的含义如下：

| **列名** | **含义** |
| :--- | :--- |
| id | SELECT 查询的序列标识符 |
| select_type | SELECT 关键字对应的查询类型 |
| table | 用到的表名 |
| partitions | 匹配的分区，对于未分区的表，值为 NULL |
| type | 表的访问方法 |
| possible_keys | 可能用到的索引 |
| key | 实际用到的索引 |
| key_len | 所选索引的长度 |
| ref | 当使用索引等值查询时，与索引作比较的列或常量 |
| rows | 预计要读取的行数 |
| filtered | 按表条件过滤后，留存的记录数的百分比 |
| Extra | 附加信息 |


篇幅问题，我这里只是简单介绍了一下 MySQL 执行计划，详细介绍请看：[MySQL执行计划分析](https://www.yuque.com/vip6688/neho4x/gb0fg4cn3or0kdsg)

### 读写分离和分库分表了解吗？
读写分离和分库分表相关的问题比较多。[读写分离和分库分表常见问题总结](https://www.yuque.com/vip6688/neho4x/me94ronuycgd8sbq)

## MySQL 学习资料推荐
[书籍推荐](https://javaguide.cn/books/database.html#mysql)。

**文章推荐**:

+ [一树一溪的 MySQL 系列教程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3NTc3NjM4Nw==&action=getalbum&album_id=2372043523518300162&scene=173&from_msgid=2247484308&from_itemidx=1&count=3&nolastread=1#wechat_redirect)[](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3NTc3NjM4Nw==&action=getalbum&album_id=2372043523518300162&scene=173&from_msgid=2247484308&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
+ [Yes 的 MySQL 系列教程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkxNTE3NjQ3MA==&action=getalbum&album_id=1903249596194095112&scene=173&from_msgid=2247490365&from_itemidx=1&count=3&nolastread=1#wechat_redirect)[](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkxNTE3NjQ3MA==&action=getalbum&album_id=1903249596194095112&scene=173&from_msgid=2247490365&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
+ [写完这篇 我的 SQL 优化能力直接进入新层次 - 变成派大星 - 2022](https://juejin.cn/post/7161964571853815822)[](https://juejin.cn/post/7161964571853815822)
+ [两万字详解！InnoDB 锁专题！ - 捡田螺的小男孩 - 2022](https://juejin.cn/post/7094049650428084232)[](https://juejin.cn/post/7094049650428084232)
+ [MySQL 的自增主键一定是连续的吗？ - 飞天小牛肉 - 2022](https://mp.weixin.qq.com/s/qci10h9rJx_COZbHV3aygQ)[](https://mp.weixin.qq.com/s/qci10h9rJx_COZbHV3aygQ)
+ [深入理解 MySQL 索引底层原理 - 腾讯技术工程 - 2020](https://zhuanlan.zhihu.com/p/113917726)[](https://zhuanlan.zhihu.com/p/113917726)

## 参考
+ 《高性能 MySQL》第 7 章 MySQL 高级特性
+ 《MySQL 技术内幕 InnoDB 存储引擎》第 6 章 锁
+ Relational Database：[https://www.omnisci.com/technical-glossary/relational-database](https://www.omnisci.com/technical-glossary/relational-database)[](https://www.omnisci.com/technical-glossary/relational-database)
+ 一篇文章看懂 mysql 中 varchar 能存多少汉字、数字，以及 varchar(100)和 varchar(10)的区别：[https://www.cnblogs.com/zhuyeshen/p/11642211.html](https://www.cnblogs.com/zhuyeshen/p/11642211.html)
+ 技术分享 | 隔离级别：正确理解幻读：[https://opensource.actionsky.com/20210818-mysql/](https://opensource.actionsky.com/20210818-mysql/)[](https://opensource.actionsky.com/20210818-mysql/)
+ MySQL Server Logs - MySQL 5.7 Reference Manual：[https://dev.mysql.com/doc/refman/5.7/en/server-logs.html](https://dev.mysql.com/doc/refman/5.7/en/server-logs.html)[](https://dev.mysql.com/doc/refman/5.7/en/server-logs.html)
+ Redo Log - MySQL 5.7 Reference Manual：[https://dev.mysql.com/doc/refman/5.7/en/innodb-redo-log.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-redo-log.html)[](https://dev.mysql.com/doc/refman/5.7/en/innodb-redo-log.html)
+ Locking Reads - MySQL 5.7 Reference Manual：[https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)[](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)
+ 深入理解数据库行锁与表锁[https://zhuanlan.zhihu.com/p/52678870](https://zhuanlan.zhihu.com/p/52678870)[](https://zhuanlan.zhihu.com/p/52678870)
+ 详解 MySQL InnoDB 中意向锁的作用：[https://juejin.cn/post/6844903666332368909](https://juejin.cn/post/6844903666332368909)[](https://juejin.cn/post/6844903666332368909)
+ 深入剖析 MySQL 自增锁：[https://juejin.cn/post/6968420054287253540](https://juejin.cn/post/6968420054287253540)[](https://juejin.cn/post/6968420054287253540)
+ 在数据库中不可重复读和幻读到底应该怎么分？：[https://www.zhihu.com/question/392569386](https://www.zhihu.com/question/392569386)[](https://www.zhihu.com/question/392569386)





> 更新: 2024-01-29 23:51:38  
原文: [https://www.yuque.com/vip6688/neho4x/fm00wr1u4fuoxxha](https://www.yuque.com/vip6688/neho4x/fm00wr1u4fuoxxha)
>



> 更新: 2024-11-25 10:09:12  
> 原文: <https://www.yuque.com/neumx/laxg2e/aae878bee1e48b793f28527be1bf15ad>