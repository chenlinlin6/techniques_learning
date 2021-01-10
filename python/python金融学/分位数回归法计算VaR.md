## 一、知识点
之前讲完了历史模拟法和蒙特卡罗模拟法计算VaR和ES，接下来要讲的是分位数回归法。回归法的目标和做法是希望用一部分的解释变量来对被解释变量进行拟合，从而作出预测。我们常用的是线性回归，线性回归是一种特殊的回归，它是用来预测被解释变量的均值，也就是二分之一位。而分位数回归则是预测解释变量的特定分位数值，比如95%置信水平的分位数回归法就预测被解释变量该分位数的的值。具体的实现的数学途径由于原文档并没有涉及，之后有需要再研究。本文主要注重代码实现部分。
## 二、数据处理部分
我们先把GOOGL -rq.csv文件下载下来，然后在终端先看一下前几行。
```
head GOOGL -rq.csv
```
![](https://upload-images.jianshu.io/upload_images/2338511-8982fd604dfbd092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从以上的数据可以看出来，一共有三列，第一列是之前计算出来损失变量loss，第二列是滞后一阶波动率（标准差），可以通过Gaussian Garch(1, 1)模型来来拟合得到后取sigma(fit)，也就是拟合以后的值的标准差，然后滞后一阶就可以了；第三列就是第一列损失变量的绝对值滞后一阶的值。
## 三、分位数回归计算VaR值
我们要用quantreg模块的rq函数进行分位数模型的拟合
建立模型拟合
```
#～左边是被解释变量，右边是解释变量可以认为是自变量x，tau是分位数，data传入数据
mm<-rq(loss~sigl+absl,tau=0.95,data=dd)
summary(mm)
```
![](https://upload-images.jianshu.io/upload_images/2338511-670fbc9b4abad87b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

summary函数可以传入一个模型，返回这个模型的参数
**输出结果：**
![](https://upload-images.jianshu.io/upload_images/2338511-101e2182b6587908.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以得出截距，sigl和absl对应的系数，注意的是三者的p-value都是小于0.05的显著性水平的，说明相关性具有一定的显著性。我们如果要预测下一天的loss的话，只需要把前一天的值的带到这个分位数回归方程即可。
我们查看一下数据的最后一天。
![](https://upload-images.jianshu.io/upload_images/2338511-8217920553110e9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后把数值带入这个回归方程当中
![](https://upload-images.jianshu.io/upload_images/2338511-726338cb501e1d37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里我觉得1.204513和0.3365871应该是tail以后的下一行，因为我们都是根据滞后一期（也就是预测当期的值用前一期的sigl和absl，但注意这里sigl和absl是滞后过的，所以它们跟同一行的loss相比是前一期的）的sigl和absl来预测下一期的loss。
