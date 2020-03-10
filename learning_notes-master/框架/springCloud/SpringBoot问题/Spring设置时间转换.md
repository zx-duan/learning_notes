1.当用new date()或者TimeZone时有8小时的时差问题：   

保证服务器时区为东八区时间即北京时间
服务启动的时候，将当前时区设置为GMT+8，代码如下：

```java
@SpringBootApplication
    public class Application {
      @PostConstruct
      void started() {
      TimeZone.setDefault(TimeZone.getTimeZone("GMT+8"));
//TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
//TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
}

public static void main(String[] args) { SpringApplication.run(Application.class, args); } }
```

这样就保证了Java程序的时区为北京东八区时间。

2.在application.yml中配置：

```yaml
spring:
    #解决前端取回日期少8个小时问题
    jackson:
        date-format: yyyy/MM/dd HH:mm:ss
        time-zone: GMT+8
    datasource:
        #基本属性
        url: jdbc:mysql://localhost:3306/gwork?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT%2b8
```

3.数据库驱动连接上配置：

url: jdbc:mysql://localhost:3306/gwork?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT%2b8

注:采用+8:00格式,没有指定MySQL驱动版本的情况下它自动依赖的驱动高版本的mysql，这是由于数据库和系统时区差异所造成的，mysql默认的是美国的时区，而我们中国大陆要比他们迟8小时,在jdbc连接的url后面加上serverTimezone=GMT即可解决问题，如果需要使用gmt+8时区，需要写成GMT%2B8，否则会被解析为空。再一个解决办法就是使用低版本的MySQL jdbc驱动不会存在时区的问题。

这个时区要设置好，不然会出现时差， 
如果你设置serverTimezone=UTC，连接不报错， 
但是我们在用java代码插入到数据库时间的时候却出现了问题。 
比如在java代码里面插入的时间为：2018-06-24 17:29:56 
但是在数据库里面显示的时间却为：2018-06-24 09:29:56 
有了8个小时的时差 
UTC代表的是全球标准时间 ，但是我们使用的时间是北京时区也就是东八区，领先UTC八个小时。

以上三个均设置，彻底解决时区问题。