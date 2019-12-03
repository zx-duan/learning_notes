# Shiro的使用

### Shiro是什么?

**Shiro是一款安全登录验证,用户权限管理的一款适用性强的插件**

shiro的执行是怎样的,如何过滤是否登录, 如何过滤动态资源



### 如何使用Shiro

在javaEE中使用Shiro需要在配置文件中加入一个`.xml文件`在文件中需要配置

1. 引入.xml文件

2. 在xml文件中需要配置安全管理器

   ```xml
   <bean class="org.apache.shiro.web.mgt.DefaultWebSecurityManager" id="securityManager">    <property name="realm" ref="crmRealm"></property></bean>
   ```

3. 还需要配置Shiro内置的过滤器

   ```xml
   <bean id="shiroFilter"      class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">    <!--引用指定的安全管理器-->     
   <property name="securityManager" ref="securityManager"/>    
       <property name="loginUrl" value="/login.html"/>    
       <property name="filterChainDefinitions">  
           <!--配置当前过滤的地址   anon是不过滤,authc是过滤-->
           <value>           
               /js/**=anon            
               /static/**=anon            
               /**=authc        
           </value>    
       </property>    
       
       <property name="filters">        <map>           
           <entry key="authc" value-ref="crmTormAuthenticationFilter"></entry>        </map>    
       </property>
   </bean>
   ```

   

   `还需要设置数据源集成授权的数据源`(集成Shiro内部的数据源让他重写)

   ![1562456663532](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1562456663532.png)

   

     需要设置响应回来的数据做什么这时候需要需要自己创建一个过滤响应器用来响应成功或者失败分别做什么

   ​	![1562456985542](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1562456985542.png)