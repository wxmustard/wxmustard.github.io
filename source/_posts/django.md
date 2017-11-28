---
title: django
author: mustard
date: 2017-11-28 10:43:31
tags:
  - ubuntu
  - django
categories:
  - 技术

---

## 安装`django`

```bash
# pip3--python3.5  pip--python2.7
sudo pip3 install Django
# 检查是否安装成功
python3
>>> import django
>>> django.VERSION
(1, 11, 7, 'final', 0)
>>> django.get_version()
'1.11.7'
# 成功输出版本号，安装成功
```

## 搭建多个互不干扰的开发环境

>在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.4。所有第三方的包都会被`pip`安装到Python3的`site-packages`目录下。
>
>如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？
>
>这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境.

- 安装`virtualenv`

  ```bash
  sudo pip install virtualenv virtualenvwrapper
  ```

- 更改`.zshrc`

  ```bash
  export WORKON_HOME=/home/mustard/.virtualenvs
  export PROJECT_HOME=/home/mustard/workspace
  export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python
  source /usr/local/bin/virtualenvwrapper.sh
  # 使更改生效
  source .zshrc
  ```

- 创建一个新的项目

  ```bash
  # 创建项目目录
  mkdir myproject
  cd myproject/
  # 创建一个独立的Python运行环境，命名为venv
  virtualenv --no-site-packages venv
  # virtualenv命令是创建一个独立的Python运行环境
  # --no-site-packages参数是创建一个干净的不包含第三方包的Python运行环境

  #进入venv环境
  source venv/bin/activate
  # 此时可以使用pip install等命令了

  ```

  ![](https://vgy.me/nS94OF.png)

