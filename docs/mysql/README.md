
# 索引

### `MySQL` 为什么使用 `B+` 树来作索引，对比 `B` 树它的优点和缺点是什么？

##### `B+`树和`B`数的定义？

- **`B`树定义**：`B`树是平衡搜索多叉树，设树的度为`2d(d>1)`，高度为`h`，那么`B`树要满足条件：
  - 每个叶子节点的高度一样，等于`h`
  - 每个非叶子结点由`n-1`个`key`和`n`个指针`point`组成，其中 `d<=n<=2d`，`key` 和 `point` 相互间隔，结点两端一定是 `key`
  - 叶子节点指针都为 `null`
  - 非叶子节点的`key`都是`[key, data]`二元组，其中`key`表示作为索引的键，`data`为键值所在行的数据
- **`B+`树定义**：`B+`树是`B`树的一个变种，设`d`为树的度数，`h`为树的高度，`B+`树和`B`树的不同主要在于：
  - `B+`树中的非叶子节点不存储数据，只存储键值
  - `B+`树的叶子节点没有指针，所有键值都会出现在叶子结点上，且`key`存储的键值对应`data`数据的物理地址
  - `B+`树的每个非叶子节由`n`个键值`key`和`n`个指针`point`组成

##### `B+`树和`B`树的区别？

- 查找结点的时间复杂度
  - `B`树非叶子结点和叶子结点都存储数据，因此查询数据时，时间复杂度最好为`O(1)`，最坏为`O(logn)`
  - `B+`树只在叶子结点存储数据，非叶子结点存储关键字，且不同非叶子结点的关键字可能重复，因此查询数据时，时间复杂度固定为`O(logn)`

- 查找方法
  - `B+`树叶子结点之间用链表相互连接，因而只需扫描叶子结点的链表就可以完成一次遍历操作
  - `B`树只能通过中序遍历

##### 为什么`B+`树比`B`树更适合应用于数据库索引？

- `B+`树更加适应磁盘的特性，相比`B`树减少了`I/O`读写的次数
  - 由于索引文件很大因此索引文件存储在磁盘上，`B+`树的非叶子结点只存关键字不存数据，因而单个页可以存储更多的关键字，即一次性读入内存的需要查询的关键字也就越多，磁盘的随机`I/O`读取次数相对就减少了
- `B+`树的查询效率相比`B`树更加稳定
  - 由于数据只存在叶子结点上，所以查找效率固定为 `O(logn)`
- `B+`树利于扫库和范围查询
  - `B+`树叶子结点之间用链表有序连接，所以扫描全部数据只需扫描一遍叶子结点
  - `B`树由于非叶子结点也存数据，所以只能通过中序遍历按序来扫描。也就是说，对于范围查询和有序遍历而言，`B+`树的效率更高

##### 拓展：`B+ Tree VS ` 红黑树

- 更少的查找次数：红黑树是二叉树，导致同样的数量的`data`，红黑树的高度会大于 `B+ Tree`
- 减少磁盘寻道：因为磁盘不是严格读取数据，而是会有预读的情况。顺序读取的过程中不需要磁盘寻道，速度更快

------

### 唯一索引与普通索引的区别是什么？使用索引会有哪些优缺点？

##### 区别

- 唯一索引和普通索引的区别是唯一索引定义了唯一性
- 从查询性能上对比，假如查询`k=5`的记录：
  - 对于普通索引来说，查到满足的第一个记录后，需要查找下一条记录，直到碰到第一个不满足`k=5`的条件，查询结束
  - 对于唯一索引，查到满足的第一个记录后，即可返回，查询结束
  - `InnoDB` 对于一条记录并不是将这条记录从磁盘中读出来，而是以页为单位，将整体读取到内存中，每个数据页的大小是`16kb`，那么对于一个`int`索引来说，一个数据存储的索引数据近千条，普通索引查询下一条需要读取两个数据页的几率很小。
  - 因此唯一索引和普通索引在查询的性能上差别不大
