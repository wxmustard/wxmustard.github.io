---
title: 远程连接MySQL数据库
author: mustard
tags:
  - ubuntu
  - MySQL
abbrlink: a104a74a
date: 2017-10-10 11:00:18
categories:
---

**服务器端**

* 安装`mysql-server`

  ```bash
  sudo apt install mysql-server
  ```

* 查看`mysql`的服务状态

  ```bash
  service mysql status
  service mysql start
  ```

* **MySQL**允许远程访问的设置

  ```bash
  cd /etc/mysql/mysql.conf.d/
  sudo vim mysqld.cnf
  # 将mysqld.cnf中的一句话注释
  # Instead of skip-networking the default is now to listen only on
  # localhost which is more compatible and is not less secure.
  # bind-address          = 127.0.0.1

  ```

* 登录mysql

  ```mysql
  # 登录mysql
  mysql -u root -p
  # 创建远程登录用户并授权
  grant all PRIVILEGES on 数据库.* to 授权用户(如root等)@'允许远程连接的IP地址' identified by '授权用户的密码';
  grant all privileges on wx.* to root@'%' identified by 'root'
  # 使刚刚的语句生效
  flush privilege;
  ```

  ​

* `mysql`因为安全的因素，默认设置无法远程连接，只能本地连接，此时需要用户自己去修改其中配置

  ```mysql
  # mysql数据库中存放着各种配置信息
  use mysql;
  # 查看用户的权限情况
  select host,user from user;
  # (%表示是所有的外部机器，如果指定某一台机，就将%改为相应的机器名；‘root’则是指要使用的用户名，里面的password需要自己修改成root的密码)
  mysql> Grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
  # 使修改生效
  flush privileges;
  ```


**客户端**

* 安装`mysql-client`

  ```bash
  sudo apt install mysql-client mycli
  ```

* 连接远程数据库

  ```bash
  mysql -h 111.186.110.177 -u root -p root
  show databases;
  ```
