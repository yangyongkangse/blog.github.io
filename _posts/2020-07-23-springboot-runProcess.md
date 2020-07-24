---
layout:     post
title:      "Spring运行流程及原理解析"
subtitle:   ""
date:       2020-07-23  +0800
author:     "YangYongKang"
header-style: text
catalog: true
archive:
    - java
    - spring boot
tags:
    - java
    - spring
---

# 运行流程大概
### 1.准备环境

- 执行ApplicationContextInitializer  initialize方法,(ApplicationContextInitializer配置在META-INFO/spring.factories,)
- 监听器回调SpringApplicationRunListener  contextPrepared方法(SpringApplicationRunListener配置在META-INFO/spring.factories)
- 加载主配置类定义信息
- 监听器回调SpringApplicationRunListener  contextLoaded方法
### 2.刷新启动IOC容器

-  扫描加载所有容器中的组件
- 包括从META-INFO/spring.factories中获取所有EnableAutoConfiguration
### 3.回调容器中所有的ApplicationRunner和CommandlineRunner的run方法(ApplicationRunner和CommandlineRunner在IOC容器中)
### 4.监听器回调SpringApplicationRunListener回调finished 方法
## 启动流程
### 调用ConfigurableApplicationContext的run方法创建SpringApplication对象
```java
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
        //保存主配置类到primarySources属性
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        //判断当前是否是web应用类型并保存至webApplicationType属性
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        //从类路径下META-INFO/spring.factories配置的所有ApplicationContextInitializer并保存起来
        //spring.factories在spring-boot-autoconfigure包META-INFO下 目前是7个Initializer
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		//从类路径下META-INFO/spring.factories配置的所有ApplicationListener并保存起来
        //spring.factories在spring-boot-autoconfigure包META-INFO下 目前是11个Listener
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		//传配置的时候可以传多个,从多个配置类中找到有main方法的主配置类
        this.mainApplicationClass = deduceMainApplicationClass();
	}
```

- 2.2.5版本的的7个Initializer

![image.png](/img/1592488249006-ac68d059-4467-4e05-8da6-4e2b80f69479.png)

- 2.2.5版本的11个Listener

![image.png](/img/1592488377514-0fd79906-5426-4256-bc68-f2122c6417aa.png)
## 运行程序
```java
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
        //awt应用相关
		configureHeadlessProperty();
        //获取SpringApplicationRunListeners
        ////从类路径下META-INFO/spring.factories获取的所有Listeners  目前为1个
		SpringApplicationRunListeners listeners = getRunListeners(args);
        //回调所有的SpringApplicationRunListeners.starting()方法
		listeners.starting();
		try {
            //封装命令行参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			//准备环境
            ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			//创建环境完成后回调所有的SpringApplicationRunListeners.environmentPrepared()方法表示环境准备完成
            configureIgnoreBeanInfo(environment);
            //打印Banner
			Banner printedBanner = printBanner(environment);
            //创建ApplicationContext IOC容器  根据创建SpringApplication对象时判断的webApplicationType类型来创建
			//从这里来决定创建的是web的ioc还是普通的ioc
            context = createApplicationContext();
            ////根据 ConfigurableApplicationContext 获取到 ConfigurableApplicationContext相关实例
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
            //准备上下文 
            //1.将Environment保存到IOC
            //2.回调创建SpringApplication对象时保存的所有的ApplicationContextInitializer的initialize()方法 
            //3.回调创建SpringApplication对象时保存的所有的SpringApplicationRunListeners.contextPrepared()方法
			//4.prepareContext运行完成以后回调所有的SpringApplicationRunListeners.contextLoaded()方法
            prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			//刷新容器,IOC初始化的过程,加载IOC容器内所有的组件(配置类,bean,组件,自动配置等)如果是web应用还会创建嵌入式的tomcat
            refreshContext(context);
            //从IOC容器中获取所有的ApplicationRunner和CommandlineRunner
            //ApplicationRunner先于CommandlineRunner回调
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
            //所有的SpringApplicationRunListeners.started()方法
			listeners.started(context);
            //如果有 ApplicationRunner || CommandLineRunner 相关的bean 启动它
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
            //如果Spring启动失败 打印信息
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            //所有的SpringApplicationRunListeners.running()方法
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
        //整个SpringBoot应用启动以后返回启动的IOC容器,里面包含所有的组件
		return context;
	}
```

- 2.2.5版本的1个Listeners

![image.png](/img/1592488927492-6ff8648f-3d5b-4f79-909c-541263dea221.png)
