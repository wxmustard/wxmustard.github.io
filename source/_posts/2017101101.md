---
title: tomcat9配置
author: mustard
tags:
  - ubuntu
abbrlink: 4b40aed8
date: 2017-10-11 11:09:02
categories:
---

* 下载**tomcat9.0**

  ```bash
  wget http://apache.mirrors.ionfish.org/tomcat/tomcat-9/v9.0.1/bin/apache-tomcat-9.0.1-windows-x64.zip
  unzip apache-tomcat-9.0.1-windows-x64.zip
  ```

* 配置**tomcat9.0**

  ```bash
  sudo mkdir /usr/lib/tomcat9
  sudo mv apache-tomcat-9.0.1 /usr/lib/tomcat9
  sudo chmod -R 777 tomcat9/*
  sudo vim tomcat9/bin/startup.sh
  # 在最后一行上方加入下述命令
  TOMCAT_HOME=/usr/lib/tomcat9
  sudo vim tomcat9/bin/shutdown.sh
  # 在最后一行上方加入下述命令
  TOMCAT_HOME=/usr/lib/tomcat9
  ```

* 启动**tomcat**

  ```bash
  cd /usr/lib/tomcat9/bin
  ./startip.sh
  ```

* 在浏览器上输入http://localhost:8080即可验证是否安装成功

  ![](https://vgy.me/zxM5b5.png)

