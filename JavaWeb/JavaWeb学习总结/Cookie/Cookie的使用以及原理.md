## Cookie

一种浏览器与服务器直接特殊通讯的手段,严格的遵循了HTTP协议的通讯规范

通过new对象的方式创建Cookie

Cookie是一种轻量级的会话存储存在用户的客户端之中

Cookie的缺陷是不安全,<font style="color:red">存储中文需要指定中文的格式很麻烦</font>,容量小.所以使用更安全容量大的Session用于浏览器与客户端直接会话的数据传输



**先来了解几个概念。**

#### 1、无状态的HTTP协议：

协议是指计算机通信网络中两台计算机之间进行通信所必须共同遵守的规定或规则，超文本传输协议(HTTP)是一种通信协议，它允许将超文本标记语言(HTML)文档从Web服务器

传送到客户端的浏览器。

HTTP协议是无状态的协议。一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接。这就意味着服务器无法从连接上跟踪会话。

#### 2、会话（Session）跟踪：

会话，指用户登录网站后的一系列动作，比如浏览商品添加到购物车并购买。会话（Session）跟踪是Web程序中常用的技术，用来跟踪用户的整个会话。常用的会话跟踪技术

是Cookie与Session。Cookie通过在客户端记录信息确定用户身份，Session通过在服务器端记录信息确定用户身份。

### **二. Cookie**

由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。用户A购买了一件商品放入购物车内，当再次购买商品时服务器已经无法判断该购买行为是属于用户A的

会话还是用户B的会话了。怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie

的工作原理。

Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端会把Cookie保存起来。

当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

1、cookie的内容主要包括：名字，值，过期时间，路径和域。路径与域一起构成cookie的作用范围。

1）Name 和 Value 属性由程序设定,默认值都是空引用。

2）Domain属性的默认值为当前URL的域名部分，不管发出这个cookie的页面在哪个目录下的。

3）Path属性的默认值是根目录，即 ”/” ，不管发出这个cookie的页面在哪个目录下的。可以由程序设置为一定的路径来进一步限制此cookie的作用范围。

4）Expires 属性，这个属性设置此Cookie 的过期日期和时间。

