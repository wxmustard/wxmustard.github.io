---
defineXMLtitle: libvirt
author: mustard
date: 2017-11-27 10:18:00
tags:
  - python
  - libvirt
  - ubuntu
categories:
  - 技术
---

## libvirt api

### libvirt python api

```bash
# 安装libvirt
# 首先ubuntu系统需要安装libvirt-dev
sudo apt install libvirt-dev
# ubuntu只有在安装了libvirt-dev后才能正常安装libvirt-python
sudo pip3 install libvirt-python
```
### 创建域

> There are up to three methods involved in provisioning guests. The createXML method will create
> and immediately boot a new transient guest domain. When this guest domain shuts down, all trace of
> it will disappear. The defineXML method will store the configuration for a persistent guest domain.
> The create method will boot a previously defined guest domain from its persistent configuration.
> One important thing to note, is that the defineXML command can be used to turn a previously booted
> transient guest domain, into a persistent domain. This can be useful for some provisioning scenarios
> that will be illustrated later.

- 首先编辑好虚拟机的配置文件vm31

  ```xml
  <domain type='kvm'>
      <name>vm31</name>
      <memory>4096000</memory>
      <currentMemory>4096000</currentMemory>
      <vcpu>2</vcpu>

     <os>
        <type arch='x86_64' machine='pc'>hvm</type>
        <boot dev='hd'/> 
     </os>

     <features>
       <acpi/>
       <apic/>
       <pae/>
     </features>

     <clock offset='localtime'/>
     <on_poweroff>destroy</on_poweroff>
     <on_reboot>restart</on_reboot>
     <on_crash>destroy</on_crash>
     <devices>
  	 <emulator>/usr/bin/kvm-spice</emulator>
  	 <disk type='file' device='cdrom'>
  	 	<driver name='qemu' type='raw'/>
  	 	<source file="/home/ubuntu/kvm/ubuntu-16.04.3-server-amd64.iso"/> 
          <target dev='hdc' bus='ide'/>
  	 	<readonly/>
  	 	<address type='drive' controller='0' bus='1' target='0' unit='0'/>
  	 </disk>
  	 <disk type='file' device='disk'>
  	ls
         <driver name='qemu' type='qcow2'/>
          <source file='/home/ubuntu/kvm/wx/wx.qcow2'/>
          <target dev='hda' bus='ide'/>
       </disk>
  	
       <interface type='bridge'>
  		<mac address="52:54:00:33:a4:31" />
        <source bridge='virbr0'/>
       </interface>

      <input type='mouse' bus='ps2'/>
       <graphics type='vnc' port='5931' autoport='no' listen = '0.0.0.0' keymap='en-us'/>
     </devices>
  </domain>

  ```

#### 方法一

- 创建`create.py`文件

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  xmldesc=open('/home/ubuntu/kvm/wx/vm31','r').read()
  dom = conn.createXML(xmldesc, 0);     
  print"Domian:id %d running %s"%(dom.ID(),dom.OSType())
  ```

#### 方法二

- 创建`create.py`

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  xmldesc=open('vm31','r').read()
  dom = conn.defineXML(xmldesc)
  dom.create()
  conn.close()
  exit(0)
  ```


- 运行`create.py`

  ```bash
  python2.7 create.py
  ```

#### 区别

|                createXML                 |                defineXML                 |
| :--------------------------------------: | :--------------------------------------: |
| 创建并立即启动一个临时域，当临时域关闭时，所有的痕迹都会消失。（类似于启动临时虚拟机） | 创建一个持久的域配置，使用create方法将域从其持久配置中引导出来(也就是启动域) |
|            virsh create ~.xml            |            virsh define ~.xml            |



###查看域的详细信息

- 创建`get_source.py`

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  domName = 'vm11'
  pool = conn.lookupByName(domName)
  if pool == None:
      print('can not find',domName)
  info = pool.info()
  print('Pool: '+pool.name())
  print(' UUID: '+pool.UUIDString())
  print(' Autostart: '+str(pool.autostart()))
  print(' Is active: '+str(pool.isActive()))
  print(' Is persistent: '+str(pool.isPersistent()))
  print(' Pool state: '+str(info[0]))
  print(' Capacity: '+str(info[1]))
  print(' Allocation: '+str(info[2]))
  print(' Available: '+str(info[3]))
  ```

- 运行`wx_getsource.py`

  ```bash
  ➜  wx python3 wx_getsource.py
  Pool: vm11
   UUID: cc74ec90-68ba-404c-a67e-6bcfe44339aa
   Autostart: 0
   Is active: 1
   Is persistent: 1
   Pool state: 1
   Capacity: 4096000
   Allocation: 4096000
   Available: 4
  ```

### 关闭域

> Stopping refers to the process of halting a running guest. A guest can be stopped by two methods:shutdown and destroy.

- 创建`wx_shut.py`

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  domName = 'vm31'
  pool = conn.lookupByName(domName)
  pool.shutdown()
  # 或者使用pool.destroy()
  conn.close()
  exit(0)
  ```

- 运行停止文件即可关闭虚拟连接远程服务器功能函数

  def ssh_remote(ip,name,passwd):
      ssh = paramiko.SSHClient()
      ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy)
      ssh.connect(ip,22,name,passwd)
      return ssh

- 机

- 区别

  | shutdown()       | destroy()       |
  | ---------------- | --------------- |
  | 正常关机，需要一点时间      | 暴力关机            |
  | virsh shutdown ~ | virsh destroy ~ |

### 删除域

- 创建`wx_undefine.py`

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  domName = 'vm31'
  pool = conn.lookupByName(domName)
  pool.undefine()
  conn.close()
  exit(0)
  ```

- 运行取消定义文件即可

### 编写`python`脚本并传参
- 创建python脚本

  ```python
  import sys
  print(sys.argv[1])
  ```

- 调用脚本

  ```bash
  python3 wx.python vm14
  ```

  ​





