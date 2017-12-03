# MySQL数据库性能测试

标签： 数据库性能测试，MySQL

----

**Contents**

- [MySQL数据库性能测试](#MySQL数据库性能测试)
    - [MySQL数据库性能测试初步认识](#MySQL数据库性能测试初步认识)
    - [MySQL慢查询工作原理及操作](#MySQL慢查询工作原理及操作)
    - [SQL的分析于调优方法](#SQL的分析于调优方法)
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
		

##MySQL慢查询工作原理及操作
* 慢查询就是查询速度慢的SQL语句，其执行速度超过定义的时间。（不同系统定义不同的慢查询标准）
	- 慢查询开启：编辑/etc/my.cnf,在[mysqld]中添加: slow_query_log = 1
	- 慢查询日志路径：slow_query_log_file = /data/slow.log
	- 慢查询的时长：long_query_time = 1
	- 未使用索引的select也记录到慢查询日志：log_queries_not_using_indexes = 1

###慢查询日志分析
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