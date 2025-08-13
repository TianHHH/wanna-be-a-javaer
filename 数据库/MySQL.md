# 内连接; 左连接和右连接的区别

内连接 (INNER JOIN)

- 只返回左右表中**<font style="color:blue">都满足连接条件</font>**的匹配行, 也就是说两张表都存在对应记录时, 才返回结果

左连接 (LEFT JOIN / LEFT OUTER JOIN)

- **左连接会返回左表的所有行; 如果某一行在右表中能根据ON条件找到对应的匹配行就合并显示**; **<font style="color:blue">如果找不到匹配行, 则右表的字段用`NULL`补上</font>**

右连接 (RIGHT JOIN / RIGHT OUTER JOIN)

- 右连接就是返回右表的所有行; 如果某一行在左表中能根据`ON`条件找到对应的匹配行就合并显示; 如果找不到匹配行, 则左表中的字段用 NULL 补上

----

再补充一个全外连接 (FULL OUTER JOIN)

**<font style="color:blue">注: MySQL 不支持全外连接, 需要使用 UNION 来模拟</font>**

> 把左表和右表中能通过`ON`条件配对的行合并成一行
>
> **对于不能配对成功的行(即孤立行), 仍然保留**, 但未匹配一侧的列补`NULL`

**全外连接仍然基于 ON 条件做匹配, 只不过不舍弃任何一边的未匹配记录**; 而笛卡尔积根本不管匹不匹配, 直接组合所有行

# 线上SQL执行比较慢怎么排查

**先用慢查询日志判断是哪些SQL执行比较慢, <font style="color:blue">它会默认记录执行时间超过10秒的语句, 这个时间由一个叫 long_query_time 的参数控制</font>**, 可以修改它来调整"慢查询"的判定标准

**但是<font style="color:blue">MySQL的慢查询日志是默认关闭的</font>, 需要手动开启**

一种方式是临时开启, 通过命令的方式设置

```sql
# 设置后新开的连接中慢查询日志就会生效, 但是服务重启后就失效
SET GLOBAL slow_query_log = ON; -- 也可以设为1
SET GLOBAL long_query_time = 1; -- 单位是秒
```

**另一种就是永久开启, 通过写入配置文件的方式设置**

编辑 `/etc/my.cnf`(Linux) 或  `my.ini`(Windows)添加

```ini
slow_query_log = 1
long_query_time = 1
# 慢日志文件默认存储在 /var/lib/mysql 文件夹中
slow_query_log_file = /var/log/mysql/slow.log  # 或你想放的位置
```

**<font style="color:blue">设置完成后通过 `systemctl restart mysqld` 命令重启MySQL服务器</font>** (Windows系统就使用 `net stop / start mysql80` 的方式重启)

+ **配置完成后, 慢查询日志就只会记录执行时间超过预设时间的SQL**. 然后通过慢查询日志就可以定位出执行效率比较低的SQL, 从而有针对性的进行优化



因为有了具体的SQL语句之后, 就可以通过 explain 命令来分析为什么查询的慢

通过 explain 命令可以得知 MySQL 是如何执行该 SQL 语句的信息, 比如是不是走了全表扫描; 用没用索引以及用了哪些索引; 以及扫了多少行等

## 讲一下explain

参考博客: https://blog.csdn.net/qq10250507350/article/details/140027211

EXPLAIN 是 MySQL 提供的一个命令, **<font style="color:blue">用于查看 SQL 语句的执行计划</font>**

**其中有几个需要重点关注的列**

- type: 表示 MySQL 在执行这条SQL时访问表的方式, 也可以理解为**数据访问策略**. 性能由好到坏的取值依次是

  - NULL: 不访问任何表 (比如 `select 1`)
  - SYSTEM: 表中只有一行数据或者是空表, 不需要匹配条件就可以直接返回
  - CONST: 单表查询且表中最多只有一条匹配记录 **(即主键或唯一索引的等值匹配)**
  - **EQ_REF: 多表连接时**, 使用主键或者非空唯一键进行等值匹配, 每次只匹配一行
  - **<font style="color:blue">REF: 使用普通索引进行等值匹配, 可能匹配多行</font>** (没有主键和唯一索引的要求)
  - **RANGE: 用索引进行范围扫描, 比如 BETWEEN、> 、<、IN 等**
  - INDEX: 全索引扫描. 当查询的字段全部来自某个二级索引(即覆盖索引), 但是没有 WHERE 过滤条件, 读取数据时**是从"二级索引"中按顺序一行一行读的**, 性能上略优于 ALL
  - ALL: 全表扫描, 说明索引没用上或压根没有索引. 出现 ALL 时一般需要考虑加索引或改写 SQL 了



- **<font style="color:blue">key: 实际使用的索引</font>**. 如果是 NULL, 说明 SQL 的写法让索引失效了



- key_len: 表示在执行 SQL 时, **实际用于索引查找的字段所占字节数, <font style="color:blue">这个值是索引字段的最大可能长度</font>**, 并非实际使用长度 (当字段允许为 NULL 时, 会比不允许的情况大 1 个字节)

  - **<font style="color:blue">该字段只计算 where 条件用到的索引长度</font>**, 排序和分组就算用到了索引也不会计算到该字段中

  - **一般用于判断联合索引中有多少字段被命中了**



- rows: MySQL 预计需要扫描多少行数据, 才能完成这一步查询操作. 这是一个估算值, 可能并不总是准确的, 但即使这样它也是**判断 SQL 是否"扫了太多行"的关键指标**



