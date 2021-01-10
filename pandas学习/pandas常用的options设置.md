

## 将小数设置为百分比的形式

```python
pd.set_option('display.float_format', '{:.2%}'.format)
```



我们可以先对单个数值进行设置

```python
'{:.2%}'.format(0.0355)
```

![输出1](https://upload-images.jianshu.io/upload_images/2338511-fb65ec57b4e4191c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/2338511-22804cb9c44376ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个设置适合于需要把浮点数设置为百分比的形式。



## 设置显示的行数

很多时候我们可能并不需要查看所有的行数，我们可以设置`display.max_rows`来设置显示的最大行数

```python
pd.set_option('display.max_rows', 5)
```

这样就限制调用df的时候最大行数为5

其他常用的一些设置：

| Sr.No |                   Parameter & Description                    |
| :---: | :----------------------------------------------------------: |
|   1   | **display.max_rows**:  Displays maximum number of rows to display |
|   2   | **display.max_columns**:  Displays maximum number of columns to display |
|   3   | **display.expand_frame_repr**:  Displays DataFrames to Stretch Pages |
|   4   |   **display.max_colwidth**:  Displays maximum column width   |
|   5   | **display.precision**:  Displays precision for decimal numbers |

通常来说，pandas用到的有关设置的方法如下：

|      参数名       |           作用           |
| :---------------: | :----------------------: |
|   get_option()    |       获得设置参数       |
|   set_option()    |         设置参数         |
|  reset_option()   |         重设参数         |
| describe_option() |    查看设置的参数列表    |
| option_context()  | 临时在某个段落内设置参数 |
