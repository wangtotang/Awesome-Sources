# ProGuard系列 - Android ProGuard

    导读：
    　　在上篇我们了解了一些ProGuard的基础，如果没有了解，也没有关系，让我们来开始学习Android中的ProGuard,也可以直接
    使用ProGuard,不过记得收藏，以后可能需要的时候，可以学习一下哦。
    　　这是这个系列的第二篇文章，主要来了解Android ProGuard配置。

### 1 Android ProGuard基本内容

　　在Android的ProGuard中主要有两个功能：压缩和混淆代码（优化和预校验一般被屏蔽，可以自行打开）。ProGuard通过移除没有
使用的类、方法、字段和特性来压缩代码，减少程序包的体积大小；而混淆代码通过使用简短的名称，如a、b、c，来命名类、方法和变量名，
增加程序被反编译的难度。

#### 1.1 SDK中的ProGuard工具

　　由于ProGuard的使用非常方便，它在Android开发中得到广泛应用。Google在SDK中集成ProGuard工具，它可以在Android SDK的
`tools/proguard`目录下找到。

　　在此目录下,有以下这么些资源：

　![ProGuard Tools][ProGuard Tools]

　　这些资源分别是：
* `ant`文件夹放的是Ant构建工具的task配置文件;
* `bin`文件夹放的是bat批处理脚本，用来运行ProGuard程序；
* `docs`文件夹放的是ProGuard HTML文档；
* `examples`文件夹放的是配置文件的例子；
* `lib`文件夹放的是jar包程序；
* 还有license和一些Android默认配置文件。


　　我们来看一下`bin`目录下的proguard.bat,用文本编辑器打开，会看到以下内容：
```bat
@ECHO OFF

REM Start-up script for ProGuard -- free class file shrinker, optimizer,
REM obfuscator, and preverifier for Java bytecode.
REM
REM Note: when passing file names containing spaces to this script,
REM       you'll have to add escaped quotes around them, e.g.
REM       "\"C:/My Directory/My File.txt\""

IF EXIST "%PROGUARD_HOME%" GOTO home
SET PROGUARD_HOME=..
:home

java -jar "%PROGUARD_HOME%"\lib\proguard.jar %*
```
　　前面的内容在这里都不重要，可以忽略，直接看到最后一句
```bat
java -jar "%PROGUARD_HOME%"\lib\proguard.jar %*
```
　　这里是运行`lib`中`proguard.jar`的程序，这里`%*`表示传入的命令行参数。在lib目录下，我们也可以命令行中输入类似下面的
形式运行程序：
```bat
java -jar proguard.jar 命令行参数
```
　　例如输入配置文件作为参数(myconfig.pro是配置文件)：
```bat
java -jar proguard.jar @myconfig.pro
```
　　又如输入配置文件和指令选项组合作为参数：
```bat
java -jar proguard.jar @myconfig.pro -verbose
```
　　到了这里，我们可以发现，配置文件里的指令选项就是作为运行参数来执行proguard.jar。

    PS:这里使用的Android SDK为25.2.5；这些默认配置文件会在下面用到。

#### 1.2 Gradle ProGuard配置
　　在Android Studio中，官方默认使用的混淆工具是ProGuard。而在这篇文章里所讲的ProGuard配置也是以Android Studio中的配置
作为例子，如果你使用Eclipse+ADT的方式进行开发，可能不尽相同。

　　在Android Studio中，使用构建工具是Gradle，要启动ProGuard,可以在`Module`的`build.gradle`文件中设置
```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
```
　　这里的`minifyEnabled true`启用ProGuard，`proguardFiles`指定ProGuard配置文件，`proguard-rules.pro`是
`Module`下的自定义配置文件；`getDefaultProguardFile()`是一个方法,获取配置文件，传入的参数`proguard-android.txt`就是
前面提到的Android SDK中`tools/proguard`文件夹下的默认配置文件之一，如果想使用开启优化的配置文件，可以传入另一个
文件名`proguard-android-optimize.txt`，这个文件比前面的文件多了三句：
```ProGuard
# 不对算术和类型转化操作符优化，不改变类层次
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*

# 优化次数
-optimizationpasses 5

# 允许提升访问修饰符，扩大可访问范围
-allowaccessmodification
```
    PS：由于ProGuard处理代码会占用一些时间，不建议在调试时开启；但是最终发布测试时，要同时启用ProGuard来进行测试,避免
    发布时引起错误。

　　`proguard-android.txt`的默认配置文件的内容：
```ProGuard
# 不使用大小写混合的类名
-dontusemixedcaseclassnames

# 不忽略类库中非public的类
-dontskipnonpubliclibraryclasses

# 打印处理代码的信息
-verbose

# 关闭优化
-dontoptimize

# 关闭预校验
-dontpreverify

# 保持Annotations属性
-keepattributes *Annotation*

# 保持ILicensingService这个公共类
-keep public class com.google.vending.licensing.ILicensingService

# 保持ILicensingService这个公共类
-keep public class com.android.vending.licensing.ILicensingService

# 所有的native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保持继承自View类的set()、get()方法
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# 保持继承自Activity类，参数是view的方法
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# 保持枚举类values()、valueOf()方法
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保持实现Serializable的可序列化类字段CREATOR
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

# 保持R（资源）下的所有类的静态字段
-keepclassmembers class **.R$* {
    public static <fields>;
}

# 不打印处理support包的警告信息
-dontwarn android.support.**

# 保持Keep类
-keep class android.support.annotation.Keep

# 保持使用@Keep注解的类及其所有成员
-keep @android.support.annotation.Keep class * {*;}

# 保持使用@Keep注解的方法及其类
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}

# 保持@Keep注解的字段及其类
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

# 保持使用@Keep注解的构造方法及其类
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}

```
    PS：由于以上内容都默认配置好了，所以在我们自己的配置文件中无需重复配置以上选项。
