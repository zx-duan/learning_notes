# RowBounds

`分页`

1. RowBounds是什么

2. RowBounds怎么使用

   





### 一、RowBounds是什么

RowBounds是MyBatis的<font style="color:red">`物理分页`</font> 如果要知道什么是物理分页 , 我们需要先了解这么一个概念

##### **(一)、物理分页与逻辑分页的区别**

> **物理分页**: 直接从数据库中拿出我们需要的数据，例如在Mysql中使用limit。

> **逻辑分页:**从数据库中拿出所有符合要求的数据，然后再从这些数据中拿到我们需要的分页数据。

##### **(二)、两者之间的优缺点**

> 物理分页每次都需要访问数据库 , 逻辑分页只需要访问一次
>
> 物理分页占用内存少, 而逻辑分页占用的内存比较多
>
> 物理分页每次获取的数据都是最新的, 逻辑分页获取的数据可能滞后

### 二、RowBounds的使用方式

##### (一)、一般的使用方法

```java
//定义个通过分页条件查询的接口
public List<Order> queryListByPage(RowBounds rowBounds);

//创建RowBounds对象来进行分页
dao.queryListPage(new RowBounds(offset,limit));
```

```java
/**
 * RowBounds
 * MyBatis的分页辅助类
 */
public class RowBounds {
	
  /** 默认的offset是0 */  
  public static final int NO_ROW_OFFSET = 0;
  /** 默认limit是int的最大值 */  
  public static final int NO_ROW_LIMIT = Integer.MAX_VALUE;
  public static final RowBounds DEFAULT = new RowBounds();

  private final int offset;
  private final int limit;

  public RowBounds() {
    this.offset = NO_ROW_OFFSET;
    this.limit = NO_ROW_LIMIT;
  }

  public RowBounds(int offset, int limit) {
    this.offset = offset;
    this.limit = limit;
  }

  public int getOffset() {
    return offset;
  }

  public int getLimit() {
    return limit;
  }

}
```

| 参数名称(parameterName) | 备注(remark)   | 类型(Type) |
| ----------------------- | -------------- | ---------- |
| **offset**              | 起始行数       | int        |
| **limit**               | 需要的数据行数 | int        |
|                         |                |            |

> RowBounds对象有2个属性，offset和limit。
>
> offset:起始行数
>
> limit：需要的数据行数
>
> 因此，取出来的数据就是：从第<font style="color:red">offset+1</font>行开始，取limit行
>
> Mybatis中使用RowBounds实现分页的大体思路：
>
> 先取出所有数据，然后游标移动到offset位置，循环取limit条数据，然后把剩下的数据舍弃。



##### (二)、Mybatis中使用RowBounds实现分页的大体思路：

> 先取出所有数据，然后游标移动到offset位置，循环取limit条数据，然后把剩下的数据舍弃。

```java
private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
    DefaultResultContext<Object> resultContext = new DefaultResultContext();
     this.skipRows(rsw.getResultSet(), rowBounds);  //游标跳到offset位置
     //取出limit条数据
     while(this.shouldProcessMoreRows(resultContext, rowBounds) && rsw.getResultSet().next()) {
         ResultMap discriminatedResultMap = this.resolveDiscriminatedResultMap(rsw.getResultSet(), resultMap, (String)null);
         Object rowValue = this.getRowValue(rsw, discriminatedResultMap);
         this.storeObject(resultHandler, resultContext, rowValue, parentMapping, rsw.getResultSet());
     }
  }
```

```java
 
private void skipRows(ResultSet rs, RowBounds rowBounds) throws SQLException {
     if (rs.getType() != 1003) {
         if (rowBounds.getOffset() != 0) {
             rs.absolute(rowBounds.getOffset());
         }
     } else {     //从头开始移动游标，直至offset位置
         for(int i = 0; i < rowBounds.getOffset(); ++i) {
             rs.next();
         }
     }
}
```







`示例`:

```java
@Override
public Page<Order> list(Queryable queryable) {
    //pageable中有数据查询的要求
    Pageable pageable = queryable.getPageable();
    //封装新的分页查询类
    Page<Order> page = new Page<Order>(pageable.getPageNumber(), pageable.getPageSize());
    //传入RowBounds，page就是RowBounds的子类，这样查询后page就有了总页数与总条数
    page.setRecords(mapper.selectList(page));
    return new PageImpl<Order>(page.getRecords(), pageable, page.getTotal());
}
```



