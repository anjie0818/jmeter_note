# Linux性能监控测试

标签： Linux,服务器,CentOS,性能监控

----

**Contents**

- [Linux性能监控测试](#Linux性能监控测试)
    - [Linux性能测试初步认识](#Linux性能测试初步认识)
    - [Linux监控命令学习](#Linux监控命令学习)

----


下载链接

>* [Apache Jmeter Project](http://jmeter.apache.org/download_jmeter.cgi)



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
	
	6.netstat 网络监控，显示本机网络连接，运行端口，路由表等信息
		> netstat -l 列出只在Listen监听的服务
		> netstat -ntlp 常用组合

	7.iostat 磁盘监控，显示磁盘读写操作的统计信息，给出CPU使用情况。