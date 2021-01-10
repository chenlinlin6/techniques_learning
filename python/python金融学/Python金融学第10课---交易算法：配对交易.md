![大纲](https://upload-images.jianshu.io/upload_images/2338511-57df19ae8690a8af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一、配对交易介绍
配对交易是一种数学交易策略，本质上是通过两只股票具有协整性，具有的差价(spread)变化来套利的方法。
## 基本原理
配对交易的基本原理是，两个公司的股票的走势虽然会在中间偏离，但最终都会趋于一致，这种性质就叫**协整性**。两家公司的股票的差价始终会围绕着一个均值在波动。我们用s1和s2来表达两家公司的股价，s1-s2就是两家公司的股票的差价。这个差价有时候会高于均值，有时候会低于均值，当高于均值的时候我们就要卖出s1，买入s2，因为根据协整性，两家股票最终会趋于一致，所以的话s1这个时候相对于s2是偏高的，因此之后是要降价的，所以这个时候要抛出，赚取差价，s2这个时候相对于s1是偏低的，之后会回升，所以要买入；同理，低于均值的时候就是相反操作。
# 二、导入数据，绘制趋势变化
我们用quandl模块来读取数据
![quandl](https://upload-images.jianshu.io/upload_images/2338511-f7717199e7b9547f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

阅读quandl的文档，里面要传入获取的公司名字，开始日期和结束日期
```
#规定起始日期和结束日期
start_date='01-07-2015'
end_date='01-07-2017'
united=quandl.get('WIKI/UAL',start_date=start_date,end_date=end_date)
america=quandl.get('WIKI/AAL',start_date=start_date,end_date=end_date)
united.head()
```
![数据概览](https://upload-images.jianshu.io/upload_images/2338511-f94b1c184c99b9ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到我们已经成功获取了联合航空和美国航空的股票数据。接下来我们绘制下价格变化。
```
plt.style.use('ggplot')
united['Adj. Close'].plot(label='United Airline',figsize=(12,8))
america['Adj. Close'].plot(label='America Airline')
plt.legend(loc='best')
```
![股票波动](https://upload-images.jianshu.io/upload_images/2338511-acc1e83d93a4a2ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图上可以看得出，有比较明显的协整性，即差价始终都保持一定的稳定性。
# 三、绘制出差价(spread)变化和均值（mean）
```
spread=united['Adj. Close']-america['Adj. Close']
spread_mean=spread.mean()
spread.plot(label='spread')
plt.axhline(y=spread_mean, color='black')
```
![](https://upload-images.jianshu.io/upload_images/2338511-27d775ef4ff64a2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/2338511-6e8e02e205fedfc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于绘制水平线用matplotlib的axhline方法，一般只用传入一个y值。
计算一下相关系数
![](https://upload-images.jianshu.io/upload_images/2338511-cb2aea86c22a9f8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里用的是numpy的corrcoef，相关系数矩阵的R值由协方差矩阵的C值得出，R值在-1到1之间
```
np.corrcoef(united['Adj. Close'], america['Adj. Close'])
```
![](https://upload-images.jianshu.io/upload_images/2338511-27353801cfa73742.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到相关性是0.96.
# 四、归一化处理
这里的归一化处理是因为从绝对值上面看比较不方便（这里其实我不太理解），可能是因为从绝对值上面来看不知道什么时候差价的变化才是显著的，所以要归一化，这样跟正态分布的z分布表才有标准的可比性。
```
#定义进行归一化的函数
def normed_price(price):
	return (price-np.mean(spread))/np.std(spread)
normed_spread=spread.apply(normed_price)
normed_spread.plot()
```
![](https://upload-images.jianshu.io/upload_images/2338511-ca2e98d1401aef47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看得到我们绘制出的图线确实是围绕着0在上下波动的，因此应该是可行的。
```
normed_spread.plot()
plt.axhline(normed_price(spread_mean))
plt.axhline(1,color='green',linestyle='--')
plt.axhline(-1,color='red',linestyle='--')
plt.legend(labels=['spread','mean','+1','-1'],loc='best')
```
![](https://upload-images.jianshu.io/upload_images/2338511-dcfa1656b1879a50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、滚动窗口的z分数变化
我们以30天的滚动窗口的均值和标准差来求每天的z分数。
```
avg_1=spread.rolling(1).mean()
avg_30=spread.rolling(30).mean()
std_30=spread.rolling(30).std()
normed_spread=(avg_1-avg_30)/std_30
normed_spread.plot(label='Rolling 30 day Z score')
plt.axhline(0,color='black')
plt.axhline(1,color='red',ls='--')
plt.legend()
```
![归一化的z score](https://upload-images.jianshu.io/upload_images/2338511-50b728776719e75d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、Quantopian上面的策略实施
![流程](https://upload-images.jianshu.io/upload_images/2338511-d50371727303c184.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们用代码来实现
```
import numpy as np
#在算法开始前都要先初始化，将相关内容定义
def initialize(context):
	schedule_function(check_pairs, date_rules.every_day(), time_rules.market_close(minutes=60))#每天都定时检查一下股价差价z score的变化情况并作出安排
    context.aal = sid(45971) #aal
    context.ual = sid(28051) #ual 
    context.short_top=False#我们在这里标注一下是否进行了交易，以差价里面的被减数的那只股票作为标志
    context.long_top=False
def check_pairs(context, data):
	aal=context.aal
	ual=context.ual
	prices=data.history([aal, ual], 'price',30,'1d')#data.history传入包括股票，字段，周期，频率等字段，在这里返回的是一个dataframe
	mavg_30=np.mean(prices[ual]-prices[aal])
	mstd_30=np.std(prices[ual]-prices[aal])
	short_price=prices.iloc[-1:]
	mavg_1=np.mean(short_price[ual]-short_price[aal])
	
	#判断z score的相对大小
	if mstd_30>0:
		z_score=(mavg_1-mavg_30)/mstd_30
		if z_score>0.5 and not context.short_top:
			order_target_percent(ual, -0.5)#卖出高价的
			order_target_percent(aal, 0.5)#买入低价的
			context.short_top=True
            context.long_top=False
		elif z_score<-0.5 and not context.long_top:
			order_target_percent(ual, 0.5)#买入被减数的股票
			order_target_percent(aal, -0.5)#卖出减数的股票
            context.short_top=False
			context.long_top=True
		if abs(z_score)<0.1:#如果z score变化很小，那么就按兵不动
			order_target_percent(ual, 0)#买入被减数的股票
			order_target_percent(aal, 0)#卖出减数的股票
			context.short_top=False
		    context.long_top=False
	record('z score', z_score)
```
![变动](https://upload-images.jianshu.io/upload_images/2338511-c9ae409fbbba43ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![日志](https://upload-images.jianshu.io/upload_images/2338511-e0580efa5aa56fb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到实现的算法能够根据z score的波动来做出反应
# 参考资源

1. [配对交易 - MBA智库百科](http://wiki.mbalib.com/wiki/%E9%85%8D%E5%AF%B9%E4%BA%A4%E6%98%93)
2. [搬砖的理论基础:配对交易 Pair Trading - 雪球](https://xueqiu.com/2401362725/59137819)
3. [Guide to Pairs Trading](https://www.investopedia.com/university/guide-pairs-trading/)
