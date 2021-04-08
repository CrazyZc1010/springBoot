# 7.开发Web应用程序 #  
Spring Boot非常适合web应用程序开发。您可以使用嵌入式Tomcat、Jetty、Undertow或Netty创建一个独立的HTTP服务器。大多数web应用程序使用springbootstarterweb模块快速启动和运行。您还可以选择使用springbootstarterwebflux模块来构建反应式web应用程序。  
如果您还没有开发一个Spring Boot web应用程序，可以按照“Hello World！”“入门”部分中的示例。  
## 7.1Spring Web MVC框 ##  
Spring Web MVC框架（通常简称为“springmvc”）是一个丰富的“模型-视图-控制器”Web框架。Spring MVC允许您创建特殊的@Controller或@RestController bean来处理传入的HTTP请求。控制器中的方法通过使用@RequestMapping注释映射到HTTP。  
下面的代码显示了一个提供JSON数据的典型@RestController：  

    @RestController
    @RequestMapping(value="/users")
    public class MyRestController {
    
    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
    // ...
    }
    
    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
    // ...
    }
    
    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
    // ...
    }
    
    }

Spring MVC是核心Spring框架的一部分，详细信息可以在参考文档中找到。Spring.io/guides上还提供了一些涵盖Spring MVC的指南。  

### 7.1.1 Spring MVC自动配置 ###  
Spring Boot为Spring MVC提供了自动配置，它可以很好地与大多数应用程序一起工作。  
自动配置在Spring默认设置的基础上添加了以下功能：
  
1. 包含ContentNegotingViewResolver和BeanNameViewResolver bean。  
1. 对服务静态资源的支持，包括对WebJars的支持（本文档稍后将介绍）。  
1. 自动注册转换器、泛型转换器和格式化程序bean。  
1. 支持httpMessageConverter（本文档稍后将介绍）。  
1. 自动注册MessageCodesResolver（本文档稍后介绍）。  
1. 静态index.html支持。  
1. 自定义Favicon支持（本文档稍后介绍）。  
1. 自动使用可配置的WebBindingInitializerbean（本文档稍后将介绍）。    

如果您想保留这些Spring-Boot MVC自定义并进行更多的MVC自定义（拦截器、格式化程序、视图控制器和其他功能），可以添加自己的@Configuration类，类型为webmvcconfiguer，但不需要@EnableWebMvc。  
如果要提供RequestMappingHandlerMapping、RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver的自定义实例，并且仍然保留Spring Boot MVC自定义，则可以声明WebMVCregistration类型的bean，并使用它来提供这些组件的自定义实例。  
如果您想完全控制Spring MVC，可以添加自己的@Configuration，并用@EnableWebMvc注释，或者添加自己的@Configuration注释delegatingwebmvc配置，如@EnableWebMvc的Javadoc所述。  

### 7.1.2条。http消息转换器 ###  
Spring MVC使用HttpMessageConverter接口来转换HTTP请求和响应。合理的违约是开箱即用的。例如，对象可以自动转换为JSON（通过使用Jackson库）或XML（通过使用Jackson XML扩展（如果可用），或者通过使用JAXB（如果Jackson XML扩展不可用）。默认情况下，字符串是用UTF-8编码的。          
如果需要添加或自定义转换器，可以使用Spring Boot的HttpMessageConverters类，如下所示：  
  
    import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
    import org.springframework.context.annotation.*;
    import org.springframework.http.converter.*;
    
    @Configuration(proxyBeanMethods = false)
    public class MyConfiguration {
    
    @Bean
    public HttpMessageConverters customConverters() {
    HttpMessageConverter<?> additional = ...
    HttpMessageConverter<?> another = ...
    return new HttpMessageConverters(additional, another);
    }
    
    }  

上下文中存在的任何HttpMessageConverter bean都将添加到转换器列表中。您也可以用同样的方法覆盖默认转换器。  


### 7.1.3. Custom JSON 序列化程序和反序列化程序 ###  
如果使用Jackson序列化和反序列化JSON数据，则可能需要编写自己的JsonSerializer和jsondersializer类。自定义序列化程序通常通过一个模块向Jackson注册，但是Spring Boot提供了一个可选的@JsonComponent注释，使得直接注册springbean更容易。  
您可以在JsonSerializer、jsondserializer或KeyDeserializer实现上直接使用@JsonComponent注释。您还可以在包含序列化程序/反序列化程序作为内部类的类上使用它，如下例所示：  

    import java.io.*;
    import com.fasterxml.jackson.core.*;
    import com.fasterxml.jackson.databind.*;
    import org.springframework.boot.jackson.*;
    
    @JsonComponent
    public class Example {
    
    public static class Serializer extends JsonSerializer<SomeObject> {
    // ...
    }
    
    public static class Deserializer extends JsonDeserializer<SomeObject> {
    // ...
    }
    
    }
    
