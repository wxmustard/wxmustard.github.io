---
title: django_REST_framework
author: mustard
date: 2017-11-28 10:43:31
tags:
  - ubuntu
  - django
categories:
  - 技术
---

> http://blog.csdn.net/ppppfly/article/details/51086122

##  搭建环境

> 参考博客 http://blog.csdn.net/ppppfly/article/details/51086122

## 搭建环境

- 使用插件`postman`进行测试

- 使用独立`python`的环境

  ```bash
  virtualenv env
  source env/bin/activate
  ```

- 安装第三方库

  ```bash
  pip3 install django
  pip3 install djangorestframework
  pip3 install pygments
  ```

- 创建`Django`项目以及`app`

  ```bash
  django-admin startproject vm 
  (venv) ➜  vm python3 manage.py startapp kvm
  ```

- 将kvm app 添加到`INSTALLED_APPS`中 `vm/vm/setting.py`

  ```python
  INSTALLED_APPS = [
  	'rest_framework',
      'kvm.apps.KvmConfig',
  ]
  ```

## 开始

- 创建模型`vm/kvm/models.py`

  ```python
  from django.db import models
  from pygments.lexers import get_all_lexers
  from pygments.styles import get_all_styles
  # Create your models here.
  LEXERS = [item for item in get_all_lexers() if item[1]]
  LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
  STYLE_CHOICES = sorted((item, item) for item in get_all_styles())
  class Kvm(models.Model):
      created = models.DateTimeField(auto_now_add=True)
      title = models.CharField(max_length=100, blank=True, default='')
      code = models.TextField()
      linenos = models.BooleanField(default=False)
      language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
      style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

      class Meta:
          ordering = ('created',)
  ```

- 将数据同步至数据库并完成数据的迁移

  ```bash
   python3 manage.py makemigrations snippets
   python3 manage.py migrate
  ```


- 使用模型序列器（ModelSerializers）创建`vm/serializers.py`

  ```python
  from rest_framework import serializers
  from kvm.models import Kvm, LANGUAGE_CHOICES, STYLE_CHOICES

  class KvmSerializer(serializers.ModelSerializer):
      class Meta:
          model = Kvm
          fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
  ```

  - 模型序列器是一种创建序列器的捷径
  - 自动声明了一套字段
  - 默认的实现了create()和update()

- 修改views.py

  ```python
  from rest_framework import status
  from rest_framework.decorators import api_view
  from rest_framework.response import Response
  from kvm.models import Kvm
  from kvm.models import host
  from kvm.serializers import KvmSerializer,hostSerializer

  @api_view(['GET','POST'])
  def kvm_list(request,format=None):
      """
      List all code snippets, or create a new snippet.
      """
      if request.method == 'GET':
          kvms = Kvm.objects.all()
          serializer = KvmSerializer(kvms, many=True)
          return Response(serializer.data)

      elif request.method == 'POST':
          serializer = KvmSerializer(data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data, status=status.HTTP_201_CREATED)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

  @api_view(['GET','PUT','DELETE'])
  def kvm_detail(request, pk,format=None):
      """
      Retrieve, update or delete a code snippet.
      """
      try:
          kvm = Kvm.objects.get(pk=pk)
      except Kvm.DoesNotExist:
          return Response(status=status.HTTP_404_NOT_FOUND)

      if request.method == 'GET':
          serializer = KvmSerializer(kvm)
          return Response(serializer.data)

      elif request.method == 'PUT':
          serializer = KvmSerializer(kvm, data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

      elif request.method == 'DELETE':
          kvm.delete()
          return HttpResponse(status=status.HTTP_204_NO_CONTENT)
  ```

- 创建apps/urls.py

  ```python
  from django.conf.urls import url
  from kvm import views
  from rest_framework.urlpatterns import format_suffix_patterns
  urlpatterns = [
      url(r'^kvms/$',views.kvm_list),
      url(r'^kvms/(?P<pk>[0-9]+)/$',views.kvm_detail),
  ]
  urlpatterns = format_suffix_patterns(urlpatterns)
  ```

- 修改projects/urls.py

  ```python
  from django.conf.urls import url
  from django.contrib import admin
  from django.conf.urls import url , include

  urlpatterns = [
     url(r'^',include('kvm.urls')),
  ]
  ```

- 进入终端

  ```bash
  python3 manage.py shell
  >>> from kvm.models import Kvm
  >>> kvm_list = Kvm.objects.all()
  >>> print(kvm_list)
  ```




## 在`python`代码中远程执行系统命令

```python
import paramiko # ssh远程连接库
# 连接远程服务器使用command功能函数
def ssh_remote(ip,name,passwd):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)
    ssh.connect(ip,22,name,passwd)
    return ssh
# 以下代码是执行命令语句
ssh = ssh_remote("172.22.107.51","ubuntu","ubuntu")
str = "sudo cp /home/ubuntu/kvm/ubuntu.qcow2 /home/ubuntu/kvm/wx/"+name+".qcow2"
print(str)
stdin, stdout, stderr = ssh.exec_command(str)
# 获得执行命令后的返回值
# 将返回值作为一个整体打印出来
# 1、一次性读取整个文件。
# 2、自动将文件内容分析成一个行的列表。
print(stdout.readlines())
# 一行一行打印返回值
# 1、readline()每次读取一行，比readlines()慢得多
# 2、readline()返回的是一个字符串对象，保存当前行的内容
print(stdout.readline())
```

### Q&A

- `ImportError: No module named 'mustard111'`

原因：找不到正确路径，可以使用上级目录导入的方法