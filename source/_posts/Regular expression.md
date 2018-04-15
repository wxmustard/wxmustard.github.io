---
title: 正则表达式
author: mustard
date: 2017-11-08 14:55:04
tags:
  - python
categories:
  - 技术
---

- http://rubular.com/   正则表达式在线测试网站

- 正则表达式

  正则表达式使用单个字符串来描述、匹配一系列匹配某个句法规则的字符串。

- 语法

  | 字符         | 描述                            |
  | ---------- | ----------------------------- |
  | *          | 0或多次                          |
  | +          | 1或多次                          |
  | ？          | 0或1次                          |
  | ^          | 字符串开始                         |
  | $          | 字符串结束                         |
  | [0-9]   \d | 任意一个0123456789                |
  | [a-z]      | 任意一个小写字母                      |
  | [A-Z]      | 任意一个大写字母                      |
  | \w         | 等同于[a-z0-9A-Z_]匹配大小写字母、数字和下划线 |
  | \W         | 等同于[ ^a-z0-9A-Z_]等同于上一条取非     |
  | \D         | 匹配数字以外的字符                     |

- 匹配学校邮箱

  ```python
  import re
  str = r'^[a-zA-Z0-9_-]+@(staff.|i.|t.|oa.|zbb.)?shu\.edu\.cn$'
  matchObj = re.match(str,''wang_xin@i.shu.edu.cn'')
  print(matchObj.group())
  if re.match(str,'wang_xin@i.shu.edu.cn'):
      print('ok')
  else:
      print('no')
  ```

  ​

