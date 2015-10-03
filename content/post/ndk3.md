---
title: "ndk学习笔记3"
date: "2013-10-03T19:01:21+08:00"
categories:
- Dev
tags:
- android
---
1. Java传递数据给Native Code
---
在`ndk学习笔记1`中我们声明了一个无参的sayHi方法。当然，我们也可声明有参数的方法。这样就可以让Java传递数据给c代码。还是基于`hello-ndk`那个例子。 <!--more-->
我们在`NativeInterface`类中增加另外两个方法:   

	public class NativeInterface {
    	public native String sayHi();
    	public static native int add(int x, int y);
    	public native String appendStr(String s);
	}
其中一个是静态的add用来实现两个数求和，另一个给指定字符串附加 hello 子串。  
然后通过`javah`重新生成.h头文件,接着在c文件中增加新增的这两个函数的实现，如下:  

	JNIEXPORT jint JNICALL Java_com_example_hello_1ndk_NativeInterface_add
	  (JNIEnv * env, jclass clazz, jint x, jint y){
		LOGD("x=%d",x);
		LOGD("y=%d",y);
		return x+y;
	  }

	JNIEXPORT jstring JNICALL Java_com_example_hello_1ndk_NativeInterface_appendStr
	  (JNIEnv * env, jobject obj, jstring jstr){
		//在c语言中 是没有java的String
		char* cstr = Jstring2CStr(env, jstr);
		LOGD("cstr=%s",cstr);
		// c语言中的字符串 都是以'/0' 作为结尾
		char arr[7]= {' ','h','e','l','l','o','\0'};
		strcat(cstr,arr);
		LOGD("new cstr=%s",cstr);
		return (*env)->NewStringUTF(env,cstr);
	  }
其中，`Jstring2CStr(env, jstr)`是一个自定义方法，用来实现将Java中的string转化为c中的char*.

	char*   Jstring2CStr(JNIEnv*   env,   jstring   jstr)
	{
		 char*   rtn   =   NULL;
		 jclass   clsstring   =   (*env)->FindClass(env,"java/lang/String"); //String
		 jstring   strencode   =   (*env)->NewStringUTF(env,"GB2312");  // 得到一个java字符串 "GB2312"
		 jmethodID   mid   =   (*env)->GetMethodID(env,clsstring,   "getBytes",   "(Ljava/lang/String;)[B"); //[ String.getBytes("gb2312");
		 jbyteArray   barr=   (jbyteArray)(*env)->CallObjectMethod(env,jstr,mid,strencode); // String .getByte("GB2312");
		 jsize   alen   =   (*env)->GetArrayLength(env,barr); // byte数组的长度
		 jbyte*   ba   =   (*env)->GetByteArrayElements(env,barr,JNI_FALSE);
		 if(alen   >   0)
		 {
		  rtn   =   (char*)malloc(alen+1);         //"\0"
		  memcpy(rtn,ba,alen);
		  rtn[alen]=0;
		 }
		 (*env)->ReleaseByteArrayElements(env,barr,ba,0);  //
		 return rtn;
	}
2. C回调Java中的代码
---
有时候在我们的native code中要实现某些功能不太方便，或者完成这个功能的代码在Java下已有现成的。这时候就需要我们在native code 中去回调Java中的代码。  
还是基于`hello-ndk`项目，增加一个`JavaMethods`类内容如下：  

	public class JavaMethods {
		public int add(int x, int y) {
			return x + y;
		}

		public static void printHello(String s) {
			System.out.println("java static"+ s);
		}
	}
我们假设上面的两个方法用native code去实现不太方便，而且在java下已有现成的。我们需要在native code 中适当的地方回调它们。  我们在`NativeInterface`中增加
两个native method 去回调它们，如下：  

	public class NativeInterface {
		public native String sayHi();
		public static native int add(int x, int y);
		public native String appendStr(String s);

	   public native int callMethodInJava_add();
	   public static native void callMethodInJava_printHello();
	}

