
# 13. Caching #  
Spring框架支持透明地向应用程序添加缓存。抽象的核心是将缓存应用于方法，从而根据缓存中可用的信息减少执行次数。缓存逻辑的应用是透明的，不会对调用程序造成任何干扰。只要通过@EnableCaching注释启用了缓存支持，Spring Boot就会自动配置缓存基础结构。  
有关更多详细信息，请查看Spring Framework[参考](https://docs.spring.io/spring/docs/5.2.6.BUILD-SNAPSHOT/spring-framework-reference/integration.html#cache)的相关部分  
简而言之，将缓存添加到服务的操作与将相关注释添加到其方法一样简单，如下例所示：  

    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Component;
    
    @Component
    public class MathService {
    
    @Cacheable("piDecimals")
    public int computePiDecimal(int i) {
    // ...
    }
    
    }  
    

此示例演示如何在可能代价高昂的操作上使用缓存。在调用computePiDecimal之前，抽象在piDecimals缓存中查找与i参数匹配的条目。如果找到一个条目，缓存中的内容会立即返回给调用方，并且不会调用该方法。否则，将调用该方法，并在返回值之前更新缓存。  

**您还可以透明地使用标准JSR-107（JCache）注释（例如@CacheResult）。但是，我们强烈建议您不要混合和匹配Spring缓存和JCache注释。**  

如果不添加任何特定的缓存库，Spring Boot会自动配置一个在内存中使用并发映射的简单提供程序。当需要缓存时（如上例中的piDecimals），此提供程序将为您创建缓存。简单的提供程序并不是真正推荐用于生产，但它对于入门和确保您了解这些特性非常有用。当您决定使用缓存提供程序时，请确保阅读其文档以了解如何配置应用程序使用的缓存。几乎所有提供程序都要求您显式配置在应用程序中使用的每个缓存。有些提供了一种自定义由spring.cache.cache-names属性定义的默认缓存的方法。

**也可以透明地从缓存中更新或收回数据。**  

## 13.1. Supported Cache Providers ##  
缓存抽象不提供实际的存储，依赖org.springframework.cache.cache和org.springframework.cache.CacheManager接口实现的抽象。  
如果尚未定义CacheManager类型的bean或名为CacheResolver的CacheResolver（请参阅cachingconfiguer），则Spring Boot将尝试检测以下提供程序（按指示的顺序）：  

1. Generic
1. JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, and others)
1. EhCache 2.x
1. Hazelcast
1. Infinispan
1. Couchbase
1. Redis
1. Caffeine
1. Simple  

**也可以通过设置spring.cache.type属性强制特定的缓存提供程序。如果需要在某些环境（如测试）中完全禁用缓存，请使用此属性 ** 

使用spring boot starter缓存“starter”快速添加基本缓存依赖项。starter提供了spring上下文支持。如果手动添加依赖项，则必须包含spring上下文支持才能使用JCache、EhCache 2.x或咖啡因支持。

如果CacheManager是由Spring Boot自动配置的，则可以通过公开实现CacheManagerCustomizer接口的bean，在完全初始化之前进一步优化其配置。以下示例设置了一个标志，表示应将空值传递给基础映射：  

    @Bean
    public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
    return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
    cacheManager.setAllowNullValues(false);
    }
    };
    }  

在前面的示例中，需要自动配置的ConcurrentMapCacheManager。如果不是这样（您提供了自己的配置或自动配置了其他缓存提供程序），则根本不会调用自定义程序。您可以拥有任意数量的自定义项，也可以使用@order或Ordered命令它们。  

### 13.1.1  通用的 ###  
如果上下文定义了至少一个org.springframework.cache.cache缓存bean。将创建包装该类型所有bean的CacheManager。  

### 13.1.2. JCache (JSR-107) ###  
JCache是通过类路径上的javax.cache.spi.CachingProvider（即类路径上存在一个符合JSR-107的缓存库）来引导的，JCacheCacheManager由spring boot starter缓存“starter”提供。提供了各种兼容的库，Spring Boot为Ehcache 3、Hazelcast和Infinispan提供了依赖性管理。也可以添加任何其他兼容库。  
可能会出现多个提供程序，在这种情况下，必须显式指定提供程序。即使JSR-107标准没有强制使用标准化的方式来定义配置文件的位置，Spring Boot也会尽其所能地使用实现细节来设置缓存，如下例所示： 
 

    # Only necessary if more than one provider is present
    spring.cache.jcache.provider=com.acme.MyCachingProvider
    spring.cache.jcache.config=classpath:acme.xml  

当缓存库同时提供本机实现和JSR-107支持时，Spring Boot更喜欢JSR-107支持，因此如果切换到不同的JSR-107实现，同样的功能也可用。  

Spring Boot 一般支持Hazelcast。如果只有一个HazelcastInstance可用，那么它也会自动为CacheManager重用，除非指定了spring.cache.jcache.config属性。  

有两种方法可以自定义底层javax.cache.cacheManager：  

1. 通过设置spring.cache.cache缓存-命名属性。如果一个custom（顾客） javax.cache.configuration.configuration配置bean被定义，它被用来定制它们。
2. org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer启动通过引用CacheManager来调用bean以进行完全定制。         

如果定义了一个标准的javax.cache.CacheManager bean，它将自动包装在抽象所期望的org.springframework.cache.CacheManager实现中。不会对其应用进一步的自定义。  



----------
2020/5/13 21:49:50 

----------


