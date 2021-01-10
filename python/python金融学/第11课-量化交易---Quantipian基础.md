本节课对quantopian这个量化交易平台进行初体验，并尝试理解一些关于股票交易的概念。由于作者为小白，且原本的Udemy课程讲得也比较简单，所以可能会有不少疏漏之处，敬请指出。
![流程图](https://upload-images.jianshu.io/upload_images/2338511-08e8288d7685ea14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一、Quantopian股票量化交易平台
![quantopian平台](https://upload-images.jianshu.io/upload_images/2338511-3e637bac7a035152.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

quantopian是一个可以进行股票量化交易的平台，你可以在上面写自己的量化交易代码，然后这些代码是可以用历史数据或者是当前的现时股票数据进行测试，同时你还可以在上面直接进行量化交易，另外还有时不时的一些量化交易问题解决的竞赛。我主要集中在将其作为一个量化交易的代码实现和测试的学习平台。由于处于初学阶段，所以本文主要介绍一些非常基础的东西。
# 二、股票的基本概念
## 2.1 头寸
头寸意思就是款项，在金融学当中指的是所持有的用于交易的总资金量或者是借来交易的总资金量。
## 2.2 持仓，开仓，平仓
这部分我大概理解的是，持仓是指你手上所持有的币种市值也就是你的股票量占你用于交易的总资金量的比例。比如说，你手上有10万元，花了6万元去买了苹果的股票，剩下4万元的投资资金，那么你这个时候的苹果股票的持仓就是6万元。开仓和平仓是相对应的，开仓和平仓构成了一个投机交易行为的开始和结束的闭环。开仓相当于是交易行为里面的交易者新买入或者新卖出一定数量的期货合同，平仓则是在交易日到期前对之前开仓操作的期货合同重新又卖出或者买入以进行对冲风险的行为。相当于我们买股票，你可以选择一直持有，直到期货合同进行实物结算或者是交易日到期进行现金结算，但是大部分投机者往往会在交易日到期之前就选择对之前开仓后的的期货合同进行逆操作以获取一些利益，这样的开仓和平仓行为就构成了一个完整的交易行为。

## 2.3 做空与做多

# 三、Quantopian基本框架
![基本框架](https://upload-images.jianshu.io/upload_images/2338511-a1f297def9e83d5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到quantopian的基本界面包括左边的代码输入界面，类似于IDE的输入，右上是股票的价格波动情况和你写的代码交易的总资本的变动情况，右下则是类似于IDE的输出。
## 3.1  初始化函数和操作
quantopian有几个比较重要的函数：
### 初始化函数initialize()
initialize函数可以理解为python当中的init函数，相当于把一些接下来要用的东西都先定义好，一些数据对象都定义为context这个字典的值，用context.键的方式定义这个python字典，之后的函数都是可以访问这个字典的。
```
def initialize(context):
    context.aapl=sid(24)
    context.csco=sid(1900)
    context.amzn=sid(16841)
def handle_data(context,data):
    order_target_percent(context.aapl,0.27)
    order_target_percent(context.csco,0.20)
    order_target_percent(context.amzn,0.53)
```
![sid函数](https://upload-images.jianshu.io/upload_images/2338511-0e2c3f1a7a19a26e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sid函数可以接受一个股票的id，然后返回对应的股票数据。
![](https://upload-images.jianshu.io/upload_images/2338511-17abca88e39e4bc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里相当于是在开盘的时候按照资本的一定比例对购买相应的股票。handle_data在这里接受两个参数，一个是context一个是data，context可以认为是一个字典，然后它有很多我们在initialize里面定义的索引，handle_data在每分钟的时候都会去根据这个索引寻找当前的状态也就是当前的股票价格，data就是一个对象，存储了一些接口的函数。
![](https://upload-images.jianshu.io/upload_images/2338511-c736e7c90816a1e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在输入以上代码以后，点击build algorithm。运行的结果是作为1000万美金作为资本返回最终的回报率，拟合的alpha和beta值，以及夏普比率。
每个时间点都有对应的资本总体收益率和市场的总体总收益率。在这里需要注意的是在每一分钟的时候handle_data都在重新地将资本归到1000万美金，然后执行同样的订单操作，计算当前的收益率。
## 3.2 抓取现在的数据
data_current方法可以抓取最近的股票数据，股票数据的字段包括之前课程数据涉及到的字段，包括'price', 'open', 'high', 'low', 'close', and 'volume'，在这里data.current要传入两个参数，一个是股票或者是股票列表，第二个就是字段
```
def initialize(context):
	context.tech=[sid(24),sid(1900),sid(16841)]
def handle_data(context, data):
	tech_close=data_current(context.tech, 'close')
	print tech_close
	print type(tech_close)
```
![](https://upload-images.jianshu.io/upload_images/2338511-6fd211a1d4ba6fda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们可以打印出每分钟的收盘价格变化。
## 3.3 data.can_trade判断股票当前是否支持交易
并不是所有的股票当前都是可以交易的，所以我们要先判断是否能交易，再出手。data.can_exchange传入两个参数，一个是特定的股票，或者是股票的序列
```
def initialize(context):
    context.amzn=sid(16841)
def handle_data(context, data):
	#如果亚马逊股票当前支持交易的话，避免报错
	if data.can_trade(context.amzn):
		#全部资产都用来买亚马逊的股票
		order_target_percent(context.amzn,1)
```
## 3.4 交易的历史价格
我们可以用data.history对股本进行查看历史价格，需要制定哪个字段，返回的天数数量，以及频率。以移动窗口的原理返回（这里没有太过理解，还得学习下）
```
def initialize(context):
    # AAPL, MSFT, and SPY
    context.assets = [sid(24), sid(1900), sid(16841)]

def before_trading_start(context,data):
	history=data.history(context.assets,fields='price', bar_count=5, frequency='1d')
	print history
```
![](https://upload-images.jianshu.io/upload_images/2338511-c385611f977919c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里的bar_count实际上是往前推的数目，比如如果是5的话，意思是往前推4天包括今天的价格都返回，类似的6天就是前5天包括今天的价格。对于美国的股票市场和期货市场一天的开始和结束时间是不同的。
参考以下的类比：
![](https://upload-images.jianshu.io/upload_images/2338511-9dd9e9931dc8bdef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.5 调度（也就是定时做某事）
我们可以在初始化函数用schedule_function来传入一个函数定时执行。可以理解为某一天，某个时刻，做某件事情
```
def initialize(context):
	context.aapl=sid(49051)
	schedule_function(open_position, date_rules.week_start(), time_rules.market_open())#在第一周开始的开市的时候就买入苹果股票
	schedule_function(close_position, date_rules.week_end(), time_rules.market_end(minitues=30))#在闭市以前30分钟平仓卖出
def open_position(context,data):
	order_target_percent(context.aapl, 0.1)#持仓0.1
def close_position(context,data):
	order_target_percent(context.aapl,0)#平仓，全部卖出，持仓为0
```
![](https://upload-images.jianshu.io/upload_images/2338511-8b8788423fc57f0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里定时运行的函数要传入两个参数context和data。
## 3.6 绘制曲线
你可以通过record方法绘制曲线，根据官方文档说明，里面至少可以记录并绘制最多5个字段。
![](https://upload-images.jianshu.io/upload_images/2338511-85b8d6bc64469609.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用法是传入多个字段，等号左边是序列的命名，右边是传入的值。我们试一下绘制亚马逊和IBM的收盘价
```
#初始化相关的对象
def initialize(context):
	context.amzn=sid(16841)
    context.ibm=sid(3766)
	#在这里我们做两件事，在开盘的时候我们做多0.5的亚马逊股票，同时做空0.5IBM的股票，用定义的rebalance函数实现
	schedule_function(rebalance, date_rules.every_day(),time_rules.market_open())
	#第二件事情就是在每天收盘的时候都绘制出亚马逊和IBM的收盘价，用定义的plot_vars函数实现
	schedule_function(plot_vars, date_rules.every_day(),time_rules.market_close())
def rebalance(context, data):
	order_target_percent(context.amzn, 0.5)
	order_target_percent(context.ibm, -0.5)
def plot_vars(context, data):
	record(amzn_close=data.current(context.amzn, 'close'))
	record(ibm_close=data.current(context.ibm, 'close'))
```
![](https://upload-images.jianshu.io/upload_images/2338511-7c56277296c09404.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里我们就可以绘制出每天亚马逊和IBM的收盘价格，上面的曲线是你的算法操作的收益率和市场的基准收益率，每天都会重新归为1000万美金。

## 3.7 滑动和佣金
### 滑动
实际上当我们下订单的时候订单的大小是会对fill rate和股票价格造成影响的。对于fill_rate我一开始没太懂是什么东西，查了一下，按照字面理解，应该是你下的订单有多少被满足了。相当于如果你下了100单，这100单很可能不会马上都成交，实际上它很可能只成交60单，那么这个时候你的填充率就是60%，填充率可以反映出库存满足需求的能力。
![填充率](https://upload-images.jianshu.io/upload_images/2338511-1d66377907aac0b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们下订单的时候，比如我们买入某只股票，实际上是可能拉高这只股票的价格的，如果是卖出有可能是拉低这只股票的价格。因此我们的填充率会受到你下的订单的大小以及市场上当时的成交量大小的影响。在我们算法里面会设定一个参数volume_limit，也就是我们限定算法每次最多只能交易的量。有几种方法可以模拟滑动slippage也就是订单对填充率和成交价格的影响，我们可以在初始化函数initialize里面定义set_slippage函数，有许多种slippage的模型，默认是VolumeShareSlippage。
`set_slippage(slippage.VolumeShareSlippage(volume_limit=0.025, price_impact=0.1))`
在这里我有点不明白price_impact是什么参数。
我理解的是，如果我们下60股的购买订单，那么我们设定了那只股票每分钟交易1000股，每次只能最多交易0.025，也就是25股。那么就会分成25+25+10股接下来的三分钟三次每分钟买入。对于设定买入第二天的开盘股票和在闭市之前买入的订单都会被算法取消，保证了股本交易的流动性。
### 佣金
股票交易都是有交易的费用的，set_commission和set_slippage类似，就是为了模拟股票交易费用，默认模型是commission.PerShare
```
set_commission(commission.PerShare(cost=0.0075,mini_trade_cost=1))
```
这里相当于每股交易都要收0.0075美元。另外一个参数等待研究。
## 总结
这篇文章主要介绍了quantopian的基本结构，以及一些股票的概念以及实现方法，由于本人是小白，在接触这部分内容之前完全没有概念，因此接下来还会继续慢慢研究，这篇文章权当入门学习。
## 参考资源
1. [量化交易实践篇（2）—— Quantopian策略实现初体验](https://www.jianshu.com/p/5f7a8f53045d)
2. [quantopian](https://www.quantopian.com/)
3. [BusinessDictionary](http://www.businessdictionary.com/definition/fill-rate.html)
4. [quantopian tutorial](https://www.quantopian.com/tutorials/getting-started)
