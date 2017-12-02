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
	3. 配置Http请求中的 服务器名称或IP,为并发测试的地址 www.whoiszxl.com
	4. 在线程组右键 -> 添加 -> 监听器 -> 查看结果树
	5. 在线程组右键继续添加 -> 配置元件 -> 用户定义的变量 添加 isrun : 1
	6. 在 如果（if）控制器 的配置项中的条件加上判断 ${isrun} == 1
	7. 点击绿色三角箭头run，在结果树中便能出现取样结果，请求头，请求数据等结果

**提示：如果请求的response data乱码，需要打开bin目录下的jmeter.properties，设置sampleresult.default.encoding=UTF-8就稳了**

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

###5.实现请求预处理
> 使用前置处理器实现：类似于Java的AOP，在before after之前之后的一种处理。

	1.使用测试计划新增一个线程组，右键 -> 添加 -> Threads(Users) -> 线程组
	2.线程组右键->前置处理器->用户参数，设置用户参数，随便设置一个，例如 name : zxl
	3.在 线程组右键 -> 添加 -> sampler -> BeanShell Sampler,通过System.out.println("${name}")便可以打印出刚才设置的name

###6.设置集合点，设置并发
> 定时器：设置操作和操作之间的等待时间，就是用户点击操作之间的时间间隔。
	
	1.BeanShell Sampler右键添加->定时器->固定定时器，设置延时时间1000
	2.然后点击run之后，之前的请求会延迟一秒来请求

###7.设置各种请求的发送
> Sapmler:取样器，性能测试中向服务器发送请求，记录响应信息，记录响应时间的最小单元。
	
	1.（Debug用法）在 线程组右键 -> 添加 -> sampler -> Debug Sampler,再次执行run,会多出一个debug结果
	
	sJMeterVariables:
	JMeterThread.last_sample_ok=true
	JMeterThread.pack=org.apache.jmeter.threads.SamplePackage@17c3a29
	START.HMS=105819
	START.MS=1512183499283
	START.YMD=20171202
	TESTSTART.MS=1512185363190

###8.实现Sampler关联
> 后置处理器：对Sampler发出请求后得到的服务器响应进行处理，也差不多是AOP了

	1.添加一个线程组，再添加一个Http请求，设置访问域名 www.whoiszxl.com
	2.添加一个后置处理器->正则表达式提取器，再添加一个Beanshell Sampler，还有一个查看结果树
	3.然后可以设置需要正则过滤的文本，打开正则表达式提取器
		>设置引用名称：title
		>正则表达式设置：<title>(.+?)</title> 匹配title标签中的内容
		>模板：$1$
	4.然后在BeanShell Sampler,通过System.out.println("${title}")，就能直接打印出正则匹配出来的title值

###9.断言
> 断言，就是检查测试中得到的数据是否符合预期，Java Junit的Assert这种

	1.新建一个线程组，新建一个Http请求，设置响应域名 www.whoiszxl.com，添加查看结果树
	2.Http请求右键->断言->响应断言，添加规则，随便输入字符，匹配响应结果，如果输入的字符属于响应结果，便请求成功，反之失败。
**注：断言在实际运用中使用不多**

###10.监控数据可视化
> 监听器：用来对测试数据结果进行处理和可视化展示的组件，就像之前的查看结果树。
	
	还有一些聚合报告，图形结果等等之类的。

###11.函数助手
	a.随机数(__Random) b.参数化助手(__CSVRead) c.计数器（__counter） d.唯一数(__UID_) 
	
	1.新建一个线程组，在线程组下新建一个Beanshell Sampler
	2.使用Ctrl+Shift+1，打开函数助手对话框，选择__counter功能,设置参数a:FALSE , b:counter