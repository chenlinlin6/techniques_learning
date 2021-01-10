# hive SQL操蛋问题解决指南

最近进入了一家新公司，开始学习Hive SQL的各种函数和语法，并且完成19个变量的计算，在整个过程中，我觉得还是遇到很多大坑的，因此我在这里做点总结，以梳理下思路。

# 总体

首先是要明确上级交代的任务。其实在这个过程中，我还是摸索了比较久的，因为我对各种表格其实不是很熟悉，字段的含义，以及变量的计算逻辑都并不是特别清楚，所以在这上面花了比较多时间。
其次就是需要参考已有的模板。在自己动手写代码以前，先好好参考一下已有的代码，当时leader就已经先给我他以前开发相似变量的代码，可是我当时着急开发，导致很多东西没想明白就去干活，后期反复调整浪费了不少时间。

## 调试代码的问题

hive sql查询命令的特点就是时间特别长，因此整个过程当中我很多时间都花在了等待跑完代码上面。试想一下，如果辛辛苦苦写完代码，长时间等待跑完之后，看到结果发现忘记加限制条件了，那又得重头跑一遍。一次两次还可以，如果反复这样，浪费的时间是很多的。因此在让代码跑起来之前一定要先思考自己要的结果，要检验的事情等等，然后注意有没有加关键的限制条件。

### **关键的限制条件有这些：**

1.一定的时间范围内

2.去除null值

> 比如`where bank_no is not null and bank_no not in ('null', 'NULL', '')`
>
> 之所以要去除null值，除了是因为null值本身对我们一般数据分析没有太多用处外，还有就是在进行关联的时候null值会产生很多次内部关联，是指数级的。

3.去除重复值

> 和null值类似，在进行关联的时候，如果有重复值，也会大大增加关联的时间成本。
>
> `select field_3 from (select distinct field_1,field_3 from 024_dynamic) a left join (select distinct field_2 from 023_dynamic) b`

4.关键字连接是否足够清晰

> 我遇到其中一个情况是，每个账单都有相应的流水记录，bill和record用rid和month关联，如果我们只用month进行关联的话，那么就会有很多重复的，这样也是要运行时间很长，而且结果也不是我们想要的。如果我们多加一个rid关键字关联的话，那么就相当于更加精准地匹配到了对应的数据进行关联。
5.是否有加关联条件on
如果不加关联条件a.user_id=b.user_id等关键字段，就会两张表之间每个字段都关联一遍，这样是指数级的运算。

### 运行时间过长的可能问题：

我经常有遇到运行时间过长的问题，可以从日志当中就看出。比如一段代码运行到了99%，接下来还是99%，然后还是99%……那就很明显是有问题的，反映出的问题是，计算量很大，这种情况几乎都是在表格关联的时候出现，也基本上是以上出现的问题。首先是否有去除重复值？关键字连接是否足够清晰能够匹配到唯一的数据？空值是否去除？一般我遇到的问题是这么几条。

### hive常用的一些技巧

1.如果现在有个表格，有历年销售额的数据，如何获得每年与上一年的销售额差别呢？在同一行数据当中既要有某一年的数据，又要有上一年的数据，那么我们可以用对应的年份+1再跟年份关联一下，那么+1的年份的数据就会挪到下一行，你想想是不是这个道理？