### 13.1.3. EhCache 2.x ###  
如果可以在类路径的根目录中找到名为EhCache.xml的文件，则使用EhCache 2.x。如果找到EhCache 2.x，则使用spring boot starter cache“starter”提供的EhCache cache manager来引导缓存管理器。还可以提供备用配置文件，如下例所示：  

    spring.cache.ehcache.config=classpath:config/another-config.xml  


### 13.1.4. Hazelcast ###  
Spring Boot一般支持Hazelcast。如果HazelcastInstance已自动配置，则它将自动包装在CacheManager中。  


### 13.1.5. Infinispan  ###   
Infinispan没有默认的配置文件位置，因此必须显式指定它。否则，将使用默认引导。  

    spring.cache.infinispan.config=infinispan.xml  
通过设置spring.cache.cache-names属性，可以在启动时创建缓存。如果定义了自定义ConfigurationBuilder bean，它将用于自定义缓存。  

Infinispan在Spring Boot中的支持仅限于嵌入式模式，是相当基础的。如果你想要更多的选择，你应该使用正式的Infinispan Spring Boot 启动。请参阅Infinispan的文档以了解更多详细信息。  


### 13.1.6. Couchbase ###  
如果Couchbase Java客户机和Couchbase spring缓存实现可用并且Couchbase已配置，则Couchbase缓存管理器将自动配置。还可以通过设置spring.cache.cache-names属性在启动时创建其他缓存。这些缓存在自动配置的Bucket上操作。还可以使用自定义程序在另一个Bucket上创建其他缓存。假设您需要“main”Bucket上的两个缓存（cache1和cache2）和“another”Bucket上的一个（cache3）缓存（自定义生存时间为2秒）。您可以通过配置创建前两个缓存，如下所示：  

    spring.cache.cache-names=cache1,cache2  

然后您可以定义一个@Configuration类来配置额外的Bucket和cache3缓存，如下所示：  

    @Configuration(proxyBeanMethods = false)
    public class CouchbaseCacheConfiguration {
    
    private final Cluster cluster;
    
    public CouchbaseCacheConfiguration(Cluster cluster) {
    this.cluster = cluster;
    }
    
    @Bean
    public Bucket anotherBucket() {
    return this.cluster.openBucket("another", "secret");
    }
    
    @Bean
    public CacheManagerCustomizer<CouchbaseCacheManager> cacheManagerCustomizer() {
    return c -> {
    c.prepareCache("cache3", CacheBuilder.newInstance(anotherBucket())
    .withExpiration(2));
    };
    }
    
    }  

此示例配置重用通过自动配置创建的集群。  


### 13.1.7. Redis ###  

如果Redis可用并已配置，则会自动配置RedisCacheManager。可以通过设置spring.cache.cache-names属性在启动时创建其他缓存，可以使用spring.cache.redis.*属性配置缓存默认值。例如，以下配置创建的cache1和cache2缓存的生存时间为10分钟：  

    spring.cache.cache-names=cache1,cache2
    spring.cache.redis.time-to-live=600000 

默认情况下，会添加一个键前缀，以便如果两个单独的缓存使用同一个键，则Redis没有重叠的键，并且不能返回无效值。如果创建自己的RedisCacheManager，强烈建议保持启用此设置。  
您可以通过添加自己的RedisCacheConfiguration@Bean来完全控制默认配置。如果您要自定义默认序列化策略，则这可能非常有用。  

如果需要对配置进行更多控制，请考虑注册RedisCacheManagerBuilderCustomizer bean。以下示例显示一个自定义程序，该自定义程序为cache1和cache2配置特定的生存时间：

    @Bean
    public RedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
    .withCacheConfiguration("cache1",
    RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofSeconds(10)))
    .withCacheConfiguration("cache2",
    RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(1)));
    
    }
     

### 13.1.8. Caffeine ###  
Caffeine是对Guava’s 缓存的Java8重写，取代了对Guava的支持。如果存在Caffeine，则会自动配置caffinecachemanager（由spring boot starter缓存“starter”提供）通过设置spring.cache.cache-names属性，可以在启动时创建缓存，并且可以通过以下方式之一进行自定义（按指定顺序）：  

1. A cache spec defined by spring.cache.caffeine.spec
1. A com.github.benmanes.caffeine.cache.CaffeineSpec bean is defined
1. A com.github.benmanes.caffeine.cache.Caffeine bean is defined  

例如，以下配置创建最大大小为500且生存时间为10分钟的cache1和cache2缓存

    spring.cache.cache-names=cache1,cache2
    spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s  

如果com.github.benmanes.caffeine.cache.CacheLoaderbean被定义，它自动关联到caffienecachemanager。由于CacheLoader将与缓存管理器管理的所有缓存关联，因此必须将其定义为CacheLoader<Object，Object>。自动配置忽略任何其他泛型类型。




----------
2020/5/15 16:58:19 

----------


### 13.1.9. Simple ###  
如果找不到其他提供程序，则配置一个使用ConcurrentHashMap作为缓存存储的简单实现。如果应用程序中不存在缓存库，则这是默认设置。默认情况下，将根据需要创建缓存，但可以通过设置“缓存名称”属性来限制可用缓存的列表。例如，如果只需要cache1和cache2缓存，请按如下所示设置cache names属性：  

    spring.cache.cache-names=cache1,cache2  

如果这样做，并且应用程序使用未列出的缓存，则当需要缓存时，它将在运行时失败，但不会在启动时失败。这与使用未声明的缓存时“真实”缓存提供程序的行为类似。  


### 13.1.10. None ###  
当@EnableCaching出现在您的配置中时，也需要一个合适的缓存配置。如果需要在某些环境中完全禁用缓存，请强制将缓存类型设置为none以使用no-op实现，如下例所示：

    spring.cache.type=none

