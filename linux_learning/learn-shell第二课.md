# learn shell第二课

## 一、字符串操作

## 1.1 字符串长度

```shell
$STRING='this is a string'
$echo ${#STRING}#"#"号往往与长度或者个数有关，比如$#就是返回传入脚本的参数个数
#16
```

## 1.2 索引

检测一个字符串里面的字母首次出现在另外一个字符串的位置

```shell
$STRING='this is a string'
$SUB_STRING='hat'
$expr index "$STRING" "$SUB_STRING"
#1
#1是hat当中的t首次在STRING当中出现的序号
```

## 1.3 切片

索引用${STRING:N:M}的方式切片，N是开始的位置，M是结束的位置

```shell
$echo ${STRING:3:7}
#s is a
```

## 1.4 子字符串替换



```shell
$STRING="to be or not to be"
$echo "${STRING[@]/be/eat}"
#to eat or not to be
```

字符串替换方法可以用${STRING[@]/replaced/new_item}这样的结构去替换，其中STRING[@]代表取出STRING里面的所有元素，也就是所有字母。

`${STRING[@]/replaced/new_item}`默认替换第一个，要全部替换，用`${STRING[@]//replaced/new_item}`

```shell
$echo "${STRING[@]//be/eat}"
#to eat or not to eat

$echo "${STRING[@]/#to be/eat}"#替换前面出现的to be
#eat or not to be

echo "${STRING[@]/%be/eat}"#替换后面出现的be
#to be or not to eat
```

## 二、判断语句

判断语句的基本语法结构是：

```shell
if [ judgement ];then
	code if the judgement is true
fi
```

在这里要注意几点，一个是if在结束的时候是有fi的代表finish；另外如果then和if同行的话，要加分号，然后还有judgement里面的判断，注意跟前后括号都有空格，另外里面的判断是跟之前的操作符是一样的，操作符前后也要有空格。

示例：

```shell
NAME="John"
if [ "$NAME" = "John" ];then
	echo "His name is John"
fi
#His name is John
```

另外也可以加入elif还有else其他条件

```shell
NAME="Peter"
if [ "$NAME" = "John" ];then
	echo "His name is John"
elif [ "$NAME" = "Mary" ];then
	echo "His name is Mary"
else
	echo "No"
fi
#No
```

数字比较的一些逻辑语句

> ```shell
> comparison    Evaluated to true when
> $a -lt $b    $a < $b
> $a -gt $b    $a > $b
> $a -le $b    $a <= $b
> $a -ge $b    $a >= $b
> $a -eq $b    $a is equal to $b
> $a -ne $b    $a is not equal to $b
> ```

字符串比较的逻辑语句

> ```shell
> comparison    Evaluated to true when
> "$a" = "$b"     $a is the same as $b
> "$a" == "$b"    $a is the same as $b
> "$a" != "$b"    $a is different from $b
> -z "$a"         $a is empty
> ```

### 2.2 case结构

```shell
case var in 
	judgement1) do something;;
	judgement2) do something;;
	judgement3) do something;;
	judgement4) do something;;
```

```shell
my_case=1
case "$my_case" in
	1) echo "hello";;
	2) echo "my name"
esac
#hello
```

在这里要注意与case相对应的是要写esac符号退出，另外每个操作之间要加`;;`两个分号分开。跟变量var有关的判断语句judgement或者condition要写完整，上面如果是判断是否等于则可以不用写完整。

## 三、循环

### 3.1 遍历

基本结构

```shell
for N in array;do
	do something;
done
```

```shell
# loop on array member
NAMES=(Joe Jenny Sara Tony)
for N in ${NAMES[@]} ; do
	let i+=1
	echo "The number ${i} is $N"
done
#The number 1 is Joe
#The number 2 is Jenny
#The number 3 is Sara
#The number 4 is Tony
```

 注意`${NAMES[@]}`是取这个整个阵列的元素集合，而`${NAMES}`是取这个阵列作为整体的一个变量

### 3.2 while循环

while循环的结构

```shell
while condition ; do
	do something
done
```

```shell
count=1
while [ "$count" -lt 4 ] ; do
	echo "The current number is $count"
	let count+=1
done
#The current number is 1
#The current number is 2
#The current number is 3
```

我们还可以增加`break`还有`continue`语句来控制，跟python一样

```shell
count=1
while [ $count -lt 10 ] ; do
	if [ $count -gt 7 ] ; then
		break
	
	elif [ $(($count % 2)) -eq 0 ] ; then
		echo "The current number is $count"
	else
		continue
        
    fi
	count=$(($count+1))
done
```



## 四、shell函数

shell函数的结构

```shell
function function_name {
    commands
}
```



## 五、特殊变量

在shell脚本当中有一些特殊变量

- `$0` - The filename of the current script.|
- `$n` - The Nth argument passed to script was invoked or function was called.|
- `$#` - The number of argument passed to script or function.|
- `$@` - All arguments passed to script or function.|
- `$*` - All arguments passed to script or function.|
- `$?` - The exit status of the last command executed.|
- `$$` - The process ID of the current shell. For shell scripts, this is the process ID under which they are executing.|
- `$!` - The process number of the last background command.|

## 六、trap命令

trap命令来对某个信号进行输出

  `trap <argv>/<function> <signal>`

信号就是用户一些中断程序运行的操作，或者是磁盘满了之类的错误使得程序无法继续往下运行，这样的一些信号就给了程序信号。可以在终端输入`kill -l`查看所有的信号类型。也可以参考tutorialspoint的[这篇文章](https://www.tutorialspoint.com/unix/unix-signals-traps.htm)。

![](https://upload-images.jianshu.io/upload_images/2338511-018530d386a96eb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/2338511-2447b161cb7cda1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们在`traptest.sh`这个文件当中键入以下代码：

```shell
 #!/bin/bash
# notice you cannot make Ctrl-C work in this shell,
# try with your local one, also remeber to chmod +x
# your local .sh file so you can execute it!

trap func SIGINT SIGTERM
echo "it's going to run until you hit Ctrl+Z"
echo "hit Ctrl+C to be blown away!"
function func {
        echo "Booh"

}



while true
do
        sleep 60
done
```

![](https://upload-images.jianshu.io/upload_images/2338511-1a42e437e5c4fe8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们在按下ctrl+c取消运行的时候会有提示。说明这个signal被捕捉到了。

## 七、文件检测

我们还可以用脚本检测文件在当前路径的一些状态。

使用判断语句

`<- command> <filename1>`

`<filename1> <- command> <filename2>`

比如`-e`检测文件是否存在

```shell
#!/bin/bash
filename=$1#这里接收第一个参数作为文件名
if [ -e "$filename" ];then
	echo "$filename exists"
fi
```

我们试一下运行

![](https://upload-images.jianshu.io/upload_images/2338511-848894596c70904a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


类似的，`-d`检测某个路径是否存在，`-r`检测文件是否可读。

