由于我对pandas的数据可视化这部分比较不熟，因此我主要把内容集中在这部分。pandas的数据可视化是在matplotlib基础上建立的，底层运行程序仍然是matplotlib。  
## 一、读取数据
```
import pandas as pd
import numpy as np
df1=pd.read_csv('df1',index_col=0)#在这里可以指定第一列为index列
df2=pd.read_csv('df2')
```
![](https://upload-images.jianshu.io/upload_images/2338511-e7cb63857918b81c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-b2b892822040c20e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 设置格式
matplotlib默认的绘图格式比较难看，我们可以选用不同的绘图格式让我们的作图更加地高大上一些。
比如说我们用ggplot这种风格，我们就可以使用以下语法：
```
import matplotlib.pyplot as plt
plt.style.use('ggplot')
```
对比使用前后的变化
![](https://upload-images.jianshu.io/upload_images/2338511-c5071dc8400b9dfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-bea92117f0e1859d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体的风格可以参考matplotlib的[官网](https://matplotlib.org/gallery.html#style_sheets)。
## 二、绘图的种类
### pandas绘图的语法
pandas可以用两种语法来绘图：
```
#通过plot.之后加绘图的类型
df['A'].plot.hist()
#通过传递kind参数指明要绘制什么图
df['A'].plot(kind='hist')
```
### 绘图的类型
>绘图的种类一共有如下：
>df.plot.area
df.plot.barh
df.plot.density
df.plot.hist
df.plot.line
df.plot.scatter
df.plot.bar
df.plot.box
df.plot.hexbin
df.plot.kde
df.plot.pie


讲几个我以前没有注意过但是挺特别的一些点。  
#### 堆积图
```
df2.plot.bar(stacked=True)
```
![](https://upload-images.jianshu.io/upload_images/2338511-34ac47f6154d86e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 散点图
散点图的colormap是挺炫酷的功能
```
df1.plot.scatter(x='A',y='B',c='C',cmap='coolwarm')
```
![](https://upload-images.jianshu.io/upload_images/2338511-28e412c3db573eed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里我用了ggplot的样式，所以总体看起来会比上面的好看很多，并且我指定颜色项为C列的值，颜色的变化样式是coolwarm，否则默认是黑白灰。还可以设置大小s='D'这个参数。
```
df1.plot.scatter(x='A',y='B',c='C',s=df1['D']*100,cmap='coolwarm')
```
![](https://upload-images.jianshu.io/upload_images/2338511-e2e8b61f31d8e56a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里指定大小的时候要输入的是series，输入列D不行（我也不太明白，提示是不安全）。当然这里可能会相互遮挡，可以通过设置alpha的值来设置透明度。
#### 六角图
```
df1.plot.hexbin(x='A',y='B',gridsize=25, cmap='coolwarm')
```
![](https://upload-images.jianshu.io/upload_images/2338511-a282d3a35b149995.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

颜色代表的值越高，那么相应的点数目就会越多。
#### 密度曲线
```
df1['A'].plot.kde()
df1['A'].plot.density()
```
![](https://upload-images.jianshu.io/upload_images/2338511-25127c506641b647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、时序图
```
#读取麦当劳数据
df=pd.read_csv('mcdonalds.csv',index_col=0, parse_dates=True)
```
![](https://upload-images.jianshu.io/upload_images/2338511-4f052de47499d188.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

matplotlib.dates可以把日期戳转化为可以理解的日期，也就是可以将日期变为按年，按星期等多种频率的尺度。也可以用pd.to_datetime把相应的日期列转换成日期数据。
```
import matplotlib.dates as dates
idx=df.loc['1970-01-02':'1971-01-10'].index
stock=df.loc['1970-01-02':'1971-01-10']['Adj. Close']
fig,ax=plt.subplots(figsize=(6,6))
ax.plot_date(idx, stock, '-')
plt.tight_layout()
plt.show()
```
![](https://upload-images.jianshu.io/upload_images/2338511-784805efc6146ce0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
fig,ax=plt.subplots(figsize=(6,6))
ax.plot_date(idx, stock, '-')
#如果坐标轴上面的tick过于密集
fig.autofmt_xdate()#自动调整xtick的间距
#网格
ax.xaxis.grid(True)
ax.yaxis.grid(True)
#设置日期为每个月
#Location也就是以什么样的频率
ax.xaxis.set_major_locator(dates.MonthLocator())
#Format坐标轴展示的样式
ax.xaxis.set_major_formatter(dates.DateFormatter('%b-%Y'))
plt.tight_layout()
plt.show()
```
![](https://upload-images.jianshu.io/upload_images/2338511-5fd602c5571dffc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以通过一个星期来展示
```
fig,ax=plt.subplots(figsize=(6,6))
ax.plot_date(idx, stock, '-')
#如果坐标轴上面的tick过于密集

#网格
ax.xaxis.grid(True)
ax.yaxis.grid(True)
#设置日期为每个月
#Location也就是以什么样的频率
ax.xaxis.set_major_locator(dates.WeekdayLocator(byweekday=1))
#Format坐标轴展示的样式，a代表你之前规定的星期几，b代表月份的前三个字母，B是月份的全称
ax.xaxis.set_major_formatter(dates.DateFormatter('%b-%a'))
fig.autofmt_xdate()#自动调整xtick的间距
plt.tight_layout()
plt.show()
```
![](https://upload-images.jianshu.io/upload_images/2338511-885bfacd59be8659.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于所涉及的时间很长，所以部分重叠了
