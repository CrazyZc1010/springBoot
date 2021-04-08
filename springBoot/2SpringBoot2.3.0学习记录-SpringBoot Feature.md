# 使用SpringBoot特点001 #     
本节将深入介绍Spring Boot的详细信息。在这里，您可以了解您可能要使用和自定义的关键功能。如果您还没有这样做，那么您可能需要阅读“getting started.html”和“using spring boot.html”部分，以便对基础知识有一个良好的基础。  

## 1. SpringApplication ##  
Spring application类提供了一种方便的方法来引导从main（）方法启动的Spring应用程序。在许多情况下，可以委托给静态SpringApplication.run方法，如下例所示：  

    public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
    }  

默认情况下，将显示信息日志消息，包括一些相关的启动详细信息，例如启动应用程序的用户。如果需要信息以外的日志级别，可以按照日志级别中的说明进行设置。应用程序版本是使用主应用程序类的包中的实现版本确定的。通过将spring.main.log-Startup-info设置为false，可以关闭启动信息日志记录。这还将关闭应用程序活动配置文件的日志记录。  
 
**要在启动期间添加额外的日志记录，可以重写SpringApplication子类中的logStartupInfo（boolean）。**   

### 1.1启动失败 ###  
如果应用程序无法启动，则注册的故障分析器将有机会提供专用的错误消息和解决问题的具体操作。  
**Spring Boot提供了许多FailureAnalyzer实现，您可以添加自己的实现。**  
如果没有故障分析器能够处理异常，您仍然可以显示完整的条件报告，以便更好地了解出了什么问题。为此，需要为org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener启用调试属性或启用调试日志记录。  
例如，如果使用java-jar运行应用程序，可以按如下方式启用debug属性：
  
    $ java -jar myproject-0.0.1-SNAPSHOT.jar --debug  

### 1.2懒加载 ###  
SpringApplication允许惰性地初始化应用程序。当启用惰性初始化时，将根据需要而不是在应用程序启动期间创建bean。因此，启用延迟初始化可以减少应用程序启动所需的时间。在web应用程序中，启用延迟初始化将导致许多与web相关的bean在收到HTTP请求之前无法初始化。  
延迟初始化的一个缺点是，它会延迟发现应用程序的问题。如果错误配置的bean被延迟初始化，则在启动期间将不再发生故障，并且只有在初始化bean时问题才会变得明显。还必须注意确保JVM有足够的内存来容纳应用程序的所有bean，而不仅仅是那些在启动期间初始化的bean。由于这些原因，默认情况下不启用延迟初始化，建议在启用延迟初始化之前对JVM的堆大小进行微调。  
可以使用SpringApplicationBuilder上的Lazy initialization方法或SpringApplication上的setLazyInitialization方法以编程方式启用延迟初始化。或者，可以使用spring.main.lazy-initialization属性启用它，如下例所示： 
 
    spring.main.lazy-initialization=true  

如果要在对应用程序的其余部分使用惰性初始化时禁用某些bean的惰性初始化，可以使用@lazy（false）注释将其lazy属性显式设置为false。  

### 1.3自定义Banner ###  
可以通过向类路径添加banner.txt文件或将spring.banner.location属性设置为此类文件的位置来更改在启动时打印的Banner。如果文件的编码不是UTF-8，则可以设置spring.banner.charset。除了文本文件，还可以将banner.gif、banner.jpg或banner.png图像文件添加到类路径中，或设置spring.banner.image.location属性。图像被转换成ASCII艺术表现形式并打印在任何文本横幅的上方。  
如果要以编程方式生成横幅，可以使用SpringApplication.setBanner（…）方法。使用org.springframework.boot.Banner接口并实现您自己的printbranner（）方法。  
您还可以使用spring.main.banner-mode属性来确定是否必须在System.out（控制台）上打印横幅、发送到配置的记录器（日志）或根本不生成横幅（关闭）。              打印的横幅注册为singleton bean，名称如下：springbootbranner。  
### 1.4自定义SpringApplication ###  
如果SpringApplication的默认值不符合您的口味，那么您可以创建一个本地实例并对其进行自定义。例如，要关闭横幅，可以写入：  

    public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
    }  
传递给SpringApplication的构造函数参数是springbean的配置源。在大多数情况下，这些引用是对@Configuration类的引用，但它们也可以是对XML配置或应该扫描的包的引用。  
也可以使用application.properties文件配置SpringApplication。有关详细信息，请参见外部化配置。              有关配置选项的完整列表，请参阅SpringApplication Javadoc。  

### 1.5.Fluent生成器API ###  
如果需要构建ApplicationContext层次结构（具有父/子关系的多个上下文），或者更喜欢使用“fluent”构建器API，则可以使用SpringApplicationBuilder。              SpringApplicationBuilder允许您将多个方法调用链接在一起，并包含允许您创建层次结构的父方法和子方法，如下例所示：  

    new SpringApplicationBuilder()
    .sources(Parent.class)
    .child(Application.class)
    .bannerMode(Banner.Mode.OFF)
    .run(args);  

**注意：创建ApplicationContext层次结构时有一些限制。例如，Web组件必须包含在子上下文中，并且父上下文和子上下文都使用相同的环境。有关详细信息，请参阅SpringApplicationBuilder Javadoc。**  

