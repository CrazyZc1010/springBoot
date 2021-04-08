# 12 使用NoSQL技术  #  
Spring Data提供了其他项目，帮助您访问各种NoSQL技术，包括：
  
1. [MongoDB](https://spring.io/projects/spring-data-mongodb "MongoDB使用")
1. Neo4J
1. Elasticsearch
1. Solr
1. Redis
1. GemFire or Geode
1. Cassandra
1. Couchbase
1. LDAP  

Spring Boot为Redis、MongoDB、Neo4j、Elasticsearch、Solr Cassandra、Couchbase和LDAP提供了自动配置。您可以使用其他项目，但必须自己配置它们。请参阅[spring.io/projects/spring-data](https://spring.io/projects/spring-data)上的相应参考文档。  


## 12.1. Redis ##  
Redis是一个缓存、消息代理和功能丰富的键值存储。Spring Boot为Lettuce和Jedis客户机库以及Spring Data Redis提供的抽象库提供了基本的自动配置。  
有一个spring boot starter数据redis“starter”，可以方便地收集依赖项。默认情况下，它使用Lettuce。这个启动程序处理传统和被动应用程序。  
我们还提供了一个spring-boot-starter-data-redis reactive“starter”，以便与具有reactive支持的其他存储保持一致  


### 12.1.1. Connecting to Redis ###  
您可以注入一个自动配置的RedisConnectionFactory、StringRedisTemplate或vanilla RedisTemplate实例，就像注入任何其他Spring Bean一样。默认情况下，该实例尝试连接到本地主机6379上的Redis服务器。下面的列表显示了这样一个bean的示例： 
 
    @Component
    public class MyBean {
    
    private StringRedisTemplate template;
    
    @Autowired
    public MyBean(StringRedisTemplate template) {
    this.template = template;
    }
    
    // ...
    
    }  

您还可以注册任意数量的bean，以实现LettuceClientConfigurationBuilderCustomizer以进行更高级的自定义。如果使用Jedis，也可以使用JedisClientConfigurationBuilderCustomizer定向配置生成器自定义程序。  

如果您添加自己的@Bean（属于任何自动配置的类型），它将替换默认值（在RedisTemplate的情况下除外，当排除基于Bean名称RedisTemplate而不是其类型时）。默认情况下，如果commons-pool2位于类路径上，则会得到一个池连接工厂。  


## 12.2. MongoDB ##  
MongoDB是一个开源的NoSQL文档数据库，它使用类似JSON的模式，而不是传统的基于表的关系数据。Spring Boot为使用MongoDB提供了一些便利，包括springbootstarterdatamongodb和springbootstarterdatamongodb反应式“启动器”。  


### 12.2.1. Connecting to a MongoDB Database ###  
要访问MongoDB数据库，可以插入自动配置的org.springframework.data.MongoDB.MongoDatabaseFactory。默认情况下，实例尝试连接到MongoDB://localhost/test上的MongoDB服务器。以下示例演示如何连接到MongoDB数据库：
  
    import org.springframework.data.mongodb.MongoDatabaseFactory;
    import com.mongodb.client.MongoDatabase;
    
    @Component
    public class MyBean {
    
    private final MongoDatabaseFactory mongo;
    
    @Autowired
    public MyBean(MongoDatabaseFactory mongo) {
    this.mongo = mongo;
    }
    
    // ...
    
    public void example() {
    MongoDatabase db = mongo.getMongoDatabase();
    // ...
    }
    
    }  

您可以设置spring.data.mongodb.uri属性来更改URL并配置其他设置，例如副本集，如下例所示：  

    spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:12345,mongo2.example.com:23456/test  

或者，可以使用离散属性指定连接详细信息。例如，可以在application.properties中声明以下设置：  

    spring.data.mongodb.host=mongoserver.example.com
    spring.data.mongodb.port=27017
    spring.data.mongodb.database=test
    spring.data.mongodb.username=user
    spring.data.mongodb.password=secret  

如果您已经定义了自己的MongoClient，它将用于自动配置合适的MongoDatabaseFactory。 
如果未指定spring.data.mongodb.port，则使用默认值27017。您可以从前面显示的示例中删除这一行。  
如果不使用Spring数据MongoDB，可以注入MongoClient bean，而不是使用MongoDatabaseFactory。如果您想完全控制MongoDB连接的建立，还可以声明自己的MongoDatabaseFactory或MongoClient bean。  
如果您使用的是反应式驱动程序，那么SSL需要Netty。如果Netty可用并且要使用的工厂尚未自定义，则自动配置将自动配置此工厂。  


### 12.2.2. MongoTemplate ###  
SpringDataMongoDB提供了一个MongoTemplate类，其设计与Spring的JdbcTemplate非常相似。与JdbcTemplate一样，Spring Boot会自动配置一个bean，供您注入模板，如下所示：  

    import org.springframework.data.mongodb.core.MongoTemplate;
    import org.springframework.stereotype.Component;
    
    @Component
    public class MyBean {
    
    private final MongoTemplate mongoTemplate;
    
    public MyBean(MongoTemplate mongoTemplate) {
    this.mongoTemplate = mongoTemplate;
    }
    
    // ...
    
    }  
有关完整的详细信息，请参阅[MongoOperations Javadoc](https://docs.spring.io/spring-data/mongodb/docs/3.0.0.RC1/api/org/springframework/data/mongodb/core/MongoOperations.html)。  


### 12.2.3. Spring Data MongoDB Repositories ### 存储库  
Spring数据包括对MongoDB的存储库支持。与前面讨论的JPA存储库一样，基本原则是根据方法名自动构造查询。  
实际上，Spring Data JPA和Spring Data MongoDB共享相同的公共基础设施。您可以从前面的JPA示例开始，假设City现在是MongoDB数据类，而不是JPA@Entity，那么它的工作方式是相同的，如下例所示：  

    package com.example.myapp.domain;
    
    import org.springframework.data.domain.*;
    import org.springframework.data.repository.*;
    
    public interface CityRepository extends Repository<City, Long> {
    
    Page<City> findAll(Pageable pageable);
    
    City findByNameAndStateAllIgnoringCase(String name, String state);
    
    }  

您可以使用@EntityScan注释自定义文档扫描位置。  
有关SpringDataMongoDB的完整详细信息，包括其丰富的对象映射技术，请参阅其[参考文档](https://spring.io/projects/spring-data-mongodb)。  

### 12.2.4 嵌入式Mongo ###  
Spring Boot为嵌入式Mongo提供了自动配置。要在SpringBoot应用程序中使用它，请添加对de.flapdoodle.embed:de.flapdoodle.embed.mongo的依赖。    
Mongo监听的端口可以通过设置spring.data.mongodb.port属性进行配置。要使用随机分配的空闲端口，请使用值0。mongoutoconfiguration创建的MongoClient会自动配置为使用随机分配的端口。  
**如果不配置自定义端口，默认情况下，嵌入式支持使用随机端口（而不是27017）**。  
如果类路径上有SLF4J，Mongo生成的输出将自动路由到名为org.springframework.boot.autoconfigure.Mongo.embedded.EmbeddedMongo的记录器。  
您可以声明自己的IMongodConfig和IRuntimeConfig bean来控制Mongo实例的配置和日志路由。可以通过声明DownloadConfigBuilderCustomizer bean自定义下载配置。  


----------
2020/4/8 21:14:37 

----------
## 12.3. Neo4j ##  
Neo4j是一个开源的NoSQL图形数据库，它使用了一个由一级关系连接的节点的丰富数据模型，比传统的RDBMS方法更适合连接大数据。Spring Boot为使用Neo4j提供了一些便利，包括Spring-Boot-starter-data-Neo4j“starter”。  
### 12.3.1 连接到Neo4j数据库 ###  
要访问Neo4j服务器，可以插入自动配置的org.Neo4j.ogm.session.session。默认情况下，该实例尝试使用Bolt协议连接到localhost:7687上的Neo4j服务器。下面的示例演示如何注入Neo4j会话：  

    @Component
    public class MyBean {
    
    private final Session session;
    
    @Autowired
    public MyBean(Session session) {
    this.session = session;
    }
    
    // ...
    
    }
    
您可以通过设置spring.data.neo4j.*属性来配置要使用的uri和凭据，如下例所示：  

    spring.data.neo4j.uri=bolt://my-server:7687
    spring.data.neo4j.username=neo4j
    spring.data.neo4j.password=secret  

您可以通过添加org.neo4j.ogm.config.Configuration bean或org.neo4j.ogm.session.SessionFactory bean来完全控制会话的创建。  

### 12.3.2 使用嵌入式模式 ###  
如果将org.neo4j:neo4j ogm embedded驱动程序添加到应用程序的依赖项中，Spring Boot会自动配置一个进程内的neo4j嵌入式实例，该实例在应用程序关闭时不会保留任何数据。  
由于嵌入的Neo4j OGM驱动程序本身并不提供Neo4j内核，因此必须声明org.Neo4j:Neo4j为依赖项。有关兼容版本的列表，请参阅Neo4j OGM文档。  
当类路径上有多个驱动程序时，嵌入式驱动程序优先于其他驱动程序。可以通过设置spring.data.neo4j.embedded.enabled=false显式禁用嵌入模式。  
如果嵌入式驱动程序和Neo4j内核位于上述类路径上，则数据Neo4j测试会自动使用嵌入式Neo4j实例。  
通过在配置中提供数据库文件的路径（例如spring.data.neo4j.uri=file://var/tmp/graph.db），可以为嵌入式模式启用持久性。  
### 12.3.3. Using Native Types ###  
Neo4j OGM可以将一些类型（如java.time.*中的类型）映射到基于字符串的属性或Neo4j提供的本地类型之一。出于向后兼容性的原因，Neo4j OGM的默认设置是使用基于字符串的表示。若要使用本机类型，请添加对org.neo4j:neo4j ogm bolt本机类型或org.neo4j:neo4j ogm嵌入式本机类型的依赖关系，并配置spring.data.neo4j.use-native-types属性，如下例所示：  

    spring.data.neo4j.use-native-types=true  

### 12.3.4条。Neo4J会话 ###  
默认情况下，如果您运行的是web应用程序，则会话将绑定到整个请求处理的线程（即，它使用“在视图中打开会话”模式）。如果不需要此行为，请将以下行添加到application.properties文件中： 

    spring.data.neo4j.open-in-view=false  

### 12.3.5 Spring数据Neo4j存储库 ###  
Spring数据包括对Neo4j的存储库支持。  
Spring Data Neo4j与许多其他Spring数据模块一样，与Spring Data JPA共享公共基础设施。您可以从前面的JPA示例中将City定义为Neo4j OGM@NodeEntity而不是JPA@Entity，存储库抽象的工作方式与此相同，如下例所示：  

    package com.example.myapp.domain;
    
    import java.util.Optional;
    
    import org.springframework.data.neo4j.repository.*;
    
    public interface CityRepository extends Neo4jRepository<City, Long> {
    
    Optional<City> findOneByNameAndState(String name, String state);
    
    }

spring-boot-starter-data-neo4j“starter”支持存储库和事务管理。通过在@Configuration bean上分别使用@enableeo4jrepositories和@EntityScan，可以自定义查找存储库和实体的位置。  
有关Spring Data Neo4j的完整细节，包括其对象映射技术，请参阅参考文档。  


## 12.4. Solr ##  
Apache Solr是一个搜索引擎。Spring Boot为Solr 5客户机库提供了基本的自动配置，以及Spring Data Solr提供的抽象。有一个spring boot starter数据solr“starter”，用于以方便的方式收集依赖项。  


### 12.4.1. Connecting to Solr ###  
您可以像注入任何其他Spring bean一样注入一个自动配置的SolrClient实例。默认情况下，该实例尝试连接到localhost:8983/solr上的服务器。下面的示例演示如何注入Solr bean：   

    @Component
    public class MyBean {
    
    private SolrClient solr;
    
    @Autowired
    public MyBean(SolrClient solr) {
    this.solr = solr;
    }
    
    // ...
    
    }  

如果您添加自己的SolrClient类型的@Bean，它将替换默认值。  

### 12.4.2 Spring数据Solr存储库 ###  
Spring数据包括对Apache Solr的存储库支持。与前面讨论的JPA存储库一样，基本原则是根据方法名自动构造查询。  
事实上，Spring Data JPA和Spring Data Solr共享相同的公共基础设施。您可以从前面的JPA示例开始，假设City现在是一个@SolrDocument类，而不是一个JPA@Entity，那么它的工作方式是相同的。  
IP:有关Spring Data Solr的完整详细信息，请参阅参考文档。  

## 12.5. Elasticsearch ##  
Elasticsearch是一个开源、分布式、RESTful的搜索和分析引擎。Spring Boot为Elasticsearch提供了基本的自动配置。  
Spring Boot支持多个客户端：  

1. 正式的Java“低级”和“高级”REST客户机  
1. Spring Data Elasticsearch提供的ReactiveElasticsearchClient  

Spring Boot提供了一个专用的“Starter”，Spring Boot Starter数据弹性搜索。  

12.5.1 使用REST客户端连接到Elasticsearch  
Elasticsearch提供了两个不同的REST客户机，您可以使用它们来查询集群：“低级”客户机和“高级”客户机。  
如果类路径上有org.elasticsearch.client:elasticsearch rest client依赖项，Spring Boot将自动配置并注册一个RestClient bean，默认目标是localhost:9200。您可以进一步调整RestClient的配置方式，如下例所示：  

    spring.elasticsearch.rest.uris=https://search.example.com:9200
    spring.elasticsearch.rest.read-timeout=10s
    spring.elasticsearch.rest.username=user
    spring.elasticsearch.rest.password=secret  
您还可以注册任意数量的bean，以实现RestClientBuilderCustomizer以进行更高级的自定义。要完全控制注册，请定义RestClient bean。  
如果您在类路径上有org.elasticsearch.client:elasticsearch rest高级客户机依赖项，Spring Boot将自动配置rest高级客户机，该客户机包装任何现有的RestClient bean，并重用其HTTP配置。  

### 12.5.2 使用反应性REST客户端连接到Elasticsearch ###  
Spring Data Elasticsearch发布了反应式ElasticSearchClient，用于以反应式方式查询Elasticsearch实例。它构建在WebFlux的WebClient之上，因此springbootstarterelasticsearch和springbootstarterwebflux依赖项都有助于实现这种支持。  
默认情况下，Spring Boot将自动配置并注册一个以localhost:9200为目标的reactivelasticsearchclient bean。您可以进一步调整它的配置方式，如下例所示：  

    spring.data.elasticsearch.client.reactive.endpoints=search.example.com:9200
    spring.data.elasticsearch.client.reactive.use-ssl=true
    spring.data.elasticsearch.client.reactive.socket-timeout=10s
    spring.data.elasticsearch.client.reactive.username=user
    spring.data.elasticsearch.client.reactive.password=secret  

如果配置属性不够，并且希望完全控制客户端配置，则可以注册自定义客户端配置bean。


----------
2020/4/9 22:57:10 

----------

### 12.5.3 使用Spring数据连接到Elasticsearch ###
要连接到Elasticsearch，必须定义RestHighLevelClient bean，由Spring Boot自动配置，或由应用程序手动提供（参见前面的部分）。使用此配置，可以像任何其他SpringBean一样注入ElasticsearchRestTemplate，如下例所示：  

    @Component
    public class MyBean {
    
    private final ElasticsearchRestTemplate template;
    
    public MyBean(ElasticsearchRestTemplate template) {
    this.template = template;
    }
    
    // ...
    
    }  

在spring data elasticsearch和使用WebClient（通常是spring boot starter webflux）所需依赖项存在的情况下，spring boot还可以将ReactiveElasticsearchClient和ReactiveElasticsearchTemplate自动配置为bean。它们是其他REST客户机的反应等价物。  

### 12.5.4 Spring数据弹性搜索存储库 ###  
Spring数据包括对Elasticsearch的存储库支持。与前面讨论的JPA存储库一样，基本原则是根据方法名自动构造查询。  
事实上，Spring Data JPA和Spring Data Elasticsearch共享相同的公共基础设施。您可以从前面的JPA示例开始，假设City现在是一个Elasticsearch@Document类，而不是JPA@Entity，那么它的工作方式是相同的。  

有关Spring数据弹性搜索的完整详细信息，请参阅[参考文档](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/)。  
Spring Boot支持经典的和反应式的Elasticsearch存储库，使用ElasticsearchRestTemplate或反应式的ElasticSearchTemplate bean。由于存在所需的依赖项，这些bean很可能是由Spring Boot自动配置的。  
如果希望使用自己的模板来支持Elasticsearch存储库，可以添加自己的ElasticsearchRestTemplate或ElasticsearchOperations@Bean，只要它名为“elasticsearchTemplate”。这同样适用于ReactiveElasticsearchTemplate和ReactiveElasticsearchOperations，bean名为“ReactiveElasticsearchTemplate”。  
您可以选择使用以下属性禁用存储库支持：  

    spring.data.elasticsearch.repositories.enabled=false  

## 12.6. Cassandra ##  
Cassandra是一个开放源代码的分布式数据库管理系统，旨在跨许多商品服务器处理大量数据。Spring Boot为Cassandra提供了自动配置，并提供了Spring Data Cassandra提供的抽象。有一个spring boot starter数据cassandra“starter”，用于以方便的方式收集依赖项。  


### 12.6.1. Connecting to Cassandra ###  
您可以注入一个自动配置的CassandraTemplate或一个cassandracklession实例，就像注入任何其他Spring Bean一样。spring.data.cassandra.*属性可用于自定义连接。通常，您可以提供键空间名称和联系人以及本地数据中心名称，如下例所示：  

    spring.data.cassandra.keyspace-name=mykeyspace
    spring.data.cassandra.contact-points=cassandrahost1:9042,cassandrahost2:9042
    spring.data.cassandra.local-datacenter=datacenter1  

如果所有联系人的端口都相同，则可以使用快捷方式，只指定主机名，如下例所示：  

    spring.data.cassandra.keyspace-name=mykeyspace
    spring.data.cassandra.contact-points=cassandrahost1,cassandrahost2
    spring.data.cassandra.local-datacenter=datacenter1  

这两个示例与端口默认值9042相同。如果需要配置端口，请使用spring.data.cassandra.port。  
您还可以注册任意数量的bean来实现DriverConfigLoaderBuilderCustomizer，以便进行更高级的驱动程序自定义。CqlSession可以使用CqlSessionBuilderCustomizer类型的bean进行定制。  
如果使用CqlSession builder创建多个CqlSession bean，请记住builder是可变的，因此请确保为每个会话注入一个新副本。  
下面的代码列表显示了如何注入Cassandra bean：  

    @Component
    public class MyBean {
    
    private final CassandraTemplate template;
    
    public MyBean(CassandraTemplate template) {
    this.template = template;
    }
    
    // ...
    
    }  

如果添加CassandraTemplate类型的@Bean，它将替换默认值。  
### 12.6.2. Spring Data Cassandra Repositories ###  
Spring数据包括对Cassandra的基本存储库支持。目前，这比前面讨论的JPA存储库更加有限，需要用@Query注释finder方法。  
有关Spring Data Cassandra的完整详细信息，请参阅[参考文档](https://docs.spring.io/spring-data/cassandra/docs/)  

## 12.7. Couchbase ##  
Couchbase是一个开源、分布式、多模型的面向NoSQL文档的数据库，为交互式应用程序优化。Spring Boot提供Couchbase的自动配置以及springdatacouchbase在其上提供的抽象。有spring boot starter data couchbase和spring boot starter data couchbase reactive“Starters”，可以方便地收集依赖项。  
### 12.7.1条。连接到Couchbase ###  
您可以通过添加Couchbase SDK和一些配置来获得集群。spring.couchbase.*属性可用于自定义连接。通常，您提供连接字符串、用户名和密码，如下例所示：  

    spring.couchbase.connection-string=couchbase://192.168.1.123
    spring.couchbase.username=user
    spring.couchbase.password=secret  
也可以自定义一些ClusterEnvironment设置。例如，以下配置更改了用于打开新存储桶并启用SSL支持的超时：  

    spring.couchbase.env.timeouts.connect=3000
    spring.couchbase.env.ssl.key-store=/location/of/keystore.jks
    spring.couchbase.env.ssl.key-store-password=secret  

有关更多详细信息，请查看spring.couchbase.env.*属性。要获得更多控制，可以使用一个或多个ClusterEnvironmentBuilderCustomizer bean。  
### 12.7.2. Spring Data Couchbase Repositories ###  
Spring数据包括对Couchbase的存储库支持。有关Spring数据库的完整详细信息，请参阅参考文档。  
如果CouchbaseClientFactory Bean可用，您可以像对任何其他springbean一样插入一个自动配置的CouchbaseTemplate实例。当集群可用（如上所述）并且指定了bucket名称时，就会发生这种情况：  

    spring.data.couchbase.bucket-name=my-bucket  
下面的示例演示如何注入CouchbaseTemplate bean：  

    @Component
    public class MyBean {
    
    private final CouchbaseTemplate template;
    
    @Autowired
    public MyBean(CouchbaseTemplate template) {
    this.template = template;
    }
    
    // ...
    
    }  

您可以在自己的配置中定义几个bean来覆盖自动配置提供的bean：  

1. CouchbaseMappingContext@Bean，名称为CouchbaseMappingContext。  
1. 一个名为couchbaseCustomConversions的CustomConversions@Bean。  
1. CouchbaseTemplate@Bean，名称为CouchbaseTemplate。   

为了避免在您自己的配置中硬编码这些名称，您可以重用Spring Data Couchbase提供的BeanNames。例如，可以自定义要使用的转换器，如下所示：  

    @Configuration(proxyBeanMethods = false)
    public class SomeConfiguration {
    
    @Bean(BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
    public CustomConversions myCustomConversions() {
    return new CustomConversions(...);
    }
    
    // ...
    
    }  


----------
2020/4/16 20:32:08 

----------
## 12.8 LDAP ##  
LDAP（轻量级目录访问协议）是一种开放的、与供应商无关的、行业标准的应用程序协议，用于通过IP网络访问和维护分布式目录信息服务。Spring Boot为任何兼容的LDAP服务器提供了自动配置，并支持UnboundID中的嵌入式内存LDAP服务器。  
LDAP抽象由springdataldap提供。有一个spring boot starter数据ldap“starter”，用于以方便的方式收集依赖项。  

### 12.8.1 连接到LDAP服务器 ###  
要连接到LDAP服务器，请确保声明对spring boot starter data LDAP“starter”或spring LDAP core的依赖，然后在application.properties中声明服务器的URL，如下例所示：  

    spring.ldap.urls=ldap://myserver:1235
    spring.ldap.username=admin
    spring.ldap.password=secret  

如果需要自定义连接设置，可以使用spring.ldap.base和spring.ldap.base-environment属性。  
LdapContextSource是基于这些设置自动配置的。如果DirContextAuthenticationStrategy bean可用，则它与自动配置的LdapContextSource关联。如果需要对其进行自定义，例如要使用PooledContextSource，仍然可以注入自动配置的LdapContextSource。确保将自定义ContextSource标记为@Primary，以便自动配置的LdapTemplate使用它。  

### 12.8.2 Spring数据LDAP存储库 ###  
Spring数据包括对LDAP的存储库支持。有关Spring Data LDAP的完整详细信息，请参阅参考文档。  
您还可以像对任何其他Spring Bean一样，注入一个自动配置的LdapTemplate实例，如下例所示：  

    @Component
    public class MyBean {
    
    private final LdapTemplate template;
    
    @Autowired
    public MyBean(LdapTemplate template) {
    this.template = template;
    }
    
    // ...
    
    }  

### 12.8.3  嵌入式内存LDAP服务器   ###  
出于测试目的，Spring Boot支持从UnboundID自动配置内存中的LDAP服务器。要配置服务器，请向com.unboundid:unboundid ldapsdk添加依赖项并声明spring.ldap.embedded.base-dn属性，如下所示：  

    spring.ldap.embedded.base-dn=dc=spring,dc=io  

但是，可以定义多个基dn值，因为可分辨名称通常包含逗号，所以必须使用正确的表示法定义它们.  
在yaml文件中，可以使用yaml列表表示法 
 
    	spring.ldap.embedded.base-dn:
      - dc=spring,dc=io
      - dc=pivotal,dc=io  
  - 
在属性文件中，必须包含索引作为属性名称的一部分：  

    spring.ldap.embedded.base-dn[0]=dc=spring,dc=io
    spring.ldap.embedded.base-dn[1]=dc=pivotal,dc=io  

默认情况下，服务器在随机端口上启动并触发常规LDAP支持。不需要指定spring.ldap.url属性。  
如果类路径上有schema.ldif文件，则它用于初始化服务器。如果要从其他资源加载初始化脚本，还可以使用spring.ldap.embedded.ldif属性。  
默认情况下，标准模式用于验证LDIF文件。通过设置spring.ldap.embedded.validation.enabled属性，可以完全关闭验证。如果有自定义属性，则可以使用spring.ldap.embedded.validation.schema定义自定义属性类型或对象类。   

## 12.9. InfluxDB ##  
InfluxDB是一个开源的时间序列数据库，针对操作监控、应用程序度量、物联网传感器数据和实时分析等领域的时间序列数据的快速、高可用性存储和检索进行了优化。  

### 12.9.1 连接到InfloxDB ###  
Spring Boot自动配置infloxdb实例，前提是infloxdb java客户端位于类路径上，并且设置了数据库的URL，如下例所示：  

    spring.influx.url=https://172.0.0.1:8086  

如果到infloxdb的连接需要用户和密码，则可以相应地设置spring.inflox.user和spring.inflox.password属性。  
infloxdb依赖于OkHttp。如果需要优化infloxdb在后台使用的http客户端，可以注册infloxdbokhtcpclientbuilderprovider bean。