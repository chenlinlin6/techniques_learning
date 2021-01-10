

# SELECT用来选取

# WHERE语句用来筛选

## 常用操作符号

![操作符号](https://upload-images.jianshu.io/upload_images/2338511-d19233533589330e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


对于文本字符串的匹配

![文本字符串.png](https://upload-images.jianshu.io/upload_images/2338511-fbe667760d57a568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



如果是精准匹配则用=和!=，否则就用LIKE和NOT LIKE进行模糊匹配，不分大小写，%用于匹配多个字符串，_匹配单个字符串。

## DISTINCT筛选独一无二的值

DISTINCT加到SELECT之后，可以去除掉重复值。

## ORDER BY可以排序

ASC/DESC升序和降序

## LIMIT+num限制显示的数目

## OFFSET设置开始显示的序号

与LIMIT搭配，决定开始显示的序号



# 简单的SELECT查询
![简单的select查询](https://upload-images.jianshu.io/upload_images/2338511-f4f7454c2f0d5534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




# 多个表格的query

## join的方式来链接两个表格

主键是指在整个数据库当中能够独一无二地辨别出实体的那一列，比如index。

Join方法对具有相同键的两张表格应用，键用on链接。在join完之后应用我们之前提到的方法。

# Nulls的处理

![null的处理](https://upload-images.jianshu.io/upload_images/2338511-20112419e41d66ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以用isnull和is not null来筛选出某一列的值是否为空值。空值对统计结果，部分函数的执行有影响。

# 表达式的索引方法

![表达式的索引方法](https://upload-images.jianshu.io/upload_images/2338511-88fdf0adcaa41e5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以用表达式对符合条件的值进行重新计算。用AS进行重新命名。

# 用聚合函数进行查询

![聚合函数查询](https://upload-images.jianshu.io/upload_images/2338511-42f7d1113f883582.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


聚合函数能够允许你对一组数据进行信息总结，比如最大值max，众数mode

![常用的聚合函数](https://upload-images.jianshu.io/upload_images/2338511-ca014cca58801c4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前面说的是如何筛选，后面说的是如何对筛选后的数据进行处理后返回。

![having对group进行筛选](https://upload-images.jianshu.io/upload_images/2338511-554794f0c2ba1fd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


where方法对group by前的行进行筛选，而having对group by后的数据进行筛选

# SQL的执行顺序

![sql的执行顺序](https://upload-images.jianshu.io/upload_images/2338511-2b744340b2a69df2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

SQL的执行顺序总体上是这样的：找到数据库当中我们所需要的数据(FROM)→筛选(filter)→以我们写的形式显示(SELECT)

具体的执行顺序是：FROM开始寻找从哪张表格当中开始寻找需要的信息，这个时候发现了JOIN，JOIN ON语法使得此表格与另外一张表格构成临时的表格，我们就从这张临时的表格当中查找数据，通过WHERE将不满足条件的行全部筛选掉，然后根据对应的column进行GROUP BY命令，HAVING筛选掉不满足条件的数据，然后SELECT DISTINCT取不重复的数据，接下来根据ORDER BY的命令来按照条件来排列，最后LIMIT显示的内容限制在前几条。

# SQL结构
![插入行](https://upload-images.jianshu.io/upload_images/2338511-1e4e256d24985537.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
SQL schema是指**数据库的结构**以及**数据类型**

数据库的结构是指行和列，每一行都是一个实体，每一列都是一个属性，每一列都有特定数据类型，如整数，字符串等等。

插入数据的语法

```sql
INSERT INTO table_name VALUES (value1, value2, value3, value4……)
```

value是跟列一一对应的，如果插入的数据列数少于表格的列数，需要指定列名

```sql
INSERT INTO table_name (col1,col2,col3,col4) VALUES (value1, value2, value3, value4……)
```

# 更新表格
![更新表格的方法update](https://upload-images.jianshu.io/upload_images/2338511-547bfa6fa080ccea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


update方法可以像插入行一样更新满足where限制条件的行。

要注明表格的名字，列，以及数据。数据和列以键值的方式给出。

# 删除行内容

![delete语句](https://upload-images.jianshu.io/upload_images/2338511-531c17cd691cd11d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以用DELETE FROM语句搭配WHERE限制条件来删除特定的行。

```sql
DELETE FROM table_name WHERE constraints
```

# 创建一个表格

![创建一个表格](https://upload-images.jianshu.io/upload_images/2338511-b7ce0740f9fd37bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建一个表格要指定几样东西吗，一个是表格名字，列名字，数值类型，默认值

## 列的类型

![列的类型](https://upload-images.jianshu.io/upload_images/2338511-9a8d1250c10a0147.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 表格的额外限制

![表格的额外限制](https://upload-images.jianshu.io/upload_images/2338511-99d09a08a0acce12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


除了列的类型外，还可以对列进行额外的限制，比如PRIMARY KEY(主键)，AUTOINCREMENT(自动增加)适合用于自动增加的序列，NOT NULL(非空值)，FOREIGN KEY(外键)。

外键和主键的区别是，主键的值在一张表格当中是独一无二的，是用于标示这一行。外键是另外一张表格需要与主表格进行连接的时候，总能在主表格当中找到有效的值。比如一张表格是记录员工的id号码的，这个时候id是主键，是唯一不重复的。而另外一个是发奖金的表格，一个员工可能会有多个奖励事项，这个表格当中的id是不唯一的，但是总能在员工id的表格当中找到有效值。

```sql
CREATE TABLE mytable (col1 col1_type, col2 col2_type, col3 col3_type)
```

# 进阶SQL命令

## timestamp

![extract function](https://upload-images.jianshu.io/upload_images/2338511-ea7cbbb564747ace.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
extract function可以从日期格式的数据当中抽取出我们需要的信息

![document](https://upload-images.jianshu.io/upload_images/2338511-cfd0d7f3aa159453.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```sql
extract(month from date_column)
```

# 字符串的运算符

![字符串的运算符](https://upload-images.jianshu.io/upload_images/2338511-9fcb549704685ebc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


||用来连接多个字符串和非字符串

# Subquery

subquery就是在query里面再进行查询

![subquery](https://upload-images.jianshu.io/upload_images/2338511-fdd75976e5aa06da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


只需要把subquery部分用括号括起来即可

![subquery2](https://upload-images.jianshu.io/upload_images/2338511-d53113736aee38a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```sql
SELECT film_id, title, rental_rate FROM film WHERE rental_rate > (SELECT AVG(rental_rate) FROM film);
```

# SELF JOIN

self join是指table自己与自己连接，为了实现self join，需要用alias方法

![self join](https://upload-images.jianshu.io/upload_images/2338511-0fad026ac2818c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![self join2](https://upload-images.jianshu.io/upload_images/2338511-7e853af1ecadc9c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们不想用硬编码的时候会考虑self join

![self join3](https://upload-images.jianshu.io/upload_images/2338511-40127a87a6a71e78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

比如在以上这个例子当中，其实可以直接限定e1.employee_location='New York'即可，但是在某些情况下不好用硬编码，因此我们找出名字joe对应的location，它为'New York'，那么就能找出location为'New York'的employee_name了。

![self join4](https://upload-images.jianshu.io/upload_images/2338511-641a5d1c9183acfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![self join5](https://upload-images.jianshu.io/upload_images/2338511-b4068c75000591f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建了两张相同的表格，只是名字不一样

![self join6](https://upload-images.jianshu.io/upload_images/2338511-f2993d19757777b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




# 创建VIEWS的方法

CREATE VIEW (你的query)
