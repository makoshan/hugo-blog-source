---
categories:
- Dev
date: 2015-10-25T22:03:13+08:00
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
```java
//原型
op(Path path, Path.Op op)
//eg
path1.op(path2,Path.Op.DIFFERENCE);
```
此方法用于对两个Path对象做相应的运算组合(combine),具体的说是根据不同的`op`参数及path2参数来影响path1对象，有点类似于数学上的集合运算。请看下面的例子：  
```java
Path path1 = new Path();
path1.addCircle(150, 150, 100, Path.Direction.CW);
Path path2 = new Path();
path2.addCircle(200, 200, 100, Path.Direction.CW);
path1.op(path2, Path.Op.DIFFERENCE);
canvas.drawPath(path1, paint1);
```
效果如下:	   
![](http://77g5pl.com1.z0.glb.clouddn.com/imgop-path-difference.png-nor)  
通过不断修改path1.op的第二个参数依次可以得到如下效果:  

`Path.Op.INTERSECT`效果：  
![](http://77g5pl.com1.z0.glb.clouddn.com/imgop-path-intersect.png-nor)  
`Path.Op.UNION`效果：  
![](http://77g5pl.com1.z0.glb.clouddn.com/imgop-path-union.png-nor)  
`Path.Op.REVERSE_DIFFERENCE`效果：   
![](http://77g5pl.com1.z0.glb.clouddn.com/imgop-path-reverse_difference.png-nor)  
`Path.Op.XOR`效果：   
![](http://77g5pl.com1.z0.glb.clouddn.com/imgop-path-xor.png-nor)  

##### 总结:

1. Path.Op.DIFFERENCE 减去path1中path1与path2都存在的部分; 	
	**path1 = (path1 - path1 ∩ path2)**
2. Path.Op.INTERSECT 保留path1与path2共同的部分;   
	**path1 = path1 ∩ path2**
3. Path.Op.UNION 取path1与path2的并集; 	
	**path1 = path1 ∪ path2**
4. Path.Op.REVERSE_DIFFERENCE 与DIFFERENCE刚好相反; 	
	**path1 = path2 - (path1 ∩ path2)**
5. Path.Op.XOR 与INTERSECT刚好相反; 	
	**path1 = (path1 ∪ path2) - (path1 ∩ path2)**

### 4.setFillType
设置path的填充模式.网上关于path的FillType的介绍很少，实际上在官方`ApiDemos`里就有个很好的例子:  
```java
/**
 * Created by ghui on 10/25/15.
 */
public class PathFillTypeView extends View {
	private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
	private Path mPath;

	public PathFillTypeView(Context context) {
		super(context);
		setFocusable(true);
		setFocusableInTouchMode(true);

		mPath = new Path();
		mPath.addCircle(40, 40, 45, Path.Direction.CCW);
		mPath.addCircle(80, 80, 45, Path.Direction.CCW);
		mPath.addCircle(120, 120, 45, Path.Direction.CCW);
	}

	private void showPath(Canvas canvas, int x, int y, Path.FillType ft,
						  Paint paint) {
		canvas.save();
		canvas.translate(x, y);
		canvas.clipRect(0, 0, 160, 160);
		canvas.drawColor(Color.WHITE);
		mPath.setFillType(ft);
		canvas.drawPath(mPath, paint);
		canvas.restore();
	}

	@Override
	protected void onDraw(Canvas canvas) {
		Paint paint = mPaint;
		paint.setColor(Color.RED);
		canvas.drawColor(0xFFCCCCCC);
		canvas.translate(20, 20);
		paint.setAntiAlias(true);
		showPath(canvas, 0, 0, Path.FillType.WINDING, paint);
		showPath(canvas, 160 * 2, 0, Path.FillType.EVEN_ODD, paint);
		showPath(canvas, 0, 160 * 2, Path.FillType.INVERSE_WINDING, paint);
		showPath(canvas, 160 * 2, 160 * 2, Path.FillType.INVERSE_EVEN_ODD, paint);
	}
}
```
效果如下:   
![](http://77g5pl.com1.z0.glb.clouddn.com/imgpath-filltypes.png)   
(上面的例子在官方ApiDemo的基础上做了适当的修改)

#### 总结:
所谓填充指的就是填充内部，`setFillType`就是用来界定哪里算内部的算法。在计算机图形学中界定一个点是不是在多边形内部有两种算法:

1. 非零环绕数规则([Nonzero-rule](https://en.wikipedia.org/wiki/Nonzero-rule))
2. 奇偶规则([Even–odd rule](https://en.wikipedia.org/wiki/Even%E2%80%93odd_rule))

关于这两种算法这里不作详细介绍，具体可以参考上面的维基链接，或者[这篇](http://blog.csdn.net/freshforiphone/article/details/8273023)中文资料(注意看评论区)

### 5.易混淆的方法

#### 1. addArc 与 arcTo	
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

若将代码2中的arcTo方法的参数修改为true则绘制的效果与代码1相同。

### 2. reset 与 rewind
`reset`清除path上的内容，重置path到 path = new Path()的初始状态。  
`rewind`清除path上的内容，但会保留path上相关的数据结构，以高效的复用。  
[Detail](http://stackoverflow.com/questions/12530684/android-draw-path)

### 其它方法
1. moveTo(float x,float y)	
	移动画笔到 (x,y) 处
2. offset(float dx, float dy)   
	平移当前path,在此path上绘制的任何图形都会受到影响
3. close()   
	闭合当前路径 (系统会自动从起点到终点绘制一条直线，使当前路径闭合)
4. reset()	
	重置path,但不会重置`fill-type`设置 
5. rewind()	   
	重置path,但会保留内部数据结构
6. set(Path src) 	
	设置新的Path到当前对象
7. setLastPoint(float x,float y)	
	设置当前path的终点
8. transform(Matrix matrix)  
	矩阵变换







