# springboot 2.3.0学习笔记--Getting Started #
系统要求的配置：  
1.java版本1.8以上，并向上兼容java13 ，  
2.Spring Framework 5.2.4及以上。  
3.支持Maven 3.3 版本以上  
4.支持Gradle 6.0。  
Spring Boot支持以下嵌入式servlet容器：  
Name	Servlet Version  
Tomcat   9.0     4.0  
Jetty    9.4    3.1  
Undertow 2.0    4.0
## Java开发人员的安装说明  ##

您可以像使用任何标准Java库一样使用Spring Boot。为此，在类路径中包含适当的spring boot-*.jar文件。Spring Boot不需要任何特殊的工具集成，因此您可以使用任何IDE或文本编辑器。另外，Spring Boot应用程序没有什么特别之处，因此可以像运行其他Java程序一样运行和调试Spring Boot应用程序。  
Spring引导依赖项使用org.springframework.Boot groupId。通常，Maven POM文件继承自spring boot starter父项目，并向一个或多个“Starters”声明依赖关系。Spring Boot还提供了一个可选的Maven插件来创建可执行jar。  
下面的列表显示了一个典型的pom.xml文件：  

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    
    <!-- Inherit defaults from Spring Boot -->
    <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.BUILD-SNAPSHOT</version>
    </parent>
    
    <!-- Override inherited settings -->
    <description/>
    <developers>
    <developer/>
    </developers>
    <licenses>
    <license/>
    </licenses>
    <scm>
    <url/>
    </scm>
    <url/>
    
    <!-- Add typical dependencies for a web application -->
    <dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    </dependencies>
    
    <!-- Package as an executable jar -->
    <build>
    <plugins>
    <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    </plugins>
    </build>
    
    <!-- Add Spring repositories -->
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
    <repository>
    <id>spring-snapshots</id>
    <url>https://repo.spring.io/snapshot</url>
    <snapshots><enabled>true</enabled></snapshots>
    </repository>
    <repository>
    <id>spring-milestones</id>
    <url>https://repo.spring.io/milestone</url>
    </repository>
    </repositories>
    <pluginRepositories>
    <pluginRepository>
    <id>spring-snapshots</id>
    <url>https://repo.spring.io/snapshot</url>
    </pluginRepository>
    <pluginRepository>
    <id>spring-milestones</id>
    <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
    </pluginRepositories>
    </project>  
    
	  
Spring Boot CLI（命令行界面）是一个命令行工具，您可以使用它来快速创建Spring原型。它允许您运行Groovy脚本，这意味着您有一个熟悉的类似Java的语法，而没有那么多样板代码。  

  从早期版本的Spring Boot升级:  
如果要从1.x版本的Spring Boot进行升级，请查看project wiki上提供详细升级说明的“迁移指南”。还可以查看“发行说明”以获取每个发行版的“新的和值得注意的”功能列表。  
升级到新功能版本时，某些属性可能已重命名或删除。Spring Boot提供了一种在启动时分析应用程序环境和打印诊断信息的方法，但也可以在运行时为您临时迁移属性。要启用该功能，请将以下依赖项添加到项目中：  

    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-properties-migrator</artifactId>
    <scope>runtime</scope>
    </dependency>  

## 开发第一个Spring Boot应用程序:  
本节介绍如何开发一个简单的“Hello World！“突出了Spring Boot的一些关键特性的web应用程序。我们使用Maven来构建这个项目，因为大多数ide都支持它。  
### 1.创建POM ###  
我们需要从创建Maven pom.xml文件开始。xml是用于构建项目的配方。  

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    
    <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.BUILD-SNAPSHOT</version>
    </parent>
    
    <description/>
    <developers>
    <developer/>
    </developers>
    <licenses>
    <license/>
    </licenses>
    <scm>
    <url/>
    </scm>
    <url/>
    
    <!-- Additional lines to be added here... -->
    
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
    <repository>
    <id>spring-snapshots</id>
    <url>https://repo.spring.io/snapshot</url>
    <snapshots><enabled>true</enabled></snapshots>
    </repository>
    <repository>
    <id>spring-milestones</id>
    <url>https://repo.spring.io/milestone</url>
    </repository>
    </repositories>
    <pluginRepositories>
    <pluginRepository>
    <id>spring-snapshots</id>
    <url>https://repo.spring.io/snapshot</url>
    </pluginRepository>
    <pluginRepository>
    <id>spring-milestones</id>
    <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
    </pluginRepositories>
    </project>  


前面的清单应该为您提供一个工作构建。您可以通过运行mvn package来测试它  
### 2.添加类路径依赖项 ###  
Spring Boot提供了许多“启动器”，允许您将jar添加到类路径中。我们的(smoke tests)烟雾测试应用程序使用POM父部分中的spring boot starter父部分。spring boot starter父程序是一个特殊的启动程序，它提供了有用的Maven默认值。它还提供了一个依赖项管理部分，以便您可以省略“（blessed）受祝福”依赖项的版本标记。  
mvn dependency:tree命令打印项目依赖项的树表示。您可以看到spring boot starter父级本身不提供依赖项。要添加必要的依赖项，请编辑pom.xml并在父部分下面添加spring boot starter web依赖项： 

    <dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    </dependencies> 


### 3.编写代码 ###  
要完成我们的应用程序，我们需要创建一个Java文件。默认情况下，Maven从src/main/java编译源代码，因此需要创建该文件夹结构，然后添加名为src/main/java/Example.java的文件以包含以下代码：  

    @RestController
    @EnableAutoConfiguration
    public class Example {
    
    @RequestMapping("/")
    String home() {
    return "Hello World!";
    }
    
    public static void main(String[] args) {
    SpringApplication.run(Example.class, args);
    }
    
    }  

我们应用的最后一部分是Main方法。这只是一个遵循Java约定的应用程序入口点的标准方法。我们的Main方法通过调用run委托给Spring Boot的SpringApplication类。Spring application引导我们的应用程序，启动Spring，然后启动自动配置的Tomcat web服务器。我们需要将Example.class作为参数传递给run方法，以告诉SpringApplication哪个是主Spring组件。args数组也被传递以公开任何命令行参数。  
### 4.运行示例 ###  
此时，您的应用程序应该可以工作。由于使用了spring boot starter父POM，因此有一个有用的运行目标，可以用来启动应用程序。键入mvn spring boot：从根项目目录运行以启动应用程序。
#### 5.创建可执行Jar ####  
我们通过创建一个完全独立的可执行jar文件来完成我们的示例，该文件可以在生产环境中运行。可执行jar（有时称为“胖jar”）是包含编译类以及代码需要运行的所有jar依赖项的归档文件。如果您在目标目录中查找，应该会看到myproject-0.0.1-SNAPSHOT.jar