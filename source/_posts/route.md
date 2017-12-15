---
title: 路由转发
author: mustard
date: 2017-12-15 16:52:32
tags:
 - linux
 - route
categories:
 - 技术
---

## 实现两台宿主机内的虚拟机间通信

### 实验环境

- lab3主机(虚拟机vm01的ip为192.168.103.101)

  ```bash
  ➜  ~ route -n 
  Kernel IP routing table
  Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
  0.0.0.0         111.186.106.254 0.0.0.0         UG    0      0        0 eno1
  10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 rename3
  111.186.106.0   0.0.0.0         255.255.255.0   U     0      0        0 eno1
  172.22.0.0      0.0.0.0         255.255.0.0     U     0      0        0 zt0
  172.28.0.0      0.0.0.0         255.255.0.0     U     0      0        0 zt1
  192.168.103.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
  ```

- lab7主机(虚拟机vm01的ip为192.168.107.101)

  ```bash
  ➜  kvm route -n
  Kernel IP routing table
  Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
  0.0.0.0         111.186.113.254 0.0.0.0         UG    0      0        0 eno1
  10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eno2
  111.186.112.0   0.0.0.0         255.255.254.0   U     0      0        0 eno1
  172.22.0.0      0.0.0.0         255.255.0.0     U     0      0        0 zt0
  192.168.107.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
  ```

### 设置路由

- 设置永久路由

  ```bash
  # vim /etc/sysctl.conf 
  net.ipv4.ip_forward=1
  ```


- lab3上的虚拟机（192.168.103.101）要访问lab7上的虚拟机（192.168.107.101）

  ```bash
  sudo route add -net 192.168.107.0/24 gw 10.0.0.7 dev rename3
  ```

- lab7上的虚拟机（192.168.107.101）要访问lab3上的虚拟机（192.168.103.101）

  ```bash
  sudo route add -net 192.168.103.0/24 gw 10.0.0.3 dev eno2
  ```

### 试验结果

- 登录lab3的虚拟机

  ```bash
  ubuntu@vm01:~$ ping -R 192.168.107.101
  PING 192.168.107.101 (192.168.107.101) 56(124) bytes of data.
  64 bytes from 192.168.107.101: icmp_seq=1 ttl=62 time=0.649 ms
  RR: 	192.168.103.101
  	10.0.0.3
  	192.168.107.1
  	192.168.107.101
  	192.168.107.101
  	10.0.0.7
  	192.168.103.1
  	192.168.103.101

  64 bytes from 192.168.107.101: icmp_seq=2 ttl=62 time=0.597 ms	(same route)
  64 bytes from 192.168.107.101: icmp_seq=3 ttl=62 time=0.564 ms	(same route)
  ^C
  --- 192.168.107.101 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss, time 1998ms
  rtt min/avg/max/mdev = 0.564/0.603/0.649/0.040 ms
  ```

- 登录lab7的虚拟机

  ```bash
  ubuntu@vm01:~$ ping -R 192.168.103.101
  PING 192.168.103.101 (192.168.103.101) 56(124) bytes of data.
  64 bytes from 192.168.103.101: icmp_seq=1 ttl=62 time=0.815 ms
  RR: 	192.168.107.101
  	10.0.0.7
  	192.168.103.1
  	192.168.103.101
  	192.168.103.101
  	10.0.0.3
  	192.168.107.1
  	192.168.107.101

  64 bytes from 192.168.103.101: icmp_seq=2 ttl=62 time=0.719 ms	(same route)
  64 bytes from 192.168.103.101: icmp_seq=3 ttl=62 time=0.658 ms	(same route)
  ^C
  --- 192.168.103.101 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss, time 2002ms
  rtt min/avg/max/mdev = 0.658/0.730/0.815/0.071 ms
  ```



成功！！！去吃饭～