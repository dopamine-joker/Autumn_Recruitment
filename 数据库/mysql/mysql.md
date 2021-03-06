# 1. MySQL体系构架、存储引擎和索引结构

https://blog.csdn.net/wangfeijiu/article/details/112454405

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126212911166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130005455679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

# 2. 事务四个特性ACID

**事务**具有四个特征：原子性（ Atomicity ）、一致性（ Consistency ）、隔离性（ Isolation ）和持续性（ Durability ）。这四个特性简称为 ACID 特性。

转账:

```mysql
start transaction;
select balance from checking where customer_id = 10233276;
update checking set balance = balance - 200.00 where customer_id = 10233276;
update savings set balance = balance + 200.00 where customer_id = 10233276;
commit;
```

1 、**原子性**

**事务**是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做

2 、**一致性**

数据库总是从一个一致性的状态转换到另一个一致性的状态。举例来说，假设用户A和用户B两者的钱加起来一共是1000，那么不管A和B之间如何转账、转几次账，事务结束后两个用户的钱相加起来应该还得是1000，这就是事务的一致性。

3 、**隔离性**

一个事务所做的修改在最终提交以前，对其它事务是不可见的。在例子中，执行完第三条语句，第四条未开始时，另外一个转账事务看到的checking.balance并没有减去200.

4 、**持续性**

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

系统发生崩溃可以用重做日志（Redo Log）进行恢复，从而实现持久性。与回滚日志记录数据的逻辑修改不同，重做日志记录的是数据页的物理修改。

二进制日志是在**存储引擎的上层**产生的，不管是什么存储引擎，对数据库进行了修改都会产生二进制日志。而redo log是innodb层产生的，只记录该存储引擎中表的修改。**并且二进制日志先于redo log被记录**。

# 3. MYSQL锁机制

1. 读锁(共享锁)和写锁(排他锁)，表锁，行锁

   https://blog.csdn.net/why15732625998/article/details/80439315#commentBox

   **修正，blog中2.3标题为页锁**
   
   ![image-20210908160424901](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160424901.png)

![image-20210908160327613](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160327613.png)

# 4. [数据库事务隔离级别](https://blog.csdn.net/riemann_/article/details/89901626?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161720291516780357280628%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161720291516780357280628&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-89901626.first_rank_v2_pc_rank_v29&utm_term=%E6%95%B0%E6%8D%AE%E5%BA%93%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB&spm=1018.2226.3001.4187)

- **读未提交(Read uncommitted)**

  读未提交，顾名思义，就是**一个事务可以读取另一个未提交事务的数据**。

- **读提交(Read committed)**

  读提交，顾名思义，就是**一个事务要等另一个事务提交后才能读取数据**。

- **重复读(Repeatable read)**

  重复读，就是在**开始读取数据（事务开启）时，不再允许修改操作**。间隙锁是RR隔离级别下防止幻读的主要原因（不能完全防止）

- **序列化(Serializable)**

  Serializable 是**最高的事务隔离级别**，在该级别下，**事务串行化顺序执行**，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。
  
  <font color=red>大多数数据库默认的事务隔离级别是**Read committed**，比如Sql Server , Oracle。Mysql的默认隔离级别是**Repeatable read**。</font>
  
  **不可重复读的重点是修改** **:**
  同样的条件 ,  你读取过的数据 ,  再次读取出来发现值不一样了
  **幻读的重点在于新增或者删除**
  同样的条件 ,  第 1 次和第 2 次读出来的记录数不一样。指的是一个事务在前后两次查询同一个范围的时候,后一次查询看到了前一次查询没有看到 的数据行
  
  ![image-20210908160302805](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160302805.png)

# 5. [Mysql的MVCC(多版本并发控制)机制](https://blog.csdn.net/riemann_/article/details/94838870?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161720399616780271574701%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161720399616780271574701&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-94838870.first_rank_v2_pc_rank_v29&utm_term=MVCC&spm=1018.2226.3001.4187)

https://zhuanlan.zhihu.com/p/148035779

MVCC中ReadView: https://zhuanlan.zhihu.com/p/110263562

**多版本并发控制**（Multi-Version Concurrency Control, MVCC）是 MySQL 的 **InnoDB** 存储引擎实现隔离级别的一种具体方式，用于实现**提交读**和**可重复读**这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，要求很低，无需使用 MVCC。**可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现**。

MVCC是通过在**每行记录后面保存两个隐藏的列**来实现的。这两个列，**一个保存了行的创建时间，一个保存行的过期时间（或删除时间）**。当然存储的并不是实际的时间值，而是**系统版本号（system version number)**。每开始一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。

