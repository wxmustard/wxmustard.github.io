---
title: git使用教程
author: mustard
tags:
  - git
categories:
  - 技术
abbrlink: de286b5f
date: 2017-10-02 20:32:11
---

* 进入 https://git.shuosc.cn/dashboard/projects 建立一个新的项目`new project`

* 公钥上传至服务器

  ```bash
  ls .ssh
  ssh-keygen -t rsa
  ls .ssh
  cat .ssh/id_rsa.pub
  ```

  将`id_rsa.pub`中的内容拷贝到 https://git.shuosc.cn/profile/keys 中

* 验证是否成功

  ```bash
  ssh -T git@git.shuosc.cn
  ```

  出现用户名即为连接成功

* 建立一个工作目录来保存代码等文档

  ```bash
  mkdir workmeau
  ```

* 进入工作目录，将目录中的文档克隆到git中

  ```bash
  cd workmeau 
  workmeau git clone ://git@git.shuosc.cn/wangxin/linux-note.git

  ```

* 此时，工作目录下会出现一个`linux-note`文件夹，里面包含了需要上传的文档，进入此目录会出现`linux-note git:(master) ✗`，代表`git`的主干，`✗`代表文件尚未提交，下面对文件进行跟踪和提交

  ```bash
  cd linux-note
  git status # 跟踪文件,检查目前所做的修改
  git add -A # 暂存需要提交的文件
  git commit -m 'firsh push'  # 提交已暂存的文件
  # 提示 please tell me who you are
  git config --global user.email "wangxin175@shu.edu.cn"
  git config --global user.name "wangxin"
  git commit -m 'first push'
  git push origin master # 将文件提交到服务器
  # 将文件从远程下载至本地
  git clone git@git.shuosc.cn:wangxin/wangxin.shuosc.cn.git 
  # 添加远程账户
  git remote add github git@github.com:wxmustard/wxmustard.github.io.git
  ```

  > ​
  >
  > ![流程](https://vgy.me/m9OQXI.png)
  >
  > ​

  ​

  > - Workspace：工作区
  > - Index / Stage：暂存区
  > - Repository：仓库区（或本地仓库）
  > - Remote：远程仓库


## 在github上提交文件后没有贡献值？

* 确认提交的用户是否是当前用户

* 进入工作目录下修改用户以及邮箱

  ```bash
  git config --global user.name wxmustard 
  git config --global user.email mustard@live.cn
  ```

  ​

  ​