# Spring注解配置

#### **为什么要使用Spring注解配置的方式来配置Spring容器 , 乃至整个项目**

> 我们通常使用Spring都会使用xml对Spring进行配置，随着功能以及业务逻辑的日益复杂，应用伴随着大量的XML配置文件以及复杂的Bean依赖关系。随着Spring 3.0的发布，Spring IO团队逐渐开始摆脱XML配置文件，并且在开发过程中大量使用“约定优先配置”（convention over confifiguration）的思想来摆脱Spring框架中各类繁复纷杂的配置 （即是Java Confifig）。 
>
> 从Spring 3起，JavaConfig功能已经包含在Spring核心模块，它允许开发者将bean的定义和Spring的配置编写到到 Java类中。但是，仍然允许使用经典的XML方式来定义bean和配置spring，JavaConfifig是另一种替代解决方案。 
>
> Spring的JavaConfifig也叫做**Annotataion**配置
>
> 



#### **使用旧的Spring配置的缺点:**

1.xml文件太多  不好管理

2.容器太多不好划分

> 在旧的配置模式中 , 需要将bena对象配置到xml文件中 , 然后在从spring容器里获取到该对象的Bean , 这样会造成xml文件过多并且配置及其繁琐



#### 新的方式替代 Spring xml文件的配置

使用`AnnotationConfifig`的方式来解决这一问题

###### <font style="color:red">@Configuration</font>

> 在类上贴该注解表示该类是取代applicationContext.xml文件,也称为Spring的配置类

###### <font style="color:red">@Bean</font>

> 在Spring的配置类的方法上贴该注解表示该方法的返回的对象交给Spring容器管理,替代 applicationContext.xml中的bean标签

###### <font style="color:red">@ComponentScan</font>

> 在Spring配置类上贴该注解表示开启组件扫描器,默认扫描当前配置类所在的包,也可以自 己指定,替代xml配置中的<context:component-scan />标签



###### 使用步骤

1. 定义一个配置类来替代xml配置文件返回一个Bean对象来交给Spring来管理

   ![1567076027894](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567076027894.png)

2. 加载配置类, 启动`AnnotationConfifigApplicationContext`容器对象,测试结果

![1567076068395](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567076068395.png)



**在上述的方法中在配置类的内部返回 Bean对象来交给Spring来管理是可行的 当时造成的后果就是方法需要写很多个**



>  使用xml中的组件扫描器

```java
@ComponentScan //开启组件扫描器,默认扫描当前类所在的包,及其子包
```

在普通的xml配置中,  可以设定xml的组件扫描器来通知Spring管理我们所要加载的容器对象 , 那么在注解的配置中也可以通知Spring来管理我们要加载的容器对象

> 使用方法
>
> 1. <font style="color:red">在主配置文件上面贴上这个组件扫描器</font>
>
>    > 在使用这个注解扫描器的时候 , 默认是会扫描当前配置文件所在的资源路径包下面的 , 所以使用的时候尽量放在根目录下面
>
> 2. <font style="color:red">在需要初始化的容器上面贴上Sprinng的IOC容器</font>
>
>    > ```java
>    > @Component   //模块化容器注解  配置该注解后该类的Bean就会被创建到Spring容器中,在使用的时候调用即可
>    > ```

一般ioc注解和组件扫描注解配合一起使用

<u>当遇到需要注入的Bean不是自己写的时候 , 还是可以使用<font style="color:red">方法</font>的形式来配置需要交给Spring的容器, 在配置文件中加入返回该Bean对象的方法即可</u> 



###### Spring加载测试类

在Spring的配置文件中 , 没有了xml文件我们如何加载Spring的容器,来实现测试类的功能呢?

```java
@ContextConfiguration(classes = JavaConfig.class)   
//这个注解的作用是用来声明加载Spring配置类用的
//因为没有xml文件 , 所以我们需要一种别的方式来指定测试类如何加载
```



###### <font style="color:red">Spring 的DI注入</font>

> 在完成IOC容器注入的同时 , 如果想进行DI注入的话最好是在需要注入的方法上面定义一个形参,然后通过<font style="color:red">Set的方式完成DI注入</font> , 参数名最好和注入的Bean的id保持一致 , 这种情况也是使用最多的

> 代码示例:

> ```java
> @Configuration //这个注解的作用是声明该`AnnotationConfig`的容器交给Spring来管理
> public class AnnotationConfig {
>     //将xml文件中需要配置的Bean放到这里进行配置
>     @Bean //这个注解的作用是声明该方法创建的类被Spring容器管理
>     public SomeBean someBean(outherBean outherBean) {
>         SomeBean someBean = new SomeBean();
>         someBean.setName(outherBean.getHobby());
>         return someBean;
>     }
>     @Bean
>     public outherBean outherBean() {
>         outherBean o = new outherBean();
>         o.setHobby("玩游戏");
>         return o;
>     }
> }
> ```
>
> test测试
>
> ```java
> @RunWith(SpringJUnit4ClassRunner.class)
> @ContextConfiguration(classes = JavaConfig.class)
> public class JavaConfigTest {
>     @Autowired
>     private SomeBean someBean;
>     @Test
>     public void test() {
>         String s = someBean.toString();
>         System.out.println(s);
>     }
> }
> ```
>
> 运行结果
>
> ![1567077930970](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567077930970.png)



