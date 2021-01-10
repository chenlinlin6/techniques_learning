## 大纲
![](https://upload-images.jianshu.io/upload_images/2338511-c3739ad7d9367ec8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 一、时间序列基础知识
时间序列有一些基本的性质。
### 1. 趋势
![](https://upload-images.jianshu.io/upload_images/2338511-c6c314335fdc4ef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以看出有个一开始向上，中间静止或者叫水平，后半段向下的趋势，这个趋势需要通过对数据求平均值才会看得更加明显。
虽然有围绕着均值上下波动的偏差，但是从较大的时间尺度上面来看，它仍然是可以看作有明显的趋势的。
### 2. 季节性
![季节性](https://upload-images.jianshu.io/upload_images/2338511-ff2ace1d5fc3e1ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

季节性比较好理解，就是值随着月份有着明显的涨落，比如谷歌搜索snowboarding的明显到了冬天就有个明显的飞涨，到了夏天就有明显的回落，重复出现。当然，从图上总体趋势还可以看出搜索snowboarding的量总体上也有个下降。
### 3. 周期性
![周期性](https://upload-images.jianshu.io/upload_images/2338511-0c5b94e81718873d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

周期性跟季节性的区别在于它带有明显的趋势，但没有重复，在上图当中不是季节性，因为并不是一到某个季节值就会有明显的涨落，而是每隔几年就有一次涨落。（这里还有些不理解）
## 二、statsmodels模块介绍
statsmodels是能够对数据进行建模，预测的工具包，它还能对数据进行假设性检验。
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import statsmodels.api as sm
nt=sm.datasets.macrodata.NOTE#datasets模块包含了很多数据集，我们在这里调用macrodata这个数据集，然后查看这个数据集的相关信息
print nt
```
![](https://upload-images.jianshu.io/upload_images/2338511-d6eea5ab5687d196.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来载入数据集
```
df=sm.datasets.macrodata.load_pandas().data
df.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-1ceb6c216afb4586.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里的意思是说以pandas的形式加载这个macrodata的数据，然后我取加载后的数据（可能是load_pandas之后不只有data这个属性，还有其他的属性）
![](https://upload-images.jianshu.io/upload_images/2338511-b929989b0c8e872d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们看一下官方文档，可以发现，它的作用是传入开始start和end的日期文本，然后会转换成一系列的datetime序列（可以认为是range函数的日期版），关键是这里的日期缩写是怎么规律？
![](https://upload-images.jianshu.io/upload_images/2338511-e15cd4255e32355e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里我们大致可以看到缩写前面一部分是年份，后面字母代表频率，如果是Q则代表季度qurter，字母后面的数字代表开始的节点，如果是1996Q2，代表1996年的第二个季度开始，往后推length个长度的单位。
```
index=pd.Index(sm.tsa.datetools.dates_from_range('1959Q1', '2009Q3'))
df.index=index
```
![](https://upload-images.jianshu.io/upload_images/2338511-3ccf7b3c044b9062.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们发现原始数据的步长都是按照季度来的，因此我们生成按照季度来的时间序列，并将这个序列作为索引，给原始数据对应上
```
#画出重新设置序列以后的图
df['realgdp'].plot(color='darkgreen', linestyle='dashed')
plt.ylabel('gdp')
```
![](https://upload-images.jianshu.io/upload_images/2338511-3f8d36d704e10590.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###  使用statsmodel模块得到趋势
Hodrick-Prescott filter能够把趋势和周期性区分开来。趋势用tao_{t}表示，而周期项用zeta_{t}
表示。所以某个时间的值可以用两者的加和表示。
![](https://upload-images.jianshu.io/upload_images/2338511-588b55f1ac2cc934.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而预测模型的组成由是最小化以下这个二次方损失函数所决定：
![](https://upload-images.jianshu.io/upload_images/2338511-4b5e963f767435d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
gdp_cyclical, gdp_trend=sm.tsa.filters.hpfilter(df['realgdp'])
gdp_trend.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-5791af2f955b765a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打印出的结果是这个。
接下来我们把趋势和原数据都一起作图。
```
df['gdp_trend']=gdp_trend
df['gdp_cyclical']=gdp_cyclical
df[['realgdp', 'gdp_trend']].plot(figsize=(16, 6), color=['darkgreen', 'red'], linestyle='dashed')
```
![](https://upload-images.jianshu.io/upload_images/2338511-6fc85deb5fee24a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看得到趋势和原始数据符合得不错。
还可以通过设置线的颜色和实虚线来突出表示。
```
df[['realgdp', 'gdp_trend']].plot(figsize=(16, 6), color=['darkgreen', 'red'], style=['-','--'])
```
![](https://upload-images.jianshu.io/upload_images/2338511-c08553450ee600a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、指数平均化移动平均值EWMA
```
#导入航班数据
df=pd.read_csv('airline_passengers.csv', index_col='Month', parse_dates=True)
#指定index column，并且尽可能按照日期格式来解析index
df.dropna(inplace=True)#丢掉缺失的行
df.index=pd.to_datetime(df.index)
plt.plot_date(data=df, x=df.index, y='Thousands of Passengers', xdate=True, marker=None, linestyle='solid', color='darkgreen')
```
![](https://upload-images.jianshu.io/upload_images/2338511-561e5dcd16d28fac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

小提示，在这里可以通过指定marker为None来消除标记。
#### SMA
接下来使用简单的移动平均，也就是不引入权重和指数。
```
df['6_rolling_mean']=df['Thousands of Passengers'].rolling(6).mean()#仍旧是在移动完rolling之后要用聚合函数
df['12_rolling_mean']=df['Thousands of Passengers'].rolling(12).mean()
df.plot(kind='line', color=['darkred', 'darkgreen', 'darkblue'])
```
![](https://upload-images.jianshu.io/upload_images/2338511-22138beaace56fe4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上分别代表原数据，6个月为一个窗口的移动平均值以及12个月为移动窗口的平均值绘图。
#### EWMA
简单移动平均值具有一些缺点：
- 过小的窗口会受到噪声的影响，而不是反映出真实的信号。
- 移动平均值一直与窗口的大小有关。
- 移动平均值不会达到数据的极值。
- 移动平均值并不会告诉你的未来的发展，它只是告诉你过去的趋势。
- 极端的历史值会影响到现在的移动平均值。
针对以上问题，引入expotentionial weighted moving average，也就是引入为指数项表达的权重，权重的大小除了与窗口有关，也跟当前时间上的远近程度有关，与当前时间相距更远的，权重会更小，可以理解为影响效果会更弱。
表达式如下：
![](https://upload-images.jianshu.io/upload_images/2338511-7be70965fcbdf1e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里的x_{t-i}代表输入（其实我这里也不太懂是什么输入），代表着距离当前时间点t前i个单位的输入的影响，wi就是这个距离为i的权值。
EW函数支持两种，一种是默认adjust=True，在这种情况下，距离为i的滞后对t的权值![](https://upload-images.jianshu.io/upload_images/2338511-1f6d4ca72d2eb45f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样以上的公式可以写成：
![](https://upload-images.jianshu.io/upload_images/2338511-f8a329b1677cdc69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果adjust=False，那么就是另外一种计算方式：
![](https://upload-images.jianshu.io/upload_images/2338511-94a773706d2d9631.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-46e9bd27ad387241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相当于使用权重：
![](https://upload-images.jianshu.io/upload_images/2338511-e5747e4b9907d9e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里其实是类似于之前的ARIMA模型，认为当前值是跟前面的一项有关，并且加上当前的误差项。
具体有进一步的推导，参考这篇[文章](http://pandas.pydata.org/pandas-docs/stable/computation.html#exponentially-weighted-windows)。
接下来要注意![](https://upload-images.jianshu.io/upload_images/2338511-bf12fcafc38d2fa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，alpha可以从以下这些值当中取。
![](https://upload-images.jianshu.io/upload_images/2338511-ebd36a38f46d9e42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 在这里s代表移动平均值所涉及的跨度，c代表移动跨度s的一半，![](https://upload-images.jianshu.io/upload_images/2338511-616f4165e74525d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ，h是指权重项衰减到原有的一半的时候所需的时间大小。
代码部分。
```
df['12_span_ewm']=df['Thousands of Passengers'].ewm(span=12).mean()
df[['12_span_ewm', 'Thousands of Passengers']].plot(style=['--', '-'], color=['darkgreen','darkred'])
```
![](https://upload-images.jianshu.io/upload_images/2338511-d283f5bc70c05d61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看得出相对于简单的移动平均值，指数权值化后的移动平均值具有更加明显的季节性，与原有的数据更加贴合。
关于这部分还有很多需要学习的，目前只是入门。
## 三、ETS模型
我们还可以通过对原始数据按照Error、Trend和Seasonality三部分来分解，具体原理可以参考[这篇文章](https://www.jianshu.com/p/9df55a3f179d)。
ETS模型分为加法和乘法模型，如果趋势Trend是线性增长的关系，那么就用加法模型additive，否则非线性则用multiplicative乘法模型。具体用法是用seasonal_decompose，传入数据，以及对应的模型参数model。
```
from statsmodels.tsa.seasonal import seasonal_decompose
result=seasonal_decompose(df['Thousands of Passengers'], model='multiplicative')
result.plot()
plt.show()
```
![](https://upload-images.jianshu.io/upload_images/2338511-dd1ef0681197c5be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

result包括四部分内容，一个是观察值，趋势，季节性的值还有残差。残差可以认为是trend和观察值之间的差距。
