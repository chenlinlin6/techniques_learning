
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
```

```python
import seaborn as sns
```

```python
np.random.seed(30)
```

```python
x=np.random.normal(0,1,1000)
y=np.random.normal(0,1,1000)
```

### 六角图

```python
sns.jointplot(x=x,y=y,kind='hex')
```



```
<seaborn.axisgrid.JointGrid at 0x1a1fb97978>
```



![](http://upload-images.jianshu.io/upload_images/2338511-167778a030e9f391.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

六角图可以显示出点集中的区域

```python
plt.rcParams['figure.figsize']=(6,6)
```

```python
import warnings
warnings.simplefilter('error', 'UserWarning')
```

### 密度分布图

```python
sns.set()
ax=plt.subplot(111)
sns.kdeplot(x,y,ax=ax,color='m')
# sns.jointplot(x,y,kind='kde')
sns.rugplot(x, ax=ax,color='g')
sns.rugplot(y, vertical=True,ax=ax)
# plt.grid(True)
```



```
<matplotlib.axes._subplots.AxesSubplot at 0x1a219b3cf8>
```



![](http://upload-images.jianshu.io/upload_images/2338511-d680f65b7333da77.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
iris=sns.load_dataset('iris')
```

### PairPlot绘制出多个变量两两组合的绘图

```python
help(sns.pairplot)
```

```
Help on function pairplot in module seaborn.axisgrid:

pairplot(data, hue=None, hue_order=None, palette=None, vars=None, x_vars=None, y_vars=None, kind='scatter', diag_kind='auto', markers=None, height=2.5, aspect=1, dropna=True, plot_kws=None, diag_kws=None, grid_kws=None, size=None)
    Plot pairwise relationships in a dataset.
    
    By default, this function will create a grid of Axes such that each
    variable in ``data`` will by shared in the y-axis across a single row and
    in the x-axis across a single column. The diagonal Axes are treated
    differently, drawing a plot to show the univariate distribution of the data
    for the variable in that column.
    
    It is also possible to show a subset of variables or plot different
    variables on the rows and columns.
    
    This is a high-level interface for :class:`PairGrid` that is intended to
    make it easy to draw a few common styles. You should use :class:`PairGrid`
    directly if you need more flexibility.
    
    Parameters
    ----------
    data : DataFrame
        Tidy (long-form) dataframe where each column is a variable and
        each row is an observation.
    hue : string (variable name), optional
        Variable in ``data`` to map plot aspects to different colors.
    hue_order : list of strings
        Order for the levels of the hue variable in the palette
    palette : dict or seaborn color palette
        Set of colors for mapping the ``hue`` variable. If a dict, keys
        should be values  in the ``hue`` variable.
    vars : list of variable names, optional
        Variables within ``data`` to use, otherwise use every column with
        a numeric datatype.
    {x, y}_vars : lists of variable names, optional
        Variables within ``data`` to use separately for the rows and
        columns of the figure; i.e. to make a non-square plot.
    kind : {'scatter', 'reg'}, optional
        Kind of plot for the non-identity relationships.
    diag_kind : {'auto', 'hist', 'kde'}, optional
        Kind of plot for the diagonal subplots. The default depends on whether
        ``"hue"`` is used or not.
    markers : single matplotlib marker code or list, optional
        Either the marker to use for all datapoints or a list of markers with
        a length the same as the number of levels in the hue variable so that
        differently colored points will also have different scatterplot
        markers.
    height : scalar, optional
        Height (in inches) of each facet.
    aspect : scalar, optional
        Aspect * height gives the width (in inches) of each facet.
    dropna : boolean, optional
        Drop missing values from the data before plotting.
    {plot, diag, grid}_kws : dicts, optional
        Dictionaries of keyword arguments.
    
    Returns
    -------
    grid : PairGrid
        Returns the underlying ``PairGrid`` instance for further tweaking.
    
    See Also
    --------
    PairGrid : Subplot grid for more flexible plotting of pairwise
               relationships.
    
    Examples
    --------
    
    Draw scatterplots for joint relationships and histograms for univariate
    distributions:
    
    .. plot::
        :context: close-figs
    
        >>> import seaborn as sns; sns.set(style="ticks", color_codes=True)
        >>> iris = sns.load_dataset("iris")
        >>> g = sns.pairplot(iris)
    
    Show different levels of a categorical variable by the color of plot
    elements:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, hue="species")
    
    Use a different color palette:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, hue="species", palette="husl")
    
    Use different markers for each level of the hue variable:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, hue="species", markers=["o", "s", "D"])
    
    Plot a subset of variables:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, vars=["sepal_width", "sepal_length"])
    
    Draw larger plots:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, height=3,
        ...                  vars=["sepal_width", "sepal_length"])
    
    Plot different variables in the rows and columns:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris,
        ...                  x_vars=["sepal_width", "sepal_length"],
        ...                  y_vars=["petal_width", "petal_length"])
    
    Use kernel density estimates for univariate plots:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, diag_kind="kde")
    
    Fit linear regression models to the scatter plots:
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, kind="reg")
    
    Pass keyword arguments down to the underlying functions (it may be easier
    to use :class:`PairGrid` directly):
    
    .. plot::
        :context: close-figs
    
        >>> g = sns.pairplot(iris, diag_kind="kde", markers="+",
        ...                  plot_kws=dict(s=50, edgecolor="b", linewidth=1),
        ...                  diag_kws=dict(shade=True))
```



