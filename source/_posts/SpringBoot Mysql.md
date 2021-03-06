---
title: SpringBoot——问题
author: mustard
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
abbrlink: a467a3ad
date: 2017-11-02 13:31:06
---

## 遇到的问题

- `Mysql`无法插入中文

  - 可视化界面：`mysql workbench`

  -  查看编码

    ```mysql
    mysql> show variables like 'char%';
    +--------------------------+----------------------------+
    | Variable_name            | Value                      |
    +--------------------------+----------------------------+
    | character_set_client     | utf8                       |
    | character_set_connection | utf8                       |
    | character_set_database   | latin1                     |
    | character_set_filesystem | binary                     |
    | character_set_results    | utf8                       |
    | character_set_server     | latin1                     |
    | character_set_system     | utf8                       |
    | character_sets_dir       | /usr/share/mysql/charsets/ |
    +--------------------------+----------------------------+
    8 rows in set (0.00 sec)
    ```

  - `character_set_database` 和 `character_set_server` 的编码方式不对，应该是`utf8`

  - 修改配置文件 

    ```bash
    sudo vim /etc/mysql/conf.d/mysql.cnf
    #添加以下语句
    [mysql]
    no-auto-rehash
    default-character-set=utf8
    [mysqld]
    socket = /var/run/mysqld.sock
    port =3306
    character-set-server=utf8
    #重启服务
    service mysql restart;
    ```

- `application.yml`

  ```yml
  spring:
    thymeleaf:
      prefix: classpath:/templates/
      suffix: .html
      check-template-location: true
      mode: HTML5
    datasource:
      url: jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&useSSL=false
      username: root
      password: root
      sql-script-encoding: utf-8
    jpa:
      database: mysql
      show-sql: true
      hibernate:
        ddl-auto: update

  ```

  ​


