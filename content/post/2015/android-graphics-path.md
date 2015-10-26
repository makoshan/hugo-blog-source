---
categories:
- Dev
date: 2015-10-25T22:03:13+08:00
draft: true
tags:
- android
- graphics
title: android绘图之Path总结
---
Path作为Android中一种相对复杂的绘图方式，官方文档中的有些解释并不是很好理解，这里作一个相对全面一些的总结，供日后查看，也分享给大家，共同进步。
### 1.基本绘图方法

1. addArc(RectF oval, float startAngle, float sweepAngle)   
	绘制弧线,配合Paint的Style可以实现不同的填充效果
2. addCircle(float x, float y, float radius, Path.Direction dir)    
	绘制圆形,其中第`dir`参数用来指定绘制时是顺时针还是逆时针
3. addOval(RectF oval, Path.Direction dir) 		
	绘制椭圆形，其中 `oval`作为椭圆的外切矩形区域
4. addRect(RectF rect, Path.Direction dir) 	
	绘制矩形
5. addRoundRect(RectF rect, float rx, float ry, Path.Direction dir) 	
	绘制圆角矩形
6. lineTo(float x, float y)	  
	绘制直线
7. addPath(Path src) 	
	添加一个新的Path到当前Path
8. arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)	
	与`addArc`方法相似，但也有区别，下文细述。
9. quadTo(float x1, float y1, float x2, float y2) 	
	绘制`二次贝塞尔曲线`,其中 (x1,y1)为控制点，(x2,y2)为终点
10. cubicTo(float x1, float y1, float x2, float y2, float x3, float y3) 	
	绘制`三次贝塞尔曲线`，其中(x1,y1),(x2,y2)为控制点，(x3,y3)为终点

### 2.rXXX方法
上面的lineTo,MoveTo,QuadTo,CubicTo方法都有与之对应的`rXXX`方法：

1. rLineTo(float dx, float dy)
2. rMoveTo(float dx, float dy)
3. rQuadTo(float dx1, float dy1, float dx2, float dy2)
4. rCubicTo(float x1, float y1, float x2, float y2, float x3, float y3)

这些方法与之对应的原方法相比，惟一的区别在于：**r方法是基于当前绘制开始点的offest**,比如当前paint位于 (100,100)处，则使用`rLineTo(100,100)`方法绘制出来的直线是从(100,100)到(200,200)的一条直接，由此可见`rXXX`方法方便用来基于之前的绘制作连续绘制。

### 3.Path.op方法


### 4.易混淆的方法

#### 1.addArc 与 arcTo	
前者指定在某处画一条弧线，仅此而已，不会受当前paint的位置所影响。而arcTo方法有两种形式:

 1. arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)
 2. arcTo(RectF oval, float startAngle, float sweepAngle) 	

 对于第一种形式的方法，若forceMoveTo参数为false，则与第二种形式的方法没区别，绘制成的最终图形会受到落笔点的影响；	  
 若forceMoveTo参数值为true，则绘制效果与`addArc`方法没有区别。   

```java
//代码1
Path path = new Path();
path.moveTo(100, 100);
path.addArc(200, 200, 400, 400, 0, 150);
canvas.drawPath(path, paint);
```
代码1效果如下图:		
![](http://77g5pl.com1.z0.glb.clouddn.com/imgarc-path-1.png)	

```java
//代码2
Path path = new Path();
path.moveTo(100, 100);
path.arcTo(200, 200, 400, 400, 0, 150, false);
canvas.drawPath(path,paint);
```
代码2效果如下图:	
![](http://77g5pl.com1.z0.glb.clouddn.com/imgarc-path-2.png)

若将代码2中的arcTo方法的参数修改为true则绘制的效果如代码1相同。


### 其它方法
1. moveTo(float x,float y)	
	移动画笔到 (x,y) 处
2. offset(float dx, float dy)   
	平移当前path,在此path上绘制的任何图形都会受到影响
3. close()   
	闭合当前路径 (系统会自动从起点到终点绘制一条直线，使当前路径闭合)
4. reset()	
	重置path,但不会重置`fill-type`设置
5. set(Path src) 	
	设置新的Path到当前对象
6. setLastPoint(float x,float y)	
	设置当前path的终点
7. rewind()		
	todo
8. 







