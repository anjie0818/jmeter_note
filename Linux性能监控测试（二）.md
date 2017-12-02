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
