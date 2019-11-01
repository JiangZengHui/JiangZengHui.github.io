---
title: Shell脚本编程
top: false
cover: false
toc: true
mathjax: true
date: 2019-10-22 22:14:03
password:
summary: Shell脚本编程的相关知识
tags: Linux网络运维
categories: Linux
---

#### 一、Shell简介
Shell是一个用C语言编写的程序，通过Shell用户可以访问操作系统内核服务，类似于DOS下的command和后来的cmd.exe。Shell既是一种命令语言，又是一种程序设计语言。作为命令语言，它交互式地解释和执行用户输入的命令；作为程序设计语言，它定义了各种变量、参数、函数、流程控制等等。它调用了系统核心的大部分功能来执行程序、建立文件并以并行的方式协调各个程序的运行。因此，对于用户来说，shell是最重要的实用程序，深入了解和熟练掌握shell的特性极其使用方法，是用好Unix/Linux系统的关键。



#### 二、两类程序设计语言
我们经常说道的shell脚本，其实是因为Shell是一种脚本语言，也就是解释性语言。程序设计语言可以分为两类：编译型语言和解释型语言。

| 语言 | 区别 |
|--|--|
| 编译型语言 | 需要预先将我们写好的源代码转换成目标代码，这个过程被称作“编译”。运行程序时，直接读取目标代码。由于编译后的目标代码非常接近计算机底层，因此执行效率很高，这是编译型语言的优点 |
| 解释型语言 | 也叫做脚本语言。执行这类程序时，解释器需要读取我们编写的源代码，并将其转换成目标代码，再由计算机运行。因为每次执行程序都多了编译的过程，因此效率有所下降 |


#### 三、Shell脚本解释器
Linux的Shell脚本解释器种类众多，一个系统可以存在多个shell脚本解释器，可以通过cat /etc/shells 命令查看系统中安装的shell脚本解释器。

```
[root@centos6-1 ~]# cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/tcsh
/bin/csh
[root@centos6-1 ~]# 
```

bash由于易用和免费，在日常工作中被广泛使用。同时，Bash也是大多数Linux系统默认的Shell脚本解释器。


#### 四、Hello World
新建helloworld.sh

```
[root@centos6-1 ~]# touch helloworld.sh
```

编辑helloworld.sh文件，添入一下内容

```
#!/bin/bash
echo "helloworld"
```

 - #! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell，这里指定bash
 - echo 是Shell的一个内部指令，用于在屏幕上打印出指定的字符串
赋予当前用户helloworld.sh的执行权限(刚创建的文件没有执行权限)

```
[root@centos6-1 ~]# chmod u+x helloworld.sh
```

执行hellowo.sh脚本方式一

```
[root@centos6-1 ~]# ./helloworld.sh 
helloworld
[root@centos6-1 ~]# 
```

> 注意，一定要写成./helloworld.sh，而不是helloworld.sh，linux系统会去PATH里寻找有没有叫helloworld.sh的，而helloworld.sh不在PATH里，所以写成helloworld.sh是会找不到命令的，要用./helloworld.sh告诉系统说，就在当前目录找。

执行hellowo.sh脚本方式二

```
[root@centos6-1 ~]# /bin/sh helloworld.sh 
helloworld
[root@centos6-1 ~]# 
```

> 这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，当使用这种方式时，脚本中的#!/bin/bash指定的解释器是不生效的，当前使用什么解释器就是什么解释器

#### 五、变量

 -  语法格式：变量名=变量值

shell变量定义的严格语法限制：

```
1.变量名和等号之间不能有空格
2.变量名首个字符必须为英文字母
3.不能包含标点符号但能够使用下划线(_)
4.不能使用空格
5.不能使用 bash 里的关键字
```

##### 1.变量类型

| 类型 | 解释 |
|--|--|
| 局部变量 | 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量 |
| 环境变量 | 所有的程序，包括shell启动的程序，都能访问环境变量，有些程 序需要环境变量来保证其正常运行。可以用过set命令查看当前环境变量 |
| shell变量 | 由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell 的正常运行 |

##### 2.定义变量

```
name="zaomianbao"
```

##### 3.引用变量

```
name="zaomianbao"
echo ${name}
echo $name
```

引用一个定义过的变量，只要在变量名前面加$即可，变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界。

##### 4.重新定义变量

> 已定义的变量，可以被重新定义

