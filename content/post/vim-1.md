+++
title = "Practical Vim读书笔记1"
date = "2013-09-17T19:01:21+08:00"
+++

发现一本好书[Practical Vim](http://pragprog.com/book/dnvim/practical-vim).以后就在这里写个读书笔记。每
篇大概总结一章，一共大概要写20篇，欢迎捧场。
今天总结第一章 *the vim way* <!--more-->
Tips1:初识点号命令
---
	Line one
	Line two
	Line three
	Line four
这里简单说几个用例：

1. 将光标放在第一行行首,然后用`x`删除一个字符，然后再按`.`号，反复按。可以看到`.`可以用来重复执行上次的删除操作
2. 同样将光标放在第一行行首，然后用`dw`删除第一个单词`Line`,接着按`j`跳到下一行，再按`.`，然后反复按`j.`会看到会反复执行
删除首字操作
3. 同样的可以。1. 光标置于行首，2.按`>G` 3.`j.`

Tips2:不要重复劳动
---
	var foo = 1
	var bar = 'a'
	var foobar = foo + bar
请在上面的代码的每行末插入`;`

1. 光标放在第一行光标放在第一行光标放在第一行光标放在第一行
2. press `A`光标跳转到行末并自动切换到Insert Mode
3. 输入`;`后按`ESC`返回到Normal Mode
4. 重复按`j.`

Tips3:磨刀不误砍柴功
---
	var foo = "method("+argument1+","+argument2+")";
请在上面代码`+`前后增加一个空格

0. 将光标放在第一行第一个+号前任意位置
1. press `f+`    
2. `s␣+␣<Esc>` 
3. `;.`

Tips4:一些重复命令总结
---
1. `.`重复执行最后一次的修改操作 逆向操作：`u`
2. `f`,`F`,`t`,`T` 通过`;`重复执行 逆向操作 : `,`
3. `:s/target/replacement`    `&`    `u`

Tips5: 快速替换
---
	...We're waiting for content before the site can go live...
	...If you are content with this, let's go ahead with it...
	...We'll launch as soon as we have the copy...
替换第1个及第3个`content`不替换第二个`content`

1. 光标置于第二个`content`上
2. `*`  (查找光标所在位置的字)
3. `cwcopy<Esc>`
4. `n`
5. `.`


