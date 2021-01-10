# 数据聚合与分组操作
pandas的聚合与分组操作是日常进行数据分析要用到的常用操作，相信很多人都很熟悉了。但是聚合分析能做到的远不止只是简单地计算组内的平均值，中位数等统计学数值。`groupby`可以对组内执行其他变换，比如计算分位数的排名，标准化，线性回归，排位和子集选择，计算分位数分析和区间分析等等，只要你能想到的，返回pandas对象或者标量的操作都可以。


# pandas聚合分析的原理
![](https://upload-images.jianshu.io/upload_images/2338511-9c4d5213fa0c60af.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 解析
pandas执行`groupby`操作以后就会按照键进行分组，从上面那个图可以看到，执行按照`key`字段进行分组后，相同的`key`就会分到一组，这个时候组内进行某个聚合函数`aggfunc`的操作后再将得到的结果纵向拼接到一起，得到最后的结果。因此，明白了`groupby`的过程之后，我们大概也就明白，自定义的函数只要是能够对一个dataframe进行操作，并且返回一个pandas对象或者一个标量的，都是可以用这样一个函数的。
```python
df = pd.DataFrame({'key': ['a', 'b', 'c'] * 4,
                   'value': numpy.arange(12.)})
df
```
![](https://upload-images.jianshu.io/upload_images/2338511-7914b1bfa07c9666.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
g=df.groupby('key')
```
很多人不知道的是，pandas聚合后的对象其实是一个键值对的对象。
```python
# 我们试一下
for i, v in g:
    print(i, v)
```

```python
a   key  value  value_1
0   a    0.0       44
3   a    3.0       15
6   a    6.0       20
9   a    9.0        6
b    key  value  value_1
1    b    1.0        4
4    b    4.0       29
7    b    7.0       10
10   b   10.0       40
c    key  value  value_1
2    c    2.0       12
5    c    5.0        7
8    c    8.0       38
11   c   11.0        0
```

`a`, `b`和`c`分别是各个分组的分组名，对应的值是各个分组的`dataframe`。
# 自定义一个函数进行组内和组间的计算
```python
# 新建一个dataframe
df = pd.DataFrame({'key1' : ['a', 'a', 'b', 'b', 'a'],
                   'key2' : ['one', 'two', 'one', 'two', 'one'],
                   'data1' : numpy.random.randn(5),
                   'data2' : numpy.random.randn(5)})
df
```
![](https://upload-images.jianshu.io/upload_images/2338511-1dcb11007b627cb6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们新建一个函数用作后面对group对象进行的计算。
```python
def get_result(group):
    return (group['data1'].sum()-group['data2'].mean())/group['data1'].sum()
```
按照`key1`进行分组。
```python
df.groupby('key1').apply(get_result)
```
![](https://upload-images.jianshu.io/upload_images/2338511-0ae09b3db5937307.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 如果`apply`里面的函数本身就有聚合操作返回一个标量的话，那么聚合应用该函数后返回的就是具体的值，而不是扩展到所有行，比如上面的`sum`是在轴方向进行加和，也就是组内进行操作，而不是单个操作，因此apply后返回的是数值。

如果我们要做的计算是对分组内每个单元格进行的，那么返回的就是跟原本的dataframe一样大小的。
```python
def get_result_single(group):
    return (group['data1']-group['data2'].mean())/group['data1'].sum()
    
    
df.groupby('key1').apply(get_result_single)
```
![](https://upload-images.jianshu.io/upload_images/2338511-66c4945127384132.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 在apply中传入自定义的函数的参数

很多时候我们总是习惯在`group`后对某一列进行操作，但很多时候我们忘了，其实我们可以直接对group后的对象进行一个复杂函数操作，这个时候建议先把函数写好，后面再调用，且记得**`apply`里面可以传入该函数的参数**
```python
tips = pd.read_csv('examples/tips.csv')
tips.head(3)
```
![](https://upload-images.jianshu.io/upload_images/2338511-5a5836acc532cbb2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这里我们尝试按照`smoker`这一列进行分组，并且排序找出分组后的`total_bill`前5位最高的记录，为了更加熟悉`apply`该函数可以传入自定义函数的参数的这个功能，我们自定义函数可以设定返回**第几位**，**降序或者逆序**。

```python
def top(group, col, n=5, ascending=True):
    return group.sort_values(by=col,ascending=ascending)[:n]
    
tips.groupby('smoker').apply(top, col='total_bill', n=5, ascending=True)
```
![](https://upload-images.jianshu.io/upload_images/2338511-9ec33a27d1c8e50b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当然，根据自定义函数，我们还可以修改不同的参数，这样构建好一个函数，我们可以重复调用，这样既提高了效率，也增加了代码的可读性。

```python
tips.groupby('smoker').apply(top, col='tip', n=3, ascending=False)
```
![](https://upload-images.jianshu.io/upload_images/2338511-dd3d8390321b45fa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上我们就按照组内的`tip`列进行降序排列，并筛选出前3行。
# 分组内自定义填充缺失值
我们有很多时候需要根据分组后不同的组别进行填充相应的值，在这里我们可以用**自定义函数**和**字典**的方式进行自定义。
我们把之前的`df`的2, 4行改成缺失值，我们尝试用分组内的平均值来对应填充，使用`fillna`方法
```python
df.iloc[[1,3],[-2,-1]]=numpy.nan
df
```
![](https://upload-images.jianshu.io/upload_images/2338511-f3bd77953d77caec.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```python
def fill_mean(group):
    return group.fillna(df.mean())

df.groupby('key1').apply(fill_mean)
```
![](https://upload-images.jianshu.io/upload_images/2338511-989fa48015de2cde.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


以上我们填充的是组内的平均数，也就是调用了`mean()`方法，但是如果我们要填充任意自定义的值的话，需要指定**分组名**和**填充值**的字典，然后用**group对象**的`name`属性去字典取出对应的值。
```python
fill_val={'a':-1, 'b':12}
fill_med=lambda g:g.fillna(fill_val[g.name])
df.groupby('key1').apply(fill_med)
```
[站外图片上传中...(image-959b14-1566618910459)]

`g.name`相当于是分组后的索引的分组名,然后我们在字典中根据分组名去填充对应的值。

# 分箱后进行聚合
使用`pd.cut`和`pd.qcut`后，得到的是一个`categorical`对象，这个对象就像是一个列表或者`series`，可以传入给`group`对象后面进行分组的依据。
```python
tips.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-06d114c296a6d331.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果我们尝试对total_bill进行分箱，然后分组
```python
bins=pd.cut(tips.total_bill,bins=10)#返回一个series，这个series可以传入到groupby当中作为一个分组的依据
tips.groupby(bins).agg({'tip':'mean'})
```
![](https://upload-images.jianshu.io/upload_images/2338511-0b2723978f4ccc64.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 按照字典或者列表来聚合
```python
people = pd.DataFrame(numpy.random.randn(5, 5),
                      columns=['a', 'b', 'c', 'd', 'e'],
                      index=['Joe', 'Steve', 'Wes', 'Jim', 'Travis'])
people.iloc[2:3, [1, 2]] = numpy.nan # Add a few NA values
people
```
![](https://upload-images.jianshu.io/upload_images/2338511-7c0ab829512d29ab.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们还可以传入一个字典，字典的名对应列名或者索引名，值代表新的分类，按照新的分类进行聚合。
```python
mapping={'a':'good', 'b':'good', 'c':'bad', 'd':'good', 'e':'bad'}
people.groupby(mapping, axis=1).agg('mean')
```
![](https://upload-images.jianshu.io/upload_images/2338511-81af5820328a2026.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然我们也可以传入列表作为聚合的依据，类似于前面的`categorical`操作，需要注意的是，列表的长度需要和长度或者宽度一致。比如我们这里有5行，我们按照行方向进行聚合的话，创建一个5个元素的列表，按照这个列表里面的值进行聚合。
```python
label=['west']*3+['east']*2
label
```
![](https://upload-images.jianshu.io/upload_images/2338511-d558956928a17a77.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```python
people.groupby(label).agg({'a':'mean'})
```
![](https://upload-images.jianshu.io/upload_images/2338511-3a2b28b32c799252.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 两个例子
## 聚合后计算相关性
如果有这种情况，我们希望能够分组，并且分组后计算其他列与某列的相关性，比如在下面，我们想计算其他变量`total_bill`，`size`与`tip`的相关性大小，我们可以考虑下怎么做。
```python
tips.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-742a66433dc2376d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通常来说，`series`对`series`的单个相关性可以用`series`的内置方法`corr`来计算，如下：
```python
tips['total_bill'].corr(tips['tip'])
```
![](https://upload-images.jianshu.io/upload_images/2338511-5e8a48ff1aba47d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`dataframe`对`series`的单个相关性可以用`dataframe`的内置方法`corrwith`来计算。
```python
tips[['total_bill', 'tip']].corrwith(tips['size'])
```
 ![](https://upload-images.jianshu.io/upload_images/2338511-883820061ab7e138.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


现在加上分组
```python
tips.groupby('smoker').apply(lambda x:x[['total_bill', 'tip']].corrwith(x['size']))
```

![](https://upload-images.jianshu.io/upload_images/2338511-afea1f81b284a803.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 聚合后计算线性回归
正如之前所说，只要是能够返回pandas对象的函数都是可以用到`apply`函数中的，`statsmodels`的线性回归模块也不例外，返回的结果的`params`参数就是一个`series`。

```python
import statsmodels.api as sm

def regression(group, x_val, y_val):
    x=group[x_val]
    x['intercept']=1
    Y=group[y_val]
    model=sm.OLS(Y,x)
    results=model.fit()
    return results.params

tips.groupby('smoker').apply(regression, x_val=['total_bill'], y_val='tip')
```
![](https://upload-images.jianshu.io/upload_images/2338511-c339e2906edcfb8f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 在列方向进行聚合
我们还可以在列方向进行聚合，如果是多层索引，指定索引名，按照对应索引名进行聚合。

```python
columns = pd.MultiIndex.from_arrays([['US', 'US', 'US', 'JUPYTER NOTEBOOK', 'JUPYTER NOTEBOOK'],
                                    [1, 3, 5, 1, 3]],
                                    names=['cty', 'tenor'])
hier_df = pd.DataFrame(numpy.random.randn(4, 5), columns=columns)
hier_df
```
![](https://upload-images.jianshu.io/upload_images/2338511-58cfb51a22249a2e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
hier_df.groupby(level='cty',axis=1).mean()
```
![](https://upload-images.jianshu.io/upload_images/2338511-bc92cf7682c3a797.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
hier_df.groupby(level='tenor',axis=1).mean()
```
![](https://upload-images.jianshu.io/upload_images/2338511-b8b5139da85f7b24.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 总结
虽然以上似乎介绍了很多pandas聚合操作的用法，但我细细想起来，其实是再简单不过的操作罢了，关于想要玩好pandas的聚合操作，有这么以下几点记住即可。
1. pandas分组后的是有`name`属性的分组对象，聚合函数是对各个分组的dataframe进行操作的，只要是能够返回一个pandas对象或者标量的函数都是可以应用到groupby以后的聚合里面的。
2. pandas传入`groupby`里面的参数可以是`series`, `list`, `dict`和`categorical`。前面都比较好理解，`categorical`是pandas的另外一种类型，目前主要应用在分桶上。
3. pandas既可以按照组内的值进行聚合，也可以按照多层索引的层级进行聚合，传入的参数是`level='level_name'`；既可以在行方向进行聚合(默认)，也可以在列方向进行聚合(传入`axis=1`)的操作。
4. 对于一个复杂一些的函数，可以先把函数和相关的参数写好，**建议把常用的函数直接保存到剪贴板增强工具里面**，后面再进行重复的调用，这样效率很高，而且代码很整洁。




只要理解上面这么几点，基本上都可以把`groupby`用得比较遛了。个人到现在觉得，之所以要把各种数据分析和数据挖掘的函数用法钻研得深透，并不是为了geek而geek，相反，是为了更好更快地探索数据，需要用的时候几行代码就实现，丝毫察觉不到这个过程实现的曲折，不需要使用的时候再疯狂去谷歌，这样效率和体验都很差。
