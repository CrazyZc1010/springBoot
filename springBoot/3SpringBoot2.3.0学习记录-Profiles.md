# 配置文件 #  
Spring配置文件提供了一种分离应用程序配置部分并使其仅在特定环境中可用的方法。任何@Component、@Configuration或@ConfigurationProperties都可以用@Profile标记，以便在加载时进行限制，如下例所示：  

    @Configuration(proxyBeanMethods = false)
    @Profile("production")
    public class ProductionConfiguration {
    
    // ...
    
    }  

如果@ConfigurationProperties bean是通过@EnableConfigurationProperties而不是自动扫描注册的，则需要在具有@EnableConfigurationProperties注释的@Configuration类上指定@Profile注释。在扫描@ConfigurationProperties的情况下，可以在@ConfigurationProperties类本身上指定@Profile。  
可以使用spring.profiles.active Environment属性指定哪些配置文件处于活动状态。您可以使用本章前面描述的任何方式指定属性。例如，可以将其包含在application.properties中，如下例所示  

    spring.profiles.active=dev,hsqldb  

也可以在命令行上使用以下开关指定它：-

    spring.profiles.active=dev，hsqldb  

## 3.1添加活动配置文件 ##  
active属性遵循与其他属性相同的排序规则：最高的PropertySource获胜。这意味着您可以在application.properties中指定活动配置文件，然后使用命令行开关替换它们。  
有时，将特定于profil的属性添加到活动配置文件而不是替换它们是很有用的。spring.profiles.include属性可用于无条件添加活动配置文件。SpringApplication入口点还有一个用于设置其他概要文件的Java API（即，在spring.profiles.active属性激活的概要文件之上）。请参阅SpringApplication中的setAdditionalProfiles（）方法。  
例如，当使用开关，--spring.profiles.active=prod运行具有以下属性的应用程序时，proddb和prodmq配置文件也会被激活：  

    ---
    my.property: fromyamlfile
    ---
    spring.profiles: prod
    spring.profiles.include:
      - proddb
      - prodmq  

请记住，可以在YAML文档中定义spring.profiles属性，以确定配置中何时包含此特定文档。有关详细信息，请参见howto.html。  

## 3.2以编程方式设置配置文件   
在应用程序运行之前，可以通过调用SpringApplication.setAdditionalProfiles（…）以编程方式设置活动配置文件。也可以使用Spring的ConfigurableEnvironment接口激活配置文件。  

## 3.3特定于配置文件的配置文件          
application.properties（或application.yml）和通过@ConfigurationProperties引用的文件的特定于配置文件的变体都被视为文件并加载。有关详细信息，请参阅“特定于配置文件的属性”。  


## 4. Logging ##  
Spring Boot使用Commons日志记录所有内部日志记录，但保持底层日志实现打开。为Java Util日志、Log4J2和Logback提供了默认配置。在每种情况下，日志记录器都预先配置为使用控制台输出，还提供可选的文件输出。  
默认情况下，如果使用“Starters”，则使用Logback进行日志记录。还包括适当的Logback路由，以确保使用Java Util日志、Commons日志、Log4J或SLF4J的依赖库都能正常工作。  

有很多可用于Java的日志框架。如果上面的列表看起来很混乱，请不要担心。通常，您不需要更改日志依赖项，而Spring引导默认值工作得很好。  

将应用程序部署到servlet容器或应用程序服务器时，通过Java Util日志API执行的日志不会路由到应用程序的日志中。这将防止容器或已部署到容器的其他应用程序执行的日志记录出现在应用程序的日志中。  


4.1. Log Format  
Spring Boot的默认日志输出类似于以下示例：  

    019-03-05 10:57:51.112  INFO 45469 --- [   main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
    2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]   : Initializing Spring embedded WebApplicationContext
    2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader: Root WebApplicationContext: initialization completed in 1358 ms
    2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean: Mapping servlet: 'dispatcherServlet' to [/]
    2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter'  

输出以下项：    
日期和时间：毫秒精度，易于排序。                
日志级别：错误、警告、信息、调试或跟踪。       
进程ID。                
分隔符，用于区分实际日志消息的开头。                
线程名：用方括号括起来（控制台输出时可能会被截断）。  
记录器名称：这通常是源类名（通常缩写）。                
日志消息。  

Logback没有致命级别。它被映射到错误。  


## 4.2. Console Output ##  
默认的日志配置会在消息写入时将其回显到控制台。默认情况下，将记录错误级别、警告级别和信息级别的消息。您还可以通过使用--debug标志启动应用程序来启用“调试”模式。  

    $ java -jar myapp.jar --debug  

