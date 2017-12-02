# Linux性能监控测试

标签： Linux,服务器,CentOS,性能监控

----

**Contents**

- [Linux性能监控测试](#Linux性能监控测试)
    - [Linux性能测试初步认识](#Linux性能测试初步认识)
    - [Linux监控命令学习](#Linux监控命令学习)

----


下载链接

>* [nmon_analyser](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power+Systems/page/nmon_analyser)



## Linux性能测试初步认识

###1.测试范围
	- CPU
	- 内存
	- 网络
	- 版本
	- 磁盘
	
###2.进程与线程
	
	进程：系统进行资源分配和调度的一个独立单位，比如说QQ,Chrome等运行的程序。
	线程：线程是进程的一个实体，基本不拥有系统资源，类似于QQ打开的一个对话框。
	
	*  一个线程只能属于一个进程，一个进程可以有多个线程
	*  一个进程会分配一个地址空间，进程与进程不共享内存
	*  线程共享父进程的地址空间




## Linux监控命令学习

###1.一些命令
	
	1.man 函数手册命令，可以查看所有命令的使用方法
	
	2.top 实时监控系统运行状态
		> -h：帮助
		> -p：监控制定进程
		> 进入top界面后：M：按内存使用率排序，P：按CPU使用率排序 z：彩色/黑白显示
		> top中的load avarage：系统运行队列的平均利用率，三个值分别为最后1，5，15分钟的平均负载值。
		> load avarage：值为1为满负荷状态，多核服务器为1*cpu数量
	
	3.vmstat 可以监控操作系统的进程状态，内存，虚拟内存，磁盘IO,CPU等信息
		> 常用： vmstat 1 10    1:表示时间间隔（秒） 2.表示执行多少次
		> vmstat -S (k K m M) 表示查看的风格 小写为1000进制，大写为1024进制
	
	4.free 监控内存使用情况
		>total 总计物理内存，Used：已使用 Free:还剩多少 shared：多少个进程共享的内存总额  buffers/cached:磁盘缓存大小
		>常用 free -h
	
	5.mpstat 查看多核心CPU中每个计算核心的统计数据
	
	6.netstat 网络监控，显示本机网络连接，运行端口，，路由表等信息
		> netstat -l 列出只在Listen监听的服务
		> netstat -ntlp 常用组合

	7.iostat 磁盘监控，显示磁盘读写操作的统计信息，给出CPU使用情况。
	
	8.sar (System Activity Reporter系统活动情况报告)，监控文件读写，系统调用，磁盘IO,CPU效率，内存状况，进程等

	9.strace 跟踪进程启动
		> strace -ff -F -t -o zk.log ./zkServer.sh start 跟踪zookeeper启动，里面有-1的则是报错的信息
	
###2.一些工具
	1.nmon 监控信息比较全面，实时捕捉监控，可输出到文件，通过nmon_analyzer图形化结果。
		a. wget http://sourceforge.net/projects/nmon/files/nmon_linux_14i.tar.gz 下载
		b. 创建nmon目录，并解压 tar -zxvf nmon_linux_14i.tar.gz
		c. 安装 mv nmon_x86_64_centos6 nmon && cp nmon /usr/bin/

		使用：nmon -f 启动  （-f必带，输出为文件）
			1.-F 同 -f,但是可以设置文件名
			2.-s 保存数据的频率
			3.-c 采集数据次数
			4.-t 输出最消耗资源的进程数据

		常用：
			nmon -f -F server.nmon -s 1 -c 10 -t

		分析：
			使用nmon_analyser,通过excel统计出表格数据。
			重点：
				1.SYS_SUMM 系统汇总页，cpu占有率变化，磁盘IO变化等
				2.AAA 操作系统和nmon本身的一些信息
				3.CPUnn CPU占用情况
				4.CPU_ALL 所有CPU占用情况

		优点：集成了之前很多的命令，比较方便

	2.crontab 定时任务 Linux系统是由cron这个系统来控制的
		* /sbin/service crond status 查看是否启动 start/stop/restart/reload
		
		服务权限：
			* crontab权限管理存储在cron.allow和cron.deny中，没有则创建到/etc/下
			* cron.allow 允许哪些用户使用  cron.deny则拒绝哪些用户使用
			* cron.allow 优先级最高，两个都配置了取allow

		使用：
			命令：crontab -e
			minute   hour   day   month   week   command
			minute： 表示分钟，可以是从0到59之间的任何整数。
			hour：表示小时，可以是从0到23之间的任何整数。
			day：表示日期，可以是从1到31之间的任何整数。
			month：表示月份，可以是从1到12之间的任何整数。
			week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
			command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。
			
			星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
			逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
			中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
			正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。
				