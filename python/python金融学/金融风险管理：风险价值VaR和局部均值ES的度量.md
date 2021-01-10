>关于时序分析的补充，为什么最后要检查残差之间是否具有相关性，这是因为我们认为一个合理的模型应该是理论上能够将相关性尽可能地刻画出来，对于一堆数据，如果你扣除掉相关性之后，剩下来的应该是一些没有相关性的随机扰动，所以要检查残差之间的相关性，如果没有相关性了，才说明这个模型能够比较准确地反映出数据的相关性，否则就说明还有一部分的相关性没有反映出来，这个模型本身是不准确的。
## 一、风险价值VaR和局部均值ES的概念
### 1.1 风险的概念
风险是与收益相对应的概念，正是因为市场具有波动性，既有获得收益的可能，也有可能造成损失的可能，造成损失的可能就是风险。在风险管理当中我们看重的是风险，而风险的来源是**不确定性**，也即是**波动**。虽然是不确定性，但是假如我们给定一定的假设，建立一套模型，可以在某种程度上理解风险出现的可能性以及对我们造成的影响，不一定能避免风险，但是能够增加对它的理性认识。有三个问题是我们要思考的：
1. 风险是什么？
2. 如何测度风险
3. 为风险我们应该做什么准备？
### 1.2 如何测度风险
关于第一个问题在上小节已经说过，那么说一下如何测度风险。风险度量工具常用有：
- 债券市场：久期
- 股票市场：贝塔系数
- 衍生品市场：delta
- 新标准：VaR，ES
新出现的标准VaR和ES相对于其他的风险度量工具来说对于没有专业背景的普通大众来说更加容易理解。
#### 风险度量VaR
VaR指的是在一定置信区间内和一定时间内的**最大损失金额**。
举个例子。某个银行发行某一种基金或者资产组合，它在1天期限内的99%的风险度量VaR为6000万元。
关于这点可以有以下三种理解：
1. 在一天后损失掉6000万元的可能性有1%。
2. 给定100天，可能会有一天遭受到6000万元的损失。
3. 有99%的可能在这一天的时间内不会有大于6000万元的损失。

