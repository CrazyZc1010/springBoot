# 11 使用SQL数据库 #  
Spring框架为使用SQL数据库提供了广泛的支持，从使用JdbcTemplate直接访问JDBC到完成Hibernate等“对象关系映射”技术。Spring数据提供了额外的功能级别：直接从接口创建存储库实现，并使用约定从方法名生成查询。  
# 11.1 配置数据源 #  
Java的javax.sql.DataSource接口提供了处理数据库连接的标准方法。传统上，“数据源”使用URL和一些凭据来建立数据库连接。  
有关更高级的示例，请参见“如何”部分，通常是为了完全控制数据源的配置。  
### 11.1.1 嵌入式数据库支持 ###  
使用内存中的嵌入式数据库开发应用程序通常很方便。显然，内存数据库不提供持久存储。您需要在应用程序启动时填充数据库，并准备在应用程序结束时丢弃数据。  
“如何”部分包括一个关于如何初始化数据库的部分。  
Spring Boot可以自动配置嵌入式H2、HSQL和Derby数据库。您不需要提供任何连接URL。只需要包含要使用的嵌入数据库的生成依赖项。  
如果在测试中使用此功能，您可能会注意到，整个测试套件都会重用同一个数据库，而不管您使用的应用程序上下文的数量如何。如果要确保每个上下文都有单独的嵌入数据库，则应将spring.datasource.generate-unique-name设置为true。  
例如，典型的POM依赖关系如下：  

    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
    </dependency>  

您需要依赖spring jdbc才能自动配置嵌入式数据库。在本例中，它是通过spring boot starter data jpa以传递方式引入的。  

如果出于任何原因，您确实为嵌入式数据库配置了连接URL，请注意确保数据库的自动关闭被禁用。如果使用H2，则应使用DB_CLOSE_ON_EXIT=FALSE来执行此操作。如果使用HSQLDB，则应确保不使用shutdown=true。禁用数据库的自动关闭允许Spring在数据库关闭时进行引导控制，从而确保在不再需要访问数据库时发生这种情况。  

### 11.1.2 连接到生产数据库 ###  
还可以使用池数据源自动配置生产数据库连接。Spring Boot使用以下算法选择特定的实现：
  
1. 我们更喜欢HikariCP的性能和并发性。如果HikariCP可用，我们总是选择它。  
1. 否则，如果Tomcat池数据源可用，我们就使用它。  
1. 如果HikariCP和Tomcat池数据源都不可用，并且Commons DBCP2可用，则使用它  

如果使用spring boot starter jdbc或spring boot starter data jpa“starters”，则会自动获得对HikariCP的依赖。  
您可以完全绕过该算法，并通过设置spring.datasource.type属性指定要使用的连接池。如果在Tomcat容器中运行应用程序，这一点尤其重要，因为Tomcat jdbc是默认提供的。  
始终可以手动配置其他连接池。如果定义自己的数据源bean，则不会自动配置。  
数据源配置由spring.DataSource.*中的外部配置属性控制。例如，可以在application.properties中声明以下部分：  

    spring.datasource.url=jdbc:mysql://localhost/test
    spring.datasource.username=dbuser
    spring.datasource.password=dbpass
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver  

至少应该通过设置spring.datasource.URL属性来指定URL。否则，Spring Boot会尝试自动配置嵌入式数据库。  

您通常不需要指定驱动程序类名，因为Spring Boot可以从url推断大多数数据库的驱动程序类名。  
为了创建一个池数据源，我们需要能够验证一个有效的驱动程序类是可用的，所以我们在做任何事情之前都要检查它。换句话说，如果设置spring.datasource.driver class name=com.mysql.jdbc.driver，那么该类必须是可加载的。  
有关支持的选项的详细信息，请参阅数据源属性。无论实际实现如何，这些都是标准选项。还可以使用它们各自的前缀（spring.datasource.hikari.*、spring.datasource.tomcat.*和spring.datasource.dbcp2.*）来微调特定于实现的设置。有关更多详细信息，请参阅正在使用的连接池实现的文档。  
例如，如果使用Tomcat连接池，可以自定义许多其他设置，如下例所示：  

    # Number of ms to wait before throwing an exception if no connection is available.
    spring.datasource.tomcat.max-wait=10000
    
    # Maximum number of active connections that can be allocated from this pool at the same time.
    spring.datasource.tomcat.max-active=50
    
    # Validate the connection before borrowing it from the pool.
    spring.datasource.tomcat.test-on-borrow=true  

