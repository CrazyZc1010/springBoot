# 使用SpringBoot002 #   

## 5.SpringBeans和依赖注入 ##  
您可以自由使用任何标准的Spring框架技术来定义bean及其注入的依赖项。为了简单起见，我们经常发现使用@ComponentScan（来查找bean）和使用@Autowired（来进行构造函数注入）可以很好地工作。    
如果按照上面的建议构造代码（在根包中定位应用程序类），则可以添加@ComponentScan而无需任何参数。所有应用程序组件（@Component，@Service，@Repository，@Controller等）都自动注册为springbean。  
以下示例显示了一个@Service Bean，它使用构造函数注入来获取所需的RiskAssessor Bean：  

    package com.example.service;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    
    @Service
    public class DatabaseAccountService implements AccountService {
    
    private final RiskAssessor riskAssessor;
    
    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
    }
    
    // ...
    
    }  

如果bean有一个构造函数，可以省略@Autowired，如下例所示：  

    @Service
    public class DatabaseAccountService implements AccountService {
    
    private final RiskAssessor riskAssessor;
    
    public DatabaseAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
    }
    
    // ...
    
    }  


## 6.使用@springbootsapplication注释 ##  

许多Spring Boot开发人员喜欢他们的应用程序使用自动配置、组件扫描，并且能够在他们的“应用程序类”上定义额外的配置。一个@springbootsapplication注释可以用于启用这三个功能，即：  
  
1. @EnableAutoConfiguration：启用Spring Boot的自动配置机制  
1. @ComponentScan：在应用程序所在的包上启用@ComponentScan（请参阅最佳实践）    
1. @Configuration：允许在上下文中注册额外的bean或导入其他配置类  
 
示例： 

 
    package com.example.myapplication;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
    public class Application {
    
    public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
    }
    
    }   


@springbootsapplication还提供别名来定制@EnableAutoConfiguration和@ComponentScan的属性。这些功能都不是必需的，您可以选择用它启用的任何功能替换此单个批注。  

## 7.运行应用程序 ##  
将应用程序打包为jar并使用嵌入式HTTP服务器的最大优点之一是，可以像运行其他应用程序一样运行应用程序。调试Spring Boot应用程序也很容易。您不需要任何特殊的IDE插件或扩展。  
本节仅介绍基于jar的打包。如果选择将应用程序打包为war文件，则应参考服务器和IDE文档。  
### 7.1从IDE运行 ###  
您可以从IDE运行一个Spring引导应用程序作为一个简单的Java应用程序。但是，首先需要导入项目。导入步骤因IDE和生成系统而异。大多数ide都可以直接导入Maven项目。例如，Eclipse用户可以从“文件”菜单中选择“导入…”→现有Maven项目。              如果不能直接将项目导入IDE，则可以使用生成插件生成IDE元数据。Maven包括Eclipse和IDEA的插件。Gradle为各种IDE提供插件。  
注意：如果意外地运行web应用程序两次，则会看到“端口已在使用”错误。STS用户可以使用resunch按钮而不是Run按钮来确保关闭任何现有实例。  
### 7.2作为打包应用程序运行 ###  
如果使用Spring Boot Maven或Gradle插件创建可执行jar，则可以使用java-jar运行应用程序，如下例所示：  

    $ java -jar target/myapplication-0.0.1-SNAPSHOT.jar  

也可以在启用远程调试支持的情况下运行打包的应用程序。这样做可以将调试器附加到打包的应用程序，如下例所示：  

    $ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar  

### 7.3使用Maven插件  ###  
Spring Boot Maven插件包含一个运行目标，可用于快速编译和运行应用程序。应用程序以分解的形式运行，就像它们在IDE中一样。下面的示例显示了运行Spring Boot应用程序的典型Maven命令：  

    $ mvn spring-boot:run  

您可能还希望使用MAVEN_OPTS操作系统环境变量，如下例所示：  

     $ export MAVEN_OPTS=-Xmx1024m    

