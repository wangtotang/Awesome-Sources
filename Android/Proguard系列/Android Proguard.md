# 有趣的 ProGuard - Android ProGuard

    导读：
       我在学习Android的时候，知道ProGuard是用来混淆和压缩代码的，但使用的时候都只是按网上的第三方库提供的ProGuard
    配置进行复制粘贴，并没有理解里面的原理，在这个系列文章里，将总结我从头学习ProGuard的知识点，让我们来一次有趣的
    ProGuard之旅吧！

#### 1.ProGuard功能

　　在Android的ProGuard中主要有两个功能：压缩和混淆代码（优化和预校验一般被屏蔽，可以自行打开）。ProGuard通过移除没有使用
的类、方法和属性来压缩代码，减少程序包的体积大小；而混淆代码通过使用简短的名称，如a、b、c，来命名类、方法和变量名，增加
程序被反编译的难度。

![ProGuard Progress][ProGuard Progress]

　　proguard按照上图的顺序依次执行，首先检查发现没有使用的类、成员方法和属性，然后移除(这里需要注意利用反射等动态加载的代
码，以免移除后报错)，然后是优化代码(因为Android打包会进行优化，默认不开启)，再然后是混淆，最后是代码预校验(同样
因为Android自带代码校验，默认不开启)，最后输出新程序包。

    PS：由于Java先被编译成.class字节码文件，然后在打成程序包，如war、jar和apk等，都很容易解压得到字节码文件，而字节码文
    件中包含几乎所有的程序信息，可以很容易被反编译出来得到完整的程序文件，被一些别有居心的人利用，不但无法保证版权，而且还
    会造成很大的损失。
        上面的

#### 2.ProGuard语法
#### 3.Android ProGuard配置
#### 4.ProGuard造成的异常
#### 5.ProGuard堆叠追踪


[使用指南]:https://developer.android.google.cn/studio/build/shrink-code.html#shrink-code
[ProGuard Progress]:https://github.com/wangtotang/Awesome-Sources/blob/master/pictures/Proguard_Progress.png