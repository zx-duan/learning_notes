# MyBatis

------------->在印象笔记六月笔记栏里

什么是MyBatis? 

MyBatis是一个半ORM的关系映射型数据库框架 , 由于它是半自动型的 , 所以我们仍然要编写SQL语句





# MyBatis底层原理





## Mapper接口

Mapper组件 = Mapper接口 +  Mapper.XML 文件



![1573179841278](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573179841278.png)

通过打印接接口，发现打印的是：class com.sun.proxy.$Proxy5，底层使用的是动态代理 , 生成Mapper接口的实现类

接口是规范，实质做的实现还是要由实现类对象来做，而这个实现类不需要我们写，实现类对象也不由我们创建，这些都 MyBatis 使用动态代理帮我们做了；
我们只需提供 Mapper 接口对应 Mapper XML 文件，获取实现类对象的时候传入 Mapper 接口就可以了（不然 MyBatis 也不知道你要获取哪个 Mapper 接口的实现类对象）；
至于实现类中操作方式底层还是和之前的一样，因为 Mapper XML 命名空间是使用 Mapper 接口的全限定名，方法名又与对应 XML 元素 id 一致，所以可以通过获取调用方法所在 Mapper 接口的权限名和方法名，拼接出 namespace + id，再配合调用方法的实参就可以像之前一样操作了。







# MyBatis的缓存

#### 为什么要用MyBatis的缓存

当我们查询一些没有什么变化的数据时候 , 如果数据可以存放在内存中 , 我们就可以减少一些IO流的操作 , 这样一是可以提高查询的效率 , 二是可以减少数据库的压力

#### MyBatis的缓存有哪些

MyBatis中的缓存分为**一级缓存**与**二级缓存** , 其中一级缓存默认是开启的状态 , 二级缓存是需要配置的

<font style="color:red">**一级缓存是与Session绑定的**</font>

![1573211065759](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573211065759.png)

==一级缓存的性能提升是有限度的 , 因为每次操作都需要创建一个新的SqlSession== , 而Session之间是不共享数据的所以一级缓存对我们的帮助并不大

>  <font style="color:red">**注: 每个SqlSession之间线程是不安全的**</font>
>
> 一级缓存是不可以关闭的 , 但是可以清空缓存中的内容 , 比如使用SqlSession的 `clearCache`





<font style="color:red">**二级缓存是与Mapper绑定的**</font>

二级缓存是Mapper级别的 , 作用域是Mapper的namespace , 二级缓存与namespace绑定在一起 , 不同SqlSession对象也可以共享缓存 , ==只要是执行同一命名空间的SQL就会执行缓存==



二级缓存的细节

> 5.1、映射文件中所有的 select 都会使用查询缓存。
> 5.2、大多数情况下，不会缓存查询列表（查询多条数据）的 select，因为只有 SQL 和查询参数完全相同，才会使用到查询缓存（设置 useCache="false"），查询条件可能太多，会导致缓存太多数据。
> 5.3、一般的，只会对 get 方法（查询单条）做查询缓存。
> 5.4、文件中所有的 insert、update、delete 语句都会刷新缓存（针对 update、delete 设置 flushCache="true"）。一般的，针对 insert 操作不需要刷新缓存（设置 flushCache="false"）。







**真正想要提升性能还是需要开启二级缓存**







#### 如何使用MyBatis的缓存?



MyBatis分为一级缓存与二级缓存 , 一级缓存放是与Session绑定的 , 默认就有 , 提升的性能有限



##### 缓存的问题

![1573211804862](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573211804862.png)

当修改的时候 , 由于请求是从内存中获取数据的 , 所以可能会造成读取假数据的情况 , 如果设置修改数据就会清空缓存的时候 , 就会造成容易因为一条数据的修改清空所有的缓存

##### **缓存性能相关属性**

![1573212793632](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573212793632.png)



##### 使用二级缓存 

1. 在全局配置中开启

![1573213297099](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573213297099.png)

​	2.在Mapper中开启

​						![1573213325164](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1573213325164.png)

​	3.被缓存的对象必须要实现序列化的接口







<font style="color:red">注: 若是想实现MyBatis的自定义的缓存 , 只需要实现MyBatis的`Cache`接口方法就可以了,MyBatis已经内置了一个缓存的实现 </font>





# MyBatis延迟加载

指的是真实需要数据执行的时候才会执行SQL语句进行查询 ,避免无所谓的性能开销 , 使用MyBatis的延时加载可以有效的减少数据库的访问压力 , 首次查询只需要查询主要的信息 , 关联信息等用户获取时在加载



```xml
<settings>
<!-- 设定延迟加载启动 -->
<setting name="lazyLoadingEnabled" value="true" />
<!-- 设定积极加载 -->
<setting name="aggressiveLazyLoading" value="false" />
<!-- 触发延迟加载 clone -->
<setting name="lazyLoadTriggerMethods" value="clone"/>
<!-- 开启延迟,不积极加载,只有调用员工的dept方法时候才加载 -->
</settings>
```

