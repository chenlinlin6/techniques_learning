

```python
# 导入模块
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import os
import seaborn as sns
import logging
from tqdm import tqdm
import warnings 
warnings.filterwarnings('ignore')
%matplotlib inline
plt.rc('font',family='SimHei', size=13)
# os.chdir('')
```

# pandas本身对字符串的操作

`pandas`文本字符串的列本身就可以调用`str`属性里面的方法，跟python文本字符串本身带有的方法是一致的


```python
movie_lens=pd.read_table('tmdb-movies.csv', sep=',')
movie_lens.head(2)
```

![](https://upload-images.jianshu.io/upload_images/2338511-981b329ac9111a59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





```python
movie_lens['genres'].str.contains('Action').head(3)
```




    0     True
    1     True
    2    False
    Name: genres, dtype: object




```python
movie_lens['genres'].str.find('Action').head(3)
```




    0    0.0
    1    0.0
    2   -1.0
    Name: genres, dtype: float64




```python
movie_lens['genres'].str.split('|', expand=True).head(3)
```


![](https://upload-images.jianshu.io/upload_images/2338511-c123bfc8580c6e15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



获得指定位置的字符串


```python
movie_lens['original_title'].str.get(1).head(3)
```




    0    u
    1    a
    2    n
    Name: original_title, dtype: object



字符串重复三遍


```python
df.head(3)
```
![](https://upload-images.jianshu.io/upload_images/2338511-0cc878b3ffaca912.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




```python
df.key.str.repeat(3)
```




    0     aaa
    1     bbb
    2     ccc
    3     aaa
    4     bbb
    5     ccc
    6     aaa
    7     bbb
    8     ccc
    9     aaa
    10    bbb
    11    ccc
    Name: key, dtype: object



`find`, `findall`, `extract`等可以结合正则表达式去用


```python
df.key.str.title().head(3)
```




    0    A
    1    B
    2    C
    Name: key, dtype: object



# pandas对时间格式的文本的转化


```python
movie_lens.columns
```




    Index(['id', 'imdb_id', 'popularity', 'budget', 'revenue', 'original_title',
           'cast', 'homepage', 'director', 'tagline', 'keywords', 'overview',
           'runtime', 'genres', 'production_companies', 'release_date',
           'vote_count', 'vote_average', 'release_year', 'budget_adj',
           'revenue_adj'],
          dtype='object')




```python
movie_lens.release_date.head(3)
```




    0     6/9/15
    1    5/13/15
    2    3/18/15
    Name: release_date, dtype: object



**python中时间日期格式化符号:**  

> 
|format|meaning|
|:----:|:----:|
|%y| 两位数的年份表示（00-99）  |
|%Y| 四位数的年份表示（000-9999）  |
|%m| 月份（01-12）  |
|%d| 月内中的一天（0-31）  |
|%H| 24小时制小时数（0-23）  |
|%I| 12小时制小时数（01-12）  |
|%M| 分钟数（00=59）  |
|%S| 秒（00-59）  |
|%a| 本地简化星期名称  |
|%A| 本地完整星期名称  |
|%b| 本地简化的月份名称  |
|%B| 本地完整的月份名称  |
|%c| 本地相应的日期表示和时间表示  |
|%j| 年内的一天（001-366）  |
|%p| 本地A.M.或P.M.的等价符  |
|%U| 一年中的星期数（00-53）星期天为星期的开始  |
|%w| 星期（0-6），星期天为星期的开始  |
|%W| 一年中的星期数（00-53）星期一为星期的开始  |
|%x| 本地相应的日期表示  |
|%X| 本地相应的时间表示  |
|%Z| 当前时区的名称  |
|%%| %号本身  |

我们可以使用`pandas.to_datetime`对日期列进行对应格式的转换，只需要传入一个对应的参数`format`附上对应的格式即可，如对电影数据集的`release_date`进行格式化


```python
pd.to_datetime(movie_lens.release_date, format='%m/%d/%y').head(10)
```




    0   2015-06-09
    1   2015-05-13
    2   2015-03-18
    3   2015-12-15
    4   2015-04-01
    5   2015-12-25
    6   2015-06-23
    7   2015-09-30
    8   2015-06-17
    9   2015-06-09
    Name: release_date, dtype: datetime64[ns]



# pandas对分组后的高阶操作

我们先加载一个`df`


```python
df = pd.DataFrame({'key': ['a', 'b', 'c'] * 4,
                   'value': np.arange(12.)})
df
```
||key|	value|
|:----:|:----:|:----:|
|0|	a|	0.0|
|1|	b|	1.0|
|2|	c|	2.0|
|3|	a|	3.0|
|4|	b|	4.0|
|5|	c|	5.0|
|6|	a|	6.0|
|7|	b|	7.0|
|8|	c|	8.0|
|9|	a|	9.0|
|10|	b|	10.0|
|11|	c|	11.0|





```python
df['value_1']=np.random.randint(low=0, high=50,size=len(df))
```


```python
df
```

||key|	value|value_1|
|:----:|:----:|:----:|:---:|
|0|	a|	0.0|	44|
|1|	b|	1.0|	4|
|2|	c|	2.0|	12|
|3|	a|	3.0|	15|
|4|	b|	4.0|	29|
|5|	c|	5.0|	7|
|6|	a|	6.0|	20|
|7|	b|	7.0|	10|
|8|	c|	8.0|	38|
|9|	a|	9.0|	6|
|10|	b|	10.0|	40|
|11|	c|	11.0|	0|



```python
g=df.groupby('key')

```

很多人不知道的是，pandas聚合后的对象其实是一个键值对的对象


```python
# 我们试一下
for i, v in g:
    print(i, v)
```

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


## 三种聚合写法  
`pandas`具有三种聚合写法。
1. 一种是聚合后直接用某个函数比如`mean()`，`max()`等聚合函数进行聚合
2. 一种是聚合后用`apply`方法来传入一个自定义函数来获得相应的聚合结果
3. 还有就是用`agg(dict)`的方法，agg里面传入一个字典，其中key是操作的列名，value是对应的聚合函数，既可以是通用的方法的字符串，比如`'mean'`，也可以是自定义的函数对象，比如`np.mean()`或者是`lambda x:x.mean()`，好处是不同列可以一次过传入不同的函数进行不同的操作

我们试一下用以上三种方法分别对dataframe `df`进行操作


```python
g['value'].mean()
```




    key
    a    4.5
    b    5.5
    c    6.5
    Name: value, dtype: float64




```python
g.agg({'value':np.mean, 'value_1':np.max})
```

![](https://upload-images.jianshu.io/upload_images/2338511-7a09d6fdf51922f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




```python
g.agg({'value':'mean', 'value_1':'max'})
```
![](https://upload-images.jianshu.io/upload_images/2338511-7a09d6fdf51922f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
g.agg({'value':lambda x:np.power(x.mean(),2), 'value_1':'max'})
```


![](https://upload-images.jianshu.io/upload_images/2338511-7a09d6fdf51922f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
city=['washington', 'new york', 'boston']
df['city']=np.random.choice(city, size=len(df))
df.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-3df612cb8724daeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 案例一  
按照`key`进行分组，统计组内去重后`city`列计数的结果，同时计算出`value_1`列的平均值


```python
g=df.groupby('key')

```


```python
g.agg({'city':lambda x:x.nunique(), 'value_1':'mean'})
```


![](https://upload-images.jianshu.io/upload_images/2338511-cd6b974bc211dc6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



可以对比下去重前后的差别


```python
g.agg({'city':lambda x:x.count(), 'value_1':'mean'})
```

| key   |   city |   value_1 |
|:------|-------:|----------:|
| a     |      4 |     21.25 |
| b     |      4 |     20.75 |
| c     |      4 |     14.25 |


## 聚合后进行apply方法


```python
g.apply(lambda df:df.sort_values(by='value_1',ascending=False))
```



|           | key   |   value |   value_1 | city       |
|:----------|:------|--------:|----------:|:-----------|
| ('a', 0)  | a     |       0 |        44 | washington |
| ('a', 6)  | a     |       6 |        20 | new york   |
| ('a', 3)  | a     |       3 |        15 | boston     |
| ('a', 9)  | a     |       9 |         6 | new york   |
| ('b', 10) | b     |      10 |        40 | boston     |
| ('b', 4)  | b     |       4 |        29 | boston     |
| ('b', 7)  | b     |       7 |        10 | boston     |
| ('b', 1)  | b     |       1 |         4 | new york   |
| ('c', 8)  | c     |       8 |        38 | boston     |
| ('c', 2)  | c     |       2 |        12 | boston     |
| ('c', 5)  | c     |       5 |         7 | new york   |
| ('c', 11) | c     |      11 |         0 | washington |



## 组内聚合后进行广播

很多时候我们想对一组内的值进行聚合操作后再广播到相应的大小，这个时候可以用`transform`或者`apply`方法


```python
g['value'].transform(lambda x:(x-x.mean())/x.std())
```




    0    -1.161895
    1    -1.161895
    2    -1.161895
    3    -0.387298
    4    -0.387298
    5    -0.387298
    6     0.387298
    7     0.387298
    8     0.387298
    9     1.161895
    10    1.161895
    11    1.161895
    Name: value, dtype: float64




```python
g['value'].transform('mean')
```




    0     4.5
    1     5.5
    2     6.5
    3     4.5
    4     5.5
    5     6.5
    6     4.5
    7     5.5
    8     6.5
    9     4.5
    10    5.5
    11    6.5
    Name: value, dtype: float64




```python
g['value'].apply(lambda x:(x-x.mean())/x.std())
```




    0    -1.161895
    1    -1.161895
    2    -1.161895
    3    -0.387298
    4    -0.387298
    5    -0.387298
    6     0.387298
    7     0.387298
    8     0.387298
    9     1.161895
    10    1.161895
    11    1.161895
    Name: value, dtype: float64



## 用pivot_table

用pivot_table可以实现聚合相同的功能


```python
df.head(3)
```


|    | key   |   value |   value_1 | city       |
|---:|:------|--------:|----------:|:-----------|
|  0 | a     |       0 |        44 | washington |
|  1 | b     |       1 |         4 | new york   |
|  2 | c     |       2 |        12 | boston     |



```python
pd.pivot_table(df, values='value_1', index='city', aggfunc='mean', fill_value=1)
```



| city       |   value_1 |
|:-----------|----------:|
| boston     |     24    |
| new york   |      9.25 |
| washington |     22    |

类似于这样，index就是要group的列，value_1相当于是要用来计算的列，aggfunc就是使用的聚合函数，fill_value就是如果对应位置为空要填充的值

# 对应替换操作

我们在`pandas`的实际数据分析中，需要按照对应值去填充空值或者是替换值，我们可以用到`map`的技巧，也就是传入一个字典去对应填充或者替换

比如说，我们在这里按照  
washington ----> 1  
new york ----> 2  
boston----> 3  


```python
city_dict={ 'washington':1, 'new york':2, 'boston':3 } 
df['city'].map(city_dict)
```




    0     1
    1     2
    2     3
    3     3
    4     3
    5     2
    6     2
    7     3
    8     3
    9     2
    10    3
    11    1
    Name: city, dtype: int64



可以看到传入一个字典到`map`当中以后，相应的值就会被替换成字典里面对应的键的值

pandas本身的`map`和`filter`也是相当好用的函数


```python
a=list(range(5))
# map函数
print(list(map(lambda x:x**2,a )))
```

    [0, 1, 4, 9, 16]



```python
# 筛选出是偶数的元素
print(list(filter(lambda x:x%2==0,a )))
```

    [0, 2, 4]


# 其他超级好用的模块

## pandas_profiling 

`pandas_profiling`这个模块用来前期的探索性分析是最好不过了，我们常用的会用`head`, `info`, `describe`等方法来实现对某个数据集的概览，而`pandas_profiling`则一步到位，直接实现了我们的所有幻想，并且生成的report还支持`to_file`方法直接导出为`html`文件，实在是省时省力，用来给老板汇报十分专业


```python
import pandas_profiling as pp
pp.ProfileReport(movie_lens)
```



![](https://upload-images.jianshu.io/upload_images/2338511-68703d1db2548724.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




```python
report=pp.ProfileReport(movie_lens)
```


```python
# 将生成的侧写报告导出为html文件
report.to_file('report.html')
```

## scatter_matrix


```python
from pandas.plotting import scatter_matrix
```


```python
scatter_matrix(movie_lens,figsize=(18, 18))
plt.show()
```


![](https://upload-images.jianshu.io/upload_images/2338511-66791932c1d302c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`scatter_matrix`可以绘制出多个变量的组合后的关系，从中我们看出一些比较明显的相关关系，进而去进一步探究它们的相关关系，比如上图的`revenue`和`budget`就存在明显的相关关系

**一些相关的参数：**  
>1。frame，pandas dataframe对象   
2。alpha， 图像透明度，一般取(0,1]   
3。figsize，以英寸为单位的图像大小，一般以元组 (width, height) 形式设置   
4。ax，可选一般为none   
5。diagonal，必须且只能在{‘hist’, ‘kde’}中选择1个，’hist’表示直方图(Histogram plot),’kde’表示核密度估计(Kernel Density Estimation)；该参数是scatter_matrix函数的关键参数   
6。marker。Matplotlib可用的标记类型，如’.’，’,’，’o’等   
7。density_kwds。(other plotting keyword arguments，可选)，与kde相关的字典参数   
8。hist_kwds。与hist相关的字典参数   
9。range_padding。(float, 可选)，图像在x轴、y轴原点附近的留白(padding)，该值越大，留白距离越大，图像远离坐标原点   
10。kwds。与scatter_matrix函数本身相关的字典参数   
11。c。颜色  


## sns.countplot

`sns.countplot`可以迅速对一列中的元素进行计数并且绘图，节省了pandas中的统计后再绘图的两步走，节省了功夫


```python
sns.countplot(df.city)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1a23e534e0>



![](https://upload-images.jianshu.io/upload_images/2338511-ac1460827d2c1601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





目前先写这么多了，接下来在工作中如果遇到新的问题和好的解决方法会继续分享
