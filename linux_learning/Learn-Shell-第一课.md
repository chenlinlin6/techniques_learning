# Learn Shell 第一课
## shell运行
shell可以在终端用命令行运行，也可以将命令写入到文本文件，文件后缀名为.sh。注意首行要写`#!/bin/bash`，代表着解释器所在的路径。#用来添加注释。
`$ps | grep $$`可以用来查看当前所用的解释器，也就是用来运行的程序。
`$which bash`可以用来查看解释器所在的路径

## 变量
### 赋值

我们可以用=来给变量赋值。

比如

```shell
$first_date=ABC
$echo $first_date
```

在这里first_date前面的`$`代表着输出这个命令或者变量

在和其他字母排在一起的时候我们还可以把变量名用`{}`将变量括号包起来防止混淆。

```shell
$My_first_character=ABC
$echo "My characters are ${My_first_character}DEF"
```

输出结果

`My characters are ABCDEF`

### 命令

除了变量名以外，`$()`或者````里面还可以加入linux执行语句，用来执行。

比如

```shell
$list=`ls`
$dir=/tmp/my-dir/file_$(/bin/date +%Y-%m-%d).txt
$echo $dir
```



在这里`$()`里面的内容被当做linux命令执行后替换了所在位置的内容。

### 练习

The target of this exercise is to create a string, an integer, and a complex variable using command substitution. The string should be named BIRTHDATE and should contain the text "Jan 1, 2000". The integer should be named Presents and should contain the number 10. The complex variable should be named BIRTHDAY and should contain the full weekday name of the day matching the date in variable BIRTHDATE e.g. Saturday. Note that the 'date' command can be used to convert a date format into a different date format. For example, to convert date value, $date1, to day of the week of date1, use:


在这里注意两点运用linux命令既可以用`$()`也可以用``这个符号。等号前后不要有空格。

## 在命令行当中传递参数到shell脚本当中

```shell
#!/bin/bash
echo $3
BIG=$5
echo 'A $BIG costs $6'
echo $#
```

我们可以在终端运行shell脚本的时候，后面跟上要传入的参数，`$num`是写到shell脚本里面的内容，代表着在终端传入的第几个参数。

比如

```bash
$ ./bin/my_shell.sh apple 5 banana 8 "Fruit Basket" 15
```

以上就是在运行my_shell.sh这个脚本，把后面6个参数传入，当然，按照这个脚本里面的代码，只有第3个，第5个，第6个参数用上。

`$#`打印出传入参数的个数。

## 阵列

shell里面的阵列用括号表示。

```shell
$my_array=(apple banana "fruit")
$echo $my_array
```

返回元素的个数用`${#arrayname[@]}`的方式

```shell
$echo ${#my_array[@]}
```


### 取出对应的元素

```shell
$echo ${my_array[3]}#取出第4个元素
```



### 赋值

`my_array[4]=pear`

```shell
$echo ${my_array[4]}
$echo ${my_array[${#my_array[@]}-1]} 
```


## 基本运算


基本运算的表达式是`$((expression))`，运算的表示是跟python的语法一样的。

## 字符串的基本操作

### 打印字符串长度

```shell
$string='abcdef'
$echo ${#string}
```


## 打印子字符串里面的首个出现在父字符串的位置

`expr index`

```shell
$string='This is us' 
$substring='hat'
$expr index "$string" "$substring"
```


