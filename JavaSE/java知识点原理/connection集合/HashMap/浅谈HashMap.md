# 浅谈 HashMap 底层实现原理

学习方向

*<font style="color:blue">HashMap简单概述</font>*

*<font style="color:blue">HashMap的数据结构 (古→今)</font>*

*<font style="color:blue">HashMap的扩容机制</font>*

*<font style="color:blue">HashMap线程安全问题解决方案</font>*





## 一、HashMap 概述 

#### 		1.1Hash 

​		Hash 是一种存储信息的方式，任意长度的字符串作为 key，通过定义一个 Hash 运算，将 key 转换为另一个特定的 字符串。相比于常规的储存方式，Hash 最主要的优点就是 能更快速地定位到数据的位置并提取数据。除此之外，运用 Hash存储数据时，除了key value外，无需存储其它结构数据， 有利于节约内存，同时提高稳定性。这些特性使得 Hash 被 广泛应用在计算机领域



#### 		1.2 Map 

Map是一种单向的映射，即映射表。Map用于保存具 有 key-value 映射关系的数据。对于一个 key，有且仅有一个 value 与之对应。



### 		1.3 常用 Hash 算法 

#### 		1.3.1 MD5 

​		MD5 是计算机安全领域广泛使用的一种散列函数，用 以提供消息的完整性保护。它是一种不可逆加密算法。对于 不同数据，MD5 的结果完全不一样，只有完全相同的数据才 会得出同样的 MD5 值。一开始，MD5 被应用在加密领域， 比如在网络开发中，不允许传输用户的明文隐私数据，使用 MD5 时，数据库仅仅保存真实密码经过 MD5 进行加密运算 后的加密密码。由于Hash的特点，对于同一个数据进行运算， 得到的结果一致，验证密码是否真实只需比较 MD5 运算的 值是否一致即可。后来，由于 MD5 可以通过散列碰撞破解， 在加密领域就很少用到 MD5 了，但 MD5 在其他领域仍有应用





#### 		1.3.2 SHA-1 

​		SHA-1 也被称为安全哈希算法，适用于数字签名标准 里面定义的数字签名算法。SHA-1 也是不可逆算法。输入任 意长度小于 2 的 64 次方的字符串，产生长度为 160bit 的散 列值。这种算法常常用于文件完整性校验。通过比较文件的 Hash 值判断文件在传输过程中是否损坏或遭到篡改。MD5 与 SHA-1 有许多相似之处，但由于 SHA-1 摘要更长，相比 于 MD5 更加安全，而 MD5 的运算速度更快。SHA-1 经过多 年来的完善和发展，越来越先进，被广泛应用于计算机的各 个领域。[2]



## 二、HashMap 的原理 

#### 		2.1 实现原理 

HashMap 是一种线性表的数据结构。通过数组与链表来 存储 Key-Value 键值对。==HashMap 的主干由数组构成，数组 的初始值为 null，键值对可以储存在这些数组当中==。数组是 有序的元素序列，**使用数组时，由于数组的特点，增添或删 去一个元素都可能需要移动大量的元素**，所以一般情况下， 数组的元素个数是不会改变的。而链表则用于解决运行中可 能发生的 index 冲突。链表与数组不同，==链表是一种物理存 储单元上非连续==、==非顺序的存储结构==。在链表中，**各个元素 间通过链表中的指针连接，增添或删去一个元素时只需要改 变指针的连接次序即可**，**无需移动其他数据。** 使用 HashMap 时，对输入的 key 值进行 Hash 运算，求 出相应的存储位置。当需要提取数据时，只需要对 key 值再进行一次 Hash 运算得出相应存储位置，即可定位并提取数 据，值得一提的是，与 Hashtable 不同，Hashmap 的 key 与 value 都可以为 null。

​		由于 HashMap 的长度有限，难免出现多个输入值对应 同一个index的情况，这便是index冲突，如果选则用创建 新的数组来解决 index 冲突则程序会变得十分复杂，这样一 来，运行效率将大幅下降，所以，HashMap 选择通过插入链 表来解决 index 冲突。HashMap 的许多设计都是从提高机器 运算效率的角度考虑的。比如发明者考虑到人们的使用习惯， 认为后插入的键值对被用到的概率较高，在插入键值对时使 用头插法，后插入的键值对通常放在链表的前端。除此之外， HashMap 使用对电脑来说更加简便的位运算而不是取模运算 来提高效率。为了减少冲突，index的分布通常要尽可能均匀。 由于位运算的特点，要使 Hash 运算更为均匀，HashMap 的 长度通常为 2 的整数幂。[3] 



#### 		2.2 常用方法 

