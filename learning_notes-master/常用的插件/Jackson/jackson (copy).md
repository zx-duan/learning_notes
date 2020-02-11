# JackSon

一种将对象或者字符串相互转换的方式



#### JackSon中将字符串转为对象的方式

```java
@ResponseBody 
//该注解的作用就是将对象转换为json的格式字符串
```



#### JackSon中将字符串转为对象的方式

```java
 JSON.parseObject(student, Student.class);
```

#### 注解

`@JsonIgnoreProperties(ignoreUnknown = true)`

> 将这个注解写在类上之后，就会忽略类中不存在的字段。这个注解还可以指定要忽略的字段，例如@JsonIgnoreProperties({ “password”, “secretKey” })

[详情](https://blog.csdn.net/russle/article/details/84147236 详情)

