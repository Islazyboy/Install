## ES核心概念

### 关系数据库 VS ElasticSearch

关系数据库： 数据库 —> 表 —> 行 —>列(Columns)

ElasticSearch：索引(index)—>文档(Docments)—>字段(Fields)—>类型(同一类)

| 关系数据库       | ElasticSearch |
| :--------------- | :------------ |
| 数据库(database) | 索引(indices) |
| 表(tables)       | types         |
| 行(rows)         | documents     |
| 字段(columns)    | fifields      |

elasticsearch(集群)中可以包含多个索引(数据库)，每个索引中可以包含多个类型(表)，每个类型下又包 含多 个文档(行)，每个文档中又包含多个字段(列)。

**物理设计：**

elasticsearch 在后台把每个索引划分成多个分片，每分分片可以在集群中的不同服务器间迁移

**逻辑设计：**

一个索引类型中，包含多个文档，比如说文档1，文档2。 当我们索引一篇文档时，可以通过这样的一各 顺序找到 它: 索引 ? 类型 ? 文档ID ，通过这个组合我们就能索引到某个具体的文档。 注意:ID不必是整 数，实际上它是个字符串。

### 文档

因为elasticsearch是面向文档的，那么意味着索引和搜索数据的最小单位是文档，在elasticsearch中，文档有以下几个重要的属性：

- 自我包含、一篇文档同时包含字段和对应的值，key:value
- 可以是层次的，一个文档中包含自文档（就是一个json对象，fastjson进行自动转换）
- 灵活的结构，文档不依赖于预先定义的模式
- 

尽管我们可以随意的新增或者忽略某个字段，但是，每个字段的类型非常重要，比如一个年龄字段类 型，可以是字符 串也可以是整形。因为elasticsearch会保存字段和类型之间的映射及其他的设置。这种 映射具体到每个映射的每种类型，这也是为什么在elasticsearch中，类型有时候也称为映射类型。

### 类型

**类型是文档的逻辑容器**，就像关系型数据库一样，表格是行的容器。 **类型中对于字段的定义称为映射**， 比如 name 映射为字符串类型。 我们说文档是无模式的，它们不需要拥有映射中所定义的所有字段， 比如新增一个字段，那么elasticsearch是怎么做的呢?elasticsearch会自动的将新字段加入映射，但是这个字段的不确定它是什么类型，elasticsearch就开始猜，如果这个值是18，那么elasticsearch会认为它是整形。 但是elasticsearch也可能猜不对，所以最安全的方式就是提前定义好所需要的映射，这点跟关系型数据库殊途同归了，先定义好字段，然后再使用。

### 索引

索引是映射类型的容器，elasticsearch中的索引是一个非常大的文档集合。索引存储了映射类型的字段和其他设置，然后他们被存储到了各个分片上。

物理设计：节点和分片如何工作

- 一个集群至少有一个节点，而一个节点就是一个elasricsearch进程，节点可以有多个索引默认的，如果 你创建索引，那么索引将会有个5个分片 ( primary shard ,又称主分片 ) 构成的，每一个主分片会有一个 副本 ( replica shard ,又称复制分片 )

主分片和对应的复制分片都不会再同一节点内，这样如果一个节点挂掉了，数据也不至于丢失。实际上，一个分片是一个Lucene索引，一个包含倒排索引的文件目录，倒排索引的结构使得elasticsearch在不扫描全部文档的情况下，就能告诉你哪些文档包含特定的关键字。那什么是倒排索引？

### 倒排索引

elasticsearch使用的是一种称为倒排索引的结构，采用Lucene倒排索作为底层。这种结构适用于快速的 全文搜索，一个索引由文档中所有不重复的列表构成，对于每一个词，都有一个包含它的文档列表。

例如，现在有两个文档， 每个文档包含如下内容：

```
Study every day, good good up to forever # 文档1包含的内容
To forever, study every day, good good up # 文档2包含的内容
```

为了创建倒排索引，我们首先要将每个文档拆分成独立的词(或称为词条或者tokens)，然后创建一个包含所有不重 复的词条的排序列表，然后列出每个词条出现在哪个文档 :

