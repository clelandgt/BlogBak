---
title: pyenv管理python虚拟环境
date: 2018-03-22 23:21:41
tags: python
---
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive-hive使用压缩/hive-hive使用压缩1.jpg)

安装环境为macos

<!-- more -->
## 安装pyenv
自动安装pyenv

``` shell
$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

安装成功后在`.bash_profile`(`.bashrc`)中添加三行添加自动补全。

``` 
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

```

## 安装python
安装一个python版本到$PYENV_ROOT/versions路径下。

``` shell 
$ pyenv install -v 2.7.14
```

安装完成后，刷新数据库。并查看安装的python版本

``` shell
$ pyenv rehash
$ pyenv versions
* system (set by /Users/cleland/.pyenv/version)
2.7.14
```

设置全局python版本

``` shell
$ pyenv global 2.7.14
$ python  # 确认python版本
Python 2.7.14 (default, Mar 22 2018, 11:12:38)
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.39.2)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

可以安装和管理多个Python版本，TODO: 待补充相关内容。

## Pyenv-virtualenv插件的使用
使用以上方式安装pyenv，会自动安装virtualenv插件。

### 结合virtualenv创建虚拟环境
以2.7.14版本的python创建一个名为test_env的虚拟环境

```
$ pyenv virtualenv 2.7.14 test_env   # 创建一个名称test_env的虚拟环境
$ pyenv virtualenvs  # 查看当前存在的虚拟环境
  2.7.14/envs/test_env (created from /Users/cleland/.pyenv/versions/2.7.14)
  test_env (created from /Users/cleland/.pyenv/versions/2.7.14)
```

删除虚拟环境最直接粗暴的方式是：直接删除虚拟环境的物理存储

```
$ rm -rf ~/.pyenv/versions/myenv/
```

### virtualenv的使用
```
$ pyenv activate test_env  # 激活test_env虚拟环境
$ pyenv deactivate  # 退出虚拟环境
```

## 更改pip源
国内使用pip下载安装第三方库，比较慢。可以使用aliyun提供的镜像。创建如下的`$HOME/.pip/pip.conf`文件。

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

## 参考
- [使用pyenv管理Python版本](http://einverne.github.io/post/2017/04/pyenv.html) 
- [使用pyenv搭建python虚拟环境](https://www.hi-linux.com/posts/63429.html)