[?](https://www.jb51.net/article/128745.htm#)

```
`HttpCookie cookie = new HttpCookie("MyCook");//初使化并设置Cookie的名称``DateTime dt = DateTime.Now;``TimeSpan ts = new TimeSpan(0, 0, 1, 0, 0);//过期时间为1分钟``cookie.Expires = dt.Add(ts);//设置过期时间``cookie.Values.Add("userid", "value");``cookie.Values.Add("userid2", "value2");``Response.AppendCookie(cookie);`
```

### 2、Path和Domain属性

--path:　　

如果http://www.china.com/test/index.html 建立了一个cookie，那么在http://www.china.com/test/目录里的所有页面，以及该目录下面任何子目录里

的页面都可以访问这个cookie。这就是说，在http://www.china.com/test/test2/test3 里的任何页面都可以访问http://www.china.com/test/index.html

建立的cookie。但是，如果http://www.china.com/test/ 需要访问http://www.china.com/test/index.html设置的cookes，该怎么办？

这时，我们要把cookies的path属性设置成“/”。在指定路径的时候，凡是来自同一服务器，URL里有相同路径的所有WEB页面都可以共享cookies。

--Domain:

比如： http://www.baidu.com/xxx/login.aspx 页面中发出一个cookie，Domain属性缺省就是www.baidu.com ，可以由程序设置此属性为需要的值。　　

值是域名，比如www.china.com。这是对path路径属性的一个延伸。如果我们想让 www.china.com能够访问bbs.china.com设置的cookies，该怎么办? 我们可以把

domain属性设置成“china.com”， 并把path属性设置成“/”。

3、==会话Cookie==和持久Cookie

若不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就消失。这种生命期为浏览器会话期的cookie被称为会话cookie。会话cookie一般不存储在

硬盘上而是保存在内存里，当然这种行为并不是规范规定的。

若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。存储在硬盘上的cookie可以在浏览器的不同进程间共享。

这种称为持久Cookie。

4、Cookie具有不可跨域名性

就是说，浏览器访问百度不会带上谷歌的cookie。

### **三. Session**

Session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录

在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

每个用户访问服务器都会建立一个session，那服务器是怎么标识用户的唯一身份呢？事实上，用户与服务器建立连接的同时，服务器会自动为其分配一个SessionId。

1、两个问题：

1）什么东西可以让你每次请求都把SessionId自动带到服务器呢？显然就是cookie了，如果你想为用户建立一次会话，可以在用户授权成功时给他一个唯一的cookie。当一个

用户提交了表单时，浏览器会将用户的SessionId自动附加在HTTP头信息中，（这是浏览器的自动功能，用户不会察觉到），当服务器处理完这个表单后，将结果返回给SessionId

所对应的用户。试想，如果没有 SessionId，当有两个用户同时进行注册时，服务器怎样才能知道到底是哪个用户提交了哪个表单呢。

2）储存需要的信息。服务器通过SessionId作为key，读写到对应的value，这就达到了保持会话信息的目的。

2、session的创建：

当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了sessionId，如果已包含则说明以前已经为此客户端创建过session，服务

器就按照sessionId把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含sessionId，则为此客户端创建一个session并且生成一个与此session相关

联的sessionId，sessionId的值是一个既不会重复，又不容易被找到规律以仿造的字符串，这个sessionId将被在本次响应中返回给客户端保存。

3、禁用cookie：

如果客户端禁用了cookie，通常有两种方法实现session而不依赖cookie。

1）URL重写，就是把sessionId直接附加在URL路径的后面。

2）表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。比如：

```xml
`<``form` `name``=``"testform"` `action``=``"/xxx"``> ``<``input` `type``=``"hidden"` `name``=``"jsessionid"` `value``=``"ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764"``> ``<``input` `type``=``"text"``> ``</``form``>`
```

4、Session共享：

对于多网站(同一父域不同子域)单服务器，我们需要解决的就是来自不同网站之间SessionId的共享。由于域名不同(aaa.test.com和bbb.test.com)，而SessionId又分别储存

在各自的cookie中，因此服务器会认为对于两个子站的访问,是来自不同的会话。解决的方法是通过修改cookies的域名为父域名达到cookie共享的目的,从而实现SessionId的共

享。带来的弊端就是，子站间的cookie信息也同时被共享了。

### **四. 总结**

1、cookie数据存放在客户的浏览器上，session数据放在服务器上。

2、cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，考虑到安全应当使用session。

3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie。

4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

5、可以考虑将登陆信息等重要信息存放为session，其他信息如果需要保留，可以放在cookie中。



### Cookie面试内容

# 默认SESSION配置

在**默认的**JSP、PHP配置中，SessionID是需要存储在Cookie中的，默认Cookie名为：

- PHPSESSIONID
- JSESSIONID

以下以PHP为例：

1. 你第一次访问网站时，
2. 服务端脚本中开启了Session`session_start();`，
3. 服务器会生成一个不重复的 SESSIONID 的文件`session_id();`，比如在`/var/lib/php/session`目录
4. 并将返回(Response)如下的HTTP头 `Set-Cookie:PHPSESSIONID=xxxxxxx`
5. 客户端接收到`Set-Cookie`的头，将`PHPSESSIONID`写入cookie
6. 当你第二次访问页面时，所有Cookie会附带的请求头(Request)发送给服务器端
7. 服务器识别`PHPSESSIONID`这个cookie，然后去session目录查找对应session文件，
8. 找到这个session文件后，检查是否过期，如果没有过期，去读取Session文件中的配置；如果已经过期，清空其中的配置

如果客户端禁用了Cookie，那`PHPSESSIONID`都无法写入客户端，Session还能用？

答案显而易见：**不能**

> 并且服务端因为没有得到`PHPSESSIONID`的cookie，会不停的生成`session_id`文件

# 取巧传递session_id

但是这难不倒服务端程序，聪明的程序员想到，如果一个Cookie都没接收到，基本上可以预判客户端禁用了Cookie，那将session_id附带在每个网址后面(包括POST)，
比如：

```xml
GET http://www.xx.com/index.php?session_id=xxxxx
POST http://www.xx.com/post.php?session_id=xxxxx
```

然后在每个页面的开头使用`session_id($_GET['session_id'])`，来强制指定当前`session_id`

这样，答案就变成了：**能**

> 聪明的你肯定想到，那将这个网站发送给别人，那么他将会以你的身份登录并做所有的事情
> （目前很多订阅公众号就将openid附带在网址后面，这是同样的漏洞）。
>
> 其实不仅仅如此，cookie也可以被盗用，比如XSS注入，通过XSS漏洞获取大量的Cookie，也就是控制了大量的用户，腾讯有专门的XSS漏洞扫描机制，因为大量的QQ盗用，发广告就是因为XSS漏洞
>
> 所以Laravel等框架中，内部实现了Session的所有逻辑，并将`PHPSESSIONID`设置为`httponly`并加密，这样，前端JS就无法读取和修改这些敏感信息，降低了被盗用的风险。

# Cookie在现代

禁用Cookie是 IE6 那个年代的事情，现在的网站都非常的依赖Cookie，禁用Cookie会造成大量的麻烦。

> 在Flash还流行的年代，Flash在提交数据会经常出现用户无法找到的情况，其实是因为Flash在IE下是独立的程序，无法得到IE下的Cookie。
> 所以在Flash的`flash_var`中，一般都会指定当前的`session_id`，让Flash提交数据的时候，将这个`session_id`附带着提交过去
> Chrome中使用 Flash沙箱 已经解决了cookie的问题，但是为了兼容IE，比如`swfupload`等flash程序都要求开发者附带一个`session_id`

