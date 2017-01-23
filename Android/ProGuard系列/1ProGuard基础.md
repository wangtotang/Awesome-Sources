# ProGuard系列 - ProGuard基础

    导读：
    　　我在学习Android的时候，知道ProGuard是用来混淆和压缩代码的，但使用的时候都只是按网上的第三方库提供的ProGuard
    配置进行复制粘贴，并没有理解里面的原理，在这个系列文章里，将总结我从头学习ProGuard的知识点，让我们来一次有趣的
    ProGuard之旅吧！
        这是这个系列的第一篇文章，主要来了解ProGuard基础知识，包括功能和基础语法。

### 1 ProGuard介绍

　　ProGuard是用于Java文件的压缩、优化、混淆和预校验的免费工具，利用它可以检查和移除无用代码减少程序包体积，优化和混淆
并对代码进行预校验。

　　ProGuard提供模板化配置，还有简单的命令行选项以及快速的处理过程的优点，广泛用于各种开发场景，特别是Android应用。

    PS：ProGuard的用处：
    　　● 创建更紧凑的代码和更小的程序包，便于网络传输、减少内存占用以及更快的加载；
    　　● 使得程序更难被反编译； 　 　　　
    　　● 检查出无用代码，并移除； 　 　　　
    　　● 重定向和预校验Java类文件。 　 　　　

#### 1.1 ProGuard功能

　　完整的ProGuard由压缩、优化、混淆和预校验四个步骤组成，每个步骤都默认开启，我们可以在配置文件中决定是否取消这些步骤。

![ProGuard Progress][ProGuard Progress]

　　ProGuard按照上图的顺序依次执行，首先检查发现没有使用的类、方法、字段和特性并对其进行移除，减少程序包的体积大小；然后
是对字节码进行优化，移除无用的指令(因为Android打包会进行优化，默认不开启)；再然后是混淆，通过使用简短的名称，如a、b、c，
来命名类、方法和变量名，增加程序被反编译的难度(这里需要注意利用反射等动态加载的代码，以免混淆后报错)；最后是代码预校验，
确保加载的class文件是可执行的(同样因为Android自带代码校验，默认不开启)，最后输出新程序包。

    PS：由于Java先被编译成.class字节码文件，然后在打成程序包，如war、jar和apk等，都很容易解压得到字节码文件，而字节码
    文件中包含几乎所有的程序信息，可以很容易被反编译出来得到完整的程序文件，被一些别有居心的人利用，不但无法保证版权，
    而且还会造成很大的损失，为了更好的保护应用程序运行，我们需要对编译好的class文件进行混淆。
    　　上面四个步骤都是可选的，可以通过以下选项进行关闭功能：
    　
    　　#不进行压缩
    　　-dontshrink
    　　#不进行优化
    　　-dontoptimize
    　　#不进行混淆
    　　-dontobfuscate
    　　#不进行预校验
    　　-dontpreverify

#### 1.2 工作原理
　　
    在上面讲到，ProGuard检查出没有使用的代码，然后移除，它是如何发现没有使用的代码呢？这里就要引入一个入口点(Entry Point)
的概念。入口点是指被保持的代码，来决定哪些代码不被移除或者混淆。
* 在压缩阶段，ProGuard会从入口点开始递归遍历，搜索哪些类和类成员被使用，而其余的没有使用的类和类成员就会被移除；
* 在优化阶段，ProGuard会用private、static或者final来修饰非入口点的类和方法，无用的参数会被移除，还有将一些方法变成
内联方法；
* 在混淆阶段，ProGuard保留作为入口点的类和类成员的原来名称，重命名非入口点的类和类成员；
* 预校验阶段是唯一一个不用知道入口点的处理阶段。

　　那入口点怎么来的呢？其实我们可以通过在ProGuard配置文件中使用Keep选项进行配置它。在接下来一节，我们就开始学习如何使用
ProGuard。

