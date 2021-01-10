python当中有个概念叫`property`，这个概念或者说模块我们经常用到，但是这个到底是什么，可能比较少谈论这个。我们不妨以一个例子开始。

# 设置一个温度模块

假设说现在有一家公司想要我们开发一个类，输入温度之后，记录某个地方的温度，并且可以转换成华氏温度

```python
class Celius:
    def __init__(self, temperature):
        self.temperature = temperature

    def to_feb(self):
        return (self.temperature * 1.8) + 32
```

```python
%config ZMQInteractiveShell.ast_node_interactivity='all'
a=Celius(37)
a.temperature
a.to_feb()
```



```
37
```





```
98.60000000000001
```



我们看到，的确输出结果，而且转换成功了。

# 版本2.0 希望对温度设置的时候有限制 

其实我们上面的温度类已经满足要求了，但是客户希望我们能够对输入温度的值有个把控，不希望低于绝对零度$-273^{\circ}$，因此需要对输入值进行一个判断先。

```python
class Celius:
    def __init__(self, temperature):
        self.temperature = self.setter_temperature(temperature)

    def to_feb(self):
        return (self.temperature * 1.8) + 32

    def setter_temperature(self, val):
        if val <= -273:
            raise ValueError(
                'The temperature you input is below absolute zero')
        else:
            self.__temperature=val
```

```python
%config ZMQInteractiveShell.ast_node_interactivity='all'
a=Celius(-300)

```

```
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-11-97b5139ec1b0> in <module>()
      1 get_ipython().run_line_magic('config', "ZMQInteractiveShell.ast_node_interactivity='all'")
----> 2 a=Celius(-300)
```

```
<ipython-input-9-a8e9c60a97bc> in __init__(self, temperature)
      1 class Celius:
      2     def __init__(self, temperature):
----> 3         self.temperature = self.setter_temperature(temperature)
      4 
      5     def to_feb(self):
```

```
<ipython-input-9-a8e9c60a97bc> in setter_temperature(self, temperature)
      9         if temperature <= 273:
     10             raise ValueError(
---> 11                 'The temperature you input is below absolute zero')
     12         else:
     13             return temperature

```

```
ValueError: The temperature you input is below absolute zero

```

在上面我们看到对于输入值先执行了`setter_temperature`函数，对输入值进行了预先判断，然后再返回或者报错。

## 私有变量

但是实际上还是有存在重新赋值不检测的情况出现，比如我先赋值一个合法的值，但重新赋值的时候不管了。

```python
a=Celius(300)
a.temperature
a.to_feb()

```



```
300

```





```
572.0

```



```python
a.temperature=-300
a.temperature

```



```
-300

```



以上就没有报错了，这样是不行的。因此，我们想到，干脆直接让`temperature`这个属性不可见，变成一个私有变量的东东，这样就不会有人手贱去恶作剧乱修改了，而变成私有变量只需要在前面加上两根下划线`__temperature`。不信的话，我们试试看

```python
class Celius:
    def __init__(self, temperature):
        self.__temperature = self.setter_temperature(temperature)

    def to_feb(self):
        return (self.__temperature * 1.8) + 32

    def setter_temperature(self, val):
        if val <= -273:
            raise ValueError(
                'The temperature you input is below absolute zero')
        else:
            self.__temperature=val

```

```python
a=Celius(300)
a.temperature


```

```
---------------------------------------------------------------------------

AttributeError                            Traceback (most recent call last)

<ipython-input-18-f2edb7aca8a1> in <module>()
      1 a=Celius(300)
----> 2 a.temperature

```

```
AttributeError: 'Celius' object has no attribute 'temperature'
```

可以看到我们的确已经没有`temperature`这个属性，但实际上我们可以通过`_ObjectName__temperature`的方式来访问。

```python
a._Celius__temperature
```



```
300
```



这样无论如何，对于不会编程的人来说，想随便不经过校验是否是绝对零度就修改`temperature`属性的可能性就大大降低了。然后，如果我们想修改temperature的属性，就可以通过`setter_temperature`的函数去修改，慢着，可我们怎么返回`temperature`的值呢？所以我们需要新增一个函数来返回`temperature`这个值。于是结果如下：

```python
class Celius:
    def __init__(self, temperature):
        self.__temperature = self.setter_temperature(temperature)

    def to_feb(self):
        return (self.__temperature * 1.8) + 32

    def getter_temperature(self):
        return self.__temperature

    def setter_temperature(self, val):
        if val <= -273:
            raise ValueError(
                'The temperature you input is below absolute zero')
        else:
            self.__temperature=val
```

```python
%config ZMQInteractiveShell.ast_node_interactivity='all'
a=Celius(300)
a.getter_temperature()
```



```
300
```



```python
a.setter_temperature(-300)
```

