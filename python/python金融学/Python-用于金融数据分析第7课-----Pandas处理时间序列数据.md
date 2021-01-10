## 一、Rolling 和 Expanding
rolling和expanding都是类似的，目的是查看股票市场价格随着时间的变化，不同的是rolling average算的是最近一个窗口期（比如说20天）的一个平均值，过了一天这个窗口又会向下滑动一天算20天的平均值；expanding的话，是从第一个值就开始累加地计算平均值。
```
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
df=pd.read_csv('walmart_stock.csv')
df.head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-bf0cbf3e1ae93b37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#在这里我把索引设为日期列
df.set_index('Date')
df.head()
```
输出结果
![](https://upload-images.jianshu.io/upload_images/2338511-f7eba066e20c4d2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

绘制一下开盘指数
```
df['Open'].plot(figsize=(16, 6), '-')
```
![](https://upload-images.jianshu.io/upload_images/2338511-2fd9ee27707c28bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### rolling
现在开始绘制滚动平均值
![](https://upload-images.jianshu.io/upload_images/2338511-384e4edf3073f69f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据官方文档，我们主要设置窗口大小就可以了。
```
#注意在滚动之后是要设置聚合函数的，expanding一样，跟groupby操作类似
df.rolling(7).mean().head(10)
```
![](https://upload-images.jianshu.io/upload_images/2338511-c5c0fe1b1e603702.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到前6天是没有值的，原因是前6天都没有之前的7天数据，所以是nan。
```
#绘制出open的原数据的曲线和滚动平均值的曲线
df['former 30 days rolling Open mean']=df['Open'].rolling(30).mean()
df[['Open', 'former 30 days rolling Open mean']].plot(figsize=(16, 6))
```
![](https://upload-images.jianshu.io/upload_images/2338511-925a4d1131529cd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到滚动平均值会更加地差异化更小。
### Expanding
![](https://upload-images.jianshu.io/upload_images/2338511-546c28ba218c1b9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
#在这里要定义最小的观察元素是1个，否则前面的可能都会是nan
df['former 30 days expanding Open mean']=df['Open'].expanding(min_periods=1).mean()
df[['Open', 'former 30 days expanding Open mean']].plot(figsize=(16, 6))
```
![](https://upload-images.jianshu.io/upload_images/2338511-254de9d95785787e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图上可以看得出，expanding曲线相对于原数据点的曲线要更加稳定一些，它可以用来看这只股票的长期稳定性。
### Bollinger Bands
```
df['former 30 days rolling Close mean']=df['Close'].rolling(20).mean()
df['upper bound']=df['former 30 days rolling Close mean']+2*df['Close'].rolling(20).std()#在这里我们取20天内的标准差
df['lower bound']=df['former 30 days rolling Close mean']-2*df['Close'].rolling(20).std()
df[['Close', 'former 30 days rolling Close mean','upper bound','lower bound' ]].plot(figsize=(16, 6))
```
![](https://upload-images.jianshu.io/upload_images/2338511-d035cefea29a70ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 二、Resampling
重采样可以认为跟 group以及上面说到的rolling和expanding都是一样的，都是分组操作。
![](https://upload-images.jianshu.io/upload_images/2338511-9e01c7a15bfe4f03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

官方文档当中主要注意的是rule，它是一个字符串的形式给出，表示我们希望以年月日工作日等等来对数据进行编组，同样地是编组完之后需要有个聚合函数。
看一下rule的种类。
![](https://upload-images.jianshu.io/upload_images/2338511-e136315a898335e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们先把以上麦当劳的数据的index进行转换成datetime格式，可以使用pd.to_datetime的方法。
```
df.index=pd.to_datetime(df.index)
type(df.index)
```
![](https://upload-images.jianshu.io/upload_images/2338511-521778cbc6a1a404.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过转换以后就是datetime格式了，接下来就是进行resample。
```
df.resample('M').mean()
```
![](https://upload-images.jianshu.io/upload_images/2338511-93cce28b87d00103.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看得到，在按照月份resample之后多余的行会去掉，只剩下最后统计的那一行（在这里就是每个月底统计上一个月的平均值）。
也可以自己定义我要对group后的元素怎么操作（比如说按照一个月group以后我想取出第一个值，或者说是其他的）
```
def first_day(grp):
	return grp[0]#返回这个月的第一天的值
df.resample('M').apply(first_day)
```
![](https://upload-images.jianshu.io/upload_images/2338511-a7ffa9b50fae6255.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样，每个月底都会返回这个月的第一天的值。

## 三、Time shifting
time shifting其实就是把索引往前或者往后挪动
![](https://upload-images.jianshu.io/upload_images/2338511-5e61e92cc83f125a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
df.shift(10)
```
![](https://upload-images.jianshu.io/upload_images/2338511-00a4c5dd043ddd83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里索引往前挪了10天，相当于数据往后挪动了10天，缺失值用nan补充。