### 7.4使用Gradle插件 ###  
SpringBootGradle插件还包括一个bootRun任务，可用于以分解形式运行应用程序。每当应用org.springframework.boot和java插件时，都会添加bootRun任务，如下例所示：  
    
    $ gradle bootRun  

您可能还希望使用JAVA_OPTS操作系统环境变量，如下例所示：  

    $ export JAVA_OPTS=-Xmx1024m  

### 7.5  Swapping ###  
由于Spring引导应用程序只是普通的Java应用程序，JVM热交换应该是现成的。JVM热交换在某种程度上受限于它可以替换的字节码。对于更完整的解决方案，可以使用JRebel。              spring boot devtools模块还支持快速应用程序重启。有关详细信息，请参阅本章后面的开发人员工具部分和热交换“如何”。  

## 8.开发人员工具 ##  
Spring Boot包括一组额外的工具，这些工具可以使应用程序开发体验更加愉快。spring boot devtools模块可以包含在任何项目中，以提供额外的开发时特性。要包含devtools支持，请将模块依赖项添加到构建中，如Maven和Gradle的以下列表所示：  
Maven：  

    <dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    </dependency>
    </dependencies>
     

Gradle：  

    configurations {
    developmentOnly
    runtimeClasspath {
    extendsFrom developmentOnly
    }
    }
    dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
    }  

注意：  
运行完全打包的应用程序时，开发人员工具将自动禁用。如果您的应用程序是从java-jar启动的，或者是从一个特殊的类加载器启动的，那么它被认为是一个“生产应用程序”。如果这不适用于您（即，如果您从容器运行应用程序），请考虑排除devtools或设置-Dspring.devtools.restart.enabled=false系统属性。  

在Maven中将依赖项标记为可选，或者在Gradle中使用自定义的仅开发配置（如上所示），这是防止devtools被传递应用到使用项目的其他模块的最佳实践。  
默认情况下，重新打包的存档文件不包含devtools。如果要使用某个远程devtools功能，则需要禁用excludeDevtools build属性才能包含它。Maven和Gradle插件都支持该属性。  

### 8.1属性默认值 ###  

Spring Boot支持的几个库使用缓存来提高性能。例如，模板引擎缓存已编译的模板，以避免重复分析模板文件。另外，Spring MVC可以在服务静态资源时向响应添加HTTP缓存头。  
虽然缓存在生产中非常有用，但在开发过程中可能会适得其反，使您无法看到刚刚在应用程序中所做的更改。因此，默认情况下，spring boot devtools会禁用缓存选项。  
缓存选项通常由application.properties文件中的设置配置。例如，Thymeleaf提供spring.Thymeleaf.cache属性。spring boot devtools模块不需要手动设置这些属性，而是自动应用合理的开发时配置。  

由于在开发Spring MVC和Spring WebFlux应用程序时需要更多关于web请求的信息，开发人员工具将为web日志组启用调试日志记录。这将提供有关传入请求、哪个处理程序正在处理它、响应结果等的信息。如果希望记录所有请求详细信息（包括潜在的敏感信息），可以打开spring.mvc.log-request-details或spring.codec.log-request-details配置属性。  

如果不希望应用属性默认值，可以在application.properties中将spring.devtools.add-properties设置为false。  

有关devtools应用的属性的完整列表，请参见DevToolsPropertyDefaultsPostProcessor。  

### 8.2自动重启 ###  
每当类路径上的文件发生更改时，使用spring boot devtools的应用程序都会自动重新启动。当在IDE中工作时，这是一个非常有用的特性，因为它为代码更改提供了一个非常快速的反馈循环。默认情况下，类路径上指向文件夹的任何条目都会被监视以进行更改。请注意，某些资源（如静态资产和视图模板）不需要重新启动应用程序。  

由于DevTools监视类路径资源，触发重新启动的唯一方法是更新类路径。导致类路径更新的方式取决于您使用的IDE。在Eclipse中，保存修改过的文件会导致类路径被更新并触发重新启动。在IntelliJ IDEA中，构建项目（Build+？+Build project）具有相同的效果。  

