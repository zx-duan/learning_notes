## redis的高级使用

#### **命令**

> **ttl**   key            //查询key的有效时间
>
> **persist**  key    //取消某一个key的失效时间  
>
> **select**   [0-15]   //切换不同的数据库
>
> **move**  key  [0-16]  //移动数据库
>
> **dbsize**    查看Redis当前数据库中有几个key
>
> <font style="color:red">**info**</font>   查看Redis的配置 通常用来==主从操作==
>
> **flushall** 清除所有库中的key
>
> **auth**  写上密码
>
> **multi**   开启事务
>
> **discard**  取消事务
>
> **incr**   将key的值增1 



**场景一 : 我们需要保存同一个key怎么办?**

使用不同的数据库来存储redis的数据



**查看当前是否是生成环境 , 看到底是哪一个库**



#### 事务

Redis的事务可以一起成功不能一起失败 , 当开启了Redis的事务后 , 如果进行保存操作 , redis会将操作的key和值放到一个队列里面 , 然后通过 `exec ` 来统一进行提交

由==multi==来开启事务  , 由==exec==来提交事务 , 由==discard==来取消事务



#### 持久化

Redis的持久化分为两种方式  **RDB** 和 **AOF**的方式

**RDB**  每隔一段时间 如果检测key超过了指定量就发送一个快照 , 如果持久化成功 会在文件中看到一个**==dump.rdb==**的文件这个就是持久化的文件

**AOF** 在某一个时间内将对应的命令存储起来 , 如果要开启AOF持久化需要打开 **appendonly** 的属性,当持久化的时候会在配置文件中保存一个 appendonly.aof的文件



#### Redis的实际运用

Redis的使用有两种方式 , 分别是 jedis 和 使用SpringBoot提供的Template

使用Jedis的方式虽然可以远程连接Redis的数据库但是每次使用都需要关闭资源这样也比较麻烦 , 比较好的做法就是通过建立一个连接池来为Redis的请求进行分发 , 配置最大连接数等



#### 主从复制

一个master可以拥有多个slave

多个slave可以连接同一个master外还可以连接其他的slave

主从复制不会阻塞master , 在同步数据时master可以继续处理client请求

提供系统的伸缩性



**主从复制的过程**

slave与master建立连接 , 发送sync同步命令

Master会开启一个后台进程 , 将数据库快照保存到文件中 , 同时master主进程会收集新的写命令并缓存

后台完成保存后 , 就将文件发给slave

Slave将此文件保存到硬盘上



主从复制的配置 : 

clone服务器之后修改slave的ip地址

修改配置文件中 redis.conf文件

第一步 slaveof==<masterip><masterip>==

第二步 : masterauth<master-password>

使用Info查看role角色即可知道是主服务或者从服务

(需要修改一下bind 0.0.0.0 和 protected-mode no)也就是开启远程访问