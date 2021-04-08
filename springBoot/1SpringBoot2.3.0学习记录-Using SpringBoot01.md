# 使用SpringBoot #  
本节将详细介绍如何使用Spring Boot。它涵盖了构建系统、自动配置和如何运行应用程序等主题.  
## 1.构建系统   
强烈建议您选择一个支持依赖关系管理的构建系统，该系统可以使用发布到“Maven Central”存储库的工件。我们建议您选择Maven或Gradle。可以让Spring Boot与其他构建系统（例如Ant）协同工作，但它们并没有得到特别的支持。  
###  1.1依赖管理 ###  
每一个Spring Boot版本都提供了一个它支持的依赖项列表。实际上，您不需要为构建配置中的任何依赖项提供版本，因为Spring Boot会为您管理这些依赖项。当您升级Spring Boot本身时，这些依赖项也会以一致的方式升级。  
这个列表包含了所有可以与spring Boot一起使用的spring模块以及第三方库的优化列表。该列表作为标准材料清单（spring boot依赖项）提供，可以与Maven和Gradle一起使用。  
### 1.2. Maven ### 
Maven用户可以从spring boot starter父项目继承来获得合理的默认值。父项目提供以下功能：  
  
1. Java 1.8作为默认编译器级别。

1. UTF-8源代码。

1. 一个依赖管理部分，继承自spring boot依赖pom，用于管理公共依赖的版本。当在您自己的pom中使用这些依赖项时，此依赖项管理允许您省略这些依赖项的<version>标记。

1. 使用重新打包执行id执行重新打包目标。

1. 合理的资源筛选。

1. 合理的插件配置（exec plugin、Git commit ID和shade）。

1. 对application.properties和application.yml进行合理的资源筛选，包括特定于配置文件的文件（例如，application-dev.properties和application-dev.yml）    


注意，由于application.properties和application.yml文件接受Spring样式的占位符（${…}），Maven筛选更改为使用@..@占位符。（您可以通过设置一个名为resource.delimiter的Maven属性来覆盖它。）  

### 1.2.1继承起始父项 ###  
要将项目配置为从spring boot starter父项继承，请按如下方式设置父项： 
    
    <!-- Inherit defaults from Spring Boot -->
    <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.BUILD-SNAPSHOT</version>
    </parent>  

使用该设置，还可以通过重写自己项目中的属性来重写各个依赖项。例如，要升级到另一个Spring数据发布系列，可以将以下内容添加到pom.xml中：  

    <properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
    </properties>    

### 1.2.2在没有父POM的情况下使用Spring Boot ###  
并不是每个人都喜欢继承spring boot starter的父POM。您可能需要使用自己的公司标准父级，或者您可能希望显式声明所有Maven配置。

如果您不想使用spring boot starter父项，那么仍然可以通过使用scope=import dependency来保留依赖项管理（而不是插件管理）的好处，如下所示：  

    <dependencyManagement>
    <dependencies>
    <dependency>
    <!-- Import dependency management from Spring Boot -->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.0.BUILD-SNAPSHOT</version>
    <type>pom</type>
    <scope>import</scope>
    </dependency>
    </dependencies>
    </dependencyManagement>
     
如上所述，前面的示例设置不允许您使用属性重写单个依赖项。要获得相同的结果，您需要在项目的dependencyManagement中添加一个条目，然后再添加spring boot dependencies条目。例如，要升级到另一个Spring数据发布系列，可以将以下元素添加到pom.xml中：  

    <dependencyManagement>
    <dependencies>
    <!-- Override Spring Data release train provided by Spring Boot -->
    <dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-releasetrain</artifactId>
    <version>Fowler-SR2</version>
    <type>pom</type>
    <scope>import</scope>
    </dependency>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.0.BUILD-SNAPSHOT</version>
    <type>pom</type>
    <scope>import</scope>
    </dependency>
    </dependencies>
    </dependencyManagement>   

