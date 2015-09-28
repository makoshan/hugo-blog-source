---
title: "JNI总结"
date: "2013-10-06T19:01:21+08:00"
tags:
- java
---
简介
---
JNI全称` Java Native Interface `,中文译为` Java本地编程接口 `. JNI 是Java平台的一大特性，它为编写Java程序提供极大的便利。
它提供了Java程序与用`Native Code`编写的程序进行交互的能力，使它们可以相互调用。JNI 允许程序员在享受Java平台的优势的前提下
而又能够不必放弃之前遗留下来的`Native Code`编写的程序。  <!--more-->
在用到JNI技术的程序中，JNI所处的位置如下图所示:  
![](/images/posts/jni1-role.png)  
从上图中我们可以清楚的看到JNI是Java与Native沟通的通道。我们还可以看到Java虚拟机实现与Native 程序是依赖于具体的主机
环境的,我们知道Java是跨平台的,所谓`compile once run everywhere`。Java的跨平台性是依赖于具体的虚拟机实现的，即Java虚拟机是不
具有跨平台性的,Java 程序(.java)编译后生成的不是可执行的文件而是 `.class`字节码,`.class`可以运行的任何主机环境下的Java虚拟机
上。Java 的跨平台性就是这么实现的。  
上面所说的基本上都是JNI的优点，下面谈一下它的缺点:  

1. 我们知道Java是跨平台的,但如果我们在Java程序中引入`Native code`后，Java 的这一最大的优势就丧失了。因为Java通过JNI调用的
用Native Code生成的库是不具有跨平台性的
2. Java 是类型安全的，而Native code 如c/c++不是类型安全的。因此，这就导致Java程序在调用Native code时要进行额外的安全性检查
基于以上两点:我们在编写JNI程序时应该让Native方法定义在尽可能少的类中，以让native code 与java code 更多的分离开。

Hello JNI 
---
下面基于一个Hello JNI 程序来看一下，如何写JNI程序  
A.编写一个类(HelloWorld.java),声明Native method  
 
	class HelloWorld{
		private native void print();

		public static void main(String[] args){
			new HelloWorld().print();
		}
		static{
			System.loadLibrary("HelloWorld");
		}
	}

B.编译java程序 `javac HelloWorld.java`  
C.生成native code头文件 `javah HelloWorld`  
D.写c实现，根据上一步中生成的`HelloWorld.h`  

	#include<jni.h>
	#include<stdio.h>
	#include"HelloWorld.h"

	JNIEXPORT void JNICALL Java_HelloWorld_print
	  (JNIEnv *env, jobject obj){
			printf("hello jni!\n");
	  }

jni.h头文件是JNI程序必须要导入的，HelloWorld.h是在每三步中生成的，要必须要导入  
E.编译c生成动态库  

	gcc -fPIC -shared -I /usr/local/jdk/include -I /usr/local/jdk/include/linux/ HelloWorld.c -o libHelloWorld.so

参数解释:  
-fPIC :表示编译为位置独立的代码  
-shared :指定生成动态连接库  
-I 指定头文件的地址,上面两个-I 后面的路径分别用来指定jni.h与jni_md.h的位置，这两个头文件是编译动态库需要的  
-o 指定输出文件名字，因为我们要输出动态库所以要按照`libXXX.so`的格式，其中`XXX`代表库的名字，在这里为`HelloWorld`  
F.运行程序  

	java HelloWorld  

一般这时，你会遇到如下的错误:  

	Exception in thread "main" java.lang.UnsatisfiedLinkError: no HelloWorld in java.library.path
		at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1878)
		at java.lang.Runtime.loadLibrary0(Runtime.java:849)
		at java.lang.System.loadLibrary(System.java:1087)
		at HelloWorld.<clinit>(HelloWorld.java:8)

这是因为java虚拟机找不到我们上面生成的`libHelloWorld.so`动态库，这时就需要我们如下来执行:  

	java -Djava.library.path=. HelloWorld

输出:

	hello jni!

这条命令是设置library库的地址为` . `即当前路径下    
下图完整的描述了上面整个进程:  
![](/images/posts/jni1-hello.png)  
(本文参考`the Java Native Interface Programmer’s Guide and Specification`)  
