# FastJson

FastJson是阿里的一款用以将对象转为JSON或者将JSON转为对象的一个插件

特点 : 使用方便 导入依赖即可







对象转为JSON

```java
//该方法可以将任何对象转为Json格式的字符串
JSON.toJSONString()
//如果想要浏览器获取到该json数据则需要用响应写出去
response.getWriter().write(JSON.toJSONString("zhangsan"));
```

JSON转为对象

```java
//此方法用于将JSON转为对象 
/**
	String  text  将什么数据转为对象
	Class.class   要转成什么对象
*/
JSON.parseObject(userInfo, UserInfo.class);
```