![img](https://img-blog.csdnimg.cn/20200531214014768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjU1MDE3,size_16,color_FFFFFF,t_70)

如图中所示，假如三个事务更新了同一行数据，那么就会有对应的三个数据版本。实际上版本1、版本2并非实际物理存在的，而图中的U1和U2实际就是**undo log**，这v1和v2版本是根据当前v3和undo log计算出来的。

下面看一下在**REPEATABLE READ**隔离级别下，MVCC具体是如何操作的。

**SELECT**
InnoDB会根据以下两个条件检查每行记录：
1、InnoDB只查找版本早于当前事务版本的数据行（也就是，行的系统版本号**小于或等于事务的系统版本号**），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
2、行的删除版本要么未定义，要么大于当前事务版本号。这可以确保事务读取到的行，在事务开始之前未被删除。
**只有符合上述两个条件的记录，才能返回作为查询结果**。
**INSERT**
InnoDB为新插入的每一行保存当前系统版本号作为行版本号。
**DELETE**
InnoDB为删除的每一行保存当前系统版本号作为行删除标识。

![image-20210912091918211](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210912091918211.png)

![image-20210912091853965](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210912091853965.png)

**UPDATE**
InnoDB为插入一行新记录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为行删除标识。
保存这两个额外系统版本号，使大多数读操作都可以不用加锁。这样设计使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行，不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作

![image-20210912091947277](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210912091947277.png)

# 6. Next-Key Locks

Next-Key Locks 是 MySQL 的 InnoDB 存储引擎的一种锁实现。

**MVCC 不能完全解决幻读问题(可以解决一部分)**，Next-Key Locks 就是为了解决这个问题而存在的。在可重复读（REPEATABLE READ）隔离级别下，使用 MVCC + Next-Key Locks 可以解决一部分幻读问题。

**<a name="hd">MVCC不能解决幻读举例</a>**

[click](https://ac.nowcoder.com/discuss/230450?type=6)

![image-20210912110202171](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210912110202171.png)

这里为什么update会涉及到444这一行？因为<font color=red>select是快照读，update是当前读</font>

**InnoDB有三种行锁的算法：**

1. **Record Locks** : 单个行记录上的锁。

锁定一个记录上的索引，而不是记录本身。

如果表没有设置索引，InnoDB 会自动在主键上创建隐藏的聚簇索引，因此 Record Locks 依然可以使用。

2. **Gap Locks** : 间隙锁，锁定一个范围，但不包括记录本身。GAP锁的目的，是为了**防止同一事务的两次当前读，出现幻读的情况**。InnoDB默认加锁方式。

锁定索引之间的间隙，但是不包含索引本身。例如当一个事务执行以下语句，其它事务就不能在 t.c 中插入 15。

```sql
SELECT c FROM t WHERE c BETWEEN 10 and 20 FOR UPDATE;
```

3. **Next-Key Locks** : 1+2，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。但不能完全解决，只能解决范围查询的语句。（因为缝隙被锁住了 过不去）

它是 Record Locks 和 Gap Locks 的结合，不仅锁定一个记录上的索引，也锁定索引之间的间隙。它锁定一个前开后闭区间，例如一个索引包含以下值：10, 11, 13, and 20，那么就需要锁定以下区间：

```sql
(-∞, 10]
(10, 11]
(11, 13]
(13, 20]
(20, +∞)
```

# 6.存储引擎

两个最常用的是**Innodb**和**MyISAM**

## 1. Innodb引擎

InnoDB 是一个事务安全的存储引擎（**支持事务**），它具备提交、**回滚(undo log)**以及崩溃恢复的功能以保护用户数据。InnoDB 的**行级别锁定**保证数据一致性提升了它的多用户并发数以及性能。InnoDB 将用户数据存储在聚集索引中以减少基于主键的普通查询所带来的 I/O 开销。为了保证数据的完整性，InnoDB 还支持外键约束。默认**使用B+TREE数据结构存储索引**。

**特点**

- **支持事务**，支持4个事务隔离级别
- **行级锁定**（更新时锁定当前行）
- 读写阻塞与事务隔离级别相关
- 既能**缓存索引**又能**缓存数据**
- **支持外键**
- InnoDB更消耗资源，读取速度没有MyISAM快
- 在InnoDB中存在着缓冲管理，通过缓冲池，将索引和数据全部缓存起来，加快查询的速度；
- 对于InnoDB类型的表，其**数据的物理组织形式是聚簇表**。所有的数据按照主键来组织。**数据和索引放在一块**，都位于B+数的叶子节点上；

**业务场景**

- 需要**支持事务**的场景（银行转账之类）
- 适合**高并发**，行级锁定对高并发有很好的适应能力，但需要确保查询是通过索引完成的
- 数据修改较频繁的业务

**InnoDB引擎调优**

- 主键尽可能小，否则会给Secondary index带来负担
- 避免全表扫描，这会造成锁表
- 尽可能缓存所有的索引和数据，减少IO操作
- 避免主键更新，这会造成大量的数据移动

## 2. MyISAM引擎

MyISAM既**不支持事务**、**也不支持外键**、其优势是访问速度快，但是**表级别的锁定**限制了它在读写负载方面的性能，因此它经常应用于只读或者以读为主的数据场景。**默认使用B+TREE数据结构存储索引**。

**特点**

- **不支持事务**
- **表级锁定**（更新时锁定整个表）
- 读写互相阻塞（写入时阻塞读入、读时阻塞写入；但是读不会互相阻塞）
- 只会**缓存索引**（通过key_buffer_size缓存索引，但是不会缓存数据）
- **不支持外键**
- <font color=red>**读取速度快**</font>

**业务场景**

- **不需要支持事务的场景**（像银行转账之类的不可行）
- 一般读数据的较多的业务
- 数据修改相对较少的业务
- 数据一致性要求不是很高的业务

**MyISAM引擎调优**

- 设置合适索引
- 启用延迟写入，尽量一次大批量写入，而非频繁写入
- 尽量顺序insert数据，让数据写入到尾部，减少阻塞
- 降低并发数，高并发使用排队机制
- MyISAM的count只有全表扫描比较高效，带有其它条件都需要进行实际数据访问

## 3. Memory引擎

在内存中创建表。每个MEMORY表只实际对应一个磁盘文件(frm 表结构文件)。MEMORY类型的表访问非常得快，因为它的**数据是放在内存中**的，并且默认使用HASH索引。要记住，在用完表格之后就删除表格，不然一直占据内存空间。

**特点**:

- 支持的数据类型有限制，比如：不支持TEXT和BLOB类型（长度不固定），对于字符串类型的数据，只支持固定长度的行，VARCHAR会被自动存储为CHAR类型；
- 支持的锁粒度为表级锁。所以，在访问量比较大时，表级锁会成为MEMORY存储引擎的瓶颈；
- 由于数据是存放在内存中，一旦服务器出现故障，数据都会丢失；
- 查询的时候，如果有用到临时表，而且临时表中有BLOB，TEXT类型的字段，那么这个临时表就会转化为MyISAM类型的表，性能会急剧降低；
- 默认使用hash索引。
- 如果一个内部表很大，会转化为磁盘表。

## 4.对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126233629589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

# 7.1. 常见的索引结构

B-Tree,B+Tree,Hash

# 7.2. mysql索引

​		索引用于快速找出在某个列中有一特定值的行。不使用索引，MySQL必须从第一条记录开始读完整个表，直到找出相关的行，表越大查询数据所花费的时间就越多。如果表中查询的列有索引，MySQL能够快速到达一个位置去搜索数据文件，而不必查看所有数据，那么将会节省很大一部分时间。

　　例如：有一张person表，其中有2W条记录，记录着2W个人的信息。有一个Phone的字段记录每个人的电话号码，现在想要查询出电话号码为xxxx的人的信息。

　　如果没有索引，那么将从表中第一条记录一条条往下遍历，直到找到该条信息为止。

　　如果有了索引，那么会将 Phone 字段，通过一定的方法进行存储，好让查询该字段上的信息时，能够快速找到对应的数据，而不必在遍历2W条数据了。其中MySQL中的索引的存储类型有两种：**BTREE**、**HASH**。 也就是用树或者Hash值来存储该字段。

https://www.cnblogs.com/nananana/p/10387720.html

https://blog.csdn.net/wangfeijiu/article/details/113409719

**索引逻辑分类**:

1. 按功能划分:
   - 主键索引：一张表只能有一个主键索引，不允许重复、不允许为 NULL；
   - 唯一索引：数据列不允许重复，允许为 NULL 值，一张表可有多个唯一索引，但是一个唯一索引只能包含一列，比如身份证号码、卡号等都可以作为唯一索引；
   - 普通索引：一张表可以创建多个普通索引，一个普通索引可以包含多个字段，允许数据重复，允许 NULL 值插入；
   - 全文索引：它查找的是文本中的关键词，主要用于全文检索。

2. 按列数划分:
   - 单例索引：一个索引只包含一个列，一个表可以有多个单例索引。
   - 组合索引：一个组合索引包含两个或两个以上的列。查询的时候遵循 mysql 组合索引的 “最左前缀”原则，即使用 where 时条件要按照建立索引的时候字段的排列方式放置索引才会生效。

**索引物理分类**:

- **聚簇索引**:聚簇是为了提高某个属性(或属性组)的查询速度，把这个或这些属性(称为聚簇码)上具有相同值的元组集中存放在连续的物理块。
- **非聚簇索引**:数据和索引是分开的，B+树叶子节点存放的不是数据表的行记录。

虽然InnoDB和MyISAM存储引擎都默认使用B+树结构存储索引，但是**只有InnoDB的主键索引才是聚簇索引**，InnoDB中的辅助索引以及MyISAM使用的都是非聚簇索引。**每张表最多只能拥有一个聚簇索引**。

**InnoDB**使用B+TREE存储数据，**除了主键索引为聚簇索引，其它索引(辅助索引，二级索引)均为非聚簇索引**。

InnoDB使用**B+Tree**数据结构存储索引，根据索引物理结构可将索引划分为**聚簇索引**和**非聚簇索引**（也可称辅助索引或二级索引）。一个表中只能存在一个**聚簇索引（主键索引）**，但可以**存在多个非聚簇索引**。

InnoDB**辅助索引**的访问需要**两次索引查找**，第一次从辅助索引树**找到主键值**，第二次根据主键值**到主键索引树中找到对应的行数据**。（回表）

B+树 叶子节点包含数据表中行记录就是**聚簇索引**（索引和数据是一块的）。

![image-20210908160527591](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160527591.png)

B+树 叶子节点没包含数据表中行记录就是**非聚簇索引**（索引和数据是分开的）。innodb中辅助索引保存的是索引值和**主键值**，而在MyISAN保存的是索引值和**行指针**。

![image-20210908160539364](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160539364.png)

MyISAM也使用B+Tree作为索引结构，但具体实现方式却与InnoDB截然不同。**MyISAM使用的都是非聚簇索引**

![image-20210908160548013](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160548013.png)

![image-20210908160637731](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160637731.png)

 **InnoDB索引**和**MyISAM索引**的区别:

**一是主索引的区别，InnoDB的数据文件本身就是索引文件。而MyISAM的索引和数据是分开的。**

**二是辅助索引的区别：InnoDB的辅助索引data域存储相应记录<font color=red>主键的值</font>而不是地址。而MyISAM的辅助索引和主索引没有多大区别，data域存储相应记录的<font color=red>地址</font>。**

InnoDB在MySQL 5.6和更高版本中提供对**全文索引(FULLTEXT )**的支持。

---

**创建索引**

```mysql
-- 创建普通索引 
CREATE INDEX index_name ON table_name(col_name);
-- 创建唯一索引
CREATE UNIQUE INDEX index_name ON table_name(col_name);
-- 创建普通组合索引
CREATE INDEX index_name ON table_name(col_name_1,col_name_2);
-- 创建唯一组合索引
CREATE UNIQUE INDEX index_name ON table_name(col_name_1,col_name_2);
```

**修改表结构创建索引**

```mysql
ALTER TABLE table_name ADD INDEX index_name(col_name);
```

**创建表时直接指定索引**

```mysql
CREATE TABLE table_name (
    ID INT NOT NULL,
    col_name VARCHAR (16) NOT NULL,
    INDEX index_name (col_name)
);
```

**删除索引**

```mysql
-- 直接删除索引
DROP INDEX index_name ON table_name;
-- 修改表结构删除索引
ALTER TABLE table_name DROP INDEX index_name;
```

**创建全文索引**

```mysql
//建表的时候
FULLTEXT KEY keyname(colume1,colume2)  // 创建联合全文索引列
//在已存在的表上创建
create fulltext index keyname on xxtable(colume1,colume2);
alter table xxtable add fulltext index keyname (colume1,colume2);
```

**使用全文索引**

全文索引有独特的语法格式，需要配合match 和 against 关键字使用

- match()函数中指定的列必须是设置为全文索引的列
- against()函数标识需要模糊查找的关键字

```mysql
 create table fulltext_test(
     id int auto_increment primary key,
     words varchar(2000) not null,a
     artical text not null,
     fulltext index words_artical(words,artical)
)engine=innodb default charset=utf8;

insert into fulltext_test values(null,'a','a');
insert into fulltext_test values(null,'aa','aa');
insert into fulltext_test values(null,'aaa','aaa');
insert into fulltext_test values(null,'aaaa','aaaa');
```

# 8. 问题

[面经链接](https://blog.csdn.net/ligupeng7929/article/details/79421205?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161742394716780357226031%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161742394716780357226031&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-79421205.first_rank_v2_pc_rank_v29&utm_term=mysql%E9%9D%A2%E7%BB%8F&spm=1018.2226.3001.4187)

1. **如何设计一个高并发的系统**

   ① 数据库的优化，包括合理的事务隔离级别、SQL语句优化、索引的优化

   ② 使用缓存，尽量减少数据库 IO

   ③ 分布式数据库、分布式缓存

   ④ 服务器的负载均衡

2. **索引的底层实现原理和优化**

   **B+树，经过优化的B+树**

   主要是在所有的叶子结点中增加了指向下一个叶子节点的指针，因此**InnoDB建议为大部分表使用默认自增的主键作为主索引**。

3. **什么情况下设置了索引但无法使用** 

   ① 以“%”开头的LIKE语句，模糊匹配

   ② OR语句前后没有同时使用索引

   ③ 数据类型出现隐式转化（如varchar不加单引号的话可能会自动转换为int型）

4.  **对于关系型数据库而言，索引是相当重要的概念，请回答有关索引的几个问题**

   a)、**索引的目的是什么？**

   快速访问数据表中的特定信息，提高检索速度

   创建唯一性索引，保证数据库表中每一行数据的唯一性。

   加速表和表之间的连接（因为B+树已经排好序了）

   使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间

   b)、**索引对数据库系统的负面影响是什么？**

   负面影响：
   创建索引和维护索引需要耗费时间，这个时间随着数据量的增加而增加；索引需要占用物理空间，不光是表需要占用数据空间，每个索引也需要占用物理空间；当对表进行增、删、改、的时候索引也要动态维护，这样就降低了数据的维护速度。

   c)、**为数据表建立索引的原则有哪些？**

   在最频繁使用的、用以缩小查询范围的字段上建立索引。

   在频繁使用的、需要排序的字段上建立索引

   d)、 **什么情况下不宜建立索引？**

   对于查询中很少涉及的列或者重复值比较多的列，不宜建立索引。

   对于一些特殊的数据类型，不宜建立索引，比如文本字段（text）等

