步骤: 

1.新建一个虚拟机用来当做主机器 设置虚拟机的ip为静态ip

![1572681910226](C:\Users\Zhangxinuser\Desktop\新的学习总结\imgs\1572681910226.png)



2.安装mysql 修改mysql的配置文件

```java
//将mysql的配置文件改名 , 删除也可以
mv /var/lib/mysql/auto.cnf /var/lib/mysql/auto.cnf_bak 
```



配置master的配置文件

2.1，在主服务器/etc/my.cnf 中添加：

```java
server-id=1    //给数据库服务的唯一标识，一般为大家设置服务器Ip的末尾号
log-bin=master-bin
log-bin-index=master-bin.index
```



3.创建一个从服务器用来关联主服务器

步骤同 : 1 , 2 

在从服务器的配置文件中添加 :

```java
	change master to master_host='127.0.0.1', //Master 服务器Ip
	master_user='root',
	master_password='admin',
	master_log_file='master-bin.000001',//Master服务器产生的日志
	master_log_pos=0;
```



3.1 查看服务的状态

```java
show slave status \G
```





