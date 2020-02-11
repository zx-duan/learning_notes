# MongoDB

### 什么是MongoDB

MongoDB是一种非关系型数据库 , 相比Redis , 可以执行更多更复杂的操作 , MongoDB支持的数据结构非常松散,是一种类似于json格式的Bson格式, 不过用法和json也差不多, **其中MongoDB最大的特点就是支持的查询语句非常强大,其语法有点类似于面向对象的查询语言 , 几乎可以实现类似关系数据单表查询的绝大部分功能, 而且还支持对数据建立索引**



**MongoDB的默认端口号是** 

27017





**特点** 

- 高性能 
- 易部署 
- 易使用 
- 非常方便的存储数据

![1567476614041](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567476614041.png)

### 数据库的结构 

 MongoDB是非关系型数据库 , 是没有表的概念的,该数据库存储的数据结构是集合 , 集合中每一个元素是一个文档(类似于树状的结构)

![1567477127549](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567477127549.png)





### 在线文档

 https://www.runoob.com/mongodb/mongodb-tutorial.html

MongoDB中有很多操作指令, 不同的指令有不同的数据格式 , 需要的时候可以去这里查看



### MongoDB的数据库操作

在使用了navcat连接MongoDB后可以使用命令操作数据库

![1567477616436](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567477616436.png)



### MongoDB支持的数据类型

> String（字符串）: mongodb中的字符串是UTF-8有效的 
>
> Integer（整数）: 存储数值。整数可以是32位或64位，具体取决于您的服务器 
>
> Boolean（布尔）: 存储布尔(true/false)值 
>
> Double（双精度）: 存储浮点值
>
> Arrays（数组）: 将数组或列表或多个值存储到⼀个键中 
>
> Timestamp（时间戳）: 存储时间戳 
>
> Object（对象）: 嵌⼊式⽂档 
>
> Null （空值）: 存储Null值
>
> Symbol（符号）: 与字符串相同，⽤于具有特定符号类型的语⾔
>
>  Date（⽇期）: 以UNIX时间格式存储当前⽇期或时间 
>
> Object ID（对象ID） : 存储⽂档ID
>
>  Binary data（⼆进制数据）: 存储⼆进制数据 
>
> Code（代码）: 将JavaScript代码存储到⽂档中 
>
> Regular expression（正则表达式）: 存储正则表达式





### 新增数据

**模型** 

db. +  集合名. + 增加语句 (insert...)({document ...})

**示例:**

```sql
//添加多条数据
db.users.insertMany([{id:1,name:"李四",age:18},{id:2,name:"闰土",age:8}])
//添加一条数据
db.users.insert({id:4,name:"狗子",age:19})
```

> 值得注意的是如果直接写数字 , 数据库中默认存储的类型是double类型 , 不过不影响
>
> 如果需要在添加数据的时候指定数据类型
>
> ```sql
> db.users.insert({id: NumberLong(1), name: "狗子", age: NumberInt(18)}) db.users.insert({id: 2, name: "bunny", age: 20})
> ```



### 修改数据

修改集合中已经存在的文档

**模型**

db. 集合名. 更新语句 ( update... )( { 修改条件 } , { $set : { 修改的元素 , 修改的元素 } } )

![1567479285660](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567479285660.png)

**参数说明**

> **query** : update的查询条件，类似sql update查询内where后面的。
>
> **update** : update的对象和一些更新的操作符（如$，$inc...）等，也可以理解为sql update查询内set后面的
>
> **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew，true为插入，默认是false，不插入。
>
> **multi** : 可选，mongodb 默认是false，只更新找到的第一条记录，如果这个参数为true，就把按条件查出来多条记录全部更新。
>
> **writeConcern** :可选，抛出异常的级别

**示例**

```sql
//修改多条数据
db.users.updateMany({name:"李四"},{$set:{name:"bunny",age:30}})
//修改数据修改的是很多条
db.users.update({name:"狗蛋"},{$set:{age:18,name:"雷神"}})
//修改单个数据
db.users.updateOne({name:"狗子"},{$set:{age:30,name:"大天狗"}})
```



### 删除操作

删除集合中的文档

**语法**

![1567479719564](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567479719564.png)

**参数说明**

> **query** :（可选）删除的文档的条件。 
>
> **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。 
>
> **writeConcern** :（可选）抛出异常的级别

**示例**

```sql
//删除单个
db.users.deleteOne({name:"张三"})
//删除全部
db.users.deleteMany({name:"张三"})
```



### <font style="color:red">查询操作</font>

**语法**

db.集合名.find(query， projection)

**参数**

> **query** ：可选，使用查询操作符指定查询条件 
>
> **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）

**示例:**

###### **查询全部**

```sql
//查询全部
db.users.find();
```

###### **查询等值条件**

```sql
//查询等值条件
db.users.find({name:"大天狗"});
```

<u>*值得注意的是 , 括号中的参数是可变的 , 也就是说可以写很多个条件*</u>

###### **排序查询**

```sql
db.users.find().sort({age:1})
db.users.find().sort({age:-1})
//该字段需要以数字格式来定比如使用年龄
```

