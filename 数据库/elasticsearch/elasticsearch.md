# Elasticsearch

一款强大的搜索引擎 

Elasticsearch是一个基于Lucene的搜索服务器。它提供了一个分布式多功能用户能力的全文搜索引擎 , 基于Restful Web接口, Elasticsearch是用java开发的 , 并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于分布式 , 云计算 ,能够达到实时搜索 , 稳定 , 可靠 ,快递 ,安装使用方便



### 使用前和使用后

**原始的** 

查询或者模糊查询的执行是获取到`关键字`然后去每一个对象中去匹配 和当前关键字是否相等 如果相等则匹配下一个字符是否相等 , 这样在数据很多的情况下 , `查询时及其浪费性能`几乎没有任何一家大型公司是使用普通的查询

![1567602713021](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567602713021.png)

**使用后**

Elasticsearch通过`分词器`这个东西将文档的内容分词提取出来,整理成一个特殊的文档 , 在做查询操作的时候会建立<font style="color:red">`倒排索引`</font>来辅助查询 , 当用户查询的时候会匹配文档内容中的词 , 首先省去了每个字母每个字母循环匹配的麻烦, 然后如果查到传递的参数符合分词后的单词 , 则找当前单词的文档 id 循环文档 id 然后将查询结果返回给客户端 , 这样将查询的性能提升到了极致

![1567603066394](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603066394.png)



### 如何使用Elasticsearch

#### 介绍

<font style="color:red">Elasticsearch</font>(搜索引擎)    <font style="color:red">Logstash</font>(日志处理)    <font style="color:red">kIbana</font>(搜索引擎可视化处理)

>  这三个是企业中最常用的三种新型的产品 , 俗称搜索三剑客
>
> 这里我们使用 `Elasticsearch`  `kIbana` 日志处理先不研究

![1567603333349](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603333349.png)



<font style="color:red">由于Es的版本更新迭代的浮动很大 所以这里我们使用 6.x的版本</font>



#### 安装

1. 解压Es的安装包 然后启动`elasticsearch.bat`文件

   直到页面出现

   ![1567603654964](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603654964.png)

   代表已经安装成功

2. 将`elasticsearch`添加到后台管理服务

   ![1567603725436](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603725436.png)

   ![1567603734145](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603734145.png)

3. 安装head插件

   ![1567603774908](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603774908.png)

4. <font style="color:red">安装**kibana**</font>

   解压Kibana安装包 , 运行bin/kibana.bat，看到启动成功的端口号即可以使用浏览器来使用了

   ![1567603846731](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603846731.png)



### elasticsearch的数据存储

#### 数据存储图

![1567603909992](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567603909992.png)

> 其中 在创建的时候 索引就是数据库 , 映射是用来做规定用的 , 每一个索引里面只能有一个映射 并且在创建索引的同时也创建了映射 , <font style="color:red">ES6开始索引和类名保持一致</font>



#### <font style="color:red">主要的组件</font>

- 索引
  - ES将数据存储于一个或多个索引中，索引是具有类似特性的文档的集合。类比传统的关系型数据库领域来说， 索引相当于SQL中的一个数据库，或者一个数据存储方案(schema)。索引由其名称(必须为全小写字符)进行标识，并通过引用此名称完成文档的创建、搜索、更新及删除操作。一个ES集群中可以按需创建任意数目的索引。
- 类型
  - 类型是索引内部的逻辑分区(category/partition)，然而其意义完全取决于用户需求。因此，一个索引内部可定义一个或多个类型(type)。一般来说，类型就是为那些拥有相同的域的文档做的预定义。例如，在索引中，可以 定义一个用于存储用户数据的类型，一个存储日志数据的类型，以及一个存储评论数据的类型。类比传统的关系型数据库领域来说，类型相当于表。
- 映射
  - Mapping,就是对索引库中索引的字段名称及其数据类型进行定义，类似于mysql中的表结构信息。不过es的mapping比数据库灵活很多，它可以动态识别字段。一般不需要指定mapping都可以，因为es会自动根据数据格式识别它的类型，如果你需要对某些字段添加特殊属性（如：定义使用其它分词器、是否分词、是否存储等），就必须手动添加mapping。<font style="color:red">需要注意的是映射是不可修改的，一旦确定就不允许改动，在使用自动识别功能时，会以第一个存入的文档为参考来建立映射，</font>后面存入的文档也必须符合该映射才能存入
