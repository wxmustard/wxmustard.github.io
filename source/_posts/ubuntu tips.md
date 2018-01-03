---
title: 2017/9/15 学习笔记
author: mustard
tags:
  - linux
  - ubuntu
categories:
  - 技术
abbrlink: 6f961e32
date: 2017-10-02 20:36:50
---

### linux的挂载点

* /    根目录，存放开机启动的主要代码 （ext4格式）
* /boot  启动分区，包含了操作系统的内核和再启动系统过程中所需要用到的文件（ext4格式）
* /home 用户目录，用户数据的存储目录   （ext4格式）
* swap    进行数据交换，缓冲区，一般是与内存等同大小 （swap格式）


### 安装ubuntu

- 选择语言环境 English
- 填写hosts name , username, passwd
- 检查network
- 选择时区，得到clock
- 选择硬盘，安装即可


### ubuntu使用tips
- 自定义`ubuntu ssh` 的欢迎信息

  - ```bash
    cd /etc/update-motd.d
    00-header     51-cloudguest         91-release-upgrade  98-fsck-at-reboot
    10-help-text  90-updates-available  97-overlayroot      98-reboot-required
    # 这个目录下的文件会在登陆成功后按照序号的顺序自动执行
    ```

  - 如：

    ```bash
    vim 00-header
    # 在文件的末尾添加以下字符
    cat << "EOT"
      /\_/\
    =( °w° )=
      )   (  //
     (__ __)//
    EOT
    # 保存并重启服务器
    ```

  - ![](https://vgy.me/Z0t2VL.png)