也可以在application.properties中指定debug=true。  
启用调试模式后，将配置一组核心记录器（嵌入式容器、Hibernate和Spring Boot）以输出更多信息。启用调试模式不会将应用程序配置为使用调试级别记录所有消息。  
或者，可以通过使用--trace标志（或者在application.properties中trace=true）启动应用程序来启用“跟踪”模式。这样做可以为选择的核心记录器（嵌入式容器、Hibernate模式生成和整个Spring组合）启用跟踪日志记录。  


### 4.2.1. Color-coded Output ###  
如果终端支持ANSI，则使用颜色输出来帮助可读性。可以将spring.output.ansi.enabled设置为支持的值以覆盖自动检测。  
使用%clr转换字配置颜色编码。在最简单的形式中，转换器根据日志级别为输出着色，如下例所示：  

    %clr(%5p)  

下表描述了日志级别到颜色的映射：  
FATAL，ERROR     Red
WARN             Yellow
INFO，DEBUG，TRACE Green  
或者，可以通过将颜色或样式作为转换选项来指定应使用的颜色或样式。例如，要使文本变为黄色，请使用以下设置：  

    %clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}  
支持以下颜色和样式：  
blue，cyan，faint，green，magenta，red，yellow  


----------
2020/3/22 21:13:18 

----------

### 4.3. File Output   ###  
也可以使用application.properties文件配置SpringApplication。有关详细信息，请参见外部化配置。  
有关配置选项的完整列表，请参阅SpringApplication Javadoc。  
日志文件在达到10 MB时会旋转，与控制台输出一样，默认情况下会记录错误级别、警告级别和信息级别的消息。可以使用logging.file.max-Size属性更改大小限制。除非设置了logging.file.max-history属性，否则默认情况下将保留最近7天的旋转日志文件。可以使用logging.file.total-size-cap限制日志存档的总大小。当日志存档的总大小超过该阈值时，将删除备份。要在应用程序启动时强制清除日志存档，请使用logging.file.clean-history-on-start属性。  
日志属性独立于实际的日志基础结构。因此，特定的配置键（例如logback.configurationFile for logback）不由spring Boot管理。
  
## 4.4条。日志级别 ##
通过使用logging.level.<logger name>=<level>可以在Spring环境（例如，在application.properties中）中设置所有受支持的日志系统的日志级别，其中level是TRACE、DEBUG、INFO、WARN、ERROR、FATAL或OFF之一。可以使用logging.level.root配置根日志记录器。  
以下示例显示了application.properties中的潜在日志记录设置：
  
    logging.level.root=warn
    logging.level.org.springframework.web=debug
    logging.level.org.hibernate=error  
也可以使用环境变量设置日志记录级别。例如，LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG将ORG.SPRINGFRAMEWORK.WEB设置为DEBUG。  

上述方法仅适用于包级日志记录。由于松弛绑定总是将环境变量转换为小写，因此不可能以这种方式为单个类配置日志记录。如果需要为类配置日志记录，可以使用SPRING_APPLICATION_JSON变量。
  
## 4.5条。日志组 ##  
能够将相关的记录器组合在一起，以便可以同时对它们进行配置，这通常是很有用的。例如，您通常可以更改所有与Tomcat相关的记录器的日志记录级别，但是您不容易记住顶级包。  
为了帮助实现这一点，Spring Boot允许您在Spring环境中定义日志组。例如，下面介绍如何通过将“tomcat”组添加到application.properties中来定义它： 

    logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat  

定义后，可以用一行更改组中所有记录器的级别：  

    logging.level.tomcat=TRACE  

Spring Boot包括以下预定义的日志记录组，可以开箱即用：  

web  

org.springframework.core.codec, org.springframework.http, org.springframework.web, org.springframework.boot.actuate.endpoint.web, org.springframework.boot.web.servlet.ServletContextInitializerBeans

sql  

org.springframework.jdbc.core, org.hibernate.SQL, org.jooq.tools.LoggerListener  

## 4.6条。自定义日志配置 ##  

可以通过在类路径上包含适当的库来激活各种日志系统，还可以通过在类路径的根目录中或在由以下Spring环境属性指定的位置提供适当的配置文件来进一步自定义日志系统：logging.config。  
通过使用org.springframework.Boot.logging system属性，可以强制springboot使用特定的日志系统。该值应该是LoggingSystem实现的完全限定类名。您还可以使用值none来完全禁用Spring Boot的日志配置。  

由于日志记录是在创建ApplicationContext之前初始化的，因此无法控制Spring@Configuration文件中@PropertySources的日志记录。更改或完全禁用日志系统的唯一方法是通过系统属性。  
根据您的日志系统，将加载以下文件：  

Depending on your logging system, the following files are loaded:


Logging System                      Customization  
Logback
logback-spring.xml, logback-spring.groovy, logback.xml, or logback.groovy  

Log4j2  
log4j2-spring.xml or log4j2.xml  

