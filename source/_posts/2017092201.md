---
title: ssh 免密码登录&&java安装
author: mustard
tags:
  - ssh
  - java
categories:
  - 技术
abbrlink: '520142e0'
date: 2017-10-02 20:31:10
---

## ssh 免密码登录

* 生成`key`

  ```bash
  ssh-keygen -t rsa
  ```

  执行上条命令将会生成两个文件，`id_rsa`,`id_rsa.pub`

* 将`id_rsa.pub`中的内容追加到`authorized_keys`中

  ```bash
  cat .ssh/id_rsa.pub >> .ssh/authorized_keys
  ```

* 使配置文件生效  ` /etc/ssh/sshd_config`文件中的一行代码(`#authorized~~~~`)取消注释即可

* 修改权限

  ```bash
  chmod 600 .ssh
  chmod 600 .ssh/authorized_keys
  ```


## 为虚拟机安装jdk

### 安装`Hadoop`和`spark`的步骤与`jdk`类似

* 下载`jdk`

  ```bash
  wget http://ftp.shu.aixinwu.org/linuxsoftware/jdk-8u101-linux-x64.tar.gz
  ```

* 解压`jdk`压缩包

  ```bash
  tar xzf 压缩包名称
  ```

* 创建安装目录

  ```bash
  sudo mkdir /usr/local/jdk
  ```

* 赋权命令

  ```bash
  sudo chown centos:centos .ssh
  ```

* 将`jdk`文件移动至`/usr/local/jdk`路径下

  ```bash
  mv jdk/* /usr/local/jdk
  ```

* 配置`java`环境变量

  ```bash
  vim .bashrc
  export JAVA_HOME=/usr/local/jdk
  export PATH=$JAVA_HOME/bin:$PATH
  export CLASSPATH=,:$JAVA_HOME/lib/dj.jar:$JAVA_HOME/lib/tools.jar
  ```

* 使`.bashrc`生效

  ```bash
  source .bashrc
  ```

* 检测安装结果

  ```bash
  javac
  java -version	
  ```

