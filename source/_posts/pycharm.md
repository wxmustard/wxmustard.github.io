---
title: pycharm
author: mustard
date: 2017-11-22 13:41:48
tags:
  - ubuntu
  - python
  - libvirt
categories:
  - 技术

---

## Pycharm使用

和IDEA的使用基本一致，导入第三方库的方法是：

- file --> deafault setting --> Project Interpreter --> 右侧上方选择python版本后 --> 点击+，选择需要添加的第三方库即可

## 遇到的问题

- 无法正常导入模块`from bs4 import BeautifulSoup `

  - `BeautifulSoup`在4.4.0以前的版本**不支持**`Python3.5`，所以需要升级`BeautifulSoup`的版本

  - 解决方法：`sudo pip3 install --upgrade beautifulsoup4`

    ​

    ​


## libvirt api

### libvirt python api

```bash
# 安装libvirt
# 首先ubuntu系统需要安装libvirt-dev
sudo apt install libvirt-dev
# ubuntu只有在安装了libvirt-dev后才能正常安装libvirt-python
sudo pip3 install libvirt-python
```