调用`javah`重新生成头文件,并在hello-ndk.c中增加相关实现具体代码如下：  

	 JNIEXPORT jint JNICALL Java_com_example_hello_1ndk_NativeInterface_callMethodInJava_1add
	  (JNIEnv *env, jobject obj){
		// java 反射
		//1 . 找到java代码的 class文件
		//    jclass      (*FindClass)(JNIEnv*, const char*);
		jclass clazz = (*env)->FindClass(env,"com/example/hello_ndk/JavaMethods");
		if(clazz==0){
			LOGI("find class error");
			return;
		}
		LOGI("find class ");

		//2 寻找class里面的方法
		//   jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);
		jmethodID method= (*env)->GetMethodID(env,clazz,"add","(II)I");
		if(method==0){
			LOGI("find method error");
			return;
		}
		LOGI("find method");
		// 3 调用这个方法
		//    jint        (*CallIntMethod)(JNIEnv*, jobject, jmethodID, ...);
		jobject jmobj= (*env)->AllocObject(env,clazz);
		int result = (*env)->CallIntMethod(env,jmobj,method,3,5);
		LOGI("c code  RESULT = %d",result);
		return result;
	  }


	 JNIEXPORT void JNICALL Java_com_example_hello_1ndk_NativeInterface_callMethodInJava_1printHello
	  (JNIEnv *env, jclass cla){
		  //1 . 找到java代码的 class文件
			//    jclass      (*FindClass)(JNIEnv*, const char*);
			jclass clazz = (*env)->FindClass(env,"com/example/hello_ndk/JavaMethods");
			if(clazz==0){
				LOGI("find class error");
				return;
			}
			LOGI("find class ");

			//2 寻找class里面的方法
			//   jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);
			// 注意 :如果要寻找的方法是静态的方法 那就不能直接去获取methodid
			//jmethodID method = (*env)->GetMethodID(env,clazz,"printStaticStr","(Ljava/lang/String;)V");
			//    jmethodID   (*GetStaticMethodID)(JNIEnv*, jclass, const char*, const char*);
			jmethodID method= (*env)->GetStaticMethodID(env,clazz,"printHello","(Ljava/lang/String;)V");
			if(method==0){
				LOGI("find method error");
				return;
			}
			LOGI("find method ");

			//3.调用一个静态的java方法
			//    void        (*CallStaticVoidMethod)(JNIEnv*, jclass, jmethodID, ...);
			(*env)->CallStaticVoidMethod(env,clazz,method ,(*env)->NewStringUTF(env," haha in c"));
	  }
上面两个方法的实现过程是类似的并且与Java中的反射基本相当：

1. 通过 FindClass方法找到方法所在类，注意此方法的第二个参数，中间用`/`分割。
2. 通过与方法对应的getMethodID或者getStaticMethodID方法找到最终要调用的方法  
> 注意：此方法的最后两个参数分别对应要调用的方法名及其签名，可通过`javap`命令生成，使用方法与`javah`命令基本相同，不过要指定`-s`参数
> 工作路径与`javah`相同，都是在`bin/classes`或`out/prduction/hello-ndk/`下，如: `javap -s com.example.hello-ndk.Javamethods`

3. 最后,根据方法的类型选择合适的调用方法，具体请参考ndk安装目录下的`jni.h`  
> 注意：上面的两个方法，第2个参数要么是`jobject`要么是`jclass`。如果你的方法是静态的就为`jclass`否则为`jobject`，它们代表的是:自己所在的类，或其对象
> 比如: 


	JNIEXPORT jint JNICALL Java_com_example_hello_1ndk_NativeInterface_callMethodInJava_1add
	  (JNIEnv *env, jobject obj)

中的obj代表的是`NativeInterface`的对象。

	 JNIEXPORT void JNICALL Java_com_example_hello_1ndk_NativeInterface_callMethodInJava_1printHello
	  (JNIEnv *env, jclass cla)

中的`cla`代表是:`NativeInterface`类。  
所以，最后我们通过CallXXXMethod方法回调Java中的方法时要注意，当前传递的obj或class对象是否正确。

![](/images/posts/ndk3-javap.png)  
最后在Activity中调用这两个native方法：  

    public void addInJava(View view) {
        Toast.makeText(this,nativeInterface.callMethodInJava_add()+"",Toast.LENGTH_SHORT).show();
    }
    public void printHelloInJava(View view) {
        NativeInterface.callMethodInJava_printHello();
        Toast.makeText(this,"log :print hello in java",Toast.LENGTH_SHORT).show();
    }

执行结果如下:  
![](/images/posts/ndk3-result.png)  
程序效果图:  
![](/images/posts/ndk3-run.png)

