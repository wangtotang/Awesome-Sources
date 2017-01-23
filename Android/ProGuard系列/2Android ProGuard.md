# ProGuard系列 - Android ProGuard

    导读：
    　　在上篇我们了解了一些ProGuard的基础，如果没有了解，也没有关系，让我们来开始学习Android中的ProGuard,也可以直接
    使用ProGuard,不过记得收藏，以后可能需要的时候，可以学习一下哦。
    　　这是这个系列的第二篇文章，主要来了解Android ProGuard配置。

### Android ProGuard配置

　　在Android的ProGuard中主要有两个功能：压缩和混淆代码（优化和预校验一般被屏蔽，可以自行打开）。ProGuard通过移除没有
使用的类、方法、字段和特性来压缩代码，减少程序包的体积大小；而混淆代码通过使用简短的名称，如a、b、c，来命名类、方法和变量名，
增加程序被反编译的难度。

#### SDK中的ProGuard工具

　　由于ProGuard的使用非常方便，它在Android开发中得到广泛应用。Google在SDK中集成ProGuard工具，它可以在Android SDK的
`tools/proguard`目录下找到。
　　在此目录下,有以下这么些资源：

![ProGuard Tools][ProGuard Tools]
　　

    PS:这里使用的Android SDK为25.2.5；

　　在Android Studio中，官方默认使用的混淆工具是ProGuard。而在这篇文章里所讲的ProGuard配置也是以Android Studio中的配置
作为例子，如果你使用Eclipse+ADT的方式进行开发，可能不尽相同。
　　在


### ProGuard造成的异常

### ProGuard堆叠追踪

[ProGuard Tools]:https://github.com/wangtotang/Awesome-Sources/blob/master/pictures/ProGuard_Tools.jpg