　　如果需要添加多渠道，可以在相应`productFlavor`配置`proguardFiles`,例子如下：
```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                   'proguard-rules.pro'
        }
    }
    productFlavors {
        flavor1 {
        }
        flavor2 {
            proguardFile 'flavor2-rules.pro'
        }
    }
    ...
}
```
　　每次构建过程中都会在`<module-name>/build/outputs/mapping/release/`目录下产生下列文件：
* dump.txt：    说明APK中所有类文件的内部结构；
* mapping.txt： 映射文件，包含原始->混淆过的类、方法和字段名称之间的映射关系；
* seeds.txt：   列出未进行混淆的类和成员；
* usage.txt：   列出从APK移除的代码。


    PS:mapping.txt会在以后的ProGuard堆栈追踪中使用，对混淆过的代码打印可读的崩溃堆栈信息。

#### 1.3 小结
　　
    在这一节，我们学习了Android ProGuard的基本内容，包括SDK工具和Gradle配置，还有输出文件的作用。这些基本默认设置，
不需要怎么改动，主要工作还是自定义我们的配置文件。

    PS：这一部分内容可以查看Android ProGuard使用指南：
    https://developer.android.google.cn/studio/build/shrink-code.html#shrink-resources
   [使用指南](https://developer.android.google.cn/studio/build/shrink-code.html#shrink-resources)

### 2 自定义配置保持代码

　　有时候ProGuard移除或者混淆了一些代码就会导致错误，在前面一节提到使用Keep选项可以让代码不被移除或者不被混淆，为了简单
起见，我们可以直接使用-keep来保持需要的代码，例如：
```ProGuard
-keep public class MyClass
```
　　但是，有时候为了更好地处理代码，就要根据情况进行适当优化，例如：
* Bean类
　　

　　很多时候Bean类都要保持，因为可能用到了自动映射的框架，大多数可能会全部保持,
```ProGuard
-kepp class com.your_package_name.bean.** { *; }
```
　　但是类名大多可以混淆，只保持其成员，
```ProGuard
-keepclassmembers class com.your_package_name.bean.** { *; }
```
* 依赖库类

　　第三方框架很多时候提供的ProGuard配置都很简单粗暴，直接全部保持，导致代码十分臃肿，然而很多时候，我们只是用到了其中
一部分功能，其他可以移除，减少程序体积。

　　除在配置文件保持之外，还可以在类、方法或字段上使用@Keep注解来保持代码。
　　

    PS：只有加上android.support.annotation库依赖，才可以使用@Keep注解。

#### 2.1 一般需保持的情况

　　对于Android配置ProGuard时产生的错误来看，主要有以下几种情况需要注意保持代码：
* 枚举类
* 自定义控件
* 序列化类
* 实体类
* 内部类
* JNI接口
* WebView
* 引用的类只来自AndroidManifest.xml
* 反射

　　在这里，着重来讨论反射的情况。由于反射是在运行时通过读取名称来动态创建和调用类或者类成员，虽然ProGuard已经对一些反射
方法进行检查，不会移除反射中动态创建和调用的代码，不过在混淆阶段，由于名称被混淆，就会找不到对应的名称而报错，所以有必要
保持使用反射的类或类成员。

　　ProGuard会自动检查以下反射方法：
```java
Class.forName("SomeClass")
SomeClass.class
SomeClass.class.getField("someField")
SomeClass.class.getDeclaredField("someField")
SomeClass.class.getMethod("someMethod", new Class[] {})
SomeClass.class.getMethod("someMethod", new Class[] { A.class })
SomeClass.class.getMethod("someMethod", new Class[] { A.class, B.class })
SomeClass.class.getDeclaredMethod("someMethod", new Class[] {})
SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class })
SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class, B.class })
AtomicIntegerFieldUpdater.newUpdater(SomeClass.class, "someField")
AtomicLongFieldUpdater.newUpdater(SomeClass.class, "someField")
AtomicReferenceFieldUpdater.newUpdater(SomeClass.class, SomeType.class, "someField")
```

    PS:对于上述反射方法，ProGuard会在处理时给出保持建议，然后可以添加相应的配置选项到配置文件中。

#### 2.2 小结
　　
    在这一节，给出了一些需要注意的配置情况，不过并没有给出相应的配置，这里需要按需配置，因为有时候这些情况并不会产生问题。

     PS：这一部分内容可以查看ProGuard配置例子：
     https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/examples.html
   [配置例子](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/examples.html)

[ProGuard Tools]:https://github.com/wangtotang/Awesome-Sources/blob/master/pictures/ProGuard_Tools.jpg