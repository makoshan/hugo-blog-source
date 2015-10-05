---
categories:
- Dev
date: 2015-10-05T01:32:40+08:00
tags:
- android
title: View的MeasureSpec确定过程
draft: true
---
我们在自定义View时通常会去重写View的`onMeasure`方法，此方法提供的有一个默认实现：

``` java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```
这个方法有两个int型的参数：`widthMeasureSpec`,`heightMeasureSpec`。这篇文章将会分析它们的确定过程。首先看一下`MeasureSpec`是什么？

### MeasureSpec
MeasureSpec是View类的一个静态内部类，源码不太多，这里贴出分析之：

```java
public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }

        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

        static int adjust(int measureSpec, int delta) {
            final int mode = getMode(measureSpec);
            if (mode == UNSPECIFIED) {
                // No need to adjust size for UNSPECIFIED mode.
                return makeMeasureSpec(0, UNSPECIFIED);
            }
            int size = getSize(measureSpec) + delta;
            if (size < 0) {
                Log.e(VIEW_LOG_TAG,
                 "MeasureSpec.adjust: new size would be negative! (" + size +") spec: " 
                 + toString(measureSpec) + " delta: " + delta);
                size = 0;
            }
            return makeMeasureSpec(size, mode);
        }

        public static String toString(int measureSpec) {
            int mode = getMode(measureSpec);
            int size = getSize(measureSpec);

            StringBuilder sb = new StringBuilder("MeasureSpec: ");

            if (mode == UNSPECIFIED)
                sb.append("UNSPECIFIED ");
            else if (mode == EXACTLY)
                sb.append("EXACTLY ");
            else if (mode == AT_MOST)
                sb.append("AT_MOST ");
            else
                sb.append(mode).append(" ");

            sb.append(size);
            return sb.toString();
        }
    }
```
widthMeasureSpec 与 heightMeasureSpec是一个int型的变量，java中int型变量由4个字节(32bit)组成，其中高2位用来封装MeasureMode,MeasureMode一共有3种:

1. UNSPECIFIED = 0 << MODE_SHIFT; 即: **00**`000000 00000000 00000000 00000000` 父容器不对子View有任何限制
2. EXACTLY     = 1 << MODE_SHIFT; 即: **01**`000000 00000000 00000000 00000000` 父容器已经测量出子View所需要的大小,即measureSpec中封装的specsize
3. AT_MOST     = 2 << MODE_SHIFT; 即: **10**`000000 00000000 00000000 00000000` 父窗口限定了一个最大值给子View即specsize

低30位用来封装size.

1. public static int makeMeasureSpec(int size, int mode) 此方法用来根据size与mode组合出特定的measureSpec.可以看到在api17之后是通过二进制 & | 计算的，这样效率更高。
2. static int adjust(int measureSpec, int delta) 此方法是用来对measurespec中的size进行调整的 和 `<inset>`相关。
3. public static String toString(int measureSpec) 此方法大概是用来方便开发者从一个int值中打印mode与size，方便调试。

MeasureSpec大概就是这样，为了说明MeasureSpec的创建过程，我们写一个CustomLinearLayout里面套一个CustomView,通过debug来分析一下，布局代码大致如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<me.ghui.customviewgroup.CustomLinearLayout
		xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:orientation="vertical">
	<me.ghui.customviewgroup.CustomView android:layout_width="match_parent"
		android:layout_height="200px"
		android:background="@android:color/holo_blue_light"/>
