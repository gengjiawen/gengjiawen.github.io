---
layout: post
title: Android App的发布版（Release）屏蔽日志问题的解决方案
---

## 动机
一般情况下，为了保证App的设计、方案和用户隐私数据等敏感信息不在Log中泄露而做的这一个。其实不仅仅是这样，如果是商业产品的话，你如果在日志中泄露了用户的mac地址等敏感信息，会有法务上的问题，很容易给公司造成声誉的损失。

## 方案
* 方案一： 写一个LogUtil类，设置boolean值控制
这个方案一般情况下问题不大，但是这个不能保证第三方包里面的Log和Code Review不强力的情况下有人还是使用了
* 方案二： Proguard方法
这个方法是我为止看到最优秀的方法
方法实现：
在build.gradle的proguard选项：

```
proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
```

请务必注意是选择的Proguard文件是带optimize选项的那个，默认的不带optimize选项的Proguard文件
然后在proguard-rules.pro中添加上这个就好了（PS: 如果有其他LogUtil，一并加上）

```
-assumenosideeffects class android.util.Log { public * ; }
```
