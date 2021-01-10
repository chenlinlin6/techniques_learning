准备数据
```sql
CREATE TABLE lxy (cookieid INT, create_time STRING, pv INT) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
LOAD DATA INPATH '/user/chenlinlin2156233/lxy.csv';
SELECT * FROM lxy;
```
查看结果
![返回表格](https://upload-images.jianshu.io/upload_images/2338511-b9455df36e3aa649.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# SUM(), MIN(),MAX(),AVG()等聚合函数
对一定窗口期内的数据进行聚合
```sql
SELECT *, 
SUM(a.pv) OVER (PARTITION BY cookieid ORDER BY create_time ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv1,
SUM(a.pv) OVER (PARTITION BY cookieid ORDER BY create_time ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING) AS pv2
FROM lxy AS a;
```
在这里根据cookieid进行分组，然后按照`create_time`进行分组，选择不同的窗口进行一定函数的聚合运算。
基本的语法是`ROWS BETWEEN 一个时间点 AND 一个时间点`
时间点分别可以是以当前行作为参考系，前面几行`n PRECEDING`或者是后面几行`n FOLLOWING`，也可以是当前行`CURRENT ROW`。总之可以想象有一个滑动窗口，我们可以规定一个滑动窗口的中心位置和大小，然后每次画过一个步长，计算一次窗口内的值。
![求解窗口期内的数据的总和](https://upload-images.jianshu.io/upload_images/2338511-95ec220dbe57a495.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 新增加序号列NTILE, ROW_NUMBER(), RANK(), DENSE_RANK()
我们先来试试看这几个函数的实际返回结果。
![数据源](https://upload-images.jianshu.io/upload_images/2338511-396f04e057fcdc74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```sql
SELECT *, 
NTILE(3) OVER (PARTITION BY cookid2 ORDER BY pv) AS n1,
ROW_NUMBER() OVER (PARTITION BY cookid2 ORDER BY pv) AS n2,
RANK() OVER (PARTITION BY cookid2 ORDER BY pv) AS n3,
DENSE_RANK() OVER (PARTITION BY cookid2 ORDER BY pv) AS n4
FROM lxy3;
```
![返回结果](https://upload-images.jianshu.io/upload_images/2338511-ded389898a3537a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们可以看到，对于NTILE函数，传入的参数n是指要切分成多少份，返回对应的序号，ROW_NUMBER()则是生成一列连续的序号，RANK()与ROW_NUMBER()类似，只是对于数值相同的这一项会同时为相同的序号，下一个序号跳过，比如倒数第二列当中有出现4，4，6没有5；而DENSE_RANK()则相反，会紧跟着下一个是紧接着的序号，比如4，4，5。
# LAG, LEAD, FIRST_VALUE, LAST_VALUE
这几个函数可以通过字面意思记得，LAG是迟滞的意思，也就是对某一列进行往后错行；LEAD是LAG的反义词，也就是对某一列进行提前几行；FIRST_VALUE是对该列到目前为止的首个值，而LAST_VALUE是到目前行为止的最后一个值。
仍旧是这张表
![lx3](https://upload-images.jianshu.io/upload_images/2338511-548390b8f9e27440.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```sql
SELECT *,
LAG(pv, 2) OVER(PARTITION BY cookid2 ORDER BY log_date) AS lag1,
LEAD(pv, 2, 0) OVER(PARTITION BY cookid2 ORDER BY log_date) AS lead1,
FIRST_VALUE() OVER(PARTITION BY cookid2 ORDER BY log_date) AS first_pv,
FIRST_VALUE() OVER(PARTITION BY cookid2 ORDER BY log_date) AS last_pv,
LAST_VALUE() OVER(PARTITION BY cookid2 ORDER BY log_date) AS current_last_pv
FROM lxy3;
```
![返回结果](https://upload-images.jianshu.io/upload_images/2338511-e5d3f2fd54e9230a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
LAG和LEAD里面都是传入三个参数，分别是排序的列名，滞后/往前的行数，以及默认填充值。因为我们在这里的LEAD()里面设置默认填充值为0，所以对于cookid后面两行缺失值填充为0。
如果我们要返回每个分组下排序后的最后一个数，可以对该组进行DESC的操作，注意ORDER BY对返回的结果很有影响。
```sql
SELECT *,
FIRST_VALUE() OVER(PARTITION BY cookid2 ORDER BY pv DESC) AS first_pv
FROM lxy3; 
```
# GROUPING SET, CUBE, ROLL UP
我们先准备一张表格
```sql
CREATE EXTERNAL TABLE lxw1234 (
month STRING,
day STRING, 
cookieid STRING 
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
LOCATION '/user/chenlinlin2156233/lxy2/';
```
![创建表格](https://upload-images.jianshu.io/upload_images/2338511-9e92fa12847301f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```sql
SELECT * FROM lxw1234;
```
![返回结果](https://upload-images.jianshu.io/upload_images/2338511-32a3877de8932163.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从上面看到我们已经成功导入了一张外部表。
GROUPING SET(key1, key2)相当于是对不同字段进行group操作以后，再进行union all的操作。
```sql
SELECT month,
day,
count(DISTINCT cookieid) AS count_id,
GROUPING__ID
FROM lxw1234
GROUP BY month, day
GROUPING SETS(month, day)
ORDER BY GROUPING__ID;
```
![返回结果](https://upload-images.jianshu.io/upload_images/2338511-1bfa302271a287c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这里注意，
1. GROUPING_ID是自动生成的，是进行了GROUPING_SET()的操作之后。
2. 下划线有两个
3. 需要先做GROUP BY操作再传入GROUPING SETS
等价于先group再union all的做法
```sql
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM lxw1234 GROUP BY month 
UNION ALL 
SELECT NULL,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM lxw1234 GROUP BY day
UNION ALL 
SELECT month,day,COUNT(DISTINCT cookieid) AS uv,3 AS GROUPING__ID FROM lxw1234 GROUP BY month,day
```
![等价效果实现](https://upload-images.jianshu.io/upload_images/2338511-f95fc415807614fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

CUBE就是比以上的GROUPING SETS多了一个两列的整合，也就是笛卡尔乘积。
```sql
SELECT month,
day,
count(DISTINCT cookieid) AS count_id,
GROUPING__ID
FROM lxw1234
GROUP BY month, day
WITH CUBE
ORDER BY GROUPING__ID;
```
![返回结果](https://upload-images.jianshu.io/upload_images/2338511-98b4407ad1ba3ee8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


假如我们把上面的代码里面的`CUBE`改成了`ROLL UP`，我们看下会返回什么结果。
```sql
SELECT month,
day,
count(DISTINCT cookieid) AS count_id,
GROUPING__ID
FROM lxw1234
GROUP BY month, day
WITH ROLLUP
ORDER BY GROUPING__ID;
```
![rollup返回的结果](https://upload-images.jianshu.io/upload_images/2338511-8118b971080e7184.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，这个时候就不会返回以右边为关键字的聚合结果，只是返回左边的键以及笛卡尔乘积的结果。
我们如果换一下聚合的关键字month和day的顺序呢？
```sql
SELECT month,
day,
count(DISTINCT cookieid) AS count_id,
GROUPING__ID
FROM lxw1234
GROUP BY day, month
WITH ROLLUP
ORDER BY GROUPING__ID;
```
![交换关键字以后的返回结果](https://upload-images.jianshu.io/upload_images/2338511-4a4f0ef85a3e4661.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从上面结果可以看到，关键字的顺序对rollup的结果也是很有影响的。  
以上就是所学习hive窗口函数的总结。  
# 参考资源
以上总结主要参考[该博客](http://www.aboutyun.com/thread-12849-1-1.html)。
