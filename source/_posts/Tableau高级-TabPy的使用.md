---
title: Tableau 高级 | TabPy的使用
date: 2018-05-16 23:35:49
tags: hexo
---
<!-- more -->

Tableau桌面版10.1以上的版本支持使用TabPy。

<!-- MarkdownTOC -->

- TabPy简介
- TabPy安装与配置
    - 安装TabPy
    - 启动TabPy服务
- TabPy的使用
    - TableauDesktop
    - TableauServer
    - TabPy的使用
    - 调试与打印日志
- 参考

<!-- /MarkdownTOC -->

## TabPy简介
TabPy实现了tableau的计算字段里嵌入python或R代码(可加入一些机器学习或数据处理的库)。它是一个基于Tornado和其他Python库的Python进程，将计算字段嵌入的代码传输到后台(web后台基于Tornado)，由后台计算完后(可调用第三方库如机器学习相关的库)将结果再放回到前端展示。所以TabPy由两个主要组件构成：

1. 基于Tornado的web服务进程(**Server**)，运行REST API传回的python代码，执行完毕后，再传回前端。
2. **client library**.

## TabPy安装与配置

### 安装TabPy

``` sh
$ cd /opt  # 一般自定义安装的软件安装包都放到这儿
$ sudo wget https://github.com/tableau/TabPy/archive/master.zip  # 下载TabPy
$ sudo unzip master.zip  # 解压到文件夹TabPy-master
$ sudo chown -R devops:devops TabPy-master  # 修改该目录权限为当前用户devops
$ cd TabPy-master
$ chmod u+x setup.sh
$ ./setup.sh  # 安装相应的第三方库，然后启动TabPy服务
```

### 启动TabPy服务
首次安装成功后，会启动TabPy服务，终端会打印出以后启动脚本的路径:

``` sh
$ ./setup.sh
...
Successfully installed Tornado-JSON-1.3.2 future-0.16.0 futures-3.2.0 simplejson-3.15.0 tabpy-server-0.2
~~~~~~~~~~~~~~~  Installation completed  ~~~~~~~~~~~~~~~

From now on, you can start the server by running /home/devops/anaconda/envs/Tableau-Python-Server/lib/python2.7/site-packages/tabpy_server/startup.sh
...
```
如上启动脚本的路径为 `/home/devops/anaconda/envs/Tableau-Python-Server/lib/python2.7/site-packages/tabpy_server/startup.sh`

当前生产环境的TabPy部署在数仓服务器上(10.30.49.80)

## TabPy的使用
### TableauDesktop
在TableauDesktop使用TabPy需进行以下配置。

1. Tableau工具栏中依次点击Help->Settings and Performance->Manage External Service Connection。 
2. 键入服务器地址(如果TabPy安装在本地，服务器地址为localhost)和端口号(默认为9004)

### TableauServer
发布到TableauServer，需要在服务器端配置连接Tabpy.

```
$ tabadmin stop
$ tabadmin set vizqlserver.allow_insecure_scripts true
$ tabadmin set vizqlserver.extsvc.host <ip address or host name of the machine hosting TabPy>
$ tabadmin set vizqlserver.extsvc.port <port for TabPy>
$ tabadmin configure
$ tabadmin start
```

### TabPy的使用
当前Tableau主要通过4个不同返回类型的函数来调用TabPy来满足使用场景: `SCRIPT_INT`, `SCRIPT_REAL`, `SCRIPT_STR`, `SCRIPT_BOOL`。这四个函数内容大体都为：Python(或R)脚本字符串 + 参数1[参数2，参数n] 且都需要return。

```
# 计算字段1
SCRIPT_STR("
import requests
return [requests.get('http://boss.huizhaofang.com/api/if/calc_breach_refund/?contract_no={}'.format(item)).text for item in _arg1]", ATTR([contract_no]))

# 计算字段2
SCRIPT_REAL("
import json
return [json.loads(item)['result']['refund_price']['breach_price']/100.00 for item in _arg1]", [response])
```

### 调试与打印日志

- 在嵌入python代码里使用print函数。相关内容会打印到TabPy服务端的日志里，如：

```
SCRIPT_REAL("
print(_arg1) 
return sum(_arg1)", SUM([Sales]))
```
- 将中间过程的数据打印到csv里

## 参考
1. [Tableau and Python Integration](https://community.tableau.com/docs/DOC-10856)
2. [Using Python in Tableau Calculations](https://github.com/tableau/TabPy/blob/master/TableauConfiguration.md)
3. [Tableau Integration with Python - Step by Step](https://community.tableau.com/message/618129#618129)
4. [将表达式传递到外部服务](https://onlinehelp.tableau.com/current/pro/desktop/zh-cn/r_connection_manage.html)
6. [外部服务连接疑难解答](https://onlinehelp.tableau.com/current/pro/desktop/zh-cn/r_connection_troubleshoot.html)