### 1.2.3使用Spring Boot Maven插件 ###  
Spring Boot包含一个Maven插件，它可以将项目打包为一个可执行jar。如果要使用插件，请将其添加到<plugins>部分，如下例所示：  
    
    <build>
    <plugins>
    <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    </plugins>
    </build>


### 1.3. Gradle   
### 1.4. Ant  
可以使用Apache Ant+Ivy构建一个Spring引导项目。spring boot antlib“antlib”模块也可以帮助Ant创建可执行jar。   
### 1.5. Starters ###  
Starters是一组方便的依赖关系描述符，可以包含在应用程序中。您可以一站式地获得所需的所有Spring和相关技术，而无需搜索示例代码和复制粘贴的依赖描述符。  
启动程序包含许多依赖项，您需要这些依赖项来快速启动和运行项目，并使用一组一致的、受支持的托管传递依赖项。  

## 2.构建代码 ##

Spring Boot不需要任何特定的代码布局就可以工作。然而，有一些最佳实践是有帮助的。
  
### 2.1.使用“默认”包 ###  
当一个类不包含包声明时，它被认为是在“默认包”中。一般不鼓励使用“默认包”，应避免使用。对于使用@ComponentScan、@configurationpropertiescan、@EntityScan或@springbootsapplication注释的Spring Boot应用程序，它可能会导致特定的问题，因为每个jar中的每个类都会被读取。  
### 2.2.定位主应用程序类 ###  
我们通常建议您将主应用程序类放在根包中，高于其他类。@springbootsapplication注释通常放在主类上，它隐式地为某些项定义了一个基本的“搜索包”。例如，如果您正在编写JPA应用程序，@springbootsapplication注释类的包用于搜索@Entity项。使用根包还允许组件扫描仅应用于项目。  

Application.java文件将声明main方法和基本的@springbootsapplication，如下所示：  

    package com.example.myapplication;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class Application {
    
    public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
    }
    
    }
     

## 3.配置类 ##  
Spring Boot支持基于Java的配置。尽管可以将SpringApplication与XML源一起使用，但我们通常建议您的主源是单个@Configuration类。通常，定义main方法的类很适合作为初级的@Configuration。  
### 3.1.导入其他配置类 ###  
您不必将所有的@Configuration放在一个类中。@Import注释可用于导入其他配置类。或者，您可以使用@components can自动获取所有Spring组件，包括@Configuration类。  
### 3.2导入XML配置 ###  
If you absolutely must use XML based configuration, we recommend that you still start with a @Configuration class. You can then use an @ImportResource annotation to load XML configuration files.  
## 4.自动配置 ##   
Spring Boot自动配置尝试根据您添加的jar依赖项自动配置Spring应用程序。例如，如果HSQLDB在您的类路径上，并且您没有手动配置任何数据库连接bean，那么Spring Boot将自动配置内存中的数据库。

您需要通过将@EnableAutoConfiguration或@springbootsapplication注释添加到一个@configuration类来选择自动配置。  
### 4.1.逐步取代自动配置 ###  
自动配置是无创的。在任何时候，您都可以开始定义自己的配置来替换自动配置的特定部分。例如，如果您添加了自己的数据源bean，那么默认的嵌入式数据库支持就会消失。

如果需要了解当前应用的自动配置以及原因，请使用--debug开关启动应用程序。这样做可以为选择的核心记录器启用调试日志，并将条件报告记录到控制台。  
### 4.2禁用特定的自动配置类 ###  
如果发现不希望应用的特定自动配置类，可以使用@springbootsapplication的exclude属性禁用它们，如下例所示：  

    import org.springframework.boot.autoconfigure.*;
    import org.springframework.boot.autoconfigure.jdbc.*;
    
    @SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
    public class MyApplication {
    }

如果类不在类路径上，则可以使用注释的excludeName属性并指定完全限定名。如果您喜欢使用@EnableAutoConfiguration而不是@springbootsapplication，那么exclude和excludeName也可以使用。最后，还可以使用spring.autoconfigure.exclude属性控制要排除的自动配置类的列表。






    