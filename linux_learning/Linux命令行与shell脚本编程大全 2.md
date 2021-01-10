# 基础理解

shell只是一个壳，或者说是一个程序，包含多个执行程序的类型，包括bash，tsch等

# 1. linux文件系统

虚拟目录结构的划分

`Linux`顶层虚拟目录名机及其内容。







物理内存和虚拟内存的区别


useradd -s shell 更改默认的登陆shell是怎么回事


# 2. 查看文件内容
## 查看文件类型

1.cat命令
cat命令查看所有的文件
加上行号

```shell
cat -n test1
cat -b test1
```

2.more命令
more是分页工具，可以去翻页，强制程序在执行完一页以后停止下来，更加符合人的阅读习惯



3.ls命令

常用参数：

| 参数 | 用处             |      |
| ---- | ---------------- | ---- |
| a    | 列出所有文件     |      |
| F    | 区分文件夹和文件 |      |
| r    | 递归地展现出目录 |      |
| l    | 完整信息         |      |
|      |                  |      |
|      |                  |      |
|      |                  |      |

4.删除命令

# shell脚本编程基础

## 显示消息

echo是脚本与用户交互的基础

在写完脚本之后，要将脚本改变权限：

```shell
$chmod u+x test.sh
```

# 11 构建基本脚本



```shell
#!/bin/bash
# This script display the date and the user who logged it 
who
date
```





> 1. 第一行`#!/bin/bash`代表选择什么shell去执行这个脚本文件，整个文件中注释只有第一行是执行的，要加`!`
> 2. 第二行就是注释了，不执行

![image-20200815113708651](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghrjz0x56zj30t601qweu.jpg)



会发现有个权限问题，需要修改权限：

```bash
$chmod u+x test1
```





```shell
#!/bin/bash
# This script display the date and the user who logged it 

echo The time and date are:
date

echo -n The time and date are:
date 
```

![image-20200815114407800](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghrjz1smzuj30ro02mgm6.jpg)

![image-20200815114440349](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghrjz1dcmvj30u601qjrx.jpg)

`-n`参数可以连接两行

# 19 sed和gawk

`sed`称为stream editor，也就是流编辑器，在输入数据的时候就已经基于预先制定的规则去处理数据的编辑器，而不是类似`VIM`的交互式编辑器

`sed`命令的格式如下：

```bash
$sed -options script file
```



sed命令格式

| 选项      | 描述                                               |
| --------- | -------------------------------------------------- |
| -e script | 在处理输入时，将script的命令添加到已有的命令中     |
| -f file   | 在处理输入时，将文件file包含命令添加到已有的命令中 |
| -n        | 不产生命令输出                                     |

sed可以接受标准输入

```bash
$echo "My name is chenlinlin"|sed -e "s/chenlinlin/Forest/"
```

![image-20200815205336733](/Users/chenlinlin/Library/Application Support/typora-user-images/image-20200815205336733.png)

也可以接受文件输入

```bash
$sed -e "s/chenlinlin/Forest/" data.txt
```

接受多个命令

```shell
$sed -e "s/chenlinlin/Forest/; s/hi/hello/" data.txt
```

也可以多行

```bash
$sed -e 's/chenlinlin/Forest/
> s/hi/hello/
> s/his/him/' data.txt
```

在这种情况下，多行分开输入不用写分号，只有当最后的单引号出现才会识别为结束。

## sed

给文件`empFile`插入某行

```
Hilesh, 1001
Bharti, 1002
Aparna, 1003
Harshal, 1004
Keyur, 1005
```

假设说，我现在有这样一个文件。然后我要在第一行插入`"Employee, EmpId"`这个语句

```shell
$sed "1i 'Employee, EmpId'" empFile
```



## 其他

sed替换同一行的多个，使用`flag`

`s/pattern/replacement/flag`

 





替换文件中的路径名

例如，想把`/etc/passwd`里面的bash shell用c shell来替换

```bash
$sed -ne 's!/bin/bash/!/bin/csh!'
```

这里的`!`就代表着是`sed`里面的命令分隔符

