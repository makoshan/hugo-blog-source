---
title: "Practical Vim读书笔记3"
date: "2013-09-29T19:01:21+08:00"
tags:
- vim
---

Tips11: Insert Normal Mode
---
紧接[上一篇](/blog/20130928/vim-2/)中最后说到的`<C-o>`可以进去`Insert Normal Mode`.这里介绍一下此模式。<!--more-->

* 这个模式只能在`insert mode`下通过`<C-o>`命令进入.
* 进入此模式后，你可以执行一个在`normal mode`下的命令，当此命令执行完后。会自动返回到之前的`insert mode`.

你可能已经想到此模式的应用场景了。有时候我们处于`insert mode`但这时我们想执行一条`normal mode`下的命令，然后
接着编辑，当然，可以通过`Esc`返回到`normal mode`下.然后执行之，最后再通过`i`返回到`insert mode`下继续刚才的操作。
但这样很明显太麻烦了。还好有这个模式。

比如，你编辑文档到当前窗口的最后一行后，想要将当前行调整到窗口的中间，接着编辑。你可以:

1. `<C-o>` 进入`insert normal mode`
2. `zz` 调整当前行到窗口中间
3. go on...

Tips12: insert mode下的paste操作
---
	Practical Vim, by Drew Neil
	Read Drew Neil's 
补全上面第二句，显然书名`Practical Vim`已经出现过了，我们可以直接`yank`.具体请看下图：

![vim-paste](/images/posts/vim3-paste.png)

其中`t`命令与`f`命令相似，只不过`t`会将光标定位到所搜索字符的前面
Tips13: 更加强大的计算
---
记得我们在上一篇中介绍的数字操作吗？这里有一个更加强大的计算命令，可以执行各种`+,-,*,/`操作。
	6 chairs, each costing $35, totals $
请计算出总和，并插入到最后。

![vim-caculator](/images/posts/vim3-caculator.png)

note:`<CR>`即是`<Enter>`
Tips14: 插入特殊符号
---
有时候我们需要插入一些特殊符号，并且这此符号在键盘上还没有。这时我们可以通过vim提供的`<C-v>{code}`命令
code 对应字符的编码比如`A`对应`065`.此命令也是在`insert mode`下直接执行的。
如果我们插入的字符编码长度超过三个数字长度，可以通过`<C-v>u{code}`来执行。此时的code代表的是16进制的编码
.但是通过记编号是不是太不方便了。所以VIM提供了`<C-k>{char}{char}`命令 （此命令在我这里没有测试通过）
通过`:h digraph-table`可以查看符号编码之间的对照关系
试一下：

1. `<C-v>123`   {
2. `<C-v>u1234` ሴ
3. `<C-k>?I`    ¿

Tips15: Virtual Replace Mode
---
	1.	name
		sex
		age
如果我们想给上面三行加上行号，其中每行都是以一个`tab`开始的。
这时我们通过`R`命令可以达到效果，你可以试一下，`R`命令把`tab`解析为一个占位符，它会用我们输入的行号整个替换一个`tab`
如果我们只想用tab中的一个空格去替换就要通过`gR`命令。(个人感觉这个不是特别有用)
