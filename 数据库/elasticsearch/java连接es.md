# JAVA连接Elasticsearch

#### 加载依赖

```xml
		<dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.8.3</version>
        </dependency>
		<!--核心依赖-->
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```



#### 加载配置文件

```xml
spring.data.elasticsearch.cluster-name=elasticsearch  //elasticsearch

spring.data.elasticsearch.cluster-nodes=localhost:9300	//设置连接号

spring.main.banner-mode=off
```

![1567682087201](C:\Users\Zhangxinuser\AppData\Roaming\Typora\typora-user-images\1567682087201.png)

> elasticsearch要和图片上的对应



#### 修改domain实体类

> <font style="color:red">@Document</font> : 配置操作哪个索引下的哪个类型
>
> - indexName   索引的名称
> - type  类型的名称
>
> <font style="color:red">@ID</font> : 用来标记文档的ID  通常设置在主键上
>
> <font style="color:red">@Field</font> : 用来配置 配置信息 比如分词器
>
> - analyzer  设置分词器的类型
> - searchAnalyzer  设置分词器的类型 (6.x的版本貌似已经没有用了)
> - type 设置文档的类型  text   keyword  integer等 , 如果设置分词的属性则必须要设置 , 否则可以不写的



##### 代码展示

```java
@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
@ToString
@Document(indexName = "shop_product", type = "shop_product")
public class Product {
    @Id
    private Long id;
    @Field(type = FieldType.Text, analyzer = "ik_smart", searchAnalyzer = "ik_smart")
    private String title;
    @Field(type = FieldType.Integer)
    private Integer price;
    @Field(type = FieldType.Text, analyzer = "ik_smart", searchAnalyzer = "ik_smart")
    private String intro;
    @Field(type = FieldType.Keyword)
    private String brand;
}
```



#### 获取对象

##### 1.ElasticsearchRepository

###### 创建方式

> ElasticsearchRepository接口包含了我们想要的crud操作所以我们直接创建一个接口用来集成这个接口就可以获取到`Repository`对象
>
> <font style="color:red">在集成接口的时候需要填写两个泛型  1.类的类型也就是domain  2.文档ID的Long类型</font>
>
> 在自定义的接口上贴上 <font style="color:red">`@Repository`</font>注解用来通知Spring 管理这个接口 通过jdk动态代理的方式

###### 代码展示

```java
@Repository
public interface ProductESRepository extends ElasticsearchRepository<Product,Long> {

}
```



##### 2.配置Spring Data 集成的ElasticsearchTemplate

###### 创建/获取方式

> 直接通过多肽的方式获取到`Template`对象 直接操作即可



###### 代码展示

```java
 @Autowired
 private ElasticsearchTemplate template;
```



##### 两种方式对比

从功能的角度来讲`ElasticsearchRepository` 就已经可以应对 90%的业务操作 , 但是如果遇到很麻烦的业务 , 涉及底层代码的话 , 还是要使用`Template` 关联的方式来操作es, `Template`操作es功能更丰富



#### 业务操作

无论哪种方式 , 对于es的基本操作是相差不多的

在es的操作中 基本和dsl语法差不多需要注意的是很多需要用到的对象 , 都不是new对象的方式创建出来的 , 这里列举几个比较重要的类和核心



##### 根据品牌进行分组统计

> ```java
>    /**
>      * 根据品牌进行分组统计数量
>      */
> @Test
>     public void testGroup() {
>         //获取查询的条件
>         NativeSearchQueryBuilder builder = new NativeSearchQueryBuilder();
>         //设置aggs层 并设置自定义名字
>         builder.addAggregation(AggregationBuilders.terms("groupByBrand").field("brand"));
>         //获得查询到的结果集
>         AggregatedPageImpl<Product> page = (AggregatedPageImpl) repository.search(builder.build());
>         //获取真实类型
>         StringTerms groupByBrand = (StringTerms) page.getAggregation("groupByBrand");
>         //获取桶
>         List<StringTerms.Bucket> buckets = groupByBrand.getBuckets();
>         //循环桶
>         for (StringTerms.Bucket bucket : buckets) {
>             //获取桶内部的元素
>             System.out.println(bucket.getKey() + "--" + bucket.getDocCount());
>         }
>     }
> ```



##### 单条件的等值查询 品牌等于华为

```java
    //等值查询
    @Test
    public void testGame() {
        //获取查询条件对象
        TermQueryBuilder queryBuilder = QueryBuilders.termQuery("brand", "华为");
        Iterable<Product> search = repository.search(queryBuilder);
        for (Product product : search) {
            System.out.println(product);
        }
    }
```



##### 分页查询

```java
    //分页查询
    @Test
    public void testPage() {
        Page<Product> products = repository.findAll(PageRequest.of(1, 3));
        for (Product product : products) {
            System.out.println(product);
        }
    }
```



##### 多条件查询

```java
    //条件查询
    @Test
    public void testProduct3() {
        NativeSearchQueryBuilder builder = new NativeSearchQueryBuilder();
        builder.withQuery(QueryBuilders.multiMatchQuery("游戏 手机", "title", "intro"));
        Page<Product> search = repository.search(builder.build());
        search.forEach(System.out::println);
    }
```



##### 排序查询

```java
    //排序查询
    @Test
    public void testProducts() {
        Iterable<Product> price = repository.findAll(Sort.by(Sort.Order.asc("price")));
        price.forEach(System.out::println);
    }
```



##### 高亮

> 在配置好主键和类型之后通过 query封装查询的条件 , 利用分词器分词 , 设置高亮的内容 使用springdata模板进行查询 , **最重要的是对封装结果的修改**
>
> 在重新修改的封装结果中 , 

```java
    @Test
    public void test() {
        ObjectMapper mapper = new ObjectMapper();
        String preTage = "<span style='color:red'>";
        String postTage = "</span>";
        NativeSearchQueryBuilder builder = new NativeSearchQueryBuilder();
        builder.withIndices("shop_product").withTypes("shop_product");
        builder.withQuery(QueryBuilders.multiMatchQuery("游戏 手机", "title", "intro"));
        builder.withHighlightFields(
                new HighlightBuilder.Field("title").preTags(preTage).postTags(postTage),
                new HighlightBuilder.Field("intro").preTags(preTage).postTags(postTage));
        AggregatedPage<Product> page = template.queryForPage(builder.build(), Product.class, new SearchResultMapper() {
            @Override
            public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {
                List<T> content = new ArrayList<>();
                //查询结果在 response中
                SearchHit[] hits = response.getHits().getHits();
                try {
                    //表示了数组中的数据
                    for (SearchHit hit : hits) {
                        //获取数组中source的数据
                        String source = hit.getSourceAsString();
                        //t是通过fastjson转换的数据
                        T t = mapper.readValue(source, clazz);
                        //取出数据内容然后进行替换
                        Collection<HighlightField> values = hit.getHighlightFields().values();
                        for (HighlightField find : values) {
                            //获取数据的名字  title shro
                            //获取内容
                            //通过工具类将泛型 , 名称, 值 都转换为对象
                            BeanUtils.setProperty(t, find.getName(), find.getFragments());
                        }
                        content.add(t);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return null;
                }
                                                                        //获取总数量
                return new AggregatedPageImpl<T>(content, pageable, response.getHits().totalHits);
            }
        });
        page.forEach(System.out::println);
        //对查询到的结果进行处理
    }
```













