---
title: mysql-docker
author: mustard
date: 2018-01-16 19:33:31
tags:
 - mysql
 - docker 
categories:
 - 技术

---

## 在本地使用`mysql`容器

- 创建`mysql`容器

```bash
# 获取mysql镜像
docker pull mysql
# 创建一个名为mysql1的容器，并为root用户设置密码为root
sudo docker run --name mysql1 -e MYSQL_ROOT_PASSWORD=root mysql:latest
# 查看刚刚创建的容器
sudo docker ps -a
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS                      PORTS               NAMES
63f66b53c702        mysql:latest               "docker-entrypoint.s…"   16 minutes ago      Up 16 minutes               3306/tcp            mysql1
# 进入容器内部
➜  ~ sudo docker exec -ti 63f66b53c702 /bin/bash      
root@63f66b53c702:/# mysql -u root -p
Enter password: 
# 查看现有的数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

```

- 在本地使用`mysql`容器

```bash
# 首先查看容器的ip地址
# 使用docker ps -a 查看容器ID，根据ID查看ip地址
sudo docker inspect 63f66b53c702 | grep IPAddress
# 得到容器地址后就可以连接使用mysql容器了，远程连接需要使用-h参数
mysql -h 172.17.0.2 -u root -p  
# 下面是个测试
# 本地创建数据库lre
➜  ~ mysql -h 172.17.0.2 -u root -p              
Enter password: 
mysql> create database lre;
Query OK, 1 row affected (0.00 sec)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lre                |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit
# 进入容器查看数据库
➜  ~ sudo docker exec -ti 63f66b53c702 /bin/bash
root@63f66b53c702:/# mysql -u root -p root
Enter password: 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| lre                |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> 
# 容器中的数据库同样发生了更改，ok！
```

## 端口转发

- 如果是在服务器上创建的容器的话，需要使用`rinetd`进行端口转发

```bash
sudo apt install rinetd
sudo vim /etc/rineed.conf
# 将以下语句写入conf文件
0.0.0.0 3269 192.168.110.115 3269
# 将所有的访问3269的请求都转发至192.168.110.115的3269上
#启动转发
rinetd -c /etc/rinetd.conf
```