| term    | doc_1 | doc_2 |
| :------ | :---- | :---- |
| Study   | √     | ×     |
| To      | ×     | ×     |
| every   | √     | √     |
| forever | √     | √     |
| day     | √     | √     |
| study   | ×     | √     |
| good    | √     | √     |
| every   | √     | √     |
| to      | √     | ×     |
| up      | √     | √     |

只要我们尝试搜索to forever，只需要查看包含每个词条的文档

| term    | doc_1 | doc_2 |
| :------ | :---- | :---- |
| to      | √     | ×     |
| forever | √     | √     |
| total   | 2     | 1     |

### elasticsearch的索引和Lucene的索引对比

在elasticsearch中， 索引这个词被频繁使用，这就是术语的使用。在elasticsearch中，索引被分为多个分片，每份分片是一个Lucene的索引。所以一个elasticsearch索引是由多个Lucene索引组成的。别问为什么，谁让elasticsearch使用Lucene作为底层呢! 如无特指，说起索引都是指elasticsearch的索引。

接下来的一切操作都在kibana中DevTools下的Console里完成。

## ES基础操作

### IK分词器

IK分词器概述：分词就是把一段中文或者句子划分成一个个的关键字，我们在搜索的时候会把自己的信息进行分词，会把数据库中或者索引库中的数据进行分词，然后进行一个匹配操作。比如“我爱编程”，就会被分为“我“，”爱“，”编“，”程“，显然是不合理的，所以我们要安装ik中文分词器来解决这个问题。

IK提供了两个分词算法：ik_smart 和 ik_max_word，其中 ik_smart 为最少切分，ik_max_word为最细粒度划分。

### IK分词器安装

官网下载 https://github.com/medcl/elasticsearch-analysis-ik/

当然也可以点我的奇妙链接口令0u3g ,更快速下载

下载后解压，并将其解压到ElasticSearch根目录下的 plugins 目录中。

![image-20201020192223942](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/a1a9f6b30337e0fb90e4c00e4ce43258.png)

此时重启ElasticSearch后，发现多加载了一个analysis-ik的服务信息，

![image-20201020192502236](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/5982a2e3f15f7eaf98d578d74963b107.png)

服务启动后在输入elasticsearch-plugin list 命令，确认 ik 插件安装成功。

![image-20201020192639952](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/a2e240c24f9f83923b4219f629164780.png)

然后我们在kibana中测试ik分词器

### 测试

运行kibana后，点击左侧

ik_max_word : 细粒度分词，会穷尽一个语句中所有分词可能



```
Get _analyze
{<!-- -->
  "analyzer": "ik_max_word",
  "text": "程序员不光头"
}
```

![image-20201020193819398](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/7be8af6f5e16bb2014af2904232de339.png)

ik_smart : 粗粒度分词，优先匹配最长词，只有1个词

```
Get _analyze
{<!-- -->
  "analyzer": "ik_smart",
  "text": "程序员不光头"
}
```

![image-20201020193848809](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/1886a22fa72331d21c3f02ab65f395bc.png)

可以发现拆分细粒度的程度不一样

如果我们自定义一个词条，冯和半仙分开了，那如果想让词库知道，冯半仙是一整个不可分开的词了。

### IK分词器增加自己的配置

在elasticsearch/plugins/ik/config目录下创建一个***.dic，随便取什么名字

把 “冯半仙” 存入保存

修改IKAnalyzer.cfg.xml（在ik/config目录下）

修改完配置文件后保存，重启elasticsearch，再次测试

```
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd"> <properties>   <comment>IK Analyzer 扩展配置</comment>   <!--用户可以在这里配置自己的扩展字典 -->   <entry key="ext_dict">***.dic</entry>    <!--用户可以在这里配置自己的扩展停止词字典-->   <entry key="ext_stopwords"></entry>   <!--用户可以在这里配置远程扩展字典 -->   <!-- <entry key="remote_ext_dict">words_location</entry> -->   <!--用户可以在这里配置远程扩展停止词字典-->   <!-- <entry key="remote_ext_stopwords">words_location</entry> --> </properties>
```

