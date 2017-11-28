---
title: Hadoop总结
author: mustard
tags:
  - hadoop
categories:
  - 技术
abbrlink: 1b8734e4
date: 2017-10-02 20:38:02
---

# Hadoop

* [hadoop-2.7下载](http://mirrors.shuosc.org/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz)

  ```bash
  wget http://mirrors.shuosc.org/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz
  ```

* 创建存放`hadoop`的目录

  ```bash
  mkdir /usr/local/hadoop
  ```

* 解压 

  ```bash
  tar xzf hadoop-2.7.4.tar.gz
  mv hadoop-2.7.4/* /usr/local/hadoop/ # 将解压文件移动到刚刚创建的目录下
  ```

* 修改配置文件 ，目录是：`/usr/local/hadoop/etc/hadoop/`

  ```xml
  <!-- core-site.xml -->
  <configuration>
  	<property>
  		<name>fs.defaultFS</name>
  		<value>hdfs://vm14:9000</value>
  	</property>
  	<property>
  		<name>hadoop.tmp.dir</name>
  		<value>file:/usr/local/hadoop/tmp</value>
  	</property>
  </configuration>
  <!-- 记得mkdir /tmp文件夹 -->
  <!-- hdfs-site.xml -->
  <configuration>
  	<property>
  		<name>dfs.namenode.secondary.http-address</name>
  		<value>vm14:50090</value>
  	</property>
  	<property>
          <name>dfs.namenode.name.dir</name>
          <value>file:/usr/local/hadoop/tmp/dfs/name</value>
      </property>
  	<property>
          <name>dfs.datanode.data.dir</name>
          <value>file:/usr/local/hadoop/tmp/dfs/data</value>
      </property>
  	<property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>

  </configuration>

  <!-- yarn-site.xml  -->
  <configuration>
  	<property>
          <name>yarn.resourcemanager.hostname</name>
          <value>vm14</value>
      </property>
  	<property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
  </configuration>

  <!-- mapred-site.xml -->
  <configuration>
  	<property>
  		<name>mapreduce.framework.name</name>
  		<value>yarn</value>
  	</property>
  </configuration>

  <!-- slaves -->
  vm15
  vm16
  ```

* 格式化`hdfs` ，目录是：`/usr/local/hadoop/bin`

  ```bash
  $ ./hdfs namenode -format 
  ```


* 启动`hadoop` ，目录是：`/usr/local/hadoop/sbin`

  ```bash
  $ ./start-all.sh
  ```

* 查看当前节点的任务

  ```bash
  $ jps
  5362 NameNode
  5715 ResourceManager
  5558 SecondaryNameNode
  6956 Jps
  #正常启动应有以上四个内容
  ```

* 关闭`hadoop`,目录是：`/usr/local/hadoop/sbin`

  ```bash
  $ ./stop-all.sh
  ```

  