---
layout: post
title: "Java中一个方法多个数据源事务控制解决方案"
subtitle: ""
date: 2020-09-24 16:19:22 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Java
    - 事务
    - Mysql    
    - Mybatis
tags:
    - Java
    - 事务
    - Mysql    
    - Mybatis
---

### 一、自定义事务注解

```java
/**
 * Author: yangyk
 * Date: 2020/9/24 14:35
 * Description: 多数据源事务控制器
 */
@java.lang.annotation.Target({java.lang.annotation.ElementType.METHOD, java.lang.annotation.ElementType.TYPE})
@java.lang.annotation.Documented
@java.lang.annotation.Retention(java.lang.annotation.RetentionPolicy.RUNTIME)
public @interface MultiDataSourceTransactional {
	/**
	 * Author: yangyk
	 * Date: 2020/9/24 14:35
	 * Description: 事务控制器数组
	 */
	String[] value() default {};
}

```

### 二、处理自定义注解切面控制

```java
/**
 * Author: yangyk
 * Date: 2020/9/24 14:35
 * Description: 多数据源事务控制器切面
 */
@Aspect
@Component
@Log4j2
public class MultiDataSourceTransactionalAspect {

	/**
	 * Author: yangyk
	 * Date: 2020/9/24 15:57
	 * Description: 定义切面，只置入带 @MultiDataSourceTransactional 注解的方法或类
	 */
	@Pointcut("@annotation(com.spring.learning.annonation.MultiDataSourceTransactional)")
	public void multiDataSourceTransactionalAspectAspect() {

	}

	/**
	 * Author: yangyk
	 * Date: 2020/9/24 15:57
	 * Description: 控制事务提交及回滚
	 */
	@Around(value = "multiDataSourceTransactionalAspectAspect() && @annotation(multiDataSourceTransactional)")
	public Object transactionalGroupAspectAround(ProceedingJoinPoint pjp, MultiDataSourceTransactional multiDataSourceTransactional) throws Throwable {
		Stack<DataSourceTransactionManager> dataSourceTransactionManagerStack = new Stack<>();
		Stack<TransactionStatus> transactionStatusStack = new Stack<>();
		if (multiDataSourceTransactional.value().length < 1) {
			log.warn("[开启事务失败]：无指定多数据源管理器");
			return null;
		}
		for (String item : multiDataSourceTransactional.value()) {
			DataSourceTransactionManager dbManager = (DataSourceTransactionManager) SpringContextUtil.getBean(item);
			TransactionStatus transactionStatus = dbManager.getTransaction(new DefaultTransactionDefinition());
			dataSourceTransactionManagerStack.push(dbManager);
			transactionStatusStack.push(transactionStatus);
		}
		try {
			Object obj = pjp.proceed();
			while (!dataSourceTransactionManagerStack.isEmpty()) {
				dataSourceTransactionManagerStack.pop().commit(transactionStatusStack.pop());
			}
			return obj;
		} catch (Exception e) {
			log.warn(e.getMessage());
			while (!dataSourceTransactionManagerStack.isEmpty()) {
				dataSourceTransactionManagerStack.pop().rollback(transactionStatusStack.pop());
			}
			return null;
		}
	}

}
```

### 三、自定义多数据源

#### 数据源1

```java
/**
 * Author: yangyk
 * Date: 2020/9/24 14:37
 * Description: 数据源配置
 */
@Configuration
@MapperScan(basePackages = "com.spring.learning.dao.cosmo", sqlSessionFactoryRef = "cosmoSqlSessionFactory")
//通过将指定的mapper指定给特定sqlSessionFactory,解决事务下datasource不能变更问题
public class CosmoDataSourceConfig {

	@Bean(name = "cosmoDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.cosmo")
	public DataSource cosmoDataSource() {
		return DataSourceBuilder.create().build();
	}

	// 事务控制器
	@Bean(name = "cosmoTransactionManager")
	public DataSourceTransactionManager cosmoTransactionManager() {
		return new DataSourceTransactionManager(cosmoDataSource());
	}

	@Bean(name = "cosmoSqlSessionFactory")
	public SqlSessionFactory cosmoSqlSessionFactory(@Qualifier("cosmoDataSource") DataSource cosmoDataSource)
			throws Exception {
		final MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();
		sessionFactory.setDataSource(cosmoDataSource);
		sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
				.getResources("classpath:/mapper/cosmo/*.xml"));
		//设置sql控制台打印
		com.baomidou.mybatisplus.core.MybatisConfiguration configuration = new com.baomidou.mybatisplus.core.MybatisConfiguration();
		configuration.setLogImpl(Log4j2Impl.class);
		sessionFactory.setConfiguration(configuration);
		return sessionFactory.getObject();
	}
}
```

#### 数据源2

```java
/**
 * Author: yangyk
 * Date: 2020/9/24 14:47
 * Description: 二维码数据源配置
 */
@Configuration
@MapperScan(basePackages = "com.spring.learning.dao.qrcode", sqlSessionFactoryRef = "qrcodeSqlSessionFactory")
//通过将指定的mapper指定给特定sqlSessionFactory,解决事务下datasource不能变更问题
public class QrcodeDataSourceConfig {

	@Bean(name = "qrcodeDataSource")
	@Primary
	@ConfigurationProperties(prefix = "spring.datasource.master")
	public DataSource qrcodeDataSource() {
		return DataSourceBuilder.create().build();
	}

	// 事务控制器
	@Bean(name = "qrcodeTransactionManager")
	@Primary
	public DataSourceTransactionManager qrcodeTransactionManager() {
		return new DataSourceTransactionManager(qrcodeDataSource());
	}

	@Bean(name = "qrcodeSqlSessionFactory")
	@Primary
	public SqlSessionFactory qrcodeSqlSessionFactory(@Qualifier("qrcodeDataSource") DataSource qrcodeDataSource)
			throws Exception {
		final MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();
		sessionFactory.setDataSource(qrcodeDataSource);
		sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
				.getResources("classpath:/mapper/qrcode/*.xml"));
		//设置sql控制台打印
		com.baomidou.mybatisplus.core.MybatisConfiguration configuration = new com.baomidou.mybatisplus.core.MybatisConfiguration();
		configuration.setLogImpl(Log4j2Impl.class);
		sessionFactory.setConfiguration(configuration);
		return sessionFactory.getObject();
	}
}
```

### 四、DAO层及Mapper层配置

- DAO层需要按照@MapperScan(basePackages = "com.spring.learning.dao.cosmo"配置
- Mapper层需要按照classpath:/mapper/cosmo/*.xml配置
- 几个数据源就配置几层

### 五、注解的使用

```java
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
```

[源码地址](https://github.com/yangyongkangse/spring-learning/tree/master/spring-datasource-transactional){:target="_blank"}