- 文档
  - 文档是Lucene索引和搜索的原子单位，它是包含了一个或多个域的容器，基于JSON格式进行表示。文档由一个或多个域组成，每个域拥有一个名字及一个或多个值，有多个值的域通常称为多值域。每个文档可以存储不同 的域集，但同一类型下的文档至应该有某种程度上的相似之处。 

#### **分片和副本**

ES的分片(shard)机制可将一个索引内部的数据分布地存储于多个节点，它通过将一个索引切分为多个底层物理的Lucene索引完成索引数据的分割存储功能，这每一个物理的Lucene索引称为一个分片(shard)。

每个分片其内部都是一个全功能且独立的索引，因此可由集群中的任何主机存储。创建索引时，用户可指定其分片的数量，默认数量为5个。

Shard有两种类型：primary和replica，即主shard及副本shard。Primary shard用于文档存储，每个新的索引会自动创建5个Primary shard，当然此数量可在索引创建之前通过配置自行定义，不过，一旦创建完成，其Primaryshard的数量将不可更改。Replica shard是Primary Shard的副本，用于冗余数据及提高搜索性能。每个Primaryshard默认配置了一个Replica shard，但也可以配置多个，且其数量可动态更改。ES会根据需要自动增加或减少这些Replica shard的数量。

> 为了将数据添加到ES，我们需要索引(index),索引是一个存储关联数据的地方。实际上，索引只是一个用来指定一个或多个分片的"逻辑命名空间"
>
> 一个分片（shard）是一个最小级别"工作单元"，它只是保存了索引中的所有数据的一部分，每个分片就是一个Lucene实例，并且它本身就是一个完整的搜索引擎。我们的文档存储在分片中，并且在分片中被索引，但是我们的应用程序不会直接与它们通信，取而代之的是，直接与索引通信。
>
> 分片是ES在进群中分发数据的关键，可以把分片想想成数据的容器。文档存储在分片中，然后分片分配到集群中的节点上。当集群扩容或缩小，ES将会自动在节点间迁移分片，以使集群保持平衡。



### 创建索引 / 映射器 /类型

##### 介绍

>  由于`elasticsearch`是遵循Resulf的规范的 , 所以在创建索引的时候将请求 写成put就可以完成对索引的创建 , <font style="color:red">并且最好在创建索引的同时创建映射, 因为往已经存在的索引里面放映射很难</font>

![1567605804459](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567605804459.png)



##### 建立索引

```java
语法：PUT /索引名
在没有特殊设置的情况下，默认有5个分片，1个备份，也可以通过请求参数的方式来指定
参数格式：
{
"settings": {
"number_of_shards": 5, //设置5个片区
"number_of_replicas": 1 //设置1个备份
	}
}
```





##### 代码展示

```java
PUT /shop_product
{
  //创建映射
  "mappings": {
  	//创建类型
    "shop_product": {
      "properties": {
        "id": {
	  "type": "long"
	},
        "title": {
          "type": "text", 
          "analyzer": "ik_smart", 
          "search_analyzer": "ik_smart",
          "fields": {
          "keyword": {
          "type": "keyword",
            "ignore_above": 256
            }  
          }
        },
        "price": {"type": "integer"},
        "intro": {
          "type": "text", 
          "analyzer": "ik_smart", 
          "search_analyzer": "ik_smart",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }  
          }
        },
      	"brand": {
      	  "type": "keyword" 
      	}
      }
    }
  }
}
```

> shop_product : 索引代表了数据库
>
> shop_product : 类型 代表了一张表
>
> mappings  :  映射 代表了创建时期的约束与规范
>
> properties: 文档  代表了数据库中字段 
>
> type  字段类型   用以指定字段的类型
>
> ik_smart : 分词器



##### **查询映射**

```java
语法：GET /索引名/_mapping
```



### <font style="color:red">分词器</font>

#### 介绍

elasticsearch中的核心组件之一 , <font style="color:red">分词器的作用就是将文档的内容通过指定的单词或汉子格式来进行拆分 , 将文档中的内容转变为小写 , 去掉标点 ,遇到中文字会当成一个单词来处理</font> , 是一个功能及其强大的组件 , 默认的分词器是standard , 该分词器只能`通过空格`来处理`英文单词`不支持中文的分词 , 如果要分词中文需要使用中文分词器

#### 用法

##### 安装

将中文分词器解压 然后创建一个文件夹 把配置放进去 最后将这个文件夹放到elasticsearch的插件文件夹下面

![1567604707280](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567604707280.png)

##### 使用

在配置好中文分词器后在`kibana`创建索引然后在控制台分词 

<font style="color:red">分词器有两种</font>

**一 . ik_smart**

效果展示:

