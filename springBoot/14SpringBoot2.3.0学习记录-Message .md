
# 14. Messaging #  
Spring框架为与消息传递系统的集成提供了广泛的支持，从使用JmsTemplate的JMS API的简化使用到异步接收消息的完整基础结构。Spring AMQP为高级消息队列协议提供了类似的功能集。Spring Boot还为RabbitTemplate和RabbitMQ提供了自动配置选项。Spring WebSocket本机包括对STOMP消息的支持，Spring Boot通过启动器和少量的自动配置支持这一点。SpringBoot还支持ApacheKafka。  


## 14.1. JMS ##  
ConnectionFactory接口提供了创建javax.jms.Connection以与jms代理交互的标准方法。尽管Spring需要一个ConnectionFactory来处理JMS，但您通常不需要自己直接使用它，而是可以依赖于更高级别的消息传递抽象。（有关详细信息，请参阅Spring框架参考文档的相关部分。）Spring Boot还自动配置发送和接收消息所需的基础结构。  

### 14.1.1 ActiveMQ支持  ###  
当ActiveMQ在类路径上可用时，Spring Boot还可以配置ConnectionFactory。如果存在代理，则会自动启动并配置嵌入式代理（前提是通过配置未指定代理URL）  

**如果您使用spring-boot-starter-activemq，那么将提供连接或嵌入activemq实例所需的依赖项，与JMS集成的spring基础设施也是如此。**

ActiveMQ配置由spring.ActiveMQ.*中的外部配置属性控制。例如，可以在application.properties中声明以下部分：

    spring.activemq.broker-url=tcp://192.168.1.210:9876
    spring.activemq.user=admin
    spring.activemq.password=secret  

默认情况下，CachingConnectionFactory使用可由中的外部配置属性控制的合理设置包装本机ConnectionFactory spring.jms.*:

    spring.jms.cache.session-cache-size=5 

如果您希望使用本机池，可以通过将依赖项添加到组织信息中心：池化jms并相应地配置JmsPoolConnectionFactory，如下例所示：  

    spring.activemq.pool.enabled=true
    spring.activemq.pool.max-connections=50  

有关更多支持的选项，请参阅ActiveMQProperties。您还可以注册任意数量的bean，这些bean实现ActiveMQConnectionFactoryCustomizer以进行更高级的定制。  

默认情况下，如果目的地尚不存在，ActiveMQ会创建一个目的地，以便根据目的地提供的名称解析目的地。 


### 14.1.2. Artemis Support ###  
当Spring Boot检测到类路径上有Artemis可用时，它可以自动配置ConnectionFactory。如果存在代理，则会自动启动并配置嵌入式代理（除非已显式设置模式属性）。支持的模式是嵌入的（明确说明需要嵌入的代理，并且如果代理在类路径上不可用，则会发生错误）和本机的（使用netty传输协议连接到代理）。当配置了后者时，Spring Boot将使用默认设置配置连接到本地计算机上运行的代理的ConnectionFactory。  

如果您使用spring-boot-starter-artemis，那么将提供连接到现有artemis实例所必需的依赖项，以及与JMS集成的spring基础设施。将org.apache.activemq:artemis jms服务器添加到应用程序中可以使用嵌入式模式。  

Artemis配置由中的外部配置属性控制spring.artemis.*. 例如，您可以在应用程序.属性:  

    spring.artemis.mode=native
    spring.artemis.host=192.168.1.210
    spring.artemis.port=9876
    spring.artemis.user=admin
    spring.artemis.password=secret  

嵌入代理时，可以选择是否要启用持久性，并列出应提供的目标。可以将它们指定为逗号分隔的列表以使用默认选项创建它们，也可以分别为高级队列和主题配置定义org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration或org.apache.activemq.artemis.jms.server.config.TopicConfiguration类型的bean。  

默认情况下，CachingConnectionFactory使用可由spring.jms中的外部配置属性控制的合理设置包装本机ConnectionFactory。*：  

    spring.jms.cache.session-cache-size=5  

如果您希望使用本机池，可以通过向org.messaginghub:pooled jms添加依赖项并相应地配置JmsPoolConnectionFactory来实现，如下例所示：  

    spring.artemis.pool.enabled=true
    spring.artemis.pool.max-connections=50  
有关更多支持的选项，请参见artemproperties。              
不涉及JNDI查找，使用Artemis配置中的name属性或通过配置提供的名称解析目的地的名称。