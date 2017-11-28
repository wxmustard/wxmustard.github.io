---
title: NFS+minio挂载+WebDav
author: mustard
tags:
  - nfs
  - minio
  - webdav
categories:
  - 技术
abbrlink: 9616d5bb
date: 2017-10-02 20:42:14
---

## nfs（ubuntu）

* `nfs(Network File System)`网络文件系统，`RPC (Remote Procedure Call)`远端程序呼叫，`RPC`最主要的功能是在指定每个`NFS`功能所对应的端口号，并且汇报给用户端，让用户端可以连接到正确的端口上。(为什么`RPC`会知道每个`NFS`功能所对应的端口号呢，这是因为当服务器在开启`NFS`会随机的取用数个端口，并且会主动的向`RPC`注册，这样` RPC`就 可以知道每个端口对应的服务，然后 `RPC` 固定使用 `port 111` 來监听用户端的需求并告诉用户端正确的端口号）

* 关闭防火墙

  ```bash
  sudo service ufw status
  sudo service ufw stop
  ```

* 下载相应的软件包

  ```bash
  sudo apt-get install rpcbind
  sudo apt-get install nfs-kernel-server  # 仅在服务器端安装
  sudo apt-get install nfs-common 
  ```

* 创建一个共享文件的目录（仅*服务器端*操作）

  ```bash
  sudo mkdir /home/nfsdir
  ```

* 修改配置文件`/etc/exports`,在文章末尾加上如下语句 （仅*服务器端*操作）

  ```xml
  /home/nfsdir 192.168.110.0/24(rw,sync,no_root_squash,no_subtree_check)
  ```

* 重新启动服务

  ```bash
  sudo service rpcbind restart
  sudo service nfs-kernel-server restart
  sudo service nfs-kernel-server status
  ```

* 创建挂载点 （仅*用户端*操作）

  ```bash
  # 创建挂载点
  mkdir wangxin 
  # 实现挂载
  sudo mount -t nfs 192.168.110.114:/home/nfsdir /home/ubuntu/wangxin 
  # 为目录修改权限
  sudo chown ubuntu:ubuntu wangxin
  ```

  经过以上操作即可在`vm15，vm16`中查看并修改`vm4`共享的文件

  ![](https://vgy.me/CyVD2I.png)

  ![](https://vgy.me/AUurtA.png)

* 使用`chown`修改权限实际上是修改`UID`和`GID`，修改为同一用户可以在多机之间同时无障碍使用`nfs`共享文件夹。例如在`vm15`中修改了挂载点的权限后，`vm16`的权限也随之改变了，无需再次修改权限。



## minio

### 服务器端

* [minio下载](https://dl.minio.io/server/minio/release/linux-amd64/minio)

  ```bash
  wget https://dl.minio.io/server/minio/release/linux-amd64/minio
  ```

* 赋予可执行权限

  ```bash
  chmod +x minio
  ```

* 创建数据目录

  ```bash
  mkdir minio1
  ```

* 启动服务，这条命令会登录页面所需要的具体信息，然后在网页端登录

  ```bash
  # 新建一个session，将minio放入其中执行
  tmux new -s minio
  # 列出当前的session
  tmux ls
  # 进入其中一个session
  tmux attach -t minio
  # 在minio中执行启动服务操作
  ./minio server minio1
  # 退出session
  ctrl + B 再按 D
  ```

  ![](https://vgy.me/YLLcGg.png)

* | AccessKey     | K5YEC0KYPFBAAPJCOVY3                     |
  | ------------- | ---------------------------------------- |
  | **SecretKey** | qM98Y4uT/kXEACaRKU+8rJvgY9F6/Vy6pxt7wB/W |

* 遇到的问题及解决方案

  * 在网页上无法打开http://192.168.110.114:9000

    解决方法：退出虚拟机，回到实体机上，查看端口转发的配置文件，将9000端口添加到转发端口中，重启服务

    ```bash
    cat /etc/rinetd.conf 
    vim /etc/rinetd.conf
    # 将端口加入到配置文件中
    0.0.0.0  9000   192.168.110.114 9000
    sudo service rinetd restart
    ```

    打开网页端输入`key`，得到下图

    ![](https://vgy.me/i7VENX.png)

    ​			

### 客户端

* [minio客户端下载](https://dl.minio.io/client/mc/release/linux-amd64/mc)

  ```bash
  wget https://dl.minio.io/client/mc/release/linux-amd64/mc
  ```

* 赋予执行权限

  ```bash
  chmod +x mc
  ```

* ```bash
  $ mc config host add myminio http://192.168.110.114:9000 K5YEC0KYPFBAAPJCOVY3 qM98Y4uT/kXEACaRKU+8rJvgY9F6/Vy6pxt7wB/W
  ```

* 安装必要的环境

  ```bash
  sudo apt-get install automake autotools-dev fuse g++ git libcurl4-gnutls-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
  ```

* 下载编译安装`s3fs`

  ```bash
  git clone https://github.com/s3fs-fuse/s3fs-fuse.git
  cd s3fs-fuse
  ./autogen.sh
  ./configure
  htop # 监控虚拟机，可以查看到虚拟机的配置
  make -j4  # 虚拟机是双核的，四线程
  sudo make install
  cd ~
  ```

* 将服务器端生成的秘钥加入`passwd`（如果没有自行创建）

  ```bash
  vim passwd
  K5YEC0KYPFBAAPJCOVY3:qM98Y4uT/kXEACaRKU+8rJvgY9F6/Vy6pxt7wB/W
  # 修改passwd权限
  chmod 600 passwd 
  ```


* 创建挂载点

  ```bash
  mkdir s3
  ```

* 创建挂载目录

  ```bash
  s3fs wangxin s3 -o passwd_file=passwd,use_path_request_style,url=http://vm14:9000,rw,uid=1000,gid=1000
  ```

* 查看挂载目录

  ```bash
  df -h |grep /home/ubuntu/s3
  ```

  ![](https://vgy.me/fGslNv.png)

此时，可以在`vm15`上查看到服务器端上传的文件，并对文件进行修改，如果文件是只读的，需要进行授权处理，`chmod -R 755 s3/*`，在`vm15`上上传的文件同样可以再服务器端进行操作。



## WebDav

### 服务器端

* 安装`Docker`

  ```bash
  curl https://git.shuosc.org/snippets/5/raw | sudo sh
  sudo usermod -aG docker $(whoami)
  # 配置加速器
  curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://e85bc34e.m.daocloud.io
  # 重启docker服务
  sudo service docker restart
  ```

* 使用`docker-compose`进行`pull`镜像

  ```bash
  # 安装docker-compose,安装前确定已经安装pip3和python3
  sudo pip3 install docker-compose
  ```

* 修改`docker-compose.yml`文件(格式很重要，不能使用`tab`只能使用空格)

  ```yml
  version: '2'

  services:
    db:
      image: mariadb
      restart: always
      volumes:
        - /home/ubuntu/mariadb:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=centos
        - MYSQL_PASSWORD=centos
        - MYSQL_DATABASE=nextcloud
        - MYSQL_USER=nextcloud

    app:
      image: nextcloud
      ports:
        - 8080:80
      links:
        - db
      volumes:
        - /home/ubuntu/nextcloud:/var/www/html/data
  ```

* 创建`docker-compose.yml`中使用的目录

  ```bash
  mkdir mariadb
  mkdir nextcloud
  ```

* 启动`docker-compose`

  ```bash
  sudo docker-compose up -d
  sudo docker run -d nextcloud -v nextcloud:/var/www/html
  ```

* 打开浏览器访问

* 网页端口上传的文件在`vm14`中的以下路径中可以查看

  ```bash
  sudo ls nextcloud/mustard/files
  ```

  ![](https://vgy.me/Ha8xqc.png)

### 客户端

* 安装`davfs2`

  ```bash
  sudo apt install davfs2
  ```

* 创建挂载点

  ```bash
  mkdir davfs
  ```

* 创建挂载目录

  ```bash
  mount -t davfs http://lab1:8080/remote.php/webdav davfs  
  ```

* 赋权

  ```bash
  sudo chown -R wangxin:wangxin davfs 
  # 查看文件
  ls davfs
  ```

  ​
