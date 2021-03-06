---
title: Jupyter Notebook笔记
author: mustard
tags:
  - centos
abbrlink: f2126ca
date: 2017-10-09 15:26:26
categories:
---

> **Jupyter Notebook** 是一个开源的 Web 应用程序，可以用来创建和共享包含动态代码、方程式、可视化及解释性文本的文档。其应用于包括：数据整理与转换，数值模拟，统计建模，机器学习等等。

* 安装`Jupyter Notebook`

  ```bash
  python --version
  # 安装pip，用来管理和安装python的工具
  yum -y install python-pip
  pip install --upgrade pip
  # 安装相关依赖
  # groupinstall批量安装软件
  yum -y groupinstall "Development Tools"
  # 准备Python的环境
  yum -y install python-devel
  # 配置虚拟环境
  # 安装virtualenv，用来创建一套隔离的Python的运行环境
  pip install virtualenv
  # 使用virtualenv创建一个环境，并直接激活进入该环境
  virtualenv venv 
  source venv/bin/activate
  # 安装jupyter
  pip install jupyter
  ```

  sha1:7cfdbedde92f:908d917657438f8cd6e56fc93c769830f0c7a510

* 配置`Jupyter Notebook`

  ```bash
  # 首先创建目录
  mkdir /data/jupyter # 为Jupyter相关文件创建目录
  cd /data/jupyter
  # 创建一个Jupyter运行的根目录
  mkdir /data/jupyter/root
  # 生成密文，执行下述命令需要输入密码，并记下返回的sha1:...
  python -c "import IPython;print IPython.lib.passwd()"
  # 使用 --generate-config 来参数生成默认配置文件：
  jupyter notebook --generate-config --allow-root
  # 配置文件地址/root/.jupyter/
  cd /root/.jupyter/
  vim jupyter_notebook_config.py
  # 在配置文件的末尾加上下述配置
  c.NotebookApp.ip = '*'
  c.NotebookApp.allow_root = True
  c.NotebookApp.open_browser = False
  c.NotebookApp.port = 8888
  c.NotebookApp.password = u'刚才生成的密文(sha:...)'
  c.ContentsManager.root_dir = '/data/jupyter/root'
  ```

* 启动`Jupyter Notebook`

  ```bash
  jupyter notebook
  # 访问http://119.29.247.233:8888可进入Jupyter首页
  ```

*  后台运行`Jupyter`，`ctrl+C+y`，并执行下述命令

  ```bash
  nohup jupyter notebook > /data/jupyter/jupyter.log 2>&1 &
  # 该命令将使得 Jupyter 在后台运行，并将日志写在 /data/jupyter/jupyter.log 文件中。
  ```

  ​

