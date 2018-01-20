---
title: Tableau | 优化数据提取
date: 2018-01-18 00:35:55
tags: tableau
---
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/figure_captions/waters-3084551_1920.jpg)

本文TableauServer基于win10环境
<!--more-->
## 增加后台进程数
1. 以管理员身份进入TableauServer的bin目录，并停止Server
	
	cd [安装目录]\Tableau\Tableau Server\10.3\bin
	tabadmin stop

2. 打开**Configure Tableau Server(Tableau Server配置使用工具)**
![||300*300](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-优化数据提取/tableau优化数据提取_1.png)


3. 点击**Servers(服务器)**选项卡
![||600*600](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-优化数据提取/tableau优化数据提取_2.png)

4. 在Number of Processes per Server栏中选中服务器，然后点击**Edit(编辑)**，设置Backgrouder(后台进程)为2，之前为1。
![||600*600](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-优化数据提取/tableau优化数据提取_3.png)

5. 点击**常规(General)**，并输入服务器的管理员账号信息，保存设置。
![||600*600](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-优化数据提取/tableau优化数据提取_4.png)

6. 启动Tableau Server
	
	tabadmin start

使用tableau的管理员浏览器访问tableau server，点击状态可查看到后台程序为2，说明修改成功。
![||600*450](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-优化数据提取/tableau优化数据提取_5.png)


## 获取Tableau Server 存储的访问权限
以管理员身份进入Tableau server的bin目录，并停止Server
	
	cd [安装目录]\Tableau\Tableau Server\10.3\bin
	tabadmin stop

开启readonly 用户，并设置密码
	
	tabadmin dbpass --username readonly 12345678

重启TableauServer
	
	tabadmin start

## 设置数据提取超时时间
数据提取默认超时时间为7200s，超时会导致数据提取失败。但可通过以下方式，重新设置超时时间。
首先以管理员身份进入Tableau server的bin目录
	
	cd [安装目录]\Tableau\Tableau Server\10.3\bin
	
通过修改querylimit参数调整超时时间，并将重新加载配置，启动Server。这里设置querylimit=18000s(5h)

	tabadmin set backgrounder.querylimit 9000
	tabadmin configure 
    tabadmin start

## 参考
1. [TableauServer存储库workgroup database](https://onlinehelp.tableau.com/current/server/en-us/data_dictionary.html#_background_tasks_anchor)
2. [错误“7200 秒内未能完成数据提取”](http://kb.tableau.com/articles/issue/error-extract-timeout-7200-seconds?lang=zh-cn)
3. [针对数据提取进行优化](https://onlinehelp.tableau.com/v10.3/server/zh-cn/perf_optimize_extracts.htm)
4. 《TableauServer 管理指南》