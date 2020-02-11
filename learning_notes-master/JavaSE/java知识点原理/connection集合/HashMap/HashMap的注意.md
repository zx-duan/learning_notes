## HashMap底层原理剖析(JDK1.8)

　1、HashMap 存储结构

　2、HashMap 各常量、成员变量作用

　3、HashMap 几种构造方法

　4、HashMap put 及其相关方法

　5、HashMap get 及其相关方法

　6、HashMap remove 及其相关方法

　7、HashMap 扩容方法　resize()



#### HashMap的继承结构

![e0d8e52eef9989fd21925c0a790bb428ec4.jpg](https://oscimg.oschina.net/oscnet/e0d8e52eef9989fd21925c0a790bb428ec4.jpg)



HashMap的存储和特性

HashMap根据**==键==**的==**hashCode值存储数据**== , 大多数情况下可以直接定位到它的值 , 因而具有==很快的访问速度== 但是遍历顺序确实不确定的 , ==HashMap只允许一个记录的键为null== , ====允许多条记录的值为null==  . 



HashMap的线程是**不安全的** , **即任一时刻可以有多个线程访问HashMap** , 可能会导致数据的不一致性 , 如果要满足线程安全 , 可以使用 **Collections** 的==synchronizedMap 方法使HashMap具有线程安全性== , 或者使用==ConcurrentHashMap== (按照字面意思应该是一个线程安全的ConcurrentHashMap类)



一. 存储结构

HashMap的存储结构是 ==链表== + ==红黑树==来实现的 这一特定在jdk1.8之中添加的

**当HashMap的链表长度大于 8 的时候 , HashMap的链表将自动转化为红黑树**

![bd46a79df358dab8a4a796f4feee4c789cb.jpg](https://oscimg.oschina.net/oscnet/bd46a79df358dab8a4a796f4feee4c789cb.jpg)







