##[Android源码编译](https://source.android.com/source/)

#### 1.建立构建环境  

 (1)[安装Ubuntu 14.04](http://www.jianshu.com/p/2eebd6ad284d)
 
 (2)安装JDK7  

    $ sudo apt-get install openjdk-7-jdk

 (3)64位Ubuntut安装所需的包 

    $ sudo apt-get install git-core gnupg flex bison gperf build-essential \
      zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
      lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
      libgl1-mesa-dev libxml2-utils xsltproc unzip

(4)优化构建环境，设置ccache

    $ export USE_CCACHE = 1
    $ export CCACHE_DIR = /<path_of_your_choice>/.ccache
    $ prebuilts/misc/linux-x86/ccache/ccache -M 50G

#### 2.下载源码

  (1)建源码目录

    $ mkdir source
    $ cd source
  (2) [在中国大陆下载源码](https://www.zhihu.com/question/20738613/answer/86302824)
  
#### 3.编译源码

  (1)初始化 

    $ source build/envsetup.sh

  (2)选择目标

    $ lunch sdk_x86-eng

  (3)编译(CPU最大支持4线程，设置-j4)

    $ make -j4 sdk