---
layout: post
category : 数据库
tagline:
tags : [mysql, sql, performance]
excerpt :
title_cn: MySQL慢查询分析
description: 最近开发微信红包活动，并发量大概在100左右，数据库存储数据几十万条。活动上线后，遇到一个问题，服务启动起来后，在几分钟之内，服务变得很慢，通过分析tcp状态（ss -s）发现，处于timewait状态的TCP达到一千多个，然后从应用前台、后台到数据库逐步分析，最后发现，由于某一张表的数据量达到几十万，而某一个逻辑需要连接该表进行数据查询，导致SQL执行非常缓慢。
---
{% include JB/setup %}

最近开发微信红包活动，并发量大概在100左右，数据库存储数据几十万条。活动上线后，遇到一个问题，服务启动起来后，在几分钟之内，服务变得很慢，通过分析tcp状态（ss -s）发现，处于timewait状态的TCP达到一千多个，然后从应用前台、后台到数据库逐步分析，最后发现，由于某一张表的数据量达到几十万，而某一个逻辑需要连接该表进行数据查询，导致SQL执行非常缓慢。于是整理下SQL慢查询分析的相关方法，以便查阅。

## 一、<code>explain</code>语句

<code>explain</code>命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，<code>explain</code>可以用来查看 SQL 语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。

Explain语法：
<pre>explain select … from … [where ...]</pre>

例如：explain select * from news;
输出：
<pre>
---- ------------- ------- ------- ------------------- --------- --------- ------- ------ -------
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
---- ------------- ------- ------- ------------------- --------- --------- ------- ------ -------
</pre>

下面对各个属性进行了解：

1、<strong>id</strong>：这是SELECT的查询序列号

2、<strong>select_type</strong>：select_type就是select的类型，可以有以下几种：
<pre>
SIMPLE：简单SELECT(不使用UNION或子查询等)
PRIMARY：最外面的SELECT
UNION：UNION中的第二个或后面的SELECT语句
DEPENDENT UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询
UNION RESULT：UNION的结果。
SUBQUERY：子查询中的第一个SELECT
DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询
DERIVED：导出表的SELECT(FROM子句的子查询)
</pre>

3、<strong>table</strong>：显示这一行的数据是关于哪张表的

4、<strong>type</strong>：这列最重要，显示了连接使用了哪种类别,有无使用索引，是使用Explain命令分析性能瓶颈的关键项之一。

<pre>
结果值从好到坏依次是：
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL<
一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。

const
表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待

eq_ref
在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用

ref
这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好

range
这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况

index
这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）

ALL
这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免
</pre>

5、<strong>possible_keys</strong>：列指出MySQL能使用哪个索引在该表中找到行

6、<strong>key</strong>：显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL

7、<strong>key_len</strong>：显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好

8、<strong>ref</strong>：显示使用哪个列或常数与key一起从表中选择行。

9、<strong>rows</strong>：显示MySQL认为它执行查询时必须检查的行数。

10、<strong>Extra</strong>：包含MySQL解决查询的详细信息，也是关键参考项之一。

<pre>
Distinct
一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

Not exists
MYSQL 优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了
Range checked for each

Record（index map:#）
没有找到理想的索引，因此对于从前面表中来的每一 个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

Using filesort
看 到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来 排序全部行

Using index
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表 的全部的请求列都是同一个索引的部分的时候

Using temporary
看到这个的时候，查询需要优化了。这 里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

Using where
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index， 这就会发生，或者是查询有问题
</pre>

其他一些Tip：

当type 显示为 “index” 时，并且Extra显示为“Using Index”， 表明使用了覆盖索引。

## 二、查询低效率sql语句

###（一）、捕获低效率SQL

MySQL数据库有几个配置选项可以帮助我们及时捕获低效SQL语句

1，slow_query_log

这个参数设置为ON，可以捕获执行时间超过一定数值的SQL语句。

2，long_query_time

当SQL语句执行时间超过此数值时，就会被记录到日志中，建议设置为1或者更短。

3，slow_query_log_file

记录日志的文件名。

4，log_queries_not_using_indexes

这个参数设置为ON，可以捕获到所有未使用索引的SQL语句，尽管这个SQL语句有可能执行得挺快。

###（二）、检测mysql中sql语句的效率的方法

####1、通过查询日志

#####（1）、Windows下开启MySQL慢查询

MySQL在Windows系统中的配置文件一般是是my.ini找到[mysqld]下面加上

<pre>
log-slow-queries = F:/MySQL/log/mysqlslowquery。log
long_query_time = 2
</pre>

#####（2）、Linux下启用MySQL慢查询

MySQL在Windows系统中的配置文件一般是是my.cnf找到[mysqld]下面加上
<pre>
log-slow-queries=/data/mysqldata/slowquery。log
long_query_time=2
</pre>

说明
<pre>
log-slow-queries = F:/MySQL/log/mysqlslowquery。
</pre>
为慢查询日志存放的位置，一般这个目录要有MySQL的运行帐号的可写权限，一般都将这个目录设置为MySQL的数据存放目录；

