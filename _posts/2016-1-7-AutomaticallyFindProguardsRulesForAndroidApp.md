---
layout: post
title: 优雅的解决Android混淆即Proguard的(半)自动化问题
---

## 问题
大家都知道，App上线的时候一般都是进行混淆的，一方面能保护我们的源码不被窃取，另一方面也能减少App的Size。
但一个新开发的Apk怎么快速的Proguard规则，让App快速完成混淆的问题呢？

## 解决方法
release版本如果混淆规则没有添加对的话，一般是无法编译通过的，编译失败的日志会显示混淆失败的问题
解决步骤：

1. 收集编译失败日志：

```
gradlew clean aR > release.txt
```

2. 用Python解析日志
核心代码为正则表达式

```python
import re

s = open("release.txt", mode='r', encoding='utf-8').read()
all_class = re.findall("Warning:.*referenced class (.*)", s)
all_field = re.findall("Warning:.* in program class (.*)", s)

all_type = all_class + all_field

seen = set()
proguard_issue = sorted([x for x in all_type if x not in seen and not seen.add(x)])
for i in proguard_issue:
    print(i)
```

3. 根据生成的编写规则
把packages替换成2步骤里面的你筛选的不混淆的规则

```python
packages = """
android.bluetooth
"""

hacklist = packages.split()
print(hacklist)

print("processing...")
for i in hacklist:
    dontwarn = "-dontwarn {}.**".format(i)
    keep = "-keep class {}.** {{*;}}".format(i)
    print("{}\n{}".format(dontwarn, keep))
```

4. 其他规则
主要是Json解析的Model类，一定不要忘了，否则程序很容易闪退
