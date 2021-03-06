---
title: Docker 学习笔记
author: mustard
tags:
  - docker
categories:
  - 技术
abbrlink: 920e92d1
date: 2017-10-02 20:44:42
---

* | Docker        | C++  |
  | ------------- | ---- |
  | 容器(Container) | 对象   |
  | 镜像(Images)    | 类机制  |

* 常用命令

  ```bash
  # 查看docker的具体详情
  sudo docker info
  # 查看所有容器，包括已经停止的
  sudo docker ps -a
  # 拉取镜像
  sudo docker pull <镜像名：tag>
  # 使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。
  # -d: 后台运行容器，并返回容器ID
  # --name="nginx-lb": 为容器指定一个名称；
  sudo docker run --name mynginx -d nginx:latest
  # 启动已被停止的容器myrunoob
  sudo docker start myrunoob
  # 停止运行中的容器myrunoob
  sudo docker stop myrunoob
  # 重启容器myrunoob
  sudo docker restart myrunoob
  # 强制移除容器db01
  sudo docker rm -f db01
  ```

* `linux`服务常见操作(Centos7&&ubuntu)

  ```bash
  # 启动name服务
  sudo systemctl start name-service
  # 重启name服务
  sudo systemctl restart name-service
  # 停止name服务
  sudo systemctl stop name-sewget https://dl.minio.io/client/mc/release/linux-amd64/mcrvice
  # 重载name服务
  sudo systemctl reload name-service
  # 检查服务状态
  sudo systemctl status name-service
  # 安装测速仪
  sudo pip3 install speedometer
  ```

* 编写shell脚本的思路

  * 检查系统中是否已经包含将要下载的文件
  * 检查系统版本
  * 为系统添加公钥，使系统信任(GPG  --keyserver)
  * 安装执行

* `docker-compose`使用

  在使用`docker-compose`前应确保机器已安装`pyhon3`和`pip3`

  ```bash
  sudo apt install python3 python3-pip
  #配置pip镜像源
  mkdir ~/.pip
  tee ~/.pip/pip.conf << EOF
  [global]
  index-url=https://pypi.shuosc.org/simple
  [list]
  format=columns
  EOF
  sudo pip3 install -U pip
  ```

  ```bash
  # docker-compose安装
  sudo pip3 install docker-compose 
  sudo apt autoremove # 自动清除无用的东西
  # 查看compose的版本信息
  docker-compose -version
  ```

  * 使用`Dockerfile`定义应用环境

  * 在`compose.yaml`中定义应用服务

  * 启动应用

    ```bash
    sudo docker-compose -f ~~~.yaml up -d # 启动
    sudo docker-compose -f ~~~.yaml down # 关闭
    ```

    ​

* `dockerfile`与`docker-compose`

  * `dockerfile`类似于一个脚本，记录一个镜像的创建过程，只需要执行`docker build `命令即可创建镜像
  * `docker-compose`类似于一个集合，很多时候为了实现一个功能，只有一个容器是不够的，例如后端存储需要数据库，前端需要`nginx`，如果把这些都放到一个容器中就没有办法复用，而是应该把数据库和`nginx`都创建成镜像，然后将这些镜像集合起来，共同服务一个项目，在`docker-compose.yml`中定义镜像，镜像的配置，挂载的卷等等，通过`docker-compose up `启动服务，`docker-compose stop/down`停止服务，

