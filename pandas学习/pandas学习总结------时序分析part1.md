# 时间序列

# 日期和时间类型工具

python标准库处理时间的模块：`time`, `calendar`和`datetime`

pandas本身自带处理时间的工具：`pd.to_datetime`

## 创建时间类型

```python
import numpy as numpy
import pandas as pd
numpy.random.seed(12345)
import matplotlib.pyplot as plt
plt.rc('figure', figsize=(10, 6))
PREVIOUS_MAX_ROWS = pd.options.display.max_rows
pd.options.display.max_rows = 20
numpy.set_printoptions(precision=4, suppress=True)	
from datetime import datetime
```

```
dt=datetime(2011,1,1)
dt
```

![](https://upload-images.jianshu.io/upload_images/2338511-db670bde5191042c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`datetime`模块对应传入年月日参数，产生python时间格式的对象。时间对象之间可以方便做相差的时间间隔计算，也可以调用各种时间属性`day`, `year`和`month`等。

![](https://upload-images.jianshu.io/upload_images/2338511-dc7fce86026f5315.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 日期间的差值

时间戳之间的差值是`delta`对象，`delta`对象也可以用`timedelta`方法创建并用于计算。

```python
from datetime import timedelta
delta=timedelta(12)
dt+delta
```

![](https://upload-images.jianshu.io/upload_images/2338511-26aaa884b5c554d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认传入的`timedelta`里面的参数是**天数**。

```python
now=datetime.now()
now-dt
```

![](https://upload-images.jianshu.io/upload_images/2338511-752bf09341dc2daf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 字符串和datetime的相互转换

![](https://upload-images.jianshu.io/upload_images/2338511-c79324ae10658517.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 字符串转换为datatime用`strptime`

这里的`p`代表`parsing`的意思

```python
value = '2011-01-03'
datetime.strptime(value, '%Y-%m-%d')
```

![](https://upload-images.jianshu.io/upload_images/2338511-ec053b431d1e8f0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/2338511-9a6b0e38c23fb4af.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以指定解析的样式。

当然，很多时候我们如果都要写好解析的样式`%Y-%m-%d`那样是比较麻烦的，在这种情况下我们可以用`datautil.parser.parse`方法去自动解析，支持大部分可以识别的日期格式。

```python
from dateutil.parser import parse

parse('20111101')
```

![](https://upload-images.jianshu.io/upload_images/2338511-636632d09951c1e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2338511-11818437a80f637a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 索引、选择、子集

如果以**日期**作为序列的话，可以传入一个可以被解释为时间的索引进行选择和切片。

```python
longer_ts = pd.Series(numpy.random.randn(1000),
                      index=pd.date_range('1/1/2000', periods=1000))
longer_ts
```

![](https://upload-images.jianshu.io/upload_images/2338511-ca974b91be9235bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下三种方式切片都是获得相同的结果。

```python
longer_ts['1/10/2001']
longer_ts['20010110']
longer_ts['2001-01-10']
```

![](https://upload-images.jianshu.io/upload_images/2338511-93e06099e7c08df6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们也可以把频率获取到月份。

```python
longer_ts['2001-01']
```

![](https://upload-images.jianshu.io/upload_images/2338511-257b649c48603fa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

索引既可以传入时间字符串，也可以传入`datetime`对象

```python
longer_ts[datetime(2001, 1,1):]
```

![](https://upload-images.jianshu.io/upload_images/2338511-77ee41d368f00f2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 日期范围、频率和移位

`pandas.date_range`可以生成指定的长度的`DatetimeIndex`：

```python
pd.date_range('2018-01-01', '2018-05-01')
```

![](https://upload-images.jianshu.io/upload_images/2338511-756154dcd225e484.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们还可以指定`periods`和`freq`参数指定周期和频率

```python
rng = pd.date_range('2000-01-01', periods=3, freq='M')
ts = pd.Series(numpy.random.randn(3), index=rng)
ts
```
![](https://upload-images.jianshu.io/upload_images/2338511-798a34cec2fa5733.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了`start`参数，我们还可以传入`end`参数。
如果是`end`参数的话就是往前追溯。

```python
rng = pd.date_range(end='2000-01-01', periods=3, freq='M')
ts = pd.Series(numpy.random.randn(3), index=rng)
ts
```
![](https://upload-images.jianshu.io/upload_images/2338511-61ec401ad272ddd9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)