​		**Put 方法是 HashMap 存储数据的方法**。当要存入一个键 值对时，首先判断 key 是否为 null，如果 key 为 null，则调 用 putForNullKey 方法，对 key 为 null 的 value 值进行更新。 当 key 不为 null 时，检查 key 是否已经在 HashMap 中存在。 如果发现输入的 key 已经存在，则用新的 value 替换原来的 value，如果不存在，则通过对 key 进行 Hash 运算确定插入 位置，也就是相对应的数组，最后将键值对存储到该数组。 当发生 index 冲突时，在同一个数组里创建链表，运用`头插法`将新的键值对插入到链表里。 



#### 		2.2.2 get 方法 

​		Get 方法是 HashMap 获取 key 对应的 value 的方法。 与put方法一样，首先要判断key是否为null。如果key为 null，直接提取 null 相对应的 entryFroNullkey 里的 value 值。 在 key 不为 null 使，根据 key 计算对应的 hash 值，然后根 据 Hash 与容器大小得到对应的 index。如果该 index 中仅有 一个键值对，直接获取相应的 value。但由于 index 冲突，可 能在同一个位置查找到多个不同的键值对，这时候，就要按 照链表的顺序往下查找，直到找到具有相同 key 的键值对， 再提取相应的 value 即可。如果 key 不存在，则输出的值为 null。 



#### 		2.2.3 remove 方法 

​		Remove 方 法 是 Hashmap 删 除 数 据 的 方 法。 判 断 key 是否为 null，如果 key 为 null，则删除 null 相对应的 entryFroNullkey 里的 value 值，如果 key 不为 null，则删除 key 所对应的 value。键值对被删除后，若再对该 key 进行 get 方法，则会输出 null。 

#### 		2.2.4 位运算 

​		==为了提高 HashMap 的运算效率==，HashMap 的发明者创 造了位运算。位运算是一种直接对二进制按位进行运算的运算方式。相比于传统的取模运算，位运算虽然在编程上比 较复杂，但对计算机而言，计算更加简便，从而大大提高 HashMap 的运行效率。[4] 



#### 		2.2.5 resize(扩容) 

​		**Resize 是 HashMap 的扩容机制**。由于 HasMap 的容量 有限<font style="color:red"> (据说好像默认的长度为16)</font>，在**==插入了很多数据后，发生index冲突的概率会上 升==**==，get 可能需要查找的链表会更长，这会降低它的运行效率==。**<font style="color:red">HashMap 的负载因子默认是 0.75</font>**，使用者可以根据实际 应用情况来自行调整，设置一个合理的负载因子有利于提 高 HashMap 的工作效率。Resize 方法是否进行由 HashMap 当前总容量和当前容量与负载因子共同决定。==当HashMap 存储的数据量达到负载因子与 HashMap 当前总容量的乘积 时，就会进行resize==。Resize分为两个步骤，分别是**扩容**和 **rehashing**。

Resize 的第一步是扩容。当满足 HashMap.Size >= Capacity(**总容量**) * LoadFactor(**负载因子**) 时， 第二步是 rehashing。<font style="color:red">`扩容的公式计算 HashMap.Size >= Capacity(总容量) * LoadFactor(负载因子)`</font>在扩容之后，hash 的规则也要进行 改变。将原数组存储的键值对按照新的 hash 运算方式得到其 新的存储位置，然后重新储存到新的数组里。`在jdk1.7版本的时候扩容在多线程的环境下会引发死循环问题`在长度扩大后， 如果仍用原来的 hash 运算方式，新的数组得不到充分利用， 扩容就失去了意义，更重要的是，需要使用新的 hash 算法 对原有的键值对重新分布。HashMap 是非线程安全的，多线 程 put 后可能导致 get 死循环。在多线程环境中进行 resize 可 能会让 HashMap 出现有环链表，在进行 get 方法时程序就会 陷入死循环。 