- 从更新性能上对比：
  - 当需要更新一个数据页时，如果数据页在内存中就直接更新
  - 如果数据页不在内存中，在不影响数据一致性的情况下，`InnoDB`会将这些更新操作缓存到 `change buffer`中，这样不需要从磁盘中读取数据页，减少磁盘读取，等到下一次查询时，先将原数据页的数据查到，然后再进行`merge`
  - 唯一索引，所有更新的操作都需要先判断操作是否违反了唯一性，所以必须将数据页读入内存才能判断，不能用到 `channge buffer`
  - 因此，在更新操作上，普通索引优于唯一索引

##### 使用索引会有哪些优缺点？

- 优点
  - 创建唯一性索引，保证数据库表中该列的唯一性
  - 大大加快数据的检索速度
  - 加速表和表之间的连接
  - 在使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间
  - 通过使用索引，可以在查询的过程中使用优化隐藏器，提高系统的性能，例如：`COUNT(*)`

- 缺点
  - 创建和维护索引要耗费时间，这种时间随着数量的增加而增加
  - 索引需要占物理空间，除了数据表占数据空间外，每一个索引还要占一定的物理空间
  - 当对表中的数据进行增加、删除和修改时，索引也要动态维护，降低了数据的维护速度

------

### 数据库有哪些常见索引？

##### 从数据结构角度

- `B+Tree`索引：`MySQL`默认索引，可用于查找、分组、排序
- `Hash`索引：基于哈希表实现，适合新增和等值查询，不是有序的，范围查询很慢，必须全表扫描；
- `FULLTEXT`索引:大量文件检索时，需要用全文索引，因为它的速度是 `like`的 `N`倍(`MySQL 5.6`版本`InnoDB`支持)
- `R-Tree`索引：空间数据索引会从所有维度来索引数据，主要用于地理数据存储。

##### 从物理存储角度

- 聚簇索引：叶子结点存放一整行记录
- 非聚簇索引：叶子节点存放索引+指向行数据的指针

##### 从逻辑角度

- 主键索引：是一种特殊的唯一索引，不允许有空值
- 普通索引/单列索引
- 复合索引/多列索引：多个字段上创建的索引，在查询条件中，使用第一个字段，所以才会被使用，符合最左原则
- 唯一索引：该列值不能重复

------

### 数据库索引的实现原理是什么？

- 索引是一个排序的列表，在这个列表中存储着索引的值和包含这个值的数据所在行的物理地址，在数据十分庞大的时候，索引可以大大加快查询的速度，这是因为使用索引后可以不用扫描全表来定位某行的数据，而是先通过索引表找到该行数据对应的物理地址然后访问相应的数据。

------

### 聚簇索引和非聚簇索引有什么区别？什么情况用聚集索引？

#### 区别

- 聚簇索引与非聚簇索引的区别是：叶节点是否存放行数据

##### 聚簇索引

- 聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。比如：`InnoDB`的聚簇索引使用`B+Tree`的数据结构存储索引和数据
- 当表有聚簇索引时，它的数据行实际存放在索引的叶子页。因为无法同时把数据行存放在两个不同的地方，所以一个表只能由一个聚簇索引。
- 聚簇索引的二级索引：叶子节点不会保存引用的行的物理位置，而是保存行的主键位置

##### 非聚簇索引

- 表数据存储顺序与索引顺序无关，叶结点包含索引字段值及指向数据页数据行的逻辑指针，其行数量与数据表行数据量一致。

##### 聚簇索引的优点

- 可以把相关数据保存在一起
- 访问数据更快
- 使用覆盖索引扫描的查询可以直接使用页借点的主键值

##### 聚簇索引的缺点

- 聚簇索引数据最大限度地提高`IO`密集性应用的性能，但如果把数据全部放在内存中，则访问的顺序没那么重要，聚簇索引就没什么优势了
- 插入速度严重依赖插入顺序
- 更新聚簇索引列的代价很高
- 基于索引的表插入新行，或主键被更新导致需要移动行的时候，可能面临页分裂
- 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据存储不连续

