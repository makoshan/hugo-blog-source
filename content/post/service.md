+++
title = "service总结"
date = "2013-11-13T19:01:21+08:00"
+++
##1. service是什么
是Android中的四大组件之一,与Activity类似但是它不提供用户界面.它主要用来提供一些不需要展示的耗时服务(比如,音乐播放,网络下载,信息监听...) <!--more-->
##2. service不是什么
1. 不是一个进程
2. 不是一个线程 
##3. service ,thread,进程
它们三者之间的关系可以简单用下图来表示:
![](http://77g5pl.com1.z0.glb.clouddn.com/imgprocess.png)
1. 默认的Service运行在当前应用的进程中,除非你在`AndroidMannifest.xml`中配置其运行在单独的进程
2. Service中的代码默认情况下是在主线程(UI线程)中执行的,因此如果你想要在Service中执行耗时操作,需要另外开启一个线程

进程很好理解,可以简单的理解为一个程序就是一个进程,下面主要说一下thread与service之间的区别:  
首先,thread是Java平台中的thread(属于Java的技术范畴,当然其它编程语言中也是有这个概念的),它是程序中的最小执行单元,主要用来实现一些异步操作,依存于具体的进程.
而`service`是android的一种系统组件(属于android的技术范畴,个人觉得它有点类似于windows中服务的概念),依存于具体的进程,在service中可以创建thread来实现异步  
操作.虽然thread可以在service中创建,但是它们之间不具有依存关系,service销毁了其创建的thread可以继续运行(前题是不在service中显式销毁之),但是我们在service中创建的线程,我们可以保存其引用,在service中的`onDestroy`方法中显式销毁之.  
为了更好的理解service与thread的区别,我举一个例子:  
假设你要开发一个天气小插件,要实现这样的功能:每隔一段时间去联网更新天气数据,然后将最新的数据更新到小插件中.在这里我们是不需要Activity的,因为我们是直接通过 
widget进行展示的,而且后台定时更新的操作要不需要让用户知道,这里我们就可以用service来实现了.而且联网的操作可以通过在service中开启一个子线程来提高性能.  
想一下这样的功能你可简单的通过几个thread实现吗?直接用thread来实现,你将会发现你不知道要将thread写在哪里,因为没有Activity,Service...你的thread甚至连一个运行的载体都找不到.
##4. 不同启动方式下service的生命周期
service有两种启动方式:  
1. 通过startService方法
2. 通过bindService方法
这两种启动方式下的生命周期如下图所示:    
![](http://77g5pl.com1.z0.glb.clouddn.com/imgservice_lifecycle.png)  
很简单是不,比起Activity来说service的生命周期是简单不少,上面两种方式下的生命周期很简单不在今天的讨论范围内,想这样一个问题,假如,一个service先通过`startServi
ce`方法开启,再通过`bindService`开启;或者先通过`bindService`,再通过`startService`.这两种情况下service的生命周期是怎么样呢?  

经过我的测试得出的答案如下:

1. 同一个service只会被创建一次(onCreate方法只执行一次)；假如你先通过`startService`(或bindService)方法开启了一个Service,然后你再通过`bindService`(或startService)方法去开启同一个(Intent中指定的action相同)Service 那么此时,将不会再执行onCreate方法而会直接执行`onBind`方法,这种情况下的生命周期流程大致如下:  
![](http://77g5pl.com1.z0.glb.clouddn.com/imgservice1.png)
2. 同时使用`startService`与`bindService`方法开启同一个Service(如step1中的情况),这种service仅仅通过`stopService`或`unbindService`是不能关闭的,要同时使用两者
仅仅使用`stopService`方法不会导致任何方法被回调,仅仅使用`unbindService`方法仅会导致`onUnBind`方法被回调而不会导致`onDestroy`方法被回调.
3. 多次执行`startService`方法会导致`onStartCommand`方法的多次回调;多次执行`bindService`方法onBind方法只会被回调一次
4. 多次调用 stopService 的话,service 只会调用一次 onDestroyed方法。
5. 多次执行 unbindService方法的话会导致程序意外停止