<pre>
long_query_time=2中的2表示查询超过两秒才记录；
</pre>

####2.show processlist 命令

<code>SHOW PROCESSLIST</code>显示哪些线程正在运行。您也可以使用<code>mysqladmin processlist</code>语句得到此信息。

各列的含义和用途：
<pre>
ID列
一个标识，你要kill一个语句的时候很有用，用命令杀掉此查询 /*/mysqladmin kill 进程号。
user列
显示单前用户，如果不是root，这个命令就只显示你权限范围内的sql语句。
host列
显示这个语句是从哪个ip的哪个端口上发出的。用于追踪出问题语句的用户。
db列
显示这个进程目前连接的是哪个数据库。
command列
显示当前连接的执行的命令，一般就是休眠（sleep），查询（query），连接（connect）。
time列
此这个状态持续的时间，单位是秒。
state列
显示使用当前连接的sql语句的状态，很重要的列，后续会有所有的状态的描述，请注意，state只是语句执行中的某一个状态，一个 sql语句，以查询为例，可能需要经过copying to tmp table，Sorting result，Sending data等状态才可以完成
info列
显示这个sql语句，因为长度有限，所以长的sql语句就显示不全，但是一个判断问题语句的重要依据。
</pre>

这个命令中最关键的就是state列，mysql列出的状态主要有以下几种：

<pre>
Checking table
正在检查数据表（这是自动的）。
Closing tables
正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
Connect Out
复制从服务器正在连接主服务器。

Copying to tmp table on disk
由于临时结果集大于tmp_table_size，正在将临时表从内存存储转为磁盘存储以此节省内存。
Creating tmp table
正在创建临时表以存放部分查询结果。
deleting from main table
服务器正在执行多表删除中的第一部分，刚删除第一个表。
deleting from reference tables
服务器正在执行多表删除中的第二部分，正在删除其他表的记录。

Flushing tables
正在执行FLUSH TABLES，等待其他线程关闭数据表。
Killed
发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效。
Locked
被其他查询锁住了。
Sending data
正在处理SELECT查询的记录，同时正在把结果发送给客户端。
Sorting for group
正在为GROUP BY做排序。
Sorting for order
正在为ORDER BY做排序。
Opening tables
这个过程应该会很快，除非受到其他因素的干扰。例如，在执ALTER TABLE或LOCK TABLE语句行完以前，数据表无法被其他线程打开。正尝试打开一个表。
Removing duplicates
正在执行一个SELECT DISTINCT方式的查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。
Reopen table
获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。已经释放锁，关闭数据表，正尝试重新打开数据表。
Repair by sorting
修复指令正在排序以创建索引。
Repair with keycache
修复指令正在利用索引缓存一个一个地创建新索引。它会比Repair by sorting慢些。
Searching rows for update
正在讲符合条件的记录找出来以备更新。它必须在UPDATE要修改相关的记录之前就完成了。
Sleeping
正在等待客户端发送新请求.
System lock
正在等待取得一个外部的系统锁。如果当前没有运行多个mysqld服务器同时请求同一个表，那么可以通过增加--skip-external-locking参数来禁止外部系统锁。
Upgrading lock
INSERT DELAYED正在尝试取得一个锁表以插入新记录。
Updating
正在搜索匹配的记录，并且修改它们。
User Lock
 正在等待GET_LOCK()。
Waiting for tables
 该线程得到通知，数据表结构已经被修改了，需要重新打开数据表以取得新的结构。然后，为了能的重新打开数据表，必须等到所有其他线程关闭这个表。以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE,或OPTIMIZE TABLE。
waiting for handler insert
 INSERT DELAYED已经处理完了所有待处理的插入操作，正在等待新的请求。
 大部分状态对应很快的操作，只要有一个线程保持同一个状态好几秒钟，那么可能是有问题发生了，需要检查一下。
 还有其他的状态没在上面中列出来，不过它们大部分只是在查看服务器是否有存在错误是才用得着。
</pre>

#### 3、开启profiling功能

MySQL 的 SQL 語法調整主要都是使用 EXPLAIN , 但是這個並沒辦法知道詳細的 Ram(Memory)/CPU 等使用量.於 MySQL 5.0.37 以上開始支援 MySQL Query Profiler, 可以查詢到此 SQL 會執行多少時間, 並看出 CPU/Memory 使用量, 執行過程中 System lock, Table lock 花多少時間等等.
啟動
<pre>
mysql> set profiling=1; # 此命令於 MySQL 會於 information_schema 的 database 建立一個 PROFILING 的 table 來紀錄.
SQL profiles show
mysql> show profiles; # 從啟動之後所有語法及使用時間, 含錯誤語法都會紀錄.
ex: (root@localhost) [test]> show profiles; # 注意 Query_ID, 下面執行時間統計等, 都是依 Query_ID 在紀錄
 +----------+------------+---------------------------+
 | Query_ID | Duration   | Query                     |
 +----------+------------+---------------------------+
 |        1 | 0.00090400 | show profile for query 1  |
 |        2 | 0.00008700 | select * from users       |
 |        3 | 0.00183800 | show tables               |
 |        4 | 0.00027600 | mysql> show profiles      |
 +----------+------------+---------------------------+
 </pre>

 查詢所有花費時間加總
 <pre>
mysql> select sum(duration) from information_schema.profiling where query_id=1; # Query ID = 1
 +---------------+
 | sum(duration) |
 +---------------+
 |      0.000447 |
 +---------------+

 </pre>

 查詢各執行階段花費多少時間
 <pre>
mysql> show profile for query 1; # Query ID = 1
 +--------------------+------------+
 | Status             | Duration   |
 +--------------------+------------+
 | (initialization)   | 0.00006300 |
 | Opening tables     | 0.00001400 |
 | System lock        | 0.00000600 |
 | Table lock         | 0.00001000 |
 | init               | 0.00002200 |
 | optimizing         | 0.00001100 |
 | statistics         | 0.00009300 |
 | preparing          | 0.00001700 |
 | executing          | 0.00000700 |
 | Sending data       | 0.00016800 |
 | end                | 0.00000700 |
 | query end          | 0.00000500 |
 | freeing items      | 0.00001200 |
 | closing tables     | 0.00000800 |
 | logging slow query | 0.00000400 |
 +--------------------+------------+
 </pre>

 查詢各執行階段花費的各種資源列表
 <pre>
mysql> show profile cpu for query 1; # Query ID = 1
 +--------------------------------+----------+----------+------------+
 | Status                         | Duration | CPU_user | CPU_system |
 +--------------------------------+----------+----------+------------+
 | (initialization)               | 0.000007 | 0        | 0          |
 | checking query cache for query | 0.000071 | 0        | 0          |
 | Opening tables                 | 0.000024 | 0        | 0          |
 | System lock                    | 0.000014 | 0        | 0          |
 | Table lock                     | 0.000055 | 0.001    | 0          |
 | init                           | 0.000036 | 0        | 0          |
 | optimizing                     | 0.000013 | 0        | 0          |
 | statistics                     | 0.000021 | 0        | 0          |
 | preparing                      | 0.00002  | 0        | 0          |
 | executing                      | 0.00001  | 0        | 0          |
 | Sending data                   | 0.015072 | 0.011998 | 0          |
 | end                            | 0.000021 | 0        | 0          |
 | query end                      | 0.000011 | 0        | 0          |
 | storing result in query cache  | 0.00001  | 0        | 0          |
 | freeing items                  | 0.000018 | 0        | 0          |
 | closing tables                 | 0.000019 | 0        | 0          |
 | logging slow query             | 0.000009 | 0        | 0          |
 +--------------------------------+----------+----------+------------+
 mysql> show profile IPC for query 1;
 +--------------------------------+----------+---------------+-------------------+
 | Status                         | Duration | Messages_sent | Messages_received |
 +--------------------------------+----------+---------------+-------------------+
 | (initialization)               | 0.000007 |             0 |                 0 |
 | checking query cache for query | 0.000071 |             0 |                 0 |
 | Opening tables                 | 0.000024 |             0 |                 0 |
 | System lock                    | 0.000014 |             0 |                 0 |
 | Table lock                     | 0.000055 |             0 |                 0 |
 | init                           | 0.000036 |             0 |                 0 |
 | optimizing                     | 0.000013 |             0 |                 0 |
 | statistics                     | 0.000021 |             0 |                 0 |
 | preparing                      | 0.00002  |             0 |                 0 |
 | executing                      | 0.00001  |             0 |                 0 |
 | Sending data                   | 0.015072 |             0 |                 0 |
 | end                            | 0.000021 |             0 |                 0 |
 | query end                      | 0.000011 |             0 |                 0 |
 | storing result in query cache  | 0.00001  |             0 |                 0 |
 | freeing items                  | 0.000018 |             0 |                 0 |
 | closing tables                 | 0.000019 |             0 |                 0 |
 | logging slow query             | 0.000009 |             0 |                 0 |
 +--------------------------------+----------+---------------+-------------------+
 </pre>

 其它屬性列表

 <pre>
ALL - displays all information
BLOCK IO - displays counts for block input and output operations
CONTEXT SWITCHES - displays counts for voluntary and involuntary context switches
IPC - displays counts for messages sent and received
MEMORY - is not currently implemented
PAGE FAULTS - displays counts for major and minor page faults
SOURCE - displays the names of functions from the source code, together with the name and line number of the file in which the function occurs
SWAPS - displays swap counts
</pre>

設定 Profiling 存的 Size
<pre>
mysql> show variables where variable_name='profiling_history_size'; # 預設是 15筆
</pre>

關閉
<pre>
mysql> set profiling=0;
</pre>