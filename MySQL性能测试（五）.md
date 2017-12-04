# MySQL数据库性能测试

标签： 数据库性能测试，MySQL

----

**Contents**

- [MySQL数据库性能测试](#MySQL数据库性能测试)
    - [MySQL数据库性能测试初步认识](#MySQL数据库性能测试初步认识)
    - [MySQL慢查询工作原理及操作](#jump)
    - [SQL的分析与调优方法](#SQL的分析与调优方法)
    - [MySQL的存储引擎](#MySQL的存储引擎)
    - [MySQL的实时监控](#MySQL的实时监控)
    - [MySQL的集群监控方案](#MySQL的集群监控方案)
    - [使用Jmeter开发MySQL性能测试脚本](#使用Jmeter开发MySQL性能测试脚本)

----


下载链接

>* [nmon_analyser](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power+Systems/page/nmon_analyser)



## MySQL数据库性能测试初步认识

	1.MySQL数据库介绍
		* 主流分支:MariaDb,MySQL作者创建，替换MySQL，完全兼容。
	
	2.MySQL数据库监控指标
		> QPS,每秒查询数量（queries per seconds）
			查询方法：show global status like 'Qeustion%';
		
		> TPS,每秒执行的事务数量 (transaction per second)
			计算方法：TPS = (Com_commit + Com_rollback)/seconds
			查询方法：show global status like 'Com_commit';  and   show global status like 'Com_rollback';
		
		>线程连接数
			查询方法：show global status like 'Max_used_connections'; and show global status like 'Threads%'
		
		>Query Cache,缓存select查询结果，同样查询直接返回缓存，适用于大量查询，数据少修改的场景
			开启方法：1.修改my.cnf,将query_cache_size设置为1024的倍数
			2.增加一行：query_cache_type=0/1/2,1缓存所有，2只缓存在select通过SQL_CACHE指定的查询
		
		> 锁定状态
			show global status like '%lock%';
			Table_locks_waited/Table_locks_immediate 值越大代表表锁阻塞越严重
			Innodb_row_lock_waits innodb行锁

		>主从延时
			查询主从延时时间：show slave status;
		

## <span id = "jump">MySQL慢查询工作原理及操作</span>
* 慢查询就是查询速度慢的SQL语句，其执行速度超过定义的时间。（不同系统定义不同的慢查询标准）
	- 慢查询开启：编辑/etc/my.cnf,在[mysqld]中添加: slow_query_log = 1
	- 慢查询日志路径：slow_query_log_file = /data/slow.log
	- 慢查询的时长：long_query_time = 1
	- 未使用索引的select也记录到慢查询日志：log_queries_not_using_indexes = 1

### 慢查询日志分析
* 使用mysqldumpslow
	- -s 表示按照什么方式排序
		- c:访问次数
		- l:锁定时间
		- r:返回记录
		- t:查询时间
		- al:平均锁定时间
		- ar:平均返回记录数
		- at:平均查询时间
	- -t top n 的意思，返回前n条数据
	- -g 后面可以跟正则

举个例子
	
	1.得到返回记录集最多的10个SQL
		mysqldumpslow -s r -t 10 slow.log
	2.得到访问次数最多的10个SQL
		mysqldumpslow -s c -t 10 slow.log
	3.得到按照时间排序的前10条含有左连接的SQL
		mysqldumpslow -s t -t 10 -g "left join" slow.log

## SQL的分析与调优方法

### SQL性能分析
* 用法：select 前加上 explain

参数含义：

1.select_type:

	SIMPLE：表示不需要union操作或者不包含子查询的简单SQL
	PRIMARY：一个需要union操作或包含子查询的SQL

2.table
	
	不涉及表则为空，涉及表则为表名
	<derived N>表示是临时表

3.**type**

	从好到坏：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
	1.system:表中只有一行数据或者空表，只能用于myisam和memory表，innodb通常是all或者index
	2.const:使用了唯一索引或者主键，返回记录一定是1行记录的等值
	3.fulltext:全文索引，全文索引优先级很高，同时又普通索引，优先全文索引
	4.unique_subquery:where查询中的in子查询，子查询返回唯一值
	5.range:索引范围扫描，<,>,null,between,in,like等
	6.index_merge:使用了两个以上索引，取交集或并集，and或or使用了不同的索引。
	7.index：索引全表扫描。
	8.all：全表扫描，最糟糕的SQL。

4.possible_keys
	
	可能使用到的索引全部列出来

5.key
	
	真正使用到的索引，如通过主键id查询则为PRIMARY

6.rows
	
	大概查询了多少行数据，不是精确值。

7.extra
	
	用到了where条件就显示一个 `using where`

### SQL性能调优

#### 索引
- 主键索引
	- 特殊的唯一索引，不允许有空值。一般使用在表的ID列。
- 全文索引
	- 只适用与MySIAM的一个索引类型，只能作用于char，varchar，text。
	- 底层通过match()和against()来实现全文索引查询。
- 唯一索引
	- 索引列的值必须唯一，但可以为空。
- 组合索引
	- 多列索引，使用多列同时创建一个索引，比分别建索引更快。 
- 普通索引
	- 最基本索引，没有条件限制。

#### 索引创建规范
1.劣势

	- 索引可以提高查询效率也会降低插入和更新的速度并占磁盘空间
	- 在插入与更新数据时，要重写索引文件

2.规范

	- 单张表中索引数量尽量不要超过五个
	- 单个索引的字段数尽量不要超过五个
	- 不要使用更新频繁的列作为主键
	- 合理创建组合索引
	- 不要在低基数列建立索引，比如 性别，status等只有0和1的列。
	- 不要在索引列进行数学运算和函数运算，会使索引失效。（sum，count等）
	- like '%xxxx'无法使用索引，前置%无法使用，后置可以
	- not in / not like 的反向查询也无法使用索引
	- 选择越小数据类型越好，如用tinyint表示性别
	- 在经常需要使用order by和group by和distinct列加索引（单独order by无法使用索引，索引考虑加where或limit）
	- 表和表的连接条件加上索引加快速度，on a.id = b.id，则id与id之间可以在索引间对应上
	

## MySQL的存储引擎

### 存储引擎介绍

#### MyISAM
优点：

- 读性能比InnoDB高很多，适用于查询多修改少的业务场景
- 索引与数据分开，使用了压缩，提高内存使用率

缺点：

- 不支持事务
- 写入数据，直接锁表，效率不高

#### InnoDB
优点：

- 支持事务
- 支持外键
- 支持行锁，写入效率高

缺点：

- 不支持全文索引
- 行级锁并不是绝对，当MySQL不确定扫描范围的时候，锁全表
- 索引与数据紧密捆绑，无压缩体积庞大

## MySQL的实时监控
使用orzdba