**绘制概率密度图**
```
curve(dnorm(x),from=-3,to=-3,las='1',xlab='return',ylab='p(x)',main='Definition of VaR Based on the Probabilty Density Function')
#绘制箭头，length确定箭头的大小
arrows(0,0.15, qnorm(0.05), 0.15,length=0.15)
#abline能够在图上画线
abline(v=0,lty=1)
abline(v=qnorm(0.05), lty=2)
text(x=qnorm(0.05), y=0, labels='5%')
text(x=-1,y=0.15,labels='VaR')
```
![标准正态分布](https://upload-images.jianshu.io/upload_images/2338511-ac15176bab85c5d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图画的是一个标准正态分布分布，从-3到3.
其中95%的置信区间对应的分位点即是VaR值，所以VaR值是相对于没有损失风险的一个分位数。
绘制累积分布函数
```
curve(pnorm(x),from=-3,to=3,las='1',xlab='return',ylab='p(x)', main='Definition of VaR based on the cumulative distribution function',cex.main=0.8)
abline(h=0.05,lty=2)
axis(side=2,at=0.05, labels='0.05', las='1', par(cex=0.6))
```
![累积分布函数](https://upload-images.jianshu.io/upload_images/2338511-67aee0cd0295bc4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



VaR有自身的缺点，不满足次可加性原则（关于次可加性我还是不是很理解），所以没有办法计算资产组合的VaR值。同时对于尾部的刻画，我们一无所知，也就是我们关心的是置信区间里面的事，但是对于万一我们确实损失值出现在了置信区间以外的话，这个损失的尾部分布是如何的呢？期望又是如何？这点VaR没有办法告诉我们，但是ES可以弥补以上两个缺点。
#### 尾部均值ES
ES是指损失超过了VaR以后，尾部损失的一个**期望值**。计算公式如下：
![尾部均值](https://upload-images.jianshu.io/upload_images/2338511-9d4fa17f27eb8839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 二、Delta-normal方法计算VaR和ES
### 引言
照理来说，给定一定的置信区间和时间T，对照着正态分布的表格应该可以查找出对应的VaR值。但实际上收益率的分布并不满足正态分布，但是模型的作用并不是反映出细枝末节，而是给定一定的前提假设，这个模型能够有多大程度能够接近现实？
### 前提假设
对于一个投资组合，Delta-normal方法的前提假设有两个：
1. 收益率序列满足正态分布。
2. 资产的收益率是各个部分收益率的线性组合（不存在风险对冲）。
### 公式推导
从以上这个假设我们知道了**资产的收益率组合是满足正态分布的**，而我们要求VaR，根据概念就是把这个**正态分布的分位数**找出来。我们知道，正态分布最重要的两个参数是均值还有标准差（或者方差），分别决定了分布的平移和拉伸压缩。在这里我们用标准差，不用方差，原因是标准差与均值具有相同的单位。在经济学或者风险管理当中，统计学当中的sigma通常叫做**波动率**，实际上是一个意思。
假设我们有价值为1的资产组合，给定置信区间为c，收益率的均值为0（标准正态分布的收益率为0）那么计算未来一天的![](https://upload-images.jianshu.io/upload_images/2338511-e4e514a1f6c4cbd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，这里的alpha就是标准正态分布q的分位数，其中q=1-t。
我在这里一开始有点不太理解，这里可以这么理解，对于标准正态分布来说，均值为0，标准差为1，那么如果资产的遭受损失大于某个值的可能性是q（比如一般是5%），那么这里标准正态分布当中相对于不标准的正态分布是相当于归一化的存在，alpha在其中就相当于是指出了偏离均值u有多少个标准差的距离（因为标准正态分布的标准差是1），然后偏离的标准差个数再乘以收益率序列的标准差sigma，也就知道了原本的投资组合VaR值。其实在这里我更愿意把VaR解释为一定置信水平下，给定时间，偏离均值的最远距离是多少。
根据时间的[平方根规则](https://quant.stackexchange.com/questions/7495/square-root-of-time)（我这里也不懂，得进一步研究），如果是对于投资资产为W0的资产组合，那么在未来T天内，VaR值也就是乘以W0倍，也就是：
![](https://upload-images.jianshu.io/upload_images/2338511-46367b07adfe6572.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 在这里alpha是可以通过置信水平找出来的，关键在于波动率sigma怎么求？所以接下来的重点在如何通过历史估算投资资产收益率的波动率。

## 三、GARCH模型
建模之前，我们需要了解的是，我们是根据历史来建模的，也就是认为过去历史是包含着一定的趋势的，并且这个趋势是会延续下去的（但我们知道随时可能会有新的冲击），并且我们要理解模型是为了刻画出数据的趋势的，真实数据与预测值之间会有残差，但真实数据扣除掉预测值之后留下的残差应该是随机波动的，也就是它们不会有相关性，这样才能说我们这个模型把数据的趋势挖掘得够彻底了。
假如收益率序列为rt，rt是由两部分组成的：本身的均值ut以及随机扰动项at。表示如下：
![](https://upload-images.jianshu.io/upload_images/2338511-03a44ca5f2a6cc02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，其中ut是满足ARMA(p, q)模型的，也就是前面一项是p个滞后项的历史收益率的自回归项，后面一项是q个滞后项的移动平均项。如下表示：
 ![](https://upload-images.jianshu.io/upload_images/2338511-6850ae5fc48c9735.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（均值方程）
 在开始介绍两种模型之前，先说下区别，其实ARCH模型只是GARCH的一个特例，GARCH是更一般化的（G代表就是Generalized的意思）。GARCH描绘的是t时刻的方差是历史的扰动项平方的线性组合加上历史的方差线性组合，而ARCH描绘的只有历史的扰动项平方面回归。知道了这一点之后，我们开始进行底下的公式推导部分。
###  ARCH模型
ARCH(p)模型假设：
t时刻的扰动项
![](https://upload-images.jianshu.io/upload_images/2338511-d2f30452fc39d86d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-aecd48abacac171d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中扰动项的因子epsilon t范围是(0, 1)，![](https://upload-images.jianshu.io/upload_images/2338511-74877bcea90cacde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，![](https://upload-images.jianshu.io/upload_images/2338511-d98b654765cdc492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（波动方程）
### GARCH模型
与ARCH模型不同的是，除了有扰动项的线性组合外，还有q项历史滞后项sigma的移动平均项。
GARCH(p, q)模型假设：
![](https://upload-images.jianshu.io/upload_images/2338511-46dc4fa635e94264.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-f634cf02099a0628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中扰动项的因子varepsilon t范围是(0, 1)，![](https://upload-images.jianshu.io/upload_images/2338511-36f831e596b00f38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，![](https://upload-images.jianshu.io/upload_images/2338511-762ef03484c7423a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，![](https://upload-images.jianshu.io/upload_images/2338511-a45f0c1ebffd1620.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ，![](https://upload-images.jianshu.io/upload_images/2338511-6c06deb2e25b7c18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（波动方程）
两个方程刻画出来的其实是收益率波动其实与扰动项不是成线性的相关，而是与扰动项的二次方成相关性。刻画的是二阶扰动相关性，而不是线性自相关性。更多请参考Ruey的[Analysis of Financial Time Series](https://book.douban.com/subject/4719140/)这本书。
## 四、RiskMetrics方法
RiskMetrics是JP Morgan提出的风险度量技术，这里只涉及简单形式。这个方法认为的是，对于t时刻的扰动项$a_{t}$，给定t-1时刻的信息，那么$a_{t}$是满足正态分布的。其中，sigma的表示方法如下：
![](https://upload-images.jianshu.io/upload_images/2338511-67e4168b193d4a88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-5aabfff1f7b90f4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里的RiskMetrics如果与以上的GARCH模型对比，会发现其实只是GARCH(1, 1)模型扣除掉漂移项滞之后的产物，并且在这里规定GARCH里面的![](https://upload-images.jianshu.io/upload_images/2338511-363568fb965f46bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 五、数据处理部分
### 5.1 实例部分
接下来我们要通过谷歌五年的历史数据预测未来一周内可能遭受的最大损失和损失的平均值。
首先获取数据：
```
wget http://labfile.oss.aliyuncs.com/courses/954/GOOGL.csv
```
然后我们接下来要用到rugarch这个工具包，可以通过install.packages()这个方法来安装。
```
install.packages('rugarch')
library('rugarch')
#接下来读取csv文件，并将其存储到da这个变量当中
da<-read.csv('GOOGL.csv')
head(da)#打印前几行看一看
dim(da)#查看以下da的形状
```

接下来我们将损失变量求出来，把负对数收益率百分比化后作为损失变量
```
loss<-(diff(log(da$Adjust.Close)))*100
head(loss)#打印出前几行看一下
```
## 六、建模部分
### 6.1 GARCH模型求VaR和ES
#### 6.1.1 建模步骤
建模包括两部分：均值方程和波动方程。均值方程是满足ARMA(p, q)模型，因此要进行建立ARMA(p, q)模型一般的步骤：
![ARMA建模流程](https://upload-images.jianshu.io/upload_images/2338511-924b7066d198ec6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于波动方程则首先要检验ARCH效应，也即是检验残差项是否二次相关。
![](https://upload-images.jianshu.io/upload_images/2338511-d7c0a2fc9f935648.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里说明一下，其实我们做了这么多，都是在提纯**相关性**，ARMA刻画的是线性相关，而GARCH刻画的是非线性相关性。我们在**不断地剔除掉相关性**，这样当相关性被完全剔除掉之后，剩下的就是随机波动的白噪声，比如在这里最后的ARCH模型建立完之后最后一个要做的就是扣除掉残差的残差后，是否剩下来的白噪声是满足一定的分布（比如GARCH就要求满足正态分布，只有这样，我们才能相信这样的白噪声是天然就存在的噪音，没有包含主要信息）。我们在前面建立了ARMA模型之后，剔除的是线性相关性，但是剩下来跟均值的差（也就是差异）是一个波动，根据我们上面提到的GARCH模型，它是可能存在着二次序列相关的，所以我们在GARCH模型建模的时候，相当于对这部分波动进行二次相关性的拟合（跟ARMA建模是一样的操作），然后再检验上一步ARMA的'残差'的残差是否还具有相关性，如果没有了，就说明相关性刻画完全了，否则还得重新选择参数，建立更好的GARCH模型去拟合这部分残差。然后最后扣除所有相关性到最后，就是白噪声了，要看这个白噪声是否真的那么无辜，所以就看它是否满足正态分布。
在这里的话，其实收益率的自相关性是十分微弱的（否则人人都可以轻松预测套利），所以就不必建立ARMA模型，直接以算术平均值来代替均值方程，接下来会重点建立GARCH模型。
#### 6.1.2 建模代码
我们直接运用GARCH(1, 1)模型，关于模型的选择一般p，q不超过2，关于模型选择和检验这里不做探讨。
```
#定义GARCH模型的参数
spec1<- ugarchspec(mean.model=list(armaOrder=c(1, 1),include.mean=True), variance.model=list(garchOrder=c(1, 1)))
#用上述模型拟合数据
fit1<-ugarchfit(data=loss, spec=spec1)
#向前5步预测
forecast5<-ugarchforecast(fit1, n.ahead=5)
```
我们可以看到我们的ugarch模型当中的均值方程的参数mean.model设置的阶次p,q为(0, 0)，并且包含了均值项，说明我们这里以简单的算术平均值作为均值方程。variance.model的波动方程的阶次p, q设为(1, 1)，然后根据历史损失率来建模，向前5步预测一周的情况，设置n.ahead=5。
```
# 接下来我们看一下预测数据的标准差或者说是波动率
> sigmahat<- sigma(forecast5)
> sigmahat
```
输出结果
> T+1, 1.127330
> T+2, 1.235484
> T+3, 1.311291
> T+4,  1.265775
> T+5,  1.405538


我们接下来把5天的方差加和，就可以得出![](https://upload-images.jianshu.io/upload_images/2338511-9ecb2f41ba199522.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，然后我们就可以计算VaR还有ES。
 ![](https://upload-images.jianshu.io/upload_images/2338511-ac8d87d5eb7317a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#5天的方差总和
wvarforce<-sum(sigmahat^2)
#带入上面的公式，其中在这里把上面提到的sigma^2 T当作一个整体
VaR<-qnorm(0.95)*sqrt(wvarforce)
#计算尾部均值
ES<-dnorm(qnorm(0.95))*sqrt(wvarforce)/0.05
print (cbind(VaR, ES))
```
输出结果
> VaR, ES
> 4.755209, 5.963223

这就说明在95%的置信水平下，5天里面最大可能损失不超过¥1000000 x 4.755209% =¥4755209，发生损失的均值为¥1000000 x 5.963223% =¥5963223
在这里我多补充一下，原本有点不太理解ES的计算，我们在这里仔细看一下其实qnorm(0.95)就是返回95%置信水平下的分位数，dnorm函数则是返回这个分位数下的密度概率，0.05则是尾部的累积概率（可以理解为95%置信水平之后所有可能发生的损失值，也就是左边的那块面积），所以人如其名，尾部均值就是求在0.95对应的分位点下的概率与对应的损失大小占总的左边的那块面积的大小（可以理解为最终发生的损失既跟发生的概率有关也跟该概率下发生损失的大小有关）。
![](https://upload-images.jianshu.io/upload_images/2338511-ddd39d781d787b72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的方法其实并没有严格套原本求VaR那个公式，而是直接把5天的![](https://upload-images.jianshu.io/upload_images/2338511-c9f0fa3375e085d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
作为一个整体，其实我们也可以先预测1天的sigma，然后乘以sqrt(5)，结果稍微有些不同。
### 6.2 RiskMetrics计算VaR和ES
在这里RiskMetrics的建模方式和GARCH的建模方式是相同的，不同的在于要选择参数(p, q)为(1, 1)，并且没有漂移项alpha0，在模型的参数model选择'igarch'即可。
```
#定义igarch模型的参数
spec2<-ugarcrhspec(mean.model=list(armaOrder=c(0, 0), include.mean=False),variance.model=list(model='iGARCH',garchOrder=c(1, 1)))
#拟合数据
fit2<-ugarchfit(data=loss, spec=spec2)
#向前一步预测
forecast1<-ugarchforecast(fit2, n.ahead=1)
#提取预测结果的波动率
sigmahat2<-sigma(forecast1)
#计算VaR
VaR<-qnorm(0.95)*sqrt(5)*sigmahat2
#计算ES
ES<-dnorm(qnorm(0.95))*sigmahat2*sqrt(5)/0.05
#打印计算结果
print (abind(VaR, ES))
```
输出结果
> VaR, ES
> 3.828855, 4.801539

这就说明在95%的置信水平下，5天里面最大可能损失不超过¥1000000 x 3.828855% =¥ 3828855，发生损失的均值为¥1000000 x 4.801539% =¥4801539
在这里的计算结果与GARCH模型预测的结果是有差异的，说明模型和参数(p, q)的选择对计算结果是有影响的。
## 七、总结
在这篇文章当中，我们介绍了VaR和ES的概念，GARCH模型，ARCH模型以及RiskMetrics方法计算VaR和ES的方法和流程，关键点在于对波动率的拟合，除了要知道怎么计算，还要知道什么时候能用这个模型。
## 参考资源
1. [实验楼：金融风险管理VaR和ES](https://www.shiyanlou.com/courses/954)
2. [RiskMetrics的维基百科介绍](https://en.wikipedia.org/wiki/RiskMetrics)
3. [VaR和绝对VaR的参考书目Value at Risk](https://book.douban.com/subject/1877500/)
4. [rugarch帮助文档](https://cran.r-project.org/web/packages/rugarch/rugarch.pdf)
5. [Ruey的Analysis of Financial Time Series](https://book.douban.com/subject/4719140/)
