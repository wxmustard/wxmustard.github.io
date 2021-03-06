---
title: ubuntu虚拟机 + hadoop + spark + scala
author: mustard
tags:
  - ubuntu
  - 虚拟机
  - hadoop
  - spark
  - scala
categories:
  - 技术
abbrlink: 7ea34fb9
date: 2017-10-02 20:39:11
---

## ubuntu 虚拟机

* [ubuntu-16.04.3-server-amd64.iso下载](http://mirrors.shuosc.org/ubuntu-releases/xenial/ubuntu-16.04.3-desktop-amd64.iso)

  ```bash
  wget http://mirrors.shuosc.org/http://mirrors.shuosc.org/ubuntu-releases/xenial/ubuntu-16.04.3-server-amd64.iso
  ```

* 配置虚拟机网络

  ```bash
  sudo virsh net-edit default
  # 选择第三个选项
  ```

  编辑`mac`与`IP`对应的文件

  ```txt
  <network>
    <name>default</name>
    <uuid>b3f7605a-0d37-472c-ae7c-b51ab7b3f34c</uuid>
    <forward mode='nat'/>
    <bridge name='virbr0' stp='on' delay='0'/>
    <mac address='52:54:00:c8:24:95'/>
    <ip address='192.168.107.1' netmask='255.255.255.0'>
      <dhcp>
        <range start='192.168.107.2' end='192.168.107.254'/>
        <host mac='52:54:00:33:07:01' name='vm01' ip='192.168.107.101'/>
        <host mac='52:54:00:33:07:02' name='vm02' ip='192.168.107.102'/>
      </dhcp>
    </ip>
  </network>
  ```

* 创建虚拟镜像

  ```bash
  qemu-img create -f qcow2 ubuntu.qcow2 40G
  ```

* 创建配置文件

  ```bash
  touch ubuntu
  # 以下是配置文件的内容
  cat ubuntu
  <domain type='kvm'>
      <name>vm14</name>
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
  	 	<source file="/home/ubuntu/kvm/ubuntu-16.04.3-server-amd64.iso"/>  # 镜像文件路径，克隆的时候不需要镜像文件
          <target dev='hdc' bus='ide'/>
  	 	<readonly/>
  	 	<address type='drive' controller='0' bus='1' target='0' unit='0'/>
  	 </disk>
  	 <disk type='file' device='disk'>
  		<driver name='qemu' type='qcow2'/>
          <source file='/home/ubuntu/kvm/ubuntu.qcow2'/>  # 磁盘路径
          <target dev='hda' bus='ide'/>
       </disk>
       <interface type='bridge'>
  		<mac address="52:54:00:33:a4:14" />
        <source bridge='virbr0'/>
       </interface>

      <input type='mouse' bus='ps2'/>
       <graphics type='vnc' port='5914' autoport='no' listen = '0.0.0.0' keymap='en-us'/>
     </devices>
   </domain>
  ```

* `virsh` 命令

  ```bash
  # 定义虚拟机
  virsh define vm14.xml
  # 启动虚拟机
  virsh start vm20
  # 正常方式关闭虚拟机，但是会有一段关闭时间
  virsh shutdown vm14 
  # 强制方式关闭虚拟机
  virsh destory vm14 
  # 删除虚拟机，仅仅是从virsh列表里面将其删除，undefine命令不会删除镜像文件和xml文件，运行状态的虚拟机是不能删除的。
  virsh undefine vm14 
  # 查看是否删除成功
  virsh list -all 
  ```

* 出现的问题及解决方法

  * 更新源：一般情况下，更改 /etc/apt/sources.list 文件中 Ubuntu 默认的源地址 <http://archive.ubuntu.com/> 为 [http://mirrors.shuosc.org](http://mirrors.shuosc.org/) 即可。

    可以使用如下命令：
    ```bash
     curl https://git.shuosc.org/snippets/1/raw | sudo sh
    ```
    ```bash
    sudo sed -i 's/archive.ubuntu.com/mirrors.shuosc.org/g' /etc/apt/sources.list
    ```

  * 主机和虚拟机可以互相ping到，但是主机不能通过ssh连接到虚拟机上？

    ```bash
    ssh: connect to host 192.168.*. port 22: Connection refused
    ```

    解决方法

    ```bash
    sudo apt-get install openssh-server # 下载ssh
    sudo /etc/init.d/ssh start # 重启ssh
    ```

  * 在`ssh`连接虚拟机时出现下图情况

    ![error](https://vgy.me/aUe1Qi.png)

    解决方法

    ```bash
    cd /home/ubuntu/.ssh/
    rm known_hosts 
    ```

  ​

  ![download](https://vgy.me/lCq1yU.png)

## ssh免密码登录

* 生成密钥对

  ```bash
  ssh-keygen -t rsa
  ```

  执行上条命令将会生成两个文件，`id_rsa`,`id_rsa.pub`

* 将`vm14`的`id_rsa.pub`远程拷贝到`vm15`文件中

  ```bash
  scp id_rsa.pub ubuntu@vm15:/home/ubuntu
  ```

* 将`id_rsa.pub`中的内容追加到`authorized_keys`中

  ```bash
  cat id_rsa.pub >> .ssh/authorized_keys
  ```



## JDK安装

* [JDK下载](http://ftp.shu.aixinwu.org/linuxsoftware/jdk-8u101-linux-x64.tar.gz)

  ```bash
  wget http://ftp.shu.aixinwu.org/linuxsoftware/jdk-8u101-linux-x64.tar.gz  # 下载软件包
  tar xzf jdk-8u101-linux-x64.tar.gz  # 解压安装包
  ```

  ![选区_002](https://vgy.me/bWzqMr.png)

* 创建`jdk`的路径，为路径赋权

  ```bash
   sudo mkdir /usr/local/jdk  # 可能提示输入密码
   sudo chown ubuntu:ubuntu /usr/local/jdk/
  ```

* 将解压好的`jdk`文件转移到刚刚建立好的路径并修改`.bashrc`配置文件

  ```bash
   mv jdk1.8.0_101/* /usr/local/jdk/
   vim .bashrc 
   # 在.bshrc文件中添加以下变量
   export JAVA_HOME=/usr/local/jdk
   export PATH=$JAVA_HOME/bin:$PATH
   export  CLASSPATH=,:$JAVA_HOME/lib/dj.jar:$JAVA_HOME/lib/tools.jar
   # 保存退出后使刚刚的修改生效
   source .bashrc
  ```

* 验证`jdk`是否安装成功

  ```bash
  java -version
  javac
  ```

  ![选区_004](https://vgy.me/BqI8gV.png)

![选区_005](https://vgy.me/d3aCPt.png)





## Hadoop 安装

* [hadoop-2.7下载](http://mirrors.shuosc.org/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz)

  ```bash
  wget http://mirrors.shuosc.org/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz
  ```

* 创建存放`hadoop`的目录

  ```bash
  mkdir /usr/local/hadoop
  ```


* 解压下载好的软件包并移动至刚刚创建好的目录下

  ```bash
  tar xzf hadoop-2.7.4.tar.gz
  mv hadoop-2.7.4/* /usr/local/hadoop/
  ```

* 修改配置文件，路径为：`/usr/local/hadoop/etc/hadoop/`

  * ```xml
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
    ```

  * ```xml
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
    ```

  * ```xml
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
    ```

  * ```xml
    <!-- mapred-site.xml -->
    <configuration>
    	<property>
    		<name>mapreduce.framework.name</name>
    		<value>yarn</value>
    	</property>
    </configuration>
    ```

  * ```xml
    <!-- slaves -->
    vm15
    vm16
    ```

  * 以上五个文件在`Master`和`Slave`中是一致的，在`slaves`中要把默认生成的`localhost`删除，写上`Slave`的名称,也就是说三台虚拟机的`slave`文件是一致的。

  * ```xml
    <!-- hadoop-env.sh -->
    export JAVA_HOME=/usr/local/jdk
    ```

* 格式化`hdfs` ，目录是：`/usr/local/hadoop/bin`

  ```bash
  $ ./hdfs namenode -format 
  ```

* 启动`hadoop` ，目录是：`/usr/local/hadoop/sbin`

  注意：只需要在`Master`中开启`Hadoop`服务即可，`Slaves`中无需启动，否则会造成进程过多的情况

  ```bash
  $ ./start-all.sh
  ```

* 关闭防火墙

  ```bash
  #（ubuntu）
  # 查看防火墙状态
  sudo service ufw status 
  # 关闭防火墙
  sudo service ufw stop

  #（centos7）
  # 关闭防火墙
  sudo systemctl stop firewalld.service
  # 禁用防火墙
  sudo systemctl disable firewalld.service
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

* 遇到的问题及解决方法

  * 在启动`hadoop`时出现Error: JAVA_HOME is not set and could not be found.

    这是因为`JAVA_HOME`环境变量没有设置，需要在`/usr/local/hadoop/etc/hadoop/hadoop-env.sh`中设置`JAVA_HOME`，`export JAVA_HOME=$JAVA_HOME`这样设置仍然会报错，只有使用绝对路径才不会报错。如：`export JAVA_HOME=/usr/local/jdk`



## Scala 安装

* 由于时`ubuntu`　系统，所以下载deb格式的`scala`软件包

  ```bash
  wget http://www.scala-lang.org/files/archive/scala-2.10.4.deb
  sudo dpkg -i scala-2.10.4.deb 
  scala -version
  ```

  ![选区_007](https://vgy.me/UEqu0v.png)



## Spark安装

* [Spark-2.2.0下载](http://mirrors.shuosc.org/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz)

  ```bash
  wget http://mirrors.shuosc.org/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz
  ```

* 创建目录，解压软件包，移动软件至目录下

  ```bash
  mkdir /usr/local/spark
  tar xzf spark-2.2.0-bin-hadoop2.7.tgz
  mv spark-2.2.0/* /usr/local/spark/
  ```

* 修改配置文件，目录为`/usr/local/spark/conf`

  ```bash
  cp slaves.template slaves
  cp spark-env.sh.template spark-env.sh
  # slaves
  vm15
  vm16
  # spark-env.sh
  export SPARK_MASTER_HOST=vm14  #vm14是Master
  export SPARK_WORKER_MEMORY=2g
  export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
  export JAVA_HOME=/usr/local/jdk
  # 切记切记，一定要在此文件中写入JAVA_HOME,不然的话会出现下图中的错误
  ```

  ![选区_008](https://vgy.me/ZgqZrH.png)

* 启动`Spark`，目录为`/usr/local/spark/sbin`

  注意：只需要在`Master`中开启`Spark`服务即可，`Slaves`中无需启动，否则会造成进程过多的情况

  ```bash
  $ ./start-all.sh 
  ```

* 查看当前节点的任务

  ```bash
  # 在vm14下运行结果
  $ jps
  5489 NameNode
  6338 SecondaryNameNode
  11592 Master
  6489 ResourceManager
  11658 Jps
  #进程号  名称
  # 在vm15下运行结果
  $ jps
  14304 NodeManager
  13904 Worker
  14180 DataNode
  14475 Jps
  ```

* 关闭`Spark`,目录为`/usr/local/spark/sbin`

  ```bash
   $ ./stop-all.sh
  ```

  ![选区_010](https://vgy.me/sQYzHN.png)

* 查看端口信息

  ```bash
  # 查看监听端口信息和IP
  netstat -an 
  # 删除本机错误ip地址
  vim /etc/hosts
  ```

  ​







