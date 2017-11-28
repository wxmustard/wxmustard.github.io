---
title: nginx 安装实验
author: mustard
tags:
  - ubuntu
categories:
  - 技术
abbrlink: 3f4e8a3a
date: 2017-10-11 22:36:46
---

# nginx 安装实验(ubuntu)

* 安装依赖包

  ```shell
  sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential
  ```

* 安装编译源码

  ```shell
  sudo apt install wget vim links
  # wget:下载源码,vim:编辑文件,links:测试访问
  wget http://nginx.org/download/nginx-1.12.1.tar.gz
  # 下载安装包
  tar xzf nginx-1.12.1.tar.gz 
  # 解压安装包
  sudo useradd www -s /sbin/nologin
  # 添加用户www,并指定用户无法登录
  cd nginx-1.12.1/
  # 进入文件夹
  ./configure --user=www --group=www --prefix=/usr/local/nginx/ --with-http_stub_status_module --with-http_ssl_module
  # 修改配置文件
  make && make install
  # 编译程序
  ```

* 测试`nginx`是否安装成功

  -**方法一: **

  ```shell
  # 查看nginx版本信息
  /usr/local/nginx/sbin/nginx -V
  ```

  ![](https://vgy.me/3Im1GG.png)

  **方法二: **

  ```shell
  # 执行启动命令
  /usr/local/nginx/sbin/nginx -c usr/local/nginx/conf/nginx.conf
  检测80端口
  netstat -an | grep 80
  ```

  ![](https://vgy.me/OZePyK.png)

  **方法三:**

  ```shell
  links http://vm15
  ```

  ![](https://vgy.me/8JCybT.png)

* 自定义`nginx`的`service`服务

  ```shell
  cd /etc/init.d
  vim nginx 
  # 创建并修改nginx的services服务文件
  ```

  `nginx`文件内容如下

  ```xml
  <!-- nginx -->
  #! /bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  NAME=nginx
  NGINX_BIN=/usr/local/nginx/sbin/$NAME
  CONFIGFILE=/usr/local/nginx/conf/$NAME.conf
  PIDFILE=/usr/local/nginx/logs/$NAME.pid
  case "$1" in
      start)
          echo -n "Starting $NAME... "
          if netstat -tnpl | grep -q nginx;then
              echo "$NAME (pid `pidof $NAME`) already running."
              exit 1
          fi
          $NGINX_BIN -c $CONFIGFILE
          if [ "$?" != 0 ] ; then
              echo " failed"
              exit 1
          else
              echo " done"
          fi
          ;;
      stop)
          echo -n "Stoping $NAME... "
  if ! netstat -tnpl | grep -q nginx; then
              echo "$NAME is not running."
              exit 1
          fi
          $NGINX_BIN -s stop
          if [ "$?" != 0 ] ; then
              echo " failed. Use force-quit"
              exit 1
          else
              echo " done"
          fi
          ;;
      status)
          if netstat -tnpl | grep -q nginx; then
              PID=`pidof nginx`
              echo "$NAME (pid $PID) is running..."
          else
              echo "$NAME is stopped"
              exit 0
          fi
          ;;
      force-quit)
          echo -n "Terminating $NAME... "
          if ! netstat -tnpl | grep -q nginx; then
              echo "$NAME is not running."
              exit 1
          fi
          kill `pidof $NAME`
          if [ "$?" != 0 ] ; then
              echo " failed"
              exit 1
          else
              echo " done"
          fi
          ;;
      restart)
          $0 stop
          sleep 1
          $0 start
          ;;
      reload)
          echo -n "Reload service $NAME... "
          if netstat -tnpl | grep -q nginx; then
              $NGINX_BIN -s reload
  echo " done"
          else
              echo "$NAME is not running, can't reload."
              exit 1
          fi
          ;;
      configtest)
          echo -n "Test $NAME configure files... "
          $NGINX_BIN -t
          ;;
      *)
          echo "Usage: $0 {start|stop|force-quit|restart|reload|status|configtest}"
          exit 1
          ;;
  esac
  ```

  ```shell
  # 修改脚本文件nginx的权限
  sudo chmod 755 nginx
  # 修改开机自启动文件
  sudo vim /etc/rc.local
  # 将下面的代码添加到rc.local文件的exit 0 之前
  sudo service nginx start
  ```

  ​