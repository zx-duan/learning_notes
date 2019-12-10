# SpringBoot

## 1.什么是SpringBoot

SpringBoot是一个社区反馈推动而生的项目 , 可以说是整个Spring社区 乃至整个JAVA社区最近几年来最有影响力的项目之一 . 

包含了以下的特性

![1567075257701](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567075257701.png)

*该框架非常火，目前新开项目几乎都是基于SpringBoot搭建，非常符合微服务架构要求，企业招聘大多都要求有* 

*SpringBoot开发经验，属于面试必问的点*

## 2.使用SpirngBoot能为我们做什么

SpringBoot搭建Web项目速度快 , 配置文件写的少 , 能完成同样的功能



## 3.如何使用SpringBoot

```xml
    <!-- 打包方式jar包 --> 
	<packaging>jar</packaging>
	<!--统一SpringBoot的版本号-->
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>
	<!--加载SpringBoot的核心资源-->
	<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    <!--加载SpringBoot Web的核心资源-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>							
</dependencies>									
```

##### SpirngBoot Hello word

1.创建SpringBoot的Main方法口

```java
public static void main(String[] args) {
    	//配置运行方法
        SpringApplication.run(HelloSpringController.class, args);
    }
```

2.贴上SpringBoot的<font style="color:red">重要核心注解</font> `@SpringBootApplication`

```java
@SpringBootApplication
```

3.贴上SpringIOC的Controller注解

> 该注解将@ResponseBody和Controller注解合并

```java
@RestController
```

4.配置SpringBoot的访问地址

```java
@RequestMapping("hello")
    public Object hello() {
        return "你好Spring";
    }
```

**以上步骤过后一个简单的SpringBoot web项目就已经搭建完成了**

![1567082406803](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567082406803.png)

## 4.SpringBoot需要注意的点

> SpringApplication.run(..)的作用 
>
> - 启动SpringBoot应用 
>
> - 加载自定义的配置类,完成自动配置功能 
>
> - 把当前项目配置到嵌入的Tomcat服务器 
>
> - 启动嵌入的Tomcat服务器
> - Spring嵌入的Tomcat服务器默认地址是8080



## 5.SpringBoot的自动装配

Spring Boot的自动装配

Spring Boot的自动配置注解是@EnableAutoConfiguration， 从上面的@Import的类可以找到下面自动加载自动配置的映射。

一旦加载了这个注解 , Spring会试图在你的classpath下找到所有配置的Bean然后进行装配 , 当然装配Bean时 , 会根据若干个(Conditional)定制规则进行初始化 , 

```sql
org.springframework.core.io.support.SpringFactoriesLoader.loadFactoryNames(Class<?>, ClassLoader)
```

```java

public static List<String> loadFactoryNames(Class<?> factoryClass,ClassLoader classLoader){
 
		String factoryClassName = factoryClass.getName();
		try{
			Enumeration<URL> urls = (classLoader!=null?classLoader.getResources(FACTORIES_RESOURCE_LOCATION):lassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			
			List<String> result = new ArrayList<String>();
 
			while(urls.hasMoreElements()){
						URL url = urls.nextElement();
					Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
					String factoryClassNames = properties.getProperty(factoryClassName);
					result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
					
			}
			return result;
		}catch(IOException ex){
			
			throw new IllegalArgumentException("Unable to load ["+factoryClass.getName()+"]factoriesfromlocation["+FACTORIES_RESOURCE_LOCATIO+"]",ex);
 		
	}
 
}

```



这个方法会加载类路径及所有jar包下META-INF/spring.factories配置中映射的自动配置的类。

```java

/**
 
 * The location to look for factories.
 
 * <p>Can be present in multiple JAR files.
 
 */
 
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

查看Spring Boot自带的自动配置的包： spring-boot-autoconfigure-1.5.6.RELEASE.jar，打开其中的META-INF/spring.factories文件会找到自动配置的映射。

![img](https://img-blog.csdnimg.cn/20190314144525943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0tldmluX0d1Ng==,size_16,color_FFFFFF,t_70)

再来看看数据源自动配置的实现注解

![img](https://img-blog.csdnimg.cn/20190314144600791.png)

@Configuration,@ConditionalOnClass就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置。





总结 :















##### SpringBoot查看是否关联依赖

![1567171139406](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171139406.png)





##### 后台部分

**SpringBoot的自动装配已经继承了JAVA常用的配置与插件在使用的时候只需要在配置文件 `application.properties`文件中配置即可**



###### 数据库连接池

> 需要配置依赖
>
> ![1567171843661](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171843661.png)







**配置数据库连接池有两种方法**

> ==1.自动配置 , 在配置文件中配置数据库连接四要素就可以了==
>
> ​	![1567171488493](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171488493.png)
>
> ​	![1567171567079](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171567079.png)
>
> ​	在SpringBoot中 已经做了自动装配处理
>
> ​	我们只需要在`application.properties`文件中配置如下信息:
>
> ```xml
> <!--配置德鲁伊连接池-->
> spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
> <!--配置Mysql的驱动-->
> spring.datasource.driver-class-name=com.mysql.jdbc.Driver
> <!--如果使用的Mysql版本要高于5.1.8需要处理时区-->
> spring.datasource.url=jdbc:mysql:///rbac?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
> spring.datasource.username=root
> spring.datasource.password=admin
> ```
>
> 
>
> 

​	

> ==手动配置==
>
> **在配置类中注入 DataSource的Bean**
>
> ![1567171916715](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171916715.png)
>
> **在测试的类中查看该对象是否已经成功创建并加载到Spirng容器中**
>
> ![1567171980276](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567171980276.png)

在日后的使用过程中我们通常使用第一种自动装配的方式配置数据库连接池



###### 配置MyBatis

>  在配置MyBatis的过程中也可以使用自动装配的模式来配置MyBatis与SpringBoot的关联
>
> <font style="color:red">重要的点是 Mapper接口怎么配置 与注入</font>

配置MyBatis首先需要加载依赖

```xml
       <!--mybatis关联spring-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
