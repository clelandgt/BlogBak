# 使用Hexo搭建个人博客
作为一枚程序猿，可以通过写博客对近段时间进行总结，同时也了提高表达能力。更重要的是在分享过程中，可以结交一些朋友，所以最近打算搭建一个博客网站。通过调研，使用**Hexo**(基于node.js的静态博客框架)+**Github.io**(用于部署网站的空间资源)可快速部署博客网站。


本文基于**MacOS**环境搭建,自带**Git**,所以不需要再下载安装。

## 申请GitHub并创建相应的代码仓库

**第一步：**申请[GitHub](https://github.com/)账号，并配置本地的git环境。

	$ ssh-keygen # 生成秘钥，一路回车即可
	$ cat ~/.ssh/id_rsa.pub # 生成的秘钥

将生成的秘钥拷贝到github的SSHKyes

![](/img/使用hexo搭建个人博客_2.png)

**第二步：**创建代码仓库,用于部署网站的空间资源。
建立名为[github用户名.github.io]的仓库

![](/img/使用hexo搭建个人博客_1.png)


## Hexo安装
### 安装Node
由于Hexo是基于node.js的静态博客框架，所以需要安装Node。可通过以下两种方法实现快速安装：

**方案1** 下载[Node.js](https://nodejs.org/en/)，然后双击该文件，无脑点击下一步即可；

**方案2** 终端上使用brew安装。

	$ brew install node

### 安装Hexo

	$ sudo npm install -g hexo

### 测试是否安装成功
创建文件夹，如clelandgt.github.io
	
	$ mkdir clelandgt.github.io
初始化该博客后台,并启动。
	
	$ cd clelandgt.github.io
	$ hexo init  # 初始化
	$ hexo server

浏览器输入http://localhost:4000。如能正常访问，则表示Hexo安装成功

## Hexo基本配置
hexo的基本配置通过编辑**_config.yml**文件实现,这里主要修改以下两个部分。
### 配置网站的基本信息

```
# Site
title: 阿哲的杂货铺
subtitle: 阿哲的杂货铺
description: 技术分享与生活随笔
author: cleland
language: zh-CN
timezone: Asia/Shanghai
```
其中的language的设置参考[语言](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)；timezone的设置参考[时区](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

### 配置部署信息
```
deploy:
  type: git
  repo: https://github.com/clelandgt/clelandgt.github.io.git
  branch: master
```
如需部署到github.io，还需安装插件

	$ npm install --save hexo-deployer-git

## 新建文章并发布
运行新建文件命令，并指定文件名：

	$ hexo new 使用Hexo搭建个人博客

会生成文件：**'./source/_posts/使用Hexo搭建个人博客.md'**。然后生成相应的静态文件

	$ hexo g

部署网站

	$ hexo d

浏览器中输入clelandgt.github.io, 查看刚刚发布的文章。
如果需要想使用其他域名，可先在万网上申请一个域名，如本文申请的为www.cleland.club。然后使用阿里云的域名解析
![](/img/使用hexo搭建个人博客_3.png)
最后在source目录下创建CNAME文件，并键入需绑定的域名

```
www.cleland.club
```
hexo的安装与部署到此为止，后面是hexo基础和一些使用方法的介绍。

## Hexo目录结构

```
clelandgt.github.io  # 项目跟目录
├── _config.yml  # 配置文件：网站相关的设置，均在这里配置
├── db.json
├── node_modules
├── package-lock.json
├── package.json
├── public
├── scaffolds
├── source  # 存放资源文件，如文章或是文章涉及的一些图片。
└── themes  # 存在网站的主题，可从[官方主题](https://hexo.io/themes/)中下载喜欢的主题，然后在_config.yml中申明一下就行。
```

## Hexo常用命令

	$ hexo new 'post_name' # 新建文章
	$ hexo generate # 生成静态文件，简写g
	$ hexo deploy # 将.deplog目录部署到github上，简写d
	$ hexo server # 开启预览访问接口
	$ hexo version # 查看hexo版本
	$ hexo help # 查看帮助

## 更换主题
这里选用一个比较主流的主题yilia

	$ cd themes
	$ git clone https://github.com/litten/hexo-theme-yilia
	$ mv hexo-theme-yilia yilia
	
修改_config.yml的主题申明

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia
```


## 参考文章
[hexo官网中文文档](https://hexo.io/zh-cn/docs/)

[【全民博主】20分钟教你使用hexo搭建github博客](https://www.jianshu.com/p/e99ed60390a8)

[HEXO搭建个人博客](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)