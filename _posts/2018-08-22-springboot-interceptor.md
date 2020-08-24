---
layout: post
title: "SpringBoot 2.x 自定义拦截器并解决静态资源访问被拦截问题"
subtitle: ""
date: 2018-08-22 21:05:14 +0800
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
在自己写一个spring boot demo的时候,想着根据session的过期时间来返回登录页面,在自定义完拦截器以后功能可以实现,但是静态资源却无法访问,通过查找文档发现spring boot 1.x和2.x是不一样,所以特别记录一下,留给后面的同学避免踩坑.

#### 1.x和2.x的静态资源访问区别
* 1.x的resources/static目录下的静态资源可以直接访问，并且访问路径上不用带static,当有配置自定义HandlerInterceptor拦截器时，请求静态资源路径不会被拦截。
* 2.x的如果自定义HandlerInterceptor拦截器时访问静态资源就会被同步拦截,这样为了实现session过期而跳转登录页面功能就会受影响.

#### 项目的目录
![目录](/img/a5o5e.jpg)

#### 自定义拦截器

```java
/**
 * UserSecurityInterceptor
 * Created with IntelliJ IDEA.
 * Author: yangyongkang
 * Date: 2018/8/22
 * Time: 14:20
 */
@Component
public class UserSecurityInterceptor implements HandlerInterceptor {
    @Autowired
    private RedisTemplate<String, Serializable> redisCacheTemplate;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        UserModel info = (UserModel) redisCacheTemplate.opsForValue().get(request.getSession().getId());
        if (info == null || StringUtils.isEmpty(info)) {
            response.sendRedirect(request.getContextPath() + "/view/login");
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
    }
}
```
* 因为我的session是放在redis里面的,所以只需要从redis里面取即可.

#### 配置访问路径及静态资源
```java
/**
 * 登陆拦截控制类
 * Created with IntelliJ IDEA.
 * Author: yangyongkang
 * Date: 2018/8/22
 * Time: 14:17
 */
@Configuration
public class WebMvcConfig implements  WebMvcConfigurer {
    @Autowired
    private UserSecurityInterceptor securityInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        InterceptorRegistration addInterceptor = registry.addInterceptor(securityInterceptor);
        // 排除配置
        addInterceptor.excludePathPatterns("/error");
        addInterceptor.excludePathPatterns("/static/**");//排除静态资源
        addInterceptor.excludePathPatterns("/view/login");
        addInterceptor.excludePathPatterns("/login/check");
        // 拦截配置
        addInterceptor.addPathPatterns("/**");
    }
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");//
    }
}
```

#### 页面加载方式
```html
//第一种在thymeleaf模版内部
    <link type="text/css" th:href="@{/static/css/page/bootstrap.min.css}" rel="stylesheet">
    <link type="text/css" th:href="@{/static/css/page/font-awesome.css}" rel="stylesheet">
    <link type="text/css" th:href="@{/static/css/page/animate.css}" rel="stylesheet">
    <link type="text/css" th:href="@{/static/css/page/style.css}" rel="stylesheet">
    <link type="text/css" th:href="@{/static/css/page/login.css}" rel="stylesheet">
    
//第二种
    <link href="/static/css/page/bootstrap.min.css" rel="stylesheet">
    <link href="/static/css/page/font-awesome.css" rel="stylesheet">
    <link href="/static/css/page/animate.css" rel="stylesheet">
    <link href="/static/css/page/style.css" rel="stylesheet">
```

#### 最后特别注意一下
* 在spring boot2.x中已经无法再使用WebMvcConfigurationAdapter,官方声明已过时.
* 可以继承WebMvcConfigurationSupport或者实现WebMvcConfigurer
```java
implements WebMvcConfigurer 
//不会覆盖@EnableAutoConfiguration关于WebMvcAutoConfiguration的配置
@EnableWebMvc + implements WebMvcConfigurer 
// 会覆盖@EnableAutoConfiguration关于WebMvcAutoConfiguration的配置
extends WebMvcConfigurationSupport 
//会覆盖@EnableAutoConfiguration关于WebMvcAutoConfiguration的配置
extends DelegatingWebMvcConfiguration
//会覆盖@EnableAutoConfiguration关于WebMvcAutoConfiguration的配置
```