```
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-27-4631b6ae290e> in <module>()
----> 1 a.setter_temperature(-300)
```

```
<ipython-input-24-70a0b8187f0a> in setter_temperature(self, temperature)
     12         if temperature <= 273:
     13             raise ValueError(
---> 14                 'The temperature you input is below absolute zero')
     15         else:
     16             return temperature
```

```
ValueError: The temperature you input is below absolute zero
```

这下子我们比较安心了，因为用户没有办法通过`a.temperature`这个属性去修改你的温度，只能通过`getter_temperature`去获取温度值，通过`setter_temperature`去校验是否是绝对零度以下。但是这样写是不是有点冗余？因为用户还得记两个函数去分别获取或者是修改值，对于完全不懂编程的用户来说，这个用户体验分数是很低的。那么这个时候就是轮到`property`方法大显身手了。

在这里我们将获取温度和设置温度的`getter_temperature`和`setter_temperature`两个函数作为参数，传入到property这个类当中，返回一个对象，这个对象是既能够通过`a.temperature`的方式返回温度值，也能通过`a.temperature=-250`的方式先检验值再设置值。这就实现了**简便**和**校验**的双重便利。

```python
class Celius:
    def __init__(self, temperature):
        self.__temperature = temperature

    def to_feb(self):
        return (self.__temperature * 1.8) + 32

    def getter_temperature(self):
        return self.__temperature

    def setter_temperature(self, val):
        if val <= -273:
            raise ValueError(
                'The temperature you input is below absolute zero')
        else:
            self.__temperature=val

    temperature = property(getter_temperature, setter_temperature)

```

```python
%config ZMQInteractiveShell.ast_node_interactivity='all'
a=Celius(300)
a.temperature

```



```
300

```



```python
a.temperature=-300

```

```
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-30-c50d65fc7ecf> in <module>()
----> 1 a.temperature=-300

```

```
<ipython-input-28-f6cfb4a4923a> in setter_temperature(self, temperature)
     12         if temperature <= 273:
     13             raise ValueError(
---> 14                 'The temperature you input is below absolute zero')
     15         else:
     16             return temperature

```

```
ValueError: The temperature you input is below absolute zero

```

在上面我们可以看到，`a.temperature`的`temperature`其实是来自于类当中`property`返回的对象，而不是其他的`temperature`。`property`对象有三个方法可以使用，分别是`getter`，`setter`，`delete`，这三个方法分别是在**返回值**，**设置值**，**删除值**的时候进行一些操作。

在这里就牵扯到了**数据封装**的概念，其实**数据封装**概念很好理解，其实就是数据和对数据的操作其实是黑匣子，你用`a.temperatue=300`这样的方式去赋值的时候并没有想到后面还有这么多函数，你还以为是简简单单地这么搞出来。所以的话，数据封装的概念就是，**将数据和对数据的操作绑定起来的这个机制**。这些操作就是上面说到的那三种，分别是`getter`，`setter`，`delete`。

我们也可以用另外一种方式来传入参数。

```python
temperature=property()
temperature=temperature.getter(getter_temperature())
temperature=temperature.setter(setter_temperature())

```

# @property语法糖

上面的类定义当中的最后一步`temperature = property(getter_temperature, setter_temperature)`其实可以用语法糖来生成。可以改成：

```python
class Celius:
    def __init__(self, temperature):
        self.__temperature = temperature

    def to_feb(self):
        return (self.__temperature * 1.8) + 32
    @property
    def temperature(self):
        return self.__temperature
    @temperature.setter
    def temperature(self, val):
        if val <= -273:
            raise ValueError(
                'The temperature you input is below absolute zero')
        else:
            self.__temperature=val

    

```

```python
%config ZMQInteractiveShell.ast_node_interactivity='all'
a=Celius(300)
a.temperature

```



```
300

```



```python
a.temperature=-300

```

```
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-60-c50d65fc7ecf> in <module>()
----> 1 a.temperature=-300

```

```
<ipython-input-58-9812e4d2cbb9> in temperature(self, val)
     12         if val <= -273:
     13             raise ValueError(
---> 14                 'The temperature you input is below absolute zero')
     15         else:
     16             self.__temperature=val

```

```
ValueError: The temperature you input is below absolute zero

```



```python
a.temperature=200
a.temperature
```



```
200

```



`@property`将下面的函数变成了一个property对象，跟之前的写法的作用是一样的

```python
temperature=property()
temperature=temperature.getter(temperature)

```

这里的第一行`temperature`是`property`对象，第二行中括号里面的`temperature`是返回`temperature`的那个函数，它作为参数传入`property`对象中。

以上就是`property`的来龙去脉，简单说，就是`property`具有返回和修改值的方法，能够使得修改值和返回值写起来更加便利。
