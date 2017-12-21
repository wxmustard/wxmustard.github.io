---
title: Android-studio
author: mustard
date: 2017-12-21 18:31:38
tags:
  - java
  - Android 
categories:
  - 技术
---

## 遇到的问题

###问题一：无法正常创建`AVD`模拟器？

通过查看`log`发现：

```bash
cat /home/mustard/.AndroidStudio3.0/system/log/idea.log
2017-12-21 18:28:43,432 [se-915-b01]   WARN - vdmanager.AvdManagerConnection - Failed to create the SD card. 
2017-12-21 18:28:43,455 [se-915-b01]   WARN - vdmanager.AvdManagerConnection - Failed to create sdcard in the AVD folder. 
```

解决方法：

找到`android`的`SDK`安装目录

```bash
➜  cd /home/mustard/Android/Sdk
➜  Sdk chmod +x tools/*
➜  Sdk chmod +x platform-tools/*
➜  Sdk sudo apt install lib32z1 lib32ncurses5 lib32stdc++6 
```

### 问题二：android studio Cannot resolve symbol '@drawable/XXX'等问题解决办法：

"Tools" -> "Android" -> "Sync Project with Gradle Files"

## 笔记

- @id与@+id

> id属性只能接受资源类型的值，也就是必须以@开头的值

如果使用@+id，表示当修改完某个布局文件并保存后，系统会自动在R.java文件中生成相应的int类型变量。变量名就是“/”后面的值。

如果使用@id表示使用系统已有的资源。

**@+id 新增一个资源id**

**@id和android:id，引用现有的资源id**