#### 1.3 小结
　　
    ProGuard处理代码有四个步骤，它们都是可选的，可以通过指令选项关闭；ProGuard通过入口点来递归处理代码，这入口点就是我们
配置的需要保持的代码。学习如何配置ProGuard，这是使用ProGuard的最主要的部分。

    PS：这一部分内容可以查看ProGuard介绍网页：
    https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html。
   [ProGuard介绍](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)

### 2 ProGuard基础语法

　　ProGuard的语法很简单：
* 以一行作为一条语句，支持单空格作为分隔符，额外的空格会被忽略；
* 支持使用 `#` 进行行注释；
* 文件名有空格等特殊符号的用单、双引号标记；
* ProGuard提供很多通配符给我们使用，不同通配符使用的地方可能不同，这里列出通配符(大致)含义：
   * '`?`'表示单个字符,
   * '`*`'表示多个字符 (但不支持包名分隔符),
   * '`**`'表示任意多个字符,
   * '`%`'表示基本类型,
   * '`***`'表示任意类型,
   * '`...`'表示多个参数,
   * `<init>`表示构造方法,
   * `<fields>`表示字段,
   * `<methods>`表示方法.

#### 2.1 Keep选项

　　想要使用ProGuard,就要熟悉ProGuard指令选项。Android中我们常用的指令选项一般是Keep选项,这里着重了解一下Keep选项的
用法。

```ProGuard
# ------------------- 不被移除且不被混淆 ----------------------
#保持类及其成员
-keep [,modifier,...] class_specification

 #保持类成员
-keepclassmembers [,modifier,...] class_specification

#保持含有指定成员及其类
-keepclasseswithmembers [,modifier,...] class_specification

# ---------------------- 不被混淆 ----------------------------
#保持类及其成员的名称
#等同于 -keep,allowshrinking class_specification
-keepnames class_specification

#保持类成员的名称
#等同于 -keepclassmembernames,allowshrinking class_specification
-keepclassmembernames class_specification

#保持含有指定成员及其类的名称
#等同于 -keepclasseswithmembers,allowshrinking class_specification
-keepclasseswithmembernames class_specification

```
　　现在我们尝试使用一下Keep选项，
* 保持一个Activity
```ProGuard
# 保持指定的类名，需要全路径类名
-keep public class mypackage.MyActivity
```
* 保持一些成员方法
```ProGuard
# 保持任意enum类中的static修饰values(),valueOf()方法
# 参数类型也需要全路径类名
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```
* 保持一些成员名称
```ProGuard
# 保持class$()方法名称,class$()方法是Java编译器实现构造的,无需理会
-keepclassmembernames class * {
    java.lang.Class class$(java.lang.String);
    java.lang.Class class$(java.lang.String, boolean);
}
```
* 保持指定成员名称及其类名称
```ProGuard
# 保持含有native修饰的方法的类名称和native修饰的方法名称
-keepclasseswithmembernames class * {
    native <methods>;
}
```

    PS:在我们单独使用*代表类名时，表示任意类名，否则,不单独使用仅匹配多个字符，例如Test*,表示Test1或TestWorld等，且不含
    有包分隔符，例如不能表示Test.World；
　　
    我们在使用的时候，如何具体地写保持的内容呢？下面是完整的模板定义：
```Java
[@annotationtype] [[!]public|final|abstract|@ ...] [!]interface|class|enum classname
    [extends|implements [@annotationtype] classname]
[{
    [@annotationtype] [[!]public|private|protected|static|volatile|transient ...] <fields> |
                                                                                  (fieldtype fieldname);
    [@annotationtype] [[!]public|private|protected|static|synchronized|native|abstract|strictfp ...] <methods> |
                                                                                                     <init>(argumenttype,...) |
                                                                                                     classname(argumenttype,...) |
                                                                                                     (returntype methodname(argumenttype,...));
    [@annotationtype] [[!]public|private|protected|static ... ] *;
    ...
}]
```