### 11.1.3条。连接到JNDI数据源  ###  
如果将Spring引导应用程序部署到应用程序服务器，则可能需要使用应用程序服务器的内置功能配置和管理数据源，并使用JNDI访问它。  
spring.datasource.jndi-name属性可以用作spring.datasource.url、spring.datasource.username和spring.datasource.password属性的替代项，以从特定jndi位置访问数据源。例如，application.properties中的以下部分显示了如何访问JBoss AS defined DataSource：

    spring.datasource.jndi-name=java:jboss/datasources/customers  


## 11.2. Using JdbcTemplate ##  
Spring的JdbcTemplate和NamedParameterJdbcTemplate类是自动配置的，您可以@Autowire直接将它们连接到自己的bean中，如下例所示：  

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.stereotype.Component;
    
    @Component
    public class MyBean {
    
    private final JdbcTemplate jdbcTemplate;
    
    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
    }
    
    // ...
    
    }  

可以使用spring.jdbc.template.*属性自定义模板的一些属性，如下例所示：  

    spring.jdbc.template.max-rows=500  

NamedParameterJdbcTemplate在后台重用相同的JdbcTemplate实例。如果定义了多个JdbcTemplate且不存在主候选项，则不会自动配置NamedParameterJdbcTemplate。 


----------
2020/4/4 18:18:26 

----------
## 11.3 JPA和Spring数据JPA ##  
Java Persistence API是一种标准技术，允许您将对象“映射”到关系数据库。spring boot starter数据jpa POM提供了一种快速入门的方法。它提供以下关键依赖项：  

1. Hibernate：最流行的JPA实现之一。  
1. Spring Data JPA：使实现基于JPA的存储库变得容易。  
1. Spring ORMs：来自Spring框架的核心ORM支持  

我们在这里不涉及太多JPA或Spring数据的细节。您可以遵循spring.io中的“使用JPA访问数据”指南，并阅读spring Data JPA和Hibernate参考文档。  
### 11.3.1 实体类   ###  
传统上，JPA“实体”类是在persistence.xml文件中指定的。对于Spring Boot，不需要这个文件，而是使用“实体扫描”。默认情况下，将搜索主配置类（用@EnableAutoConfiguration或@springbootsapplication注释的包）下面的所有包。  
任何用@Entity、@Embeddable或@MappedSuperclass注释的类都将被考虑。典型的实体类类似于以下示例：  

    package com.example.myapp.domain;
    
    import java.io.Serializable;
    import javax.persistence.*;
    
    @Entity
    public class City implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private String state;
    
    // ... additional members, often include @OneToMany mappings
    
    protected City() {
    // no-args constructor required by JPA spec
    // this one is protected since it shouldn't be used directly
    }
    
    public City(String name, String state) {
    this.name = name;
    this.state = state;
    }
    
    public String getName() {
    return this.name;
    }
    
    public String getState() {
    return this.state;
    }
    
    // ... etc
    
    }  

您可以使用@EntityScan注释自定义实体扫描位置。请参阅“how to.html”操作方法。  
### 11.3.2 Spring数据JPA存储库 ###  
Spring Data JPA存储库是可以定义来访问数据的接口。JPA查询是根据方法名自动创建的。例如，CityRepository接口可以声明findAllByState（字符串状态）方法来查找给定状态下的所有城市。  
对于更复杂的查询，可以使用Spring数据的查询注释来注释方法。  
Spring数据存储库通常从存储库或原始接口扩展。如果使用自动配置，则会从包含主配置类（用@EnableAutoConfiguration或@springbootsapplication注释的配置类）的包中搜索存储库。  
以下示例显示了典型的Spring数据存储库接口定义：  

    package com.example.myapp.domain;
    
    import org.springframework.data.domain.*;
    import org.springframework.data.repository.*;
    
    public interface CityRepository extends Repository<City, Long> {
    
    Page<City> findAll(Pageable pageable);
    
    City findByNameAndStateAllIgnoringCase(String name, String state);
    
    }

Spring数据JPA存储库支持三种不同的引导模式：default, deferred, and lazy。要启用延迟或延迟引导，请分别将spring.data.jpa.repositories.bootstrap-mode属性设置为延迟或延迟。使用deferred or lazy引导时，自动配置的EntityManagerFactoryBuilder将使用上下文的AsyncTaskExecutor（如果有）作为引导执行器。如果存在多个，将使用名为applicationTaskExecutor的。  
我们几乎没有触及Spring数据JPA的表面。有关完整的详细信息，请参阅Spring Data JPA参考文档。  
### 11.3.3 创建和删除JPA数据库 ###  
默认情况下，只有在使用嵌入式数据库（H2、HSQL或Derby）时，才会自动创建JPA数据库。您可以使用spring.JPA.*属性显式地配置JPA设置。例如，要创建和删除表，可以将以下行添加到application.properties：  

    spring.jpa.hibernate.ddl-auto=create-drop  

