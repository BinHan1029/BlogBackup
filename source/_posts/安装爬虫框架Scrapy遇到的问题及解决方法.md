title: 安装爬虫框架Scrapy遇到的问题及解决方法
date: 2016-01-06 22:33:54
comments: true
tags:
- Python
- Scrapy
---

## 关于环境
**MAC OS** :10.10

## 安装	Scrapy
依然像安装 MySQL-python 模块一样，在终端下使用 pip 进行安装：

```ruby
	sudo pip install Scrapy
```
如果下载完成后提示 successful installed Scrapy，那么 Scrapy 就安装成功了。但通常都事与愿违，我在安装的时候遇到了这样的一个错误：

<!-- more -->

```ruby
    /tmp/xmlXPathInitIVW9Pp.c:1:10: fatal error: 'libxml/xpath.h' file not found
    #include "libxml/xpath.h"
             ^
    1 error generated.
    *********************************************************************************
    Could not find function xmlCheckVersion in library libxml2. Is libxml2 installed?
    Perhaps try: xcode-select --install
    *********************************************************************************
    error: command 'cc' failed with exit status 1

    ----------------------------------------

    Command "/System/Library/Frameworks/Python.framework/Versions/2.7/Resources/Python.app/Contents/MacOS/Python -c "import setuptools, tokenize;__file__='/private/tmp/pip-build-LpH_ul/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-XfCz9W-record/install-record.txt --single-version-externally-managed --compile" failed with error code 1 in /private/tmp/pip-build-LpH_ul/lxml

```
解决方法有如下几种：

1、终端执行命令安装或更新命令行开发工具：


```ruby
	xcode-select --install

```
2、配置路径：C_INCLUDE_PATH


```ruby
	C_INCLUDE_PATH=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2:/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2/libxml:/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include

```
3、参照官网使用如下命令安装 Scrapy

```ruby
	STATIC_DEPS=true pip install lxml

```
一般此三个方法就可解决错误成功安装Scrapy，如果还是失败，只能再去 [baidu](https://www.baidu.com/)/[google](https://www.google.com.hk)/[StackOverflow](http://stackoverflow.com/) 上寻找同样的错误。我使用的是第一种方法，成功的安装了 Scrapy。


```ruby
Successfully installed Scrapy-1.0.4 characteristic-14.3.0 lxml-3.5.0 pyasn1-0.1.9 pyasn1-modules-0.0.8 service-identity-14.0.0

```
但是在终端下使用 scrapy 命令确又出现错误

```ruby
Traceback (most recent call last):
  File "/usr/local/bin/scrapy", line 7, in <module>
    from scrapy.cmdline import execute
  File "/Library/Python/2.7/site-packages/scrapy/__init__.py", line 48, in <module>
    from scrapy.spiders import Spider
  File "/Library/Python/2.7/site-packages/scrapy/spiders/__init__.py", line 10, in <module>
    from scrapy.http import Request
  File "/Library/Python/2.7/site-packages/scrapy/http/__init__.py", line 12, in <module>
    from scrapy.http.request.rpc import XmlRpcRequest
  File "/Library/Python/2.7/site-packages/scrapy/http/request/rpc.py", line 7, in <module>
    from six.moves import xmlrpc_client as xmlrpclib
ImportError: cannot import name xmlrpc_client

```
最后查到是因为 six 这个 module 有问题导致，卸载之，重新通过 easy_install six安装后即可：

```ruby
sudo pip uninstall six	
sudo easy_install six
```
之后我们就可以使用 scrapy 命令了：

```ruby
scrapy version//查看版本
scrapy startproject test//创建项目
```