# Nagios

## 安装必备软件

```bash
# 安装apache2 web服务
sudo apt install apache2
# GCC编译器与开发库
sudo apt install build-essential
# GD库与开发库
sudo apt install libgd2-dev
```

http://nagios-cn.sourceforge.net/nagios-cn/beginning.html#quickstart-ubuntu

** 这种方式有点问题，使用下面的方式进行安装



## 使用`docker`方式配置`nagios`

- ```bash
  # pull nagios的镜像
  docker pull quantumobject/docker-nagios
  # 创建容器
  docker run -d -p 25 -p 80 quantumobject/docker-nagios
  # 进去容器
  docker exec -it container_id  /bin/bash
  # 更改web界面登录的密码
  # 初始用户名为nagiosadmin，密码为admin
  htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
  # 访问web界面
  # 查看容器映射的端口
  sudo docker ps -a
  links http://localhost:端口/nagios/
  ```

- 由于实验是在`KVM`虚拟机vm15上实现的，在本地访问需要进行端口转发，具体操作如下

  ```bash
  # 在服务器lab1上进行端口转发
  sudo apt install rinetd
  sudo vim /etc/rinetd.conf
  # 添加需要转发的端口
  0.0.0.0  32768  192.168.110.115  32768
  # 将所有发往本机的32768端口服务都转发至vm15的32768端口
  ```

- 至此，在本地可以通过访问`http://111.186.106.99:32768/nagios/`来查看`nagios`

  ![](https://vgy.me/3Sxg9m.png)