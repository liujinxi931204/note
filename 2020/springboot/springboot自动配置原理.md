SpringBoot相比较Spring而言，一大优点就是省去了纷繁复杂的XML配置，通过少量的配置就可以使用一个第三方的组件。这主要得益于SpringBoot自动装配。

## 什么是SpringBoot的自动装配  

现在提到自动装配，一般会和SpringBoot联系起来。但是，实际上，Spring Framework早就实现了这个功能。SpringBoot只是在其基础上，通过SPI的方式，做了进一步优化  

SpringBoot定义了一套接口规范，SpringBoot在启动时会扫描外部引用jar包中的META-INF/spring.factories文件，将文件中的类型信息加载到Spring容器，并执行类中定义的各种操作。对于外部jar来说，只需要按照SpringBoot定义的标准，就能将自己的功能装置进SpringBoot。  

### SpringBoot是如何实现自动装配的  

在使用SpringBoot的时候有一个非常重要的注解就是```@SpringBootApplication```，这个注解可以说是SpringBoot的核心，SpringBoot的自动装配也是依赖于这个注解的，下来就看一下这个注解  

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication{
    //省略
}
```

可以看到```@SpringBootApplication```注解是一个复合注解，其中重要的注解有三个：

+ ```@SpringBootConfiguration```：标注当前类是一个配置类，其实该注解也是一个复合注解，继承自```@Configuration```注解  

+ ```@ComponentScan```：用于类或者接口上主要是指定扫描路径下被@Component注解的bean。默认情况下，该注解会扫描启动类下所在的包下的所有类  

+ ```@EnableAutoConfiguration```：启动SpringBoot的自动配置机制  

可以看到我们SpringBoot自动配置需要的关心的注解就是```@EnableAutoConfiguration```，而且这个注解也是一个复合注解  

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
//省略
}
```

在这个注解中需要重点关注的是```@Import({AutoConfigurationImportSelector.class})```这个注解，```@Import```注解会导入一个普通类到Spring容器中，接受Spring容器的管理。在这里导入了一个AutoConfigurationImportSelector的类，也就是在这个类中实现了自动装配  

从上面可以看出，AutoConfigurationImportSelector这个类实现了SpringBoot的自动装配。先来看一下AutoConfigurationImportSelector的继承结构

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    
}

public interface DeferredImportSelector extends ImportSelector {
    
}

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}
```

可以看到AutoConfigurationImportSelector实现了DeferredImportSelector接口，而DeferredImportSelector接口又实现了ImportSelector接口，所以也就在AutoConfigurationImportSelector方法中实现了selectImports方法的具体逻辑。接下来看一下ImportSelecttor这个方法  

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    //判断自动装配开关是否打开
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        //获取所需要装配的bean
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

对于自动装配开关可以在application.properties中配置```Spring.boot.enableautoconfiguration```，默认为ture

重点说一下getAutoConfigurationEntry方法。

getAutoConfigurationEntry这个方法主要就是装配需要的bean  

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    //判断自动装配开关有没有打开
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        //获取EnableAutoConfiguration注解中的参数，排除一些不需要加载的配置类，在EnableAutoConfiguration注解中使用exclude()/excludeName()
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        //获取所有META-INF/spring-factories，其中有一个Key是EnableAutoConfiguration，它的value是一个以AutoConfiguration结尾类名的列表
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        //去重
        configurations = this.removeDuplicates(configurations);
        //获取第一步中排除加载的配置类
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        //检查排除加载的配置类是不是在configurations中，如果不在报错
        this.checkExcludedClasses(configurations, exclusions);
        //从configurations移除所有的排除加载的配置类
        configurations.removeAll(exclusions);
        //对configurations进行过滤，剔除掉@Conditional条件不成立的配置类
        configurations = this.getConfigurationClassFilter().filter(configurations);
        // 把AutoConfigurationImportEvent绑定在所有AutoConfigurationImportListener子类实例上
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        // 返回(configurations, exclusions)组
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```

来看一下getCandidateConfigurations方法  

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    //加载所有META-INF/spring.factories中的配置类
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

可以看到有一个SpringFactoriesLoader，它会读取MET-INF/spring-factories下的EnableAutoConfiguration配置。spring-factories的形式如下  

![image-20210723181334322](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/image-20210723181334322.png)

总结一下,selectImports方法主要做了以下几点  

+ 获取`META-INF/spring.factories`中`EnableAutoConfiguration`所对应的`Configuration`类列表

+ 由`@EnableAutoConfiguration`注解筛选出exclude、excludeName的参数，也就是不需要加载的配置类

+ 再由私有内部类`ConfigurationClassFilter`筛选一遍不满足`@Conditional`注解的配置类

### 自动配置生效  

接下来以`ServletWebServerFactoryAutoConfiguration`为例说明一下全局配置文件中的属性如何生效，比如`server.port=8088`是如何生效的（如果不配置也会有默认值，这个默认值来源于`org.apache.catalina.startup.Tomcat`）  

```java
// 标记为配置类
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
// 如果有ServletRequest.class 才会生效
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
// 把@ConfigurationProperties注解的类注入为Spring容器的Bean。
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
    //省略
}
```

可以看到`@EnableConfigurationProperties`注解里面了配置了`ServerPropertie.class` ，然后进入到ServerProperties类  

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {

	/**
	 * Server HTTP port.
	 */
	private Integer port;
```

在这个类上有一个注解`@ConfiguratioinProperties`，它的作用就是从配置文件中绑定属性到对应的bean上。也就是将application.properties中的server.port的值映射到ServerProperties类的port属性，而`@EnableConfigurationProperties`注解就是把这个已经绑定了属性的bean注入到Spring容器中  

### 总结  

SpringBoot启动的时候会通过`@EnableAutoConfiguratioin`注解找到META-INF/spring.factories配置文件中的所有自动配置类，在对这些配置类筛选之后对其进行加载，而这些配置类都是以AutoConfiguration结尾来命名的。它实际上就是一个JavaConfig形式的Spring容器配置，它能通过以Properties结尾命名的类中取得全局配置文件中的属性，例如`server.port`，而xxxProperties类是通过`@ConfiguratioinProperties`注解与全局配置文件对应的属性进行绑定的  

![srpingboot自动配置原理](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/srpingboot%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86.png)  





