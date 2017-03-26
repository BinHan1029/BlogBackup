title: Mac 下的 Python 环境搭建及 MySQL-python 模块安装
date: 2015-12-25 14:03:21
comments: true
tags:
- Python
---

## 关于环境
**MAC OS** :10.10

**PyCharm** :推荐使用，最开始使用的是 Sublime Text 加终端的方式，但是来回切换实在太繁琐，后来切换到了 PyCharm 下，好的 IDE 真是会使效率事半功倍，更重要还提供了一些很好用的功能用于[Django](https://www.djangoproject.com/)框架开发。

## 安装 MySQL
由于之前一直做移动开发，并没有真正使用过 MySQL，而 MySQL 数据库在第一次安装后出现了各种莫名其妙的问题，无论是使用系统偏好设置启动，还是终端启动服务均失败，总是出现各种 <a name="fenced-code-block">permission failed或Can’t connect to local MySQL server through socket ‘/tmp/mysql.sock’</a>等问题，各种方法使用后无解还是尝试重新安装。MySQL[下载地址](http://dev.mysql.com/downloads/mysql/)，一定要选择对应的系统版本及位数。

重新安装前一定要将原本机器上的MySQL删除干净，建议使用终端方法进行删除，<font color=red>注意安装完成后会弹出提示，提示中 password 就是 MySQL 的 root 密码</font>：


<!-- more -->

```
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
vim /etc/hostconfig   (复制前面部分回车，然后删掉这一行: MYSQLCOM=-YES-，control+O回车保存，control+X退出编辑界面)  
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*
```


## 安装 MySQLWorkbench
* 安装完成后创建链接，此时会提示让你修改 MySQL 的 root 密码，此时输入安装 MySQL 时保存的旧密码设置新密码就可以了
* 使用系统偏好设置输入密码打开 MySQL 服务
* 使用 MySQLWorkbench 链接数据库，此时我们就可以使用 MySQLWorkbench 可视化创建我们的数据库和表了，而这些并不需要我们输入 sql 语句就可以完成

## 安装MySQL-python模块
要想使Python可以操作Mysql需要 [MySQL-python](https://pypi.python.org/pypi/MySQL-python/) 驱动，它是 Python 操作 MySQL 必不可少的模块。

* 安装此模块前必须机器上成功安装了 MySQL 数据库并可以成功开启MySQL服务
* 使用pip在终端下进行安装：<font color=red>sudo pip install MySQL-python</font>
* pip在我们安装PyCharm后既可以直接在终端下直接使用，可以快速安装一些Python的开发包

## 测试Python测试 MySQL-pytho n模块
* 事先已经使用 MySQLWorkbench 创建了数据库及一张学生表
* 需要修改对应的 root 密码及端口号、及要查询的数据库表名

```python
# coding=utf-8
import MySQLdb

try:
conn=MySQLdb.connect(host='localhost',user='root',passwd='111111',db='test',port=3306)
    cur=conn.cursor()
    students = cur.execute('SELECT * FROM student')
    info = cur.fetchmany(students)
    for ii in info:
        print ii
    cur.close()
    conn.close()
except MySQLdb.Error,e:
     print "Mysql Error %d: %s" % (e.args[0], e.args[1])

数据库查询的数据信息：
(1L, '\xe5\xbc\xa0\xe4\xb8\x89', 11L, 1L)
(2L, '\xe6\x9d\x8e\xe5\x9b\x9b', 13L, 2L)
(9L, '\xe7\x88\xb1\xe4\xb8\x8a', 11L, 2L)
(15L, '\xe9\xbb\x84\xe9\x87\x91\xe5\xae\xa2\xe6\x88\xb7', 11L, 2L)
```

下来我们就可以开始Django框架的学习了。


