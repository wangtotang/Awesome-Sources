# Gradle系列 - Gradle基础

    导读：
    　　这是新开的系列，我们来学习一下Gradle构建系统。作为Android Coder,我们应该很了解Gradle是个
    怎样的东西，熟悉它的简单配置过程，可能还有些高手已经能开发Gradle的插件了。但是仍有不少同学对其
    不甚了解，就例如我这个假的Android Coder，所以有必要深入地学习一下，并记录下来供大家参考。
    　　作为系列的第一篇文章，照例从Gradle基础说起。

### 1 Gradle介绍

 　　Gradle是一个类似Ant的项目自动化构建工具，但不使用XML进行配置，而是使用Groovy编写的
 DSL(特定领域语言)作为构建脚本。它广泛应用于Java生态，现在还可以用于C++项目上。在Android开发中，
 Google在Android Studio上采用了Gradle作为默认构建工具。
 　　尽管Gradle使用Groovy编写构建脚本，然而我们并不需要深入学习这门语言，因为Gradle提供了很多
 已经写好的DSL。如果需要扩展功能，就要熟悉Groovy，对于Java开发者来说，Groovy也是很容易入门的。

    PS：使用Gradle的好处：
        ● 类似Ant的构建工具，方便移植Ant工程；
        ● 声明式构建脚本，使用Groovy编写，方便扩展自定义； 　 　　　
        ● 支持Maven和Ivy仓库，多项目构建； 　 　　　
        ● 更强大的依赖管理。 　

### 2 Groovy基础

　　尽管Gradle已经预定义了很多Groovy编写的DSL，我们还是从Groovy开始学习，了解Groovy基础，方便
 编写构建脚本。
 　 　

### 3 Android Gradle