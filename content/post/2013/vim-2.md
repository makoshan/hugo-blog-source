---
title: "Practical Vim读书笔记2"
date: "2013-09-28T19:01:21+08:00"
categories:
- Dev
tags:
- vim
---

好久没写博客了，国庆假期要好好补补。今天继续写VIM的读书笔记。<!--more-->
Tips6:可复用的更改 
---
	I am your baby girl	
假设你当前的光标在最后一个单词girl的最后一个字母`l`上，请删除此单词。
### a. 方案1
1. `db` 将会删除gir
2. `x`  删除l

### b. 方案2
1. `b` 将光标跳回到`g`上
2. `dw` 删除`girl`

### c. 方案3
1. `daw` delete a word(删除整个单词)

### d. 分析
上面三种方案最终都需要按三次按键，但方案3的改变发生在最后。所以此动作是可复用的。比如，你可能还需要girl前面的一个单词baby
甚至更多的。这时你你可以只通过按`.`就能重复执行刚才的`daw`操作
Tips7:数字操作
---
	.blog { background-position: 0px 0px }
假设现在你需要将第一个`0px`更改为`180px`
VIM提供的有一个简单的计算命令：

1. `num<C-x>` 用光标所在行的第一个数字减去`num`所得结果替换原数字
2. `num<C-a>` 同上,只不过此命令执行相加操作 

针对上面的案例你可以这样:

1. 首先将光标置于要操作的行上(光标应位于要操作的数字上，或者之前)
2. `180<C-a>`

Tips8:大小写
---
	i am a good boy
1. `gUU` 整行全部大写
2. `guu` 整行全部小写
3. 指定部分切换大小写
	1. 通过`v`或`V`选定区域
	2. `gu` or `gU`执行操作

Tips9:`insert mode`下的修正操作
---
我们所有的输入操作基本上都是在`insert mode`下，在此模式下产生错误也是不可避免的，那么有没有方法直接
在`insert mode`下完成修改操作

1. `<C-h>` 向前删除一个字母(同backspace)
2. `<C-w>` 向前删除一个字
3. `<C-u>` 向前删除整行

Tips10:回到`normal mode`
---
1. `<ESC>` 返回`normal mode`
2. `<C-[>` 同上(你可能会觉得Esc太远)
3. `<C-o>` 返回到`insert normal mode`