###### **分页查询**

```sql
db.users.find().skip(2).limit(2)
skip : 表示分页的第几页 就是以前经常写的currentPage
limit : 表示分页的一页的内容数量 就是以前写的pageSize
```

###### **比较查询**

**语法**

find({字段: {比较操作符: 值， ...}})

**比较运算符**

> (>) 大于 - $gt 
>
> (<) 小于 - $lt 
>
> (>=) 大于等于 - $gte 
>
> (<= ) 小于等于 - $lte 
>
> (!=) 不等 - $ne 
>
> 集合运算 - $in 如:{name: {$in: ["xiaoyao"，"bunny"]}} 
>
> 判断存在 - $exists 如:{name: {$exists:true}}

**示例**

```sql
//查询年龄大于20的人
db.users.find({age:{$gt:20}})
```



###### **逻辑查询**

**语法**

find({逻辑操作符: [条件1， 条件2， ...]})

> (&&) 与 - $and 
>
> (||) 或 - $or 
>
> (!) 非 - $not

**示例**

```sql
//值得注意的是 逻辑操作符后面是要追加一个中括号的
db.users.find({$and:[{name:"大天狗"},{age:30}]})
```

###### 模糊查询

MongoDB的模糊查询使用的是正则表达式的语法 

```java
如:{name: {$regex: /^.*keyword.*$/}}
```

但是实际上MongoDB也是不擅长执行模糊查询的，在实际开发中也是不使用的，**该功能了解即可**

**示例**

```sql
db.users.find({name:{$regex: /^.*张.*$/}})
```



# JAVA连接MongoDB

**加载依赖**

```xml
<!--spring boot data mongodb-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```

MongoDB不像Redis那样可以使用加载依赖或者交给Spring管理的

#### 配置application.properties的连接参数

```xml
# application.properties
# 配置数据库连接
#格式: mongodb://账号:密码@ip:端口/数据库?认证数据库
spring.data.mongodb.uri=mongodb://root:admin@localhost/mongotest?authSource=admin
# 配置MongoTemplate的执行日志
logging.level.org.springframework.data.mongodb.core=debug
```

```java
没有密码的连接是
#mongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/demo
```







**Java连接MongoDB有两种方式**

#### 集成接口的方式

创建一个接口用来装载MongoDB的容器 , 使用这个接口通过继承`MongoRepository`这个类来与当前所操作的MongoDB数据库关联

**需要在实体类中设置实体类的的配置**

```java
@Document("users") //设置文档所在的集合 
public class User { 
    @Id 
    //文档的id使用ObjectId类型来封装，并且贴上@Id注解 
    private ObjectId _id; 
    private Long id; 
    private String name; 
    private Integer age; 
}
```

> ObjectId 一种特殊的属性  作用是在操作数据库的时候 这个属性是集合中文档对象的ID
>
> 可以看做是Mysql中的主键
>
> `@Document` 该注解的作用是设置文档所在的集合 也就是设置连接的数据库中的集合

###### **<font style="color:red">MongoRepository</font>**

> 该接口对MongoDB数据的常用操作进行了封装，我们只需要写个接口去继承该接口就能直接拥有的CRUD等基本操作，再学习下JPA的方法命名规范，可以毫不费力的完成各种复杂的高级操作

```java
/**
* 自定义一个接口继承MongoRepository，
* 泛型1：domain类型
* 泛型2：文档主键类型
* 贴上@Repository注解，底层(spring-data-jpa)会创建出动态代理对象，交给Spring管理
*/
@Repository
public interface UserMongoRepository extends MongoRepository<User, ObjectId> {
// 使用Spring Data命名规范做高级查询
List<User> findByName(String name);
}
```

###### <font style="color:red">**Spring Data的命名规范**</font>



![1567495350500](C:\Users\Administrator\Desktop\记录\learning_notes-master\imgs\1567495350500.png)

###### **API大全**

|   返回值类型   | 调用的方法 |                            方法名                            | 作用                                            |
| :------------: | ---------- | :----------------------------------------------------------: | ----------------------------------------------- |
| Optional<User> | 自定义接口 |  findById(<font style="color:red">ObjectId objectid</font>)  | 根据ID查询单个文档                              |
|    List<T>     | 自定义接口 |                   findByName(String name)                    | 根据名字返回一组文档                            |
|      null      | 自定义接口 | deleteById(<font style="color:red">ObjectId</font> objectid) | 根据ID删除一个文档                              |
|      User      | 自定义接口 |       insert(<font style="color:red">User user</font>)       | <font style="color:red">保存或者修改数据</font> |
|      User      | 自定义接口 |        save(<font style="color:red">User</font> user)        | 保存或者修改数据                                |

> insert 保存的文档是一次性保存 
>
> save 保存的文档是循环保存 
>
> <font style="color:red">保存和修改都是用save方法 , 不过值得注意的是保存的这个方法存入ID的时候就变成了修改</font>
>
> 通过java保存进去的文档会自动追加一个列 存放的是当前对象的字节码文件路径
>
> 其余的通过查看文档就可以看 比如where and 只要命名规范就可以查询得到





#### 通过Spring来管理

