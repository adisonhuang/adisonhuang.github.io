# shell语法核心

## 1. 什么是Shell

shell是外壳的意思，就是操作系统的外壳。我们可以通过shell命令来操作和控制操作系统，比如Linux中的Shell命令就包括ls、cd、pwd等等。总结来说，Shell是一个命令解释器，它通过接受用户输入的Shell命令来启动、暂停、停止程序的运行或对计算机进行控制。

shell 是一个应用程序，它连接了用户和 Linux 内核，让用户能够更加高效、安全、低成本地使用 Linux 内核，这就是 Shell 的本质。

shell 本身并不是内核的一部分，它只是站在内核的基础上编写的一个应用程序。

## 2. shell脚本

shell脚本就是由Shell命令组成的执行文件，将一些命令整合到一个文件中，进行处理业务逻辑，脚本不用编译即可运行。它通过解释器解释运行，所以速度相对来说比较慢。

shell脚本中最重要的就是对shell命令的使用与组合，再使用shell脚本支持的一些语言特性，完成想要的功能。

## 3. Shell 变量

### 3.1 变量类型

运行shell时，会同时存在三种变量：

- **局部变量** 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
- **环境变量** 所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
- **shell变量** shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

### 3.2 变量操作

```shell

# 定义变量
name="adison"

# 使用变量
echo $name
echo ${name}

# 变量重新赋值
name="new_test" 

# 删除变量
unset name

```

### 3.3 字符串变量

```shell
your_name="runoob"
str="Hello, I know you are \"$your_name\"! \n"

# 字符串拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

# 获取字符串长度
echo ${#your_name} 

# 从字符串第 2 个字符开始截取 4 个字符：
echo ${your_name:1:4} 

# 查找字符 r 或 o 的位置(哪个字母先出现就计算哪个)：
echo `expr index "$your_name" ro` 

```

## 4. Shell 数组

数组中可以存放多个值，Bash Shell 只支持一维数组（不支持多维数组）

```shell
my_array=(A B "C" D)

# 读取数组
echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"

# 使用@ 或 * 可以获取数组中的所有元素
echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 获取数组的长度
echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```

## 5. Shell 传递参数

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $n       | n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推…… |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。 如" \$* "用「"」括起来的情况、以"\$1 \$2 … $n"的形式输出所有参数。 |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"\$@"用「"」括起来的情况、以"\$1" "\$2" … "\$n" 的形式输出所有参数。 |
| $-       | 显示Shell使用的当前选项。                                    |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

```shell
#!/bin/bash
echo "Shell 传递参数实例！";
echo "第一个参数为：$1";

echo "参数个数为：$#";
echo "传递的参数作为一个字符串显示：$*";
```

执行脚本，输出结果如下所示：

```sh
$ chmod +x test.sh 
$ ./test.sh 1 2 3
Shell 传递参数实例！
第一个参数为：1
参数个数为：3
传递的参数作为一个字符串显示：1 2 3
```



## 6. Shell 基本运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

### 6.1 通用运算符

下表列出了常用的字符串运算符, 假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                          | 举例                          |
| :----- | :-------------------------------------------- | :---------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。   |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。  |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ \$a == \$b ] 返回 false。   |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ \$a != \$b ] 返回 true。    |
| -eq  | 检测两个数是否相等，相等返回 true。                   | [ \$a -eq \$b ] 返回 false。 |
| -ne  | 检测两个数是否不相等，不相等返回 true。               | [\$a -ne \$b ] 返回 true。 |
| -gt  | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ \$a -gt \$b ] 返回 false。 |
| -lt  | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ \$a -lt \$b ] 返回 true。 |
| -ge  | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ \$a -ge \$b ] 返回 false。 |
| -le  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ \$a -le $b ] 返回 true。 |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o \$b -gt 100 ] 返回 true。 |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a \$b -gt 100 ] 返回 false。 |
| &&     | 逻辑的 AND | [[ \$a -lt 100 && \$b -gt 100 ]] 返回 false |
| \|\|   | 逻辑的 OR  | [[ \$a -lt 100 \|\| \$b -gt 100 ]] 返回 true |

**注意**：条件表达式要放在方括号之间，并且要有空格，例如: **[$a==\$b]** 是错误的，必须写成 **[ \$a == \$b ]**。

### 6.2 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                         | 举例                       |
| :----- | :------------------------------------------- | :------------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ \$a = \$b ] 返回 false。 |
| !=     | 检测两个字符串是否不相等，不相等返回 true。  | [ \$a != \$b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。     |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。    |
| $      | 检测字符串是否为空，不为空返回 true。        | [ $a ] 返回 true。         |

### 6.3 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

## 7. Shell echo命令

```shell
echo "It is a test"
# 显示转义字符
echo "\"It is a test\""

echo -e "OK! \n" # -e 开启转义

# 显示结果定向至文件
echo "It is a test" > myfile

# 原样输出字符串，不进行转义或取变量(用单引号)
echo '$name\"'

# 显示命令执行结果
echo `date`
```



##  8. Shell 流程控制

### 8.1 if else

```shell
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi

# 输出
a 小于 b
```

### 8.2 for 循环

```shell
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done

#输出
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```

### 8.3 while 语句

```shell
#!/bin/bash
# 返回数字 1 到 5，然后终止
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

> 以上实例使用了 Bash let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量

### 8.4 until 循环

```shell
#!/bin/bash
#输出 0 ~ 9 的数字：
a=0
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```



## 9. Shell 函数

```shell
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
    return 255
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73

# 输出

第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```

注意：

1. return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值n(0-255

2. 在函数体内部，通过 \$n 的形式来获取参数的值，例如，\$1表示第一个参数，\$2表示第二个参数...

3. \$10 不能获取第十个参数，获取第十个参数需要\${10}。当n>=10时，需要使用${n}来获取参数。

   另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本或函数的参数个数                                   |
| $*       | 以一个单字符串显示所有向脚本传递的参数                       |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |



## 10. Shell test 命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

```shell
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```



## 11 .Shell 输入/输出重定向

### 11.1  重定向命令

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

!!! info "注意"
      文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。



```shell
# 将 stdout 和 stderr 合并后重定向到 file
$ command > file 2>&1

或者

$ command >> file 2>&1
```

### 11.2  /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：

```shell
$ command > /dev/null
```

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到**"禁止输出"**的效果。

如果希望屏蔽 stdout 和 stderr，可以这样写：

```shell
$ command > /dev/null 2>&1
```



## 12 Shell 文件包含

创建两个 shell 脚本文件。

test1.sh 代码如下：

```shell
#!/bin/bash
url="http://www.baidu.com"
```

test2.sh 代码如下：

```shell
#!/bin/bash

#使用 . 号或者source来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "百度地址：$url"
```

接下来，我们为 test2.sh 添加可执行权限并执行：

```shell
$ chmod +x test2.sh 
$ ./test2.sh 

#输出
百度地址：http://www.baidu.com
```
