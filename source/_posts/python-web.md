---
title: python_web
author: mustard
date: 2017-11-29 09:13:53
tags:
  - python 
  - web
categories:
  - 技术

---

## 基础

- `pip`命令自动补全

  ```bash
  ➜  ~ pip completion --zsh >> ~/.zprofile
  ➜  ~ source ~/.zprofile 
  # pip升级
  pip install --upgrade pip
  ```

- Python项目中必须包含一个 requirements.txt 文件，用于记录所有依赖包及其精确的版本号。以便新环境部署。

  ```bash
  pip freeze > requirements.txt # 生成requirements.txt
  pip install -r requirements.txt # 从requirements.txt安装依赖
  ```

  ​

