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

**具体代码见`github`**

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

  - 导入第三方库

    ```python
    from django.shortcuts import render
    from rest_framework import status
    from rest_framework.decorators import api_view
    from rest_framework.response import Response
    from django.http import HttpResponse
    import sys
    import json  # 返回json格式数据
    import libvirt
    import subprocess 
    import random  # 生成随机字符串
    # 生成随机字符串语句
    # str = ''.join(random.sample(string.ascii_letters + string.digits, 8))
    import string
    import paramiko # ssh远程连接库
    ```

  - 创建远程连接的函数

    ```python
    # 连接远程服务器使用libvirtapi功能函数
    def conn_remote(username,hostname):
        str_conn = 'qemu+ssh://'+username+'@'+hostname+'/system'
        print(str_conn)
        conn = libvirt.open(str_conn)
        return conn
    # 连接远程服务器使用command功能函数
    def ssh_remote(ip,name,passwd):
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)
        ssh.connect(ip,22,name,passwd)
        return ssh
    ```

  - 创建虚拟机配置文件函数

    ```python
    def create_xml(name,memory,currentmemory,vcpu,mac,port):
        str1 = "<domain type='kvm'> <name>"+name +"</name> <memory>"+memory+"</memory> <currentMemory>"+currentmemory+"</currentMemory> <vcpu>"+vcpu+"</vcpu>"
        str2 = "<os> <type arch='x86_64' machine='pc'>hvm</type> <boot dev='hd'/>  </os> <features> <acpi/> <apic/>  <pae/> </features> <clock offset='localtime'/>   <on_poweroff>destroy</on_poweroff>   <on_reboot>restart</on_reboot>   <on_crash>destroy</on_crash>   <devices>	 <emulator>/usr/bin/kvm-spice</emulator>" 
        str3 = "<disk type='file' device='disk'> <driver name='qemu' type='qcow2'/> <source file='/home/ubuntu/kvm/wx/"+name+".qcow2'/> <target dev='hda' bus='ide'/> </disk>"
        str4 = "<interface type='bridge'> <mac address='"+mac+"' /> <source bridge='virbr0'/> </interface> <input type='mouse' bus='ps2'/> <graphics type='vnc' port='"+port+"' autoport='no' listen = '0.0.0.0' keymap='en-us'/>   </devices></domain>"
        xml_dsc = str1 + str2 + str3 + str4 
        print(xml_dsc)
        return xml_dsc
    ```

  - ***全部代码如下：***

    ```python
    from django.shortcuts import render
    from rest_framework import status
    from rest_framework.decorators import api_view
    from rest_framework.response import Response
    from django.http import HttpResponse
    import sys    
    import json  # 返回json格式数据
    import libvirt
    import subprocess 
    import random  # 生成随机字符串
    import string
    import paramiko # ssh远程连接库
    # Create your views here.
    
    # 连接远程服务器使用libvirtapi功能函数
    def conn_remote(username,hostname):
        str_conn = 'qemu+ssh://'+username+'@'+hostname+'/system'
        print(str_conn)
        conn = libvirt.open(str_conn)
        return conn
    # 连接远程服务器使用command功能函数
    def ssh_remote(ip,name,passwd):
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)
        ssh.connect(ip,22,name,passwd)
        return ssh
    # 创建虚拟机配置文件函数
    def create_xml(name,memory,currentmemory,vcpu,mac,port):
        str1 = "<domain type='kvm'> <name>"+name +"</name> <memory>"+memory+"</memory> <currentMemory>"+currentmemory+"</currentMemory> <vcpu>"+vcpu+"</vcpu>"
        str2 = "<os> <type arch='x86_64' machine='pc'>hvm</type> <boot dev='hd'/>  </os> <features> <acpi/> <apic/>  <pae/> </features> <clock offset='localtime'/>   <on_poweroff>destroy</on_poweroff>   <on_reboot>restart</on_reboot>   <on_crash>destroy</on_crash>   <devices>	 <emulator>/usr/bin/kvm-spice</emulator>" 
        str3 = "<disk type='file' device='disk'> <driver name='qemu' type='qcow2'/> <source file='/home/ubuntu/kvm/wx/"+name+".qcow2'/> <target dev='hda' bus='ide'/> </disk>"
        str4 = "<interface type='bridge'> <mac address='"+mac+"' /> <source bridge='virbr0'/> </interface> <input type='mouse' bus='ps2'/> <graphics type='vnc' port='"+port+"' autoport='no' listen = '0.0.0.0' keymap='en-us'/>   </devices></domain>"
        xml_dsc = str1 + str2 + str3 + str4 
        print(xml_dsc)
        return xml_dsc
    # 创建虚拟机硬盘文件(.qcow2)
    def create_qcow2(ssh,name):
        str = "sudo cp /home/ubuntu/kvm/ubuntu.qcow2 /home/ubuntu/kvm/wx/"+name+".qcow2"
        print(str)
        stdin, stdout, stderr = ssh.exec_command(str)
        return 0
    # 获得虚拟机参数函数
    def get_info(conn,name):
        pool = conn.lookupByName(name)
        info = pool.info()
        dominfo = {'name':pool.name(),'UUID':pool.UUIDString(),'Autostart':str(pool.autostart()),
                    'Is active':str(pool.isActive()),'Is persistent':str(pool.isPersistent()),'Pool state':str(info[0]),
                    'Capacity':str(info[1]),'Allocation':str(info[2]),'Available':str(info[3])}
        return dominfo
    
    # 获得远程主机上已创建的虚拟机列表
    @api_view(['GET','POST'])
    def kvm_list(request,format=None):
        conn = conn_remote('ubuntu','172.22.107.51')
        if conn == None:
            print('Failed to open connection to qemu:///system', file=sys.stderr)
            exit(1)
    # 返回虚拟机列表
        if request.method == 'GET':
            if conn == None:
                print('Failed to open connection to qemu:///system', file=sys.stderr)
                exit(1)
            domains = conn.listAllDomains(0)
            if len(domains) != 0:
                for domain in domains:
                    print(' '+domain.name())
            conn.close()
            return render(request,'vps.html',{'data':domains})
    # 根据页面传回的json数据创建虚拟机
        if request.method == 'POST':
            req = json.loads(request.body.decode('utf-8'))
            # 随机生成8位字符串作为虚拟机名称
            #kvmname = ''.join(random.sample(string.ascii_letters + string.digits, 8))
            kvmname = "kmdhEbu9"
            print(kvmname)
            memory = str(req['memory'])
            currentmemory = str(req['currentmemory'])
            vcpu = str(req['vcpu'])
            # mac 和 port之后会从数据库中获得
            mac = "52:54:00:33:a4:31"
            port = "5931"
            # 根据页面的json数据创建xml配置文件
            xml_dsc = create_xml(kvmname,memory,currentmemory,vcpu,mac,port)
            ssh = ssh_remote("172.22.107.51","ubuntu","ubuntu")
            #create_qcow2(ssh,kvmname)
            dom = conn.defineXML(xml_dsc)
            dom.create()
            dominfo = get_info(conn,kvmname)
            conn.close() 
            return HttpResponse(json.dumps(dominfo),content_type="application/json")
        # 虚拟机的关闭，删除，更新配置操作
    @api_view(['GET','DELETE','PATCH','PUT'])
    def kvm(request,name,format=None):
        conn = conn_remote('ubuntu','172.22.107.51')
        if conn == None:
            print('Failed to open connection to qemu:///system', file=sys.stderr)
            exit(1)
        
        if request.method == 'DELETE':
            pool = conn.lookupByName(name)
            flag = pool.isActive()
            if flag == True:
                pool.destroy()
            pool.undefine()
            pool1 = conn.lookupByName(name)
            if pool1 == None:
                resp = {'name':name,'delete':1}
            else:
                resp = {'name':name,'delete':0}
            conn.close()
            return HttpResponse(json.dumps(resp),content_type="application/json")
    
        # 根据虚拟机名称删除虚拟机
        if request.method == 'PATCH':
            pool = conn.lookupByName(name)
            pool.destroy()
            flag = pool.isActive()
            conn.close()
            resp = {'name':name,'status':flag}
            return HttpResponse(json.dumps(resp),content_type="application/json")
           
        # 根据虚拟机名称获得虚拟机信息
        if request.method == 'GET':
            dominfo = get_info(conn,name)
            return HttpResponse(json.dumps(dominfo),content_type="application/json")
        # 根据页面传回的json数据修改虚拟机配置
        if request.method == 'PUT':
            req = json.loads(request.body.decode('utf-8'))
            kvmname = req['name']
            memory = str(req['memory'])
            currentmemory = str(req['currentmemory'])
            vcpu = str(req['vcpu'])
            # 根据页面的json数据创建xml配置文件
            xml_dsc = create_xml(kvmname,memory,currentmemory,vcpu)
            pool = conn.lookupByName(name)
            flag = pool.isActive()
            if flag == True:
                pool.destroy()
            pool.undefine()
            dom = conn.defineXML(xml_dsc)
            dom.create()
            dominfo = get_info(conn,name)
            conn.close()
            return HttpResponse(json.dumps(dominfo),content_type="application/json")
    ```






  ## 现存的问题：

  - 目前还没有创建数据库，在创建虚拟机时`mac`和`port`是代码中给定的，之后会从数据库中得到。
  - 在创建虚拟机的过程中存在一个硬盘文件拷贝问题，目前的拷贝文件大小为13G，正在寻找替代方案。（cloudimages）
  - 未完待续


