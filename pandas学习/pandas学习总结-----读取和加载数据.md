# 读取相应的文件类型
## 常用参数
![IMG_0797](https://tva1.sinaimg.cn/large/008eGmZEgy1gmiurffbdqj30u016y7ic.jpg)
图上标黄的这样几个参数可以关注下。
### 指定列名
`names`可以在加载文件的时候，传入列名，结合`header=None`
比如
```python
pd.read_csv('examples/ex2.csv')
```
![-w251](https://tva1.sinaimg.cn/large/008eGmZEgy1gmiurgdwl1j30dy05ujrk.jpg)
```python
names = ['a', 'b', 'c', 'd', 'message']
pd.read_csv('examples/ex2.csv', names=names, index_col='message')
```
![-w260](https://tva1.sinaimg.cn/large/008eGmZEgy1gmiurehsekj30eg094glx.jpg)
在这里就传入了列名的列表names，并且用`index_col`指定了`message`这一列作为索引。


### 不想要一次过加载那么多数据的时候

当你不想要一次性读取全部数据的时候，请想起`nrows`, `skiprows`, `chunksize`这三个参数。
`nrows`可以用来选择性读多少行，当你只想要查看数据的总体概况而不想全部读入的时候，可以选定读几行。
`skiprows`可以跳过不读的行数。
`chunksize`适用于迭代式地读取操作，产生的是迭代器`TextParser`。
```python
for gm_chunk in pd.read_csv(csv_url,chunksize=500):
    print(gm_chunk.shape)
(500, 6)
(500, 6)
(500, 6)
(204, 6)
```
### 对日期进行处理
`parse_date`和`date_parser`可以对日期文本进行解析转化成日期格式的字段。

# 操作数据库的两种方式
## 使用sqlite3进行操作
```python
import sqlite3
query = """
CREATE TABLE test
(a VARCHAR(20), b VARCHAR(20),
 c REAL,        d INTEGER
);"""
# 连接数据库
con = sqlite3.connect('mydata.sqlite')
# 对返回的连接对象进行execute和commit()
con.execute(query)
con.commit()
```

## 使用sqlalchemy方式
这种方式会简单很多，直接先创建一个查询引擎，然后用`read_sql`方法使用创建的引擎`engine`去执行语句。
```python
import sqlalchemy as sqla
db = sqla.create_engine('sqlite:///mydata.sqlite')
pd.read_sql('select * from test', db)
```
以上主要是pandas操作常用的csv文件和数据库的方式，其他的pandas还可以读取web api，json文件等等，这些就先不展开讲了。