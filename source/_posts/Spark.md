---
title: Spark安装
author: mustard
tags:
  - spark
categories:
  - 技术
abbrlink: f688dfae
date: 2017-10-02 20:40:52
---

##  Spark

* [Spark-2.2.0下载](http://mirrors.shuosc.org/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz)

  ```bash
  wget http://mirrors.shuosc.org/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz
  ```

* 解压

  ```bash
  tar xzf spark-2.2.0-bin-hadoop2.7.tgz
  ```

* 修改配置文件，目录为`/usr/local/spark/conf`

  ```bash
  cp slaves.template slaves
  cp spark-env.sh.template spark-env.sh
  ```

  ```sh
  # slaves
  vm15
  vm16
  # spark-env.sh
  export SPARK_MASTER_HOST=vm14
  export SPARK_WORKER_MEMORY=2g
  export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
  ```

* 启动`Spark`，目录为`/usr/local/spark/sbin`

  ```bash
  $ ./start-all.sh
  ```

* 查看当前节点的任务

  ```bash
  # 在vm14下运行结果
  $ jps
  5362 NameNode
  5715 ResourceManager
  6004 Master
  7093 Jps
  5558 SecondaryNameNode
  #进程号  名称
  # 在vm15下运行结果
  $ jps
  3760 Jps
  3042 NodeManager
  2940 DataNode
  3660 Workerscala
  ```

* 关闭`Spark`,目录为`/usr/local/spark/sbin`

  ```bash
   $ ./stop-all.sh
  ```

## Scala

* [scala-2.12.3下载](https://downloads.lightbend.com/scala/2.12.3/scala-2.12.3.rpm)

  ```bash
  wget https://downloads.lightbend.com/scala/2.12.3/scala-2.12.3.rpm
  ```


* 安装

  ```bash
  rpm -ivh scala-2.12.3.rpm
  ```

* 验证是否安装成功

  ```bash
  scala -version
  ```

  ​

***

##端口转发

* 安装端口映射工具(在lab1)

  ```bash
  sudo apt install rinetd
  # 修改配置文件 /etc/rinetd.conf
  cat /etc/rinetd.conf
  # bindadress    bindport  connectaddress  connectport
  0.0.0.0  50070  192.168.110.114 50070
  0.0.0.0  8088   192.168.110.114 8088  
  # 重启服务
  sudo service rinetd restart
  # 查看状态
  sudo service rinetd status
  # 显示网络连接、路由表和网络接口信息
  netstat -an
  # 扫描端口
  nmap -v -A 192.168.110.114
  # 关闭防火墙
  sudo systemctl stop firewalld.service
  # 查看防火墙状态
  sudo systemctl status firewalld.service
  ```

