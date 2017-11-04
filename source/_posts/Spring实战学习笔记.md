---
title: Spring实战学习笔记
date: 2017-10-28 11:07:06
tags:
    - Java
    - Spring
---

***Spring核心使命就是简化 java 开发，Spring 的全部功能都是围绕这个核心使命实现的。***

#### Spring 核心

**Spring 的两个核心特性：依赖注入（DI）和面向切面编程（AOP）。**

##### 依赖注入
由 Spring 容器控制需要注入的对应接口的具体实现类，降低代码的耦合度。
依赖注入的四种方法：
1. Set注入
2. 构造器注入
3. 静态工厂的方法注入
4. 实例工厂的方法注入

##### 面向切面
将遍布应用的各处的功能点分离出来形成可重用的组件，例如：日志、事务管理、安全等系统服务，将这些服务模块化，通过声明的方式将它们应用到需要具体业务逻辑中。


##### Bean 的生命周期
1. Spring 对 Bean 进行实例化。

2. 使用依赖注入，Spring 按照 Bean 定义信息配置 Bean 所有属性。

3. 如果 Bean 类有实现BeanNameAware接口，工厂调用 Bean 的 setBeanName() 方法将Bean的ID传递给Bean。

  _实现该接口的Bean能够在初始化时知道自己在BeanFactory中对应的名字_

4. 如果 Bean 类有实现 BeanFactoryAware 接口，工厂调用setBeanFactory()方法传入工厂自身。、

  _实现该接口的Bean能够在初始化时知道自己所在的BeanFactory的名字_

5. 如果 Bean 实现了 ApplicationContextAware 接口，Spring 将调用 setApplicationContext() 方法，将 Bean 所在的上下文引用传递进来。

6. 如果 Bean 实现了 BeanPostProcessors 接口，那么其postProcessBeforeInitialization()方法将被将被调用。

  _实现一些在Bean完成初始化之前需要的一些逻辑功能_

7. 如果Bean类已实现InitializingBean接口，则执行他的afterProPertiesSet()方法。

8. 如果Bean使用init-method声明了初始化方法，则该初始化方法被调用。

9. 如果 Bean 实现了 BeanPostProcessors 接口，那么其postProcessAfterInitialization()方法将被将被调用。

10. Bean准备继续，驻留在应用上下文中等待被调用。

11. 当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean 接口，会调用它的destroy()方法。

12. 如果Bean使用destroy-method声明了销毁方法，则该方法被调用。



#### Bean 的装配

##### 条件化的装配
1. @Conditional
通过条件中类的matches()方法判断该类是否加载

2. @Profile
指定在对应的环境中是否加载对应的类

##### 接口多实现确定该注入的类
1. @Autowired 注解会对注解的set()方法或者属性进行自动装配，但是如果可以装配的类有多个有歧义的时候，我们需要对需要装配的类加注@Primary注解。
2. 我们也可以通过@Qualifier注解声明需要注入的Bean是哪一个。

##### Bean 的作用域
通过@Scope注解指定Bean的作用域：
1. 单例（默认）：整个应用中只创建一个Bean的实例
2. 原型：每次注入或者通过Spring应用上下文获取的时候，都会创建一个Bean的实例
3. 会话：为每一个会话创建一个bean实例
4. 请求：为每一个请求创建一个bean实例


#### 面向切面
在软件开发中，散布于应用中多处的功能点被称为横切关注点。
DI有助于应用对象之间的解耦，AOP可以实现横切关注点与它所影响的对象之间的解耦。

通知的5种类型：前置通知、后置通知、返回通知、异常通知、环绕通知。


#### Spring MVC

##### Spring MVC请求执行步骤
1. 请求到达Spring MVC的前端控制器DispacthServlet。
2. DispacthServlet进行请求分发，将请求发送给对应的Spring MVC控制器（controller）。
3. controller对请求包含的信息进行对应的处理，产生需要返回的信息称之为模型（model）。
4. controller将模型数据打包，标示出用于渲染模型的视图名。
5. controller将模型和视图名发送给DispacthServlet。
6. DispacthServlet使用视图解析器（view resolver）将逻辑视图名匹配为特定的视图实现，并将模型数据交付对应视图。

##### 配置DispacthServlet
AbstractAnnotationConfigDispatcherServletInitializer
_Servlet 3.0 中，容器会在类路径中查找实现了 javax.servlet.ServletContainerInitializer 接口的类，如果发现，则使用该实现类配置 Servlet 容器。_
_Spring中提供了 ServletContainerInitialize r接口的实现类 SpringServletContainerInitializer，在 SpringServletContainerInitializer 中又会反过来查找实现了 WebApplicationInitializer 类，并将配置任务交给它们完成。Spring 3.2 中引入了便利的 WebApplicationInitializer 的实现类 AbstractAnnotationConfigDispatcherServletInitializer。我们自己编写的程序之后只需要配置类继承并且重写AbstractAnnotationConfigDispatcherServletInitializer 中的方法就能快速部署到Servlet 3.0 以上的容器中，并且完成Servlet上下文的配置。_