###### **<font style="color:red">MongoTemplate</font>**

该对象SpringBoot已经完成了自动装配 , 存入Spring的容器之中, 我们可以直接注入并使用, 依靠该对象可以完成<font style="color:red">任何</font>MongoDB操作,一般和MongoRepository分工合作, 多数用于复杂的高级查询以及底层操作

###### 使用方式

```java
//注入MongoTemplate
@Autowired
private MongoTemplate template;
```

**使用MongoTemplate时与上述的Repository不同**

<font style="color:red">在使用的时候该属性有个 ==Query==的概念 Query 类似于查询条件的封装 , 在Query中 使用==Criteria==类来添加查询的条件</font>

示例 : 

```java
//Criteria
@Test
  public void testQuery() {
     template.find(new Query(Criteria.where("name").is("海王八")), User.class);
  }
```

> 上述代码中 
>
> where 是 根据什么查  
>
> is 是 查询的条件 
>
> 结果: User(_id=5d6e1d9e45ec42432cab3035, id=3, name=海王八, age=1000)



###### 修改

| 返回值类型   | 调用者        | 方法名                                                       | 作用                                    |
| ------------ | ------------- | ------------------------------------------------------------ | --------------------------------------- |
| UpdateResult | MongoTemplate | upsert(<font style="color:red">Query</font> query,<font style="color:red">Update</font> update,Class<T> class) | 作用是根据query条件更新update里面的内容 |
|              |               |                                                              |                                         |
|              |               |                                                              |                                         |
|              |               |                                                              |                                         |
|              |               |                                                              |                                         |

示例:

> ```java
>  Query query = new Query();
>  query.addCriteria(Criteria.where("name").is("海王八"));
>  Update update = Update.update("name", "海王");
>  UpdateResult upsert = template.upsert(query, update, User.class);
>  System.out.println(upsert); 
> ```
>
> query : 更新的条件
>
> update : 要更新的内容
>
> Update跟Criteria作用差不多都是作为条件或者内容存在的



**<font style="color:red">查询数组内的内容</font>**

> ```java
> db.schools.find( { zipcode: "63109" },
>                  { students: { $elemMatch: { school: 102 } } } )
>      db.online_goods.find({"goodsProStandard.sku":"201811271109159154641"},{goodsProStandard:{$elemMatch:{sku:"201811271109159154641"}}})
> ```



<font style="color:red">~~旧的~~</font>

```java
Query querys = new Query();
        querys.addCriteria(Criteria.where("goodsProStandard.sku").is("201811271109159154641")
                .and("goodsProStandard").elemMatch(new Criteria().and("sku").is("201811271109159154641")));
        List<OnlineGoods> goods = mongoTemplate.find(querys, OnlineGoods.class);
```

> ~~elemMatch 用于设置集合内的查询条件~~



```java
    @Override
    public List<GoodsAttribute> queryBySku() {
        Query query = new Query();
        Criteria criteria = Criteria.where("sku").is("201811271109159154641");

        query.addCriteria(Criteria.where("goods_attribute_list").elemMatch(criteria));
        query.fields().elemMatch("goods_attribute_list", Criteria.where("sku").is("201811271109159154641"));
        List<GoodsActivityApply> goods_activity_apply = mongoTemplate.find(query, GoodsActivityApply.class, "goods_activity_apply");
        for (GoodsActivityApply goodsActivityApply : goods_activity_apply) {
            System.out.println(goodsActivityApply);
        }
        return null;
    }
```









###### 删除

| 返回值类型   | 调用者        | 方法名                                                       | 作用                     |
| ------------ | ------------- | ------------------------------------------------------------ | ------------------------ |
| DeleteResult | MongoTemplate | remove(<font style="color:red">Query </font>query,<font style="color:red">Class</font> class) | 根据条件删除相对应的文档 |
|              |               |                                                              |                          |
|              |               |                                                              |                          |
|              |               |                                                              |                          |

> 只传递一个查询条件是不够的的 , 需要在传入一个字节码对象用来找到对象里的属性

示例:

```java
//删除
    @Test
    public void testDelete() {
        Query query = new Query();
        query.addCriteria(Criteria.where("name").is("闰土"));
        DeleteResult remove = template.remove(query, User.class);
        System.out.println(remove);
    }
```







###### 保存

| 返回值类型 | 调用者        | 方法名                                              | 作用                         |
| ---------- | ------------- | --------------------------------------------------- | ---------------------------- |
| T          | MongoTemplate | save(<font style="color:red">T</font> objectToSave) | 可以保存一条或者多条文档数据 |
|            |               |                                                     |                              |
|            |               |                                                     |                              |
|            |               |                                                     |                              |

> save
>
> insert
>
> 虽然两个都可以实现保存的操作但是insert是一次性添加多条数据 而 save是循环保存数据
>
> 两者都可以传入多个或者传入集合



###### 查询









# 应用的场景

地理位置存储

MongoDB支持地理位置 , 二维空间索引 , 可以存储经纬度 , 因此可以很快速的算出两点之间的距离 , 等位置信息 , 比如查询附近的人 , 或者订餐系统 , 配送系统