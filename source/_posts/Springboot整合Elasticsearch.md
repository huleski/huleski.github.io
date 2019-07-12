---
title: Springboot整合Elasticsearch
categories: Elasticsearch
tags: Elasticsearch
date: 2019-07-11 20:51:55
---

版本兼容
------

请一定注意版本兼容问题。这关系到很多maven依赖。参考: [Spring Data Elasticsearch Spring Boot version matrix](https://github.com/spring-projects/spring-data-elasticsearch/wiki/Spring-Data-Elasticsearch---Spring-Boot---version-matrix)

Spring Boot Version (x)	 | Spring Data Elasticsearch Version (y)	| Elasticsearch Version (z)
-----------|------------|--------------
x <= 1.3.5 | y <= 1.3.4 | z <= 1.7.2*
x >= 1.4.x | 2.0.0 <=y < 5.0.0** | 2.0.0 <= z < 5.0.0**

maven依赖 `pom.xml`:
-------------------

```xml
<parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.4.1.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
 </dependency>
 ```

 配置文件`application.yml`
 -------------------

 ```yml
spring:
  data:
    elasticsearch:
      # 集群名
      cluster-name: syncwt-es
      # 连接节点,注意在集群中通信都是9300端口，否则会报错无法连接上！
      cluster-nodes: localhost:9300,119.29.38.169:9300
      # 是否本地连接
      local: false
      repositories:
        # 仓库中数据存储
        enabled: true
 ```

启动项目后没有报错，日志出现以下说明代表成功。

```log
2017-03-30 19:35:23.078  INFO 20881 --- [           main] o.s.d.e.c.TransportClientFactoryBean     : adding transport node : localhost:9300
```

实体类
------

```java
@Document(indexName = "news", type = "news", shards = 3, replicas = 0)
public class News {

    @Id
    @Field(type = FieldType.Long)
    private Long newsId;

    @Field(type = FieldType.Text, searchAnalyzer = "ik_smart", analyzer = "ik_max_word")
    private String title;

    @Field(type = FieldType.Text, searchAnalyzer = "ik_smart", analyzer = "ik_max_word")
    private String summary;

    @Field(type = FieldType.Date, format = DateFormat.custom, pattern = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
    private Date publishTime;

    @Field(type = FieldType.Keyword)
    private String lang;

    @Field(type = FieldType.Date, format = DateFormat.custom, pattern = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
    private Date createTime;

    @Field(type = FieldType.Date, format = DateFormat.custom, pattern = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
    private Date updateTime;

    // setter and getter method
}
```

dao接口可以直接继承spring封装好的接口
-----

```java
public interface NewsRepository extends ElasticsearchRepository<News, Long> {
    // 自定义查询
    public List<News> findByTitle(String title);
}
```

Controller类
-----------

```java
@RestController
@RequestMapping("/es")
public class NewsController {
 
    @Autowired
    private NewsRepository newsRepository;
 
    /**
     * 添加
     * @return
     */
    @RequestMapping("add")
    public String add() {
        News news = new News();
        news.setId("1");
        news.setTitle("this is a title");
        news.summary("China No.1 !!!");
        news.setCreateTime(new Date());
        newsRepository.save(news);
        return "success";
    }
 
    /**
     * 删除
     * @return
     */
    @RequestMapping("delete")
    public String delete() {
        News news = newsRepository.queryById("1");
        newsRepository.delete(news);
        return "success";
    }
 
    /**
     * 局部更新
     * @return
     */
    @RequestMapping("update")
    public String update() {
        News news = newsRepository.queryById("1");
        news.setTitle("哈哈");
        newsRepository.save(news);
        return "success";
    }
    /**
     * 查询
     * @return
     */
    @RequestMapping("query")
    public News query() {
        News news = newsRepository.getById("1");
        return news;
    }
}
```

NewsRepository已经封装好了基本的增删改查:

```java
@NoRepositoryBean
public interface ElasticsearchRepository<T, ID extends Serializable> extends ElasticsearchCrudRepository<T, ID> {
    <S extends T> S index(S var1);

    Iterable<T> search(QueryBuilder var1);

    Page<T> search(QueryBuilder var1, Pageable var2);

    Page<T> search(SearchQuery var1);

    Page<T> searchSimilar(T var1, String[] var2, Pageable var3);

    void refresh();

    Class<T> getEntityClass();
}
```

分页排序查询
---------

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);

    Page<T> findAll(Pageable var1);
}
```

支持异步查询

```java
@Async
Future<News> findByTitle(String title);
```

NativeSearchQueryBuilder构建查询

```java
@Autowired
private ElasticsearchTemplate elasticsearchTemplate;

SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withFilter(boolFilter().must(termFilter("id", documentId)))
    .build();

Page<SampleEntity> sampleEntities = elasticsearchTemplate.queryForPage(searchQuery,SampleEntity.class);
```

利用Scan和Scroll进行大结果集查询

```java
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withIndices("test-index")
    .withTypes("test-type")
    .withPageable(new PageRequest(0,1))
    .build();
String scrollId = elasticsearchTemplate.scan(searchQuery,1000,false);
List<SampleEntity> sampleEntities = new ArrayList<SampleEntity>();
boolean hasRecords = true;
while (hasRecords){
    Page<SampleEntity> page = elasticsearchTemplate.scroll(scrollId, 5000L , new ResultsMapper<SampleEntity>() {
        @Override
        public Page<SampleEntity> mapResults(SearchResponse response) {
            List<SampleEntity> chunk = new ArrayList<SampleEntity>();
            for(SearchHit searchHit : response.getHits()){
                if(response.getHits().getHits().length <= 0) {
                    return null;
                }
                SampleEntity news = new SampleEntity();
                news.setId(searchHit.getId());
                news.setTitle((String)searchHit.getSource().get("title"));
                chunk.add(news);
            }
            return new PageImpl<SampleEntity>(chunk);
        }
    });
    if(page != null) {
        sampleEntities.addAll(page.getContent());
        hasRecords = page.hasNextPage();
    }
    else{
        hasRecords = false;
    }
}
```

自行封装Util方法

```java
@Autowired
private ElasticsearchTemplate elasticsearchTemplate;

public void searchHelper() throws IOException {
        Client transportClient = elasticsearchTemplate.getClient();
        News news = new News(1, "title", "content");
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(news);
        XContentBuilder builder = jsonBuilder()
                .startObject()
                .field("title", "China No.1")
                .endObject();

        IndexResponse response = transportClient.prepareIndex("es-news", "news")
                .setSource(jsonBuilder()
                        .startObject()
                        .field("title", "China No.1")
                        .endObject()
                )
                .execute()
                .actionGet();
        transportClient.close();
    }
```

spring-data-elasticsearch对es有很好的支持, 我们能通过spring-data很方便地操作Elasticsearch
