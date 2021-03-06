---
title: SpringBoot——第一个SpringBoot应用
author: mustard
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
abbrlink: ce5362ed
date: 2017-10-29 19:00:16
---

## 第一个SpringBoot应用

- 使用软件： IDEA 

- 检查`JAVA`和`Maven`版本

- 打开IDEA：new project -->  Spring Initializr -->  SprintBoot version 1.4.7 -->  web --> web --> finish

- 第一次使用`springboot`会加载很多`jar`包，为了加快速度，可以使用阿里云的镜像源

  ```bash
  cd /usr/local/maven/conf
  vim setting.xml
  # 加入以下语句
  <mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
  </mirror>
  ```

- 删除不相关的文件

  ![](https://vgy.me/G6SnUy.png)

- new java class

  ```java
  package com.wx.girl;

  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  import org.springframework.web.bind.annotation.RestController;

  /**
   * Created by mustard
   * 17-10-29 下午7:28
   */
  @RestController
  public class HelloController {
      @RequestMapping(value = "/hello", method = RequestMethod.GET)
      public String say(){
          return "Hello Spring Boot!";
      }
  }
  ```

- 启动HelloController，有三种方法

  - 在`IDEA`界面上方点击`run`

  - 在终端中进入项目目录，使用下面的命令启动项目

    ```bash
    mvn spring-boot:run
    ```

  - 在终端里编译项目

    ```bash
    mvn install
    cd target
    java -jar ~.jar
    ```

- 以上三种方式均可启动项目，在浏览器中输入`http://localhost:8080/hello`即可访问刚刚创建的项目

  ​

  ​

