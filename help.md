```
1. 所有已有服务域名均修改为新域名 dlcloud.info
                  * notebook和rstudio均修改为 https://tf.dlcloud.info (公网)
                  * wekan看板域名修改为 https://wekan.dlcloud.info (公网)
                  * 内部常用下载点修改为 https://ftp.dlcloud.info (内网)
                  * 代码托管站点修改为 https://git.dlcoud.info (内网) 代码静态部署gitlab-page域名为 https://{username}.blog.dlcloud.info/{repostitory} (内网)
                  * Ubuntu/Centos/Apache/Pypi/Rubygems 镜像源 域名为 https://mirrors.dlcloud.info (内网)
                  * 云平台文档(含各种镜像源使用教程)域名为 https://blcc.blog.dlcloud.info/shyunhelp/ (内网)
                  * Nextcloud云盘域名修改为 https://yun.dlcloud.info (公网)  https://yun-i.dlcloud.info (内网) 具体使用请见 https://blcc.blog.dlcloud.info/shyunhelp/install/nextcloud/
        2.新增服务
                  * Maven/NPM 源加速 https://registry.dlcloud.info (内网)，具体使用请见 https://blcc.blog.dlcloud.info/shyunhelp/usage/mirrors/
                  * Docker Registry 仓库加速 https://docker.dlcloud.ifno (内网)，具体使用请见 https://blcc.blog.dlcloud.info/shyunhelp/usage/mirrors/
                  * dlcloud.info 域名邮箱 ，由 Google G Suite 提供免费服务，可以添加到500人左右，包括Gmail、谷歌云端硬盘等多款谷歌出品的产品均享受无限容量，需要的话请日后在云平台控制台中申请
                  * 校内无污染DNS服务: 10.0.4.111 、10.0.4.112   (内网)  202.121.199.229  202.121.199.235 (公网)
                  * 在线编程C9服务 测试 https://test-c9.dlcloud.info (内网) 账户名 c9  密码 rules
        3.待增服务
                  *  实验室主页 https://blcc.shu.edu.cn (公网) 中英文两版，静态生成，使用内部Git服务托管 (预计在4月15日左右完成----胡红青负责)
                  *  云平台控制台 https://app.dlcloud.info (内网) 只适用于中文、PC访问，虚拟机申请、管理模块(预计在4月25日左右完成----郭治廷负责)，对象存储模块(预计在4月25日左右完成----汪鑫负责)，notebook/rstudio/c9在线编程申请、管理模块 (预计在4月25日左右完成----徐涛负责)，Nextcloud用户申请、管理模块 (预计在4月25日左右完成----周高峰负责)，所述均为后台API，统一采用Python+Django+REST编写，4月13日前完成数据库设计，用户模块、控制台界面由李盛洲负责。
```