```
name="zaomianbao"
echo ${name}
name="tiechui"
echo ${name}
```

##### 5.只读变量

> 使用readonly命令可以将变量定义为只读变量，只读变量的值不能被改变

```
name="zaomianbao"
readonly name
name="tiechui"
```

##### 6.删除变量

```
name="zaomianbao"
unset name
echo $name
```

使用unset命令可以删除变量，变量被删除后不能再次使用，同时unset命令不能删除只读变量。

#### 六、参数传递

> 在执行Shell脚本时，可以向脚本传递参数。脚本内获取参数的格式为:$n。n代表一个数字，1为执行脚本的第一个参数，2为执行脚本的第二个参数，以此类推…
> $0 表示当前脚本名称。

1.修改helloworld.sh为

```
#!/bin/bash
echo $1
echo $2
echo $3
```

2.执行携带参数

```
[root@centos6-1 ~]# ./helloworld.sh haha wowow nini
haha
wowow
nini
[root@centos6-1 ~]#
```

#### 七、Shell字符串

> shell字符串可以用单引号，也可以用双引号，也可以不用引号

单引号

```
name='my name is zaomianbao'
```

 - 单引号字符串中不支持引用变量，任何字符都会原样输出
 - 单引号字串中不能出现单引号（对单引号使用转义符后也不行）
双引号

```
name='my name is zaomianbao'
name_again="\"${name}\""
```

 - 双引号里可以引有变量
 - 双引号里支持转义字符
 
字符串长度

```
name='my name is zaomianbao'
echo ${#name}   //执行输出为21
```

截取字符串

```
name='my name is zaomianbao'
echo ${name:11:20}   //执行输出zaomianbao
```

#### 八、shell数组
 - bash支持一维数组（不支持多维数组），并且没有限定数组的大小。在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：
　　array_name=(value1 … valuen)
　　

> 下面的例子将展示shell数组常见的所有操作

```
//第一数组
[root@centos6-1 ~]# usernames=(1 2 33 44 adsd1)
//默认读取第一个
[root@centos6-1 ~]# echo ${usernames}
1
//读取下标为0的
[root@centos6-1 ~]# echo ${usernames[0]}
1
//读取下标为1的
[root@centos6-1 ~]# echo ${usernames[1]}
2
//读取所有
[root@centos6-1 ~]# echo ${usernames[*]}
1 2 33 44 adsd1
//同样是读取所有
[root@centos6-1 ~]# echo ${usernames[@]}
1 2 33 44 adsd1
//获取数组长度
[root@centos6-1 ~]# echo ${#usernames[@]}
5
//同样可以获取数组长度
[root@centos6-1 ~]# echo ${#usernames[*]}
5
[root@centos6-1 ~]# 
```


#### 九、shell运算符
 - Shell和其他编程语音一样，支持包括:算术、关系、布尔、字符串等运算符。
 - 原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。expr 是一款表达式计算工具，使用它能完成表达式的求值操作。
 
##### 1.算术运算符
|运算符|意义|
|--|--|
|+|加法|
|-|	减法|
|*|	乘法|
|/|	除法|
|%|	模，即取余|

下面是详细例子

```
#!/bin/bash

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

执行

```
[root@centos6-1 ~]# ./helloworld.sh 
a + b : 30
a - b : -10
a * b : 200
b / a : 2
b % a : 0
a 不等于 b
[root@centos6-1 ~]# 
```


> 注意：
> 
>  1.乘号(*)前边必须加反斜杠(\)
>  2.条件表达式要放在方括号之间，并且要有空格

##### 2.关系运算符

> 关系运算符只支持数字，不支持字符串，除非字符串的值是数字

|运算符|	意义|
|--|--|
|-eq|	EQUAL 等于|
|-ne|	NOT EQUAL 不等于|
|-gt|	GREATER THAN 大于|
|-lt	|LESS THAN 小于|
|-ge|	GREATER THAN OR EQUAL 大于等于|
|-le|	LESS THAN OR EQUAL 小于等|

下面是详细例子
```
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi
if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
else
   echo "$a -gt $b: a 不大于 b"
fi
if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
else
   echo "$a -lt $b: a 不小于 b"
fi
if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
else
   echo "$a -ge $b: a 小于 b"
fi
if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
else
   echo "$a -le $b: a 大于 b"
