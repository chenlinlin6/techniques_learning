

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
# %cat /Users/chenlinlin/Desktop/通关情况.xlsx|less
```

matplotlib.pyplot设置语法有几种：
1. plt.+模块名字，比如plt.xticks()里面传入参数  
2. 用axes来设置的方法
```
fig,ax=plt.subplot(111)
#在ax里面设置ax.set_模块名字
ax.set_title()
```

3. 绘制的方法
```
plt.plot()
ax.plot()
```

关于matplotlib的原理请[参考链接](https://realpython.com/python-matplotlib-guide/)，这个文章对了解matplotlib.pyplot的组块很有帮助

我们先读取数据


```python
data=pd.read_excel('/Users/chenlinlin/Desktop/通关情况.xlsx',index_col='姓名', decoding='utf-8')
```


```python
data.head()
```







已经成功读取了数据，我们对三列进行汇总统计，画成柱状图


```python
data.sum().plot.bar()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x10e7ecf28>




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-e376d658b00586e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



可以看到我们已经成功画出来了，但还有些问题，比如x轴的tick中文显示不出来，颜色需要改成白色比较显眼，方向变成水平方向，加上x轴和y轴的标题


```python
#我们修改列名为英文
data.columns=['first class', 'Public class', 'Graduation']
data.sum().plot.bar()
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-e9ea65ff9559702b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 字体的设置 

关于字体的设置可以参考[链接](https://matplotlib.org/users/text_props.html)


```python
fig=plt.figure()
ax=fig.add_subplot(111)

data.sum().plot.bar()
plt.title('Activity Vs Number')
plt.xlabel('Activity')
plt.ylabel('Number')
ax.tick_params(direction='out', length=6, width=2, colors='white',
               grid_color='b', grid_alpha=0.5)
