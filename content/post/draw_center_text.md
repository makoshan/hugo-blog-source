---
categories:
- Dev
date: 2015-10-19T00:10:16+08:00
tags:
- android
title: Draw Center Text(FontMetrics解析)
---
使用`canvas.drawText`方法可以直接在canvas上画text,通常情况下需要文字在水平与垂直方向上均居中显示，水平居中很容易实现。假设我们需要在`rectF`所标识的区域中实现这样的效果，
代码如下：
```java
Paint.Align align = paint.getTextAlign();
float x;
float y;
//x
if (align == Paint.Align.LEFT) {
	x = rectF.centerX() - paint.measureText(text) / 2;
} else if (align == Paint.Align.CENTER) {
	x = rectF.centerX();
} else {
	x = rectF.centerX() + paint.measureText(text) / 2;
}
```
Paint对象可以设置绘制文字开始的锚点，如果`align`为`LEFT`则代表 `canvas.drawText(String text,float x,float y,Paint paint)`中`x,y`为文字最左边的点，
依次类推:若为`CENTER`则相当于直接设置文字在水平方向上的中点,为`RIGHT`则相当于设置文字最右边的点。  
总之，通过上面的代码可以完美实现文字在水平方向上居中显示，下面看一下如何实现在垂直方向上居中显示。
通常情况下会有下面这样的代码:
```java
int textSize = paint.getTextSize();
y = rectF.centerY() + textSize/2f;
```
上面的代码基本上可以实现垂直居中，但如果你仔细观察会发现文字略微有点靠下，似乎是没有居中。实际上文字应该算是居中了，但在视觉上它的确没有居中，要想搞明白其中的原因，需要了解一下`Paint.FontMetrics`类,源码如下：
```java
public static class FontMetrics {
/**
 * The maximum distance above the baseline for the tallest glyph in
 * the font at a given text size.
 */
public float   top;
/**
 * The recommended distance above the baseline for singled spaced text.
 */
public float   ascent;
/**
 * The recommended distance below the baseline for singled spaced text.
 */
public float   descent;
/**
 * The maximum distance below the baseline for the lowest glyph in
 * the font at a given text size.
 */
public float   bottom;
/**
 * The recommended additional space to add between lines of text.
 */
public float   leading;
}
```
通俗的讲

- top是一行文字的上边界
- ascent是文字可视区域的上边界
- descent是文字可视区域的下边界
- bottom是一行文字的下边界
- leading是行与行之间的间距(通常为0,bottom与descent及top与ascent之间的间距足够间隔行行)

为了更好的理解上面的概念请看下面的图：  
![](http://77g5pl.com1.z0.glb.clouddn.com/imgfontmetrics.png)

现在回过头来看一下为什么上面我们垂直居中的代码有问题，从上面的图中可以发现：**文字的可视区域在ascent与descent之间**,而我们上面的代码实际上是将`top`与`bottom`之间的部分居中了，并没有将可视区域居中，实际上通过log可以发现`top`与`ascent`之间的间距大概是`bottom`与`descent`之间间距的5倍，这也就解释了为什么上面的代码在垂直方向上
略微偏下了，原因找到了，解决方法也自然有了.

为了达到文字在视觉上的垂直居中效果，我们需要将文字的可视区域在垂直方面上的中点与rectF的centerY重合,
,就需要计算出`可视区域在垂直方向上的中点与baseline之间的间距deltaY`,使baseline的y值=rectF.centerY + deltaY。这样就能保证文字的可视区域的中点位于rectF的中点了，达到了视觉上的垂直居中。整体代码如下:

```java
public static void drawCenterText(String text, RectF rectF, Canvas canvas, Paint paint) {
	Paint.Align align = paint.getTextAlign();
	float x;
	float y;
	//x
	if (align == Paint.Align.LEFT) {
		x = rectF.centerX() - paint.measureText(text) / 2;
	} else if (align == Paint.Align.CENTER) {
		x = rectF.centerX();
	} else {
		x = rectF.centerX() + paint.measureText(text) / 2;
	}
	//y
	Paint.FontMetrics metrics = paint.getFontMetrics();
	float acent = Math.abs(metrics.ascent);
	float descent = Math.abs(metrics.descent);
	y = rectF.centerY() + (acent - descent) / 2f;
	canvas.drawText(text, x, y, paint);
}
```
抽成了一个方法，可以直接拿来用，这里需要注意一点drawText垂直方向上是基于baseline画的，baseline之上的`ascent`及`top`均为负值，之下的`descent`及`bottom`为正值,baseline的y值为0;