### 1.6应用程序事件和侦听器 ###  
除了常见的Spring框架事件（如ContextRefreshedEvent）之外，Spring应用程序还发送一些附加的应用程序事件。  
有些事件实际上是在创建ApplicationContext之前触发的，因此不能将侦听器注册为@Bean。您可以使用SpringApplication.addListeners（…）方法或SpringApplicationBuilder.listeners（…）方法注册它们。  
如果希望自动注册这些侦听器，不管应用程序是如何创建的，都可以将META-INF/spring.factories文件添加到项目中，并使用org.springframework.context.ApplicationListener键引用侦听器，如以下示例所示：   

    org.springframework.context.ApplicationListener=com.example.project.MyListener
    
应用程序运行时，应用程序事件按以下顺序发送：
 
1. ApplicationStartingEvent在运行开始时但在任何处理之前发送，但注册侦听器和初始化器除外。  
1. 当已知要在上下文中使用的环境时，但在创建上下文之前，将发送ApplicationEnvironmentPreparedEvent。  
1. 当ApplicationContext准备好并调用了ApplicationContextInitializedEvent时，但在加载任何bean定义之前，将发送ApplicationContextInitializedEvent。   
1. ApplicationPreparedEvent在刷新开始之前，但在加载bean定义之后发送。  
1. ApplicationStartedEvent在刷新上下文之后、调用任何应用程序和命令行运行程序之前发送。  
1. 在调用任何应用程序和命令行运行程序之后，将发送ApplicationReadyEvent。它表示应用程序已准备好为请求提供服务。  
1. 如果启动时出现异常，则发送ApplicationFailedEvent。  

上面的列表只包括绑定到SpringApplication的SpringApplicationEvents。除此之外，还将在ApplicationPreparedEvent之后和ApplicationStartedEvent之前发布以下事件：  

1. 刷新ApplicationContext时发送ContextRefreshedEvent。              在Web服务器准备就绪后发送WebServerInitializedEvent。  
1. ServletWebServerInitializedEvent和ReactiveWebServerInitializedEvent分别是servlet和reactive变量。  

**
您通常不需要使用应用程序事件，但是知道它们的存在是很方便的。在内部，Spring Boot使用事件来处理各种任务。**  

应用程序事件通过使用Spring框架的事件发布机制发送。此机制的一部分确保在子上下文中发布给侦听器的事件也在任何祖先上下文中发布给侦听器。因此，如果应用程序使用SpringApplication实例的层次结构，则侦听器可能会接收同一类型应用程序事件的多个实例。  

为了允许侦听器区分其上下文的事件和子上下文的事件，它应该请求注入其应用程序上下文，然后将注入的上下文与事件的上下文进行比较。上下文可以通过实现ApplicationContextAware注入，如果侦听器是bean，则可以使用@Autowired注入。  

### 1.7Web环境 ###  
SpringApplication试图代表您创建正确类型的ApplicationContext。用于确定WebApplicationType的算法相当简单：  
1. 如果存在Spring MVC，则使用注释configservletwebserverapplicationcontext  
1. 如果Spring MVC不存在并且Spring WebFlux存在，则使用注释configreactivewebserverapplicationcontext  
1. 否则，将使用AnnotationConfigApplicationContext  

这意味着，如果您在同一个应用程序中使用Spring MVC和Spring WebFlux中的新WebClient，那么默认情况下将使用springmvc。您可以通过调用setWebApplicationType（WebApplicationType）轻松地覆盖它。  
还可以完全控制调用setApplicationContextClass（…）所使用的ApplicationContext类型。  

**在JUnit测试中使用SpringApplication时，通常需要调用setWebApplicationType（WebApplicationType.NONE）。**  