Hibernate自己的内部属性名（如果你记得更好的话）是Hibernate.hbm2ddl.auto。您可以使用spring.jpa.properties.*（在将前缀添加到实体管理器之前去掉前缀）将其与其他Hibernate本机属性一起设置。下一行显示了为Hibernate设置JPA属性的示例：  

    spring.jpa.properties.hibernate.globally_quoted_identifiers=true  

上例中的行将hibernate.globally_quoted_identifiers属性的值true传递给hibernate实体管理器。  
默认情况下，DDL执行（或验证）将延迟到ApplicationContext启动。还有一个spring.jpa.generate-ddl标志，但如果Hibernate auto configuration处于活动状态，则不使用该标志，因为ddl auto设置更细粒度。  
### 11.3.4 在视图中打开实体管理器 ###  
如果您正在运行一个web应用程序，Spring Boot默认情况下会注册OpenEntityManagerInViewInterceptor来应用“Open EntityManager in View”模式，以允许在web视图中延迟加载。如果不需要此行为，则应在application.properties中将spring.jpa.open-in-view设置为false。  
## 11.4 Spring数据JDBC  ##  
Spring数据包括对JDBC的存储库支持，并将自动为CrudRepository上的方法生成SQL。对于更高级的查询，将提供@Query注释。  
当必要的依赖项在类路径上时，Spring Boot将自动配置Spring Data的JDBC存储库。它们可以通过对spring boot starter数据jdbc的单一依赖性添加到您的项目中。如有必要，您可以通过向应用程序添加@EnableJdbcRepositories注释或JdbcConfiguration子类来控制Spring数据JDBC的配置。  
有关Spring数据JDBC的完整详细信息，请参阅参考文档。  

## 11.5. Using H2’s Web Console ##  
H2数据库提供了一个基于浏览器的控制台，Spring Boot可以为您自动配置。当满足以下条件时，控制台将自动配置：  

1. 您正在开发一个基于servlet的web应用程序。 
1. h2database:h2在类路径上。  
1. 您正在使用Spring Boot的开发工具。 

如果您不使用Spring Boot的开发工具，但仍希望使用H2的控制台，则可以将Spring.H2.console.enabled属性的值配置为true。  
H2控制台仅在开发期间使用，因此应注意确保spring.H2.console.enabled在生产中未设置为true。  

### 11.5.1 更改H2控制台的路径 ###  
默认情况下，控制台位于/h2控制台。可以使用spring.h2.console.path属性自定义控制台的路径。  


----------
2020/4/6 22:32:04 

----------


## 11.6 使用jOOQ ##  
jOOQ面向对象查询（jOOQ）是datageery的一个流行产品，它从数据库生成Java代码，并允许您通过其fluent API构建类型安全的SQL查询。商业版和开源版都可以与Spring Boot一起使用。  

### 11.6.1 代码生成 ###  
为了使用jOOQ类型安全查询，需要从数据库模式生成Java类。您可以按照jOOQ用户手册中的说明进行操作。如果您使用jooq codegen maven插件，并且还使用spring boot starter父级“parent POM”，则可以安全地省略该插件的<version>标记。您还可以使用SpringBoot定义的版本变量（例如h2.version）来声明插件的数据库依赖项。下面的列表显示了一个示例：

    <plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
    ...
    </executions>
    <dependencies>
    <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>${h2.version}</version>
    </dependency>
    </dependencies>
    <configuration>
    <jdbc>
    <driver>org.h2.Driver</driver>
    <url>jdbc:h2:~/yourdatabase</url>
    </jdbc>
    <generator>
    ...
    </generator>
    </configuration>
    </plugin>  

### 11.6.2 使用DSLContext  ###  
jOOQ提供的fluent API是通过org.jOOQ.DSLContext接口启动的。Spring Boot将DSLContext自动配置为Spring Bean，并将其连接到应用程序数据源。要使用DSLContext，可以@Autowire，如下例所示：  

    @Component
    public class JooqExample implements CommandLineRunner {
    
    private final DSLContext create;
    
    @Autowired
    public JooqExample(DSLContext dslContext) {
    this.create = dslContext;
    }
    
    }  

jOOQ手册倾向于使用一个名为create的变量来保存DSLContext。  
然后可以使用DSLContext构造查询，如下例所示：  
  
    public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
    .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
    .fetch(AUTHOR.DATE_OF_BIRTH);
    }  

### 11.6.3. jOOQ SQL Dialect ###  
除非配置了spring.jooq.sql-dialect属性，否则spring Boot将确定要用于数据源的sql Dialect。如果Spring Boot无法检测  Dialect，则使用默认值。  
Spring Boot只能自动配置jOOQ开源版本支持的Dialect。  