![image-20201020194918676](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/19991bbabd85e52cd2aca9ae18939300.png)

启动的时候发现，我们的自定义词典已经加载进来了，再去kibana测试

![image-20201020195059616](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/03eac405c1ddf10709ecbe2c6e36dd6f.png)

发现冯半仙变成一个词啦

### REST风格

rest是一种软件架构风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

基于rest命令说明

| method | url地址                                         | 描述                   |
| :----- | :---------------------------------------------- | :--------------------- |
| PUT    | localhost:9200/索引名称/类型名称/文档id         | 创建文档（指定文档id） |
| POST   | localhost:9200/索引名称/类型名称                | 创建文档（随机文档id） |
| POST   | localhost:9200/索引名称/类型名称/文档id/_update | 修改文档               |
| DELETE | localhost:9200/索引名称/类型名称/文档id         | 删除文档               |
| GET    | localhost:9200/索引名称/类型名称/文档id         | 查询文档通过文档id     |
| POST   | localhost:9200/索引名称/类型名称/_search        | 查询所有数据           |

### 测试rest

进入kibana的Console

输入



运行后返

```
PUT /索引名/文档类型/文档id
PUT /test1/type1/1
{<!-- -->
    "name":"冯半仙", // 属性
    "age":22 // 属性
}
```

回

```
#! Deprecation: [types removal] Specifying types in document index requests is deprecated, use the typeless endpoints instead (/{<!-- -->index}/_doc/{<!-- -->id}, /{<!-- -->index}/_doc, or /{<!-- -->index}/_create/{<!-- -->id}).
#警告：不支持在文档索引请求中指定类型
# 而是使用无类型的端点(/{<!-- -->index}/_doc/{<!-- -->id}， /{<!-- -->index}/_doc，或
#{<!-- -->index}/_create/{<!-- -->id})。

{<!-- -->
  "_index" : "test1",   //索引
  "_type" : "type1",    //类型
  "_id" : "1",          //id
  "_version" : 1,       //版本
  "result" : "created", //操作信息
  "_shards" : {<!-- -->         //分片信息
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

在elasticsearch-head里发现我们类型是type1

![image-20201020200254306](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/1286ba5c94daa7d17499386b80e59d73.png)

#### 指定类型

- 字符串类型

text 、 keyword

- 数值类型

long, integer, short, byte, double, float, half_float, scaled_float

- 日期类型

date

- 布尔值类型

boolean

- 二进制类型

binary

### 指定字段类型

![image-20201020202012582](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/680ca0f661f8d18085ee36f13638d732.png)

我们GET一下test2看看

```
PUT /test2
{<!-- -->
  "mappings": {<!-- -->
    "properties": {<!-- -->
      "name":{<!-- -->
        "type": "text"
      },
      "age":{<!-- -->
        "type": "long"
      }
    }
  }
}
```



对比

```
{<!-- -->
  "test2" : {<!-- -->
    "aliases" : {<!-- --> },
    "mappings" : {<!-- -->
      "properties" : {<!-- -->
        "age" : {<!-- -->
          "type" : "long"
        },
        "name" : {<!-- -->
          "type" : "text"
        }
      }
    },
    "settings" : {<!-- -->
      "index" : {<!-- -->
        "creation_date" : "1603196400652",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "soFUINV1TSyBvt2LVZXdhQ",
        "version" : {<!-- -->
          "created" : "7080099"
        },
        "provided_name" : "test2"
      }
    }
  }
}
```

关系型数据库 ：

PUT test1/type1/1 ： 索引test1相当于关系型数据库的库，类型type1就相当于表 ，1 代表数据中的主 键 id

### 查看索引

我们上面用get可以获得信息，那看索引的信息呢？也可以用get试一下

```
GET _cat/indices?v
```

![image-20201020202440890](https://i2.wp.com/img-blog.csdnimg.cn/img_convert/652649d63491ed34c704d7dcb3d2a220.png)

返回的是我们所有索引的状态、健康情况 分片，数据存储大小等

我们也可以用delete命令来删除索引

```
DELETE /test2
```

返回

```
{<!-- -->
  "acknowledged" : true
}
```

