+++
title = "ndk学习笔记2"
date = "2013-10-02T19:01:21+08:00"
+++
在上一篇中，主要介绍了一下NDK的一些基本概念，最后通过一个hello ndk 程序展示了如何进行实际的ndk开发。在这一篇中，将对上一篇
中涉及到的相关知识做一个简单说明。以及如何调试ndk中的native code。<!--more-->

Android.mk文件
---
下面是上一节中的`hello-ndk`例子中的Android.mk文件的内容。

		LOCAL_PATH := $(call my-dir)
		include $(CLEAR_VARS)
		LOCAL_MODULE    := hello-ndk
		LOCAL_SRC_FILES := hello-ndk.c
		include $(BUILD_SHARED_LIBRARY)

每个Android.mk文件必须以 `LOCAL_PATH := $(call my-dir)`开始，这条代码的作用是：通过编译系统提供的`my-dir`函数获取当前
Android.mk所在的路径,并赋给`LOCAL_PATH`变量.

`include $(CLEAR_VARS)`的作用是：用来清除`LOCAL_XXX`型的变量。如：LOCAL_MODULE, LOCAL_SRC_FILES, LOCAL_STATIC_LIBRARIES等，
但不会包括上面定义的LOCAL_PATH。在定义一个新的MODULE之前都要调用此句代码，以清空前一个MODULE中定义的值。

`LOCAL_MODULE    := hello-ndk`的作用是指定编译后生成的库文件的名字，注意它的命名不能含有空格.并且生成的库文件会由ndk自动
增加前缀`lib`及后缀`.so`代表它是一个动态库。如上例中将会生成名为`libhello-ndk.so`的动态库。<br>
并且有一点需要特别注意,假如你的LOCAL_MODULE命名为:`libhello-ndk`那么生成的库文件还是`libhello-ndk.so`

`LOCAL_SRC_FILES := hello-ndk.c`用来指定本地编写的`.c`或`.cpp`源代码,注意这里不要包含头文件。多个源文件之间以空格分割。比如
：`LOCAL_SRC_FILES := a.c b.c`. c++源代码的扩展名默认为`.cpp`但是你也可以自定义.比如可以通过如下代码将其改为`.cc`

	LOCAL_CPP_EXTENSION=.cc	
	
include $(BUILD_SHARED_LIBRARY)是用来收集`LOCAL_XXX`变量的信息，并决定如何编译动态库。

NDK日志
---
ndk提供的有一个ndk-gdb命令但，似乎不太好用。所以一般我们还是通过打log的方式来调试c代码。
若想在native code中打log需要做如下设置：  
1. 在Android.mk文件中增加 `LOCAL_LDLIBS += -llog`  
2. 在相关c代码中增加
	#include <android/log.h>
	#define LOG_TAG "ndk-log"
	#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__)
	#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
	#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)

日志级别：

	ANDROID_LOG_UNKNOWN,
    ANDROID_LOG_DEFAULT,    
    ANDROID_LOG_VERBOSE,
    ANDROID_LOG_DEBUG,
    ANDROID_LOG_INFO,
    ANDROID_LOG_WARN,
    ANDROID_LOG_ERROR,
    ANDROID_LOG_FATAL,
    ANDROID_LOG_SILENT, 

在需要打log的地方直接调用define的相关函数即可，如： LOGD("DEBUG"),LOGI("str=%s",s);