### 11.6.4 定制jOOQ ###  
可以通过定义自己的@Bean定义来实现更高级的定制，该定义在创建jOOQ配置时使用。可以为以下jOOQ类型定义bean：  

1. ConnectionProvider
1. ExecutorProvider
1. TransactionProvider
1. RecordMapperProvider
1. RecordUnmapperProvider
1. Settings
1. RecordListenerProvider
1. ExecuteListenerProvider
1. VisitListenerProvider
1. TransactionListenerProvider  


如果您想完全控制jooq配置，还可以创建自己的org.jooq.Configuration@Bean。  


## 11.7. Using R2DBC ##  
反应式关系数据库连接（R2DBC）项目将反应式编程api引入关系数据库。R2DBC的io.R2DBC.spi.Connection提供了使用非阻塞数据库连接的标准方法。连接是通过ConnectionFactory提供的，类似于使用jdbc的数据源。    
ConnectionFactory配置由spring.r2dbc.*中的外部配置属性控制。例如，可以在application.properties中声明以下部分：  

    spring.r2dbc.url=r2dbc:postgresql://localhost/test
    spring.r2dbc.username=dbuser
    spring.r2dbc.password=dbpass    

不需要指定驱动程序类名，因为Spring Boot从R2DBC的连接工厂发现中获取驱动程序。  
至少应该提供url。URL中指定的信息优先于单个属性，即名称、用户名、密码和池选项。  
“如何”部分包括一个关于如何初始化数据库的部分。  
要自定义由ConnectionFactory创建的连接，即设置不希望（或无法）在中央数据库配置中配置的特定参数，可以使用ConnectionFactoryOptionsBuilderCustomizer@Bean。下面的示例演示如何在从应用程序配置获取其余选项时手动覆盖数据库端口：
  
    @Bean
    public ConnectionFactoryOptionsBuilderCustomizer connectionFactoryPortCustomizer() {
    return (builder) -> builder.option(PORT, 5432);
    }  

以下示例演示如何设置一些PostgreSQL连接选项：  

    @Bean
    public ConnectionFactoryOptionsBuilderCustomizer postgresCustomizer() {
    Map<String, String> options = new HashMap<>();
    options.put("lock_timeout", "30s");
    options.put("statement_timeout", "60s");
    return (builder) -> builder.option(OPTIONS, options);
    }  

当ConnectionFactory bean可用时，常规的jdbc数据源自动配置将退出。  

### 11.7.1 嵌入式数据库支持 ###  
与JDBC支持类似，Spring Boot可以自动配置嵌入式数据库以进行响应性使用。您不需要提供任何连接URL。只需包含要使用的嵌入数据库的生成依赖项，如下例所示  

    <dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
    <scope>runtime</scope>
    </dependency>
  
如果在测试中使用此功能，您可能会注意到，整个测试套件都会重用同一个数据库，而不管您使用的应用程序上下文的数量如何。如果要确保每个上下文都有单独的嵌入式数据库，则应将spring.r2dbc.generate-unique-name设置为true。  

### 11.7.2 使用数据库客户端 ###  
Spring Data的DatabaseClient类是自动配置的，您可以@Autowire直接将其连接到自己的bean中，如下例所示：  

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.r2dbc.function.DatabaseClient;
    import org.springframework.stereotype.Component;
    
    @Component
    public class MyBean {
    
    private final DatabaseClient databaseClient;
    
    @Autowired
    public MyBean(DatabaseClient databaseClient) {
    this.databaseClient = databaseClient;
    }
    
    // ...
    
    }  

11.7.3. Spring Data R2DBC Repositories  
Spring Data R2DBC存储库是可以定义以访问数据的接口。查询是根据方法名自动创建的。例如，CityRepository接口可以声明findAllByState（字符串状态）方法来查找给定状态下的所有城市。  
对于更复杂的查询，可以使用Spring数据的查询注释来注释方法。              Spring数据存储库通常从存储库或原始接口扩展。如果使用自动配置，则会从包含主配置类（用@EnableAutoConfiguration或@springbootsapplication注释的配置类）的包中搜索存储库。  
以下示例显示了典型的Spring数据存储库接口定义：  

    package com.example.myapp.domain;
    
    import org.springframework.data.domain.*;
    import org.springframework.data.repository.*;
    import reactor.core.publisher.Mono;
    
    public interface CityRepository extends Repository<City, Long> {
    
    Mono<City> findByNameAndStateAllIgnoringCase(String name, String state);
    
    }  

我们几乎没有触及Spring数据R2DBC的表面。有关完整的详细信息，请参阅Spring Data R2DBC参考文档。