### spring boot  启动配置原理
#### 启动原理
1. 创建springApplication对象
```java
    /**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param resourceLoader the resource loader to use
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #setSources(Set)
	 */
	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        // 保存主配置类，判断null
		this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        // 判断当前是否是一个web应用
        this.webApplicationType = deduceWebApplicationType();
        // 此类路径下找到"META-INF/spring.factories"文件中配置的ApplicationContextInitializer的值，然后保存起来
		setInitializers((Collection) getSpringFactoriesInstances(
                ApplicationContextInitializer.class));
        // 从类路径下找到"META-INF/spring.factories"文件中配置的ApplicationListener的值
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 从多个配置类中找到主配置类（根据main方法名字匹配）
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/spring-boot启动.png)
***
![](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/spring-boot-setListeners.png)

2. 执行run方法
```java
	/**
	 * Run the Spring application, creating and refreshing a new
	 * {@link ApplicationContext}.
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return a running {@link ApplicationContext}
	 */
	public ConfigurableApplicationContext run(String... args) {
		// 开始监听
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		// 获取"META-INF/spring.factories"文件中配置的SpringApplicationRunListeners
		SpringApplicationRunListeners listeners = getRunListeners(args);
		// 回调所有的listener的starting()方法
		listeners.starting();
		try {
			// 封装参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			// 创建、准备环境
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			
			configureIgnoreBeanInfo(environment);
			// 打印 spring图标
			Banner printedBanner = printBanner(environment);
			// 创建 ioc容器
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
			// 刷新容器;ioc容器初始化,将bean放到容器中
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			// 结束监听 
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
			listeners.running(context);
			return context;
	}

```
- 其中有个公共方法：根据type的name属性从"META-INF/spring.factories"中取出对应的value。
```java 
SpringFactoriesLoader.loadFactoryNames(type, classLoader));
```


##### 运行流程

##### 自动配置原理