只要启用了forking，您还可以使用支持的构建插件（Maven和Gradle）启动应用程序，因为DevTools需要一个独立的应用程序类加载器才能正常运行。默认情况下，Gradle和Maven插件会分叉应用程序进程。  

与LiveReload一起使用时，自动重启非常有效。详情请参阅LiveReload部分。如果使用JRebel，则会禁用自动重新启动，以支持动态类重新加载。其他devtools特性（如LiveReload和属性重写）仍然可以使用。  

DevTools依赖于应用程序上下文的关闭挂钩在重新启动期间关闭它。如果禁用了关闭挂钩（SpringApplication.setRegisterShutdownHook（false））则无法正常工作。  

当决定类路径上的条目在发生更改时是否应该触发重新启动时，DevTools会自动忽略名为spring boot、spring boot DevTools、spring boot autoconfigure、spring boot actuator和spring boot starter的项目。  

DevTools需要自定义ApplicationContext使用的ResourceLoader。如果您的应用程序已经提供了一个，它将被包装。不支持直接重写ApplicationContext上的getResource方法。  

重新启动vs重新加载              Spring Boot提供的重启技术通过使用两个类加载器来工作。不改变的类（例如，来自第三方jar的类）被加载到基类加载器中。正在积极开发的类将加载到重新启动的类加载器中。当应用程序重新启动时，重新启动类加载器将被丢弃并创建一个新的类加载器。这种方法意味着应用程序重启通常比“冷启动”快得多，因为基本类加载器已经可用并已填充。              如果您发现重新启动对于您的应用程序不够快，或者遇到类加载问题，您可以考虑从ZeroTurnaround重新加载JRebel等技术。这些工作是在加载类时重写类，以使它们更易于重新加载。  

#### 8.2.1记录条件评估中的更改 ####  

默认情况下，每次应用程序重新启动时，都会记录显示条件求值增量的报告。在您进行更改（如添加或删除bean以及设置配置属性）时，报表将显示应用程序自动配置的更改。 要禁用报表的日志记录，请设置以下属性：  

    spring.devtools.restart.log-condition-evaluation-delta=false  