![024_dynamic](https://upload-images.jianshu.io/upload_images/2338511-81854c4d4a51d183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

比如我们现在有`024_dynamic`这张表格，我们尝试把年份+1以后再与原表格相关联。

```sql
select * from 024_dynamic a
left join 
(select field_2, field_3, field_4
	from 024_dynamic
) b
on a.field_4=b.field_4+1
```

![返回结果](https://upload-images.jianshu.io/upload_images/2338511-1cbb8a5c21e189c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到年份+1之后再关联，右边的数据就会往下错开了一个年份，相当于是每年的数据和上一年的数据放在同一行了，这样方便操作。

2.去重的方法

去重的方法大家都知道用`distinct`，但实际上，`distinct`所能运用的场景其实比较狭窄。比如说一个淘宝用户的流水账单有`user_id`， `card_no`，`month`，`id`，`outcome`等字段，实际上来说交易一次会产生一个id，但有些情况下有可能会产生重复的id。那如果我们要选取独一无二的交易记录，用这样的语句`select distinct user_id, card_no, month, id, outcome from bill表`?
这样是不行的，原因是数据与数据之间只有上述5个变量同时重复才会算是一条重复数据，而在这里outcome有可能不重复，而我们只是想要每个月份下每个id对应唯一一条数据，这个时候可以用`partition`的方法，也就是对id进行标记序号的方法。

```sql
select distinct 
user_id, 
card_no, month, 
id, row_number() over (partition by id) as id_partition,
outcome from bill表
where row_number() over (partition by id) =1
```

在这里我们以`024-dynamic`这张表为例子，我们可以看到对field_3字段进行partition by之后，相同变量之间多了序号，在新的变量之后重新开始新的序号，也就是变量内的排序。
![partition by](https://upload-images.jianshu.io/upload_images/2338511-6d960ad6bbab1e3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以的话，相当于如果我们只关心field_3，类似于我们的id是否有重复，只需要选取id_partition=1的筛选条件就可以了。

### hive的几条心法

我认为做hive的计算其实有几条心法是一定要注意的。

1. 首先就是，我们做的运算，除了窗口函数和聚合函数外，其实很多都是列间运算，而不是跨行的运算，也就是在同一行进行的运算。所以在上面有要求每年与上一年的差别（或者是其他复杂运算），这些其实都是将要操作的数据都放在了同一行当中，然后程序每次select一行显示出来（当然实际上肯定不是一行一行操作，应该是分块）。比如我们用以下的语句，

```sql
select field_4, if(field_4='2013', 1, 0) from 024_dynamic;
```

在这里我们从024_dynamic这张表当中取出年份变量，并且判断年份变量是否是2013年，是的话返回1否则返回0.
**大部分的操作语句都是在一行里面的列间操作，所以下次有什么运算的时候记得先放到同一行。**
![if对同一行的数据操作](https://upload-images.jianshu.io/upload_images/2338511-2f0ce237253c2095.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 条件筛选的方式
   说起条件筛选，大家头脑里面肯定想到的是一般表格当中的`where`语句，以及group by以后的`having`语句。但实际上这两个筛选其实有一些不方便的地方，而一个更广阔的筛选方法是用关联(left join, join, right join)的方式进行，这个是个用法很广的方式。最常见的情况就是，被筛选的表格是一张表，要生成筛选条件的是另外一张表。
   比如说，一张信用卡(card表)会有多个月的账单(bill表)，多个流水记录(record表)，它们依次关联（也就是card表不可以越过bill表与record表关联），我想找出过去我想找出过去一年中有5个月以上消费支出总和都在2000元以上且账单应还金额在2500以上的信用卡。那么在这里就有好几个条件了，听起来好像很复杂，其实拆解下来都是可以依次进行的小行动。在这里稍微理一下思路。

2.  对同一张表的满足不同条件的部分进行各自的操作

   这个听起来有点绕，其实也就是比如有一张学校的体能测试的表格，有个字段是“达标与否”，将学生分为两类，达标与未达标的。现在需要对同一张表格的达标与未达标的学生的体重做一些计算，但是达标与未达标的计算公式是不同的，所以要分开计算。其实之前在解决相关问题的时候我纠结了很久，试了用关联的方式，试了奇奇怪怪的方法，后来发现，最简单的解决方法就是老老实实把筛选出达标的学生计算，筛选出未达标的学生进行计算，然后计算结果用`union all`的方式进行行拼接就可以了。所以问题一定要朝着简单的解决方案走。

**条件包括：**

1. 过去一年(可以取bill表或者是record表的时间，月份都是对应的)
2. 月支出总和在2000以上
3. 账单应还金额在2500以上
4. 每张满足以上3个条件的月份总数有5个以上
   首先我们要想一个问题，就是card表底下会有多个月的账单bill数据，bill底下又会有多个记录record，我们要筛选的主体是card表，bill表和record表都是我们用来生成筛选条件的。
   其次，我们要知道条件里面是要做什么的，需不需要group by并进行一些操作？还是只是简单的筛选where？
   我们看了下，条件2有“月支出总和”，条件4有“计数”，所以我们知道了，对于条件2，我们需要对每张卡的每个月的支出流水进行聚合计算总和，对于条件4，最终我们需要对每张卡统计满足条件的月份数目，其他的条件没有聚合，可以通过where进行筛选。
   因此我们的策略是：

```sql
select card_no from       ---------------最终筛选出满足条件的卡片
(select * from card表 a
left join
(select * from
	(select * from bill表
	where to_date(bill_date)>add_months(current_date(),-5)---------条件1在近5个月的数据
	) b
left join
(select rid, month, sum(outcome) from record表
group by rid, month having sum(outcome)>2000----------条件2月支出在2000以上的月份
) c
on b.rid=c.rid and b.month=c.month
where b.balance>2500        ---------------条件3账单应还金额在2500以上
) b
on a.card_no=b.card_no
) a
group by card_no
having count(month)>5     ---------------条件4每张卡满足条件的月份数目在5个月以上的
```

![解析](https://upload-images.jianshu.io/upload_images/2338511-9ba9220ce81628f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面的代码其实我们可以看出一些东西：

1. 条件是层层嵌套的，像盗梦空间一样，我们要从最内部的条件开始写起，在这里就是条件1和条件2。
2. 每一层的结构是类似的，也就是join左边是要被筛选的数据主体，join右边是用来筛选的条件，然后我们select的东西包括了`需要选取的信息`，`要用来关联的关键字`，`要用来筛选的条件`。比如我们分析黄色区域内，在这个区域内是b模块与c模块进行关联，b模块写在join左边，因为b里面的内容是我们要筛选的数据(也就是bill数据)，然后进一步在黄色区域外又作为card表的筛选条件(所以你了解了为什么像盗梦空间了吧)。在黄色区域内，b模块(bill表)里面的select语句需要选出我们想要知道的关于账单的字段，还有要用来分别跟c以及黄色区域外的card表关联的字段，还有筛选该表的条件字段。
   所以最后应该写成了

```sql
select id, card_no, rid, month, balance from bill表
```

在这里id是可以选可不选的，card_no是要用来跟黄色区域外的card表 a关联的关键字，rid和month是跟c模块(record表)关联的字段，balance是筛选自身的条件(>2500)。
**总结一下，就是join可以用来筛选出满足条件的数据，join左边放筛选的主体，右边放生成筛选条件的表格，通过rid，id或者是no之类的编号一关联以后，就会只剩下满足条件的rid对应的数据。**





## hive平台与sublime联用的技巧

sublime是一个很好用的编辑器，也可以用于写SQL的代码。

我一般是在sublime上面写好代码以后再放到hive hue大数据平台上面运行，原因是sublime本身可以提供很多高效的操作。我们可以在sublime官网上面看一下有哪些用户所喜欢的高效操作。

![sublime的官网](https://upload-images.jianshu.io/upload_images/2338511-535791efc8b06700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 批量修改变量

![多行选择](https://upload-images.jianshu.io/upload_images/2338511-0960f63f88d11a22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们可以同时选择多行，进行一次性的修改，而不用相同的地方改10遍。
![sublime多行编辑.gif](https://upload-images.jianshu.io/upload_images/2338511-5b91d3b8bfcd1d21.gif?imageMogr2/auto-orient/strip)

只需要选中列以后，按cmd+shift+l就可以进入批量编辑模式，你也可以用cmd+d依次选中。你也可以用列模式编辑

![列模式编辑的操作](https://upload-images.jianshu.io/upload_images/2338511-c82155d16e9bd9b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![sublime多行编辑_列模式.gif](https://upload-images.jianshu.io/upload_images/2338511-cabcc7f6c2affebf.gif?imageMogr2/auto-orient/strip)

在这里Mac osx系统下使用ctrl+l的快捷键，就可以依次选中这一列的末尾或者是一列的内容。
如果是要退回一个选择，windows下是用`ctrl+u`，找了好久才找到这个快捷键，u应该是类似于vim里面的撤回操作。

如果你是要将某个区域的所有行合并为一行的话，可以用快捷键`ctrl+j`

### sublime的搜索功能

sublime的搜索功能很好用，总的搜索你可以用ctrl+p，键入相关的文件名，就可以打开你最近编辑的文件，可以用作快速跳转。进一步你还可以通过组合

- 行号跳转：`:20`跳转到文件的第20行
- 关键字跳转：`#keyword`
- symbol跳转：`@symbol`跳转到对应symbol

如果是要定位当前文件的某一行，则可以用ctrl+g的快捷键；ctrl+r定位相关的函数。

### 自定义修改快捷键

sublime还支持自定义修改快捷键，只不过修改快捷键是通过简单的代码进行的，对于具有代码恐惧症的业余选手可能会有些不友好，但是代码真的很简单。

首先点击preferences->key bindings，就会跳转到两个文件的页面。

![sublime设置快捷键](https://upload-images.jianshu.io/upload_images/2338511-43867dbb747cdf2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这两个文件，左边是sublime快捷键的默认设置，右边是可以自定义的快捷键代码，自定义的快捷键会覆盖默认的设置。比如我在这里就设置展开侧边栏的快捷键为`ctrl+shift+x`，具体的语法其实可以参考左边的写法，个人的理解是一个字典，有两个键，一个是keys，另一个是command，对应的值是用什么快捷键以及对应什么功能。

### 分屏解决方案

alt+num对应着把屏幕分成多少屏。

### 增加一行编辑

ctrl+enter可以在下面新增加一行重新进行编辑，ctrl+shift+enter可以在上面新增加一行，并且在那行进行编辑。

### 移动

windows的sublime的`ctrl+↑/↓`可以移动显示区域，而`ctrl+→/←`可以按照一个一个词来移动

### 自定义sublime的一些操作

你可以在preferences->settings下点击，打开配置文件。如下图所示：

![配置sublime](https://upload-images.jianshu.io/upload_images/2338511-c4bdf06bba180880.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个文件夹你可以粘贴以下的配置代码到右边的用户自定义的设置文件当中。

```python
"trim_trailing_white_space_on_save": true,#自动移除行尾多余空格
"ensure_newline_at_eof_on_save": true,#文件末尾自动保留一个空行
"font_face": "Microsoft YaHei Mono",#设置字体
"disable_tab_abbreviations": true,#禁用Emmet的tab功能
"translate_tabs_to_spaces": true,#tab对其转换为空格对齐
"tab_size": 2,
"draw_minimap_border": true,
"save_on_focus_lost": true,
"highlight_line": true,
"word_wrap": "true",
"fade_fold_buttons": false,
"bold_folder_labels": true,
"highlight_modified_tabs": true,
"default_line_ending": "unix",
"auto_find_in_selection": true#在选中范围内搜索
"""
作者：搁浅被注册了
链接：https://www.zhihu.com/question/24896283/answer/102462506
来源：知乎
"""
```

### 粘贴复制

有个很好用的快捷键`ctrl+shift+v`它可以将代码按照当前的位置的缩进进行粘贴，我们感受一下用普通的`ctrl+v`和`ctrl+shift+v`的区别吧。

![制表符对应复制粘贴.gif](https://upload-images.jianshu.io/upload_images/2338511-00a8b57d6962f2f5.gif?imageMogr2/auto-orient/strip)

## hive结合sublime的工作系统和流程

其实这次leader给的开发变量的任务让我拖得有点久，我总结了下，在工作流程以及工作界面，还有工作习惯方面都可以进行优化。

#### 首先是总的工作流程

> 一个任务从leader那里进来以后，首先并不着急答应，要询问清楚要达成的目标，避免草率地以为理解了，实际上做出来的并不是想要的结果，所以在一开始一定要问清楚细节，以及预估好时限。

#### 充分了解背景知识

> 然后要充分了解要实现的变量的背景知识，这些变量的含义，实现的方式，以及参考已有的模板代码，第一个上午甚至第一天先不着急敲代码，一定要在工作开始之前大概心里有数，不要被压力推着走。

#### 设定任务时限

> 了解了背景知识后，就要开始设定总体任务的完成时限，首先最重要给自己设定一个截止时间，虽然这个截止时间之前你不一定能完成，但是如果不设定的话就会无限期拖延下去，最好是每个小任务都设定一个截止时间。

#### 任务分解

> 然后是对应你思考的实现逻辑去进行大任务的往下分解，在这里我推荐奇妙清单或者是滴答清单，总体来说我觉得滴答清单更加好用和深入工作，之后会专门写一篇文章。

#### 任务进行中

> 在码代码之前在sublime text上面写下注释，说明这个部分的内容，要返回的东西，实现的目标，以及步骤等等，这样对照着去看，不容易迷失方向。然后把行动安排在奇妙清单或者是滴答清单里面，完成一项勾选一项，直到所有的行动完成就勾选掉这个项目，如果过程中有一些限制条件，一定要列出来，完成一项勾选一项，这样就不容易漏掉条件，造成返工浪费很多时间了。

#### 任务完成后

>  任务完成后及时反馈清晰可见的结果和代码，在完成本职工作以后再对整个工作流程进行重新复盘，审视是否还有不完善和浪费时间的地方。

#### 工作界面

我认为一个良好的工作界面对时间和精力的节约十分之重要，在时间和精力当中，精力比时间重要得太多太多了，虽然每天大部分时间坐在办公室里面，但实际上只有有限的时间是很高效率的，因此如果这些集中精力的时间都花在了来回切换界面上面，是十分让人恼火的。目前不太允许配备双显示屏，因此我发明了自己的hive+sublime text界面配置规则，根据所要用到的部分，我将sublime text整个界面分成了四个区域，如下显示，快捷键可以通过shift+alt+5实现，而shift+alt+2则实现二分屏，shift+alt+1实现单屏幕，shift+alt+8是上下分屏。

![四分屏](https://upload-images.jianshu.io/upload_images/2338511-a04f2da7f8c1b17e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主代码区是我写主要代码的区域，可以看做是考试时候要用的答题卡；草稿区是写好的代码调试的，比较零散，像是考试时候随心涂画的草稿纸；表格结构区是放要调用的表格的字段说明和结构信息的，参考代码区主要是放一些现有的代码。其实还可以考虑增加一个回收站的文件，专门回收可能出问题的代码，但是之后可能又要用上而不用重新敲的代码。

最终交给领导的代码应该是只有左上角的主代码区的代码，里面是能够完整执行下来不报错，删除了不必要的代码和注释的工工整整的代码。

# 总结
本篇文章主要从几个方面说了下自己这段时间在工作当中学习hive的一些心得，以及如何基于hive和sublime text构建一个比较完善和高效的工作流，其实我在整个过程中是十分低效的，因为一开始并没有找对路子，也是磕磕碰碰去尝试，踩了很多坑，决心要建立自己比较高效和完善的工作流系统，这样我就能在6点的时候下班了。
