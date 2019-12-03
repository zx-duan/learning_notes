# Redies

### Redies有什么用 Redies是什么

在关系型数据库数据都是存放在表中,有分类存放,连接查询,主键,外键等概

NoSQL泛指非关系型数据库,采用区别于关系型数据库的设计,主要是针对关系型数据库性能瓶颈来设计的,专门处理关系型数据库不擅长做的业务场景,不同的NoSQL针对的点不一样,大致分为以下几类:

- 键值存储: Redis 多用于项目的高速缓存 
- 文档存储: MongoDB 广泛用于社交类应用 
- 文件存储: FastDFS 多用于以文件为载体的在线服务,如相册网站/视频网站等等 
- 列式存储: HBase 主要用于数据分析领域

### Redies的数据结构

Redis是属于键值存储的NoSQL数据库,该数据库主要是提供高性能的键值存储,可以简答的理解为是一个极其高性能的超大Map,由于这种数据库的数据结构比较简单,所以没有关系型数据库那么多的功能,如: 

- Redis中事务只有成功,没有失败 
- 没有表的概念和表相关的操作 
- 单线程执行,没有线程安全问题

### <font style="color:red">Redies的数据类型</font>

<font style="color:red">string </font>

hash 

<font style="color:red">list </font>

set 

<font style="color:red">zset</font>



### **Redies的特点**

1. 性能高,能达到10w次读/s,8w次写/s 

2. 具有比较简单的==ACID==，如：没有事务回滚 

3. 单线程操作,每个操作都是原子操作,没有并发相关问题



### <font style="color:red">Redies的命令提示 (极其重要)</font>

##### String

![String](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344559604.png)

###### 	写法

> ![1567344602213](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344602213.png)
>
> 



如何运行 jar包 

cmd

>  java -jar xxx.jar 



##### 查询Redis中所有集合的值

> keys *

##### 删除数据库中所有的内容

> flushDB

##### 设置Redis key的缓存时间

> ```java
> template.expire(token, RedisKeys.USER_INFO_LOGIN.getCodeTime(), TimeUnit.SECONDS);
> ```
>
> Redis 有四个不同的命令可以用于设置键的生存时间(键可以存在多久)或过期时间(键什么时候会被删除) :
>
> EXPlRE <key> <ttl> 命令用于将键key 的生存时间设置为ttl 秒。
>
> PEXPIRE <key> <ttl> 命令用于将键key 的生存时间设置为ttl 毫秒。
>
> EXPIREAT <key> < timestamp> 命令用于将键key 的过期时间设置为timestamp所指定的秒数时间戳。
>
> PEXPIREAT <key> < timestamp > 命令用于将键key 的过期时间设置为timestamp所指定的毫秒数时间戳。





##### list

![1567344690697](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344690697.png)

###### 写法

> ![1567344710923](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344710923.png)



##### zset类型

###### 写法

![写法](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344840035.png)



##### hash类型

![1567344900733](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344900733.png)

###### 写法

![1567344911913](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344911913.png)



##### set类型

![1567344959703](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344959703.png)

![1567344987671](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567344987671.png)



### Redis的管理命令

##### 管理key的命令

![1567345133297](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567345133297.png)

##### 设置密码

![1567345142538](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567345142538.png)



### <font style="color:red">Redis与java连接</font>

有两种连接方式

**Jedis**

**Spring Data Redis**



**需要加载依赖**

```xml
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.wolfcode</groupId>
    <artifactId>springredis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <!-- SpringBoot整合Spring Data Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- redis的驱动包:jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
    </dependencies>
```





##### Jedis

> 这种连接方式类似于以往连接Mysql获取的连接对象 , 也是通过new
>
> 的方式来创建连接对象 然后使用连接对象操作数据库

这种连接方式的好处是 一般的Redis命令都可以在这里找到对应的方法 , 直接使用即可

==代码展示:== 

```java
JedisPool pool = new JedisPool();
Jedis jedis = pool.getResource();
String student = jedis.get("cn.wolfcode.springredis.domain.Student1");
Student student1 = JSON.parseObject(student, Student.class);
```



##### **spring data Redis**

> ![1567347632572](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347632572.png)
>
> 这种方式特点是将Redis交给Spring来管理 , 只需要在Spring 的配置文件中配置相关的信息就可以使用了
>
> ![1567347648606](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347648606.png)
>
> 如果使用Spring来管理Redis有两种获取Bean的方式
>
> 第一种方式是获取原生的 , 所有保存或者修改数据库内容是由二进制来完成的 , 所以我们看不到也取不出来
>
> ![1567347842685](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347842685.png)
>
> ![1567347738687](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347738687.png)
>
> ![1567347745246](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347745246.png)
>
> 
>
> 第二种是使用了一个以String为输入格式的类不过本质还是继承了原生的
>
> 这里我们推荐使用第二种 理由就是方便观看和操作数据库
>
> ![1567347802836](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347802836.png)
>
> 

这种连接的方式是省略了创建对象这一过程 代码量更少不过缺点是有的时候方法名对应不上命令需要寻找而且不同的数据类型需要调用不同的方法感觉很麻烦

==代码展示:==

###### Redis的查询

```java
  		//查询单个
        String name = redisTemplate.opsForValue().get("name");
        System.out.println(name);
        //查询全部
        Set<String> keys = redisTemplate.keys("*");
        Iterator<String> iterator = keys.iterator();
        while (iterator.hasNext()) {
            String next = iterator.next();
            System.out.println(next);
        }
```



###### 设置Redis的过期时间

```java
//该方法是设置Redis的缓存时间 
/**
*TimeUnit.SECONDS  以秒为单位
 Token             设置哪个Key的时间
 time              设置多长的时间
*/

template.expire(String token, Long time, TimeUnit.SECONDS);
```











两种方式见仁见智



### Redis的持久化

**为什么Redis要持久化**

因为Redis的所有保存都是用缓存做的, 如果在没有保存的时候Redis服务器崩溃了怎么办

持久化的前提是 , 要持久化的属性与本身业务并没有关系 , 比如 点赞的数量 , 不能持久化的属性比如金额



![1567347907009](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567347907009.png)

> 两种持久化 不同的是 全量的话是每隔60秒就执行一次保存 , 而增量是每隔一秒执行上一秒的操作





![1567691772896](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567691772896.png)



### **缓存的数据**

==冷数据==: 修改不频繁的数据  条件查询远大于修改

==热数据==: 修改的频率比较高  数据频繁的改动

​			1: 数据非核心数据 

​			2: 可以减少数据库的访问

​			3: 不要求实施但是最终一致











### 面试中Redis的雷区

Redis是单线程的 , 不存在线程安全问题

Redis所有操作都是原子性的 , 不存在某些功能才有原子性









