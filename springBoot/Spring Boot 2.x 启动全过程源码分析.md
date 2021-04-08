# Spring Boot 2.x 启动全过程源码分析 #   
入口类的要求是最顶层包下面第一个含有 main 方法的类，使用注解 @SpringBootApplication 来启用 Spring Boot 特性，使用 SpringApplication.run 方法来启动 Spring Boot 项目。


    @SpringBootApplication
    public class SpringBootBestPracticeApplication {
    	public static void main(String[] args) {
    		SpringApplication.run(SpringBootBestPracticeApplication.class, args);
    }
    }  

来看一下这个类的run方法调用关系源码：  

    public static ConfigurableApplicationContext run( Class<?> primarySource,
    String... args) {
    	return run(new Class<?>[] { primarySource }, args);
    }  
    
    public static ConfigurableApplicationContext run(Class<?>[] primarySources,
    String[] args) {
    	return new SpringApplication(primarySources).run(args);
    }  

第一个参数primarySource：加载的主要资源类；  
第二个参数 args：传递给应用的应用参数。  
先用主要资源类来实例化一个 SpringApplication 对象，再调用这个对象的 run 方法，所以我们分两步来分析这个启动源码。 
 
## **SpringApplication 的实例化过程** ##
  

接着上面的 SpringApplication 构造方法进入以下源码：  

    public SpringApplication(Class<?>... primarySources) {
    	this(null, primarySources);
    }
    
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    	// 1、初始化资源加载器为 null
    	this.resourceLoader = resourceLoader;
    
    	// 2、断言主要加载资源类不能为 null，否则报错
    	Assert.notNull(primarySources, "PrimarySources must not be null");
    
    	// 3、初始化主要加载资源类集合并去重
    	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    
    	// 4、推断当前 WEB 应用类型
    	this.webApplicationType = deduceWebApplicationType();
    
    	// 5、设置应用上下 文初始化器
    	setInitializers((Collection) getSpringFactoriesInstances(
    	ApplicationContextInitializer.class));  
    
    	// 6、设置监听器
    	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    
    	// 7、推断主入口应用类
    	this.mainApplicationClass = deduceMainApplicationClass();
    
    }  

可知这个构造器类的初始化包括以下 7 个过程。  

### 1.资源初始化资源加载器为 null  ### 

    this.resourceLoader = resourceLoader;  

### 2.断言主要加载资源类不能为 null，否则报错  ### 

    Assert.notNull(primarySources, "PrimarySources must not be null");  

### 3.初始化主要加载资源类集合并去重  ### 

    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));  

### 4.推断当前 WEB 应用类型   ###

    this.webApplicationType = deduceWebApplicationType();  

来看下 deduceWebApplicationType 方法和相关的源码：  
    
    static WebApplicationType deduceFromClasspath() {
    		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
    				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
				//响应式 WEB 项目
    			return WebApplicationType.REACTIVE;
    		}
    		for (String className : SERVLET_INDICATOR_CLASSES) {
    			if (!ClassUtils.isPresent(className, null)) {
					//非 WEB 项目
    				return WebApplicationType.NONE;
    			}
    		}
			//SERVLET WEB 项目
    		return WebApplicationType.SERVLET;
    	}  
		
    private static final String REACTIVE_WEB_ENVIRONMENT_CLASS = "org.springframework."
    	+ "web.reactive.DispatcherHandler";

    private static final String MVC_WEB_ENVIRONMENT_CLASS = "org.springframework."+ "web.servlet.DispatcherServlet";

    private static final String[] WEB_ENVIRONMENT_CLASSES = { "javax.servlet.Servlet","org.springframework.web.context.ConfigurableWebApplicationContext"
     }; 

    public enum WebApplicationType {
    
    /**
     * 非 WEB 项目
     */
    NONE,
    
    /**
     * SERVLET WEB 项目
     */
    SERVLET,
    
    /**
    * 响应式 WEB 项目
     */
     REACTIVE
    
    }


这个就是根据类路径下是否有对应项目类型的类推断出不同的应用类型。  