</me.ghui.customviewgroup.CustomLinearLayout>
```
然后我们给CustomLinearLayout,CustomView的onMeasure方法加入一些log,如下：

```	
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	Log.e("ghui", "CustomLinearLayout." + "widthMeasureSpec:" 
	+ MeasureSpec.toString(widthMeasureSpec)
	+ ", heightMeasureSpec: " + MeasureSpec.toString(heightMeasureSpec));
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	Log.e("ghui", "CustomView." + "widthMeasureSpec:" 
	+ MeasureSpec.toString(widthMeasureSpec)
	+ ", heightMeasureSpec: " + MeasureSpec.toString(heightMeasureSpec));
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	}
```
我们在CustomView的onMeasure方法中加入一个断点，通过android studio观察onMeasure方法的调用栈信息，截图如下：
![](http://77g5pl.com1.z0.glb.clouddn.com/imgmeasure_process.png-nor)
从上面可以发现CustomView的onMeasure方法调用过程源起其Parent CustomLinearLayout中的onMeasure.在这条调用链上最后一个ViewGroup的方法是`measureChildWithMargins()`,源码如下:
```
protected void measureChildWithMargins(View child,
    int parentWidthMeasureSpec, int widthUsed,
    int parentHeightMeasureSpec, int heightUsed) {

    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
可以看到此方法的最后一行中会调用child.measure方法将计算好的childWidthMeasureSpec 与 childHeightMeasureSpec传给子View,再往上看会发现这两个参数的计算过程被完全封装到了
`getChildMeasureSpec`中，此方法的源码如下:
```
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);

    int size = Math.max(0, specSize - padding);

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```
从中可以看出子View的MeasureSpec的计算过程被分为了3部分：

1. parentMeasureMode = EXACTLY

	**a.** childDimension >=0 即子View设置的有具体的大小(如：100px)，这种情况下childSpecSize即为其自身LayoutParams里指定的大小，childSpecMode为EXACTLY

	**b.** childDimension = MATCH_PARENT 即子View的LayoutParams里设置的为`MATCH_PARENT`,这种情况下childSpecSize即为parentLeftSize(父容器剩余的空间)，childSpecMode为`EXACTLY`

	**c.** childDimension = WRAP_CONTENT 即子View的LayoutParams里设置的为`WRAP_CONTENT`,这种情况下childSpecSize为parentLeftSize(父容器剩余的空间),childSpecMode为`AT_MOST`

2. parentMeasureMode = AT_MOST

	**a.** childDimension >=0 即子View设置的有具体的大小(如：100px)，这种情况下childSpecSize即为其自身LayoutParams里指定的大小，childSpecMode为EXACTLY (同 1.a)

	**b.** childDimension = MATCH_PARENT 即子View的LayoutParams里设置的为`MATCH_PARENT`,这种情况下childSpecSize即为parentLeftSize(父容器剩余的空间)，childSpecMode为`AT_MOST` (同 1.c)

	**c.** childDimension = WRAP_CONTENT 即子View的LayoutParams里设置的为`WRAP_CONTENT`,这种情况下childSpecSize即为parentLeftSize(父容器剩余的空间),childSpecMode为`AT_MOST`(同1.c)

3. parentMeasureMode = UNSPECIFIED

	**a.** childDimension >=0 即子View设置的有具体的大小(如：100px)，这种情况下childSpecSize即为其自身LayoutParams里指定的大小，childSpecMode为EXACTLY (同 1.a)

	**b.** childDimension = MATCH_PARENT 即子View的LayoutParams里设置的为`MATCH_PARENT`,这种情况下childSpecSize为0，childSpecMode为`UNSPECIFIED`

	**c.** childDimension = WRAP_CONTENT 即子View的LayoutParams里设置的为`WRAP_CONTENT`,这种情况下childSpecSize为0，childSpecMode为`UNSPECIFIED`(同3.b)

整理一下可以得到如下表格:
![](http://77g5pl.com1.z0.glb.clouddn.com/imgQQ20151005-0@2x.png)
### 总结

1. View的`measureSpec`由其父容器的measureSpec及自身的LayoutParams共同决定
2. 若子View为具体的大小(如100px),则不管其父容器的specMode为哪种，子View对应的specMode均为`EXACTLY`,specSize均为`childSize`
3. 若子View的LayoutParams中的宽或高为`wrap_content`,则不管其父容器的specMode为哪种，子View对应的specMode均为`AT_MOST`,specSize均为`parentLeftSize`
4. 子View的前期measure过程实际上在其父容器的onMeasure中就基本完成了，父容器会把计算好的measureSpec传递给子View，子View在自己的onMeasure中可以得到这些值，子View可以根据这些值设置自己的大小，当然也可以不参考它们。最终在onMeasure方法通过调用
`setMeasuredDimension`方法设置view的最终measureSize。但View的真实大小是在Layout阶段才确定下来的，通过`child.layout(left,top,right,bottom)`.View的measure size与 layout size不必相等，但绝大多数情况下是相等的。