#### 使用场合

##### 聚簇索引的使用场合

- 查询命令的回传结果是以该字段为排序依据的
- 查询的结果返回一个区间值
- 查询的结果返回某值相同的大量结果集

##### 非聚簇索引的使用场合

- 查询所获数据量较少时
- 某个字段中的数据的唯一性比较高时

#### 拓展：`InnoDB`的主键列

- 如果没有定义主键，`InnoDB`会选择一个唯一的非空索引代替
- 如果没有这样的索引，`InnoDB`会隐式定义一个主键作为聚簇索引

#### 拓展：为什么主键通常建议使用自增 `id`

- 聚簇索引的数据的物理存放顺序与索引的顺序一致。即：只要索引是相邻的，那么对应的数据一定也是相邻地存放在磁盘上
- 如果主键不是自增 `id`，那么需要不断调整数据的物理地址、分页等；
- 如果是自增的，它只需要一页一页地写，索引结构相对紧凑，磁盘碎片少，效率也高
- 
------

### 索引覆盖、最左原则、索引下推

#### 简述什么是覆盖索引？

##### 概念

- 覆盖索引指的是，在使用索引查询时，需要查询的值已经在索引树上，因此可以直接提供查询结果，不需要回表。
- 由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段

##### 例子

```sql
/* id 为主键，c 为索引 */
SELECT id FROM T WHERE k BETWEEN 3 AND 5;
```

- 因为索引`k`的数据时`k+id`，包含了需要查询的值，不需要再回表，所以符合覆盖索引查询。


#### 简述什么是最左匹配原则？

##### 概念

- 最左匹配原则是指，在执行查询过程中，不需要索引的全部定义，只需要满足最左前缀，就可以利用索引来加速检测。
- 最左前缀可以是联合索引的最左`N`个字段，也可以是字符串索引的最左`M`个字符。

##### 例子

```sql
/* 联合索引，a, b, c */
SELECT * FROM T WHERE A=val1 /* A 符合最左匹配 */
SELECT * FROM T WHERE A=val1 AND B=val2 /* AB 符合最左匹配 */
SELECT * FROM T WHERE A=val1 AND C=val3 /* 该SQL会选择 A 作为索引去查询 */
SELECT * FROM T WHERE B=val1 AND C=val3 /* 没有用到索引，不符合 */

/* 假设 a 是字符串 */
SELECT * FROM T WHERE A="A%" /* 也是符合最左匹配原则的 */
```

##### 拓展

- 一个索引可以最多包含16列
- 取字符串的最左部分字符可以作为前缀索引，但是不能使用覆盖索引，因为索引查询时必须回表。


#### 假设建立联合索引 (a, b, c) 如果对字段 a 和 c 查询，会用到这个联合索引吗？

- 准确来说，查询字段`(a, c)` 的话，其中一部分字段会用到联合索引，也就是字段`a`，但`c`的部分没有使用索引。

- 从定义来说，我们说查询用到联合索引的话，一般指所有字段的查询都用到了该索引。也就是 `(a), (a,b),(a,b,c)` 这三种情况

  
#### 简述什么是索引下推？

##### 概念

索引下推是 `MySQL5.6` 引入的特性，指在索引遍历的过程中，在符合最左匹配原则情况下，对索引中包含的字段先进行判断，直接过来掉不满足条件的记录，减少回表次数。

##### 例子

```sql
/* 联合索引 (name, age) */
SELECT * FROM T WHERE name like 'A%' AND age = 10
```