> ![1567604801415](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567604801415.png)

代码展示:

> ```java
> GET /shop_product/_analyze
> {
>   "text": "指纹 双卡", 
>   "analyzer": "ik_smart"
> }
> ```

**二. ik_max_word**

效果展示:

> ![1567605016679](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567605016679.png)

代码展示

> ```java
> GET /shop_product/_analyze
> {
>   "text": "英特尔酷睿i7处理器", 
>   "analyzer": "ik_max_word"
> }
> ```

##### 总结

> 尽管两者分词器都可以进行分词的处理 , 不过在分词的结果上 `ik_max_word (细颗度)` 比 `ik_smart(粗颗度)`分词分的更加详细 , 两者的用谁 这看需要就可以了, 
>
> <font style="color:red">可以在`ik`分词器的配置文件中的`config -> main.dic `添加词组用作分词的条件</font>
>
> `analyzer` 是用什么分词器
>
> `search_analyzer` 指定分词器的类型



### <font style="color:red">CRUD</font>

#### <font style="color:red">**文档操作**</font>

##### **新增和替换**

> 语法：PUT /索引名/类型名/文档ID
> {
> field1: value1,
> field2: value2,
> ...
> }
> 注意：当索引/类型/映射不存在时，会使用默认设置自动添加
> ES中的数据一般是从别的数据库导入的，所以文档的ID会沿用原数据库中的ID
> 索引库中没有该ID对应的文档时则新增，拥有该ID对应的文档时则替换
> 需求1：<font style="color:red">新增一个文档</font>
>
> ```java
> PUT /index/users/2
> {
>   "id":12,
>   "name":"狗蛋",
>   "intro":"nixx"
> }
> ```
>
> 需求2：<font style="color:red">替换一个文档</font>
>
> > 替换的条件就是id存不存在
>
> ```c++
> PUT /index/users/2
> {
>   "id":12,
>   "name":"闰土",
>   "intro":"nixx"
> }
> ```
>
> 每一个文档都有一段
>
> ![1567606504592](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567606504592.png)
>
> 其中版本号通常用作乐观锁 而source使我们查询出的数据源



##### <font style="color:red">查询文档</font>

###### 普通查询

**语法:**

> GET /`索引号`/ `类型`/_search

**代码示例:**

```java
GET /shop_product/shop_product/_search
```

**效果展示:**

![1567606843204](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567606843204.png)





###### 排序查询

**语法:**

> GET /`索引`/`类型`/_search
> {
>
> //排序的规则
>
>   "sort":
>   [
>
> ​					`正序还是倒叙`
>
> ​    {"price":"desc"}
>   ]
> }

**代码示例**

```java
#排序查询
GET /shop_product/shop_product/_search
{
  "sort":
  [
    {"price":"desc"}
  ]
}
```

**效果展示**

![1567607176914](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567607176914.png)



###### 分页条件查询

**语法:**

> GET /`索引`/`类型`/_search
> {
>
>   "sort":
>   [
>    {
>      "price": "asc"
>    }
>   ],
>
> `当前页`
>
>   "from": 3,
>
> `页数`
>
>   "size": 3
> }

**代码展示**

```java
#分页查询
GET /shop_product/shop_product/_search
{
 
  "sort":
  [
   {
     "price": "asc"
   }
  ],
  "from": 3,
  "size": 3
}
```



###### 检索条件查询

**语法:**

> GET /shop_product/shop_product/_search
> {
>   "query": 
>   {
>
> `检索条件`
>
> ​    "term":
> ​    {
> ​      "brand": "荣耀"
> ​    }
>   }
> }

**代码展示**

<font style="color:red">检索条件</font>

> term : 等值条件
>
> > ```java
> > #检索查询
> > GET /shop_product/shop_product/_search
> > {
> >   "query": 
> >   {
> >     "term":
> >     {
> >       "brand": "荣耀"
> >     }
> >   }
> > }
> > ```
>
> match : 将文本内容进行分词处理
>
> > 代码展示
> >
> > ```java
> > #将查询的条件也进行分词 游戏 手机
> > GET /shop_product/shop_product/_search
> > {
> >   "query":
> >   {
> >     "match":
> >     {
> >       "title":"游戏 手机"
> >     }
> >   }
> > }
> > ```
>
> range : 根据数据查询范围内的数据
>
> > 代码展示
> >
> > ```java
> > 
> > #查询商品5000-10000的并且从大到小排序------------------
> > GET /shop_product/shop_product/_search
> > {
> >   "query": 
> >   {
> >     "range":
> >     {
> >       "price":
> >       {
> >         "gte": 5000,
> >         "lte": 10000
> >       }
> >     }
> >   },
> >   "sort":
> >   {
> >     "price":"desc"
> >   }
> > }
> > ```







