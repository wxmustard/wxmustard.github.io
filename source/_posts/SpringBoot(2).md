---
title: SpringBoot——Controller
author: mustard
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
abbrlink: a707a952
date: 2017-10-31 20:16:40
---

| 注解              | 作用                                       |
| --------------- | ---------------------------------------- |
| @Controller     | 处理http请求                                 |
| @RestController | Spring4新增注解，等同于@Controller + @ResponseBoby |
| @RequestMapping | 配置url映射，希望用户通过某个url访问到我们写的方法             |

- `url`映射，访问`say`方法

  ![](https://vgy.me/mFHieZ.png)

  需要访问`http://localhost:8080/hello/hi`才能得到结果

| 注解            | 作用        |
| ------------- | --------- |
| @PathVariable | 获取url中的数据 |
| @RequestParam | 获取请求参数的值  |
| @GetMapping   | 组合注解      |

- @PathVariable("id") 获取url中的参数。

  如：localhost:8080/hello/say/100 

- @RequestParam("id") 获取url?id=100