- 在无索引下推的情况下，找到符合 `like A%` 的行，直接回表查主键的行记录，然后在判断 `age=10`
- 在有索引下推的情况下，先判断 `age = 10，再回表。

------

### `MySQL` 的索引什么情况下会失效？

- 联合索引，没有使用最左匹配原则
  -例如：联合索引`(a, b)`，`SELECT * FROM T WHERE b = 1`
  - 因为索引都是先基于`a`排序的，`a`相同的情况，再基于`b`排序，所以查询条件没有`a`，对于索引来说，`b`是无序的，所以索引失效；

- 范围查询，右边失效
  -  例如：联合索引`(a, b)`, `SELECT * FROM T WHERE a > 1 AND b = 1`
  -  字段`a` 在 `B+ Tree`上是有序的，可以用二分查找去定位，`a`的索引能被用到，`b`有序的前提是 `a`是确定值，在之前逻辑中，`b`是无序的，所以`b`用不到索引

- `like`的 `%` 放在左边，索引失效
  - 例如：`SELECT * FROM T WHERE a LIKE "%a"`
  - 字符串的排序方式是，先按照第一个字母排序，如果第一个字母相同，就按照第二个字母排序，依次类推

- 条件字段做函数操作，导致索引失效
  - 对索引字段做函数操作，可能会破坏索引的有序性，优化器一刀切，不管有没有破坏有序性，都不会考虑使用索引
  
- 隐式转换，导致索引失效
  - 数据类型转换，如果字符串和数字比较，则将字符串转换成数字
  - 如果条件条件字段是字符串，查询条件是数字，则会隐式的将字符串转为数字，即对字段做函数操作

------

# 事务

### 数据库的事务隔离级别有哪些？各有哪些优缺点？

##### 数据库的事务隔离级别有哪些？

事务隔离级别主要有四种：

- 读未提交(`READ UNCOMMITED`)
  - 定义：一个事务可以读取另一个事务已修改但未提交的数据
  - 存在问题：脏读
- 读已提交(`READ COMMITED`)
  - 定义：一个事务只能读取另一个事务已经提交的数据
  - 存在问题：不可重复读
- 可重复读(`REPEATABLE READ`)，`MySQL` 默认隔离级别
  - 定义：在一个事务中多次读取同一条记录，结果一致，无论其他事务是否对这条记录做了修改
  - 存在问题：幻读
- 串行化(`SERIALIZABLE`)
  - 定义：所有事务顺序执行
  - 不存在脏读、不可重复读、幻读等问题

##### 各有哪些优缺点？

- 隔离级别从上到下，并发性能越来越差，但对于数据的隔离性一致性保证程度越好

##### 脏读定义

- 一个事务读到另一个事务已修改未提交的数据，如果前一个事务回滚或修改之前的值，读到的数据就是错误的

##### 不可重复读定义

- 事务`A`修改某条数据，事务`B` 在事务`A`提交前读取到的数据和提交后读取到的数据不一致

##### 幻读定义

- 在可重复度隔离级别下，普通的查询时快照度，是不会看到别的事务插入的数据的，幻读在当前读下才会出现，幻读仅专指新插入的行

------

### 什么是数据库事务，MySQL 为什么会使用 InnoDB 作为默认选项

------

### 简述脏读和幻读的发生场景，`InnoDB` 是如何解决幻读的？

##### 脏读发生的场景

- `MySQL`在读未提交的事务隔离级别下，可能会发生脏读
- 当把事务的隔离级别提升到读提交，则可以避免脏读

##### 幻读发生的场景

- 在可重复度隔离级别下，普通的查询时快照度，是不会看到别的事务插入的数据的，因此幻读在当前读下才会出现
- 幻读仅专指新插入的行，更新或删除的行在当前读下出现，不算幻读
- 当前读出现的方式
  - `update` 和 `delete` 语句
  - `select` 加读锁 `lock in share mode`
  - `select` 加写锁 `for update`

##### `InnoDB` 是如何解决幻读的？

- 产生幻读的原因是，行锁只能锁住行，但是新插入记录这个动作，要更新的是记录之间的间隙
- 为了解决幻读的问题，`InnoDB` 在引入间隙锁，间隙锁锁住了记录之间的间隙，在事务的可重复度隔离级别下，间隙锁才会生效
- 间隙锁和行锁合称临键锁( `Netx-Key Locks`)，是`InnoDB` 解决幻读的手段

------

### 并发事务会引发哪些问题？如何解决？

------

# 锁

### 简述乐观锁以及悲观锁的区别以及使用场景

------

### 什么情况下会发生死锁，如何解决死锁？

------

### 简述 `MySQL` 的间隙锁

- 间隙锁，顾名思义，锁的就是两个值之间的空隙，`InnoDB` 是为了解决幻读而引入的新锁
- 间隙锁是在可重复隔离级别下才会生效，如果把事务的隔离级别设置未读提交，就没有间隙锁
- 间隙锁之间不互锁，因为它们都是保护间隙，不允许锁住的间隙里插入值
- 间隙锁和行锁合称临键锁( `Netx-Key Locks`)，每个间隙锁是开区间的，`Netx-Key Locks` 是前开后闭区间

------

# 应用

### `MySQL` 有什么调优的方式？

------

### 简述 `MySQL` 的主从同步机制，如果同步失败会怎么样？

------

### 简述数据库中什么情况下进行分库，什么情况下进行分表？

------

### 什么是 `SQL` 注入攻击？如何防止这类攻击？

------

### `MySQL` 中 `join` 与 `left join` 的区别是什么？

------

### 数据库查询中左外连接和内连接的区别是什么？

------

### 模糊查询是如何实现的？

------

### `SQL`优化的方案有哪些，如何定位问题并解决问题？

------

# 特性

### 简述数据库中的 `ACID` 分别是什么？

- `ACID`是指数据库在写入或更新资料的过程中，为了保证事务时正确可靠的，所必须具备的四个特性：`A`代表原子性，`C`代表一致性，`I`代表隔离性，`D`代表持久性
- `A`：(`Atomicity`)原子性
  - 一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节；
  - 事务在执行过程中发生错误，会被回滚(`Rollback`)到事务开始前的状态；
- `C`:(`Consistency`)一致性
  - 在事务开始之前和事务结束以后，数据库的完整性没有被破坏；
- `I`:(`Isolation`)隔离性
  - 数据库允许多个并发事务同时对其数据进行读写和修改，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据不一致。
  - 事务的隔离分为不同级别：读未提交、读提交、可重复读、串行化
- `D`:(`Durability`)耐用性
  - 事务处理结束后，对数据的修改是永久的，即使系统故障也不会丢失。

------

### MySQL 有哪些常见的存储引擎？它们的区别是什么？

------

### 数据库设计的范式是什么？

- 第一范式：每个列都不可以再拆分
- 第二范式：在第一范式基础上，非主键列完全依赖于主键，而不能依赖主键的一部分
- 第三范式：在第二范式基础上，非主键只依赖主键，不依赖于其他非主键

------

### 数据库反范式设计会出现什么问题？

------
### `MySQL` 中 `varchar` 和 `char` 的区别是什么？

------

# 持久化

### 简述 MySQL 三种日志的使用场景

------

### 简述 undo log 和 redo log 的作用

------

### 简述 `MySQL MVCC` 的实现原理

##### 概念

- `MVCC`，多版本并发控制。在`MySQL InnoDB` 中主要是为了提高数据库并发性能，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读

// TODO

##### 拓展：当前读

- 概念
  - 读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁
- 当前读使用场景
  - `select lock in share mode`(共享锁)
  - `select for update`(排他锁)
  - `update、insert、delete`

##### 拓展：快照读

- 不加锁的 `select`操作就是快照读，即不加锁的非阻塞读；
- 当事务级别是串行级别下的快照读退化成为当前读；
- 快照读的实现是基于多版本并发控制(`MVCC`)，快照读可能读到的并不是数据的最新版本

------

### 数据库的读写分离的作用是什么？如何实现？

------

### 简述什么是两阶段提交？

------
