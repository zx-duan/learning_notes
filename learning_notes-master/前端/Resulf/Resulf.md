# Resulf

### 什么是Resulf

**Resulf是一种SpringMvc的增强 , Resulf是基于HTTP协议1.1的 , 是一种新的架构风格**

**在REST中**

（1）每一个URI代表一种资源；
（2）客户端和服务器之间，传递这种资源的某种表现层；
（3）客户端通过HTTP提供的统一接口，对服务器端资源进行操作，实现"表现层（使用HTTP动词去促使服务器端资源的）状态转化"。

<font style="color:red">其特点是直接使用Http + JSON的方式来开发</font>

**Http1.1 的特性**

通过异步请求,或者提交表单让Http请求头状态发生改变

新增：从无到有 状态的变化
更新：从某个状态变成另外一种状态的转化 
删除：从有到无 状态的变化

**Http统一接口**
REST要求，必须通过统一的接口来对资源执行各种操作。对于每个资源只能执行一组有限的操作。

HTTP1.1协议为例：
7个HTTP方法：**GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS**
HTTP头信息（可自定义）
HTTP响应状态代码（可自定义）
这些就是HTTP1.1协议提供的统一接口

以前
http://www.wolfcode.cn/employee/saveOrUpdate.do?id=1&name=xx

现在
http://www.wolfcode.cn/employees
新增：POST
更新：PUT
删除：DELETE
查询：GET



###### Resulf的规定

> 1. 在Resulf中,请求的地址不能含有英文中的动词 
>
> 2. 作移动端的时候需要一个版本号的概念 , 在访问的参数中携带版本信息 , 如果用户选择不更新则调用原先的接口
> 3. 在获取单个路径的时候需要区分地址传参和资源路径传参的区别
> 4. /  与 ? 携带参数的方式并没有什么本质的区别
> 5. ![1566045633218](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566045633218.png?Expires=1566049309&OSSAccessKeyId=TMP.hW7V69nVJ2sXKeMVMvbPZtHusmTjDLqjpQuVFqcm3zCzuKorcqx7NTZzAEQHzgdryHsi2Y9wZqUx2qxxVods7TSnfSko7B4agGYgdUQSgdMGwU8MGYiuD2EgFEVn6m.tmp&Signature=B0oRMYzG1JS5gaqBzSqR1WIOFcs%3D)





##### Resulf对于特殊的请求处理

对于put请求Resulf有特殊的处理方式

<font style="color:red">将请求设置为POST 并在web.xml中设置处理put请求的特殊过滤器</font>

> ```xml
>    <!--    设置Put请求等特殊请求访问-->
> <filter>
>         <filter-name>HiddenHttpMethodFilter</filter-name>
>         <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
>     </filter>
>     <filter-mapping>
>         <filter-name>HiddenHttpMethodFilter</filter-name>
>         <servlet-name>dispatcherServlet</servlet-name>
>     </filter-mapping>
> 
> <filter>
>         <filter-name>httpPutFormContentFilter</filter-name>
>         <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
> </filter>
> <filter-mapping>
>         <filter-name>httpPutFormContentFilter</filter-name>
>         <servlet-name>dispatcherServlet</servlet-name>
> </filter-mapping>
> ```
>
> 

<font style="color:red">在表单提交的过程中,需要添加隐藏域 来设定当前提交的请求是put</font>

![1566049120638](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566049120638.png)

<font style="color:red">底层实现的原理:</font>

![1566049163987](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566049163987.png)



##### 常见的响应码

![1566045852439](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566045852439.png)





### 接口

###### <font style="color:red">★什么是面向接口</font>

<u>*在实际的开发中 , 接口通常是指获取数据的对应口 , 通过访问对应的接口 , 获取该业务逻辑的数据*</u>



###### 请求头

**Accept与Content-Type的区别**
1.Accept属于请求头， Content-Type属于实体头。 
Http报头分为通用报头，请求报头，响应报头和实体报头。 
请求方的http报头结构：通用报头|请求报头|实体报头 
响应方的http报头结构：通用报头|响应报头|实体报头