1. getServletMapping（） 它会将一个或者多个路径映射到 DispacthServlet 上。
2. getServletConfigClasses()：定义 DispatcherServlet 应用上下文中的 beans。DispatcherServlet，加载包含 web 组件的 bean，比如 controllers，view resolvers 和 hanlder mappings。
3. getRootConfigClasses()：定义拦截器 ContextLoaderListener 应用上下文中的 beans。ContextLoaderListener，加载其他 bean，通常是一些中间层和数据层的组件（比如数据库配置 bean 等）。

##### 启用 Spring MVC
我们可以通过在配置类上加上@EnabWebMvc来启用SpringMVC组件，同时需要实现 WebMvcConfigurer 接口，WebMvcConfigurerAdapter 抽象类是对WebMvcConfigurer接口的简单抽象（增加了一些默认实现），所以一般选择选择直接继承WebMvcConfigurerAdapter,这时会全部使用默认配置：
1. 配置视图解析器，如果没有配置，那么Spring将会默认使用 BeanNameViewResolver 来查找ID与视图匹配的Bean，并查找的Bean要实现View接口。
2. 没有启用组件扫描，这样 Spring 只能找到显式声明在配置类中的控制器。
3. DispacthServlet会映射为应用默认的 Servlet，DispacthServlet 会处理所以得请求，包括对静态资源的请求。

我在继承 WebMvcConfigurerAdapter 后的配置类中相应的方法进行重写来满足我们的需求：异常处理（configureHandlerExceptionResolvers）、配置视图解析器（configureViewResolvers）、静态资源处理（addResourceHandlers）等。

##### Controller 中重定向或者转发
在请求处理完成后，使用 ` return "redirect:/xxx" ` 表示重定向到/xxx,使用` return "forward:/xxx" ` 表示前往/xxx


##### 表单校验
表单提交时，对于方法接收到的参数，必须校验参数对象中的属性是否符合要求，我们可以使用Java的校验API。

eg：

```
public class DeliverFeeTemplate{
    private Long id;

    @Min(value = 1L, message = "user.id.invalid")
    private Long shopId;

    @NotNull(message = "deliver.template.name.invalid")
    private String name;

    @Min(value = 0, message = "deliver.rule.fee.invalid")
    @Max(value = 100000000, message = "deliver.rule.fee.invalid")
    private Integer fee;
}

```

```
public Long createTemplate(@RequestBody DeliverFeeTemplate deliverFeeTemplate) {

}
```

如果请求中的deliverFeeTemplate对应的属性值不正确，则抛出相应的错误。

#### Spring 视图解析
Spring自带了13个视图解析器，可以逻辑视图名转化为物理实现。
InternalResourceViewResolver 一般用于JSP的解析。

#### Spring MVC 配置的替代方案
如果我们不想通过扩展 AbstractAnnotationConfigDispatcherServletInitializer 来快速的搭建 Spring MVC 环境，那么我们需要自行实现 DispacthServlet 和 ContextLoaderListener

1. 注册 Servlet
eg：
```
public class MyServletInitializer implements WebApplicationInitializer {

  @Override
  public void onStartup(ServletContext servletContext) throws ServletException{

    javax.servlet.ServletRegistration.Dynamic mySerlvlet = servletContext.addServlet("mySerlvlet", MyServlet.class);
    MyServlet.addMapping("/custom/**");

  }

}
```


2. 注册Filter
eg：
```
public class MyFilterInitializer implements WebApplicationInitializer {

  @Override
  public void onStartup(ServletContext servletContext) throws ServletException{

    javax.servlet.FilterRegistration.Dynamic myFilter = servletContext.addServlet("myFilter", MyFilter.class);
    myFilter.addMappingForUrlPatterns(null,false,"/custom/*");

  }

}
```

#### Web 应用异常处理

##### 将异常映射为指定的HTTP状态码
在继承 RuntimeException 的异常方法上使用 @ResponseStatus 注解

```
@ResponseStatus(value=HttpStatus.NOT_FOUND,reason="Shop Not Found")
public class ShopNotExistException extends RuntimeException {
    private static final long serialVersionUID = -9209919140458325125L;

    public ShopNotExistException(String message){
        super(message);
    }

    public ShopNotExistException(String message, Throwable cause){
        super(message , cause);
    }
}
```

##### 在处理请求的类中增加异常处理方法

```
@ExceptionHandler(ShopNotExistException.class)
public String handlerShopNotFoundException() {
    return "error/noShop";    
}
```

##### 控制器通知
在处理请求的类中增加异常处理方法只能处理该类中的方法抛出的异常，我们需要一个切面能够运行到整个应用的所有控制器中，Spring 3.2 中为我们提供了解决方案：控制器通知（controller advice）。
我们可以创建一个类，添加 @ControllerAdvice 的注解，在这个类中编写包含@ExceptionHandler 注解的方法，将能被全部控制器使用。

##### 保护 Web 应用
Spring Security 是为基于Spring 的应用程序提供的声明式安全保护的安全性框架。

为Spring MVC 启用Web 安全性功能的最简单配置

```
@EnableWebMvcSecurity
public class SecurityConfig extends WebSecurityConfigureAdapter {

}
```
WebSecurityConfigureAdapter 中包含三个方法
1. configure(WebSecurity) 配置Spring Security 的 Filter 链
2. configure(HttpSecurity) 配置如何通过拦截器保护请求
3. configure(AuthenticationManagerBuilder) 配置 user-detail 服务
