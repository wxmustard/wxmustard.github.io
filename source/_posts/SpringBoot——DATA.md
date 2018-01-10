---
title: SpringBoot——数据库
author: mustard
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
abbrlink: 6de21891
date: 2017-11-01 08:59:32
---

- `Spring-Data-Jpa` ：`Spring` 与`Hibernate`的组合

  `JPA`:定义了一系列对象持久化的标准，目前实现这一规范的产品有`Hibernate`和`Toplink`等

- 实例：

  - 添加数据库依赖

    - ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
      </dependency>
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
      </dependency>
      ```

    - ？在pom.xml中添加依赖时不自动提示

      解决方法：setting --> Build,execution,deployment --> Build tools -->Maven --> Repositories --> update

    - ?  pom.xml中添加的依赖勾红线

      解决方法： 更换`Maven`的镜像源，打开

      ![](https://vgy.me/ihYmy4.png)

      查看User setting file 的地址，大多数情况下此地址下是没有`settings.xml`文件的，需要新建，在文件中添加阿里云的镜像源 **override的复选框一定要打钩**

      ```xml
      <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                                https://maven.apache.org/xsd/settings-1.0.0.xsd">

            <mirrors>
              <mirror>  
                  <id>alimaven</id>  
                  <name>aliyun maven</name>  
                  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
                  <mirrorOf>central</mirrorOf>          
              </mirror>  
            </mirrors>
      </settings>
      ```

      回到`IDEA`，刷新`pom.xml`即可

  - 设置`application.yml`中的数据库信息

    ```yml
    spring:
      profiles:
        active: dev

      datasource:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://172.28.0.1:3306/wx?useUnicode=true&characterEncoding=utf-8&useSSL=false
        username: root
        password: root
      jpa:
        hibernate:
          ddl-auto: create
        show-sql: true
    ```

  - `ddl-auto`类型区分：

    |     类型      |                  作用                  |
    | :---------: | :----------------------------------: |
    |   create    | 每一次运行时都创建一个新的表，如果之前有同名的表，删掉同名的在创建新的表 |
    |   update    |   第一次运行时也会创建相对应的表结构，但是不会删除其中已存在的数据   |
    |  validate   |     验证数据库里的表结构和类中的属性是否一致，不一致将会报错     |
    | create-drop |            应用停下来的时候，将表删除             |

  - 添加一个对应数据库中表的类

    ```java
    package com.wx.girl;
    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.Id;

    /**
     * Created by mustard
     * 17-11-1 上午9:43
     */
    @Entity  //与数据库对应的注解
    public class Girl {

        @Id
        @GeneratedValue   //添加自增的Id字段
        private Integer id;

        public Integer getId() {
            return id;
        }

        public String getName() {
            return name;
        }

        public Integer getAge() {
            return age;
        }

        private String name;

        private Integer age;
        //使用无参构造函数
        public Girl() {
        }
    }
    ```


  - 运行时出现错误提醒：

    WARN: Establishing SSL connection without server’s identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn’t set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

    解决方法：在数据库连接的库名后添加`useUnicode=true&characterEncoding=utf-8&useSSL=false`

    ​

  - ![](https://vgy.me/bbfsC0.png)

  - ​

