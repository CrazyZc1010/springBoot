# 8.平滑关闭 #  
所有四个嵌入式web服务器（Jetty、Reactor Netty、Tomcat和Undertow）以及反应式和基于Servlet的web应用程序都支持正常关机。启用后，应用程序的关闭将包括一个可配置持续时间的宽限期。在此宽限期内，将允许现有请求完成，但不允许新请求。不允许新请求的确切方式取决于所使用的web服务器。Jetty、Reactor Netty和Tomcat将停止在网络层接受请求。Undertow将接受请求，但会立即响应服务不可用（503）响应。  
在应用程序关闭处理期间和销毁任何bean之前，正常关机是第一步。这可以确保bean可供允许完成飞行请求时发生的任何处理使用。要启用正常关机，请配置server.shutdown.grace-period属性，如下例所示：  

    server.shutdown.grace-period=30s  

# 9 RSocket #  
RSocket是用于字节流传输的二进制协议。它通过异步消息、通过单个连接来启用对称交互模型。  
spring框架的spring消息传递模块在客户端和服务器端为RSocket请求者和响应者提供支持。有关更多详细信息，包括RSocket协议的概述，请参见Spring Framework参考的RSocket部分。  
## 9.1.RSocket策略自动配置 ##  
Spring Boot自动配置一个RSocketStrategies bean，它为RSocket有效负载的编码和解码提供了所有必需的基础设施。默认情况下，自动配置将尝试配置以下内容（按顺序）：  
CBOR codecs with Jackson  
JSON codecs with Jackson  

spring boot starter rsocket starter提供了这两个依赖项。查看Jackson支持部分，了解更多定制可能性。  
开发人员可以通过创建实现RSocketStrategiesCustomizer接口的bean来定制RSocketStrategies组件。注意，它们的@Order很重要，因为它决定编解码器的顺序。  

## 9.2 RSocket服务器自动配置 ##   
Spring Boot提供RSocket服务器自动配置。所需的依赖项由spring boot starter rsocket提供。  
Spring Boot允许从WebFlux服务器通过WebSocket公开RSocket，或者建立一个独立的RSocket服务器。这取决于应用程序的类型及其配置。  
对于WebFlux应用程序（即Web application type.REACTIVE类型），只有在下列属性匹配时，RSocket服务器才会插入Web服务器：  

    spring.rsocket.server.mapping-path=/rsocket # a mapping path is defined
    spring.rsocket.server.transport=websocket # websocket is chosen as a transport
    #spring.rsocket.server.port= # no port is defined  

只有Reactor Netty支持将RSocket插入web服务器，因为RSocket本身就是用这个库构建的。  
或者，RSocket TCP或websocket服务器作为独立的嵌入式服务器启动。除了依赖项要求外，唯一需要的配置是定义该服务器的端口：  

    spring.rsocket.server.port=9898 # the only required configuration
    spring.rsocket.server.transport=tcp # you're free to configure other properties  

## 9.3. Spring Messaging RSocket support ##  
Spring Boot将为RSocket自动配置Spring消息传递基础设施。  
这意味着Spring Boot将创建一个RSocketMessageHandler bean，它将处理对应用程序的RSocket请求。  

## 9.4 使用RSocketRequester调用RSocket服务 ##  
一旦在服务器和客户端之间建立了RSocket通道，任何一方都可以向另一方发送或接收请求。  
作为服务器，可以在RSocket@Controller的任何处理程序方法上注入RSocketRequester实例。作为客户机，您需要首先配置并建立RSocket连接。Spring Boot使用预期的编解码器自动为此类情况配置RSocketRequester.Builder。  
Builder实例是一个原型bean，这意味着每个注入点都将为您提供一个新实例。这是有目的的，因为这个构建器是有状态的，您不应该使用同一个实例创建具有不同设置的请求程序。  
以下代码显示了一个典型示例：  

    @Service
    public class MyService {
    
    private final Mono<RSocketRequester> rsocketRequester;
    
    public MyService(RSocketRequester.Builder rsocketRequesterBuilder) {
    this.rsocketRequester = rsocketRequesterBuilder
    .connectTcp("example.org", 9898).cache();
    }
    
    public Mono<User> someRSocketCall(String name) {
    return this.rsocketRequester.flatMap(req ->
    req.route("user").data(name).retrieveMono(User.class));
    }
    
    }  