### 5.设置应用上下文初始化器   ###

    setInitializers((Collection) getSpringFactoriesInstances(
    ApplicationContextInitializer.class));  

ApplicationContextInitializer 的作用是什么？源码如下。  

    public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
    
    		/**
     		* Initialize the given application context.
     		* @param applicationContext the application to configure
     		*/
    	void initialize(C applicationContext);
    
    }  


用来初始化指定的 Spring 应用上下文，如注册属性资源、激活 Profiles 等。  

来看下 setInitializers 方法源码，其实就是初始化一个 ApplicationContextInitializer 应用上下文初始化器实例的集合。  

    public void setInitializers(
    Collection<? extends ApplicationContextInitializer<?>> initializers) {
    	this.initializers = new ArrayList<>();
    	this.initializers.addAll(initializers);
    }  

再来看下这个初始化 getSpringFactoriesInstances 方法和相关的源码  

    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    		return getSpringFactoriesInstances(type, new Class<?>[] {});
    	}
    
    	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    		ClassLoader classLoader = getClassLoader();
    		// Use names and ensure unique to protect against duplicates
    		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    		AnnotationAwareOrderComparator.sort(instances);
    		return instances;
    	}  

设置应用上下文初始化器可分为以下 5 个步骤。  

5.1 获取当前线程上下文类加载器  

    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();  

