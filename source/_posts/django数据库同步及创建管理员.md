title: django数据库同步及创建管理员
date: 2016-01-28 11:43:03
tags:
- Python
- Django
---

## 数据库同步
首先在项目setting.py下对需要修改对应的<a name="fenced-code-block">DATABASES</a>定义，对应的我们需要创建一个名为djangodemo的数据库并保证我们的MySQL服务是开启的：

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
<!-- more -->

执行命令：

``` python
	python manage.py makemigrations
	python manage.py migrate
```

使用安装MySQLWorkbench打开数据库查看表，django已经自动为我们创建基本用户、组、日志、seesion的表：
![Alt text](/assets/blogImg/django_8.png)

## 创建admin帐号
* 首先我们要新建一个用户名，用来登陆管理网站，可以使用如下命令：

	python manage.py createsuperuser
* 输入想要使用的用户名：

	Username (leave blank to use 'administrator'): user01
* 输入email：
 
	Email address: (在这里输入你的自己的邮箱帐号)
* 输入密码，需要输入两次，并且输入密码时不会显示出来：

	Password:
	Password (again):
* 当两次密码都相同的时候，就会提示超级帐号创建成功。

	Superuser created successfully.
* 运行服务：

	python manage.py runserver
	
浏览器地址栏输入：[http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)

![Alt text](/assets/blogImg/django_6.png)

登陆成功后我们既可以添加新的用户和对用户的权限进行添加移除等操作：

![Alt text](/assets/blogImg/django_7.png)

查看auth_user表我们就可以看到我们创建的admin超级管理员账号及一个子用户：

![Alt text](/assets/blogImg/django_9.png)
