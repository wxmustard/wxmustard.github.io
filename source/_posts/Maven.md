---
title: Maven安装
author: mustard
tags:
  - centos
  - null
categories:
  - 技术
abbrlink: b387cbb1
date: 2017-10-03 12:57:26
---

* java 安装

  ```bash
  yum -y install java-1.8.0-openjdk-devel
  ```

* Maven 安装

  ```shell
  cd /home
  # 下载软件包
  wget http://mirrors.shuosc.org/apache/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz
  # 解压软件包
  tar xzvf apache-maven-3.5.0-bin.tar.gz
  mkdir /usr/local/apache-maven
  mv apache-maven-3.5.0  /usr/local/apache-maven
  ```

* 配置环境变量

  ```bash
  sudo vim /etc/profile
  MAVEN_HOME=/usr/local/apache-maven
  export MAVEN_HOME
  export PATH=${PATH}:${MAVEN_HOME}/bin
  ```

* 测试是否安装成功

  ```bash
  source /etc/profile
  mvn -version
  ```

  ​