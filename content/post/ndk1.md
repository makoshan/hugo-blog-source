---
title: "ndk学习笔记1"
date: "2013-10-01T19:01:21+08:00"
categories:
- Dev
tags:
- android
---

1. 什么是NDK
---
官方的解释:ndk是一个工具集，通过它你能够用`native code`如c/c++来实现你app的部分功能。`ndk`就是一个`toolset`.如`adb`所扮演的角色。they are all tools.
具体的讲，NDK只是一个`交叉编译`的工具。它可以让我们在基于`x86`的pc上直接编译出能在基于`arm`的android平台上运行的2进制文件。<!--more-->
2. WHY NDK
---
为什么需要`ndk`呢？as we all known,android程序是用java开发的。因此就存在性能的问题.对于某些对实时性要求较高的程序，用java来实现就会遇到困难。再者，
java是运行在虚拟机上的，不适合做驱动相关的开发。最后，还有个原因，c/c++的历史比java要悠久，在c/c++下已经积累了很多成熟的库并且有些在java下很难找到替
代。因此，就需要在java下复用它们。`Why NDK`,总结下来有一下3点：

1. 解决某些应用对高性能的需求
2. 某些底层开发
3. 复用c/c++下的成熟库

3. NDK安装
---
NDK的安装非常简单，到官网下载安装包，解压到一个合适的目录下。配置一下环境变量即可。如下图：

![ndk环境变量配置](http://77g5pl.com1.z0.glb.clouddn.com/imgndk1-env.png)

为了能够在eclipse或IDEA下高亮显示c/c++代码，最好安装各自平台下的相关插件。如果你是在windows平台下开发，还需要安装`cygwin`。
4. JNI
---
JNI全称:Java Native Interface.翻译过来就是`Java本地开发接口`.用来实现Java代码与其它代码(特别是C/C++代码)之间的相互调用。

![JNI](http://77g5pl.com1.z0.glb.clouddn.com/imgndk1-jni.png)

5. NDK 与 JNI
---
JNI属于Java技术的一部分，是一个标准，一种协议。而NDK是一个工具，用来编译在`arm`平台下运行的2进制文件.我们基于`JNI`协议编写好相关代码，然后用`NDK`提供的
命令进行编译。最后再打包到最终的apk包中。

6. Hello NDK
---
基于NDK的android应用程序的一般开发步骤:

1. 创建一个普通的android工程
2. 编写native方法
3. 在工程根目录下创建`jni`目录,并编写c/c++方法
4. 在jni目录下编写`android.mk`文件
5. 利用`ndk-build`生成动态库
6. 通过java静态代码块加载在第5步中生成的库

下面就来写一个`hello world`程序来验证一下：

1. 编写一个名为`hello-ndk`的普通android工程
2. 在其根目录下创建`jni`目录
3. 编写一个普通Java类`NativeInterface`,并在其中声明一个Native方法`sayHi`
4. 利用`javah`命令生成native方法的签名<br>
 若你是用`eclipse`开发请用终端定位到当前工程的`bin`目录下的classes文件夹下<br>
 若你是用`IDEA`开发则定位到当前工程的`out/production/hello-ndk`下<br>
 然后执行：`javah 包名.类名` 如本例：`javah com.example.hello-ndk.Nativeinterface`	<br>
 然后，就会在当前目录下生成一个头文件，如本例的为：`com_example_hello_ndk_NativeInterface.h`<br>
5. 将上一步中生成的头文件复制到工程的jni目录下,并在此目录下建立一个`hello-ndk.c`的c文件。<br>
6. 打开生成的头文件，copy其中的native方法的声明，paste到`hello-ndk.c`中,添加相关实现,并引入第四步中生成的头文件
具体如下示:

		#include <string.h>
		#include "com_example_hello_ndk_NativeInterface.h"
		
		JNIEXPORT jstring JNICALL Java_com_example_hello_1ndk_NativeInterface_sayHi
  		(JNIEnv * env, jobject obj){
     		return (*env)->NewStringUTF(env, "Hello NDK !");
  		}
		
7. 编写`Android.mk`文件并放于`jni`目录下

		LOCAL_PATH := $(call my-dir)
		include $(CLEAR_VARS)
		LOCAL_MODULE    := hello-ndk
		LOCAL_SRC_FILES := hello-ndk.c
		include $(BUILD_SHARED_LIBRARY)
8. 利用ndk提供的`ndk-build`命令编译动态库，编译成功后将会自动添加到当前工程的`libs`目录下。(如果你是在windows下开发,此步需要用到`cygwin`)
9. 在相关类中通过静态代码块加载上一步中生成的动态库`hello-ndk`,在需要使用native方法的地方调用之。
10. 打包运行此android工程，查看效果。

工程目录结构图

![](http://77g5pl.com1.z0.glb.clouddn.com/imgndk1-stru.png)

程序运行效果图

![](http://77g5pl.com1.z0.glb.clouddn.com/imgndk1-run.png)

具体分析请看下一篇
