##### 1）启动报错： 

###### 没有权限问题

```
$/usr/local/elasticsearch/bin/elasticsearch -dJava HotSpot(TM) 64-Bit Server VM warning: Cannot open file logs/gc.log due to Permission denied
```

这是因为elasticsearch的logs目录下的一个日志文件没有权限所致，这个日志文件不知ES的访问、错误配置，而是在ES的安装目录下的一个logs目录，里面的gc.log没有权限所致，默认应该是“-rw-rw-r--”，所以解决问题最简单的方法是删除了，然后重启es集群即可，这个日志文件ES会自动创建。



###### 该问题是es启动是没有找到log4j日志输出端的类

2019-03-01 21:52:02,176 main ERROR Unable to locate appender "rolling" for logger config "root

网上答案：检查log4j2.properties文件中属性配置是否正确，末尾不能有空格（笔者检查了半天，并且试着替换了该文件，替换了jar包，还是没有解决问题）
后来通过查找最后两行报错，发现是用root用户登录启动es之后查看日志，导致elasticsearch.log的属主变成了root：
![es启动日志异常](ES%E5%BC%82%E5%B8%B8%E6%80%BB%E7%BB%93.assets/20190302110819501.png)

