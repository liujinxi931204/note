## 简单介绍  

Spring是一个轻量级的控制反转(IOC)和面向切面(AOP)的容器框架。Spring使你能够编写更干净、更可管理、并且更容易测试的代码  

Spring MVC是Spring的一个模块，一个web框架。通过Dispatcher Servlet，ModelAndView和View Resolver，开发web应用变得很容易。主要针对的是网站应用程序或者服务开发--URL路由、session、模板引擎、静态web资源等  

Spring配置复杂，繁琐，所以推出Spring Boot，约定优于配置，简化了Spring的配置流程  

Spring Cloud构建于Spring Boot之上，是一个关注全局的服务治理框架  

### Spring和SpringMVC  

Spring是一个一站式的轻量级的java开发框架，核心是控制反转(IOC)和面向切面(AOP)，针对于开发的WEB层(SpringMVC)、业务层(IOC)、持久层(JDBC Template)等都提供了多种配置解决方案  

SpringMVC是Spring基础之上的一个MVC框架，主要处理web开发的路径映射和视图渲染，属于Spring框架中的WEB开发的一部分  

### SpringMVC和SpringBoot  

SpringMVC属于一个企业WEB开发的框架，涵盖面包括前端视图开发、文件配置、后台逻辑开发等，xml、config等配置相对比较复杂  

SpringBoot框架想对于SpringMVC框架来说，更专注于开发微服务后台接口，不开发前端视图  

### SpringBoot和SpringCloud  

SpringBoot使用了约定大于配置的理念，集成了快速开发的Spring多个插件，同时自动过滤不需要配置的多余插件，简化了项目的开发配置流程，一定程度上取消了xml配置，是一套快速配置开发的脚手架，能快速开发单个微服务  

SpringCloud大部分的功能插件都是基于SpringBoot去实现的，SpringCloud关注于全局的微服务整合和管理，将多个SpringBoot单体微服务进行整合以及管理；SpringCloud依赖于SpringBoot开发，而SpringBoot可以单独开发  

## 总结  

Spring是核心，提供了基础功能  

SpringMVC是基于Spring的一个MVC框架  

SpringBoot是为简化Spring配置的快速开发整合包  

SpringCloud是构建在SpringBoot之上的服务治理框架  

