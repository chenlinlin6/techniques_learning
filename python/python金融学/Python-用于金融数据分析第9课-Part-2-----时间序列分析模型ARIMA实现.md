ARIMA模型实现预测的流程
![流程图](https://upload-images.jianshu.io/upload_images/2338511-cdd2f0d08fbe2a63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 一、导入数据
```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
```
 我们先在终端看一下牛奶产量这个文件的前几行再决定要怎么样读取这个csv文件。
![文件概览](https://upload-images.jianshu.io/upload_images/2338511-f6f187c2d0d8298d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到有两列，第一列是时间，第二列是产量，因此我们要做的是将第一列按照日期时序的方式读取，作为index。
```
df=pd.read_csv('monthly_milk_production_pounds_p.csv')
df.columns=['Month', 'Milk Production']
df.drop(168, axis=0, inplace=True)
df.set_index(pd.to_datetime(df['Month']), inplace=True)
df.head(10)
```
![尾部](https://upload-images.jianshu.io/upload_images/2338511-5092ef078ba3ded9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于第169行有不正常的值，所以要丢弃掉这一行
![](https://upload-images.jianshu.io/upload_images/2338511-37a97fdc7c659ad2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看得出来，已经是一个正确的格式了。
## 二、绘制观测点图
接下来绘制观测值。
```
plt.plot_date(x=df.index, y=df['Milk Production'], xdate=True, marker=None, linestyle='solid',color='darkblue', label=df.columns[1])
df['Milk Production'].rolling(6).mean().plot(color='darkgreen', label='rolling mean')
df['Milk Production'].rolling(6).std().plot(color='yellow', label='rolling std')
plt.legend(loc='upper left')
```
![](https://upload-images.jianshu.io/upload_images/2338511-4533f8b780dbc90d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-d8504215d298a9ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 三、adfuller检测不变性的显著性以及转换
### 3.1 检测不变性的显著性
从图上可以明显看出来观察点并不具有不变性，当然我们也可以用Augmented Dickey fuller test来检验是否有不变性（也就是有没有季节的影响）。也即是检验在序列相关存在的情况下单变量过程的单位根，可以参考下adfuller的[官方文档](http://www.statsmodels.org/dev/generated/statsmodels.tsa.stattools.adfuller.html?highlight=adfuller#statsmodels-tsa-stattools-adfuller)。
![](https://upload-images.jianshu.io/upload_images/2338511-03426f0571d25fc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们查阅官方文档，可以看得到输入的参数介绍，以及返回的值。
![](https://upload-images.jianshu.io/upload_images/2338511-f60b547fe8d47a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里主要选择的参数是x（输入的序列），maxlag（多少个延迟，默认是12个单位）。
我们在这里根据p-value来判断是否有明显的不变显著性，也就是我们设定零假设是数据是非静态的，备择假设是数据是静态的，如果p-value小于0.05，说明这种差别是有显著性的，推翻零假设，接受备择假设，也就是数据是静态的。
```
from statsmodels.tsa.stattools import adfuller
results=adfuller(x=df[df.columns[1]], maxlag=12)
labels=['adf statistics', 'pvalue', 'used lag', 'Number of observations']
for label, result in zip(labels, results):
	print label+': '+str(result)
if results[1]<=0.05:
	print "strong evidence against the null hypothesis, reject the null hypothesis. Data has no unit root and is stationary"
else:
	print "weak evidence against the null hypothesis, accept the null hypothesis. Data has unit root and is nonstationary"
```
![](https://upload-images.jianshu.io/upload_images/2338511-869f0e41bf2a9142.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里反馈的结果是非静态的，所以要转换成静态的数据。可以用到差分的方法。
（这里其实并不明白unit root是什么东西）
写成一个函数
```
def adfuller_test(df_series):
	results=adfuller(x=df_series)
	labels=['adf statistics', 'pvalue', 'used lag', 'Number of observations']
	for label, result in zip(labels, results):
		print label+': '+str(result)
	if results[1]<=0.05:
		print "strong evidence against the null hypothesis, reject the null hypothesis. Data has no unit root and is stationary"
	else:
		print "weak evidence against the null hypothesis, accept the null hypothesis. Data has unit root and is nonstationary"
```
### 3.2 转换不变性
用差分的方法，差分相当于给定两列相同的序列，其中一列去掉第一个值，另外一列去掉最后一个值，然后让这两列相减，这样它们就错开了，相当于以后面一个值减去前面一个值得到它们的差值，这就是一级差分。二级差分则是在一级差分后的序列基础上再进行多一次差分。
#### 3.2.1 一次差分
```
#shift方法把索引往上移动一位，相当于数据往下移动一位
print df['Milk Production'].head(2)
print df['Milk Production'].shift(1).head(2)
```
![](https://upload-images.jianshu.io/upload_images/2338511-7f63017b36546ecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
df['Milk Production one difference']=df['Milk Production']-df['Milk Production'].shift(1)
df['Milk Production one difference'].head()
```
![](https://upload-images.jianshu.io/upload_images/2338511-d3b9557d30579193.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
df['Milk Production one difference'].drop(1, axis=0, inplace=True).plot(color='darkgreen',label='Milk Production one difference')
plt.legend(loc='upper left')
```
![](https://upload-images.jianshu.io/upload_images/2338511-9d1cc34004be8f47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们尝试一下运行adfuller test
```
adfuller_test(df['Milk Production one difference'].dropna(axis=0))
```
![](https://upload-images.jianshu.io/upload_images/2338511-9f9e791bea59bd91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经过一次差分以后已经是静态数据了。
#### 3.2.2 季节性差分
我们看一下12个月为单位的季节性差分
```
df['seasonal difference']=df['Milk Production']-df['Milk Production'].shift(12)
#df['seasonal difference'].head()
df['seasonal difference'].plot(color='darkred',label='seasonal difference')
plt.legend(loc='upper left')
```
![](https://upload-images.jianshu.io/upload_images/2338511-0dad8c5cd02f8dc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

执行adfuller test
```
adfuller_test(df['seasonal difference'].dropna(axis=0))
```
#### 3.2.3 季节性一阶差分
对季节性差分我们继续进行一阶差分。
```
df['seasonal one difference']=df['seasonal difference']-df['seasonal difference'].shift(1)
#df['seasonal one difference'].head()
df['seasonal one difference'].plot(color='darkred',label='seasonal one difference')
plt.legend(loc='upper left')
```
![](https://upload-images.jianshu.io/upload_images/2338511-a0ecd77c64344a19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


执行adfuller test
```
adfuller_test(df['seasonal one difference'].dropna(axis=0))
```
![](https://upload-images.jianshu.io/upload_images/2338511-4a5a705d36048be9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对季节性差分再进行多一次一阶差分后能够获得静态数据。
## 四、绘制自相关和偏相关图，确定p, d, q
### 4.1  自相关和偏相关
自相关图和偏相关图可以得出p, d, q合适的值，对于自相关图，如果第一个滞后值是正的，那么自回归项AR应该保留，反之，如果为负的，则移动平均项MA保留。在实际应用当中，很少有两项都用上的情况。
对于偏相关图，要关注是否有个滞后的节点，在这个节点以后有个突然的下降，如果有这样的猛降，那么说明应该用自回归项AR，如果pcf值随着k的增大缓慢下降，则用移动平均项MA。这里我的理解是如果有个k增大猛降的过程，那么说明只跟自身的因素有比较大的关系，等到滞后继续增大，新的自变量也没办法造成更大的pcf偏相关项的大影响，说明自回归项作用更大，反之则说明不同滞后自变量的输入始终对pcf偏相关值有影响，因此要保留自回归项。
### 4.2 绘制一次差分和季节性一次差分的acf和pcf图
![](https://upload-images.jianshu.io/upload_images/2338511-5925cc6823f7a953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们看一下官方文档，主要传入的参数是序列x以及滞后项
#### 4.2.1 一次差分的acf和pcf
acf图
```
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf 
plot_acf(df['Milk Production one difference'].dropna())
```
![](https://upload-images.jianshu.io/upload_images/2338511-57b99ee12b9832f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

pacf图
```
plot_pacf(df['Milk Production one difference'].dropna())
```
![](https://upload-images.jianshu.io/upload_images/2338511-eb8b6453ddd19f67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出对于自相关图lag为1的自相关值是要大于0的，并且偏相关是猛降的，因此应该要用AR自回归模型。
#### 4.2.2 季节性差分的一次差分的acf和pcf
acf图
```
plot_acf(df['seasonal one difference'].dropna())
```
![](https://upload-images.jianshu.io/upload_images/2338511-b43da03d84e0c561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

pacf图
```
plot_pacf(df['seasonal one difference'].dropna())
```
![](https://upload-images.jianshu.io/upload_images/2338511-3e4f4c7506fd3669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于季节性差分的一次差分也是有相似的结果。
注意以上的自相关图和偏相关图都是只能对静态数据绘制。
#### 4.2.3 对季节性差分的一次差分最终绘图
```
#设定画布的大小
fig=plt.figure(figsize=(16, 6))
#增加第一个子图
ax1=fig.add_subplot(211)

plot_acf(df['seasonal one difference'].dropna(), ax=ax1, lags=40)#在这里我选择第一个子图绘图，并且以前面的40个时序数据为滞后来绘制自相关图

#增加第二个子图
ax2=fig.add_subplot(212)
plot_pacf(df['seasonal one difference'].dropna(), ax=ax2, lags=40)
```
![](https://upload-images.jianshu.io/upload_images/2338511-0baf329cdcef86c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 在这里提一下，绘制子图有两种方法，一种是ax=figure.add_subplot(xxx)另一种是plt.subplot(xxx)
## 五、构建模型
### 5.1 构建模型
![](https://upload-images.jianshu.io/upload_images/2338511-f7a80cdc2d42ca43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们要用季节性的ARIMA模型来构建我们的预测模型。
通过官方文档我们知道这个模型由两部分组成，一部分是非季节项，另外一部分是季节项，主要传入p, d, q项，对于季节项要多传入一项s周期频率项，对于月份数据为12，对于季度数据为4。
```
from statsmodels.tsa.statespace.sarimax import SARIMAX
model=SARIMAX(df['Milk Production'], order=(0, 1, 0), seasonal_order=(1, 1, 1, 12))
results=model.fit()
print results.summary()
```
![](https://upload-images.jianshu.io/upload_images/2338511-c0ec7113e3930a24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>注意，在这里建模的时候，是以原始观测点为建模数据，不是以差分后的季节性数据来建模，之前差分的目标是为了找出合适p, d, q值
利用最大似然估计来拟合数据。
### 5.2 绘制残差
 最后的结果有个属性是残差项，是关于真实值和预测值的差距，可以打印出来看看。
```
results.resid.dropna(axis=0, inplace=True)
results.resid
```
![image.png](https://upload-images.jianshu.io/upload_images/2338511-6dc5c9337ad02a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
results.resid.plot()
```
![](https://upload-images.jianshu.io/upload_images/2338511-3f4f0f2b7a00e8fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
results.resid.plot(kind='kde')
```
![](https://upload-images.jianshu.io/upload_images/2338511-33e585172792ed71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


残差大部分集中在0附近。
## 六、预测未来值
### 6.1 预测值
接下来我们就可以用建立好的模型去预测未来的值。
![](https://upload-images.jianshu.io/upload_images/2338511-c71ff6c1e2054db2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主要传入三个参数，start，end和dynamic。start和end分别是预测值的序号，在这里选择158和168，只预测10个值。dynamic可以传入多个类型，在开始动态预测的时候相对于开始的地方的一个整数位移，说起来有点拗口，也就是离我们开始的地方有多远开始预测，在这个值以前真实值用作预测，在这个值以后前面的预测值用来预测。
```
df['predicted_values']=results.predict(start=158, end=168, dynamic=True)
df['predicted_values']
df[['Milk Production','predicted_values']].plot()
```

![](https://upload-images.jianshu.io/upload_images/2338511-db70a5328ec2ff01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 6.2 预测未来值
pandas的date_range可以按照自己的需要生成一系列相应的日期序列
```
future_dates=pd.date_range(start='1976-1-1', periods=24,freq='MS')
future_dates
```
![](https://upload-images.jianshu.io/upload_images/2338511-6abcdd0a5721a7ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MS是频率的单位，在这里表示的是按每个月步长，返回每个月的开始第一天month start的意思。
接下来构建新的dataframe，然后把这个dataframe接到原先的dataframe之后，然后绘制图线。
```
future=pd.DataFrame(index=future_dates, columns=df.columns)
final=pd.concat(objs=[df, future], axis=0)
```
```
final['forecast_values']=results.predict(start=168, end=188, dynamic=True)
final[['Milk Production','forecast_values']].plot()
```
![](https://upload-images.jianshu.io/upload_images/2338511-2fcf3e2d4508db99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/2338511-147b8c75a8816acf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>写到最后，其实还有一些问题没有明白，比如构建模型的时候模型的p, d, q参数是如何通过自相关图和偏相关图读出的？这点我需要再阅读多点资源，然后再补充。