###### 多个检索条件查询

**语法:**

> {
>   "query":
>   {
>
> ​	`检索条件`
>
> ​    "multi_match":
> ​    {
> ​      "query": "指纹 双卡", 
> ​      "fields": ["title","intro"]
> ​    }
>   }
> }

**代码展示**

```java
#多个检索条件(指纹 双卡)查询
GET /shop_product/shop_product/_search
{
  "query":
  {
    "multi_match":
    {
      "query": "指纹 双卡", 
      "fields": ["title","intro"]
    }
  }
}
```



###### 高亮显示

**语法:**

> highlight : 设置高亮部分
>
> fields : 高亮区域那些属性
>
> pre_tags: 设置前缀 (用来替换页面的代码)
>
> post_tags : 设置后缀 (用来替换页面代码)

**代码展示**

```java
#增加高亮
GET /shop_product/shop_product/_search
{
  "query":
  {
    "multi_match": {
      "query": "指纹 双卡",
      "fields": ["title","intro"]
    }
  },
  "highlight": 
  {
    "fields": 
    {
      "title": {},
      "intro": {}
    },
    "pre_tags": "<span style='color:red'>",
    "post_tags": "</span>"
  }
}
```



###### 逻辑查询

**语法:**

> must 用来设置判断条件 含义是 and
>
> should : 条件经常被用来当做谁或谁 or
>
> must_not :条件经常用来取反  not 



**<font style="color:red">逻辑条件</font>**

> 与  (must)
>
> > 代码展示
> >
> > ```java
> > #逻辑查询    查询 标题是i7并且金额大于7000的设备-------------
> > GET /shop_product/shop_product/_search
> > {
> >   "query": 
> >   {
> >     "bool": 
> >     {
> >       "must": [
> >         {
> >           "term":
> >           {"title": "i7"}
> >         },
> >         {
> >          "range": 
> >           {
> >             "price": {"gte": 5000}
> >           }  
> >         }
> >               ]
> >     }
> >   }
> > }
> > ```
>
> 或(should)
>
> > 代码展示
> >
> > ```java
> > 
> > #查询标题是 荣耀 华为 或者金额大于5000 并小于 8000的
> > GET /shop_product/shop_product/_search
> > {
> >   "query": 
> >   {
> >     "bool":
> >     {
> >       "should": [
> >         {"term":
> >           {
> >             "title":"荣耀"
> >           }
> >         },
> >         {"range":
> >           {"price":
> >             {
> >               "lte": 8000,
> >               "gte": 5000
> >             }
> >         }
> >         }
> >       ]
> >     }
> >   }
> > }
> > 
> > ```
>
> 非 (must_not)
>
> > 代码展示
> >
> > ```java
> > #查询金额 不等于 5000-10000的
> > GET /shop_product/shop_product/_search
> > {
> >   "query":
> >   {
> >     "bool":
> >     {
> >       "must_not": 
> >       [
> >         {
> >           "range":
> >           {
> >             "price":
> >             {
> >               "lt":10000,
> >               "gt":5000
> >             }
> >           }
> >         }
> >       ]
> >     }
> >   }
> > }
> > 
> > 
> > ```

###### 过滤查询

**语法:**

###### 分组查询

**语法:**

> aggs : 用来分组的语法规范
>
> groupByBrand 自定义的名字 用什么分组
>
> terms 分组的条件  这里采用等值进行分组

**代码展示**

```java
#分组查询   size设置为0
GET /shop_product/shop_product/_search
{
  "size":0,
  "aggs":
  {
    "groupByBrand":
    {
      "terms": {
        "field": "brand"
      }
    }
  }
}
```



###### **分组查询并使用函数**

**语法:**

> 在分组的条件中新创建一个aggs用来执行函数
>
> "aggs": {
>      "priceMax": {
>        "max": 
>        {
>          "field": "price"  
>        }
>      }
>    }
>
> max : 取最大值
>
> avg : 取平均值
>
> value_count 总数
>
> min 最小值
>
> 

**代码展示**

```java
#分组查询找出最大的那个值
GET /shop_product/shop_product/_search
{
  "size":0,
  "aggs":
  {
    "groupByBrand":
    {
      "terms":
      {
        "field":"brand"
      }
      ,
      "aggs": {
        "priceMax": {
          "max": 
          {
            "field": "price"  
          }
        }
      }
    }
  }
}
```