```python
sns.pairplot(iris,kind='reg',diag_kind='kde',markers='o')
```



```
<seaborn.axisgrid.PairGrid at 0x1a2145a4a8>
```



![](http://upload-images.jianshu.io/upload_images/2338511-888715ad8102f096.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PairGrid的绘图原理是先产生$4\times4=16$个数据组合，然后再分别选择对角线和非对角线上的映射形式。

```python
g=sns.PairGrid(iris)

g.map_diag(sns.kdeplot)
g.map_offdiag(sns.kdeplot,n_levels=20)
```



```
<seaborn.axisgrid.PairGrid at 0x1a22dd5d30>
```



![](http://upload-images.jianshu.io/upload_images/2338511-9ff848d7f1ac5834.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 探索变量间的关系

```python
tips=sns.load_dataset('tips')
```

```python
tips.head()
```





散点+线性回归拟合线+95%置信区间

```python
sns.lmplot(data=tips,x='size', y='tip')
```



```
<seaborn.axisgrid.FacetGrid at 0x1a28925b00>
```



![](http://upload-images.jianshu.io/upload_images/2338511-fb5389bc613707b6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看不清楚点的时候，因为x方向上很多点都重合了

方法一：加个抖动

```python
sns.lmplot(data=tips,x='size', y='tip',x_jitter=0.08)
```



```
<seaborn.axisgrid.FacetGrid at 0x1a28b979b0>
```



![](http://upload-images.jianshu.io/upload_images/2338511-12988d4edd65dd12.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
anscombe=sns.load_dataset('anscombe')
```

```python
anscombe.head()
```




我们可以通过设定order的参数来限定用来拟合的次数

```python
sns.lmplot(data=anscombe.query("dataset == 'II'"), x='x', y='y',order=2)
```



```
<seaborn.axisgrid.FacetGrid at 0x1a28c7ed30>
```



![](http://upload-images.jianshu.io/upload_images/2338511-3ab19f667e9e6f60.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果有异常值，需要传入一个`robust=True`的参数来限定不将异常值点也纳入到拟合内

```python
sns.lmplot(data=anscombe.query("dataset == 'III'"), x='x', y='y')
```



```
<seaborn.axisgrid.FacetGrid at 0x1a28c68240>
```



![](http://upload-images.jianshu.io/upload_images/2338511-3a0dc9fc07923f9a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
sns.lmplot(data=anscombe.query("dataset == 'III'"), x='x', y='y', robust=True,ci=None)
```



```
<seaborn.axisgrid.FacetGrid at 0x1a28d267f0>
```



![](http://upload-images.jianshu.io/upload_images/2338511-b22483f3f8a2c729.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
sns.lmplot(data=tips,x='total_bill',y='if_smoker',logistic=True)
```



```
<seaborn.axisgrid.FacetGrid at 0x1a26c0b4a8>
```



![](http://upload-images.jianshu.io/upload_images/2338511-650b3d763dfa5d21.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以设定一个`logistic=True`的参数来拟合二分类问题。

```python
x=np.random.normal(0,1,1000)
y=np.random.normal(1,3,1000)
```

```python
ax=plt.subplot(111)

ax2=ax.twinx()

sns.kdeplot(x,ax=ax)
sns.kdeplot(y,ax=ax2)
```



```
<matplotlib.axes._subplots.AxesSubplot at 0x1a25ddb6d8>
```



![](http://upload-images.jianshu.io/upload_images/2338511-ca43b42181e4c9fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
tips.head()
```



### 回归散点图

```python
sns.lmplot(data=tips,x='total_bill', y='tip',hue='smoker')
```



```
<seaborn.axisgrid.FacetGrid at 0x1a25e60f28>
```



![](http://upload-images.jianshu.io/upload_images/2338511-06ba63e28ce45244.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以传入`hue`参数对数据进行分类别的展示

```python
sns.lmplot(data=tips,x='total_bill', y='tip',hue='day')
```



```
<seaborn.axisgrid.FacetGrid at 0x1a2a3ec940>
```



![](http://upload-images.jianshu.io/upload_images/2338511-41499298282d9223.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**结论：**从上面大致可以看出星期天人们不太愿意给小费

还可以传入`col`和`row`参数进行分子图展示

`height`设置图的高度，aspect设置图的压缩比

```python
sns.lmplot(data=tips,x='total_bill', y='tip',hue='smoker', col='time', row='sex',height=10,aspect=0.7)
```



```
<seaborn.axisgrid.FacetGrid at 0x1a2f38e978>
```



![](http://upload-images.jianshu.io/upload_images/2338511-402eee7e609470cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### swarmplot

`swarmplot`用于分类散点图，避免点的重叠

```python
sns.swarmplot(data=tip,x='day',y='tip')
```



```
<matplotlib.axes._subplots.AxesSubplot at 0x1a27dba208>
```



![](http://upload-images.jianshu.io/upload_images/2338511-84af4057fb22f46b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 小提琴图

箱图+kde图

```python
sns.violinplot(data=tips,x='day',y='tip',hue='sex')
```



```
<matplotlib.axes._subplots.AxesSubplot at 0x1a275cee10>
```



![](http://upload-images.jianshu.io/upload_images/2338511-664cf97f2b2f19a8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

非对称小提琴图

非对称小提琴图适用于两种类别的hue同时画在左右两边一起对比

```python
sns.violinplot(data=tips,x='day',y='tip',split=True,hue='sex',inner='stick')
```



```
<matplotlib.axes._subplots.AxesSubplot at 0x1a2636fdd8>
```



![](http://upload-images.jianshu.io/upload_images/2338511-e04a333a518f4ab9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 灰度柱状图

类似于`pandas`的`value_counts()`函数对某个列进行分类统计后绘制的图。

```python
sns.countplot(tips.smoker)
plt.legend(('Yes', 'No'))
```



```
<matplotlib.legend.Legend at 0x1a26b898d0>
```



![](http://upload-images.jianshu.io/upload_images/2338511-654a1c03dcd0096e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```python
tips['smoker'].head()
```



```
0    No
1    No
2    No
3    No
4    No
Name: smoker, dtype: category
Categories (2, object): [Yes, No]
```



```python
type(tips)
```



```
pandas.core.frame.DataFrame
```



```python
tips['if_smoker']=tips['smoker'].map({'Yes':1, 'No':0})
```

```python
tips.if_smoker.head()
```



```
0    0
1    0
2    0
3    0
4    0
Name: if_smoker, dtype: int64
```



我们可以自由选择进入`PairGrid`的变量

```python
tips.head()
```




```python
g=sns.PairGrid(tips,x_vars=['smoker','sex', 'day'],y_vars=['total_bill','tip'],height=5, aspect=0.7)
g.map(sns.violinplot, palette='bright')
```



```
<seaborn.axisgrid.PairGrid at 0x1a278fa4e0>
```



![](http://upload-images.jianshu.io/upload_images/2338511-987148e27e7c1ba7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前面用到的是`g.map_diag`和`g.map_offdiag`分别是设置对角线和非对角线的绘制图类型，适用于方阵的情况，如果是统一设置则用`g.map`即可