#### 8.2.2排除资源 ####  
某些资源在更改时不一定需要触发重新启动。例如，可以就地编辑Thymeleaf模板。默认情况下，更改/META-INF/maven、/META-INF/resources、/resources、/static、/public或/templates中的资源不会触发重新启动，但会触发实时重新加载。如果要自定义这些排除，可以使用spring.devtools.restart.exclude属性。例如，要仅排除/static和/public，您可以设置以下属性：  

    spring.devtools.restart.exclude=static/**,public/**  

如果要保留这些默认值并添加其他排除项，请改用spring.devtools.restart.additional-exclude属性。  

#### 8.2.3 监视其他路径    ####
更改不在类路径上的文件时，您可能希望重新启动或重新加载应用程序。为此，请使用spring.devtools.restart.additional-paths属性配置其他路径以监视更改。您可以使用前面描述的spring.devtools.restart.exclude属性来控制附加路径下的更改是触发完全重新启动还是实时重新加载。  

#### 8.2.4禁用重新启动     ####
如果不想使用重新启动功能，可以使用spring.devtools.restart.enabled属性禁用它。在大多数情况下，可以在application.properties中设置此属性（这样做仍然会初始化restart classloader，但它不会监视文件更改）。 如果需要完全禁用重新启动支持（例如，因为它不适用于特定库），则需要在调用SpringApplication.run（…）之前将spring.devtools.restart.enabled系统属性设置为false，如下例所示：  

    public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
    }  

#### 8.2.5使用触发器文件      #### 
如果使用的IDE持续编译更改的文件，则可能希望仅在特定时间触发重新启动。为此，您可以使用“触发器文件”，这是一个特殊文件，当您希望实际触发重新启动检查时，必须对其进行修改  
对该文件的任何更新都将触发检查，但只有在Devtools检测到有事情要做时，才会实际重新启动。  
要使用触发器文件，请将spring.devtools.restart.trigger-file属性设置为触发器文件的名称（不包括任何路径）。触发器文件必须出现在类路径的某个位置。例如，如果项目具有以下结构： 

    src
    +- main
       +- resources
         +- .reloadtrigger  

那么您的触发器文件属性将是：  

    spring.devtools.restart.trigger-file=.reloadtrigger  

只有更新src/main/resources/.reloaddrigger时，才会重新启动。  

您可能希望将spring.devtools.restart.trigger-file设置为全局设置，以便所有项目的行为都相同。  
有些ide的特性使您不必手动更新触发器文件。用于Eclipse和IntelliJ IDEA（终极版）的Spring工具都有这样的支持。使用Spring工具，您可以从控制台视图中使用“reload”按钮（只要您的触发器文件名为.reloadtrigger）。对于IntelliJ，您可以按照文档中的说明进行操作。  

#### 8.2.6自定义重新启动类加载器 ####  

如前面的Restart vs Reload部分所述，Restart功能是通过使用两个类加载器实现的。对于大多数应用程序，这种方法工作得很好。但是，它有时会导致类加载问题。  
默认情况下，IDE中任何打开的项目都用“restart”类加载器加载，而任何常规的.jar文件都用“base”类加载器加载。如果您在一个多模块项目上工作，并且不是每个模块都导入到您的IDE中，那么您可能需要定制一些东西。为此，可以创建一个META-INF/spring-devtools.properties文件。  
spring-devtools.properties文件可以包含前缀为restart.exclude和restart.include的属性。include元素是应该向上拉入“restart”类加载器的项，exclude元素是应该向下推入“base”类加载器的项。属性的值是应用于类路径的正则表达式模式，如下例所示：  

    restart.exclude.companycommonlibs=/mycorp-common-[\\w\\d-\.]+\.jar
    restart.include.projectcommon=/mycorp-myproj-[\\w\\d-\.]+\.jar  

所有属性键必须唯一。只要属性以restart.include开头。或重新启动。排除。它是被考虑的。  

将加载类路径中的所有META-INF/spring-devtools.properties。可以将文件打包到项目内部或项目使用的库中。  

#### 8.2.7已知限制 ####  

重新启动功能不能很好地处理使用标准ObjectInputStream反序列化的对象。如果需要反序列化数据，可能需要将Spring的ConfigurableObjectInputStream与Thread.currenthread（）.getContextClassLoader（）结合使用。  
不幸的是，一些第三方库在反序列化时没有考虑上下文类加载器。如果您发现这样的问题，您需要请求与原始作者修复。  

### 8.3在线重新加载 ###  
spring boot devtools模块包括一个嵌入式LiveReload服务器，当资源发生更改时，该服务器可用于触发浏览器刷新。从LiveReload.com可以免费获得针对Chrome、Firefox和Safari的LiveReload浏览器扩展。              如果不想在应用程序运行时启动LiveReload服务器，可以将spring.devtools.LiveReload.enabled属性设置为false。  
一次只能运行一个LiveReload服务器。在启动应用程序之前，请确保没有其他LiveReload服务器正在运行。如果从IDE启动多个应用程序，则只有第一个应用程序支持LiveReload。  
### 8.4全局设置 ###  
您可以通过将以下任何文件添加到$HOME/.config/spring boot文件夹来配置全局devtools设置：  

1. spring-boot-devtools.properties
1. spring-boot-devtools.yaml
1. spring-boot-devtools.yml  

添加到这些文件中的任何属性都适用于使用devtools的计算机上的所有Spring引导应用程序。例如，要将restart配置为始终使用触发器文件，可以添加以下属性：  
**~/.config/spring-boot/spring-boot-devtools.properties**  

    spring.devtools.restart.trigger-file=.reloadtrigger  

如果在$HOME/.config/spring boot中找不到devtools配置文件，则会在$HOME文件夹的根目录中搜索是否存在.spring-boot-devtools.properties文件。这允许您与不支持$HOME/.config/Spring引导位置的旧版本Spring引导上的应用程序共享devtools全局配置。   
在上述文件中激活的配置文件不会影响配置文件特定配置文件的加载。  
### 8.5远程应用程序 ###  
Spring Boot开发工具并不局限于本地开发。您还可以在远程运行应用程序时使用一些功能。远程支持是可选的，因为启用它可能会带来安全风险。只有在受信任的网络上运行或使用SSL保护时才应启用它。如果这两个选项都不可用，则不应使用DevTools的远程支持。您不应该在生产部署上启用支持。  
要启用它，需要确保重新打包的归档文件中包含devtools，如下所示：  

    <build>
    <plugins>
    <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
    <excludeDevtools>false</excludeDevtools>
    </configuration>
    </plugin>
    </plugins>
    </build>  

然后需要设置spring.devtools.remote.secret属性。与任何重要的密码或秘密一样，该值应该是唯一的、强的，这样就不会被猜测或暴力破解。  
远程devtools支持分为两部分：接受连接的服务器端端点和在IDE中运行的客户端应用程序。设置spring.devtools.remote.secret属性时，服务器组件将自动启用。必须手动启动客户端组件。  

#### 8.5.1运行远程客户端应用程序 ####    
远程客户端应用程序设计为从IDE中运行。您需要使用与连接到的远程项目相同的类路径运行  
org.springframework.boot.devtools.RemoteSpringApplication。应用程序的唯一必需参数是它连接到的远程URL。  
例如，如果您使用的是Eclipse或STS，并且有一个名为my app的项目已部署到Cloud Foundry，那么您将执行以下操作：
  
Select Run Configurations…​ from the Run menu.  

Create a new Java Application “launch configuration”.  

Browse for the my-app project.  

Useorg.springframework.boot.devtools.RemoteSpringApplication as the main class.  

Add https://myapp.cfapps.io to the Program arguments (or whatever your remote URL is).  

注意：因为远程客户端使用与实际应用程序相同的类路径，所以它可以直接读取应用程序属性。这就是读取spring.devtools.remote.secret属性并将其传递给服务器进行身份验证的方式。  

建议始终使用https://作为连接协议，以便对通信进行加密，并且不能拦截密码。  

如果需要使用代理访问远程应用程序，请配spring.devtools.remote.proxy.host和spring.devtools.remote.proxy.port属性。  

#### 8.5.2远程更新 ###   
远程客户机以与本地重新启动相同的方式监视应用程序类路径的更改。任何更新的资源都被推送到远程应用程序，并（如果需要）触发重新启动。如果您迭代使用本地没有的云服务的功能，这将非常有用。通常，远程更新和重新启动要比完整的重建和部署周期快得多。  

只有在远程客户端运行时才会监视文件。如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器。  

#### 8.5.3配置文件系统监视程序 ####  
FileSystemWatcher的工作方式是在一定的时间间隔内轮询类更改，然后等待预定义的安静时间段以确保不再有更改。然后将更改上载到远程应用程序。在较慢的开发环境中，可能会出现安静期不够的情况，并且类中的更改可能会被分成批。第一批类更改上载后，服务器将重新启动。下一批无法发送到应用程序，因为服务器正在重新启动。  
这通常表现在RemoteSpringApplication日志中关于未能上载某些类的警告，以及随后的重试。但这也可能导致应用程序代码不一致，并且在第一批更改上载后无法重新启动。    
如果您经常观察到此类问题，请尝试将spring.devtools.restart.poll-interval和spring.devtools.restart.quiet-period参数增加到适合您的开发环境的值：  

    spring.devtools.restart.poll-interval=2s
    spring.devtools.restart.quiet-period=1s  

现在，监视的类路径文件夹每2秒轮询一次以获取更改，并保持1秒的静默期以确保没有其他类更改。  
## 9.将应用程序打包以供生产 ##  
可执行jar可用于生产部署。由于它们是自包含的，因此也非常适合基于云的部署。  
对于额外的“生产就绪”特性，如运行状况、审计和度量REST或JMX端点，请考虑添加spring boot执行器。有关详细信息，请参见production-ready-features.html。