```


![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-78b3d3f3c9b2f163.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



也可以对ax进行tick_params的设置使得对x轴或者y轴同时成立或者部分成立

# 颜色的设置 


```python
#我们在这里设置一下颜色
data.sum().plot.bar(color=['lightblue','darkgreen','#eeefff'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-bcc62417ba266201.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



颜色的设置有几种规则：
(关于颜色的设置要参考这个[链接](https://matplotlib.org/users/colors.html).)
1. 按照名字来：
> 比如light/dark+颜色的英文名，比如'lightblue'，就像上面所示。
2. 使用一些简单的记号：  

| alias | color |  
| ----  | ----  |  
| ‘b’   | blue  |  
| ‘g’   | green |  
|‘r’	|red|
|‘c’	|cyan|
|‘m’	|magenta|
|‘y’	|yellow|
|‘k’	|black|
|‘w’	|white|


```python
from IPython.display import Image
Image('/Users/chenlinlin/Downloads/matplotlib.pyplot常用总结/颜色命名规则.png',width=600)
```




![颜色规则](https://upload-images.jianshu.io/upload_images/2338511-0822bceee852aaeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





```python
data.sum().plot.bar(color=['c','m','#eeefff'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-e2b222f0f32d9cef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



3. 传入html16进位制的字符串，比如`'#eeeff'`

比如在这里我们可以获得[口袋怪兽的配色](http://pokepalettes.com/#pikachu)。

比如我们在这里就可以选择皮卡丘的配色
![皮卡丘](https://upload-images.jianshu.io/upload_images/2338511-ec61dbeef0e4986a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把鼠标挪上去以后对应的html16位进制的颜色代码就会出现，直接用就可以了
还有很多其他宝可梦的配色可以选择，只要输入对应的宝可梦的英文名就可以找到对应的宝可梦
![喷火龙](https://upload-images.jianshu.io/upload_images/2338511-a91b9c206f3476b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![水箭龟](https://upload-images.jianshu.io/upload_images/2338511-b2d9d10619f720ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/2338511-50aa786848b14442.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你认得这些颜色分别是哪几只宝可梦的吗？
```python
data.sum().plot.bar(color=['#f6bd20','#9c5200','#de9400'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
plt.grid(alpha=0.5)
```


![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-44ef973dd5d2c303.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



4. 传入一个合法的html name


```python
data.sum().plot.bar(color=['red','burlywood'\
,'chartreuse'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-5143174fff96b3af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



关于颜色的命名可以参考这个[链接](https://matplotlib.org/examples/color/named_colors.html)


```python
data.sum().plot.bar(color=['tab:blue','blue'\
,'chartreuse'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-09e8c60ffeaf263b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



5. 设置一个浮点数，代表一个连续的灰度变化


```python
data.sum().plot.bar(color=['0','0.5'\
,'0.7'])
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-5accedabbbddac15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



设置连续变化颜色的代码


```python
T=np.linspace(0,0.6,3)**2
colors=[(0,0.5,i) for i in T]
data.sum().plot.bar(color=colors)
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-13b94d33b91b8a44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



我们在这里用的是三原色的方式来设置颜色，保证两个通道的值不变，用numpy的linspace方法来递增一系列值来得到递增变化颜色的效果，传入到三原色的最后一个通道值当中


```python
import matplotlib.cm
print(matplotlib.cm.cmap_d.keys())
```

    dict_keys(['Blues', 'BrBG', 'BuGn', 'BuPu', 'CMRmap', 'GnBu', 'Greens', 'Greys', 'OrRd', 'Oranges', 'PRGn', 'PiYG', 'PuBu', 'PuBuGn', 'PuOr', 'PuRd', 'Purples', 'RdBu', 'RdGy', 'RdPu', 'RdYlBu', 'RdYlGn', 'Reds', 'Spectral', 'Wistia', 'YlGn', 'YlGnBu', 'YlOrBr', 'YlOrRd', 'afmhot', 'autumn', 'binary', 'bone', 'brg', 'bwr', 'cool', 'coolwarm', 'copper', 'cubehelix', 'flag', 'gist_earth', 'gist_gray', 'gist_heat', 'gist_ncar', 'gist_rainbow', 'gist_stern', 'gist_yarg', 'gnuplot', 'gnuplot2', 'gray', 'hot', 'hsv', 'jet', 'nipy_spectral', 'ocean', 'pink', 'prism', 'rainbow', 'seismic', 'spring', 'summer', 'terrain', 'winter', 'Accent', 'Dark2', 'Paired', 'Pastel1', 'Pastel2', 'Set1', 'Set2', 'Set3', 'tab10', 'tab20', 'tab20b', 'tab20c', 'Blues_r', 'BrBG_r', 'BuGn_r', 'BuPu_r', 'CMRmap_r', 'GnBu_r', 'Greens_r', 'Greys_r', 'OrRd_r', 'Oranges_r', 'PRGn_r', 'PiYG_r', 'PuBu_r', 'PuBuGn_r', 'PuOr_r', 'PuRd_r', 'Purples_r', 'RdBu_r', 'RdGy_r', 'RdPu_r', 'RdYlBu_r', 'RdYlGn_r', 'Reds_r', 'Spectral_r', 'Wistia_r', 'YlGn_r', 'YlGnBu_r', 'YlOrBr_r', 'YlOrRd_r', 'afmhot_r', 'autumn_r', 'binary_r', 'bone_r', 'brg_r', 'bwr_r', 'cool_r', 'coolwarm_r', 'copper_r', 'cubehelix_r', 'flag_r', 'gist_earth_r', 'gist_gray_r', 'gist_heat_r', 'gist_ncar_r', 'gist_rainbow_r', 'gist_stern_r', 'gist_yarg_r', 'gnuplot_r', 'gnuplot2_r', 'gray_r', 'hot_r', 'hsv_r', 'jet_r', 'nipy_spectral_r', 'ocean_r', 'pink_r', 'prism_r', 'rainbow_r', 'seismic_r', 'spring_r', 'summer_r', 'terrain_r', 'winter_r', 'Accent_r', 'Dark2_r', 'Paired_r', 'Pastel1_r', 'Pastel2_r', 'Set1_r', 'Set2_r', 'Set3_r', 'tab10_r', 'tab20_r', 'tab20b_r', 'tab20c_r', 'magma', 'magma_r', 'inferno', 'inferno_r', 'plasma', 'plasma_r', 'viridis', 'viridis_r', 'cividis', 'cividis_r'])



```python
np.linspace(0,1,20)
```




    array([0.        , 0.05263158, 0.10526316, 0.15789474, 0.21052632,
           0.26315789, 0.31578947, 0.36842105, 0.42105263, 0.47368421,
           0.52631579, 0.57894737, 0.63157895, 0.68421053, 0.73684211,
           0.78947368, 0.84210526, 0.89473684, 0.94736842, 1.        ])




```python
color_map=matplotlib.cm.BuGn(np.linspace(0.4,0.5,3))
color_map
```




    array([[0.56      , 0.82980392, 0.75921569, 1.        ],
           [0.47843137, 0.79461745, 0.70003845, 1.        ],
           [0.39772395, 0.75955402, 0.64030757, 1.        ]])




```python
T=np.linspace(0,0.6,3)**2
colors=[(0,0.5,i) for i in T]
data.sum().plot.bar(color=color_map)
plt.title('Activity Vs Number',color='white')
plt.xlabel('Activity',color='white')
plt.ylabel('Number',color='white')
plt.xticks(rotation=0,color='white',backgroundcolor='b')#设置xticks的方向
plt.yticks(color='white')
```




    (array([ 0., 10., 20., 30., 40., 50., 60., 70.]),
     <a list of 8 Text yticklabel objects>)




![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-20a019999f8f4bc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



设置连续变化颜色的第二种方法是按照colormap的方法来设置

matplotlib的colormap的直观图


```python
Image('/Users/chenlinlin/Downloads/matplotlib.pyplot常用总结/colormap.png',width=600)
```




![颜色条](https://upload-images.jianshu.io/upload_images/2338511-47027e808dd01606.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




在这里，colormap是一些连续变化的条，我们通过`np.linspace`可以获得一些间断的值，这些间断的值对应去去这个colormap上面的颜色rgb值，然后这些rgb值可以被我们所用

获取colormap有两种方法：
1. 参考链接，[colormap](https://matplotlib.org/examples/color/colormaps_reference.html)
2. 用`maplotlib.cm.cmap_d.keys()`方法
除了以上的自己设置颜色的方法，也可以用一些现有的解决方案快速解决问题。
比如我们可以直接套用style
```python
plt.style.use('ggplot')#用ggplot的主题
```

也可以导入seaborn的包，其颜色主题会随之修改，好看很多
```python
import seaborn as sns
```
```python
data.columns=['first class', 'Public class', 'Graduation']
data.sum().plot.bar()
plt.title('Activity Vs Number',color='black')
plt.xlabel('Activity',color='black')
plt.ylabel('Number',color='black')
plt.xticks(rotation=0,color='black')#设置xticks的方向
plt.yticks(color='black')
```
![绘图结果](https://upload-images.jianshu.io/upload_images/2338511-b7b72804fb4ab90f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上两种方法效果是等效的，用哪种都可以。
# 关于matplotlib的设置--通过修改[`matplotlib.rcParams`](https://matplotlib.org/api/matplotlib_configuration_api.html#matplotlib.rcParams "matplotlib.rcParams")实现

![](https://upload-images.jianshu.io/upload_images/2338511-308c7054dd6d7ab8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 暂时修改
matplotlib有许多现成的样式在`plt.style`中，可以通过`plt.style.avaliable`来看到
```python
['seaborn-ticks', 'ggplot', 'dark_background', 'bmh', 'seaborn-poster', 'seaborn-notebook', 'fast', 'seaborn', 'classic', 'Solarize_Light2', 'seaborn-dark', 'seaborn-pastel', 'seaborn-muted', '_classic_test', 'seaborn-paper', 'seaborn-colorblind', 'seaborn-bright', 'seaborn-talk', 'seaborn-dark-palette', 'tableau-colorblind10', 'seaborn-darkgrid', 'seaborn-whitegrid', 'fivethirtyeight', 'grayscale', 'seaborn-white', 'seaborn-deep']
```
这些样式都放在了`mpl_configdir/stylelib/`目录下，你甚至可以自己设置`.mplstyle`后缀名的文件，放到该目录下。
比如设置名为`presentation.mplstyle`的文件，放到该目录下
```python
axes.titlesize : 24
axes.labelsize : 20
lines.linewidth : 3
lines.markersize : 10
xtick.labelsize : 16
ytick.labelsize : 16
```
要用的时候就直接用
```python
plt.style.use('presentation')
```
## 避免全局影响
如果你只是想这个地方用一用，而不是应用到全局。可以考虑用`plt.style.context`这个管理器。
```python
with plt.style.context(('dark_background')):
  plt.plot(np.sin(np.linspace(0, 2*np.pi)), '-o')
plt.show()
```
![](https://upload-images.jianshu.io/upload_images/2338511-1a130e54df643a3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 修改设置表
你还可以通过修改`plt.rcParams`的方式来实现样式的修改，这个修改会直接反映到packages上面。
修改的语句有两种：
第一种
```python
matplotlib.rcParams['lines.linewidth']=2
plt.plot(data)
```
![第一种方法](https://upload-images.jianshu.io/upload_images/2338511-3000e6c31e8f17e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
语法是`mpl.rcParams['部件.部件参数']=你想设置的值`
第二种
```python
mpl.rc('line', lindwidth=4,color='r')
plt.plot(data)
```
语法是`mpl.rc(部件, 部件所具有的参数*)`
![第二种方法](https://upload-images.jianshu.io/upload_images/2338511-a7801604ec8dc7ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关于里面传的具体参数的名字，参考[官方文档](https://matplotlib.org/tutorials/introductory/customizing.html)。
![关于修改部件的说明](https://upload-images.jianshu.io/upload_images/2338511-69a6117df8a8757f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
也可以根据python的kwargs的语法传入一个字典。
比如
```
lines={'linewidth':1.5, 'linestyle':'--','color':'g'}#这是我要赋值的参数名字和参数
mpl.rc('lines', **lines)
```
然后绘制正弦曲线
```
plt.plot(np.sin(np.linspace(0, 2 * np.pi)))
```
![修改样式后的正弦曲线](https://upload-images.jianshu.io/upload_images/2338511-a2f53f5fff1c3da4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一般来说，如果分析工作用的样式是统一的，最好是修改style文件，或者是在前面用rcParams的方法去统一接下来绘制的图的相同样式。
# 关于maplotlib的其他用法 


我之后还会一直更新，目前提供一张小抄供有需要的人去对照着看一下
```python
Image('/Users/chenlinlin/Downloads/matplotlib.pyplot常用总结/cheatsheet.png',width=600)
```




![matplotlib小抄](https://upload-images.jianshu.io/upload_images/2338511-f5a78dc41163f34a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




参考[链接](https://s3.amazonaws.com/assets.datacamp.com/blog_assets/Python_Matplotlib_Cheat_Sheet.pdf)
