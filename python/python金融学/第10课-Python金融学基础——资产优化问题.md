在上一节课当中主要讲了几个重要的概念，包括股票的日回报率，日回报率的均值，累积回报率，波动(也就是标准差)，进而以日回报率的均值和波动来计算夏普比率，并说明夏普比率可以用来衡量每份风险背后所具有的收益，以此来定量衡量我们投资决策的真实收益大小。

在这节课，我们要解决第二个问题，给定一定的股票，比如说我给你四只股票的一系列的时间序列数据，以及给你限定10000元的投资，那么如何根据所给的股票数据来划分每只股票各投资多少占比以求得最大收益，也就是我要计算出股票购买所占资本的权重，这个权重可以实现总体的夏普比率最大化。
总体大纲：
![流程](https://upload-images.jianshu.io/upload_images/2338511-fd26c35694b94c2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一、加载数据及预处理
## 1.1 计算累积收益率，对数收益率和算术收益率
```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
AAPL=pd.read_csv('AAPL_CLOSE',index_col='Date',parse_dates=True)
AMZN=pd.read_csv('AMZN_CLOSE',index_col='Date',parse_dates=True)
CISCO=pd.read_csv('CISCO_CLOSE',index_col='Date',parse_dates=True)
IBM=pd.read_csv('IBM_CLOSE',index_col='Date',parse_dates=True)
AAPL.head()
```
![输出结果](https://upload-images.jianshu.io/upload_images/2338511-35d0a72f6486a20b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们把这四只股票的收盘价都合在一张dataframe里面
```
stock=pd.concat([AAPL, AMZN, CISCO, IBM],axis=1)
stock.columns=['AAPL', 'AMZN', 'CISCO', 'IBM']
stock.head()
```
![输出结果](https://upload-images.jianshu.io/upload_images/2338511-4904300b8dd344ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们计算一下日收益率的平均值
```
stock.pct_change(1).mean()
```
![输出结果](https://upload-images.jianshu.io/upload_images/2338511-bf960994b54c40bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们接下来看一下日收益率的偏相关性所引起的偏方差。关于这里的偏相关性我是这么理解的，因为这是股票市场，并且这几只股票都是科技股票，它们的市场是趋同的，因此在这里应该是认为彼此的价格变化是有相互影响的，因此在计算波动的时候，要考虑权重转变后其他股票对单只股票的影响。偏方差函数是cov()
```
stock.pct_change(1).cov()
```
![](https://upload-images.jianshu.io/upload_images/2338511-ad19ee519a60bb73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里以第一行为例子，AAPL是苹果股票的代号，它除了有自身波动的影响，即第一列。也有其他三只股票的影响，也就是其他三列，这部分的相关性导致的波动即为偏方差。

我们绘制一下股票的累积收益率的变化情况。
```
normed_ret=stock/stock.iloc[0]
normed_ret.plot(kind='line',figsize=(12,8),grid=True)
plt.legend(loc='best')
```
![](https://upload-images.jianshu.io/upload_images/2338511-20bf0ff2aaf29034.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们在这里引入对数收益率，原因是对数收益率相对于我们上面提到的算术收益率来说更加适合模型的计算，具体原因我觉得知乎这个[回答](https://www.zhihu.com/question/22012482/answer/34795487)是不错的，一方面是为了让我们的数据符合模型的平稳性假设，另外一方面应该是方便计算表达，而且可以发现对数收益率和算术收益率其实差的不多。关于对数收益率我还有些不理解的地方，还得继续研究下。我们绘制下对数收益率和算术收益率的分布吧。
```
#算术收益率
stock.pct_change(1).plot.hist(bins=100)
plt.legend(loc='best')
#对数收益率
log_ret=np.log(stock/stock.shift(1))
log_ret.plot.hist(bins=100)
plt.legend(loc='best')
```
![算术收益率的分布](https://upload-images.jianshu.io/upload_images/2338511-211245c7ffae9bed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![对数收益率的分布](https://upload-images.jianshu.io/upload_images/2338511-dbdfde21270912fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到差别并不是很大。
## 1.2 计算252天的平均收益率的总和以及偏方差矩阵
```
#252天的四只股票的平均对数收益率总和
total_mean_ret=log_ret.mean()*252
#252天的四只股票构成的偏方差矩阵
total_covariance=log_ret.cov()*252
print total_mean_ret
print total_covariance
```
![](https://upload-images.jianshu.io/upload_images/2338511-a3fde80c21ffee04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在这里计算资产组合的最优化权重，我们可以用到两个方法：一个是蒙特卡罗模拟法，另外一个是限制条件下的数学优化方法。我们分别来看一下。
# 二、蒙特卡罗模拟法
##  2.1 原理
这个方法听起来很高大上，其实方法很接地气，也很暴力。就是举出很多很多权重组合（多到几乎穷举），然后计算所有的点的夏普比率，哪个最高就取哪个点的均值和波动。
## 2.2 方法实现
### 2.2.1 计算单个权重的夏普比率
在这里我们先用代码实现一组投资资产权重的计算夏普比率，穷举只用在单次的外围加上一个遍历就可以了。
![numpy的random函数用法](https://upload-images.jianshu.io/upload_images/2338511-5098a1a8b4621dcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




```
#在这里我们用numpy的random.randn函数实现随机取四个数，这个方法是从[0, 1]中取出特定个数的数字的方法
weight=np.random.random(4)
print weight
```
![](https://upload-images.jianshu.io/upload_images/2338511-def3389f763b1681.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里我们看到的确是返回了四个随机的[0, 1]的值，但是要注意我们的权重之和是需要等于1的，所以要进行归一化。
```
normed_weight=weight/np.sum(weight)
print normed_weight
```
![](https://upload-images.jianshu.io/upload_images/2338511-004baf7133bb6d53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到归一化以后的权重确实是之和为1了。然后我们试下用这个权重去计算对应的资产组合的夏普比率。
按照权重去计算对应的252天的预期回报率和波动（也就是方差）
```
#预期回报率
exp_ret=np.sum(log_ret.mean()*normed_weight*252)
#波动
volat=np.sqrt(np.dot(normed_weight.T, np.dot(log_ret.cov()*252, normed_weight)))
print exp_ret, volat
#计算夏普比率
SR=exp_ret/volat
print SR
```
![](https://upload-images.jianshu.io/upload_images/2338511-1e8d6b0ff26e5577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![夏普比率](https://upload-images.jianshu.io/upload_images/2338511-03b93d0d6cc9d71e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里关于波动的计算方式，我是这么理解的后面的一个点乘实际上是计算出每个股票单独的方差，然后每只股票的方差又按照权重加和得到最终整体的方差。原因是每只股票波动是跟其他三只股票都是有联系的，所以要按照权重先算出每只股票自身的波动。第一步得到的是4*1的矩阵，然后前面的weights在转置以后是1*4的矩阵，两者的点乘就是最终经过权重转化的方差，开方后为标准差。
### 2.2.3 生成多个随机权重进行模拟
接下来我们就在外面套上一层循环，生成许多组不同的权重序列，按照上面的方式计算每组权重所得的期望回报和波动。
```
#随机权重的组数为15000
num=15000
#初始化总预期收益率和波动以及夏普比率的numpy数列
exp_rets=np.zeros(num)
exp_vols=np.zeros(num)
exp_SRs=np.zeros(num)
#迭代15000次
for ind in range(num):
	weight=np.random.random(4)
	normed_weight=weight/np.sum(weight)
	#预期回报率
	exp_ret=np.sum(log_ret.mean()*normed_weight*252)
	exp_rets[ind]=exp_ret
	#波动
	volat=np.sqrt(np.dot(normed_weight.T, np.dot(log_ret.cov()*252, normed_weight)))
	exp_vols[ind]=volat
	#计算夏普比率
	SR=exp_ret/volat
	exp_SRs[ind]=SR
#我们找出Sharp Ratio对应最大的Volatility和return那个点，可以用numpy的argmax函数
max_ind=np.argmax(exp_SRs)
max_SR_vol=exp_vols[max_ind]
max_SR_ret=exp_rets[max_ind]
print 'Sharp Ratio maximum is ', exp_SRs[max_ind]
print '-'*20
print 'Volatility is ', exp_vols[max_ind]
print '-'*20
print 'Return is ', exp_rets[max_ind]

#我们以波动为横坐标，总收益率为纵坐标，夏普比率为颜色条，绘制出散点图的分布，并且标识出最大夏普比率的那个点
plt.scatter(x=exp_vols, y=exp_rets, c=exp_SRs, alpha=0.5, cmap='coolwarm')
plt.title('Volatility versus Return')
plt.xlabel('Volatility')
plt.ylabel('Return')
plt.colorbar(label='Sharp Ratio')
plt.scatter(max_SR_vol, max_SR_ret, s=50, edgecolors='black')
```
![三个值的输出结果](https://upload-images.jianshu.io/upload_images/2338511-b3c11788a25fef71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![可视化模拟结果](https://upload-images.jianshu.io/upload_images/2338511-2bd7bbea6e857202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 在这里我们标识出了那个夏普比率最大的那个点。
# 三、数学优化方法
上面是通过随机模拟的方式来找出最优的投资配置，但实际上我们也可以通过边界条件下的优化来找出最优的权重。这里要用到scipy.optimize.minimize方法。
![scipy.optimize.minimize方法](https://upload-images.jianshu.io/upload_images/2338511-0c9dec2006828d8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们可以看到这个函数要做的事情就是给定边界条件
![](https://upload-images.jianshu.io/upload_images/2338511-37692f54cb45c078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-eced507fc0c5aece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后计算这个一个或多个变量所决定的标量目标函数f(x)的最小值。
![](https://upload-images.jianshu.io/upload_images/2338511-fcbc8da509db31c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-1371bece9c87a957.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从官方文档可以看到有几个重要参数，我们在这里只传入几个参数：fun最小化的目标函数，x0是一开始的初始值，method是用于优化的方法，这里选用‘SLSQP’，constraints是约束条件，整体是一个字典，type有两种：‘eq’和'ineq'，分别表示后面传入的约束条件函数'fun'等于0或者大于等于0。
在这里要注意两点，一个是我们是要最大化夏普比率的，所以为了用这个minimize方法，我们应该定义目标函数为夏普比率的负值；第二个是约束条件函数在这里应该是权重之和也就是自变量之和为1，要稍微调整成权重之和减1等于0的约束条件形式才满足minimize方法的要求。
```
#以权重作为输入，计算所得252天的总收益率，总的波动，夏普比率
def get_ret_vol_SR(weight):
	#预期回报率
	exp_ret=np.sum(log_ret.mean()*weight*252)
	#波动
	volat=np.sqrt(np.dot(weight.T, np.dot(log_ret.cov()*252, weight)))
	#计算夏普比率
	SR=exp_ret/volat
	return np.array([exp_ret,  volat, SR])
#定义目标函数，返回夏普比率的负值
def get_neg_sharp_ratio(weight):
	return get_ret_vol_SR(weight)[2]*-1
#定义限制的边界函数，返回权重之和减1的计算结果，之后会用来跟0作比较，判断是否相等
def con(weight):
	return np.sum(weight)-1
#初始化四个值为0.25
x0=np.array([0.25,0.25,0.25,0.25])
#设置自变量的取值范围
bounds=((0,1),(0,1),(0,1),(0,1))
#设置限制条件的字典
constraints={'type':'eq','fun':con }
#进行边界条件下对目标函数优化的权重计算
from scipy.optimize import minimize
res=minimize(fun=get_neg_sharp_ratio, x0=x0, constraints=constraints, method='SLSQP',bounds=bounds)
print res.x#打印出最优化夏普比率后的权重
```
![](https://upload-images.jianshu.io/upload_images/2338511-e9d76e47370fed02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后我们接下来根据计算得到的权重，来带入到我们计算三个值的函数当中求出总的收益率，总的波动，以及夏普比率。
```
print 'Epxcted return is', get_ret_vol_SR(res.x)[0]
print '-'*20
print 'Epxcted volatility is', get_ret_vol_SR(res.x)[1]
print '-'*20
print 'Epxcted return is', get_ret_vol_SR(res.x)[2]
```
![](https://upload-images.jianshu.io/upload_images/2338511-9c1d7459887d8588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、有效边界
有效边界可以这么认为，是一条在给定波动也就是风险下，所能达到的最大收益的那个点，这种类似的点的组合即为有效边界。我们绘制一下这条有效边界。
![](https://upload-images.jianshu.io/upload_images/2338511-176e98d44dc3713e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
#numpy的linspace方法可以生成一系列梯度渐进的值
lin_y=np.linspace(0,0.3,100)
#然后我们在这里定义的最小化的目标函数是波动，也就是在确定的return也就是y值以及权重之和为1的限制条件下，最小化波动值。
def get_vol(weight):
	return get_ret_vol_SR(weight)[1]
frontier_volatility=[]
#初始化四个值为0.25
x0=np.array([0.25,0.25,0.25,0.25])
#设置自变量的取值范围
bounds=((0,1),(0,1),(0,1),(0,1))
for y in lin_y:
	constraints2=[{'type':'eq','fun':lambda x:get_ret_vol_SR(x)[1]-y},{'type':'eq','fun':con}]
	res2=minimize(fun=get_vol, x0=x0, constraints=constraints2, method='SLSQP',bounds=bounds)
	frontier_volatility.append(res2.fun)
print frontier_volatility[:10]
```
输出结果
![](https://upload-images.jianshu.io/upload_images/2338511-5510aaea6ab10a6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们绘制一下边界曲线
```
#我们以波动为横坐标，总收益率为纵坐标，夏普比率为颜色条，绘制出散点图的分布，并且标识出最大夏普比率的那个点
plt.scatter(x=exp_vols, y=exp_rets, c=exp_SRs, alpha=0.5, cmap='coolwarm')
plt.title('Volatility versus Return')
plt.xlabel('Volatility')
plt.ylabel('Return')
plt.colorbar(label='Sharp Ratio')
plt.scatter(max_SR_vol, max_SR_ret, s=50, edgecolors='black')
plt.plot(frontier_volatility, lin_y, 'g--', linewidth=1.5)
```
![有效边界](https://upload-images.jianshu.io/upload_images/2338511-d5a21c75a9be4a6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这里如果从纵向上看，相同的波动情况下，有效边界上面的点能够达到最大的收益，相同的收益下，有效边界上面的点能够达到风险最小，因此有效边界上面的点都是投资配置最优化的点。
