| API                 |                             |                                                          |
| ------------------- | --------------------------- | -------------------------------------------------------- |
| Map<String, Object> | .putIfAbsent(String,Object) | 作用是往Map集合中添加内容 , 不过会判断内容是否重复<br /> |
|                     |                             |                                                          |
|                     |                             |                                                          |

**putIfAbsent**

> **put与putIfAbsent区别:**
>
> put在放入数据时，如果放入数据的key已经存在与Map中，最后放入的数据会覆盖之前存在的数据，
>
> 而putIfAbsent在放入数据时，如果存在重复的key，那么putIfAbsent不会放入值。
>
> 
>
> **putIfAbsent**  如果传入key对应的value已经存在，就返回存在的value，不进行替换。如果不存在，就添加key和value，返回null