###### 配置文件如何关联配置文件

在以往的时候使用 `mvc.xml` 配置文件的时候需要关联`applicationContext`文件来一起使用

那么抛弃了xml配置的方式我们如何让文件关联起来呢

```java
@Import()  
//该注解的作用是引入配置类 来关联两个配置类
ps:谁导入别人谁是主配置
```

代码示例:

> ```java
> @Configuration
> @Import(JavaConfig.class)
> public class AnnotationConfig {
> //由于关联了另外的配置类,所以我们可以使用别的配置类中的容器对象
> }
> 
> ```
>
> JavaConfig
>
> ```java
> @Configuration
> public class JavaConfig {
>     @Bean
>     public SomeBean someBean() {
>         SomeBean someBean = new SomeBean();
>         someBean.setName("张三");
>         return someBean;
>     }
> }
> ```
>
> 运行结果:
>
> ![1567078493060](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567078493060.png)



###### **引入xml配置文件**

```java
@ImportResource("classpath:applicationContext.xml") //在主配置类中关联XML配置
```





###### Mapper扫描器

用来通知主配置文件扫描Mapper的接口文件

```java
@MapperScan("cn.wolfcode.springbootrbac.mapper")

```

==参数是mapper的路径地址==

![1567927912597](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567927912597.png)





###### 关联属性文件

在以往的时候我们在xml的文件中配置处理器来处理我们的db.properties(数据库连接配置文件)

![1567078730146](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567078730146.png)

现在没有xml我们如何配置属性文件呢?

> 我们可以将属性的文件加载到一个对象中来实现这样的功能

```java
@PropertySource(String path) ---"classpath:db.properties"
//该注解的作用是加载属性文件的信息 将属性文件的信息加载到Spring的环境变量中

```

代码示例:

> ```java
> @Configuration
> @PropertySource("classpath:db.properties") //该注解的作用是加载本地项目中的属性文件的信息
> public class AnnotationConfig {
>     @Value("${jdbc.url}")
>     private String url;
>     @Value("${jdbc.username}")
>     private String username;
>     @Value("${jdbc.password}")
>     private String password;
> 
>     @Bean
>     public DruidDataSource dataSource() {
>         DruidDataSource dc = new DruidDataSource();
>         dc.setUrl(url);
>         dc.setUsername(username);
>         return dc;
>     }
> }
> ```
>
> 测试类:
>
> ```java
> @Configuration
> @PropertySource("classpath:db.properties")
> public class AnnotationConfig {
>     @Value("${jdbc.url}")
>     private String url;
>     @Value("${jdbc.username}")
>     private String username;
>     @Value("${jdbc.password}")
>     private String password;
>     @Bean
>     public DruidDataSource dataSource() {
>         DruidDataSource dc = new DruidDataSource();
>         dc.setUrl(url);
>         dc.setUsername(username);
>         return dc;
>     }
> }
> ```
>
> db.properties
>
> ![1567079744534](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567079744534.png)
>
> 
>
> 运行结果
>
> ![1567079709320](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567079709320.png)

###### 环境的切换

在真实开发中, 测试和开发人员用的不是一个数据库 , 但是如果通过修改db.properties来指定数据库的话未免太过麻烦了 , 所以我们需要给每个部门一套专属的运行环境

设置的步骤

```java
@Profile("dev") 
//该注解的作用是为当前的配置类指定一个运行环境   在配置该注解后,我们需要在`@PropertySource`注解中引用当前定义好了的环境
@PropertySource("classpath:dev.properties")
```

```java
@ActiveProfiles("dev")
//该注解的作用是通知Spring使用什么环境
```

代码示例:

> dev环境下的数据库连接对象
>
> ```java
> @Configuration
> @Profile("dev")
> @PropertySource("classpath:dev.properties")
> public class DevEvnConfig {
> }
> ```
>
> test环境下的数据库连接对象
>
> ```java
> @Configuration
> @Profile("test")
> @PropertySource("classpath:test.properties")
> public class TestEvnConfig {
> }
> ```
>
> 加载这两个配置文件的配置类
>
> ```java
> @Configuration
> @Import({DevEvnConfig.class, TestEvnConfig.class})
> @ToString
> public class DataSourceConfig {
>     @Value("${jdbc.url}")
>     private String url;
>     @Value("${jdbc.username}")
>     private String username;
>     @Bean
>     public DruidDataSource dataSource() {
>         System.out.println(url);
>         System.out.println(username);
>         return null;
>     }
> }
> ```
>
> 测试类
>
> ```java
> @ActiveProfiles("dev")
> @RunWith(SpringJUnit4ClassRunner.class)
> @ContextConfiguration(classes = DataSourceConfig.class)
> public class App {
>     @Autowired
>     private DruidDataSource dataSources;
>     @Test
>     public void test() {
>         System.out.println(dataSources);
>     }
> }
> ```
>
> 运行结果
>
> ![1567081523869](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567081523869.png)

