# CRM

## FreeMark



#### 什么是Freemark

Freemark是一个模板引擎,一种基于文本模板,用来生成输出文本

#### Freemark能帮我们做什么

*为什么要使用Freemark?*

答: 因为JSP是一个Servlet,页面在响应jsp的时候 , 会编译jsp , 这样会对内存造成浪费 , 而Freemark通过视图解析器直接渲染成一个指定



#### 使用的方法

​	1.加载Freemark的依赖

```xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.23</version>
</dependency>
```

​	2.配置Freemark视图解析器

		> ```xml
		>     	  <!--配置Freemark视图解析器-->
		>     <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
		>         <!-- 是否在model自动把session中的attribute导入进去; -->
		>         <property name="exposeSessionAttributes" value="true"/>
		>         <!-- 配置逻辑视图自动添加的后缀名 -->
		>         <property name="suffix" value=".ftl"/>
		>         <!-- 配置视图的输出HTML的contentType -->
		>         <property name="contentType" value="text/html;charset=UTF-8"/>
		>     </bean>
		>           <!--配置Freemark模板路径-->
		>      <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer"> 
		>      <!-- 配置freemarker的文件编码 -->
		>      <property name="defaultEncoding" value="UTF-8" />
		>      <!-- 配置freemarker寻找模板的路径 -->
		>      <property name="templateLoaderPath" value="/WEB-INF/views/" />
		>      </bean>
		> 
		> ```

​	

#### 使用Freemark需要注意的事项

1.在Freemark中 ==${}== 取到的值不允许有空值如果有空值Freemark是会报错的

​			解决办法: ${ name !}     加一个 !号

#### <font style="color:red">Freemark的常见规范与常用写法</font>

<font style="color:red">list : </font>

```xml
<#list lst as dizhi >
    <b>${dizhi.country}</b> <br/>
</#list>
```



<font style="color:red">if:</font>

```xml
<#if num0 gt 18>  <#--不是使用>，大部分时候，freemarker会把>解释成标签结束！ -->
    及格！
<#else>
    不及格！
</#if>
---------------------------------------------------
if else if else语句测试：
<#if random gte 90>
    优秀！
<#elseif random gte 80>
    良好！
<#else>
    一般！ 
</#if>
```



<font style="color:red">?? 代表了 判断值是否存在</font>



<font style="color:red">时间对象</font>

```xml
root.put("date1", **new** Date());
${date1?string("yyyy-MM-dd HH:mm:ss")}
```





使用Freemark引入外部资源

![引用相对路径的文件地址](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1565188764150.png)

![1565188775955](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1565188775955.png)



## MyBatis分页插件

#### MyBatis分页插件是什么

MyBatis分页插件是MyBatis为我们集成的一款分页的插件



#### MyBatis分页插件的介绍

*MyBatis如何实现的分页插件*

<u>答: MyBatis利用内置的拦截器, 将执行的SQL语句拦截 , 并为我们自动设置SQL语句拼接处理</u>



#### 为什么要使用MyBatis分页插件

由于平时写的分页**使用SQL语句末尾加 LIMIT 来实现分页功能** , 但是LIMIT是MySql专属的语言 , 如果以后要使用性能更好 , 更稳定的数据库 , 那么所有实现分页的地方 都会失效 , 所以 MyBatis来帮我们封装了一个通用的可以在各种数据库中互通的分页插件



> MyBatis是怎么找到我们使用哪些数据库的? 
>
> MyBatis通过 我们配置文件中的 Url 来寻找当前使用的哪一种数据库



#### 如何使用MyBatis分页插件?

1.加载依赖

```xml
     <!--MyBatis分页插件-->
     <dependency>
          <groupId>com.github.pagehelper</groupId>
          <artifactId>pagehelper</artifactId>
           <version>5.1.2</version>
      </dependency>
```



<font style="color:red">2.在MyBatis-config.xml的文件配置MyBatis分页的配置信息</font>

```xml
    <plugins>
        <!-- com.github.pagehelper为PageHelper类所在包名 -->
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!--当pageNum<=0时，将pageNum设置为1-->
            <!--当pageNum>pages时，将pageNum设置为pages-->
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>
```



3.修改Mapper文件

在已经分页的地方将 **sql 语句中的 Limit 去掉**并且把查询总数量的sql语句去掉 , 因为MyBatis分页插件会帮我们完成这个步骤



4.修改Servcie 并修改返回值

在查询数据的地方 也就是Service层 我们需要通知 MyBatis哪些数据需要分页

需要添加   `PageHelper.startPage(qo.getCurrentPage(), qo.getPageSize());` 来通知哪些数据需要分页

将返回值 修改为`PageInfo<T>` 



5.在前端页面配置MyBatis分页插件的信息

```xml
$("#pagination").twbsPagination({
            totalPages: ${pageResult.pages} || 1,
            startPage: ${pageResult.pageNum} ||1,
            visiblePages:5,
            first:"首页",
            prev:"上页",
            next:"下页",
            last:"尾页",
            initiateStartPageClick:false,
            onPageClick:function (event, page) {
            $("#currentPage").val(page);
            $("#searchForm").submit();
        }
})
```