> > ```java
> > /**
> > 
> > - Transfers all entries from current table to newTable.
> >   */
> >   void transfer(Entry[] newTable) {
> >    Entry[] src = table;
> >    int newCapacity = newTable.length;
> >    for (int j = 0; j < src.length; j++) {
> >    Entry<K,V> e = src[j];
> >    if (e != null) {
> >        src[j] = null;
> >        do {
> >            // B线程执行到这里之后就暂停了
> >            Entry<K,V> next = e.next;
> >            int i = indexFor(e.hash, newCapacity);
> >            e.next = newTable[i];
> >            newTable[i] = e;
> >            e = next;
> >        } while (e != null);
> >    }
> >    }
> >   }
> > 
> > ```
>
> 假设有两个线程A和B都在执行这一段代码，数组大小由2扩容到4,
>
> 在扩容前 `tab[1]=1-->5-->9。`
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9pYWozVHU3OGliSWdjMnBTM09PRVZwZVlXYTdTSVdMWlY2MHV0eXJYZVBiV0ZycFhZcXVjQ1pZNGw2aldKM1F6SkRNSmdBSmdsWFE4OFNMcmlhMU1rRTF3dy82NDA_d3hfZm10PXBuZyZ3eGZyb209NSZ3eF9sYXp5PTEmd3hfY289MQ)
>
> 当B线程执行到 next = e.next时让出时间片,A线程执行完整段代码但是还没有将内部的table设置为新的newTable时，线程B继续执行。
>
> 此时A线程执行完成之后，挂载在`tab[1]的元素是9-->5-->1,`注意这里的顺序被颠倒了。
>
> 此时`e = 1, next = 5;`
>
> tab[i]按照循环次数变更顺序
>
> 1. tab[i]=1
> 2. tab[i]=5-->1
> 3. tab[i]=9-->5-->1
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9pYWozVHU3OGliSWdjMnBTM09PRVZwZVlXYTdTSVdMWlY2ZVdoaWNEdjViWU42aldVQ0lSbXJ6aWJobmRXaWM4S1puZFlOOEQ5SVJpYnlkaENmazNtckNkU21RZy82NDA_d3hfZm10PXBuZyZ3eGZyb209NSZ3eF9sYXp5PTEmd3hfY289MQ)
>
> 
>
> 线程A执行完成后
>
> 同样B线程我们也按照循环次数来分析
>
> 第一次循环执行完成后,newTable[i]=1, e = 5
>
> 第二次循环完成后: newTable[i]=5-->1, e = 1。
>
> 第三次循环,e没有next,所以next指向null。当执行e.next = newTable[i](1-->5)的时候,就形成了 1-->5-->1的环,再执行newTable[i]=e,此时newTable[i] = 1-->5-->1。
>
> 当在数组该位置get寻找对应的key的时候，就发生了死循环,引起CPU 100%问题。
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9pYWozVHU3OGliSWdjMnBTM09PRVZwZVlXYTdTSVdMWlY2ZURMTk1qanNtbTM4b1plaWJOM1I1RXhMQXBpY01jZW53aWMxZ2Ria1hJem9FTGJ1WUZuWHB1dld3LzY0MD93eF9mbXQ9cG5nJnd4ZnJvbT01Jnd4X2xhenk9MSZ3eF9jbz0x)

线程B执行扩容过程

而**JDK8就不会出现这个问题**,它在这里就有一个优化，它使用了两个指针来分别指向头节点和尾节点，而且还保证了元素原本的顺序。


==当然HashMap仍然是不安全的,所以在多线程并发条件下推荐使用ConcurrentHashMap==。









#### 		2.2.6 红黑树 

​		==Hashmap 中若某数组元素的链表过长，在进行 get 方法 时相当于只用链表存储数据，这样一来 hashmap 就会失去其 优势==。在 JDK 1.8 后引入了红黑树结构。==若链表长度过长， 就把链表转化为红黑树==。红黑树运用了二分查找的思维模式， 是一种特殊的二叉树。在插入数据时，运用旋转和变色的方法，保证了红黑树特有的规则不被打破。红黑树的特性能确 保从根到叶子的最长路径不超过最短路径的两倍，从而实现 了数据的快速提取。 

**HashMap的数据结构 (古→今)**

![1571147729617](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1571147729617.png)

Hash表是一个数组+链表的结构，这种结构能够保证在遍历与增删的过程中，如果不产生hash碰撞，仅需一次定位就可完成，时间复杂度能保证在O(1)。  在jdk1.7中，只是单纯的数组+链表的结构，但是如果散列表中的hash碰撞过多时，会造成效率的降低，所以在JKD1.8中对这种情况进行了控制，当一个hash值上的链表长度大于8时，该节点上的数据就不再以链表进行存储，而是转成了一个红黑树.







## 三、HashMap 的优点 

#### 		3.1 适合大量数据 

​		Hashmap 运用 resize 方法进行扩容，每次扩容都可以将 容量变为原来的两倍。当前的容量越大，扩容所增加的容量 也越大。也就是说，扩容的次数越多，扩容的频率就越低。 这样一来，在处理大量数据时也能保证较高的效率。数组与 链表的有机结合，使得 Hashmap 相比于仅仅使用数组的存储 方式在储存数据时增添、删改数据更为方便。 



#### 		3.2 提取速度

​		Hashmap 根据 hashcode 的值直接算出 index，数组越多 查找效率就越高。相比于单链表，Hashmap 需要查找的范围 更小，效率更高





## 四. HashMap的缺点与解决方案

##### HashMap线程不是安全的

在通常的情况下 , HashMap在使用的过程中是线程不安全的 , 即一个时刻可以有多个线程访问HashMap , 这样做可能会导致**数据的不一致性** , **多线程put操作后，get操作导致死循环。** **多线程put非NULL元素后，get操作得到NULL值。** **多线程put操作，导致元素丢失。**

[详情](https://www.cnblogs.com/aspirant/p/11450839.html)

解决线程不安全的方法目前我知道的有两种

**1. 通过使用Collections的synchronizedMap方法使HashMap具有线程的安全性**



**2.或者使用ConcurrentHashMap来创建一个安全线程的Map集合**

`通常情况下我们推荐这种解决方式`







可以使用

HashMap