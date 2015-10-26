---
title: "如何通过Gradle实现一套代码开发不同特性的APK"
date: "2015-03-10T19:01:21+08:00"
categories:
- Dev
tags:
- android

---
Android tools团队于去年底最终发布了Android Studio1.0正式版及gradle plugin for android 1.0正式版，然后业余时间就研究了一下Gradle,前段时间也在公司内部做了一个相关分享，感觉gradle带来的最大便利就是通过
*Product Flavor*实现在一个工程中开发不同特性的apk，以及更方便的依赖管理，下面通过一个小demo来演示这些:<!--more-->   
这个demo对应两个版本免费版及收费版。主要区别在于免费版中使用的是普通的loading圈，收费版中使用的是google风格的progressbar,最终效果如下:   
![](http://77g5pl.com1.z0.glb.clouddn.com/imggradle-demo-free.gif)  ![](http://77g5pl.com1.z0.glb.clouddn.com/imggradle-demo-pay.gif)    

仔细观察上面的视频，你会发现这两个应用除了progressbar的样式不一样之外，它们对应的app-name,app-icon也不同，而且我将它们同时安装到了一台设备上，这就说明它们的包名也不一样。而这些差异我可以通过ProductFlavor及Source sets轻松实现。  

首先，介绍一下*Productflavor*,它是gradle plugin for android中的一个dsl type通过它你可以定义不同的app 变种，这里定义了两种 free 及 pay 。通过Productflavor你可以配置此flavor对应的包名，签名信息，版本名，
版本号等，具体可配置项可以到[这里](http://apdr.qiniudn.com/com.android.build.gradle.internal.dsl.ProductFlavor.html)查看,如果你对Productflavor不了解可以到[这里](http://tools.android.com/tech-docs/new-build-system/build-system-concepts)查看.下面是我的完整的build.gradle 文件:

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    signingConfigs {
        release {
            storeFile file(RELEASE_STORE_FILE)
            storePassword RELEASE_STORE_PASSWORD
            keyAlias RELEASE_KEY_ALIAS
            keyPassword RELEASE_KEY_PASSWORD
        }
    }
    defaultConfig {
        applicationId "me.ghui.gradledemo"
        minSdkVersion 15
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            zipAlignEnabled false
        }
    }
    productFlavors {
        pay {
            applicationId "me.ghui.gradledemo.pay"
        }
        free {}
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.2'
    compile 'com.jpardogo.googleprogressbar:library:+'
    compile 'com.mcxiaoke.volley:library:1.0.+'
    compile 'com.jakewharton:butterknife:6.1.0'
}
```
我对pay flavor中的包名做了修改，free中没有做单独设置它会使用defaultconfig中定义的包名，这就解释了为什么我可以同时将这两个apk装到手机上。  
那么如何实现pay , free 对应不同的特性呢，这里要引入另外一个概念—— "SourceSets"可以到[这里](http://tools.android.com/tech-docs/new-build-system/build-system-concepts)了解，每个flavor都可以对应一个Sourceset,你可以在与`src/main`同级的目录下新建另外两个目录pay,free这里面可以放各自特性相关的代码，然后这些代码会与main下面的代码按照一定的规则进行合并，最终组成当前variant对应的完整代码。main,pay,free就是3个Sourceset 如下图:

![](http://77g5pl.com1.z0.glb.clouddn.com/imggradledemo-stru.jpg-nor)

你最终能编译出的apk的数量是由productflavor与BuildType决定的，虽然我这里没有定义Buildtype但gradle默认提供了两种buildType —— `debug`,`release`，所以，一共可以编译出4种apk,你可以到Android Studio 的BuildVariants面板中查看如下图:  
![](http://77g5pl.com1.z0.glb.clouddn.com/imggradle-demo-3.png)  
实际上在你上图中的面板中切换variant时，Android视图下对应的代码是会发生变化的，这里显示出的代码就是你当前variant所拣取的代码，可以看到这里我选择编译`paydebug`上面的代码就由main+pay下面的代码组成了

在pay下面的ImgViewer中我使用了googleprogressbar而在free下的ImgViewer中我使用了普通的Progressbar,代码如下：  

```java

    GoogleProgressBar mProgressBar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_img_viewer);
        mProgressBar = (GoogleProgressBar) findViewById(R.id.google_progress);
        mProgressBar.setIndeterminateDrawable(getRandomProgressDrawable());
        onLoadImg();
    }
```
当然了pay与free下面对应的Imgviewer的布局文件也是不同的在各自的下面都有一个activity_img_viewer.xml
pay下面是:  
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:paddingLeft="@dimen/activity_horizontal_margin"
                android:paddingRight="@dimen/activity_horizontal_margin"
                android:paddingTop="@dimen/activity_vertical_margin"
                android:paddingBottom="@dimen/activity_vertical_margin"
                tools:context="me.ghui.gradledemo.ImgViewer">
    <ImageView
            android:id="@+id/networkImageView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:src="@drawable/waitting"
            android:layout_centerHorizontal="true" />
    <com.jpardogo.android.googleprogressbar.library.GoogleProgressBar
            android:id="@+id/google_progress"
            android:layout_centerInParent="true"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_gravity="center"
            />
</RelativeLayout>
```

free下面就是普通的progressbar,这里就不再贴了。
至于为什么app-name与app-icon不同那是因为我在pay ,free下面都各自添加了不同的app-name 与app-icon,在编译对应variant时gradle会去做选取。实际上buildtype也可以对应自己的sourceset,也可以有自己的代码。

上面说到了合并规则，下面总结一下：  

图片、音频、 XML 类型的 Drawable 等资源文件，将会进行文件级的覆盖
字符串、颜色值、整型等资源以及 AndroidManifest.xml ，将会进行元素级的覆盖
代码资源，同一个类， buildTypes 、 productFlavors 、 main 中只能存在一次，否则会有类重复的错误
覆盖等级为：buildTypes > productFlavors > main


如果你还有不懂可以到[这里](https://github.com/ghuiii/gradledemo)将代码clone下来自己跑一下就明白了。

最后，说一下在gradle中新的依赖管理方式：  

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.2'
    compile 'com.jpardogo.googleprogressbar:library:+'
    compile 'com.mcxiaoke.volley:library:1.0.+'
    compile 'com.jakewharton:butterknife:6.1.0'
}
```
上面我分别依赖了support包，googleprogressbar开源库，volley,butterknife，我只需添加以上几行compile命令，gradle 就会去jcenter仓库帮我完成依赖的下载、添加到classpath等一系统操作,而且这些依赖会在gradld第一次编译时缓存到~/.gradle/caches下面，以后再编译就直接去引用了，而且还有最重要的一点，通过这种方式你的工程与依赖就完全分开了，这些依赖也不会再混入到你的git仓库中了，它们之间的耦合性也降到了最低，以后如果不再需要某个依赖可以直接删除对应的compile即可