ApplicationContext中的所有@JsonComponent bean都会自动注册到Jackson。因为@JsonComponent是用@Component进行元注释的，所以通常的组件扫描规则适用  
Spring Boot还提供了JsonObjectSerializer和JsonObjectDeserializer基类，它们在序列化对象时提供了标准Jackson版本的有用替代品。有关详细信息，请参阅Javadoc中的JsonObjectSerializer和JsonObjectDeserializer。  
### 7.1.4条。消息代码解算器  ###          
Spring MVC有一个生成错误代码的策略，用于呈现来自绑定错误的错误消息：MessageCodesResolver。如果设置了spring.mvc.message-codes-resolver-format属性PREFIX_ERROR_CODE或POSTFIX_ERROR_CODE，则spring Boot会为您创建一个前缀（请参阅DefaultMessageCodesResolver.format中的枚举）。

### 7.1.5条。静态内容 ###  
默认情况下，Spring Boot从类路径中名为/static（或/public或/resources或/META-INF/resources）的目录或ServletContext的根目录中提供静态内容。它使用来自Spring MVC的ResourceHttpRequestHandler，这样您就可以通过添加自己的webmvcconfiguer和重写addResourceHandlers方法来修改该行为。  
在一个独立的web应用程序中，容器中的默认servlet也被启用，并充当回退，如果Spring决定不处理它，则从ServletContext的根目录提供内容。大多数情况下，这种情况不会发生（除非修改默认的MVC配置），因为Spring总是可以通过DispatcherServlet处理请求。  
默认情况下，资源映射在/*，但可以使用spring.mvc.static-path-pattern属性对其进行优化。例如，可以将所有资源重新定位到/resources/*中，如下所示：  

    spring.mvc.static-path-pattern=/resources/**  

还可以使用spring.resources.static-locations属性（用目录位置列表替换默认值）自定义静态资源位置。根Servlet上下文路径“/”也会自动添加为一个位置。  
除了前面提到的“标准”静态资源位置之外，Webjars内容还有一个特殊情况。如果jar文件是以webjars格式打包的，那么路径为/webjars/**的任何资源都可以从jar文件中获得服务。  

**如果应用程序打包为jar，请不要使用src/main/webapp目录。尽管这个目录是一个通用的标准，但它只适用于war打包，如果生成jar，大多数构建工具都会忽略它。**  


----------
2020/3/25 21:21:49 

----------

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用诸如缓存破坏静态资源或对webjar使用与版本无关的url等用例。  
要对webjar使用版本不可知的url，请添加Webjars定位器核心依赖项。然后声明你的Webjar。以jQuery为例，添加“/webjars/jQuery/jQuery.min.js”将导致“/webjars/jQuery/x.y.z/jQuery.min.js”，其中x.y.z是Webjar版本。  
如果使用JBoss，则需要声明webjars定位器JBoss vfs依赖项，而不是webjars定位器核心。否则，所有webjar都将解析为404。  
要使用缓存破坏，以下配置将为所有静态资源配置缓存破坏解决方案，有效地在URL中添加内容哈希，例如  

`<link href=“/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css/>： ` 

    spring.resources.chain.strategy.content.enabled=true
    spring.resources.chain.strategy.content.paths=/**  

到资源的链接在运行时在模板中重写，这要感谢为Thymeleaf和FreeMarker自动配置的resourceurencodingfilter。当使用jsp时，您应该手动声明这个过滤器。当前不自动支持其他模板引擎，但可以使用自定义模板宏/帮助程序和ResourceUrlProvider。  
当使用JavaScript模块加载器动态加载资源时，重命名文件不是一个选项。这就是为什么其他战略也得到支持并可以结合起来的原因。“固定”策略在URL中添加静态版本字符串，而不更改文件名，如下例所示：  

    spring.resources.chain.strategy.content.enabled=true
    spring.resources.chain.strategy.content.paths=/**
    spring.resources.chain.strategy.fixed.enabled=true
    spring.resources.chain.strategy.fixed.paths=/js/lib/
    spring.resources.chain.strategy.fixed.version=v12  

通过这种配置，“/js/lib/”下的JavaScript模块使用一种固定的版本控制策略（“/v12/js/lib/mymodule.js”），而其他资源仍然使用内容策略（<link://css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css“/>）。  
有关更多支持的选项，请参阅ResourceProperties。  
这个特性已经在一篇专门的博客文章和Spring框架的参考文档中详细描述过了。  

### 7.1.6条。欢迎页面      ###      
Spring Boot支持静态和模板化的欢迎页面。它首先在配置的静态内容位置中查找index.html文件。如果找不到索引模板，它将查找索引模板。如果找到其中一个，它将自动用作应用程序的欢迎页。  
### 7.1.7条。定制Favicon   ###          
与其他静态资源一样，Spring Boot在配置的静态内容位置中查找favicon.ico。如果存在这样的文件，它将自动用作应用程序的favicon。  
### 7.1.8条。路径匹配与 Content Negotiation   ###           
Spring MVC可以通过查看请求路径并将其与应用程序中定义的映射（例如，控制器方法上的@GetMapping注释）匹配，将传入的HTTP请求映射到处理程序。  
Spring Boot默认选择禁用后缀模式匹配，这意味着“GET/projects/Spring Boot.json”之类的请求将不会与@GetMapping（“/projects/Spring Boot”）映射匹配。这被认为是Spring MVC应用程序的最佳实践。这个特性在过去对于没有发送正确的“Accept”请求头的HTTP客户机非常有用；我们需要确保向客户机发送正确的内容类型。如今，内容协商更加可靠。  
有其他方法可以处理不一致地发送适当的“接受”请求头的HTTP客户端。我们可以使用查询参数来确保像“GET/projects/spring boot”这样的请求，而不是使用后缀匹配？format=json“将被映射到@GetMapping（“/projects/spring boot”）：  

    spring.mvc.contentnegotiation.favor-parameter=true
    
    # We can change the parameter name, which is "format" by default:
    # spring.mvc.contentnegotiation.parameter-name=myparam
    
    # We can also register additional file extensions/media types with:
    spring.mvc.contentnegotiation.media-types.markdown=text/markdown  

如果您理解这些注意事项，并且仍然希望应用程序使用后缀模式匹配，则需要以下配置：  

    spring.mvc.contentnegotiation.favor-path-extension=true
    spring.mvc.pathmatch.use-suffix-pattern=true  
或者，与其打开所有后缀模式，不如只支持注册的后缀模式： 
 
    spring.mvc.contentnegotiation.favor-path-extension=true
    spring.mvc.pathmatch.use-registered-suffix-pattern=true
    
    # You can also register additional file extensions/media types with:
    # spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc  

### 7.1.9 ConfigurableWebBindingInitializer  ###              
Spring MVC使用webindinginitializer为特定请求初始化WebDataBinder。如果您创建自己的configurablewebindinginitializer@Bean，Spring Boot会自动将Spring MVC配置为使用它。  
### 7.1.10条。模板引擎   ###            
除了REST web服务，您还可以使用Spring MVC来提供动态HTML内容。Spring MVC支持多种模板技术，包括Thymeleaf、FreeMarker和jsp。此外，许多其他的模板引擎还包括它们自己的Spring MVC集成。  
Spring Boot包括对以下模板引擎的自动配置支持：  
FreeMarker  
Groovy  
Thymeleaf  
Mustache  

如果可能，应避免使用jsp。在将它们与嵌入式servlet容器一起使用时，有几个已知的限制。  
当您使用这些具有默认配置的模板引擎之一时，您的模板将自动从src/main/resources/templates中获取。  

根据应用程序的运行方式，IntelliJ IDEA对类路径的排序方式不同。在IDE中从主方法运行应用程序的顺序与使用Maven或Gradle或其打包的jar运行应用程序的顺序不同。这可能会导致Spring Boot无法在类路径上找到模板。如果遇到此问题，可以在IDE中重新排序类路径，以将模块的类和资源放在第一位。或者，可以将模板前缀配置为搜索类路径上的每个模板目录，如下所示：classpath*：/templates/。  

### 7.1.11条。错误处理    ###           
默认情况下，Spring Boot提供了一个/error映射，它以合理的方式处理所有错误，并在servlet容器中将其注册为“全局”错误页。对于机器客户机，它生成一个JSON响应，其中包含错误、HTTP状态和异常消息的详细信息。对于浏览器客户端，有一个“whitelabel”错误视图，它以HTML格式呈现相同的数据（要自定义它，请添加一个解析为错误的视图）。要完全替换默认行为，可以实现ErrorController并注册该类型的bean定义，或者添加ErrorAttributes类型的bean以使用现有机制，但替换内容。  
基本错误控制器可以用作自定义错误控制器的基类。如果要为新内容类型添加处理程序（默认情况下是专门处理text/html并为其他所有内容提供回退），这一点特别有用。为此，扩展BasicErrorController，添加一个带有products属性的@RequestMapping的公共方法，并创建一个新类型的bean。  
您还可以定义一个用@ControllerAdvice注释的类，以自定义JSON文档以返回特定的控制器和/或异常类型，如下例所示：  

    @ControllerAdvice(basePackageClasses = AcmeController.class)
    public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {
    
    @ExceptionHandler(YourException.class)
    @ResponseBody
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
    HttpStatus status = getStatus(request);
    return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
    }
    
    private HttpStatus getStatus(HttpServletRequest request) {
    Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
    if (statusCode == null) {
    return HttpStatus.INTERNAL_SERVER_ERROR;
    }
    return HttpStatus.valueOf(statusCode);
    }
    
    }

在前面的示例中，如果异常是由与AcmeController在同一个包中定义的控制器引发的，则使用CustomErrorType POJO的JSON表示，而不是ErrorAttributes表示。  

自定义错误页              
如果要显示给定状态代码的自定义HTML错误页，可以将文件添加到/error文件夹。错误页可以是静态HTML（即添加到任何静态资源文件夹下）或使用模板生成。文件名应为确切的状态代码或序列掩码。              
例如，要将404映射到静态HTML文件，文件夹结构如下：  

    src/
     +- main/
     +- java/
     |   + <source code>
     +- resources/
     +- public/
     +- error/
     |   +- 404.html
     +- <other public assets>  

要使用自由标记模板映射所有5xx错误，文件夹结构如下：  

    src/
     +- main/
     +- java/
     |   + <source code>
     +- resources/
     +- templates/
     +- error/
     |   +- 5xx.ftlh
     +- <other templates>  

对于更复杂的映射，还可以添加实现errorviewsolver接口的bean，如下例所示：  

    public class MyErrorViewResolver implements ErrorViewResolver {
    
    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request,
    HttpStatus status, Map<String, Object> model) {
    // Use the request or status to optionally return a ModelAndView
    return ...
    }
    
    }  

您还可以使用常规的Spring MVC特性，比如@ExceptionHandler方法和@ControllerAdvice。然后，ErrorController将拾取任何未处理的异常。  


----------
2020/3/26 20:37:48 

----------


在Spring MVC之外映射错误页  

对于不使用Spring MVC的应用程序，可以使用errorpageregistrator接口直接注册ErrorPages。这种抽象直接与底层的嵌入式servlet容器一起工作，即使您没有Spring MVC DispatcherServlet也可以工作。  
  

    @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
    return new MyErrorPageRegistrar();
    }
    
    // ...
    
    private static class MyErrorPageRegistrar implements ErrorPageRegistrar {
    
    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
    registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }
    
    }  

如果使用最终由筛选器处理的路径注册ErrorPage（这在某些非Spring web框架（如Jersey和Wicket）中很常见），则必须将筛选器显式注册为错误分派器，如下例所示：  

    @Bean
    public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
    }  

注意，默认的FilterRegistrationBean不包括错误分派器类型。  
注意：当部署到servlet容器时，Spring Boot使用其错误页过滤器将带有错误状态的请求转发到相应的错误页。只有在尚未提交响应的情况下，才能将请求转发到正确的错误页。默认情况下，websphereapplicationserver8.0及更高版本在成功完成servlet的服务方法时提交响应。您应该通过将com.ibm.ws.webcontainer.invokeFlushAfterService设置为false来禁用此行为。  
7.1.12. Spring HATEOAS  
如果您开发了一个使用超媒体的RESTful API，那么Spring Boot为Spring HATEOAS提供了自动配置，它可以很好地与大多数应用程序一起工作。自动配置取代了使用@EnableHypermediaSupport的需要，并注册了许多bean来简化基于超媒体的应用程序的构建，包括一个linkdiscoverer（用于客户端支持）和一个ObjectMapper，它们被配置为将响应正确地封送到所需的表示中。ObjectMapper是通过设置各种spring.jackson.*属性来定制的，如果有的话，可以通过Jackson2ObjectMapperBuilder bean来定制。  
您可以使用@EnableHypermediaSupport来控制Spring HATEOAS的配置。注意，这样做会禁用前面描述的ObjectMapper自定义。  

7.1.13. CORS Support  
Cross-origin resource sharing（CORS）是由大多数浏览器实现的W3C规范，它允许您以灵活的方式指定哪些类型的跨域请求被授权，而不是使用一些不太安全和不太强大的方法，如IFRAME或JSONP。  
从版本4.2开始，SpringMVC支持CORS。在Spring Boot应用程序中使用带有@CrossOrigin注释的控制器方法CORS配置不需要任何特定的配置。全局CORS配置可以通过使用自定义的addCorsMappings（CorsRegistry）方法注册WebMvcConfigurer bean来定义，如下例所示： 

    @Configuration(proxyBeanMethods = false)
    public class MyConfiguration {
    
    @Bean
    public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/api/**");
    }
    };
    }
    }  


7.2. The “Spring WebFlux Framework”  
Spring WebFlux是Spring framework 5.0中引入的新的反应式web框架。与Spring MVC不同，它不需要Servlet API，完全异步且无阻塞，并通过Reactor项目实现反应流规范。  
Spring WebFlux有两种风格：函数式和基于注释的。基于注释的模型非常接近Spring MVC模型，如下例所示：  

    @RestController
    @RequestMapping("/users")
    public class MyRestController {
    
    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long user) {
    // ...
    }
    
    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long user) {
    // ...
    }
    
    @DeleteMapping("/{user}")
    public Mono<User> deleteUser(@PathVariable Long user) {
    // ...
    }
    
    }  



----------
2020/3/29 21:51:37 

----------
函数变体“WebFlux.fn”将路由配置与请求的实际处理分离，如下例所示：
  
    @RestController
    @RequestMapping("/users")
    public class MyRestController {
    
    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long user) {
    // ...
    }
    
    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long user) {
    // ...
    }
    
    @DeleteMapping("/{user}")
    public Mono<User> deleteUser(@PathVariable Long user) {
    // ...
    }
    
    }
    

函数变体“WebFlux.fn”将路由配置与请求的实际处理分离，如下例所示：  

    @Configuration(proxyBeanMethods = false)
    public class RoutingConfiguration {
    
    @Bean
    public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
    return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
    .andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
    .andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
    }
    
    }
    
    @Component
    public class UserHandler {
    
    public Mono<ServerResponse> getUser(ServerRequest request) {
    // ...
    }
    
    public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
    // ...
    }
    
    public Mono<ServerResponse> deleteUser(ServerRequest request) {
    // ...
    }
    }  

WebFlux是Spring框架的一部分，其参考文档中提供了详细信息。  
您可以定义任意多个RouterFunction bean来模块化路由器的定义。如果需要应用优先级，可以对bean进行排序。  
要开始，请将spring boot starter webflux模块添加到应用程序中。  

在应用程序中同时添加springbootstarterweb和springbootstarterwebflux模块会导致spring boot自动配置spring MVC，而不是webflux。之所以选择这种行为，是因为许多Spring开发人员将springbootstarterwebflux添加到他们的Spring MVC应用程序中，以使用reactive WebClient。您仍然可以通过将所选应用程序类型设置为SpringApplication.setWebApplicationType（WebApplicationType.REACTIVE）来强制您的选择。  


### 7.2.1. Spring WebFlux Auto-configuration ###  
Spring Boot为Spring WebFlux提供了自动配置，它可以很好地与大多数应用程序一起工作。  
自动配置在Spring默认设置的基础上添加了以下功能：  
为HttpMessageReader和HttpMessageWriter实例配置编解码器（本文档稍后介绍）。         
对服务静态资源的支持，包括对WebJars的支持（本文档稍后将介绍）。  
如果您想保留Spring Boot WebFlux特性，并且想添加额外的WebFlux配置，那么您可以添加自己的@configuration类，类型为WebFluxConfigurer，但不需要@EnableWebFlux。  
如果您想完全控制Spring WebFlux，可以添加自己的@Configuration，并用@EnableWebFlux注释。  


### 7.2.2. HTTP Codecs with HttpMessageReaders and HttpMessageWriters ###  
Spring WebFlux使用HttpMessageReader和HttpMessageWriter接口来转换HTTP请求和响应。它们通过CodecConfigurer配置为通过查看类路径中可用的库具有敏感的默认值。  
Spring Boot为编解码器Spring.codec.*提供专用配置属性。它还通过使用CodecCustomizer实例应用进一步的自定义。例如，jump.jackson.*配置键应用于jackson编解码器。  
如果需要添加或自定义编解码器，可以创建自定义编解码器自定义程序组件，如下例所示：  

    import org.springframework.boot.web.codec.CodecCustomizer;
    
    @Configuration(proxyBeanMethods = false)
    public class MyConfiguration {
    
    @Bean
    public CodecCustomizer myCodecCustomizer() {
    return codecConfigurer -> {
    // ...
    };
    }
    
    }  

您还可以利用Boot的自定义JSON序列化程序和反序列化程序。  
### 7.2.3条。静态内容 ###    
默认情况下，Spring Boot服务于类路径中名为/static（或/public或/resources或/META-INF/resources）的目录中的静态内容。它使用来自springwebflug的ResourceWebHandler，这样您就可以通过添加自己的WebFluxConfigurer并重写addResourceHandlers方法来修改该行为。  
默认情况下，资源被映射到/*，但是您可以通过设置spring.webflux.static-path-pattern属性来优化它。例如，可以将所有资源重新定位到/resources/**中，如下所示：   

    spring.webflux.static-path-pattern=/resources/**   

还可以使用spring.resources.static-locations自定义静态资源位置。这样做会将默认值替换为目录位置列表。如果这样做，默认的欢迎页面检测将切换到您的自定义位置。因此，如果在启动时在任何位置都有index.html，那么它就是应用程序的主页。  
除了前面列出的“标准”静态资源位置之外，Webjars内容还有一个特殊情况。如果jar文件是以webjars格式打包的，那么路径为/webjars/**的任何资源都可以从jar文件中获得服务。
Spring WebFlux应用程序并不严格依赖于Servlet API，因此它们不能作为war文件部署，也不能使用src/main/webapp目录。  
### 7.2.4条。模板引擎 ###  
除了REST web服务，您还可以使用Spring WebFlux来提供动态HTML内容。Spring WebFlux支持多种模板技术，包括Thymeleaf、FreeMarker和Mustache。  
Spring Boot包括对以下模板引擎的自动配置支持：  

FreeMarker
Thymeleaf
Mustache  

当您使用这些具有默认配置的模板引擎之一时，您的模板将自动从src/main/resources/templates中获取。  

### 7.2.5条。错误处理 ###  
Spring Boot提供了一个WebExceptionHandler，它以合理的方式处理所有错误。它在处理顺序中的位置在WebFlux提供的处理程序之前，WebFlux被认为是最后一个处理程序。对于机器客户机，它生成一个JSON响应，其中包含错误、HTTP状态和异常消息的详细信息。对于浏览器客户端，有一个“whitelabel”错误处理程序以HTML格式呈现相同的数据。您还可以提供自己的HTML模板来显示错误（请参阅下一节）。  
自定义此功能的第一步通常涉及使用现有机制，但替换或增加错误内容。为此，可以添加ErrorAttributes类型的bean。  
要更改错误处理行为，可以实现ErrorWebExceptionHandler并注册该类型的bean定义。由于WebExceptionHandler是非常低级的，Spring Boot还提供了一个方便的AbstractErrorWebExceptionHandler，使您能够以WebFlux函数的方式处理错误，如下例所示：  

    public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {
    
    // Define constructor here
    
    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
    
    return RouterFunctions
    .route(aPredicate, aHandler)
    .andRoute(anotherPredicate, anotherHandler);
    }
    
    }  

为了获得更完整的图片，您还可以直接将DefaultErrorWebExceptionHandler子类化，并重写特定的方法  
**自定义错误页**  
如果要显示给定状态代码的自定义HTML错误页，可以将文件添加到/error文件夹。错误页可以是静态HTML（即添加到任何静态资源文件夹下）或使用模板生成。文件名应为确切的状态代码或序列掩码。  
例如，要将404映射到静态HTML文件，文件夹结构如下：  

    src/
     +- main/
     +- java/
     |   + <source code>
     +- resources/
     +- public/
     +- error/
     |   +- 404.html
     +- <other public assets>  
要使用Mustache模板映射所有5xx错误，文件夹结构如下：  

    src/
     +- main/
     +- java/
     |   + <source code>
     +- resources/
     +- templates/
     +- error/
     |   +- 5xx.mustache
     +- <other templates>  

### 7.2.6条。Web筛选器 ###  
Spring WebFlux提供了一个WebFilter接口，可以实现该接口来过滤HTTP请求-响应交换。在应用程序上下文中找到的WebFilter bean将自动用于筛选每个exchange。  
如果过滤器的顺序很重要，那么它们可以实现Ordered或用@order注释。Spring Boot自动配置可以为您配置web过滤器。执行此操作时，将使用下表中显示的顺序：  

Web Filter    Order   
MetricsWebFilter   Ordered.HIGHEST_PRECEDENCE + 1  
WebFilterChainProxy (Spring Security)-100   
HttpTraceWebFilter  Ordered.LOWEST_PRECEDENCE - 10  


----------
2020/3/30 20:51:46 

----------

## 7.3. JAX-RS and Jersey ##  
如果您更喜欢REST端点的JAX-RS编程模型，那么可以使用其中一个可用的实现，而不是Spring MVC。Jersey和Apache CXF在开箱即用的情况下工作得很好。CXF要求您在应用程序上下文中将其Servlet或过滤器注册为@Bean。Jersey有一些本地的Spring支持，因此我们也在Spring Boot中为它提供自动配置支持，同时还提供了一个starter。  
要开始使用Jersey，请将spring boot starter Jersey作为依赖项，然后需要一个ResourceConfig类型的@Bean，在其中注册所有端点，如下例所示：  

    @Component
    public class JerseyConfig extends ResourceConfig {
    
    public JerseyConfig() {
    register(Endpoint.class);
    }
    
    }    
Jersey’s对扫描可执行文件的支持相当有限。例如，当运行可执行war文件时，它无法扫描完全可执行jar文件或WEB-INF/类中的包中的端点。为了避免这种限制，不应该使用packages方法，而应该使用register方法单独注册端点，如前一个示例所示。
对于更高级的自定义，还可以注册任意数量的bean来实现ResourceConfigCustomizer。  
所有注册的端点都应该是带有HTTP资源注释的@Components（@GET和others），如下例所示：  

    @Component
    @Path("/hello")
    public class Endpoint {
    
    @GET
    public String message() {
    return "Hello";
    }
    
    }  

由于端点是Spring@Component，其生命周期由Spring管理，您可以使用@Autowired注释注入依赖项，并使用@Value注释注入外部配置。默认情况下，Jersey servlet已注册并映射到/*。您可以通过向ResourceConfig添加@ApplicationPath来更改映射。  
默认情况下，Jersey在一个名为jerseyServletRegistration的ServletRegistrationBean类型的@Bean中设置为一个Servlet。默认情况下，servlet是惰性初始化的，但是您可以通过设置spring.jersey.servlet.load-on-startup自定义该行为。您可以通过使用相同的名称创建自己的bean来禁用或重写该bean。还可以通过设置spring.jersey.type=filter来使用过滤器而不是servlet（在这种情况下，要替换或重写的@Bean是jerseyFilterRegistration）。过滤器有一个@Order，可以用spring.jersey.filter.Order设置。通过使用spring.jersey.init.*指定属性映射，可以为servlet和过滤器注册提供init参数。  
## 7.4嵌入式Servlet容器支持 ##  
Spring Boot包括对嵌入式Tomcat、Jetty和Undertow服务器的支持。大多数开发人员使用适当的“Starter”来获得完全配置的实例。默认情况下，嵌入式服务器侦听端口8080上的HTTP请求。  

### 7.4.1. Servlets, Filters, and listeners ###  
使用嵌入式servlet容器时，可以使用Spring beans或通过扫描servlet组件从servlet规范注册servlet、过滤器和所有侦听器（例如HttpSessionListener）。  

Registering Servlets, Filters, and Listeners as Spring Beans  
任何作为SpringBean的Servlet、Filter或Servlet*侦听器实例都会注册到嵌入式容器中。如果要在配置过程中引用application.properties中的值，这将非常方便。   
默认情况下，如果上下文只包含一个Servlet，则将其映射到/。在多个servlet bean的情况下，bean名称用作路径前缀。过滤器映射到/*。  

如果基于约定的映射不够灵活，则可以使用ServletRegistrationBean、FilterRegistrationBean和ServletListenerRegistrationBean类进行完全控制。  
通常情况下，leave Filter beans unordere是安全的。如果需要特定的顺序，则应使用@order注释过滤器，或使其实现有序。不能通过用@order注释过滤器的bean方法来配置过滤器的顺序。如果不能将Filter类更改为add@Order或implement Ordered，则必须为筛选器定义Filter registration bean，并使用setOrder（int）方法设置注册bean的顺序。避免配置以Ordered.HIGHEST_优先级读取请求正文的筛选器，因为它可能违反应用程序的字符编码配置。如果Servlet筛选器包装请求，则应使用小于或等于OrderedFilter.request_WRAPPER_filter_MAX_order的顺序对其进行配置。  
要查看应用程序中每个筛选器的顺序，请为web日志记录组启用调试级别日志记录（logging.level.web=debug）。注册过滤器的详细信息，包括它们的顺序和URL模式，将在启动时记录下来。  
注册Filter beans时要小心，因为它们在应用程序生命周期的早期就被初始化了。如果需要注册与其他bean交互的筛选器，请考虑改用DelegatingFilterProxyRegistrationBean。

### 7.4.2. Servlet Context Initialization ###  
嵌入式servlet容器不会直接执行Servlet3.0+javax.servlet.ServletContainerInitializer接口或Spring的org.springframework.web.WebApplicationInitializer接口。这是一个有意的设计决策，旨在降低设计用于在war中运行的第三方库可能会破坏Spring引导应用程序的风险。  
如果需要在SpringBoot应用程序中执行servlet上下文初始化，应该注册一个实现org.springframework.Boot.web.servlet.ServletContextInitializer接口的bean。单一onStartup方法提供对ServletContext的访问，如果需要，可以很容易地用作现有WebApplicationInitializer的适配器。  


**Scanning for Servlets, Filters, and listeners**  
使用嵌入式容器时，可以使用@ServletComponentScan启用用@WebServlet、@WebFilter和@WebListener注释的类的自动注册。  
@ServletComponentScan在一个独立的容器中不起作用，而是使用容器的内置发现机制。  


### 7.4.3. The ServletWebServerApplicationContext ###  
在幕后，Spring Boot使用不同类型的ApplicationContext来支持嵌入式servlet容器。ServletWebServerApplicationContext是一种特殊类型的WebApplicationContext，它通过搜索单个ServletWebServerFactory bean来引导自己。通常，TomcatServletWebServerFactory、JettyServletWebServerFactory或UndertowServletWebServerFactory已自动配置。  

**您通常不需要知道这些实现类。大多数应用程序都是自动配置的，相应的ApplicationContext和ServletWebServerFactory是代表您创建的。**  

### 7.4.4 自定义嵌入式Servlet容器 ###  
可以使用Spring环境属性配置通用的servlet容器设置。通常，您会在application.properties文件中定义属性。  
常用服务器设置包括：  

1. 网络设置：接收HTTP请求的侦听端口（server.port）、要绑定到server.address的接口地址，等等。  
1. 会话设置：会话是否持久（server.servlet.Session.persistent）、会话超时（server.servlet.Session.timeout）、会话数据位置（server.servlet.Session.store dir）和会话cookie配置（server.servlet.Session.cookie.*）。  
1. 错误管理：错误页的位置（server.Error.path）等。  
1. SSL协议  
1. HTTP压缩  

Spring Boot尽可能多地尝试公开公共设置，但这并不总是可能的。对于这些情况，专用名称空间提供特定于服务器的定制（请参阅server.tomcat和server.undertow）。例如，可以使用嵌入式servlet容器的特定功能配置访问日志。  

有关完整列表，请参见ServerProperties类。  

**程序化定制**  
如果需要以编程方式配置嵌入式servlet容器，可以注册实现WebServerFactoryCustomizer接口的Spring bean。WebServerFactoryCustomizer提供对ConfigurableServletWebServerFactory的访问，其中包括许多自定义setter方法。以下示例显示了以编程方式设置端口：  

    import org.springframework.boot.web.server.WebServerFactoryCustomizer;
    import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
    import org.springframework.stereotype.Component;
    
    @Component
    public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
    
    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
    server.setPort(9000);
    }
    
    }  

TomcatServletWebServerFactory、JettyServletWebServerFactory和UndertowServletWebServerFactory是ConfigurableServletWebServerFactory的专用变体，它们分别具有Tomcat、Jetty和Undertow的其他自定义设置器方法。  

**直接自定义ConfigurableServletWebServerFactory**   
如果前面的定制技术太有限，您可以自己注册TomcatServletWebServerFactory、JettyServletWebServerFactory或servletwebserverfactory bean。  

    @Bean
    public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
    return factory;
    }  

为许多配置选项提供了setter。如果您需要做一些更奇特的事情，还提供了一些受保护的方法“钩子”。有关详细信息，请参阅源代码文档。 
 
### 7.4.5条。JSP限制   ###  
当运行使用嵌入式servlet容器（并打包为可执行归档文件）的Spring引导应用程序时，JSP支持有一些限制。  

1. 对于Jetty和Tomcat，如果使用war包装，它应该可以工作。可执行的war在用java-jar启动时可以工作，并且可以部署到任何标准容器。使用可执行jar时不支持jsp。  
1. Undertow不支持JSP。  
1. 创建自定义error.jsp页面不会覆盖错误处理的默认视图。应改用自定义错误页。



----------
2020/3/31 22:28:07 

----------
## 7.5条。嵌入式反应式服务器支持 ##  
Spring Boot包括对以下嵌入式反应式web服务器的支持：Reactor Netty、Tomcat、Jetty和Undertow。大多数开发人员使用适当的“Starter”来获得完全配置的实例。默认情况下，嵌入式服务器侦听端口8080上的HTTP请求。  
## 7.6条。反应式服务器资源配置 ##  
当自动配置Reactor Netty或Jetty服务器时，Spring Boot将创建特定的bean，为服务器实例提供HTTP资源：ReactorResourceFactory或JettyResourceFactory。  
默认情况下，这些资源还将与Reactor Netty和Jetty客户端共享，以获得最佳性能，前提是：  
服务器和客户端使用相同的技术  
客户机实例是使用由Spring Boot自动配置的WebClient.Builder bean构建的。  
开发人员可以通过提供自定义的ReactorResourceFactory或JettyResourceFactory bean来覆盖Jetty和Reactor Netty的资源配置-这将应用于客户端和服务器。  
您可以在WebClient运行时部分中了解有关客户端资源配置的更多信息。