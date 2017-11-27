---
title: libvirt
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
### 使用libvirt api创建虚拟机

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

- 创建`create.py`文件

  ```python
  import libvirt
  conn=libvirt.open('qemu:///system')
  xmldesc=open('/home/ubuntu/kvm/wx/vm31','r').read()
  dom = conn.createXML(xmldesc, 0);     
  print"Domian:id %d running %s"%(dom.ID(),dom.OSType())
  ```

- 运行`create.py`

  ```bash
  python2.7 create.py
  ```

### 