5. **SQL注入漏洞产生的原因？如何防止？**

   SQL注入产生的原因：程序开发过程中不注意规范书写sql语句和对特殊字符进行过滤，导致客户端可以通过全局变量POST和GET提交一些sql语句正常执行。

   mysql注入的例子:

![image-20210908160800573](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160800573.png)

6. **sql注入的主要特点**

   变种极多，攻击简单，危害极大

2. **简单描述mysql中，索引，主键，唯一索引，联合索引的区别，对数据库的性能有什么影响（从读写两方面）（新浪网技术部）**

   索引是一种特殊的文件(InnoDB数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。
   普通索引(由关键字KEY或INDEX定义的索引)的唯一任务是加快对数据的访问速度。
   普通索引允许被索引的数据列包含重复的值。如果能确定某个数据列将只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该用关键字UNIQUE把它定义为一个唯一索引。也就是说，唯一索引可以 保证数据记录的唯一性。
   主键，是一种特殊的唯一索引，在一张表中只能定义一个主键索引，主键用于唯一标识一条记录，使用关键字 PRIMARY KEY 来创建。
   索引可以覆盖多个数据列，如像INDEX(columnA, columnB)索引，这就是联合索引。
   索引可以极大的提高数据的查询速度，但是会降低插入、删除、更新表的速度，因为在执行这些写操作时，还要操作索引文件。

3. **数据库中的事务是什么?**

   事务（transaction）是作为一个单元的一组有序的数据库操作。如果组中的所有操作都成功，则认为事务成功，即使只有一个操作失败，事务也不成功。如果所有操作完成，事务则提交，其修改将作用于所有其他数据库进程。如果一个操作失败，则事务将回滚，该事务所有操作的影响都将取消。事务的 **四大特性**(ACID),原子性、隔离性、一致性、持久性。

4. **写出三种以上MySQL数据库存储引擎的名称（提示：不区分大小写）**

   MyISAM、InnoDB、Memory（Heap）、BDB（BerkeleyDB）、Merge、Example、Federated、
   Archive、CSV、Blackhole、MaxDB 等等十几个引擎

5. **优化数据库的方法**

   [MySQL数据库优化的八大方式（经典必看）](https://www.jianshu.com/p/dac715a88b44)

   · 选取最适用的字段属性，尽可能减少定义字段宽度，尽量把字段设置NOTNULL，例如'省份'、'性别'最好适用ENUM

   · 使用连接(JOIN)来代替子查询

   · 适用联合(UNION)来代替手动创建的临时表

   · 事务处理

   · 锁定表、优化事务处理

   · 适用外键，优化锁定表（外键能保持数据的一致性、完整性）	//一般公司不用

   · 建立索引

   · 优化查询语句

6. **mysql中char与varchar的区别分析**

   1. 都是用来存储字符串的，只是他们的保存方式不一样。

   2. char有固定的长度，而varchar属于可变长的字符类型。
   3. char(M)类型的数据列里，每个值都占用M个字节，如果某个长度小于M，MySQL就会在它的右边用空格字符补足．在varchar(M)类型的数据列里，每个值只占用刚好够用的字节再加上一个用来记录其长度的字节（即总长度为L+1字节）

7. **什么叫视图？游标是什么？**

   答：视图是一种虚拟的表，具有和物理表相同的功能。可以对视图进行增，改，查，操作，视图通常是有一个表或者多个表的行或列的子集。对视图的修改不影响基本表。它使得我们获取数据更容易，相比多表查询。

     游标：是对查询出来的结果集作为一个单元来有效的处理。游标可以定在该单元中的特定行，从结果集的当前行检索一行或多行。可以对结果集当前行做修改。一般不使用游标，但是需要逐条处理数据的时候，游标显得十分重要。

8. **什么是存储过程？用什么来调用？**

   存储过程是一个预编译的SQL语句，优点是允许模块化的设计，就是说只需创建一次，以后在该程序中就可以调用多次。如果某次操作需要执行多次SQL，使用存储过程比单纯SQL语句执行要快。可以用一个命令对象来调用存储过程。

9. **什么是基本表？什么是视图？**

   基本表是本身独立存在的表，在 SQL 中一个关系就对应一个表。  视图是从一个或几个基本表导出的表。视图本身不独立存储在数据库中，是一个虚表  

10. **[索引失效的情况](https://www.cnblogs.com/wdss/p/11186411.html)**

    - like 以%开头，索引无效；当like前缀没有%，后缀有%时，索引有效。
    - or语句前后没有同时使用索引。当or左右查询字段只有一个是索引，该索引失效，只有当or左右查询字段均为索引时，才会生效
    - 组合索引，不是使用第一列索引，索引失效。
    - 数据类型出现隐式转化。如varchar不加单引号的话可能会自动转换为int型，使索引无效，产生全表扫描。
    - (在索引列上使用 IS NULL 或 IS NOT NULL操作，索引不一定失效)[https://mp.weixin.qq.com/s/CEJFsDBizdl0SvugGX7UmQ]！！！
    - 在索引字段上使用not，<>，!=，不等于操作符是可能不会用到索引的，因此对它的处理只会产生全表扫描。 优化方法： key<>0 改为 key>0 or key<0
    - 对索引字段进行计算操作或使用函数
    - 当全表扫描速度比索引速度快时，mysql会使用全表扫描，此时索引失效

11. **[mysql一级索引和二级索引](http://www.mybatis.cn/archives/941.html)**

    (1) **一级索引**

    索引和数据存储在一起，都存储在同一个B+tree中的叶子节点。一般主键索引都是一级索引。

    (2) **二级索引**

    二级索引树的叶子节点存储的是主键而不是数据。也就是说，在找到索引后，得到对应的主键，再回到一级索引中找主键对应的数据记录。

     一级索引和二级索引的关系：**回表**
    一级索引可以单独存在，二级索引不能单独存在，必须依附于一级索引，这叫做“回表”。

12. **使用Memory引擎的表可以进行范围查询吗**

    Hash索引本身确实不支持范围查询，因为它是通过Hash算法来存储的。Memory引擎中使用Hash索引，但是使用Memory引擎的表是可以进行范围查询的，只是它在范围查询时Hash索引会失效。

![image-20210908160832572](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160832572.png)

13. **分析SQL的执行**

    使用**EXPLAIN**关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理SQL语句。
    
14. **[INNER JOIN，LEFT JOIN，RIGHT JOIN](https://www.cnblogs.com/94cool/p/13985665.html)**

15. **utf8和utf8mb4的区别**

    utf8最多只支持三个字节的UTF-8字符，如char(100) mysql会为这个字段保留300字节长度（因为一个UTF-8最多三字节）。但保存emoji字符和一些不常用汉字需要4个字节，这时候就需要使用utf8mb4了。

# 9. group_by 原理

https://blog.csdn.net/qq403580298/article/details/90756352

# 10. 主要存储引擎

- **MyISAM**: 拥有较高的插入，查询速度，但不支持事务(hash索引)
- **InnoDB** ：5.5.8版本后Mysql的默认数据库引擎，支持ACID事务，支持行级锁定
- **Memory** ：所有数据置于内存的存储引擎，拥有极高的插入，更新和查询效率。但是会占用和数据量成正比的内存空间。并且其内容会在Mysql重新启动时丢失

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126233629589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

# 11. InnoDB引擎调优

- 主键尽可能小，否则会给Secondary index带来负担
- 避免全表扫描，这会造成锁表
- 尽可能缓存所有的索引和数据，减少IO操作
- 避免主键更新，这会造成大量的数据移动

# 12. 什么是B-Tree和B+ tree

**B树**

B树是一种多路搜索树，一棵m阶的B树满足下列条件：

- 树中每个结点至多有m个孩子
- 根结点的儿子数为[2, M]；
- 除根结点以外的非叶子结点的儿子数为[M/2, M]；
- 每个结点存放至少M/2-1（取上整）和至多M-1个关键字；（至少2个关键字）
- 非叶子结点的关键字个数 = 指向子节点的指针个数-1；
- 非叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；
- 非叶子结点的指针：P[1], P[2], …, P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1], K[i])的子树；
- 所有叶子结点位于同一层；

以下是3阶B树

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131224804956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

磁盘读取数据是以盘块(block)为基本单位的。

以下结合磁盘块作图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128210127904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

B树的特征：

- **关键字集合分布在整颗树中**；
- 任何一个关键字出现且只出现在一个结点中；
- 搜索有可能在非叶子结点结束；
- 其搜索性能等价于在关键字全集内做一次二分查找；
- 自动层次控制；

B树的搜索，从根结点开始，对结点内的关键字（有序）序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的儿子结点；重复，直到所对应的儿子指针为空，或已经是叶子结点

**B+树**

B+树是B-树的变体，也是一种多路搜索树：（❀ 表示两者间的不同点）

- 树中每个结点至多有m个孩子
- 根结点的儿子数为[2, M]；
- 除根结点以外的非叶子结点的儿子数为[M/2, M]；
- 非叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；
- **❀ 非叶子结点的子树指针与关键字个数相同；**
- **❀ 非叶子结点的子树指针P[i]，指向关键字值属于[K[i], K[i+1])的子树；（B树是开区间,因为B+树非叶子节点不作为存储）；**
- **❀ 为所有叶子结点增加一个链指针；**
- **❀ 所有关键字都在叶子结点出现；**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128213406422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

B+树的特征：

- 所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的；
- **不可能在非叶子结点命中**；
- **非叶子结点**相当于是叶子结点的**索引（稀疏索引）**，**叶子结点**相当于是存储（关键字）数据的**数据层**；
- **每一个叶子节点都包含指向下一个叶子节点的指针**，从而方便叶子节点的范围遍历。
- 更适合文件索引系统；

**B+树的搜索与B-树也基本相同，区别是B+树只有达到叶子结点才命中（B-树可以在非叶子结点命中），其性能也等价于在关键字全集做一次二分查找**；

# 13. 为什么B+树比B树更适合作索引

1. **B+ 树的磁盘读写代价更低**
    B+ 树的数据都集中在叶子节点，分支节点 只负责指针（索引）；B 树的分支节点既有指针也有数据 。这将导致B+ 树的层高会小于B 树的层高，也就是说B+ 树平均的I/O次数会小于B 树。（因为B+树非叶子节点不存储数据，因此相比于B树，每一次的I/O(每次I/O固定读入磁盘页大小)可以获得更多的索引）(或者说可以提前将索引缓存)
2. **B+ 树的查询效率更加稳定**
    B+ 树的数据都存放在叶子节点，故任何关键字的查找必须走一条从根节点到叶子节点的路径。所有关键字的查询路径相同，每个数据查询效率相当。
3. **B+树更便于遍历**
    由于B+树的数据都存储在叶子结点中，分支结点均为索引，遍历只需要扫描一遍叶子节点即可；B树因为其分支结点同样存储着数据，要找到具体的数据，需要进行一次中序遍历按序来搜索。
4. **B+树更擅长范围查询**
    B+树叶子节点存放数据，数据是按顺序放置的**双向链表**。B树范围查询只能中序遍历。
5. **B+ 树占用内存空间小**
    B+ 树索引节点没有数据，比较小。在内存有限的情况下，相比于B树索引可以加载更多B+ 树索引。

**性能上**（也即为什么说B+树比B树更适合实际应用中操作系统的文件索引和数据库索引？）

- 不同于B树只适合**随机检索**，B+树同时支持**随机检索**和**顺序检索**；
- B+树的磁盘读写代价更低。B+树的**内部结点并没有指向关键字具体信息的指针**，其内部结点比B树小，**盘块能容纳的结点中关键字数量更多**，一次性读入内存中(一次I/O读入的磁盘页大小固定)可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素。
- B+树的查询效率更加稳定。B树搜索有可能会在非叶子结点结束，越靠近根节点的记录查找时间越短，只要找到关键字即可确定记录的存在，其性能等价于在关键字全集内做一次二分查找。而在B+树中，顺序检索比较明显，随机检索时，任何关键字的查找都必须走一条从根节点到叶节点的路，所有关键字的查找路径长度相同，导致每一个关键字的查询效率相当。
- （数据库索引采用B+树的主要原因是，）B-树在提高了磁盘IO性能的同时并没有解决**元素遍历的效率**低下的问题。B+树的叶子节点使用指针顺序连接在一起，**只要遍历叶子节点就可以实现整棵树的遍历**。而且**在数据库中基于范围的查询是非常频繁的**，而B树不支持这样的操作（或者说效率太低）。

# 14. [聚簇索引和非聚簇索引(辅助索引)](https://www.cnblogs.com/sy270321/p/12864357.html)

1. 什么是聚簇索引？

    很简单记住一句话：找到了索引就找到了需要的数据，那么这个索引就是聚簇索引，所以主键就是聚簇索引，修改聚簇索引其实就是修改主键。

    聚簇索引并不是一种单独的索引类型，而**是一种数据存储方式**。具体细节依赖于其实现方式。

2. 什么是非聚簇索引

    索引的存储和数据的存储是分离的，也就是说找到了索引但没找到数据，需要根据索引上的值(主键)再次回表查询,非聚簇索引也叫做辅助索引。

可以按以下四个维度回答：

- 一个表中只能拥有一个聚集索引，而非聚集索引一个表可以存在多个。
- 聚集索引，索引中键值的逻辑顺序决定了表中相应行的物理顺序；非聚集索引，索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同。
- 索引是通过二叉树的数据结构来描述的，我们可以这么理解聚簇索引：索引的叶节点就是数据节点。而非聚簇索引的叶节点仍然是索引节点，只不过有一个指针指向对应的数据块。
- 聚集索引：物理存储按照索引排序；非聚集索引：物理存储不按照索引排序；

主键一定是聚簇索引，MySQL的InnoDB中一定有主键，即便研发人员不手动设置，则会使用unique索引，没有unique索引，则会使用数据库内部的一个行的id来当作主键索引,其它普通索引需要区分SQL场景，当SQL查询的列就是索引本身时，我们称这种场景下该普通索引也可以叫做聚簇索引，MyisAM引擎没有聚簇索引。

# 15. InnoDB B+Tree结构存储索引

InnoDB使用B+Tree数据结构存储索引，根据索引物理结构可将索引划分为**聚簇索引**和**非聚簇索引**（也可称辅助索引或二级索引）。

**一个表中只能存在一个聚簇索引（主键索引），但可以存在多个非聚簇索引**。

每个InnoDB表都需要一个聚簇索引。该聚簇索引可以帮助表优化增删改查操作。Innobd中的主键索引是一种聚簇索引。

如果你为表定义了一个主键，MySQL将使用主键作为聚簇索引。

如果你不为表指定一个主键，MySQL讲索第一个组成列都not null的唯一索引作为聚簇索引。

如果InnoBD表没有主键且没有适合的唯一索引（没有构成该唯一索引的所有列都NOT NULL），MySQL将自动创建一个隐藏的名字为“`GEN_CLUST_INDEX` ”的聚簇索引。

**Memory引擎有一种特殊功能叫“自适应哈希索引”，InnoDB注意到某些索引值被使用得特别频繁时，则会在内存中基于B+Tree上再创建一个哈希索引，这样就让索引也有哈希索引的一些特点，比如快速的哈希查找。这是一个完全自动的，内部的行为，用户无法控制或者配置，不过如果有必要，完全可以关闭该功能。**

**因此每个InnoDB表都有且仅有一个聚簇索引**。

1) **聚簇索引**

B+树 叶子节点包含数据表中行记录就是聚簇索引（索引和数据是一块的）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131235047821.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70#pic_center)

![img](https://img2018.cnblogs.com/i-beta/1464190/201911/1464190-20191106145200302-932404581.png)

2. **非聚簇索引**

B+树 叶子节点没包含数据表中行记录就是非聚簇索引（索引和数据是分开的）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021013123462610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70#pic_center)

![img](https://img2018.cnblogs.com/i-beta/1464190/201911/1464190-20191106145241480-1330791289.png)

InnoDB 表是基于聚簇索引建立的。因此InnoDB 的索引能提供一种非常快速的主键查找性能。不过，它的辅助索引（Secondary Index， 也就是非主键索引）也会包含主键列，所以，**如果主键定义的比较大，其他索引也将很大**。如果想在表上定义 、很多索引，则争取尽量把主键定义得小一些。InnoDB 不会压缩索引。

# 16. MyISAM B+Tree结构存储索引

MyISAM也使用B+Tree数据结构存储索引，但**都是非聚簇索引**。

以下是MyISAM主键索引存储图

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020100081078.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdmZWlqaXU=,size_16,color_FFFFFF,t_70)

可见，索引和数据是分开的 **索引的data部分只是索引的地址值**。其实上文也提到过，.MYI就是MyISAM表的索引文件，MYD是MyISAM表的数据文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201001937236.png)

​	**非聚簇索引(辅助索引)**

在MyISAM中，主索引和辅助索引（Secondary key）在结构上没有任何区别，**只是主索引要求key是唯一的，而辅助索引的key可以重复**。如果我们在Col2上建立一个辅助索引，则此索引的结构如下图所示：

# 17. InnoDB和MyISAM中的聚簇索引和非聚簇(辅助)索引

innodb中辅助索引保存的是索引值和**主键值**，而在MyISAN保存的是索引值和**行指针**。 

![img](https://img2018.cnblogs.com/i-beta/1464190/201911/1464190-20191106151527647-152458631.png)

# 18. HASH索引

哈希索引就是采用一定的哈希算法，把键值换算成新的哈希值，检索时不需要类似B+树那样从根节点到叶子节点逐级查找，只需**一次哈希算法**即可立刻定位到相应的位置，速度非常快。**Memory存储引擎使用Hash**。

hash索引只保存哈希值和数据指针。

**innoDb利用hash小技巧**:

如当要对url列进行**搜索查找**时，直接对url建立索引查找时会很慢，这时候可以对url先进行CRC32哈希，然后对这个哈希值建索引。搜索时这样搜索会快得多。注意这里url_crc是额外建立的列，用来建立索引。注意如果哈希冲突了，则必须在where条件中带上列值+哈希值。

```sql
select id from url where url="http://www.mysql.com" and url_crc = CRC32("http://www.mysql.com")
```



![image-20210908161043196](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908161043196.png)

**HASH索引本身是不支持范围查询的，但是如使用Memory存储引擎的表还是可以进行范围查询操作，只是此时的HASH索引会失效**

![image-20210908160905548](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20210908160905548.png)

# 19. 回表和索引覆盖

简单来说就是数据库根据索引（非主键）找到了指定的记录所在行后，还需要根据主键再次到数据块里获取数据。

![img](https://upload-images.jianshu.io/upload_images/4459024-a75e767d0198a6a4?imageMogr2/auto-orient/strip|imageView2/2/w/421/format/webp)

粉红色的路径需要扫描两遍索引树，第一遍先通过普通索引定位到主键值id=5，然后第二遍再通过聚集索引定位到具体行记录。这就是所谓的**回表查询**，即**先定位主键值，再根据主键值定位行记录**，性能相对于只扫描一遍聚集索引树的性能要低一些.

**索引覆盖**

索引覆盖是一种避免回表查询的优化策略。具体的做法就是**将要查询的数据作为索引列建立普通索引**（可以是单列索引，也可以一个索引语句定义所有要查询的列，即联合索引），这样的话就可以直接返回索引中的的数据，不需要再通过聚集索引去定位行记录，避免了回表的情况发生。

**索引覆盖（covering index）**指一个查询语句的执行只用从索引中就能够取得，不必从数据表中读取。也可以称之为实现了索引覆盖。 当一条查询语句符合覆盖索引条件时，MySQL只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。 如，表covering_index_sample中有一个普通索引 idx_key1_key2(key1,key2)。当我们通过SQL语句：select key2 from covering_index_sample where key1 = ‘keytest’;的时候，就可以通过覆盖索引查询，无需回表。

如何实现索引覆盖?

常见的方法是：**将被查询的字段，建立到联合索引里去**。

# 20. 索引覆盖优化SQL举例

1. **全表count查询优化**

原表为：

```sql
user(PK id, name, sex)；
```

直接：

```sql
select count(name) from user;
```

不能利用索引覆盖。

添加索引：

```sql
alter table user add key(name);
```

就能够利用索引覆盖提效。

2. **列查询回表优化**

```sql
select id,name,sex ... where name='shenjian';
```

这个例子不再赘述，将单列索引(name)升级为联合索引(name, sex)，即可避免回表。

3. **分页查询**

```sql
select id,name,sex ... order by name limit 500,100;
```

将单列索引(name)升级为联合索引(name, sex)，也可以避免回表。

# 21. 应当建立索引的情况

- 在经常需要搜索的列上，可以加快搜索的速度
- 在作为主键的列上，强制该列的唯一性和组织表中数据的排列结构
- 在经常用在连接（JOIN）的列上，这些列主要是一些**外键**，可以加快连接的速度
- 在经常需要根据范围（<，<=，=，>，>=，BETWEEN，IN）进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的
- 在经常需要排序（order by）的列上创建索引，因为**索引已经排序**，这样查询可以利用索引的排序，加快排序查询时间；
- 在经常使用在WHERE子句中的列上面创建索引，加快条件的判断速度。

# 22. 不应当建立索引的情况

- 对于那些在查询中很少使用或者参考的列不应该创建索引。

    若列很少使用到，因此有索引或者无索引，并不能提高查询速度。相反，由于增加了索引，反而降低了系统的维护速度和增大了空间需求。

- 对于那些只有**很少数据值**或者**重复值多的列**也不应该增加索引。
    这些列的取值很少，例如人事表的性别列，在查询的结果中，结果集的数据行占了表中数据行的很大比例，即需要在表中搜索的数据行的比例很大。增加索引，并不能明显加快检索速度。

- 对于那些定义为**text, image和bit**数据类型的列不应该增加索引。
    这些列的数据量要么相当大，要么取值很少。

- 当该列修改性能要求远远高于检索性能时，不应该创建索引。（修改性能和检索性能是互相矛盾的）

# 23. 最左前缀原则

在MySQL建立联合索引时会遵守最左前缀匹配原则，即最左优先（查询条件精确匹配索引的左边连续一列或几列，则构建对应列的组合索引树），在检索数据时也从联合索引的最左边开始匹配。

---

当我们创建一个组合索引的时候，如 (a1,a2,a3)，相当于创建了（a1）、(a1,a2)和(a1,a2,a3)三个索引，这就是最左匹配原则。

若有order by的场景，则可以通过让order by中的顺序和索引的列顺序一定来达到利用索引排序的效果。

但也有一种情况可以让order by子句不用满足最左前缀原则，即前导列为常量的时候。

如有索引rental_date(rental_date, inventory_id, customer_id)

```mysql
select rental_id, staff_id from sakila.rental where rental_date = '2005-05-25'
	order by inventory_id, customer_id
```

这种查询则还是可以使用索引进行排序。即使order by不满足索引的最左前缀要求，也可以用于查询排序，这是因为索引的第一列被指定为一个常数。

这种也可以

```mysql
where rental_date > '2005-05-25' order by rental_date, inventory_id
```

下面是不可以使用索引排序的查询

```mysql
1. 查询四用了两种不同的排序方向，但是索引列都是正序排序的 
...where rental_date = '2005-05-25' order by inventory_id desc, customer_Id asc;
```



# 24. 前缀索引

有时候需要索引很长的字符列，这会让索引变得大且慢。通常可以以某列开始的部分字符作为索引，这样可以大大节约索引空间，从而提高索引效率。但这样也会降低索引的选择性。索引的选择性是指不重复的索引值和数据表的记录总数的比值，索引的选择性越高则查询效率越高。

以下是一个百万级数据表的简化呈现
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210206144840714.png#pic_center)

# 25. 全文索引

通过数值比较、范围过滤等就可以完成绝大多数我们需要的查询，但是，如果希望通过关键字的匹配来进行查询过滤，那么就需要基于相似度的查询，而不是原来的精确数值比较。全文索引，就是为这种场景设计的，通过建立**倒排索引**,可以极大的提升检索效率,解决判断字段是否包含的问题。

但是实际使用的时候，这个辅助表是存放在磁盘上的，因此innoDB存储引擎不可能每次查询都到磁盘里面查询，因此还有一个全文索引缓存（FTS index cache）,用来提高全文索引的性能，这个全文索引缓存的底层是**红黑树结构**。

# 26. redo log,bin log(二进制日志),undo log

![img](https://img-blog.csdnimg.cn/img_convert/1af0b4169b7d248fdb05e9d1d189b854.png)

### redo log日志模块

redo log是**InnoDB存储**引擎层的日志，又称重做日志文件，**用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来**。在实例和介质失败（media failure）时，redo log文件就能派上用场，如数据库掉电，InnoDB存储引擎会使用redo log恢复到掉电前的时刻，以此来保证数据的完整性。

**在一条更新语句进行执行的时候，InnoDB引擎会把更新记录写到redo log日志中**，然后更新内存，此时算是语句执行完了，然后在空闲的时候或者是按照设定的更新策略将redo log中的内容更新到磁盘中，这里涉及到`WAL`即`Write Ahead logging`技术，他的关键点是**先写日志，再写磁盘**。

有了redo log日志，那么在数据库进行异常重启的时候，可以根据redo log日志进行恢复，也就达到了`crash-safe`。

### binlog日志模块

binlog是属于**MySQL Server**层面的，又称为归档日志，属于逻辑日志，是以二进制的形式记录的是这个语句的原始逻辑，依靠binlog是没有`crash-safe`能力的

### redo log和binlog区别

- **redo log是属于innoDB层面，binlog属于MySQL Server层面的**，这样在数据库用别的存储引擎时可以达到一致性的要求。
- redo log是**物理日志**，记录该**数据页更新的内容**；binlog是**逻辑日志**，记录的是这个更新语句的原始逻辑
- redo log是**循环写**，日志空间大小固定；binlog是**追加写**，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
- binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。

**undo log**

**作用：**

保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读





redo和binlog这两种日志有以下三点不同：
 1、 **redo log** 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
 2、 redo log 是**物理日志**，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
 3、redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

https://www.bilibili.com/read/cv8405294/

# 27. [锁](https://zhuanlan.zhihu.com/p/95207161)

1. **共享锁((shared)S锁，读锁)**

    事务A给数据对象1加上S锁，则事务A**可以读取数据**对象1但是**不能修改**。其他事务也只能再对数据对象加S锁，不能加X锁。

    这保证其他事务**可以读取**数据对象1而**不能修改**。

2. **排它锁(X锁，写锁)**

    事务A给数据对象1加上S锁，则数据对象**可以读取**1对象也**可以修改**1对象。其他事务**不能再对数据对象1加任何锁**。

3. **意向共享锁(IS)与意向排它锁(IX)**

    事务想要获取表中某些记录的共享锁，需要在表上先加共享意向锁。

    事务想要获取表中某些记录的互斥锁，需要在表上先加共享排它锁。

    这两个统称意向锁。是为了支持**Innodb支持多粒度锁**

    <font color=red>**意向锁是表级锁**</font>

    理由:当我们需要给一个加表锁的时候，我们需要根据意向锁去判断表中有没有数据行被锁定，以确定是否能加成功。如果意向锁是行锁，那么我们就得遍历表中所有数据行来判断。如果意向锁是表锁，则我们直接判断一次就知道表中是否有数据行被锁定了。所以说将意向锁设置成表级别的锁的性能比行锁高的多。

    有了意向锁之后，前面例子中的事务A在申请行锁（写锁）之前，数据库会自动先给事务A申请表的意向排他锁。当事务B去申请表的写锁时就会失败，因为表上有意向排他锁之后事务B申请表的写锁时会被阻塞。

    所以，**意向锁的作用**就是：

    当一个事务在需要获取资源的锁定时，如果该资源**已经被排他锁占用**，则数据库会自动给该事务**申请一个该表的意向锁**。

    如果自己需要一个**共享锁**，就申请一个**意向共享锁**。如果需要的是**某行（或者某些行）的排他锁**，则申请一个**意向排他锁**。

4. **乐观锁**

    乐观锁不是数据库自带的，需要我们自己去实现。乐观锁是指操作数据库时(更新操作)，想法很乐观，认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理（也就是不加锁），而在进行更新后，再去判断是否有冲突了。

5. **悲观锁**

    悲观锁，正如其名，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）

6. **间隙锁(Gap Locks)**

    一般是针对**非唯一索引**而言的，间隙锁（Gap Lock）是Innodb在<font color=red>可重复读</font>提交下为了**解决幻读问题**时引入的锁机制。([这里MVCC还是可能会出现幻读情况](#hd))

7. **next-key lock**

    记录锁和间隙锁的结合，同样用来解决**幻读问题**(不能完全解决)，对于InnoDB中，更新非唯一索引对应的记录。会加上Next-Key Lock。如果更新记录为空，就不能加记录锁，只能加间隙锁。

# 28. 快照读，当前读

**当前读**, 读取的是最新版本, 并且**对读取的记录加锁, 阻塞其他事务同时改动相同**记录，避免出现安全问题。如update，这也是MVCC会幻读的原因。

快照读，单纯的select操作，**不包括**上述 select ... lock in share mode, select ... for update。　　　　

- Read Committed隔离级别：每次select都生成一个快照读。
- Read Repeatable隔离级别：**开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。**



# 29. join

1. Left join
2. right join
3. outer join

 

# 30. 联合索引底层结构怎么样

![Selection_003](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/Selection_003.png)

这里同样使用b+树，不过排序时按照字段的顺序来，先比较第一个字段的大小，再比较第二个...依次类推来排序的。

**这也是最左前缀原则**，若不使用最左边的字段，而直接使用后面的，而可以看到其实在索引中这几个字段并不是有序的。比如跳过name字段，直接使用age字段，如where age = 30，可以看到即使定位到age30这个节点，后面的age也并不是有序的。若此时使用where name='Bill' and age > 30, 则发现在name为'bill'的节点中，age是有序的。此时可以使用**索引查找**。

# 31. ACID如何保证

A原子性由undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql

B一致性由其他三大特性来保证，程序代码要保证业务上的一致性

I隔离性由MVCC来保证

D持久性由**内存+redo log**来保证，mysql修改数据同时在内存和redo log记录这次操作，宕机时可以从redo log恢复。

```shell
InnoDB redo log 写盘， InnoDB事务进入prepare状态
如果前面prepare成功，bin log写盘，再继续将事务日志持久化到binlog,如果持久化成功。那么InnoDB事务则进入commit状态(在redo log里面写一个commit 记录)
```

redolog的刷盘会在系统空闲时进行。

# 32. MYSQL主从同步

![image-20211004000114393](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20211004000114393.png)

![image-20211004000142576](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/image-20211004000142576.png)

# 33. MVCC版本链

https://blog.csdn.net/weixin_41582192/article/details/111545118

# 34. 高性能索引策略

- **独立的列**

    索引不能是表达式的一部分，也不能是函数的参数。

    如下列查询无法使用`actor_id`索引

    应该尽量把索引单独放在一边

    ```mysql
    mysql> select actor_id from sakila.actor where actor_id + 1 = 5;
    mysql> select ... where TO_DAYS(CURRENT_DATE) - TO_DAYS(date_col) <= 10;
    ```

- **前缀索引和索引选择性**

    索引很长的字符列

    (1) 给字符列增加一个哈希字段列，模拟哈希查询

    (2) 索引最开始的部分字符(前缀索引)

    如何确定前缀索引的长度？

    看下述例子

    ```mysql
    mysql> select count(distinct city)/count(*) from sakila.city_demo; # 计算城市区分对总数的占比
    +-----------------------------+
    |COUNT(DISTINCT city)/COUNT(*)|
    +-----------------------------+
    |					  0.0312|
    +-----------------------------+
    
    # left是city字符串串的前多少个
    mysql> select count(distinct LEFT(city, 3))/count(*) AS sel3,
    	-> count(distinct LEFT(city, 4))/count(*) AS sel4,
    	-> count(distinct LEFT(city, 5))/count(*) AS sel5,
    	-> count(distinct LEFT(city, 6))/count(*) AS sel6,
    	-> count(distinct LEFT(city, 7))/count(*) AS sel7,
    	-> from sakila.city_demo;	#计算各种长度，看对城市的区分度的占比，这里可以看到长度为4或5就已经很接近了
    +----------------------------------+
    | sel3 | sel4 | sel5 | sel6 | sel7 |
    +----------------------------------+
    |0.0239|0.0293|0.0305|0.0309|0.0310|
    +----------------------------------+
    ```

    注意这种情况如果数据分布不均匀会有陷阱

    ```mysql
    mysql> select count(*) as cnt, left(city, 4) as pref
    	-> from sakila.city_demo group by pref order by cnt desc limit 5;
    +-----+------+
    | cnt | pref |
    +-----+------+
    | 205 | San  |
    | 200 | Sant |
    | 135 | Sout |
    | 104 | Chan |
    | 91  | Toul |
    +-----+------+
    ```

    这里可以看到Sant和San有重复的部分。

- **多列索引(联合索引)**

- **选择合适列顺序**

    将选择性高的放前面，举例

    ```mysql
    mysql> select * from payment where staff_id = 2 and customer_id = 584;
    ```

    这里要创建索引(staff_id, customer_id)还是(customer_id, staff_id)呢？应该得看哪个列的选择性更高。

    ```mysql
    mysql> select sum(staff_id = 2), sum(customer_id = 584) from paymeng\G;
    
    SUM(staff_id = 2): 7992
    SUM(customer_id = 584): 30
    ```

    可以看到customer_id的过滤性更高，即数量更少，所以应该是(customer_id, staff_id)

    **当然这么优化非常依赖具体值**。
    
- **innodb中按顺序插入行**

    最简单就是使用auto_increment。避免随机插入，比如使用UUID来做聚簇索引，性能会很低，因为这使得插入完全随机。 

    这种随机插入的缺点:

    - 写入的目标可能已经刷到磁盘上并从缓存中移除，或者是还没有被加载到缓存中，InnoDB在插入之前不得不找到并从磁盘上读取目标页到内存中。这将导致大量的随机I/O。

    - 因为写入是乱序的，InnoDB不得不频繁地做页分裂操作，以便为新的行分配空间。**页分裂**会导致移动大量数据，一次插入最少需要修改三个页而不是一个页。

        > **页分裂(page split)**，当行的主键值要求必须将这一行插入到某个已满的页中时，存储引擎会将该页分裂成两个页面来容纳该行，这就是一次页分裂操作。页分裂会导致表占用更多的磁盘空间。

    - 由于频繁的页分裂，页会变得稀疏并被不规则地填充，所以最终数据会有碎片。

    这些随机插入后可以使用OPTIMIZE TABLE来重建优化。
    
- **索引使用技巧**

    1. 比如对于(sex, country)索引，我们可以使用set in('m', 'f')条件从而得到使用sex索引的目的

    2. 尽可能把范围查询放在后面，比如(sex, country, region, age)中age一般是基于范围查询的。**<font color=red>查询只能使用索引的最左前缀，直到遇到第一个范围条件列</font>**。

        当然这种情况也可以使用age in(xx,xx,xx...)形式，但是不能滥用，因为每增加一个`IN()`条件，优化器需要做的组合就以指数形式增加，最终降低查询性能。如

        ```mysql
        where eye_color IN('brown','blue','hazel')
        	AND hair_color IN('black', 'red', 'blonde', 'brown')
        	AND sex	IN('M', 'F')
        ```

        优化器会转化为4\*3\*3=24种组合。

- **避免多个范围条件**

    ```mysql
    where eye_color IN('brown','blue','hazel')
    	AND hair_color IN('black', 'red', 'blonde', 'brown')
    	AND sex	IN('M', 'F')
    	AND last_online > DATE_SUB(NOW(), INTERVAL 7 DAY)	
    	AND age BETWEEN 18 AND 25
    ```

    这里在使用了last_online范围查询后，age的范围查询就不能使用索引了。因此last_online可以用active字段代替，0代表7周内没上线，1代表有，这种做法active列并不是完全精确的，但它可以让age范围查询走到索引。

- **limit使用索引覆盖优化**

    https://juejin.cn/post/6844903939247177741

# 35. optimize table

使用mysql的时候，可能会发现尽管一张表删除了许多数据，但是这张表的数据文件和索引文件大小却没有变小。这是因为mysql在删除数据(特别是有Text和BLOB)时，会在原来被删除的位置留下数据空洞，等待新数据来填充。这种空洞不仅额外增加了存储代价，而且还因为数据碎片化降低了表的扫描效率。因此可以通过optimize table + 表名 来优化去除这些空洞。

注意使用时mysql会锁表。并且该命令只对myisam,bdb和innodb起作用。

# 36. [innodb_autoinc_lock_mode调优](https://www.cnblogs.com/widgetbox/p/10178035.html)(不重要)

https://blog.csdn.net/corleone_4ever/article/details/106700787

https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html#innodb-auto-increment-initialization

1. 三种insert

    - **simple insert**

        insert时就可以知道插入的行数量，包括没有嵌套的单行或多行的insert，replace，但不包括insert...onduplicate key update.

    - **bulk inserts**

        插入时不知道要插入多少条数据，包括insert ... select, replace..select, load data语句。innodb在处理自增行(auto_increment)时，每次都会分配一个新值。

    - **Mixed-mode inserts**

        插入时制定了部分(不是全部)新行的auto_increment值。如

        ```mysql
        INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');
        ```

2. **`innodb_autoinc_lock_mode`三种模式**

    首先，该参数仅在InnoDB引擎下生效，myisam引擎下，无论什么样自增id锁都是表级锁
    0：traditonal （每次都会产生表锁）
    1：consecutive （mysql的默认模式，会产生一个轻量锁，simple insert会获得批量的锁，保证连续插入）
    2：interleaved （不会锁表，来一个处理一个，并发最高）

    **如何理解，以及参数如何选择？**

    1、innodb_autoinc_lock_mode为0时的，也就是官方说的traditional级别

    该自增锁是**表锁级别**，且**必须等待当前SQL执行完成后或者回滚掉才会释放**，这样在高并发的情况下可想而知自增锁竞争是比较大的。

    这种模式下，所有的insert语句在开始时都会获得一个表锁autoinc_lock.该锁会一直持有到insert语句执行结束才会被释放。对于一条insert插入多个行记录的语句，他保证了同一条语句插入的行记录的自增ID是连续的。
    **这个锁并不是事务级别的锁**。
    在这种模式下，**主从复制**时，**基于语句复制模式**下，主和从的同一张表的同一个行记录的自增ID是一样的。但是同样基于语句复制模式下，interleaved模式，也就是innodb_autoinc_lock_mode=2时则不能保证主从同一张表的同一个行记录的自增ID一样。
    这种模式下，表的并发性最低。

    2、innodb_autoinc_lock_mode为1时的，也就是官方说的consecutive级别

    这种模式下，insert语句在开始时会获得一个表锁autoinc_lock, simple insert在获取到需要增加的ID的量后，autoinc_lock就会被释放,不必等到语句执行结束。但对于bulk insert，自增锁会被一直持有直到语句执行结束才会被释放。

    3、innodb_autoinc_lock_mode为2时，也就是官方说的interleaved 级别

    这种模式下，simple insert语句能保证ID是连续的，但是bulk insert的ID则可能有空洞。
    主从复制的同一张表下的同一行id有可能不一样。

# 37. 避免死锁

**如何避免：**

- 尽量使用主键更新数据，防止使用非聚簇索引回表时和使用聚簇索引冲突造成死锁。
- 在允许幻读和不可重复度的情况下，尽量使用RC的隔离级别，避免gap lock造成的死锁。
- 避免长事务，将事务拆解
- 设置锁超时等待`innodb_lock_wait_timeout`

# 38. 数据库设计

满足三个范式

1NF：无重复的列，属性不可以拆分（强调列的原子性，比如家庭电话和个人电话需要拆开）

2NF：属性完全依赖于主键

（老师，班级）

（老师）=》性别

（班级）=》课程

性别和课程只单独依赖于主键的一部分。

做法: .1 性别和课程拆分出来

​			2. 使用逻辑主键，如自增的id

3NF：属性不传递依赖于其他非主属性

（ID)=>（老师）=》（班级）=》（教室）

教室传递依赖于班级

班级依赖于老师

老师依赖于主键ID

做法：拆分班级和教室到单独的表
