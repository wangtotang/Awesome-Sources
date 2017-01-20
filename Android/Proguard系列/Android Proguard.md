# 有趣的 ProGuard - Android ProGuard

    导读：
    　　我在学习Android的时候，知道ProGuard是用来混淆和压缩代码的，但使用的时候都只是按网上的第三方库提供的ProGuard
    配置进行复制粘贴，并没有理解里面的原理，在这个系列文章里，将总结我从头学习ProGuard的知识点，让我们来一次有趣的
    ProGuard之旅吧！

　　ProGuard是用于Java文件的压缩、优化、混淆和预校验的免费工具，利用它可以检查和移除无用代码减少程序包体积，优化和混淆
并对代码进行预校验。
　　ProGuard提供模板化配置，还有简单的命令行选项以及快速的处理过程的优点，广泛用于各种开发场景，特别是Android应用。

    PS：ProGuard的用处：
    　　● 创建更紧凑的代码和更小的程序包，便于网络传输、减少内存占用以及更快的加载；
    　　● 使得程序更难被反编译； 　 　　　
    　　● 检查出无用代码，并移除； 　 　　　
    　　● 重定向和预校验Java 6的现有类文件。 　 　　　

#### 1.ProGuard功能

　　在Android的ProGuard中主要有两个功能：压缩和混淆代码（优化和预校验一般被屏蔽，可以自行打开）。ProGuard通过移除没有使用
的类、方法和属性来压缩代码，减少程序包的体积大小；而混淆代码通过使用简短的名称，如a、b、c，来命名类、方法和变量名，增加
程序被反编译的难度。

![ProGuard Progress][ProGuard Progress]

　　ProGuard按照上图的顺序依次执行，首先检查发现没有使用的类、成员方法和属性，然后移除(这里需要注意利用反射等动态加载的代
码，以免移除后报错)，然后是优化代码(因为Android打包会进行优化，默认不开启)，再然后是混淆，最后是代码预校验(同样
因为Android自带代码校验，默认不开启)，最后输出新程序包。

    PS：由于Java先被编译成.class字节码文件，然后在打成程序包，如war、jar和apk等，都很容易解压得到字节码文件，而字节码文
    件中包含几乎所有的程序信息，可以很容易被反编译出来得到完整的程序文件，被一些别有居心的人利用，不但无法保证版权，而且还
    会造成很大的损失。
    　　上面四个步骤都是可选的，可以通过以下选项进行关闭功能：
    　
    　　#不进行压缩
    　　-dontshrink
    　　#不进行优化
    　　-dontoptimize
    　　#不进行混淆
    　　-dontobfuscate
    　　#不进行预校验
    　　-dontpreverify


#### 2.ProGuard基础语法

　　ProGuard的语法很简单：
1. 以一行作为一条语句，支持单空格作为分隔符，额外的空格会被忽略；
2. 支持使用 `#` 进行行注释；
3. 文件名有空格等特殊符号的用单、双引号标记；
4. ProGuard提供很多通配符给我们使用，不同通配符使用的地方可能不同，这里列出通配符(大致)含义：
   * '`?`'表示单个字符,
   * '`*`'表示多个字符 (但不支持包名分隔符),
   * '`**`'表示任意多个字符,
   * '`%`'表示基本类型,
   * '`***`'表示任意类型,
   * '`...`'表示多个参数,
   * `<init>`表示构造方法
   * `<fields>`表示字段
   * `<methods>`表示方法

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
1. 保持一个Activity
```ProGuard
# 保持指定的类名，需要全路径类名
-keep public class mypackage.MyActivity
```
2. 保持一些成员方法
```ProGuard
# 保持任意enum类中的static修饰values(),valueOf()方法
# 参数类型也需要全路径类名
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```
3. 保持一些成员名称
```ProGuard
# 保持class$()方法名称,class$()方法是Java编译器实现构造的,无需理会
-keepclassmembernames class * {
    java.lang.Class class$(java.lang.String);
    java.lang.Class class$(java.lang.String, boolean);
}
```
4. 保持指定成员名称及其类名称
```ProGuard
# 保持含有native修饰的方法的类名称和native修饰的方法名称
-keepclasseswithmembernames class * {
    native <methods>;
}
```

    PS:在我们单独使用*代表类名时，表示任意类名，否则仅匹配多个字符，例如Test*,表示Test1或TestWorld等，且不含有包分隔符，例如不能表示Test.World；
　　
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

　　Keep选项给我们一种很混乱的感觉，有时候可能不确定选择使用哪一个，其实我们大可不必纠结，直接使用-keep就好了，它能确保指
定的类及其成员不被移除且不被混淆。例如：
```ProGuard
# 保持model包及其子包下的所有类和所有成员
-keep class com.google.gson.examples.android.model.** { *; }
```

    PS:对于Keep选项，我们要记得两点：
       1.只指定类但没有指定类成员，那么类成员可能会被移除、优化或者混淆；
       2.只指定类成员，与其相关联的代码可能会被优化或者混淆。

　　指令选项分成7种，这里只列出在Android中常用到的部分。

```ProGuard
# ------------------ Begin: I/O选项 ------------------

# 不忽略类库中非public的类
-dontskipnonpubliclibraryclasses

# ------------------ End  : I/O选项 ------------------

# ------------------ Begin: Keep选项 ------------------

# 保持ILicensingService这个公共类，名称要使用全路径类名
-keep public class com.google.vending.licensing.ILicensingService

# 保持View类的set()、get()方法
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

# ------------------ End  : Keep选项 ------------------

# ------------------ Begin: 压缩选项 ------------------

# 列出无用代码
-printusage

# ------------------ End  : 压缩选项 ------------------

# ------------------ Begin: 优化选项 ------------------
# 关闭优化
-dontoptimize

# 优化控制选项，通过指定优化名称进行过滤操作
# 不对算术和类型转化操作符优化，'!'表示排除匹配到的优化
-optimizations !code/simplification/arithmetic,!code/simplification/cast

# 优化次数，如果没有发现可优化的，就会结束优化
-optimizationpasses 5

# 允许提升访问修饰符，扩大可访问范围
-allowaccessmodification

# ------------------ End  : 优化选项 ------------------

# ------------------ Begin: 混淆选项 ------------------

# 不使用大小写混合的类名（默认设置）
-dontusemixedcaseclassnames

# 保持Annotations属性
-keepattributes *Annotation*

# ------------------ End  : 混淆选项 ------------------

# ------------------ Begin: 预校验选项 ------------------

# 关闭预校验
-dontpreverify

# ------------------ End  : 预校验选项 ------------------

# ------------------ Begin: 通用选项 ------------------

# 打印处理代码的信息，出现异常时，会打印完整栈追踪信息
-verbose

# 不打印处理过程中的警告信息
-dontwarn android.support.**

# 不打印关于错误或缺省配置的说明信息
-dontnote rx.internal.util.PlatformDependent

# ------------------ End  : 通用选项 ------------------


```
    PS：优化名称清单（挺多的，简略写）：
    　  ● class/marking/final  #尽量使用final修饰类
    　  ● class/merging/*      #尽量使类层次成vertical或者horizontal
    　  ● field/*              #字段优化相关
    　  ● method/*             #方法优化相关
    　  ● code/*               #代码优化相关

#### 3.Android ProGuard配置
#### 4.ProGuard造成的异常
#### 5.ProGuard堆叠追踪


[使用指南]:https://developer.android.google.cn/studio/build/shrink-code.html#shrink-code
[ProGuard Progress]:https://github.com/wangtotang/Awesome-Sources/blob/master/pictures/Proguard_Progress.png