JDK (Java Util Logging)    
logging.properties  

**如果可能，我们建议您在日志配置中使用-spring变量（例如，logback-spring.xml而不是logback.xml）。如果使用标准配置位置，Spring无法完全控制日志初始化。  
Java Util日志记录中存在已知的类加载问题，这些问题在从“可执行jar”运行时会导致问题。如果可能的话，我们建议您在从“可执行jar”运行时避免使用它。**   
所有受支持的日志记录系统在分析其配置文件时都可以参考系统属性。有关示例，请参见spring-boot.jar中的默认配置：    
Logback    
Log4j 2
Java实用程序日志记录  

如果要在日志属性中使用占位符，应该使用Spring Boot的语法，而不是底层框架的语法。值得注意的是，如果使用Logback，则应使用：作为属性名与其默认值之间的分隔符，而不是使用：-。  
您可以通过仅用Logback覆盖logu-LEVEL模式（或logging.PATTERN.LEVEL）来将MDC和其他特殊内容添加到日志行。例如，如果使用logging.pattern.level=user:%X{user}%5p，则默认日志格式包含“user”的MDC条目（如果存在）  


----------
2020/3/23 21:39:57 

----------
### 4.7条。日志恢复扩展 ###  
Spring Boot包含了许多对Logback的扩展，可以帮助进行高级配置。您可以在logback-spring.xml配置文件中使用这些扩展名。  
由于标准logback.xml配置文件加载太早，因此不能在其中使用扩展名。您需要使用logback-spring.xml或定义logging.config属性。  
扩展不能与Logback的配置扫描一起使用。如果尝试执行此操作，则对配置文件进行更改会导致类似于以下记录之一的错误：  

    ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
    ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]  

#### 4.7.1条。配置文件特定配置 ####  
<springProfile>标记允许您根据活动的Spring配置文件选择包括或排除配置部分。Profile部分在<configuration>元素中的任何地方都受支持。使用name属性指定接受配置的配置文件。<springProfile>标记可以包含一个简单的配置文件名（例如staging）或一个配置文件表达式。profile表达式允许表达更复杂的profile逻辑，例如production&（eu central | eu west）。查看参考指南了解更多详细信息。下面的列表显示了三个示例配置文件：  

    <springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
    </springProfile>
    
    <springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
    </springProfile>
    
    <springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
    </springProfile>  

#### 4.7.2条。环境属性 ####  
<springProperty>标记允许您公开Spring环境中的属性，以便在Logback中使用。如果您想访问Logback配置中application.properties文件中的值，那么这样做很有用。标记的工作方式与Logback的标准<property>标记类似。但是，不是指定直接值，而是指定属性的源（从环境）。如果需要将属性存储在本地作用域以外的其他位置，则可以使用scope属性。如果需要回退值（如果未在环境中设置属性），则可以使用defaultValue属性。下面的示例演示如何公开属性以在Logback中使用：  

    <springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
    defaultValue="localhost"/>
    <appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
    </appender>  
必须在kebab case中指定源（例如my.property name）。但是，可以使用放宽的规则将属性添加到环境中。  
## 5 国际化 ##  
Spring Boot支持本地化消息，因此您的应用程序可以满足不同语言偏好的用户。默认情况下，Spring Boot会在类路径的根目录中查找消息资源包的存在。  
当配置的资源包的默认属性文件可用时（即默认情况下为messages.properties），将应用自动配置。如果资源包仅包含特定于语言的属性文件，则需要添加默认值。如果找不到与任何配置的基名称匹配的属性文件，则不会有自动配置的消息源。  
可以使用spring.messages命名空间配置资源包的基名以及其他几个属性，如下例所示：  

    spring.messages.basename=messages,config.i18n.messages
    spring.messages.fallback-to-system-locale=false  

basename支持以逗号分隔的位置列表，可以是包限定符，也可以是从类路径根解析的资源。  


## 6. JSON ##  
Spring Boot提供了与三个JSON映射库的集成：  
Gson
Jackson
JSON-B  
Jackson是首选的默认库。  
### 6.1条。Jackson    ###           
提供了Jackson的自动配置，Jackson是springbootstarterjson的一部分。当Jackson在类路径上时，会自动配置一个ObjectMapper bean。为自定义ObjectMapper的配置提供了几个配置属性。  
### 6.2条。Gson   ###    
提供Gson的自动配置。当Gson位于类路径上时，将自动配置Gson bean。提供了几个spring.gson.*配置属性来定制配置。要获得更多控制，可以使用一个或多个GsonBuilderCustomizer bean。   
### 6.3条。JSON-B格式   ###     
提供了JSON-B的自动配置。当JSON-B API和实现在类路径上时，JSON B bean将自动配置。首选的JSON-B实现是Apache Johnzon，它提供了依赖管理。