5.2 获取 `ApplicationContextInitializer 的实例名称集合并去重  

    Set<String> names = new LinkedHashSet<>(
    SpringFactoriesLoader.loadFactoryNames(type, classLoader));  

loadFactoryNames 方法相关的源码如下  

    public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    	String factoryClassName = factoryClass.getName();
    	return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    	}
    
    	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
    
    	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    	MultiValueMap<String, String> result = cache.get(classLoader);
    	if (result != null) {
    	return result;
    	}
    
    	try {
    	Enumeration<URL> urls = (classLoader != null ?
    	classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
    	ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
    	result = new LinkedMultiValueMap<>();
    	while (urls.hasMoreElements()) {
    	URL url = urls.nextElement();
    	UrlResource resource = new UrlResource(url);
    	Properties properties = PropertiesLoaderUtils.loadProperties(resource);
    	for (Map.Entry<?, ?> entry : properties.entrySet()) {
    	List<String> factoryClassNames = Arrays.asList(
    	StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));
    	result.addAll((String) entry.getKey(), factoryClassNames);
    		}
    	}
    	cache.put(classLoader, result);
    	return result;
    	}
    	catch (IOException ex) {
    	throw new IllegalArgumentException("Unable to load factories from location [" +
    	FACTORIES_RESOURCE_LOCATION + "]", ex);
    	}
    }  


根据类路径下的 META-INF/spring.factories 文件解析并获取 ApplicationContextInitializer 接口的所有配置的类路径名称。  
 
spring-boot-autoconfigure-2.0.3.RELEASE.jar!/META-INF/spring.factories 的初始化器相关配置内容如下：  

    # Initializers
    org.springframework.context.ApplicationContextInitializer=\
    org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
    org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener  

5.3 根据以上类路径创建初始化器实例列表  

    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
    classLoader, args, names);
    
    private <T> List<T> createSpringFactoriesInstances(Class<T> type,
    Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
    Set<String> names) {
    List<T> instances = new ArrayList<>(names.size());
    for (String name : names) {
    try {
    Class<?> instanceClass = ClassUtils.forName(name, classLoader);
    Assert.isAssignable(type, instanceClass);
    Constructor<?> constructor = instanceClass
    .getDeclaredConstructor(parameterTypes);
    T instance = (T) BeanUtils.instantiateClass(constructor, args);
    instances.add(instance);
    }
    catch (Throwable ex) {
    throw new IllegalArgumentException(
    "Cannot instantiate " + type + " : " + name, ex);
    }
    }
    return instances;
    }  
    

5.4 初始化器实例列表排序  


    AnnotationAwareOrderComparator.sort(instances);  

5.5 返回初始化器实例列表  

    return instances;  

### 6设置监听器  ###  

    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));  

ApplicationListener 的作用是什么？源码如下。  

    @FunctionalInterface
    public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    
    /**
     * Handle an application event.
     * @param event the event to respond to
     */
    void onApplicationEvent(E event);
    
    }  

看源码，这个接口继承了 JDK 的 java.util.EventListener 接口，实现了观察者模式，它一般用来定义感兴趣的事件类型，事件类型限定于 ApplicationEvent的子类，这同样继承了 JDK 的 java.util.EventObject 接口。

设置监听器和设置初始化器调用的方法是一样的，只是传入的类型不一样，设置监听器的接口类型为： getSpringFactoriesInstances，对应的 spring-boot-autoconfigure-2.0.3.RELEASE.jar!/META-INF/spring.factories 文件配置内容请见下方。

    # Application Listeners
    org.springframework.context.ApplicationListener=\
    org.springframework.boot.autoconfigure.BackgroundPreinitializer  

可以看出目前只有一个 BackgroundPreinitializer 监听器。  

### 7.推断主入口应用类  ###   

    this.mainApplicationClass = deduceMainApplicationClass();
    
    private Class<?> deduceMainApplicationClass() {
    	try {
    		StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
    		for (StackTraceElement stackTraceElement : stackTrace) {
    			if ("main".equals(stackTraceElement.getMethodName())) {
    				return Class.forName(stackTraceElement.getClassName());
    			}
    		}
    	}catch (ClassNotFoundException ex) {
    	// Swallow and continue
    	}
    	return null;
	}  

这个推断入口应用类的方式有点特别，通过构造一个运行时异常，再遍历异常栈中的方法名，获取方法名为 main 的栈帧，从来得到入口类的名字再返回该类。  

## **SpringApplication 实例 run 方法运行过程** ##  

上面分析了 SpringApplication 实例对象构造方法初始化过程，下面继续来看下这个 SpringApplication 对象的 run 方法的源码和运行流程。  

![](https://github.com/CrazyZc1010/pic/raw/master/3b3696dcfafbdb23f6adf9a2eae92ba.png)

所以，我们可以按以下几步来分解 run方法的启动过程。  

#### 1、创建并启动计时监控类 ####  

    StopWatch stopWatch = new StopWatch();
    stopWatch.start();  

来看下这个计时监控类 StopWatch 的相关源码：  

    public void start() throws IllegalStateException {
    	start("");
    }
    
    public void start(String taskName) throws IllegalStateException {
    	if (this.currentTaskName != null) {
    		throw new IllegalStateException("Can't start StopWatch: it's already running");
    	}
    	this.currentTaskName = taskName;
    	this.startTimeMillis = System.currentTimeMillis();
    }  

首先记录了当前任务的名称，默认为空字符串，然后记录当前 Spring Boot 应用启动的开始时间。  

#### 2、初始化应用上下文和异常报告集合 ####  

    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();  

#### 3、设置系统属性 java.awt.headless 的值 ####  

    configureHeadlessProperty();  
设置该默认值为：true，Java.awt.headless = true 有什么作用？  
对于一个 Java 服务器来说经常要处理一些图形元素，例如地图的创建或者图形和图表等。这些API基本上总是需要运行一个X-server以便能使用AWT（Abstract Window Toolkit，抽象窗口工具集）。然而运行一个不必要的 X-server 并不是一种好的管理方式。有时你甚至不能运行 X-server,因此最好的方案是运行 headless 服务器，来进行简单的图像处理。  

#### 4、创建所有 Spring 运行监听器并发布应用启动事件 ####  

    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();  

来看下创建 Spring 运行监听器相关的源码：  


    private SpringApplicationRunListeners getRunListeners(String[] args) {
    	Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
    	return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
    	SpringApplicationRunListener.class, types, this, args));
    }
    
    SpringApplicationRunListeners(Log log,
    	Collection<? extends SpringApplicationRunListener> listeners) {
    	this.log = log;
    	this.listeners = new ArrayList<>(listeners);
    }  
创建逻辑和之前实例化初始化器和监听器的一样，一样调用的是 getSpringFactoriesInstances 方法来获取配置的监听器名称并实例化所有的类  

SpringApplicationRunListener 所有监听器配置在 spring-boot-2.0.3.RELEASE.jar!/META-INF/spring.factories 这个配置文件里面。  

    # Run Listeners
    org.springframework.boot.SpringApplicationRunListener=\
    org.springframework.boot.context.event.EventPublishingRunListener  


### 5、初始化默认应用参数类 ###  

    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);  

### 6、根据运行监听器和应用参数来准备 Spring 环境 ###  

    ConfigurableEnvironment environment = prepareEnvironment(listeners,
    applicationArguments);
    configureIgnoreBeanInfo(environment);  

下面我们主要来看下准备环境的 prepareEnvironment 源码：  

    private ConfigurableEnvironment prepareEnvironment(
    SpringApplicationRunListeners listeners,
    ApplicationArguments applicationArguments) {
    	// 6.1) 获取（或者创建）应用环境
    	ConfigurableEnvironment environment = getOrCreateEnvironment();
    
    	// 6.2) 配置应用环境
    	configureEnvironment(environment, applicationArguments.getSourceArgs());
    	listeners.environmentPrepared(environment);
    	bindToSpringApplication(environment);
    	if (this.webApplicationType == WebApplicationType.NONE) {
    		environment = new EnvironmentConverter(getClassLoader())
    			.convertToStandardEnvironmentIfNecessary(environment);
    	}
    	ConfigurationPropertySources.attach(environment);
    	return environment;
    }

#### 6.1 获取（或者创建）应用环境 ####  

    private ConfigurableEnvironment getOrCreateEnvironment() {
    	if (this.environment != null) {
    		return this.environment;
    	}
    	if (this.webApplicationType == WebApplicationType.SERVLET) {
    		return new StandardServletEnvironment();
    	}
    	return new StandardEnvironment();
    }  

这里分为标准 Servlet 环境和标准环境。  

#### 6.2 配置应用环境 ####

    protected void configureEnvironment(ConfigurableEnvironment environment,String[] args) {
    	configurePropertySources(environment, args);
    	configureProfiles(environment, args);
    }  

这里分为以下两步来配置应用环境。  
1. 配置 property sources  
1. 配置 Profiles  

这里主要处理所有 property sources 配置和 profiles 配置。  

### 7、创建 Banner 打印类 ###

    Banner printedBanner = printBanner(environment);  

这是用来打印 Banner 的处理类，这个没什么好说的。  

### 8、创建应用上下文 ###  

    context = createApplicationContext();  

来看下 createApplicationContext() 方法的源码：  

    protected ConfigurableApplicationContext createApplicationContext() {
    	Class<?> contextClass = this.applicationContextClass;
    	if (contextClass == null) {
    		try {
    			switch (this.webApplicationType) {
    			case SERVLET:
    				contextClass = Class.forName(DEFAULT_WEB_CONTEXT_CLASS);
    				break;
    			case REACTIVE:
    				contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
    				break;
    			default:
    				contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
    			}
    		}
    		catch (ClassNotFoundException ex) {
    			throw new IllegalStateException(
    				"Unable create a default ApplicationContext, "
    					+ "please specify an ApplicationContextClass",
    				ex);
    		}
    	}
    	return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
    }  

其实就是根据不同的应用类型初始化不同的上下文应用类。  

### 9、准备异常报告器 ###  

    exceptionReporters = getSpringFactoriesInstances(
    	SpringBootExceptionReporter.class,
    	new Class[] { ConfigurableApplicationContext.class }, context);  

逻辑和之前实例化初始化器和监听器的一样，一样调用的是 getSpringFactoriesInstances 方法来获取配置的异常类名称并实例化所有的异常处理类。  
该异常报告处理类配置在 spring-boot-2.0.3.RELEASE.jar!/META-INF/spring.factories 这个配置文件里面。  

    # Error Reporters
    org.springframework.boot.SpringBootExceptionReporter=\
    org.springframework.boot.diagnostics.FailureAnalyzers  

10、准备应用上下文  

    prepareContext(context, environment, listeners, applicationArguments,
    printedBanner);  
来看下 prepareContext() 方法的源码：  


    private void prepareContext(ConfigurableApplicationContext context,
        ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments, Banner printedBanner) {
    // 10.1）绑定环境到上下文
    context.setEnvironment(environment);

    // 10.2）配置上下文的 bean 生成器及资源加载器
    postProcessApplicationContext(context);

    // 10.3）为上下文应用所有初始化器
    applyInitializers(context);

    // 10.4）触发所有 SpringApplicationRunListener 监听器的 contextPrepared 事件方法
    listeners.contextPrepared(context);

    // 10.5）记录启动日志
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }

    // 10.6）注册两个特殊的单例bean
    context.getBeanFactory().registerSingleton("springApplicationArguments",
            applicationArguments);
    if (printedBanner != null) {
        context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
    }

    // 10.7）加载所有资源
    Set<Object> sources = getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[0]));

    // 10.8）触发所有 SpringApplicationRunListener 监听器的 contextLoaded 事件方法
    listeners.contextLoaded(context);
}

### 11、刷新应用上下文 ###  

    refreshContext(context);  

这个主要是刷新 Spring 的应用上下文，源码如下。  


    private void refreshContext(ConfigurableApplicationContext context) {
    	refresh(context);
    	if (this.registerShutdownHook) {
        	try {
            	context.registerShutdownHook();
        	}
        	catch (AccessControlException ex) {
            	// Not allowed in some environments.
        	}
    	}

    }  

### 12、应用上下文刷新后置处理 ###  

    afterRefresh(context, applicationArguments);  

看了下这个方法的源码是空的，目前可以做一些自定义的后置处理操作。  

    /**
     * Called after the context has been refreshed.
     * @param context the application context
     * @param args the application arguments
     */
    protected void afterRefresh(ConfigurableApplicationContext context,
    	ApplicationArguments args) {
    }

### 13、停止计时监控类 ###  

    stopWatch.stop();  

    public void stop() throws IllegalStateException {
    	if (this.currentTaskName == null) {
    		throw new IllegalStateException("Can't stop StopWatch: it's not running");
    	}
    	long lastTime = System.currentTimeMillis() - this.startTimeMillis;
    	this.totalTimeMillis += lastTime;
    	this.lastTaskInfo = new TaskInfo(this.currentTaskName, lastTime);
    	if (this.keepTaskList) {
    		this.taskList.add(this.lastTaskInfo);
    		}
    	++this.taskCount;
    	this.currentTaskName = null;
    }
    ```java
    计时监听器停止，并统计一些任务执行信息。
    
    **14、输出日志记录执行主类名、时间信息**
    ```java
    if (this.logStartupInfo) {
    	new StartupInfoLogger(this.mainApplicationClass)
    	.logStarted(getApplicationLog(), stopWatch);
    }

### 15、发布应用上下文启动完成事件 ###  
    listeners.started(context);  

触发所有 SpringApplicationRunListener 监听器的 started 事件方法。  

### 16、执行所有 Runner 运行器 ###  

    callRunners(context, applicationArguments);  

    private void callRunners(ApplicationContext context, ApplicationArguments args) {
    	List<Object> runners = new ArrayList<>();
    	runners.addAll(context.getBeansOfType (ApplicationRunner.class).values());
    	runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    	AnnotationAwareOrderComparator.sort(runners);
     	for (Object runner : new LinkedHashSet<>(runners)) {
    		if (runner instanceof ApplicationRunner) {
    		callRunner((ApplicationRunner) runner, args);
    		}
    		if (runner instanceof CommandLineRunner) {
    			callRunner((CommandLineRunner) runner, args);
    		}
    	}
    }


### 17、发布应用上下文就绪事件 ###  

    listeners.running(context);

触发所有 SpringApplicationRunListener 监听器的 running 事件方法。  

### 18、返回应用上下文 ###  

    return context;  

