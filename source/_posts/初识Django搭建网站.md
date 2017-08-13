title: 初识 django 搭建网站
date: 2016-01-27 14:38:13
tags:
- Python
- Django
---

### 开发工具

* [python](https://www.python.org/downloads/) 2.7.10
* [PyCharm](http://www.jetbrains.com/pycharm/download/) 5.0.2
* [django](https://www.djangoproject.com/download/) 1.9.1

推荐使用 [PyCharm](http://www.jetbrains.com/pycharm/download/) 直接进行 [django](https://www.djangoproject.com/download/)  框架的安装，当然 [https://www.djangoproject.com/download/](https://www.djangoproject.com/download/) 也有使用pip安装的详细说明。

 *** 

### 创建项目
![Alt text](/assets/blogImg/django_1.png)
关于 app 的概念，比如[NGA www.nga.cn](http://www.nga.cn/)下包含了[论坛http://bbs.nga.cn/](http://bbs.nga.cn/)、还会有[直播http://tv.nga.cn/](http://tv.nga.cn/)，从运维上讲代表不同的配置段 app1 和 app2 可以连接不同的数据库，占用不同的进程，不一样的 IP 地址和服务器 因为 html 是跳链，所以你感觉不到 IP 的不同。

<!-- more -->

### 目录结构
![Alt text](/assets/blogImg/django_2.png)

* setting.py  包括了系统的数据库配置、应用配置和其他配置
* urls.py  表示 web 工程 Url 映射的配置
* 子目录 demo 则是在该工程下创建的 app，包含了 models.py、tests.py 和 views.py 等文件
* templates 目录则为模板文件的目录
* manage.py 是 Django 提供的一个管理工具，可以同步数据库等等

项目的入口是 manage.py 文件，首先加载的是 settings.py 文件，这个是我们的项目配置文件。包含了 django 自带的一些 app，指定文件模板的路径，及相关的数据库配置，默认情况下使用的是 [sqlite3](http://www.sqlite.org/)，在这里我们使用 [MySQL](http://www.mysql.com/)。
需要修改对应的 <a name="fenced-code-block">DATABASES</a>定义，对应的我们需要创建一个名为 djangodemo 的数据库并保证我们的MySQL服务是开启的：

``` python
# Database
# https://docs.djangoproject.com/en/1.9/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'djangodemo',
        'USER': 'root',
        'PASSWORD': '111111',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
``` 
在 Terminal 下运行项目：

``` python
python manager.py runserver
```
 
此时我们就已经可以在浏览器中访问 [http://127.0.0.1:8000/](http://127.0.0.1:8000/)了，效果如图：

![Alt text](/assets/blogImg/django_3.png)

当然这是在 Debug 下，正式发布部署的时候服务器上运行的 django 项目应当设置 DEBUG=False，修改关闭 DEBUG, 这样不法分子便无法从 debug 信息中找到对他们有用的信息.

在设置 DEBUG=false 后, 一定要正确设置 ALLOWED_HOSTS, 避免出现 SuspiciousOperation 错误。

### 创建第一个视图函数
在 views.py 文件下添加下面的函数，我们定义个一段 html 代码，然后将其返回，当请求一个页面时，Django 把请求的 metadata 数据包装成一个 HttpRequest 对象，然后 Django 加载合适的 view 方法，把这个 HttpRequest 对象作为第一个参数传给 view 方法。任何 view 方法都应该返回一个 HttpResponse 对象。

```python
import datetime
from django.http import HttpResponse

def sayHello(request):
    s = 'Hello World!'
    current_time = datetime.datetime.now()
    html = '<html><head></head><body><h1> %s </h1><p> %s </p></body></html>' % (s, current_time)
    return HttpResponse(html)
```


修改url.py文件，进行url映射的配置:

```python
url(r'^sayHello/', sayHello),
```


此时我们就已经可以在浏览器中访问 [http://127.0.0.1:8000/sayHello/](http://127.0.0.1:8000/sayHello/) 了，效果如图：


![Alt text](/assets/blogImg/django_4.png)


###  创建第一个模板文件
在 [django](https://www.djangoproject.com/download/) 中所有的模板文件均是放在 templates 文件夹下，下面创建一个页面模板，命名为 index.html。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>粤语歌单</title>
</head>
<body>
    <h2>我常听的歌曲排名</h2>
    <ul>
        {% if list %}
            {% for value in list %}
                <li>
                   排名第{{ forloop.counter}}位:{{value}}
                </li>
            {% endfor %}
        {% endif %}
    </ul>
</body>
</html>
```


在视图文件中添加视图 view，并命名为 songList，引入 from django.shortcuts import render_to_response 模块，response 将会已我们制定的模板进行渲染，注意参数 list 需要与模板中遍历的参数相同：


```python
def songList(request):
    list = ["钟无艳", "够钟", "一丝不挂", "失忆蝴蝶", "献世"]
    return render_to_response('index.html',{'list': list})
```


在 url.py 文件中配置 url，我们可以使用这种写法，便于业务的区分：

```python
urlpatterns += [
    url(r'^songList/', songList),
]
```

此时服务之前是已经开启了，所以我们可以直接在浏览器中进行访问 [http://127.0.0.1:8000/songList/](http://127.0.0.1:8000/songList/)了，效果如图：

![Alt text](/assets/blogImg/django_5.png)