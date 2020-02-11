# VO

一种模仿真实类型而诞生的方便操作的工具类 , 大多数用来处理我们封装数据的问题



> 代码实现
>
> ```java
> @Getter
> @Setter
> @ToString
> public class StrategyStatisVO implements Serializable {
> 
> 
>     //2：排行数据查询
>     private boolean isabroad;
>     private Long destId;
>     private String destName;
>     private String title;
> 
>     //1：数据统计(回复，点赞，收藏，分享，查看)
>     private Long strategyId;  //攻略id
>     private int viewnum;  //点击数
>     private int replynum;  //攻略评论数
>     private int favornum; //收藏数
>     private int sharenum; //分享数
>     private int thumbsupnum; //点赞个数
> }
> 
> ```
>
> 值得注意的是 ,如果要进行 初始化 / 数据落地等关系到数据库的操作时 , 建议还是使用封装的Domain来进行操作比较好