**2.Accept代表发送端（客户端）希望接受的数据类型。** 
比如：Accept：application/json; 
代表仅客户端希望接受的数据类型是json类型,后台返回json数据

Content-Type代表发送端（客户端|服务器）发送的实体数据的数据类型。
比如：Content-Type：application/json; 
代表发送端发送的数据格式是json, 后台就要以这种格式来接收前端发过来的数据。
Content-Type 是无论前台后台都可以携带请求头并通知携带的数据类型

**二者合起来** 
Accept:application/json；  :  <font style="color:red">代表了服务器接收客户端的请求后 应该返回给客户端一个json格式的数据</font>
Content-Type:application/json; : <font style="color:red">代表了服务器需要一个json格式的数据并返回一个json格式数据</font>
即代表希望接受的数据类型是json格式，本次请求发送的数据的数据格式也是json格式。



### Spring MVC 如何搭建Resulf

1.在spring - mvc.xml中 添加`<mvc:annotation-driven/>`注解扫描器

2.开启spring-mvc.xml 中添加`IOC`扫描器

```xml
<context:component-scan base-package="cn.wolfcode.resulf"></context:component-scan>
```

3.在web.xml的配置文件中将Servlet的过滤器设置为 `/`并且在`mvc.xml`中设置静态资源过滤

```xml
<mvc:default-servlet-handler></mvc:default-servlet-handler>
```



#### Spring MVC 的Resulf有什么特点?

**修改控制器中的访问方式**

需要使用新的注解

> @RestController    :   <font style="color:red">作用是将json和ioc注解合并</font>
>
> @<font style="color:red">Get</font>Mapping | <font style="color:red">Post</font>  | <font style="color:red">delete </font>| <font style="color:red">put </font>... 
>
> 

**<font style="color:red">请求资源时携带参数的问题</font>**

**1.可以直接在请求的地址上写上要传递的参数**

```java
@DeleteMapping("{id}")
```

**2.可以根据表单提交 来传递参数**

```java
localhost:80/tests/xml?id=123
```



<font style="color:red">**获取请求地址中参数需要使用到一个注解**</font>`@PathVariable`

Spring MVC通过该注解 自动的去匹配该类型的参数并将数据封装进去



### Resulf的注意事项

#### 时间

在做删除请求的时候Resul的规范默认是认为空返回的 , 并且删除操作并不会处理 ` ?` 后面携带的参数

在从客户端像服务器中发送日期格式的数据时要将时间转换 使用 

```java
@DateTimeFormat(pattern=("yyyy-MM-dd HH:mm:ss"))
```

从服务器响应回客户端的时间格式要做日期转换并计算地区时差 使用 

```java
@JsonFormat(pattern = ("yyyy-MM-dd HH:mm:ss"), timezone = "GMT+8")
```



### Resulf对于请求头的处理

在访问请求的注解中如果配置了`headers`这个属性那么可以对从客户端发送的请求来进行判断请求头

![1566047073154](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566047073154.png)

> ```java
> @GetMapping(value = "{id}/test", headers = "content-type=application/xml")
> ```

用来限定请求头的信息 如果不存在则报 **415** 异常

限定参数:

![1566047223841](https://trip723.oss-cn-shenzhen.aliyuncs.com/%E8%9E%BA%E7%AA%9D%E7%AA%9D/imgs/1566047223841.png)





**实际开发中对于不同前端的请求处理**

因为在其他的前端比如 安卓 , ios中不存在From表单的这个概念 , 所以我们服务器接收到的数据有可能本身就是`json`格式,或者其余`XML`的 , 此时我们需要对这类数据做一个统一的处理

**1.如果数据是json格式需要在请求的参数中 贴上一个新的注解 `@RequestBody`**

**<u>这样做的前提是 前端的请求头数据中指定的是 application / json格式的数据</u>**