```

在加载依赖后我们需要将MyBatis注入到Spring的容器中

> 在主配置文件中开启<font style="color:red">Mapper</font>扫描器
>
> ```java
> //需要使用一个新的注解
> @MapperScan("cn.wolfcode.springbootrbac.mapper")
> ```
>
> 该注解的作用就是通过扫描的方式将本地项目的mapper文件中的接口注入到Spring中
>
> 可以理解为以为项目结构中xml的Mapper扫描器



**在成功的配置好扫描器之后要实现以往xml形式的sql处理有两种方法**

**第一种**

> 通过注解的方式来完全取代xml , 包括sql语句也要写在这里面
>
> ```java
> @Mapper 
> //该接口的作用是在配置Mapper扫描器之后使用该注解贴在接口类上面 , 将接口赋予Mapper.xml一样的特性
> ```
>
> 展示:
>
> ```java
> @Mapper
> public interface DepartmentMapper {
>     @Select("select * from Department where id = #{did}")
>     Department selectByPrimaryKey(Integer dId);
> }
> ```
>
> **值得注意的是如果使用了这种方式就不用在配置 mapper.xml文件了 同理 配置了Mapper.xml文件就不用在写这个注解包括这种sql的方式**



**第二种**

> 还是使用mapper.xml来写sql , 将mapper.xml的文件跟以前一样对应接口的地址就可以
>
> ![1567173114650](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567173114650.png)
>
> 需要值得注意的是如果配置了xml方式作为sql的映射执行文件的话需要在SpringBoot的总配置文件中配置用来扫描Mapper映射接口的xml文件
>
> ```xml
> mybatis.mapper-locations=classpath:cn/wolfcode/xxx/mapper/*Mapper.xml
> ```
>
> 





###### 日志

日志只要在总配置文件中配置自动装配就可以了



logging.level<font style="color:red">.cn.wolfcode.springbootrbac</font>=trace

level:级别

通常情况下配置日志只需要配置完这个就可以看到sql语句的执行代码了



###### 事务

> 在以往的过程中配置事务通常都是使用.xml的方式开启aop动态代理与tx增强类型就可以了
>
>  在springboot中 可以使用注解的方式 , 但是如果使用了注解那么每一个要加事务的类就需要贴上一个注解 , 这样造成的后果就是会使用很多意义一样的注解来配置业务层, 一般我们是不推荐使用这种情况的
>
> <font style="color:red">我们可以延续以往使用xml配置的方式来进行配置事务的开启与管理</font>
>
> 创建xml文件
>
> ```xml
>    <!-- CRUD通用事务的配置-->
>     <tx:advice id="txAdvice" transaction-manager="transactionManager">
>         <tx:attributes>
>             <tx:method name="select*" read-only="true"/>
>             <tx:method name="get*" read-only="true"/>
>             <tx:method name="list*" read-only="true"/>
>             <tx:method name="query*" read-only="true"/>
>             <tx:method name="count*" read-only="true"/>
>             <tx:method name="*"/>
>         </tx:attributes>
>     </tx:advice>								
>     <aop:config>
>         <aop:pointcut id="txPointcut"
>                       expression="execution(* cn.wolfcode.springboot.service.impl.*ServiceImpl..*(..))"/>
>         <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
>     </aop:config>
> ```
>
> 在创建了xml文件后我们需要使用配置类来引用 (可以是主配置文件也可以是新创建的配置文件)
>
> ```java
> @ImportResource("classpath:spring-tx.xml")
> //该注解的作用是将本配置类关联 xml文件
> ```
>
> 需要注意的是及时配置以上的内容后 但是由于springboot已经舍弃了xml文件所以我们需要在拦截器或者其他用到aop的地方贴上`@Component`来声明将此配置交给spring来管理
>
> 



##### 前端部分

###### Freemarker

由于SpringBoot已经自动装配了Freemarker所以我们只需要将Freemarker的配置属性加载到总配置文件就可以了

> 其中最重要的就是Freemarker的前缀与后缀了
>
> 为了能让前端控制器管理到Freemarker的视图我们可以在总配置文件中配置一个视图地址通知SpringBoot来管理Freemarker
>
> ```java
> spring.freemarker.prefix=views/
> //这个配置文件的作用就是通知SpringBoot用什么前缀来修饰Freemarker
> ```
>
> 





### 代码展示:

```xml
#去掉Banner
spring.main.banner-mode=off

#重新设置端口号
server.port=80

#配置数据库连接四要素
#配置数据库连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql:///rbac?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=admin

#配置MyBatis扫描别名
mybatis.type-aliases-package=cn.wolfcode.springbootrbac.domain
mybatis.configuration.lazy-loading-enabled=true
mybatis.mapper-locations=classpath:cn/wolfcode/xxx/mapper/*Mapper.xml

#开启日志
logging.level.cn.wolfcode.springbootrbac=trace

#配置freemarker前置视图
spring.freemarker.prefix=views/

#pagehelper分页插件配置
pagehelper.helperDialect=mysql
pagehelper.reasonable=true																	
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql
```



**SpringBoot查找该类是否被Spring加载**

![1567349011477](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1567349011477.png)



SpringBoot的核心注解有哪些 ? 是由哪几个注解组成的 ? 

> 启动类上的注解是**@SpringBootApplication** , 它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：
>
> 

