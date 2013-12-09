---
layout: post
title: N tips for shell newbie (1)
description: 记录在写shell脚本时遇到的坑与技巧。
keyword: linux, bash, shell
---

##Tip 1. 注意shell执行的具体类型

Shell脚本的第一行（Shebang line, e.g. `#!/bin/bash`）指定了其执行的具体类型，`dash/bash/ksh/…`等等，之间会有些细微的区别。如果你的脚本没有Shebang line，那么由系统变量$SHELL来决定其执行类型。

PS1：`/etc/shells`中记录着系统可用的shell类型。所以当你发现同样的脚本在不同系统表现不同的时候，有可能是dash和bash（或是其他shell）之间的区别。

PS2：sh是一种POSIX规范，而众多的shell类型是它的实现。linux系统中的sh一般是个symlink，例如ubnutu中默认的sh就会指向/bin/dash。

##Tip 2. 赋值语句两边的空格

`a = 'test'`，这条语句是错误的。因为在shell里赋值语句=的两边是不能有空格的，`a=test`才是正确写法。这个错误几乎是初学者必犯的错误了。。。

但是在判断语句中[]两侧则必须留空格：`if [a == 'test']`是错误的；`if [ a == 'test' ]`是正确的。

##Tip 3. 各种预留的$变量

- `$*`表示参数列表$0, $1, $2…
- `$#`表示参数列表个数
- `$?`表示上一条命令的执行结果
- `$!`表示上一条命令的PID号

PS1：对于整个shell来说，参数列表是运行时接的各个参数；对于shell中的函数来说，参数列表是调用时传入的各个参数。

PS2：对于`test.sh '1 2' 3 4`来说，`$*`与`$@`是等价的（均包含1 2 3 4四个参数）。`"$*"`与`"$@"`则不同，`"$*"`是一个参数`1 2 3 4`，而`"$@"`为三个参数`1 2` `3` `4`）在。Google的[shell code](http://google-styleguide.googlecode.com/svn/trunk/shell.xml)规范推荐使用`$@`。

##Tip 4. 检查文件/文件夹是否存在等等

- `if [ –d ${dir} ]`判断文件夹是否存在
- `if [ –f ${file} ]`判断文件是否存在
- ……

这里想说的是如何查询诸如此类判断条件的说明：`man [` 或 `man test`即可。

##Tip 5. `[ … ]` 与 `[[ … ]]`

`[ … ]`指的是标准POSIX规范中的条件判断，而`[[ … ]]`是对POSIX规范的扩展。

这些区别有：（注意：对POSIX Shell适用的对Extended POSIX Shell均适用，反之则不然）

|               | POSIX Shell   | Extended POSIX Shell  |
| ------------- |:-------------:|:---------------------:|
| 判断字符串相等  | =             | ==                    |
| 多条件判断      | -o            | \|\|                  |
|                | -a           | &&                    |
| 正则匹配        | N/A          | =~                    |

##Tip 6. 替换if判断语句

```
if [ ! false command ]; then 
  dosomthing
```
可以用`false command || dosomething`替代。同理：
```
if [ true command ]; then
  dosomthing
```
可以用`true command && dosomething`替代。

##Tip 7. 将执行结果赋予变量

有两种方式：
```
var1=`command` 或者 var1=$(command)
```
Google的[shell code](http://google-styleguide.googlecode.com/svn/trunk/shell.xml)规范推荐后者。

##Tip 8. 变量替换

- `${var1-str}`判断当var1为空时，返回str（不改变var1的值）
- `${var1+str}`判断当var1不为空时，返回str（不改变var1的值）
- `${var1=str}`判断当var1为空时，返回str，且将str赋给var1
- `${var1?str}`判断当var1为空时，在错误输出中输出str

PS：也有在var1后加上`:`的写法，二者大部分情况下是等价的。除了一个变量被定义但没赋值的情况（`var1=`），这种情况下加`:`会报错。

- `${var1:pos}`返回第pos个字符后的值
- `${var1:pos:len}`返回第pos字符后，长度为len的值
- `${var1/pattern/replacement}`将var1匹配到的第一个pattern替换为replacement并返回
- `${var1//pattern/replacement}`将var1中的所有pattern替换为replacement并返回
- `${var1/#pattern/replacement}`相当于标准正则中的`^`pattern，即匹配的pattern必须在var1首部
- `${var1/%pattern/replacement}`相当于标准正则中的pattern`$`，即匹配的pattern必须在var1尾部

- `${var1#pattern}`将var1中**从前往后**遇到的第一个尽可能短的pattern删除并返回
- `${var1##pattern}`将var1中**从前往后**遇到的第一个尽可能长的pattern删除并返回
- `${var1%pattern}`将var1中**从后往前**的第一个尽可能短的pattern删除并返回
- `${var1%pattern}`将var1中**从后往前**的第一个尽可能长的pattern删除并返回

以上操作均不改变var1的原始值（即不带-i的sed）。实际上以上的所有操作都能通过sed完成，但我的习惯是能用原生的shell替换，就不再调一个sed。

##Tip 9. 数值计算与判断

因为shell中没有原生的浮点数，以下所说的数值均是指整数。总体来说，赋值/计算的方法有三种：

- let 
- expr
- (())

```
#!/bin/bash
foo=1

let bar1=foo+1
let bar2=$foo+2
bar3=$(expr $foo + 3) ＃不能用foo
bar4=$((foo+4)) ＃不能用$foo

echo $bar1$bar2$bar3$bar4 # 输出2345
```

关于数值的比较，初学者很容易犯错，想当然的使用`==`, `<=`等等。实际上bash必须使有特定的数值比较符：`-eq`, `-ne`, `-lt`, `-gt`, etc。

##Tip 10. 比较字符串/数值的大小

TODO...