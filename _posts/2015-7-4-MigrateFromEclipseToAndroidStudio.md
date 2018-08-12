---
layout: post
title: 从Eclipse迁移到Android Studio
---

## 目的
从Eclipse迁移到Android Studio
* 在Eclipse平台android的65536方法数限制并没有解决方案
* ADT的话Google后续也不维护了
* 个人觉得Studio开发效率更高(All glory belong to Jetbrains)

## 导出
Eclipse自带的导出并不是标准的Gradle形式结构，而且so库，aidl也不符合标准
我写了一个Python库来处理，已经用这个库处理过两个大型项目（代码在30万行以上），自己用着也舒服，开源出来
[https://github.com/gengjiawen/migratetogradle](https://github.com/gengjiawen/migratetogradle)

## 迁移的坑（编译）
代码导出后最大的问题是编译问题，那么都有那些坑呢，以下是我遇到的

* .9图片必须是真正的.9图片
* 图片后缀名要正确，jpeg图片后缀名如果是png的话编译无法通过
* 不能有重复的包依赖
* 编码问题，必须是UTF-8编码，GBK编码，UTF-8 with bom都不行

## 填坑
* 重新作图
* 换回正确的后缀名,检测方法（Python: imghdr [https://docs.python.org/3.4/library/imghdr.html](https://docs.python.org/3.4/library/imghdr.html)
* 重新梳理包依赖
* 很多平台下都有chardet方案，我在项目中使用的是Python方案


## 结语
编译通过了，可能最大的问题就是熟悉Gradle和Android Studio的问题了，这个要花时间慢慢学了
Welcome to Gradle & Studio
