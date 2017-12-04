---
title: django
author: mustard
date: 2017-11-28 10:43:31
tags:
  - ubuntu
  - django
categories:
  - 技术

---

## 搭建多个互不干扰的开发环境

>在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.4。所有第三方的包都会被`pip`安装到Python3的`site-packages`目录下。
>
>如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？
>
>这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境.

- 安装`virtualenv`

  ```bash
  sudo pip install virtualenv virtualenvwrapper
  ```

- 更改`.zshrc`

  ```bash
  export WORKON_HOME=/home/mustard/.virtualenvs
  export PROJECT_HOME=/home/mustard/workspace
  export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python
  source /usr/local/bin/virtualenvwrapper.sh
  # 使更改生效
  source .zshrc
  ```

- 创建一个新的项目

  ```bash
  # 创建项目目录
  mkdir myproject
  cd myproject/
  # 创建一个独立的Python运行环境，命名为venv
  virtualenv --no-site-packages venv
  # virtualenv命令是创建一个独立的Python运行环境
  # --no-site-packages参数是创建一个干净的不包含第三方包的Python运行环境

  #进入venv环境
  source venv/bin/activate
  # 此时可以使用pip install等命令了

  ```

  ![ ](https://vgy.me/nS94OF.png)

  ​


## 安装`Django`

- 安装`bpython`，添加自动补全功能

  ```python
  # 进入venv环境后
  # 1.升级pip
  pip install --upgrade pip
  # 2.安装Django
  pip install Django
  # 3.安装bpython
  pip install bpython
  # 进入bpython，查询django版本
  >>> import django
  >>> django.VERSION
  (1, 11, 7, 'final', 0)
  ```


### `Django`基本命令

- ```bash
  # 创建新项目 mysite
  django-admin startproject mysite
  # 创建新应用（app），learn
  python manage.py startapp learn
  # 将新定义的app加到settings.py中的INSTALL_APPS中
  # 修改mysite/mysite/settings.py
  INSTALLED_APPS = (
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
   
      'learn',
  )
  # 新建的 app 如果不加到 INSTALL_APPS 中的话, django 就不能自动找到app中的模板文件(app-name/templates/下的文件)和静态文件(app-name/static/中的文件)

  ```

- 定义视图

  ```python
  # learn/views.py 
  from django.shortcuts import render
  from django.http import HttpResponse
   
  def add(request):
      a = request.GET（'a',0）
      b = request.GET('b',0)
      c = int(a)+int(b)
      return HttpResponse(str(c))
  ```

- 添加网址来对应视图函数

  ```python
  # mysite/urls.py
  from django.conf.urls import url
  from django.contrib import admin
  from learn import views as learn_views  #new
  urlpatterns = [
      url(r'^add/$',learn_views.add,name='add'),  #new
      url(r'^admin/', admin.site.urls),
  ]
  ```

  `Django中`的 `urls.py `用的是**正则**进行匹配的

- 打开开发服务器并访问

  ```bash
  python manage.py runserver
  # python manage.py runserver 8001 
  link http://127.0.0.1:8000/add/?a=4&b=5
  ```

- 初始化数据库

  ```bash
  python3 manage.py migrate 
  ```

- 忘记数据库的用户名和密码

  ```bash
  python3 manage.py createsuperuser
  ```

- 同步（合并）数据库

  ```bash
  python3 manage.py makemigrations
  python3 manage.py migrate
  ```

- 文件结构如下

  ```bash
  mysite
  ├── db.sqlite3 # 数据库文件
  ├── learn # 应用app
  │   ├── admin.py # 管理数据库后台
  │   ├── apps.py
  │   ├── __init__.py
  │   ├── migrations  
  │   │   ├── __init__.py
  │   │   └── __pycache__
  │   │       └── __init__.cpython-35.pyc
  │   ├── models.py # 定义数据库中的表，类似于MVC中的M
  │   ├── __pycache__
  │   │   ├── admin.cpython-35.pyc
  │   │   ├── __init__.cpython-35.pyc
  │   │   ├── models.cpython-35.pyc
  │   │   └── views.cpython-35.pyc
  │   ├── tests.py # 测试代码文件
  │   └── views.py # 响应客户请求返回html页面，代码逻辑处理的主要地点
  ├── manage.py # 管理项目
  └── mysite
      ├── __init__.py
      ├── __pycache__
      │   ├── __init__.cpython-35.pyc
      │   ├── settings.cpython-35.pyc
      │   ├── urls.cpython-35.pyc
      │   └── wsgi.cpython-35.pyc
      ├── settings.py  # 整个网站的配置文件
      ├── urls.py  # 一个url对应与views.py中的一个函数
      └── wsgi.py  # Python应用程序或框架和web服务器之间的接口
  ```



### 模板

- 在`app`目录下创建模板文件夹`templates`

  ```bash
  learn
  ├── admin.py
  ├── apps.py
  ├── __init__.py
  ├── migrations
  │   ├── __init__.py
  │   └── __pycache__
  │       └── __init__.cpython-35.pyc
  ├── models.py
  ├── __pycache__
  │   ├── admin.cpython-35.pyc
  │   ├── __init__.cpython-35.pyc
  │   ├── models.cpython-35.pyc
  │   └── views.cpython-35.pyc
  ├── templates
  │   └── home.html
  ├── tests.py
  └── views.py
  ```

#### 显示简单字符串

- 修改`views.py`

  ```python
  from django.shortcuts import render
  #coding:utf-8
  from django.http import HttpResponse
  def home(request):
      string = u"你是无意穿堂风，却偏偏引山洪"
      return render(request, 'home.html', {'string': string}) 
  ```

- 修改`templates/home.html`

  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>欢迎光临</title>
  </head>
  <body>
  {{string}}
  </body>
  </html>
  ```

#### 还有很多种模板显示方法，未完待续

### 搭建`Django`的过程

- 创建项目

- 创建应用

- 将应用添加到`setting.py`文件中

- 编写`app/view.py`文件

- 修改`urls.py`文件，使之对应到应用的`views.py` 中的函数

- 运行项目 runserver

- 打开浏览器访问

  ​

## 搭建一个丑哭了的博客

- 创建项目等前期工作

  ```bash
  django-admin startproject blog
  cd blog
  python3 manage.py startapp blog_wx
  # 初始化数据库
  python3 manage.py migrate 
  # 添加app到blog/blog/setting.py
  vim blog/blog/setting.py
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'blog_wx',
  ]
  ```

- 访问后台应用

  ```bash
  python3 manage.py runserver
  ```

  打开浏览器访问http://127.0.0.1:8000/admin/，输入用户名和密码，进入后台管理界面

  ![](https://vgy.me/KMNqpO.png)

- 设计数据库（blog/blog_wx/models.py）

  ```python
  from django.db import models
  from django.contrib import admin
  # Create your models here.
  class BlogsPost(models.Model):
      title = models.CharField(max_length = 150)
      body = models.TextField()
      timestamp = models.DateTimeField()
   
  class BlogsPostAdmin(admin.ModelAdmin):
      list_display = ('title','timestamp')

  admin.site.register(BlogsPost,BlogsPostAdmin)
  ```

- 再次初始化数据库

  ```bash
  python3 manage.py makemigrations blog_wx
  python3 manage.py migrate   
  # 不建议使用python3 manage.py syncdb
  ```

- 再次后台应用

  ```bash
  python3 manage.py runserver
  ```

  ![](https://vgy.me/8hzmnx.png)

  ![](https://vgy.me/nTGaQG.png)

  > **创建blog的公共部分**
  >
  > 从Django的角度看，一个页面具有三个典型的组件：
  >
  > 一个模板（template）：模板负责把传递进来的信息显示出来。
  >
  > 一个视图（view）：视图负责从数据库获取需要显示的信息。
  >
  > 一个URL模式：它负责把收到的请求和你的试图函数匹配，有时候也会向视图传递一些参数。

- 创建模板

  - 在blog_wx(app)目录下创建`templates`文件夹

  - base.html 内容如下

    ```html
    <html>
        <style type="text/css">
          body{color:rgb(168, 166, 35);background:rgb(74, 99, 49);padding:0 5em;margin:0}
          h1{padding:2em 1em;background:#675}
          h2{color:rgb(52, 54, 51);border-top:1px dotted #fff;margin-top:2em}
          p{margin:1em 0}
        </style>
       
        <body>
          <h1>Mustard</h1>
          <h3>你是无意穿堂风，却偏偏引山洪</h3>
          {% block content %}
          {% endblock %}
        </body>
    </html>
    ```

  - index.html 内容如下

    ```html
    {% extends "base.html" %}
        {% block content %}
            {% for post in blog_list %}
            <h2>{{ post.title }}</h2>
            <p>{{ post.timestamp }}</p>
            <p>{{ post.body }}</p>
            {% endfor%}
        {% endblock %}
    ```

  - 修改视图函数 blog/blog_wx/views.py

    ```python
    from django.shortcuts import render
    from blog_wx.models import BlogsPost
    from django.shortcuts import render_to_response
    #coding:utf-8
    # Create your views here.
    def index(request):
        blog_list = BlogsPost.objects.all()
        return render_to_response('index.html',{'blog_list':blog_list})
    ```

  - 修改网址入口文件 blog/blog/urls.py

    ```python
    #coding=utf-8
    from django.conf.urls import url
    from django.contrib import admin
    from blog_wx import views as blog_wx
    urlpatterns = [
        url(r'^index/', blog_wx.index),
        url(r'^admin/', admin.site.urls),
    ]
    ```

- 查看前台页面(丑哭了的页面就在下面)

  ![](https://vgy.me/PGFZzw.png)

- 博客目录结构如下

  ```bash
  blog
  ├── blog
  │   ├── __init__.py
  │   ├── __pycache__
  │   │   ├── __init__.cpython-35.pyc
  │   │   ├── settings.cpython-35.pyc
  │   │   ├── urls.cpython-35.pyc
  │   │   └── wsgi.cpython-35.pyc
  │   ├── settings.py
  │   ├── urls.py
  │   └── wsgi.py
  ├── blog_wx
  │   ├── admin.py
  │   ├── apps.py
  │   ├── __init__.py
  │   ├── migrations
  │   │   ├── 0001_initial.py
  │   │   ├── __init__.py
  │   │   └── __pycache__
  │   │       ├── 0001_initial.cpython-35.pyc
  │   │       └── __init__.cpython-35.pyc
  │   ├── models.py
  │   ├── __pycache__
  │   │   ├── admin.cpython-35.pyc
  │   │   ├── __init__.cpython-35.pyc
  │   │   ├── models.cpython-35.pyc
  │   │   └── views.cpython-35.pyc
  │   ├── templates
  │   │   ├── base.html
  │   │   └── index.html
  │   ├── tests.py
  │   └── views.py
  ├── db.sqlite3
  └── manage.py
  ```

  ​

##使用`django`调用`python`脚本

- 连接远程服务器

  ```python
  # 首先安装必要的python第三方库
  sudo apt install python-crypto python-paramiko python-devel
  # 具体的使用
  import paramiko
  ssh = paramiko.SSHClient()
  ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
  # ssh.connect("111.186.106.99",22,"ubuntu","ubuntu")
  ssh.connect("远程主机ip",端口,"用户名","密码")
  # 执行命令
  stdin, stdout, stderr = ssh.exec_command("xml_create")
  ```

- 使用上述方法完成远程创建，销毁虚拟机

  ```python
  from django.shortcuts import render
  from django.http import HttpResponse
  from django.shortcuts import render_to_response
  import paramiko
  # Create your views here.

  # 连接远程服务器功能函数
  def ssh_remote(ip,name,passwd):
      ssh = paramiko.SSHClient()
      ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)
      ssh.connect(ip,22,name,passwd)
      return ssh

  def kvm(request):
      return render_to_response('kvm.html')
  def kvm_create(request):
      if 'create_name' in request.GET:
          name = request.GET['create_name']
      else:
          name = '空'
      #ssh = paramiko.SSHClient()
      #ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
      #ssh.connect("111.186.106.99",22,"ubuntu","ubuntu")
      ssh = ssh_remote("111.186.106.99","ubuntu","ubuntu")
      xml_create = "/home/ubuntu/kvm/wx/xml_create.sh " + name 
      print(xml_create)
      stdin, stdout, stderr = ssh.exec_command(xml_create)
      create_py = "python3 /home/ubuntu/kvm/wx/wx_create.py vm" + name
      stdin, stdout, stderr = ssh.exec_command(create_py)
      message = "虚拟机已创建成功，地址为192.168.110.1" + name 
      return HttpResponse(message)

  def kvm_destroy(request):
      if 'down_name' in request.GET:
          name = request.GET['down_name']
      else:
          name = '空'
      ssh = ssh_remote("111.186.106.99","ubuntu","ubuntu")
      down_py = "python3 /home/ubuntu/kvm/wx/wx_shut.py vm" + name
      stdin, stdout, stderr = ssh.exec_command(down_py)
      message = "虚拟机已关闭"
      return HttpResponse(message)

  ```

  ​