fi
```
执行

```
[root@centos6-1 ~]# ./helloworld.sh  
10 -eq 20: a 不等于 b
10 -ne 20: a 不等于 b
10 -gt 20: a 不大于 b
10 -lt 20: a 小于 b
10 -ge 20: a 小于 b
10 -le 20: a 小于或等于 b
[root@centos6-1 ~]# 
```


##### 3.布尔运算符
|运算符|	意义|
|--|--|
|&&|	与|
| \|\| |	或|

下面是详细例子
```
#!/bin/bash

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```


执行

```
[root@centos6-1 ~]# ./helloworld.sh 
返回 false
返回 true
[root@centos6-1 ~]# 
```

##### 4.字符串运算符
|操作符|	意义|
|--|--|
|-z|	字符串长度是否为0，为0返回 true|
|-n|	字符串长度是否为0，不为0返回 true|
|str|	字符串是否为空，不为空返回 true|

下面是详细例子
```
#!/bin/bash

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n $a ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

执行

```
[root@centos6-1 ~]# ./helloworld.sh 
abc = efg: a 不等于 b
abc != efg : a 不等于 b
-z abc : 字符串长度不为 0
-n abc : 字符串长度不为 0
abc : 字符串不为空
[root@centos6-1 ~]# 
```

##### 5.文件测试运算符
| 操作符 |	意义 |
|--|--|
|-b file|	文件是否是块设备文件，如果是，则返回 true|
|-c file|	文件是否是字符设备文件，如果是，则返回 true|
|-d file|	是否是目录，如果是，则返回 true|
|-f file|	文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。|
|-g file|	文件是否设置了 SGID 位，如果是，则返回 true|
|-k file|	文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true|
|-p file|	文件是否是具名管道，如果是，则返回 true|
|-u file|	文件是否设置了 SUID 位，如果是，则返回 true|
|-r file|	文件是否可读，如果是，则返回 true|
|-w file|	文件是否可写，如果是，则返回 true|
|-x file|	文件是否可执行，如果是，则返回 true|
|-s file|	文件是否为空（文件大小是否大于0），不为空返回 true|
|-e file|	文件（包括目录）是否存在，如果是，则返回 true|

下面是详细例子
```
#!/bin/bash

file="/com/zaomianbao"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```


执行

```
[root@centos6-1 ~]# ./helloworld.sh 
文件不可读
文件不可写
文件不可执行
文件为特殊文件
文件不是个目录
文件为空
文件不存在
[root@centos6-1 ~]# 
```

#### 十、流程控制
##### 1.if-else

```
if condition
then
    //做你想做的事
else
    //做你想做的事
fi


if condition1
then
    //做你想做的事
elif condition2 
then 
    //做你想做的事
else
    //做你想做的事
fi
```

##### 2.case
case 语句匹配一个值或一个模式，如果匹配成功，执行相匹配的命令

```
case 值 in
模式1)
    //做你想做的事
    ;;
模式2）
    //做你想做的事
    ;;
*)
    //做你想做的事
    ;;
esac
```


取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。;; 与其他语言中的 break 类似，意思是跳到整个 case 语句的最后。取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

##### 3.for

```
for 变量 in 列表
do
    //做你想做的事
done
```

##### 4.while

```
while command
do
   //做你想做的是
done
```


##### 5.until
until 循环执行一系列命令直至条件为 true 时停止。until 循环与 while 循环在处理方式上刚好相反。一般while循环优于until循环，但在某些时候，也只是极少数情况下，until 循环更加有用。

```
until command
do
   //做你想做的事
done
```

> command 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

#### 十一、函数
函数可以让我们将一个复杂功能划分成若干模块，让程序结构更加清晰，代码重复利用率更高。像其他编程语言一样，Shell 也支持函数。Shell 函数必须先定义后使用。

```
#!/bin/bash

demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun 
echo "-----函数执行完毕-----"
```


执行

```
[root@centos6-1 ~]# ./helloworld.sh 
-----函数开始执行-----
这是我的第一个 shell 函数!
-----函数执行完毕-----
[root@centos6-1 ~]# 
```

> 如果函数有返回值，则函数返回值可以在调用该函数后通过 $? 来获得。

> 参考资料：
>  https://baike.baidu.com/item/shell/99702?fr=aladdin
> http://www.runoob.com/linux/linux-shell.html
> https://www.cnblogs.com/maybe2030/p/5022595.html
> https://baike.baidu.com/item/POSIX/3792413?fr=aladdin
