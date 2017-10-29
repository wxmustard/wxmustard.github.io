---
title: ubuntu 网卡问题
author: mustard
tags:
  - ubuntu
  - network
categories:
  - 技术
abbrlink: 422f71fe
date: 2017-10-16 23:52:06
---

## ifconfig 只有lo

* ```bash
  cd /etc/network
  sudo vim interfaces
  # 在interfaces中添加以下语句
  auto eno1
  iface eno1 inet dhcp # dhcp是动态获取IP信息
  # 开启eno1 
  ifconfig eno1 up 
  # 查看IP信息
  ifconfig
  ```

## ifconfig中显示eno1，但是没有inet/地址/广播/掩码

* ```bash
  # 更新IP地址
  sudo dhclient eno1
  # 运行eno1
  sudo ifconfig eno1
  ```

  ​

## 禁用IPV6

* ```bash
  sudo vim /etc/sysctl.conf
  # 在sysctl.conf中加入下面语句
  net.ipv6.conf.all.disable_ipv6     = 1
  net.ipv6.conf.default.disable_ipv6 = 1
  net.ipv6.conf.lo.disable_ipv6      = 1
  ```

* 重启

  ```bash
  reboot
  ```

  ​