### 1.8访问应用程序参数 ###  
如果需要访问传递给SpringApplication.run（…）的应用程序参数，可以插入org.springframework.boot.application arguments bean。ApplicationArguments接口提供对原始字符串[]参数以及已分析的选项和非选项参数的访问，如下例所示：  

    import org.springframework.boot.*;
    import org.springframework.beans.factory.annotation.*;
    import org.springframework.stereotype.*;
    
    @Component
    public class MyBean {
    
    @Autowired
    public MyBean(ApplicationArguments args) {
    boolean debug = args.containsOption("debug");
    List<String> files = args.getNonOptionArgs();
    // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
    
    }  

**Spring Boot还向Spring环境注册CommandLinePropertySource。这还允许您使用@Value注释注入单个应用程序参数。**  

### 1.9使用ApplicationRunner或CommandLineRunner ###  
如果在SpringApplication启动后需要运行某些特定代码，可以实现ApplicationRunner或CommandLineRunner接口。这两个接口以相同的方式工作，并提供一个单独的run方法，该方法在SpringApplication.run（…）完成之前调用。  
CommandLineRunner接口以简单的字符串数组形式提供对应用程序参数的访问，而ApplicationRunner使用前面讨论的ApplicationArguments接口。以下示例显示了带有run方法的CommandLineRunner：  

    import org.springframework.boot.*;
    import org.springframework.stereotype.*;
    
    @Component
    public class MyBean implements CommandLineRunner {
    
    public void run(String... args) {
    // Do something...
    }
    
    }  

如果定义了几个必须按特定顺序调用的CommandLineRunner或ApplicationRunner bean，则可以另外实现org.springframework.core.Ordered接口或使用org.springframework.core.annotation.order注释。  

### 1.10应用程序退出 ###  
每个SpringApplication都向JVM注册一个关闭钩子，以确保ApplicationContext在退出时正常关闭。可以使用所有标准的Spring生命周期回调（例如DisposableBean接口或@PreDestroy注释）。  
此外，如果bean希望在调用SpringApplication.exit（）时返回特定的退出代码，则可以实现org.springframework.boot.ExitCodeGenerator接口。然后，可以将此退出代码传递给System.exit（）以将其作为状态代码返回，如下例所示：  

    @SpringBootApplication
    public class ExitCodeApplication {
    
    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
    return () -> 42;
    }
    
    public static void main(String[] args) {
    System.exit(SpringApplication.exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }
    
    }  

此外，ExitCodeGenerator接口可以通过异常实现。当遇到这种异常时，Spring Boot返回由实现的getExitCode（）方法提供的退出代码。  

### 1.11管理功能   ###

通过指定spring.application.admin.enabled属性，可以为应用程序启用与管理相关的功能。这将在MBeanServer平台上公开SpringApplicationAdminMXBean。您可以使用此功能远程管理您的Spring Boot应用程序。这个特性对于任何服务包装器实现都是有用的。  
如果要知道应用程序在哪个HTTP端口上运行，请获取键为local.server.port的属性。  
  
### 2外部化配置 ###  
Spring Boot允许您将配置外部化，以便可以在不同的环境中使用相同的应用程序代码。可以使用属性文件、YAML文件、环境变量和命令行参数将配置外部化。属性值可以通过@Value注释直接注入bean，通过Spring的环境抽象访问，或者通过@ConfigurationProperties绑定到结构化对象。  
Spring Boot使用一个非常特殊的PropertySource顺序，该顺序被设计为允许对值进行合理的重写。属性按以下顺序考虑：
  
1. Devtools处于活动状态时，$HOME/.config/spring boot文件夹中的Devtools全局设置属性。  
1. @TestPropertySource对测试的注释。  
1. 测试的属性。在@SpringBootTest和测试注释中提供，用于测试应用程序的特定部分。  
1. 命令行参数。  
1. 来自SPRING_APPLICATION_JSON（内嵌在环境变量或系统属性中的JSON）的属性。  
1. ServletConfig初始化参数。   
1. ServletContext初始化参数。  
1. 来自java的JNDI属性：comp/env。   
1. Java系统属性（System.getProperties（））。  
1. 操作系统环境变量  
1. RandomValuePropertySource，其属性仅为random.*。  
1. 打包jar之外的特定于概要文件的应用程序属性（application-{Profile}.properties和YAML变量）。  
1. 打包在jar中的特定于概要文件的应用程序属性（application-{Profile}.properties和YAML变量）。  
1. 打包jar之外的应用程序属性（Application.properties和YAML变体）。  
1. 打包在jar中的应用程序属性（Application.properties和YAML变体）。  
1. @Configuration类上的PropertySource注释。请注意，在刷新应用程序上下文之前，这些属性源不会添加到环境中。现在配置某些属性（如logging.*和spring.main.*等）已经太迟了，这些属性在刷新开始之前就会被读取。  
1. 默认属性（通过设置SpringApplication.setDefaultProperties指定）。  

为了提供一个具体的示例，假设您开发了一个使用name属性的@Component，如下例所示：  

    import org.springframework.stereotype.*;
    import org.springframework.beans.factory.annotation.*;
    
    @Component
    public class MyBean {
    
    @Value("${name}")
    private String name;
    
    // ...
    
    }  

在应用程序类路径（例如，在jar中）上，可以有一个application.properties文件，该文件为name提供合理的默认属性值。在新环境中运行时，可以在jar外部提供application.properties文件，该文件覆盖名称。对于一次性测试，可以使用特定的命令行开关启动（例如，java-jar app.jar--name=“Spring”）。  

在加载配置文件时，Spring Boot还支持通配符位置。默认情况下，支持jar外部的config/*/通配符位置。指定spring.config.additional-location和spring.config.location时也支持通配符位置。  

当存在多个配置属性源时，通配符位置在Kubernetes等环境中特别有用。例如，如果您有一些Redis配置和一些MySQL配置，您可能希望将这两个配置分开，同时要求这两个配置都存在于应用程序可以绑定到的application.properties中。这可能会导致在不同的位置安装两个单独的application.properties文件，例如/config/redis/application.properties和/config/mysql/application.properties。在这种情况下，通配符位置为config/*/，将导致两个文件都被处理。   


**注意:具有通配符的位置不会以确定的顺序进行处理，并且与通配符匹配的文件不能用于重写另一个位置中的键。**  

SPRING_APPLICATION_JSON属性可以在带有环境变量的命令行上提供。例如，可以在UN*X shell中使用以下行：  

    $ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar  

在前面的示例中，您最终在Spring环境中使用acme.name=test。您还可以在系统属性中将JSON作为spring.application.JSON提供，如下例所示：  

    $ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar

还可以使用命令行参数提供JSON，如下例所示：  

    $ java -jar myapp.jar --spring.application.json='{"name":"test"}'  

您还可以将JSON作为JNDI变量提供，如下所示： 
 
    java:comp/env/spring.application.json.  

#### 2.1配置随机值 ####  
RandomValuePropertySource对于注入随机值（例如，在机密或测试用例中）非常有用。它可以生成整数、long、uuid或字符串，如下例所示：   

    my.secret=${random.value}
    my.number=${random.int}
    my.bignumber=${random.long}
    my.uuid=${random.uuid}
    my.number.less.than.ten=${random.int(10)}
    my.number.in.range=${random.int[1024,65536]}  

random.int*语法是OPEN value（，max）CLOSE，OPEN，CLOSE是任何字符和值，max是整数。如果提供max，则value是最小值，max是最大值（独占）。  

### 2.2访问命令行属性 ###   
默认情况下，SpringApplication将任何命令行选项参数（即以--开头的参数，例如--server.port=9000）转换为属性，并将它们添加到Spring环境中。如前所述，命令行属性始终优先于其他属性源。  
如果不希望命令行属性添加到环境中，可以使用SpringApplication.setAddCommandLineProperties（false）禁用它们。  

#### 2.3应用程序属性文件 ####  

Spring application从application.properties文件加载以下位置的属性，并将其添加到Spring环境中：  

当前目录的A/config子目录  
当前目录  
类路径/配置包  
类路径根  

列表按优先级排序（在列表中较高位置定义的属性将覆盖在较低位置定义的属性）。  

也可以使用YAML（'.yml'）文件作为“.properties”的替代。  

如果不喜欢将application.properties作为配置文件名，可以通过指定spring.config.name环境属性切换到另一个文件名。还可以使用spring.config.location环境属性（目录位置或文件路径的逗号分隔列表）引用显式位置。以下示例演示如何指定其他文件名：  

    $ java -jar myproject.jar --spring.config.name=myproject  

The following example shows how to specify two locations:  

    $ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties  

spring.config.name和spring.config.location很早就用于确定必须加载哪些文件。它们必须定义为环境属性（通常是OS环境变量、系统属性或命令行参数）。  

如果spring.config.location包含目录（而不是文件），则它们应该以/（并且，在运行时，在加载之前，应该附加从spring.config.name生成的名称，包括特定于配置文件的文件名）。在spring.config.location中指定的文件按原样使用，不支持特定于配置文件的变量，并且由任何特定于配置文件的属性重写。  

按相反的顺序搜索配置位置。默认情况下，配置的位置是classpath:/，classpath:/config/，file:./，file:./config/*/，file:./config/。结果搜索顺序如下：  

file:./config/
file:./config/*/
file:./
classpath:/config/
classpath:/  

当使用spring.config.location配置自定义配置位置时，它们将替换默认位置。例如，如果使用值classpath:/custom config/，file:./custom config/配置spring.config.location，则搜索顺序如下：  

file:./custom-config/
classpath:custom-config/  

或者，当使用spring.config.additional-location配置自定义配置位置时，除了默认位置之外，还将使用它们。在默认位置之前搜索其他位置。例如，如果配置了classpath:/custom config/、file:。/custom config/的其他位置，则搜索顺序如下：  

file:./custom-config/
classpath:custom-config/
file:./config/
file:./config/*/
file:./
classpath:/config/
classpath:/  

此搜索顺序允许您在一个配置文件中指定默认值，然后有选择地重写另一个配置文件中的这些值。您可以在其中一个默认位置的application.properties（或使用spring.config.name选择的任何其他基名称）中为应用程序提供默认值。然后，可以在运行时使用位于其中一个自定义位置的其他文件覆盖这些默认值。  

如果使用环境变量而不是系统属性，大多数操作系统都不允许使用句点分隔的键名，但是可以使用下划线（例如，spring.config.name而不是spring.config.name）。  

如果应用程序在容器中运行，那么可以使用JNDI属性（在java:comp/env中）或servlet上下文初始化参数来代替环境变量或系统属性，或者也可以使用环境变量或系统属性。  


----------


2020/3/16 19:27:49 

----------  


### 2.4特定于配置文件的属性 ###  

除了application.properties文件外，还可以使用以下命名约定定义特定于配置文件的属性：application-{profile}.properties。环境有一组默认配置文件（默认情况下，[默认]），如果未设置活动配置文件，则使用这些文件。换句话说，如果没有显式激活配置文件，则加载application-default.properties中的属性。  

特定于配置文件的属性与标准application.properties从相同的位置加载，无论特定于配置文件的文件是否在打包的jar内部或外部，特定于配置文件的文件始终覆盖非特定的文件。   

**如果指定了多个配置文件，则应用最后一个wins策略。例如，spring.profiles.active属性指定的配置文件将添加到通过SpringApplication API配置的配置文件之后，因此具有优先权**

**如果在spring.config.location中指定了任何文件，则不考虑这些文件的特定于配置文件的变体。如果还想使用特定于配置文件的属性，请使用spring.config.location中的目录。**
  

### 2.5属性中的占位符 ###  
application.properties中的值在使用时通过现有环境进行筛选，因此您可以引用以前定义的值（例如，从系统属性）。  

    app.name=MyApp
    app.description=${app.name} is a Spring Boot application  

**您还可以使用此技术创建现有Spring Boot属性的“短”变体。有关详细信息，请参阅how to.html how-to。**  

### 2.6加密属性 ###   
Spring Boot不提供任何内置的对加密属性值的支持，但是，它提供了修改Spring环境中包含的值所必需的挂接点。EnvironmentPostProcessor接口允许您在应用程序启动之前操作环境。有关详细信息，请参见howto.html。  
              
如果您正在寻找一种安全的方式来存储凭据和密码，那么Spring Cloud Vault项目将为在HashiCorp Vault中存储外部化配置提供支持。  


### 2.7使用YAML代替Properties ###  

YAML是JSON的超集，因此，它是一种用于指定分层配置数据的方便格式。只要类路径上有SnakeYAML库，SpringApplication类就会自动支持YAML作为属性的替代。  

如果您使用“Starters”，SnakeYAML将由spring boot starter自动提供。   

#### 2.7.1. Loading YAML ####  
Spring框架提供了两个方便的类，可用于加载YAML文档。YamlPropertiesFactoryBean将YAML加载为属性，YamlMapFactoryBean将YAML加载为映射。                
例如，考虑以下YAML文档：  

 前面的示例将转换为以下属性：  

    environments.dev.url=https://dev.example.com
    environments.dev.name=Developer Setup
    environments.prod.url=https://another.example.com
    environments.prod.name=My Cool App  

YAML列表用[index]解引用程序表示为属性键。例如，考虑以下YAML：  

    my:
       servers:
       - dev.example.com
       - another.example.com   

前面的示例将转换为这些属性：  

    my.servers[0]=dev.example.com
    my.servers[1]=another.example.com  

要通过使用Spring Boot的Binder实用程序（这就是@ConfigurationProperties所做的）绑定到类似的属性，需要在目标bean中有一个java.util.List（或Set）类型的属性，并且需要提供一个setter或用一个可变值初始化它。例如，以下示例绑定到前面显示的属性：  

    @ConfigurationProperties(prefix="my")
    public class Config {
    
    private List<String> servers = new ArrayList<String>();
    
    public List<String> getServers() {
    return this.servers;
    }
    }  

#### 2.7.2在Spring环境中将YAML作为属性公开 ####  

YamlPropertySourceLoader类可用于在Spring环境中将YAML公开为PropertySource。这样做可以使用@Value注释和占位符语法来访问YAML属性。  

#### 2.7.3多配置文件YAML文档 ####   

    server:
    address: 192.168.1.100
    ---
    spring:
    profiles: development
    server:
    address: 127.0.0.1
    ---
    spring:
    profiles: production & eu-central
    server:
    address: 192.168.1.120  

在前面的示例中，如果开发配置文件处于活动状态，则server.address属性为127.0.0.1。类似地，如果production和eu central概要文件处于活动状态，则server.address属性为192.168.1.120。如果未启用开发、生产和欧盟中心配置文件，则属性值为192.168.1.100。 
 
因此，profiles可以包含一个简单的概要文件名（例如production）或概要文件表达式。profile表达式允许表达更复杂的profile逻辑，例如production&（eu central | eu west）。查看参考指南了解更多详细信息。  

如果应用程序上下文启动时没有显式激活，则将激活默认配置文件。因此，在下面的YAML中，我们为spring.security.user.password设置了一个仅在“默认”配置文件中可用的值：  

    server:
      port: 8000
    ---
    spring:
      profiles: default
      security:
    user:
      password: weak  

但是，在下面的示例中，始终设置密码，因为它没有附加到任何配置文件，并且必须在所有其他配置文件中根据需要显式重置密码:  

    server:
      port: 8000
    spring:
      security:
    user:
      password: weak  

使用Spring.profiles元素指定的Spring配置文件可以通过使用！性格。如果为单个文档同时指定了否定配置文件和非否定配置文件，则必须至少有一个非否定配置文件匹配，并且不能有否定配置文件匹配。  

#### 2.7.4YAML缺点 ####  

无法使用@PropertySource注释加载YAML文件。因此，如果需要以这种方式加载值，则需要使用属性文件。   

在特定于概要文件的YAML文件中使用multi-YAML文档语法可能会导致意外行为。例如，考虑文件中的以下配置：  

application-dev.yml  

    server:
      port: 8000
    ---
    spring:
      profiles: "!test"
      security:
    user:
      password: "secret"  

如果使用参数spring.profiles.active=dev运行应用程序，您可能希望security.user.password设置为“secret”，但事实并非如此。  
将筛选嵌套文档，因为主文件名为application-dev.yml。它已经被认为是特定于配置文件的，嵌套文档将被忽略。   

**我们建议您不要将特定于配置文件的YAML文件和多个YAML文档混合使用。坚持只使用其中一个。**  



----------
2020/3/17 17:27:49  


----------

### 2.8安全类型配置属性 ###  

使用@Value（${property}）注释注入配置属性有时会很麻烦，特别是在处理多个属性或数据本质上是分层的情况下。Spring Boot提供了另一种处理属性的方法，这种方法允许强类型bean控制和验证应用程序的配置。  

另请参见@Value和类型安全配置属性之间的区别。  

#### 2.8.1JavaBean属性绑定 ####  

可以绑定声明标准JavaBean属性的bean，如下例所示：  


    package com.example;
    
    import java.net.InetAddress;
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;
    
    import org.springframework.boot.context.properties.ConfigurationProperties;
    
    @ConfigurationProperties("acme")
    public class AcmeProperties {
    
    private boolean enabled;
    
    private InetAddress remoteAddress;
    
    private final Security security = new Security();
    
    public boolean isEnabled() { ... }
    
    public void setEnabled(boolean enabled) { ... }
    
    public InetAddress getRemoteAddress() { ... }
    
    public void setRemoteAddress(InetAddress remoteAddress) { ... }
    
    public Security getSecurity() { ... }
    
    public static class Security {
    
    private String username;
    
    private String password;
    
    private List<String> roles = new ArrayList<>(Collections.singleton("USER"));
    
    public String getUsername() { ... }
    
    public void setUsername(String username) { ... }
    
    public String getPassword() { ... }
    
    public void setPassword(String password) { ... }
    
    public List<String> getRoles() { ... }
    
    public void setRoles(List<String> roles) { ... }
    
    }
    }


前面的POJO定义了以下属性： 
 
1. 启用acme.enabled，默认值为false。  
1. acme.remote-address，可以从字符串强制转换其类型。  
1. acme.security.username，具有嵌套的“security”对象，其名称由属性的名称确定。特别是，返回类型根本没有在那里使用，可能是securityproperty。  
1. acme.security.password.密码。  
1. acme.security.roles，默认为USER的字符串集合。  

映射到Spring Boot中可用的@ConfigurationProperties类的属性（通过属性文件、YAML文件、环境变量等配置）是公共API，但类本身的访问器（getter/setter）不是直接使用的。  

这种安排依赖于默认的空构造函数，getter和setter通常是必需的，因为绑定是通过标准的Java Beans属性描述符进行的，就像Spring MVC一样。在下列情况下，可省略setter：
  
1. 映射只要被初始化，就需要一个getter，但不一定是setter，因为绑定器可以对它们进行修改。  
1. 可以通过索引（通常使用YAML）或使用单个逗号分隔的值（属性）访问集合和数组。在后一种情况下，setter是强制性的。我们建议始终为此类类型添加setter。如果初始化集合，请确保它不是不可变的（如前一个示例所示）。  
1. 如果嵌套的POJO属性已初始化（与前面示例中的Security字段类似），则不需要setter。如果希望活页夹使用其默认构造函数动态创建实例，则需要一个setter。  

有些人使用Project Lombok自动添加getter和setter。确保Lombok不会为此类类型生成任何特定构造函数，因为容器会自动使用它来实例化对象。  

最后，只考虑标准的Java Bean属性，不支持对静态属性的绑定。  

#### 2.8.2构造函数绑定 ####  
上一节中的示例可以以不变的方式重写，如下例所示：  

    package com.example;
    
    import java.net.InetAddress;
    import java.util.List;
    
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.context.properties.ConstructorBinding;
    import org.springframework.boot.context.properties.DefaultValue;
    
    @ConstructorBinding
    @ConfigurationProperties("acme")
    public class AcmeProperties {
    
    private final boolean enabled;
    
    private final InetAddress remoteAddress;
    
    private final Security security;
    
    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
    }
    
    public boolean isEnabled() { ... }
    
    public InetAddress getRemoteAddress() { ... }
    
    public Security getSecurity() { ... }
    
    public static class Security {
    
    private final String username;
    
    private final String password;
    
    private final List<String> roles;
    
    public Security(String username, String password,
    @DefaultValue("USER") List<String> roles) {
    this.username = username;
    this.password = password;
    this.roles = roles;
    }
    
    public String getUsername() { ... }
    
    public String getPassword() { ... }
    
    public List<String> getRoles() { ... }
    
    }
    
    }  

在此设置中，@constructor binding注释用于指示应使用构造函数绑定。这意味着绑定器将期望找到一个具有希望绑定的参数的构造函数。  

@ConstructorBinding类的嵌套成员（如上例中的安全性）也将通过其构造函数绑定。  

可以使用@DefaultValue指定默认值，并应用相同的转换服务将字符串值强制为缺少属性的目标类型。  

要使用构造函数绑定，必须使用@EnableConfigurationProperties或配置属性扫描启用类。不能对由常规Spring机制创建的Bean使用构造函数绑定（例如@Component Bean、通过@Bean方法创建的Bean或使用@Import加载的Bean）  

如果类有多个构造函数，也可以直接在应该绑定的构造函数上使用@ConstructorBinding。  

#### 2.8.3启用@ConfigurationProperties注释类型 ####  

Spring Boot提供了绑定@ConfigurationProperties类型并将其注册为bean的基础结构。可以逐个类启用配置属性，也可以启用与组件扫描类似的配置属性扫描  

有时，用@ConfigurationProperties注释的类可能不适合扫描，例如，如果您正在开发自己的自动配置或希望有条件地启用它们。在这些情况下，请使用@EnableConfigurationProperties注释指定要处理的类型列表。这可以在任何@Configuration类上完成，如下例所示：  

    @Configuration(proxyBeanMethods = false)
    @EnableConfigurationProperties(AcmeProperties.class)
    public class MyConfiguration {
    }  

要使用配置属性扫描，请将@configurationpropertiescan注释添加到应用程序中。通常，它被添加到用@springbootsapplication注释的主应用程序类中，但可以添加到任何@Configuration类中。默认情况下，将从声明注释的类的包中进行扫描。如果要定义要扫描的特定包，可以按以下示例所示执行此操作：  

    @SpringBootApplication
    @ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
    public class MyApplication {
    }  

当使用配置属性扫描或通过@EnableConfigurationProperties注册@ConfigurationProperties bean时，bean有一个常规名称：<prefix>-<fqn>，其中<prefix>是@ConfigurationProperties注释中指定的环境键前缀，<fqn>是bean的完全限定名。如果注释没有提供任何前缀，则只使用bean的完全限定名。  
上面例子中的bean名是acme-com.example.AcmeProperties。  

我们建议@ConfigurationProperties只处理环境，特别是不要从上下文中注入其他bean。对于角点情况，可以使用setter注入或框架提供的任何*感知接口（例如，如果需要访问环境，则使用Environment Aware）。如果仍要使用构造函数注入其他bean，则必须使用@Component注释配置属性bean，并使用基于JavaBean的属性绑定。  

#### 2.8.4使用@ConfigurationProperties注释类型 ####  

这种类型的配置在SpringApplication外部YAML配置中特别适用，如下例所示：  

    # application.yml
    
    acme:
    remote-address: 192.168.1.1
    security:
    username: admin
    roles:
      - USER
      - ADMIN
    
    # additional configuration as required  

要使用@ConfigurationProperties bean，可以以与任何其他bean相同的方式注入它们，如下例所示：  

    @Service
    public class MyService {
    
    private final AcmeProperties properties;
    
    @Autowired
    public MyService(AcmeProperties properties) {
    this.properties = properties;
    }
    
    //...
    
    @PostConstruct
    public void openConnection() {
    Server server = new Server(this.properties.getRemoteAddress());
    // ...
    }
    
    }  
      

使用@ConfigurationProperties还可以生成元数据文件，IDE可以使用这些文件为自己的密钥提供自动完成功能。详见附件。  

#### 2.8.5第三方配置 ####  
除了使用@ConfigurationProperties注释类之外，还可以在public@Bean方法上使用它。如果要将属性绑定到不在您控制范围内的第三方组件，那么这样做特别有用。  
要从环境属性配置bean，请将@ConfigurationProperties添加到其bean注册中，如下例所示：  

    @ConfigurationProperties(prefix = "another")
    @Bean
    public AnotherComponent anotherComponent() {
    ...
    }  

用另一个前缀定义的任何JavaBean属性都映射到另一个组件bean上，映射方式类似于前面的AcmeProperties示例。  

#### 2.8.6 宽松的数据绑定 ####  
Spring Boot使用一些宽松的规则将环境属性绑定到@ConfigurationProperties be an，因此环境属性名和bean属性名之间不需要完全匹配。这很有用的常见示例包括短划线分隔的环境属性（例如，上下文路径绑定到上下文路径）和大写的环境属性（例如，端口绑定到端口）。  
例如，请考虑以下@ConfigurationProperties类：

    @ConfigurationProperties(prefix="acme.my-project.person")
    public class OwnerProperties {
    
    private String firstName;
    
    public String getFirstName() {
    return this.firstName;
    }
    
    public void setFirstName(String firstName) {
    this.firstName = firstName;
    }
    
    }  

使用前面的代码，可以使用以下属性名称： 
 
![](https://i.imgur.com/5XxFuYa.png)

注释的前缀值必须是kebab大小写（小写并用-分隔，例如acme.my project.person）。

![](https://i.imgur.com/vxVBqKT.png)  

我们建议尽可能将属性存储为小写的烤肉串格式，例如my.property name=acme。  

绑定到映射属性时，如果键包含除小写字母数字字符或-以外的任何字符，则需要使用括号表示法，以便保留原始值。如果键没有被[]包围，则将删除不是字母数字或-的任何字符。例如，考虑将以下属性绑定到映射：
  
    acme:
      map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3  

上面的属性将绑定到以/key1、/key2和key3作为映射中键的映射。  

对于YAML文件，括号需要用引号括起来，以便正确解析密钥。  

#### 2.8.7合并复杂类型 ####  
当在多个位置配置列表时，重写通过替换整个列表来工作。              例如，假设一个MyPojo对象的name和description属性默认为空。下面的示例公开来自AcmeProperties的MyPojo对象列表：   

    @ConfigurationProperties("acme")
    public class AcmeProperties {
    
    private final List<MyPojo> list = new ArrayList<>();
    
    public List<MyPojo> getList() {
    return this.list;
    }
    
    }   

考虑以下配置：  
  
    acme:
      list:
    - name: my name
      description: my description
    ---
    spring:
      profiles: dev
    acme:
      list:
    - name: my another name  

如果dev概要文件未处于活动状态，则AcmeProperties.list包含一个MyPojo条目，如前所定义。但是，如果启用了dev配置文件，则列表仍然只包含一个条目（名称为my another name，说明为空）。此配置不会向列表中添加第二个MyPojo实例，也不会合并项。  
在多个配置文件中指定列表时，将使用优先级最高的配置文件（且仅使用该配置文件）。请考虑以下示例：  

    acme:
      list:
    - name: my name
      description: my description
    - name: another name
      description: another description
    ---
    spring:
      profiles: dev
    acme:
      list:
    - name: my another name  

在前面的示例中，如果dev概要文件处于活动状态，那么AcmeProperties.list包含一个MyPojo条目（名称为我的另一个名称，说明为空）。对于YAML，逗号分隔列表和YAML列表都可以用于完全重写列表的内容。  
对于映射属性，可以使用从多个源绘制的属性值进行绑定。但是，对于多个源中的同一属性，将使用优先级最高的属性。以下示例公开来自AcmeProperties的Map<String，MyPojo>：  

    @ConfigurationProperties("acme")
    public class AcmeProperties {
    
    private final Map<String, MyPojo> map = new HashMap<>();
    
    public Map<String, MyPojo> getMap() {
    return this.map;
    }
    
    }  

考虑以下配置：  

    acme:
      map:
    key1:
      name: my name 1
      description: my description 1
    ---
    spring:
      profiles: dev
    acme:
      map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2  

如果dev配置文件未处于活动状态，则AcmeProperties.map包含一个键为key1的条目（名称为my name 1，说明为my description 1）。但是，如果启用了dev profile，那么map包含两个条目，键key1（名称为dev name 1，说明为my description 1）和key2（名称为dev name 2，说明为dev description 2）。  

**前面的合并规则适用于所有属性源的属性，而不仅仅是YAML文件。**  


----------
2020/3/19 19:10:09 


----------
#### 2.8.8 属性转换 ####  
Spring Boot试图在绑定到@ConfigurationProperties bean时将外部应用程序属性强制为正确的类型。如果需要自定义类型转换，可以提供ConversionService bean（带有名为ConversionService的bean）或自定义属性编辑器（通过CustomEditorConfigurer bean）或自定义转换器（bean定义注释为@ConfigurationPropertiesBinding）。   
由于此bean在应用程序生命周期的早期被请求，请确保限制ConversionService正在使用的依赖项。通常，您需要的任何依赖项在创建时都可能未完全初始化。如果配置键强制不需要自定义转换服务，并且仅依赖于使用@ConfigurationPropertiesBinding限定的自定义转换器，则可能需要重命名该服务。  
  
#### 2.8.8.1转换持续时间 ####  
SpringBoot对表示持续时间有专门的支持。如果公开java.time.Duration属性，则应用程序属性中的以下格式可用：
  
1. 常规的长表示（除非指定了@DurationUnit，否则使用毫秒作为默认单位）  
1. java.time.Duration使用的标准ISO-8601格式  
1. 一种更可读的格式，其中值和单位是耦合的（例如，10s表示10秒）
  
请考虑以下示例：  

    @ConfigurationProperties("app.system")
    public class AppSystemProperties {
    
    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);
    
    private Duration readTimeout = Duration.ofMillis(1000);
    
    public Duration getSessionTimeout() {
    return this.sessionTimeout;
    }
    
    public void setSessionTimeout(Duration sessionTimeout) {
    this.sessionTimeout = sessionTimeout;
    }
    
    public Duration getReadTimeout() {
    return this.readTimeout;
    }
    
    public void setReadTimeout(Duration readTimeout) {
    this.readTimeout = readTimeout;
    }
    
    }  

要指定30秒的会话超时，30、PT30S和30s都是等效的。读取超时500ms可以用以下任何形式指定：500、PT0.5S和500ms。  
也可以使用任何受支持的单元。这些是：  

    ns for nanoseconds   纳秒 
    us for microseconds  微秒 
    ms for milliseconds  毫秒 
    s for seconds        秒
    m for minutes        分钟
    h for hours          小时 
    d for days           天   

默认单位是毫秒，可以使用@DurationUnit重写，如上面的示例所示。  

**注意:如果您是从简单使用Long来表示持续时间的以前版本升级，请确保定义单位（使用@duration unit），前提是它不是切换到持续时间旁边的毫秒数。这样做提供了一个透明的升级路径，同时支持更丰富的格式。**  

#### 2.8.8.2转换数据大小 ####  
Spring框架有一个DataSize值类型，它以字节表示大小。如果公开DataSize属性，则应用程序属性中的以下格式可用：
  
1. 常规的长表示（除非指定了@DataSizeUnit，否则使用字节作为默认单位）  
1. 一种更可读的格式，其中值和单位是耦合的（例如，10MB表示10兆字节）
  
请考虑以下示例：  

    @ConfigurationProperties("app.io")
    public class AppIoProperties {
    
    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);
    
    private DataSize sizeThreshold = DataSize.ofBytes(512);
    
    public DataSize getBufferSize() {
    return this.bufferSize;
    }
    
    public void setBufferSize(DataSize bufferSize) {
    this.bufferSize = bufferSize;
    }
    
    public DataSize getSizeThreshold() {
    return this.sizeThreshold;
    }
    
    public void setSizeThreshold(DataSize sizeThreshold) {
    this.sizeThreshold = sizeThreshold;
    }
    
    }  

要指定10兆字节的缓冲区大小，10兆字节和10兆字节是等效的。256字节的大小阈值可以指定为256或256B。  
也可以使用任何受支持的单元。这些是：  

    B for bytes
    KB for kilobytes    千字节
    MB for megabytes    兆字节
    GB for gigabytes    千兆字节
    TB for terabytes  

默认单位是字节，可以使用@DataSizeUnit重写，如上面的示例所示。  
如果您要从以前的版本升级，而以前的版本只是使用Long来表示大小，请确保定义单元（使用@DataSize unit），如果它不是字节，则切换到DataSize。这样做提供了一个透明的升级路径，同时支持更丰富的格式。  

### 2.8.9 @ConfigurationProperties验证   ###
每当使用Spring的@Validated注释对@ConfigurationProperties类进行注释时，Spring Boot就会尝试验证它们。您可以直接在配置类上使用JSR-303javax.validation约束注释。为此，请确保类路径上有一个兼容的JSR-303实现，然后将约束注释添加到字段中，如下例所示：  

    @ConfigurationProperties(prefix="acme")
    @Validated
    public class AcmeProperties {
    
    @NotNull
    private InetAddress remoteAddress;
    
    // ... getters and setters
    
    }  

您还可以通过注释@Bean方法来触发验证，该方法使用@Validated创建配置属性。  
为了确保始终为嵌套属性触发验证，即使找不到属性，也必须用@Valid注释关联的字段。以下示例基于前面的AcmeProperties示例：  

    @ConfigurationProperties(prefix="acme")
    @Validated
    public class AcmeProperties {
    
    @NotNull
    private InetAddress remoteAddress;
    
    @Valid
    private final Security security = new Security();
    
    // ... getters and setters
    
    public static class Security {
    
    @NotEmpty
    public String username;
    
    // ... getters and setters
    
    }
    
    }  

您还可以通过创建名为configurationPropertiesValidator的bean定义来添加自定义Spring验证程序。@Bean方法应该声明为静态的。配置属性验证器是在应用程序生命周期的早期创建的，将@Bean方法声明为static可以创建Bean，而无需实例化@configuration类。这样做可以避免任何可能由早期实例化引起的问题。  
spring boot执行器模块包含一个端点，该端点公开所有@ConfigurationProperties bean。将web浏览器指向/actuator/configprops或使用等效的JMX端点。有关详细信息，请参阅“生产就绪功能”部分。  

### 2.8.10. @ConfigurationProperties vs. @Value ###  
@Value注释是一个核心容器特性，它不提供与类型安全配置属性相同的特性。下表总结了@ConfigurationProperties和@Value支持的功能：

![](https://i.imgur.com/Wp7Er39.png)    


如果您为自己的组件定义了一组配置键，我们建议您将它们分组到一个带有@ConfigurationProperties注释的POJO中。您还应该注意，由于@Value不支持松散绑定，如果需要使用环境变量提供值，则它不是一个好的候选值。  
最后，虽然可以在@Value中编写SpEL表达式，但此类表达式不会从应用程序属性文件中处理。
----------
2020/3/20 20:00:58 

----------