#### 使用MyBatis分页插件需要注意的点

<font style="color:red">当前页 : pageNum     总页数 : pages</font>





## JqueryFrom

#### 什么是JqueryFrom

JqueryFrom是帮我们封装了ajax请求的插件 , 使Ajax请求更方便



#### 如何使用JqueryFrom

1.引入JqueryFrom的js文件

```javascript
<script type="text/javascript" src="/js/plugins/jquery-form/jquery.form.js"></script>
```



#### JqueryFrom能为我们带来什么好处?

用起来更加方便

#### 使用JqueryFrom需要注意哪些地方

JqueryFrom插件中 为我们提供了两种Ajax的提交请求方式 ==(ajaxFrom , ajaxSubmit)== , 其中最大的区别是一种提交方式是在有提交按钮的基础上,而另一种方式是没有按钮时的提交 , 相同的是无论使用哪一种方式 , 最终都会实现异步提交的请求

![ajaxForm和ajaxSubmit区别](C:\Users\Zhangxinuser\Desktop\学习资料\CRM\资料\05_常用插件\jquery-form\ajaxForm和ajaxSubmit区别.png)



JqueryFrom的参数配置情况

![jQuery Form参数配置](C:\Users\Zhangxinuser\Desktop\学习资料\CRM\资料\05_常用插件\jquery-form\jQuery Form参数配置.png)



实际开发应用场景:

```javascript
//当表单不存在提交按钮时候的处理请求 
$('.editbtn').click(function () {
      $('#editForm').ajaxSubmit(function (data) {
         timereload(data);
      })
});

//当表单中存在按钮
$('.editbtn').click(function () {
      $('#editForm').ajaxFrom(function (data) {
         timereload(data);
      })
});
```









## 统一异常处理

#### 为什么要搭配统一异常处理

在EE的开发过程中,不管是对底层的数据库操作过程，还是业务层的处理过程，还是控制层的处理过程，都不可避免会遇到各种可预知的、不可预知的异常需要处理。
每个过程都单独处理异常，系统的代码耦合度高，工作量大且不好统一，维护的工作量也很大。 
那么，能不能将所有类型的异常处理从各处理过程解耦出来，这样既保证了相关处理过程的功能较单一，也实现了异常信息的统一处理和维护？答案是肯定的。下面将介绍使用Spring MVC统一处理异常的解决和实现过程。 



#### 如何实现统一异常处理

>  有三种实现方式

1.使用SpringMvc提供的简单异常处理器 SimpleMappingExceptionResolver              <font style="color:red"> (应用较少)</font>

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <!-- 定义默认的异常处理页面，当该异常类型的注册时使用 -->
        <property name="defaultErrorView" value="common/error"/>
        <!-- 定义异常处理页面用来获取异常信息的变量名，默认名为exception -->
        <property name="exceptionAttribute" value="ex"/>
        <!-- 定义需要特殊处理的异常，用类名或完全路径名作为key，异常也页名作为值 -->
        <property name="exceptionMappings">
            <value>
                org.apache.shiro.authz.UnauthorizedException=common/nopermission
                <!-- 这里还可以继续扩展不同异常类型的异常处理 -->
            </value>
        </property>
</bean>
```

<font style="color:red">2.使用接口来动态的为Cortoller进行增强</font>       以后在SpringBoot会用

```java
@ControllerAdvice
public class CRMExceptionHandler {
    @ExceptionHandler(RuntimeException.class)
    public String handleException(Exception ex, Model model, HandlerMethod hmethod, HttpServletResponse response) throws ServletException, IOException {
        //针对异步请求作出的异常处理 ,利用json将数据写到客户端
        if (hmethod.getMethod().isAnnotationPresent(ResponseBody.class)) {
            //设置编码格式
            response.setContentType("application/json;charset=UTF-8");
            JsonResult json = new JsonResult();
            json.mark("操作失败");
            //将json响应到页面
            response.getWriter().write(JSON.toJSONString(json));
            return null;
        } else {
            model.addAttribute("ex", ex);
            //将错误页面和错误信息展示到页面上
            return "common/error";
        }
    }
}
```

*该类的应用思想就是利用动态代理, 动态的用**ConTollerAdvice(控制器增强)**的思想来对控制器发出的异常进行捕捉将捕捉到的不同的异常进行分批处理*

#### 小细节

> ```java
> //负责检测异常的类型
> @ExceptionHandler(RuntimeException.class)
> ```
>
> > 当检测到异常的时候,会通过过滤器被抛出到这里 , 经过上面的注解检测,检测到异常后寻找该异常是什么类型在方法中做相对的处理





3.实现Spring的异常处理接口HandlerExceptionResolver 自定义自己的异常处理器；

猜测: 需要配置文件中扫描到自定义的异常处理器 并且要重写异常处理器中的异常处理方法



## 页面时间按钮插件

1. 如何使用时间插件

   引入文件![1565669297342](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1565669297342.png)

   加载时间插件js

   在接收参数的地方贴上`@DateFormart`时间标签指定日期格式

