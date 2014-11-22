---
layout: post
title: Tips for shell newbie (2)
description: 记录在写shell脚本时遇到的坑与技巧。
keyword: linux, bash, shell
categories: programming
---

## <a name="tip11"></a>Tip 11. 函数的返回值

* 和大多数语言一样，返回值用`return`（如果不指定则返回最后一条语句的返回值）。但和大多数语言不一样，shell中返回值**只能为整数**。
  
  ```bash
  fun1() {
    return 100 # 正确
  }

  fun2() {
    return "fun2" # 错误
  }
  ```

- 调用时接收不能用`foo=fun1`（实际上这不是调用函数`fun1`，而是将字符串fun1赋给foo）。得先调用`fun1`，再用`$?`得到返回值。

  ```bash
  foo=fun1 # 错误
  echo $foo # 输出fun1
  
  fun1
  echo $? # 输出100
  ```

## Tip 12. 命令的返回值
一般的linux命令，都可以在shell中直接使用，我们也经常利用这些命令（包括[Tip 11](#tip11)中的自定义函数）的标准输出（管道1）来当作"hacked"返回值。

```bash
foo() {
  echo "foo()"
  return 100
}

out=$(foo)
echo $? # 100
echo $out # foo()

out=$(ls)
echo $? # 0
echo $out # 当前目录的文件列表
```

这里也有一个坑要注意：在自定义函数返回结果时，所有此函数中的标准输出都会返回，而不是最后一条标准输出：

```bash
foo() {
  echo "$FUNCNAME is being invoked"
  echo "foo()"
}

out=$(foo)
[[ $out == "foo()" ]] && echo "true" # 不会输出true因为out的值是两条echo
```

## Tip 13. 调用shell时传入read等待的输入

假设有一个程序等待用户的输入yes or no:

```bash
#!/bin/bash
# read.sh

echo "Input yes or no:"
read input
echo $input
```

而这个脚本被串在另一个自动化脚本中，这个等待用户输入的行为会打断自动化。这种情况下可以用 `echo yes | ./read.sh` 来做到。

那么再考虑这种情况：

```bash
#!/bin/bash
# read.sh

echo "Input yes or no:"
read input1
echo "Confirm: Input yes or no:"
read input2
echo $input1
echo $input2
```

你会怎么做？`echo "yes\nyes"`？这样你的输入还只是会被赋给input1，`\n`并没有如你所愿转义并当作两次输入，结果如下：

```bash
Input yes or no:
Confirm: Input yes or no:
yesnyes
```

这种需要多输入的有两种解决方案：

- 使用文件重定向(准备一个文件事先写入分行的输入)

  ```bash
  cat input
  yes
  yes
  
  ./read.sh < input
  ```

- `echo -e "yes\nyes" | ./read.sh`

## Tip 14. 变量的范围

bash中的变量分为全局变量和本地变量。

- `var1=value`此类声明的变量为全局变量。若脚本读到（按执行顺序）的变量沿未被赋值，则该变量为空。另外，在shell中也可以引用任何预定义的变量，用`printenv`可以查看。
- `local var1=value`此类声明的变量为局部变量，且只能用在函数中。

  PS：变量无需声明就可以使用，但通过`declare option var`声明可以限制变量的类型。`option` could be:
  
  - -r read only variable
  - -i integer variable
  - -a array variable
  - -f for funtions
  - -x declares and export to subsequent commands via the environment

## Tip 15. 数组

从一个初学者的角度，bash的数组语法是比较晦涩的，远没有其他高级语言那么直观。thegeekstuff上的[这篇文章](http://www.thegeekstuff.com/2010/06/bash-array-tutorial/)是很不错的入门材料。

## Tip 16. 重用函数

假设我们有一个shell：

```bash
#!/bin/bash
# util.sh

func1() {...}
func2() {...}
```

那么在别的shell中可以用以下方法重用：

```bash
source ./util.sh

func1
func2
```

## Tip 17. alias转义

在写shell来自动化一些任务时，你可能不想看到烦人的confirm yes or no的信息，所以经常性地会使用类似`-f`的flag。但这些命令有可能其实是一个`alias`：

```bash
hweicdl@hweicdl-t430:~$ alias rm
alias rm='rm -i'
```

在上面这个例子的环境中，`rm -f`相当于`rm -f -i`，而`-i`会覆盖`-f`。所以在上面的环境中`rm -f`还是会要求你confirm是否删除。解决方案是`\rm -f`来使用原始的`rm`。

## Tip 18. 删除\^M

^M英文名叫garbage character，出现在文件的末尾，一般是由于windows上以`\r\n`结尾的文件传到linux上来导致的。

以下几种方法都可以删除它：

- `sed -i -e 's/^M//' filename`
- 在vi下，`:%s/^M//g`
- `dos2unix filename` (可能需要先安装dos2unix)

但要注意的是这个garbage character其实是**一个字符**，在匹配的时候**不能分别输入**`^`和`M`，得先按住`Ctrl`，再按`V`和`M`。

## Tip 19. I/O重定向

在linux上，一般至少有三个文件是打开着的：`stdin`（输入，默认来自键盘），`stdout`（标准输出，默认输出至屏幕）和`stderr`（错误输出，默认输出至屏幕）。这三个文件被分配了三个FD(File Descriptor)：`0`,`1`,`2`。我们在操作bash的输入输出重定向，需要和它们打交道。

我常用的几个操作包括：

- `command < input-file > output-file` 命令接收input-file的输入，并将输出重定向至output-file
- `> file` 清空一个文件的内容
- `command &> /dev/null` 将标准输出和错误都丢弃，等价于`command 2>&1 1>/dev/null`
- `echo "err info..." >&2` 在捕获住的错误放至错误输出，而不是直接丢到屏幕或标准输出

以下两篇文章介绍的比较详细：

- [http://cloudbbs.org/forum.php?mod=viewthread&tid=15515](http://cloudbbs.org/forum.php?mod=viewthread&tid=15515)
- [http://www.tldp.org/LDP/abs/html/io-redirection.html](http://www.tldp.org/LDP/abs/html/io-redirection.html)

## Tip 20. Here Document

在打印脚本的help信息时，可能我们会这么做：

```bash
echo "usage: blabla..."
echo "-a means..."
echo "-b means..."
```

也可以这么写：

```bash
cat << EOF
usage: blabla...
-a means...
-b means...
EOF
```

这个语法在一个脚本执行时动态生成另一个脚本尤其方便：

```
cat << EOF > another_script.sh
#!/bin/bash
command1...
command2...
EOF

bash another_script.sh
```
这样就不用一句句`echo`了。

## [Go to Tip 1 - Tip 10]({{site.url}}/programming/2013/12/06/tips-for-shell-newbie-1.html)
