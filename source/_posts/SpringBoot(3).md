---
title: SpringBoot——项目属性配置
author: mustard
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
abbrlink: 278b56e5
date: 2017-10-29 20:01:28
---

## 项目属性配置

- 目录结构

  ![](https://vgy.me/4cHX62.png)

- new file --> application.yml

  ```yml
  server:
    port: 8081
    context-path: /girl
  ```

- 打开浏览器访问：`http://localhost:8081/girl/hello`



## 使用注解的方式获得配置文件的数据

- 在`application.yml`中写入属性

  ```yml
  StarNum: 22
  Age: 18
  content: "starNum: ${StarNum}. age: ${Age}"
  ```

  content 在配置中使用配置

- 在`HelloController.java`中获得数据并返回页面

  ```java
  @RestController
  public class HelloController {
      @Value("${StarNum}")
      private String StarNum;
      @Value("${Age}")
      private Integer Age;
      @Value("${content}")
      private String content;
      @RequestMapping(value = "/hello", method = RequestMethod.GET)
      public String say(){
          return content;
      }
  }
  ```

  无论在配置文件中数据是什么类型都无所谓，只需要在获得数据时表明类型即可

***

### 一次性设置多个属性

- 将配置写入一个类中

- 添加配置分组 `application.yml`

  ```yml
  girl:
    StarNum: 22
    Age: 18
  ```

- 创建一个类文件`GirlProperties.java`

  ```java
  @Component
  @ConfigurationProperties(prefix = "girl")  //获取前缀是girl的配置
  public class GirlProperties {
      private String StarNum;
      private Integer age;
      public String getStarNum() {
          return StarNum;
      }

      public void setStarNum(String starNum) {
          StarNum = starNum;
      }

      public Integer getAge() {
          return age;
      }

      public void setAge(Integer age) {
          this.age = age;
      }
  }
  ```

- `HelloController.java`

  ```java
  @RestController
  public class HelloController {
      @Autowired
      private GirlProperties girlProperties;
      @RequestMapping(value = "/hello", method = RequestMethod.GET)
      public String say(){
          return girlProperties.getStarNum();
      }
  }

  ```

- 如果出现Spring Boot Configuration Annotation Processor not found in classpath错误，在`pom.xml`中添加依赖

  ```xml
  prefer xxxxxxxxxx <dependency>            <groupId>org.springframework.boot</groupId>            <artifactId>spring-boot-configuration-processor</artifactId>            <optional>true</optional></dependency>

  ```

- 如果在运行时出现Field girlProperties in com.wx.girl.HelloController required a bean of type 'com.wx.girl.GirlProperties' that could not be found.错误

  在`GirlProperties.java`文件中添加`@Component`注解


***

### 分离配置环境

- 复制`application.yml`文件，创建为`dev`版本和`prod`版本

- 修改`application.yml`文件

  ```yml
  spring:
    profiles:
      active: prod
  ```

- 此时启动项目，将从prod版本的配置文件中读取数据

- 还可以使用终端的方式选择配置版本

  ```bash
  mvn install
  java -jar target/~.jar --spring.profiles.active=prod
  ```

  ​

***

@Value 实现配置环境的注入

