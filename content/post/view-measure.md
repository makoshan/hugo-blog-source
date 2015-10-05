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
这个方法有两个int型的参数：`widthMeasureSpec`,`heightMeasureSpec`。这篇文章将会说明它们的确定过程。首先看一下`MeasureSpec`是什么？

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

1. UNSPECIFIED = 0 << MODE_SHIFT; 即: **00**`000000 00000000 00000000 00000000`
2. EXACTLY     = 1 << MODE_SHIFT; 即: **01**`000000 00000000 00000000 00000000`
3. AT_MOST     = 2 << MODE_SHIFT; 即: **10**`000000 00000000 00000000 00000000`

低30位用来封装size.

1. public static int makeMeasureSpec(int size, int mode) 此方法用来根据size与mode组合出特定的measureSpec.可以看到在api17之后是通过二进制 & | 计算的，这样效率更高。
2. static int adjust(int measureSpec, int delta) 此方法是用来对measurespec中的size进行调整的 和 <inset>相关。
3. public static String toString(int measureSpec) 此方法大概是用来方便开发者从一个int值中打印mode与size，方便调试。

MeasureSpec大概就是这样，为了说明MeasureSpec的创建过程，我们写一个CustomLinearLayout里面套一个CustomView,通过debug来分析一下，布局代码大致如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<me.ghui.customviewgroup.CustomLinearLayout
		xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="match_parent">
	<me.ghui.customviewgroup.CustomView 
		android:layout_width="wrap_content"
		android:layout_height="200dp"
		android:background="@android:color/holo_blue_light"/>
</me.ghui.customviewgroup.CustomLinearLayout>

```











