---
title: Dmoj
author: mustard
date: 2017-11-08 19:58:31
tags:
  - ubuntu
  - Dmoj
categories:
  - 技术
---

- 从`github`上`clone`仓库

  ```bash
  git clone https://github.com/DMOJ/judge.git
  ```

- 为镜像更改源

  在`Dockerfile`文件的同级建立文件`sources.list`

  ```list
  deb http://mirrors.shuosc.org/debian stable main contrib non-free
  # deb-src http://mirrors.shuosc.org/debian stable main contrib non-free
  deb http://mirrors.shuosc.org/debian stable-updates main contrib non-free
  # deb-src http://mirrors.shuosc.org/debian stable-updates main contrib non-free

  # deb http://mirrors.shuosc.org/debian stable-proposed-updates main contrib non-free
  # deb-src http://mirrors.shuosc.org/debian stable-proposed-updates main contrib non-free
  ```

- 修改`Dockerfile`文件

  ```bash
  FROM cgswong/min-jessie:latest
  MAINTAINER Quantum <quantum@dmoj.ca>

  RUN groupadd -r judge && useradd -r -g judge judge
  # 添加镜像源
  COPY sources.list /etc/apt/sources.list
  RUN apt-get -y update && apt-get install -y --no-install-recommends python python2.7-dev python3 gcc g++ wget file && apt-get clean
  RUN wget -q --no-check-certificate -O- https://bootstrap.pypa.io/get-pip.py | python && \
      pip install --no-cache-dir pyyaml watchdog cython ansi2html termcolor && \
      rm -rf ~/.cache
  RUN mkdir /problems
  # 创建挂载点
  VOLUME ["/problems"]
  COPY . /judge
  WORKDIR /judge

  RUN env DMOJ_REDIST=1 python setup.py develop && rm -rf build/
  ```

- 构建镜像(在`Dockerfile`当前目录)

  ```bash
  sudo docker build -t .
  ```

- 查看镜像并更改镜像名

  ```bash
  docker images
  sudo docker tag imagesID dmoj1:lastest
  ```

- 创建容器

  ```bash
  sudo docker run -it -v /home/dmoj:/peoblems dmoj1:lastest
  ```

- 未完待续