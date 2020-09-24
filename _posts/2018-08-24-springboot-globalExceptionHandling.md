---
layout: post
title: "SpringBoot 全局异常处理"
subtitle: ""
date: 2018-08-24 21:22:57 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Java
    - Spring Boot
tags:
    - Java
    - Spring Boot
---



#### 说明
spring boot全局异常是以ControllerAdvice注解和@ExceptionHandler注解实现对全局异常的特殊处理,当然你也可以使用
@Controller + @ExceptionHandler进行局部异常处理.

#### 局部异常处理

- 局部异常处理 @Controller + @ExceptionHandler

  局部异常主要用到的是@ExceptionHandler注解，此注解注解到类的方法上，当此注解里定义的异常抛出时，此方法会被执行。如果@ExceptionHandler所在的类是@Controller，则此方法只作用在此类,示例代码如下:
``` java
	/**
	 * author: yangyongkang
	 * date:2018/8/22
	 * time:13:58
	 * description:添加企业文化
	 **/
	//局部异常处理
	@ExceptionHandler(Exception.class)
	@PostMapping("/addCultureInfo")
	public ResponseEntity addCultureInfo(CultureModel cultureModel,HttpServletRequest request) {
		cultureService.addCultureInfo(cultureModel,request);
		return ResponseEntity.build(200, "企业文化添加成功!", cultureModel);
	}
```
#### 全局异常处理

- 全局异常处理 @ControllerAdvice + @ExceptionHandler

在spring 3.2中，新增了@ControllerAdvice 注解，可以用于定义@ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping中。
简单的说，进入Controller层的错误才会由@RestControllerAdvice，拦截器抛出的错误以及访问错误地址的情况@ControllerAdvice处理不了，由SpringBoot默认的异常处理机制处理,示例代码如下:

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {

	@ExceptionHandler(value = Exception.class)
	public ResponseEntity exceptionHandler(Exception e) {
		return ResponseEntity.build(Constant.ERROR_CODE, e.getMessage(), null);
	}

	@ExceptionHandler(value = ResourceNotFoundException.class)
	public ResponseEntity resourceNotFoundExceptionHandler(ResourceNotFoundException e) {
		return ResponseEntity.build(Constant.ERROR_CODE, e.getMessage(), null);
	}

	/**
	 * 自定义验证异常(参数传值)
	 */
	@ExceptionHandler(value = ConstraintViolationException.class)
	public ResponseEntity validatedBindException(ConstraintViolationException e) {
		return ResponseEntity.build(Constant.ERROR_CODE, e.getMessage(), null);
	}
}
```

