+++
title= "Fibonacci"
date = "2014-06-20T19:01:21+08:00"

+++
前天面试时遇到了`斐波纳契数列`的一道编程题，这里总结一下:<!--more-->
 
下面的方法返回斐波纳契数列(0,1,1,2,3,5,8,13....),即从第二项开始每一项都等于前两项之和.针对这一规律很容易得到如下的算法来计算:  
```java
public static long fib(int n){
	return n<2?n:fib(n-1)+fib(n-2);
}
```
通过递归来做，这是很自然就会想到的解决方案。但是递归存在一个问题效率很低。比如用上面的算法计算fib(10),计算过程如下:  
![](/images/posts/fibonacci.png)

可以看到有很多重复的部分，而且随着n值的增加这种重复计算会承指数级增加，最后导致的时间开销是惊人的，你可以尝试用这种方法计算一下`fib(100)`.

那显示要想提高它的效率只有通过减少重复的计算，如上图所示最左边的fib(8),fib(7),fib(6),这里是已经计算好了fib(8),fib(7),但在上图中它们又被计算了一遍，所以简单的这里可以通过三个变量来记录fibN(fib(n)的值),fibNMinusOne(fib(n-1)的值),fibNMinusTwo(fib(n-2)的值)中间生成的值，这样在再次需要它们时可以不用再重复计算。  
以fib(10)要想计算fib(10)可以从fib(0),fib(1)计算得到fib(2),然后再用fib(1),fib(2)计算得到fib(3)...最终得到fib(10)的值。

```java
public static long fib1(int n){
	long fibN=0;
	if(n<2){
		fibN = n;
	} else{
		long fibNMinusOne = 1;
		long fibNMinusTwo = 0;
		for(int i=2;i<=n;i++){
			fibN = fibNMinusOne + fibNMinusTwo;
			fibNMinusTwo = fibNMinusOne;
			fibNMinusOne = fibN;
		}
	}
	return fibN;
}
```
上面的算法通过简单的三个变量就避免了重复计算,时间复杂度O(n).  
其实还有一种时间复杂度更低的为O(log(n)),通过矩阵运算可以参考文末的链接.
斐波纳契数列还有很多有趣的变换，这里有一个TED上的[演讲](https://www.ted.com/talks/arthur_benjamin_the_magic_of_fibonacci_numbers),有兴趣可以看一下。

参考
---
[3种方法求解斐波那契数列](http://www.cnblogs.com/python27/archive/2011/11/25/2261980.html)
