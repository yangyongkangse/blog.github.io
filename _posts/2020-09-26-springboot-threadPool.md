---
layout: post
title: "SpringBoot中线程池的配置及使用"
subtitle: ""
date: 2020-09-26 03:54:22 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Java
    - Spring Boot
    - Thread    
tags:
    - Java
    - Spring Boot
    - Thread    
---

### 一、线程池配置

- SpringBoot有默认线程池配置,可以不自定义直接使用,但为了配置更加灵活,推荐使用自定义配置


```java
/**
 * author: yangyk
 * date: 2020/9/17 10:47
 * description:线程池配置
 */
@Configuration
@EnableAsync
public class ThreadPoolTaskConfig {


	/**
	 * 核心线程数（默认线程数）
	 */
	private static final int SIZE_CORE_POOL = 5;
	/**
	 * 最大线程数
	 */
	private static final int SIZE_MAX_POOL = 20;
	/**
	 * 允许线程空闲时间（单位：默认为秒）
	 */
	private static final int ALIVE_TIME = 60;
	/**
	 * 缓冲队列大小
	 */
	private static final int QUEUE_CAPACITY = 100;
	/**
	 * 线程池名前缀
	 */
	private static final String THREAD_NAME_PREFIX = "Async-Service-";

	/**
	 * 默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，
	 * 当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
	 * 当队列满了，就继续创建线程，当线程数量大于等于maxPoolSize后，开始使用拒绝策略拒绝
	 */
	@Bean("asyncServiceExecutor") // bean的名称，默认为首字母小写的方法名
	public ThreadPoolTaskExecutor asyncServiceExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(SIZE_CORE_POOL);
		executor.setMaxPoolSize(SIZE_MAX_POOL);
		executor.setQueueCapacity(QUEUE_CAPACITY);
		executor.setKeepAliveSeconds(ALIVE_TIME);
		executor.setThreadNamePrefix(THREAD_NAME_PREFIX);
		// 线程池对拒绝任务的处理策略
		// CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务
		executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
		// 初始化
		executor.initialize();
		return executor;
	}


}

```

### 二、异步处理
- 在需要异步线程处理的方法上添加@Async("asyncServiceExecutor")注解即可,asyncServiceExecutor为自定义线程池bean的名字

```java
@Component
@Log4j2
public class QrcodeAsyncServiceImpl implements QrcodeAsyncService {

	@Resource
	private com.spring.learning.dao.cosmo.BaseBarcodeInfoDao baseBarcodeInfoDao;

	@Resource
	private com.spring.learning.dao.qrcode.BaseDataGroupDao baseDataGroupDao;

	@Async("asyncServiceExecutor")
	@MultiDataSourceTransactional(value = {"qrcodeTransactionManager", "cosmoTransactionManager"})
	@Override
	public void createQrcodeAsync() {
		BaseBarcodeInfoModel model = new BaseBarcodeInfoModel();
		model.setCreateTime(new Date());
		model.setUpdateUserCode("test");
		model.setUpdateTime(new Date());
		model.setSpare1("test");
		baseBarcodeInfoDao.insert(model);
		BaseDataGroupModel baseDataGroupModel = new BaseDataGroupModel();
		baseDataGroupModel.setGroupCode("test1");
		baseDataGroupModel.setGroupDesc("test2");
		baseDataGroupDao.insert(baseDataGroupModel);
		throw new RuntimeException();
	}
}

```

[源码地址](https://github.com/yangyongkangse/spring-learning/tree/master/spring-datasource-transactional){:target="_blank"}

