`Networkx`是用来进行图数据创建，存储，分析和计算的一个python工具包，它可以存储节点和边数据，使用一些图网络的算法进行发现一些我们想要对这个网络的一些洞察信息。

# 使用方法

### 创建一个空的图

```python
import networkx as nx
G=nx.Graph()
```

### 添加节点和边

添加节点和边的方式是类似的，唯一的区别就是节点是只有一个值，而边则包含两个节点，包括目标节点和源节点

```python
G.add_node(1) #添加单个节点
G.add_nodes_from([1,2]) #添加多个节点
# 你还可以添加节点和节点的属性，以tuple形式给出(node, node_attributes_dict)
```

添加节点和边有两种方式，一种是添加单个节点和单个边，另外一种是批量添加，区别就是批量添加的api名称后面加个`from`里面传入的是一个列表。



### 查看节点



# 初始化内置的图

`networkx`本身自带一些成型的图，比如`karate_club_graph`和`barbell_graph`

# 社区发现

`networkx`本身自带的`algorithms`模块可以帮助我们实现很多算法，其中一个就是社区发现

,社区发现的模块是`community`，下面有`girvan_newman`算法

```python
from networkx.algorithms import community
G=nt.karate_club_graph()
comp=community.girvan_newman(G)


```

