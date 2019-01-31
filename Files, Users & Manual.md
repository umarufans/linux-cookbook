# 文件 / 用户 / 手册

## 一种认识 Linux 操作系统（服务）的方式

* 了解这些系统程序 / 指令（System Program）是如何使用的
* 学习系统调用（System Call）
* 编写自己的系统程序

## 文件 / File

在 Linux 操作系统里，**“一切都是文件”**。而文件的“打开”等状态，是用**文件描述符**表示的。

> **自驱式学习**
>
> Linux 提供了完善且清晰的**用户手册**（Manual），可以帮助我们自驱地学习如何使用该操作系统。我们可以使用指令在 Manual 中进行关键字搜索，以获取我们需要的指令及语法。
>
> 我们可以使用`man <name>` 查询某个指令 / API 的手册，同时可以使用`man man`获取 Linux Manual 更详细的用法。
>
> **注意：Server 版本的 Linux 发行版中，出于空间资源的考虑，系统默认删除了很多详尽的说明或者不常用的指令文档，所以请尽量使用 Desktop 版本进行 Linux 学习。**

### 如何解决“如何打开文件”这个问题？

尝试使用 Linux Manual 进行搜索：`man -k file | grep open | grep \(2\)`。该指令的含义是：在 Manual 中搜索 "file" 关键词，并过滤保留包含 "open" 的条目，再次过滤保留包含 \(2\) 的条目（即 Manual 的第二章）

> 管道（Pipe）：`|`符号，又称管道操作符，它的功能是将它左边程序输出的内容输入到右边的程序中，即为左右两个程序建立一个“管道”，实现 I/O Redirection。

_如果你的_ `man -k` _不能正常工作，不要急，往下看。_

### 在终端中浏览大量内容

在传统的 UNIX 操作系统环境中，由于缺少 GUI ，只能显示一个屏幕大小的文本内容。为了解决这个问题，Linux 提供了诸如 `less`，`more`等辅助显示大量文本的工具。通过与`|`配合可以很轻松地在屏幕范围内浏览大量文本。

`less` 和 `more` 都是用来浏览无法在一个屏幕显示完全的大文件的工具。`more` 是一个很古老的指令，顾名思义，他只能“more”，即只支持向下翻页。与 `more` 不同的是，`less` 不仅可以向下翻页，同时也可以向上翻页。不过，很多现代的 Linux 发行版在执行 `more` 指令的时候自动执行 `less` ，或是提供了一个更接近 `less` 的 `more` 以方便用户使用。

_相关链接：_[_https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less_](https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less)

### 编程：仿写 cat 指令

`cat` 指令是 Linux 中一个非常常用的指令，其接收若干个文件作为参数，功能是将这些文件的内容按顺序合并并打印。

#### 语言提示：参数列表

相信很多学过 C 语言的同学，都见过如下代码：

`int main ()`

但实际上，main 函数的完整原型是这样的：

`int main (int argc, char* argv[])`

`argc` 是 arguments count 的缩写，表示该可执行程序收到的参数个数，`argv` 是 argument value 的缩写，是收到的参数构成的数组列表。

当你的程序需要接收用户传递的参数时，可以使用 `argc` 判断用户传递的参数个数是否正确，然后使用 `argv` 获取用户传入的参数。以 `cp` （复制）指令为例： `cp -r ~/Desktop/test-folder ~/Documents/`，那些用空格分隔开的就是程序的参数，也就是 `argv`。这个语句为 `cp` 传入了 `-r`，`~/Desktop/test-folder` 和 `~/Documents/` 三个参数。

需要注意的是，`argc` 的**起始值是 1**，即没有任何参数传入的时候 `argc` 的值为 1。因为程序本身会作为 `argv[0]` 被传入程序中。比如 `cp` 的 `argv[0]` 就是 `"cp"`。

#### 代码实现

##### 思路

思路很简单：获取参数列表 —> 打开文件（暂时不考虑异常情况）-> 逐个字符输出 -> 关闭文件 -> 打开下一个文件 ...

