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

  ![](https://vgy.me/nS94OF.png)

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

- 文件结构如下

  ```bash
  mysite
  ├── db.sqlite3
  ├── learn
  │   ├── admin.py
  │   ├── apps.py
  │   ├── __init__.py
  │   ├── migrations
  │   │   ├── __init__.py
  │   │   └── __pycache__
  │   │       └── __init__.cpython-35.pyc
  │   ├── models.py
  │   ├── __pycache__
  │   │   ├── admin.cpython-35.pyc
  │   │   ├── __init__.cpython-35.pyc
  │   │   ├── models.cpython-35.pyc
  │   │   └── views.cpython-35.pyc
  │   ├── tests.py
  │   └── views.py
  ├── manage.py
  └── mysite
      ├── __init__.py
      ├── __pycache__
      │   ├── __init__.cpython-35.pyc
      │   ├── settings.cpython-35.pyc
      │   ├── urls.cpython-35.pyc
      │   └── wsgi.cpython-35.pyc
      ├── settings.py
      ├── urls.py
      └── wsgi.py
  ```

  ​