　　Keep选项给我们一种很混乱的感觉，有时候可能不确定选择使用哪一个，其实我们大可不必纠结，直接使用-keep就好了，它能确保
指定的类及其成员不被移除且不被混淆。例如：
```ProGuard
# 保持model包及其子包下的所有类和所有成员
-keep class com.google.gson.examples.android.model.** { *; }
```

    PS:对于Keep选项，我们要记得两点：
       1.只指定类但没有指定类成员，那么类成员可能会被移除、优化或者混淆；
       2.只指定类成员，与其相关联的代码可能会被优化或者混淆。

#### 2.2 指令选项

　　指令选项分成7种，这里只列出在Android中常用到的部分。

##### I/O选项
```ProGuard
# 不忽略类库中非public的类
-dontskipnonpubliclibraryclasses
```

##### Keep选项
```ProGuard
# 保持ILicensingService这个公共类，名称要使用全路径类名
-keep public class com.google.vending.licensing.ILicensingService

# 对于R（资源）下的所有类及其方法，都不能被混淆
# 这里的$是Java编译器自动编译时内部类产生的
-keep class **.R$* {
    *;
}

# 保持继承自View类的set()、get()方法
# 这里的class *表示任意类，不考虑包名
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# 保持使用@Keep注解的字段及其类
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

# 保持native方法及其类(移除后剩下的)
# 等同于 -keepclasseswithmembers,allowshrinking,这里的allowshrinking是Keep选项的修饰符
-keepclasseswithmembernames class * {
    native <methods>;
}
```

##### 压缩选项
```ProGuard
# 列出无用代码
-printusage
```

##### 优化选项
```ProGuard
# 关闭优化
-dontoptimize

# 优化控制选项，通过指定优化名称进行过滤操作
# 不对算术和类型转化操作符优化，'!'表示排除匹配到的优化
-optimizations !code/simplification/arithmetic,!code/simplification/cast

# 优化次数，如果没有发现可优化的，就会结束优化
# 在0和7之间，默认为5
-optimizationpasses 5

# 允许提升访问修饰符，扩大可访问范围
-allowaccessmodification
```

##### 混淆选项
```ProGuard
# 不使用大小写混合的类名（默认设置）
# 混淆后的类名为小写
-dontusemixedcaseclassnames

# 保持Annotations属性
-keepattributes *Annotation*

# 保持泛型属性
-keepattributes Signature

# 保持代码行号
# 方便在异常分析中定位
-keepattributes SourceFile,LineNumberTable
```

##### 预校验选项
```ProGuard
# 关闭预校验
-dontpreverify
```

##### 通用选项
```ProGuard
# 打印处理代码的信息，出现异常时，会打印完整栈追踪信息
-verbose

# 生成映射文件
# 包含有类名->混淆后类名的映射关系
# 与-verbose一起使用
-printmapping proguardMapping.txt

# 不打印处理过程中的警告信息
-dontwarn android.support.**

# 不打印关于错误或缺省配置的说明信息
-dontnote rx.internal.util.PlatformDependent
```
    PS：优化名称清单（挺多的，简略写）：
    　  ● class/marking/final  #尽量使用final修饰类
    　  ● class/merging/*      #尽量使类层次成vertical或者horizontal
    　  ● field/*              #字段优化相关
    　  ● method/*             #方法优化相关
    　  ● code/*               #代码优化相关

#### 2.3 小结
　　
    ProGuard的配置文件类似脚本文件，指令选项就是它的命令行。指令选项虽然有7种，不过Keep选项是ProGuard指令选项的主要部分，
学习掌握Keep选项和常用的指令选项可以让我们使用Android ProGuard事半功倍。

    PS：这一部分内容可以查看ProGuard使用指南：
    https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/usage.html。
   [使用指南](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/usage.html)

[ProGuard Progress]:https://github.com/wangtotang/Awesome-Sources/blob/master/pictures/ProGuard_Progress.png