##### 实现

[cat.c](https://github.com/lcx960324/linux-tuto-demo-code/blob/master/Files%2C%20Users%20%26%20Manual/cat.c)

### 编程：仿写 cp 指令

#### 语言提示：buffer

buffer（缓冲区）

buffer的大小会影响二进制流传递的效率。

#### 语言提示：文件操作

### 文件操作及系统调用

#### 文件操作

对于一个文件的操作，无外乎以下四种：打开、关闭、写入、读取。Linux Manual 给出了非常清晰的关于文件 open / close / write / read 操作的文档说明，不妨使用上边提到的`man -k`尝试搜索一下。我们也会在下边的编程练习中给出提示。

我们之前提到过，在 Linux 中，“万物皆文件”。在 Linux 中，对设备、SOCKET等进行的一系列开、关、读、写操作归根结底都是在对其对应的文件进行操作。而这些文件的“打开状态”等一系列状态都是通过文件描述符表示的。这就引出了一个新的问题：**系统是如何通过文件描述符描述一个文件的？**

\[WIP: file description\]

## 用户 / User

### 思考：系统如何表示 / 记录“一个用户登录了”？

#### 从指令出发

Linux 为我们提供了一个指令，用来查看当前在线的用户列表：`who`。它是怎么实现的呢？不妨先在 Manual 里边看一看：

`man who` - `who [OPTION]... [ FILE | ARG1 ARG2 ]`

可以看到，who 接收一组参数，除了配置参数外，还包含一个文件（或者两个额外参数）。如果你的 Linux 拥有完整的 Manual ，你应该会看到如下片段：

`If FILE is not specified, use /var/run/utmp.  /var/log/wtmp as FILE is common.`

也就是说，如果没有指定 FILE 的话，系统会默认从上述两个文件中读取用户登录日志。自此，我们可以猜想，Linux 通过修某个文件来记录用户的登录状态。\[WIP\]

#### 从文件出发

我们之前提到过，在 Linux 中，“万物皆文件”。所以关于用户的一系列信息，自然也离不开文件。事实上，用户的信息被分散的保存在了以下几个文件中：

/etc/passwd - 保存用户的基本信息

/etc/shadow - 保存用户密码

/var/run/utmp - 保存用户登录记录

\[WIP\]

对于用户来说，这些文件的敏感度极高，我们一定不希望自己的信息被其他用户看到。为了对在这些文件上进行的操作进行干预和控制，Linux 设计了一套非常精妙的权限控制系统 - 基于 ACL（Access Control Lists）的权限控制。

### 初窥 Linux 权限控制系统

\[WIP\]

### 编程：仿写 who 指令

## 手册 / Manual

### man -k 与 whatis database

#### man -k

之前提到，`man -k`可以用来在 Manual 中搜索我们想找的关键字。而它又是如何做到快速索引整个 Linux Manual 的呢？不妨使用`man man`指令查看一下关于 Linux  Manual 的文档，找到`-k`参数的说明：

> Equivalent to apropos.

然后继续`man apropos`

> apropos - search the whatis database for strings

看来这个指令是在一个叫做 whatis database 的数据库中搜索信息的。所以，接下来的问题就是：what is whatis & what is whatis database。

#### whatis & whatis database

我们继续在 Manual 中查询：`man whatis`

> whatis - search the whatis database for complete words.

似曾相识吧。先不去追究`strings`和`complete words`的区别，我们继续往下看：

> whatis searches a set of database files containing short descriptions of system commands for keywords and displays the result on the standard out-put.

看来这个 a set of databases 指的就是 whatis database 了，它是一个包含了系统指令的关键词的简述的数据库。所以索引这个数据库，就可以在 Manual 中很快的找到我们需要的信息了。

那些没有提供 whatis database 的 Linux 发行版，基本都提供了`makewhatis`指令，用来让用户创建 whatis database。如果执行`man -k`的时候没有成功，不妨试一下`makewhatis`。

