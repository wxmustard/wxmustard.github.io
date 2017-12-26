---
title: kvm-django-restframework
author: mustard
date: 2017-12-26 18:32:10
tags:
  - libvirt 
  - django
categories:
  - 技术
---

## 目标

使用`django rest framework`实现虚拟机的创建、修改、关闭、销毁等功能

## 实现步骤

### 搭建环境

- 安装`django、djangorestframework`

  ```bash
  pip3 install django
  pip3 install djangorestframework
  pip3 install pygments
  ```

- 创建项目以及APP

  ```bash
  django-admin startproject kvm
  cd kvm
  python3 manage.py startapp kvm
  ```

### 具体的实验步骤

- 将`app`添加至项目中

  ```python
  # 将'vm.apps.VmConfig'添加到kvm/settings.py中
  INSTALLED_APPS = [
      'vm.apps.VmConfig',
  ]
  ```

- 修改项目的url

  ```python
  # 打开kvm/urls.py,将以下语句添加进去
  # 提示：vm/urls.py需要自行创建
  from django.conf.urls import url , include
  urlpatterns = [
      path('admin/', admin.site.urls),
      url(r'^',include('vm.urls'))
  ]
  ```

- 创建vm/urls.py

  ```python
  # url是预先设计好的
  from django.conf.urls import url
  from vm import views
  from rest_framework.urlpatterns import format_suffix_patterns
  urlpatterns = [
      url(r'^v1/vps/$',views.kvm_list),
      url(r'^v1/vps/kvmname/(?P<name>[a-zA-Z0-9]+)/$',views.kvm),
  ]
  urlpatterns = format_suffix_patterns(urlpatterns)
  ```

- 修改vm/views.py

  - 创建远程连接的函数

  ​