- Extra: 提供了**关于查询执行过程的附加信息**, 常见的取值有:

  - Using where: 查询需要根据 WHERE 子句对数据进行进一步的过滤, 通常是因为索引未完全满足查询条件. 也就是查询条件部分使用了索引, 但仍需在扫描结果中进一步过滤 

  ```mysql
  SELECT * FROM users WHERE email = 'test@example.com' AND age > 30;
  # 如果 email 有索引, 但 age 没有索引, 则会显示 Using where
  ```

  - Using index: 查询的数据完全在索引里, 不需要进行回表查询 (也就是"覆盖索引")
  - Using filesort: MySQL发现结果**需要排序, 但无法用索引排序**, 数据较小时从内存排序, 否则需要在磁盘完成排序. **<font style="color:blue">通常是因为 ORDER BY 语句中的列没有使用索引</font>**
  - Using temporary: **<font style="color:blue">查询中用了临时表来保存中间结果</font>**, 说明 MySQL 会在内存或磁盘中创建临时表来执行操作, 这是额外的资源开销
    - 通常出现在复杂的 GROUP BY 或 ORDER BY 查询中, 尤其是这些字段未被索引覆盖时
  - Using index condition:  MySQL 5.6+ 版本引入的, **表示先按条件过滤索引, 过滤完索引后找到所有符合索引条件的数据行, 随后用 WHERE 子句中的其他条件去过滤这些数据行**
    - 这个机制仅在**查询字段不被索引完全覆盖**时才会出现
  - Distinct: 优化器检测到查询中的 DISTINCT 语义, 仅读取符合条件的第一条记录



除此之外还有一些其它不太需要关注的字段比如

- id: select 查询的序列号. **id 相同执行顺序从上到下(<font style="color:blue">大多数情况下 id 相同的几行就是在执行多表查询操作</font>)**; i**d不同值越大越先执行(即先执行子查询语句)**



- select_type: **表示查询的类型**, 下面列出一些常见的取值 (并非所有)
  - SIMPLE -- 简单表, 即不使用表连接或者子查询
  - PRIMARY -- 主查询. 也就是最外层的查询; SUBQUERY -- 子查询中的第一个 select 语句
  - UNION -- **UNION 中的第二个或者后面的查询语句**
  - DERIVED -- from 子句中出现的子查询, 也叫做派生表



- **possible_keys: <font style="color:blue">MySQL 认为可能能用的索引</font>**, 但最终用不用还是看 key 字段



- ref: **表明索引是怎么被使用的, 比如对比的值是 常量(const) 还是 字段(field) 或者 是和函数结果比较(func)**. 没有使用索引或无法判断就是 NULL



- filtered: 表示返回结果的行数占需读取行数的百分比, 所以 filtered 的值越大越好

# MySQL事务的四个特性

MySQL (以及所有关系型数据库) 中事务的四大特性被称为ACID:

> Atomicity; Consistency; Isolation; Durability

- **原子性: 一个事务必须被视为一个不可分割的工作单元**, 事务中的操作要么全部成功提交, 要么全部失败回滚
- 一致性: 事务执行前后, 数据库都处于**一致状态**, 不会破坏数据的完整性约束. 即**<font style="color:blue">数据库总是从一个一致性状态转换到下一个一致性状态</font>**
- 隔离性: **通常来说, 一个事务所做的修改在最终提交以前, 对其它事务是不可见的**. 所以多个事务可以交叉执行, 数据互相隔离
- **持久性: <font style="color:blue">事务一旦提交, 所做的修改就会被永久保存到数据库中</font>**. 即使系统崩溃数据也不会丢失

# 不常考问题

## blob和text的区别

**BLOB 用于存储二进制数据 (如图片 / 音频 / 视频 / 压缩文件等)**, 这种类型不进行字符集转换或排序, 因此也不受字符编码的限制

**TEXT 用于存储大量文本数据 (例如文章内容 / 长文本等)**, 数据在存储时会根据定义的字符集进行编码, 根据指定的字符集进行存储和排序, 并且在排序或比较时也会使用该字符集

> BLOB 类型不支持字符集相关的排序(collation 排序), 但可以按照字节值进行原始排序. 即**<font style="color:blue">排序时是按每个字节的二进制值从左到右比较</font>**
>
> 一旦某个字节能分出大小, 就立即返回排序结果; 若前缀一致, 则较短的字节数组排在前



另外由于BLOB存储的是二进制数据, 所以**<font style="color:blue">它不支持常规的字符串操作函数</font>**. **对于BLOB数据的操作通常限于插入 & 更新 & 删除及等值匹配等二进制级别的检索**

**<font style="color:blue">TEXT 支持大多数字符串操作函数</font>**, 例如 `CONCAT` / `SUBSTRING` / `LENGTH` 等. **此外 TEXT 数据可以使用 LIKE 操作符进行模糊匹配查询, 并支持全文搜索**



在进行数据库备份时, BLOB 数据可能会占用大量的存储空间, 特别是当存储大量的图像或视频文件时. 因此对于数据库管理而言, 可能需要采取特殊的策略来处理这些大文件

- 比如 **BLOB 存储外部化: 也就是将图片 / 视频等数据存储在对象存储服务中(如阿里云OSS / 亚马逊S3), 数据库中只存路径或URL** -- 减少数据库体积、加快备份和恢复

虽然 TEXT 数据同样可以非常大, 但由于其是字符串格式, 因此在备份和恢复时更为直接



> BLOB 和 TEXT 都有四种子类型, 分别用于存储不同大小的数据. 它们分别是:
>
> TINYBLOB / TINYTEXT: 最大存储容量为 255 字节
> BLOB / TEXT: 最大存储容量为 65,535 字节 (即 64 KB)
> MEDIUMBLOB / MEDIUMTEXT: 最大存储容量为 16,777,215 字节 (即 16 MB)
> LONGBLOB / LONGTEXT: 最大存储容量为 4,294,967,295 字节 (即 4 GB)































