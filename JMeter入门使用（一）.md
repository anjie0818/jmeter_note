# Jmeter性能测试

标签： Jmeter

----

**Contents**

- [Jmeter性能测试](#%E8%BD%AF%E4%BB%B6%E6%B5%8B%E8%AF%95%E5%88%9D%E6%AD%A5%E8%AE%A4%E8%AF%86)
    - [性能测试的工作流程](#%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)
    - [Jemeter简单使用](#Jemeter简单使用)

----


下载链接

>* [Apache Jmeter Project](http://jmeter.apache.org/download_jmeter.cgi)


# Jmeter性能测试

## 性能测试的工作流程

步骤
- 需求分析
- 性能指标制定
- 脚本开发
- 场景设置
- 监控部署
- 测试执行
- 性能分析
- 性能调优
- 测试报告

## Jemeter简单使用
###1.打开apache-jmeter-3.3\bin目录下的ApacheJMeter.jar

###2.使用测试计划，右键 -> 添加 -> Threads(Users) -> 线程组
	设置线程组的参数：

	- 线程数 （设置同时访问的模拟用户数量,并发数量）
	- Ramp-Up Period(in seconds) （多少秒钟之内跑完上面的线程数）
	- 循环次数 （跑多少轮上述的配置）

###3.实现测试的IF逻辑控制（可以通过模拟访问的userid，goodsid等进行不一样的策略）
	
	1. 步骤：在线程组右键 -> 添加 -> 逻辑控制器 -> 如果（if）控制器
	2. 在 如果（if）控制器右键 -> 添加 -> sampler -> Http请求
	3. 配置Http请求中的 服务器名称或IP,为并发测试的地址
	4. 在线程组右键 -> 添加 -> 监听器 -> 查看结果树
	5. 在线程组右键继续添加 -> 配置元件 -> 用户定义的变量 添加 isrun : 1
	6. 在 如果（if）控制器 的配置项中的条件加上判断 ${isrun} == 1
	7. 点击绿色三角箭头run，在结果树中便能出现取样结果，请求头，请求数据等结果

###4.实现配置管理
> 配置元件：提供配置相关信息，如cookie，http头，亦可自行定义变量常量。
	
	1.在线程组右键继续添加->配置元件->JDBC Connection Configuration
	2.配置Variable Name: 随便取一个名
		> 配置Database URL:jdbc:mysql://app.chenyuspace.com/cdbook?serverTimezone=UTC
		> 配置JDBC Driver Class:com.mysql.jdbc.Driver
		> username password 填写相对应的账号密码
	3.在线程组右键继续添加->Sampler->JDBC Request，Variable Name填入刚才随便填的名
	4.再添加一个查看结果树，就可以run查看结果了
		>如果出现了 Cannot load JDBC driver class 'com.mysql.jdbc.Driver'，
		>需要下载一个javamysql驱动包（mysql-connector-java-5.1.28.jar），复制到Jmeter目录下的lib/ext
	
