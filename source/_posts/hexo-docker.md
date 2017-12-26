---
title: hexo-docker
author: mustard
date: 2017-12-20 09:33:47
tags:
  - Docker
  - hexo
categories:
  - 技术

---

## 目标

- 学习使用`hexo`
- 将`github`上的博客`docker`化

## 具体实现步骤

### 方法一：编写`Dockerfile`

```dockerfile
FROM shuosc/ubuntu:latest
MAINTAINER mustard

# install node 
RUN echo y|apt-get install curl git 
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash && export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" 
RUN tee -a ~/.bashrc << NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node && tee -a ~/.bashrc << NVM_DIR="$HOME/.nvm" && /bin/bash -c 'source ~/.bashrc' && cat /root/.bashrc
RUN /bin/bash -c 'nvm install node'

# install hexo
RUN npm install hexo-cli -g
RUN git clone https://github.com/wxmustard/wxmustard.github.io.git && cd wxmustard.github.io && npm install hexo --save && hexo d && hexo s


EXPOSE 4000
```

上面的`Dockerfile`还存在一些问题，在`docker build`过程中默认使用`bin/sh`,而有些命令如`source`只有在`bin/bash`才能执行，所以使用`RUN /bin/bash -c 'source ~/.bashrc' `

###方法二：直接在容器中操作

- 创建容器

```bash
docker pull shuosc/ubuntu
docker run -tid shuosc/ubuntu:latest /bin/bash
docker ps -a
docker exec -ti 容器ID /bin/bash
```

- 在容器内将`hexo  docker`化

```bash
# 安装一些必备的软件
apt-get install wget curl git 
# 使用nvm安装node
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
tee -a ~/.zshrc << EOF
NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
EOF
source ~/.zshrc
nvm install node
# 验证是否安装成功
node -v

# 安装hexo
npm install -g hexo-cli
# 新建一个网站
hexo init blog
cd blog
# 安装需要的组件
npm install
# 生成静态文件
hexo g
# 将文件部署到网站
hexo d
# 启动服务器
hexo s
# 打开浏览器访问localhost:4000
```

![](https://vgy.me/YLqxjz.png)

- 将`github`上的博客`docker`化

```bash
# 安装一些必备的软件
apt-get install wget curl git 
# 使用nvm安装node
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
tee -a ~/.zshrc << EOF
NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
EOF
source ~/.zshrctuozhanbao
nvm install node
# 验证是否安装成功
node -v

# 安装hexo
npm install -g hexo-cli
git clone https://github.com/wxmustard/wxmustard.github.io.git
cd wxmustard.github.io 
npm install hexo --save 
hexo g
hexo d 
hexo s
```



### 问题：

- **Deployer not found: git** 

  安装hexo拓展包

  ```bash
  npm install hexo-deployer-git --save
  ```

  ​

### 安装node.js

- 下载node安装包 [node.js安装包下载地址](https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-x64.tar.xz)

```bash
wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-x64.tar.xz
tar xvJf node-v8.9.3-linux-x64.tar.xz
cd node-v8.9.3-linux-x64/bin
./node -v
# 建立软连接
ln -s /usr/local/node-v0.10.28-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/local/node-v0.10.28-linux-x64/bin/npm /usr/local/bin/npm
```

- 使用nvm方式安装

```bash
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
# 退出shell，再进入
nvm ls-remote 
nvm install v8.9.3
```



## 使用其他docker images完成hexo docker化

```dockerfile
FROM mhart/alpine-node:8

MAINTAINER mustard ,<mustard@live.cn>

RUN apk --update --no-progress add git 

RUN npm install hexo-cli -g && git clone https://github.com/wxmustard/wxmustard.github.io.git \
    && cd wxmustard.github.io/ && npm install 
COPY start.sh /start.sh
EXPOSE 4000
ENTRYPOINT ["/start.sh"]
```

