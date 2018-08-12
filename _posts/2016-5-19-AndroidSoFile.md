---
layout: post
title: Android So文件杂谈
---

前一段时间的遇到了项目工程里面遇到So的问题,问题表现是明明放了So库，为什么运行起来程序会崩溃掉,堆栈提示找不到相应的So库。查了很多资料，最后读了一篇很好的博客，终于明白了。英文好的可以读参考文献的链接。

## 为什么会报找不到So库
这是因为你的so库不全，比如手机首选的是armeabi-v7a，只要这个库少一个相应的so，程序运行的加载对应so库的地方，就会崩溃。就算你armeabi或者abi文件夹有这个库也不行，系统只会加载一个对应的文件夹。所以，你必须保证每一个abi文件夹so库都是全的。


## 解决方案

国内比较可行的方案是，只维护armeabi， 或者armeabi-v7a。微信就是这么做的
原因猜测是armeabi兼容性最好。本身so库是很大的，你如果每个都维护的话，工作量是很大的，而且有的第三方类库提供的并不全。你要支持更多的abi的话，就得手动编译so库了。现在Android共支持7中abi，分别是armeabi, armeabi-v7a, x86, mips, arm64-v8a, mips64, x86_64。全部支持的话工作量很大。而且手机mips， x86架构的手机基本见不到。另外，so库本身的大小也是特别大的。维护比较多的话，应用大小也是一个问题。那么怎么可能兼容最多的设备呢？这就涉及到ABI的兼容性问题了。

### ABI的兼容
arm64架构的cpu是兼容armeabi和armeabi-v7a的，armeabi-v7a是兼容armeabi。x86也是兼容armeabi的。

所以微信、支付宝选择只维护armeabi并不是一个随意的选择。另外只维护armeabi-v7a也很常见。

### 其他考量
如果你的app在so库的性能上比较看重，可以测下不同ABI的性能差异。如果差异很明显，维护多个abi也是是一个更好的选择

如果你的app只上传google play的话，可以分ABI上传不同的包。

### 怎么做
那么怎么只打包某种ABI（譬如armeabi）的apk呢，是不是我把其他的ABI文件夹都删掉就好了呢？
其实不然，有的依赖是通过gradle引入的,像[Android gif drawable](https://github.com/koral--/android-gif-drawable) 这样里面包含so文件的类库就不太好处理。
那么怎么解决呢？我一开始想尝试用packageOptions选项可以屏蔽掉，后来发现有更简单的方案。
其实在主工程的build.gradle里面加入类似下面的代码就好了，比如，你想只打包armeabi的so文件

```groovy
android {
    ...
    splits {
        abi {
            enable true
            reset()
             //select ABIs to build APKs for
            include 'armeabi' //'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
        }
    }
    ...
}
```


## 参考资料
[http://ph0b.com/android-abis-and-so-files/](http://ph0b.com/android-abis-and-so-files/)