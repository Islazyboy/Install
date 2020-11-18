#### 简介

Redis是现在最受欢迎的NoSQL数据库之一，Redis是一个使用ANSI C编写的开源、包含多种数据结构、支持网络、基于内存、可选持久性的键值对存储数据库，其具备如下特性：

- 基于内存运行，性能高效
- 支持分布式，理论上可以无限扩展
- key-value存储系统
- 开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API

相比于其他数据库类型，Redis具备的特点是：

- C/S通讯模型
- 单进程单线程模型
- 丰富的数据类型
- 操作具有原子性
- 持久化
- 高并发读写
- 支持lua脚本

哪些大厂在使用Redis？

- github
- twitter
- 微博
- Stack Overflow
- 阿里巴巴
- 百度
- 美团
- 搜狐

#### Redis的应用场景有哪些？

Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统。

[![De4peP.png](https://s3.ax1x.com/2020/11/18/De4peP.png)](https://imgchr.com/i/De4peP)

#### Redis的数据类型及主要特性

Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。

[![De4kWQ.png](https://s3.ax1x.com/2020/11/18/De4kWQ.png)](https://imgchr.com/i/De4kWQ)

##### String类型

它是一个二进制安全的字符串，意味着它不仅能够存储字符串、还能存储图片、视频等多种类型, 最大长度支持512M。

```
常用的字符串操作：

set key value #设置一个值
get key #返回key对应的value
keys * #查看所有value
del key #根据key删除数据
```

##### 哈希类型

该类型是由field和关联的value组成的map。其中，field和value都是字符串类型的。

```
hset key value #设置value值
hget key	#获取value
hdel key	#删除value
```

##### 列表类型

该类型是一个插入顺序排序的字符串元素集合, 基于双链表实现。

```
lrange key 0 -1 #从左往右输出value
lpush key 0,1,2 #从左边插入
rpush key 4,5,6 #从右边插入
```

##### 集合类型

Set类型是一种无顺序集合, 它和List类型最大的区别是：集合中的元素没有顺序, 且元素是唯一的。

Set类型主要应用于：在某些场景，如社交场景中，通过交集、并集和差集运算，通过Set类型可以非常方便地查找共同好友、共同关注和共同偏好等社交关系。

```
smembers #查看集合中所有的元素
srem #结合中指定的元素
```

##### 顺序集合类型：

ZSet是一种有序集合类型，每个元素都会关联一个double类型的分数权值，通过这个权值来为集合中的成员进行从小到大的排序。与Set类型一样，其底层也是通过哈希表实现的。

```
zadd myzset1 10 v1 20 v2 30 v3 #添加元素
 zrange myzset1 0 -1 withscores #查看元素
```



#### 除了Redis，还有什么NoSQL型数据库

市面上类似于Redis，同样是NoSQL型的数据库有很多，如下图所示，除了Redis，还有MemCache、Cassadra和Mongo。下面，我们就分别对这几个数据库做一下简要的介绍：

[![De5e9e.png](https://s3.ax1x.com/2020/11/18/De5e9e.png)](https://imgchr.com/i/De5e9e)

 

 

**Memcache**：这是一个和Redis非常相似的数据库，但是它的数据类型没有Redis丰富。Memcache由LiveJournal的Brad Fitzpatrick开发，作为一套分布式的高速缓存系统，被许多网站使用以提升网站的访问速度，对于一些大型的、需要频繁访问数据库的网站访问速度的提升效果十分显著。

**Apache Cassandra**：（社区内一般简称为C*）这是一套开源分布式NoSQL数据库系统。它最初由Facebook开发，用于储存收件箱等简单格式数据，集Google BigTable的数据模型与Amazon Dynamo的完全分布式架构于一身。Facebook于2008将 Cassandra 开源，由于其良好的可扩展性和性能，被 Apple、Comcast、Instagram、Spotify、eBay、Rackspace、Netflix等知名网站所采用，成为了一种流行的分布式结构化数据存储方案。

**MongoDB**：是一个基于分布式文件存储、面向文档的NoSQL数据库，由C++编写，旨在为WEB应用提供可扩展的高性能数据存储解决方案。MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系型数据库的，它支持的数据结构非常松散，是一种类似json的BSON格式。