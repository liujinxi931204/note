## 循环依赖  

循环依赖是因为对象A依赖B对象作为属性，同时对象B依赖对象A作为属性或者对象A依赖自己作为属性。  

![循环依赖](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)  

在Spring中循环依赖分为三种情况  

### 构造器循环依赖  

```java
public class CircleA{
    private CircleB circleB;
    
    public CircleA(CircleB circleB){
        this.circleB=circleB;
    }
}
```

```java
public class CircleB{
    private CircleA circleA;
    
    public CircleB(CircleA circleA){
        this.circleA=circleA;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    <bean id="circleA" class="com.sogou.CircleA">
        <constructor-arg index="0" ref="circleB"/>
    </bean>
    
    <bean id="circleB" class="com.sogou.CircleB">
        <constructor-arg index="0" ref="circleA"/>
    </bean>
</beans>
```

通过上面的配置，在初始化的时候就会抛出`BeanCurrentlyInCreationException`的异常  

```java
public static void main(String[] args) {
	// 报错原因： Requested bean is currently in creation: Is there an unresolvable circular reference?
	ApplicationContext context = new ClassPathXmlApplicationContext("circle/circle.xml");
}
```

这种通过**构造器注入的循环依赖，是无法解决的**  

### `prototype`范围的循环依赖处理  

`prototype`原型属于一种作用域。首先先来了解一下`Spring`中作用域：`scope`的概念  

在`Spring`容器中， 作用域`scope`是指其创建的`Bean`对象相对于其他`Bean`对象的请求可见范围   

最常用的是单例`singleton`作用域的`Bean`，在整个容器中只会存在一个`Bean`实例，所以每次获取的时候都会获取到同一个`Bean`实例  

`prototype`作用域的`Bean`，会在每次`Spring`容器调用获取`Bean`的时候创建一个全新的`Bean`，相当于`new Object()`  

**因为`Spring`容器对`prototype`作用域的`Bean`不进行缓存，因此无法提前暴露一个创建中的`Bean`，所以也是无法解决这种情况的循环依赖的**  

### `Setter`循环依赖  

```java
public class CircleA{
    private CircleB circleB;
    
    public void setCircleB(CircleB circleB){
        this.circleB=circleB;
    }
}
```

```java
public class CircleB{
    private CircleA circleA;
    
    public void setCircleA(CircleA circleA){
        this.circleA=circleA;
    }
}
```

```xml
<bean id="circleA" class="com.sogou" scope="singleton">
	<property name="circleB" ref="circleB"></property>
</bean>
<bean id="circleB" class="com.sogou" scope="singleton">
	<property name="circleA" ref="circleA"></property>
</bean>
```

这种情况下的循环依赖是可以解决的，下面就来分析一下  

## `Spring`如何解决循环依赖  

### 对象的实例化和初始化  

对象的创建分为两部分，一部分是实例化，此时堆内存中分配了内存空间，但是未对其属性进行赋值；另一部分为初始化，完成实例化后对对象的属性的赋值操作。因此类对象在内存空间中有两种状态，实例化状态（完成实例化，未初始化），完全状态（完成实例化，完成初始化）  

### 三级缓存  

`Spring`为了解决循环依赖，使用了三个`map`对象，也被称为三级缓存  

```java
/** Cache of singleton objects: bean name to bean instance. */
	// 一级缓存，存放的是完整状态的Bean，也就是完成实例化和初始化的Bean
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	// 三级缓存，存放的是ObjectFactory<?>
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
	// 二级缓存，存放的是半成品的Bean，完成了实例化，没有完成初始化的Bean
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

### 如何解决循环依赖  

以`XML`文件为例，代码如下，进入`debug`模式  

```java
public class SpringTestApplication {
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    }
}
```

进入到上下文的处理构造器中  

```java
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        // 从这里进入
        refresh();
    }
}
```

`refresh`方法，在`AbstractApplicationContext`内  

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 完成准备，设置时间啊等等，我们无需太关心
        prepareRefresh();

        // 无需多关心
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // bean factory 创建前的准备阶段
        prepareBeanFactory(beanFactory);

        try {
            //BeanFactory的处理过程，跟我们的对象创建关系不大 开始
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();
            //BeanFactory的处理过程，跟我们的对象创建关系不大 完成

            // Instantiate all remaining (non-lazy-init) singletons.
            //实例化，所有的非懒加载的单例bean对象，可见第二站入口来了，开启
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

根据注释，可以看到需要进入到`finishBeanFactoryInitialization`方法内  

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration();

    // Instantiate all remaining (non-lazy-init) singletons.
    // 实例化，所有的非懒加载的单例bean对象
    beanFactory.preInstantiateSingletons();
}
```

可以看到在方法的最后实例化`Bean`，进入看一下  

```java
@Override
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    // Iterate over a copy to allow for init methods which in turn register new bean definitions.
    // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // Trigger initialization of all non-lazy singleton beans...
    // 根据BeanDefinition 获取bean定义信息
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        // 首先 CircleA对象 此处非抽象类 单例 非懒加载才能进入，CircleA对象符合条件，进入
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                    ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                    getAccessControlContext());
                    }
                    else {
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                       ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                //A对象 进入
                getBean(beanName);
            }
        }
    }

    // 省略
}
```

进入`getBean(beanName)`方法  

```java
@Override
public Object getBean(String name) throws BeansException {
    // Spring源码内do开头，都是真正做事情的方法
    return doGetBean(name, null, null, false);
}
```

可以看到，真正的获取`Bean`的逻辑在`deGetBean`内，进入`doGetBean`方法内部  

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    // Eagerly check singleton cache for manually registered singletons.
    // 1. 获取单例bean对象 第四站入口，解决循环依赖的一个重要的方法
    Object sharedInstance = getSingleton(beanName);
    // 根据第四站代码分析，此时A对象为空，返回空，将继续执行
    if (sharedInstance != null && args == null) {
        //代码省略，和循环依赖无关
    }

    else {
        //代码省略，和循环依赖无关
        }

        if (!typeCheckOnly) {
            //代码省略，和循环依赖无关
        }

        try {
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            // Guarantee initialization of beans that the current bean depends on.
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
               //代码省略，和循环依赖无关

            // Create bean instance.
            // 3. 创建对象
            if (mbd.isSingleton()) {
                // 进入方法，解决循环依赖的一个重要的方法
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        // 5. 创建对象 实现A 对象实例
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        // Explicitly remove instance from singleton cache: It might have been put there
                        // eagerly by the creation process, to allow for circular reference resolution.
                        // Also remove any beans that received a temporary reference to the bean.
                        destroySingleton(beanName);
                        throw ex;
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }

            else if (mbd.isPrototype()) {
                // prototype类型，新创建一个Bean
                // It's a prototype -> create a new instance.
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            else {
                //代码省略
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    // Check if required type matches the type of the actual bean instance.
    if (requiredType != null && !requiredType.isInstance(bean)) {
        //代码省略
    }
    // 返回
    return (T) bean;
}
```

这里精简了一下`doGetBean`方法，解决循环依赖，有两个方法非常重要，一个是`getSingleton(String beanName, boolean allowEarlyReference)`方法，另一个是`getSingleton(beanName,ObjectFactory<?> singletonFactory)`。这两个方法已经在上面的注释中标注出来了  

先看一下第一个`getSingletion(String beanName, boolean allowEarlyReference)`方法  

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //  
    // 一级缓存中查找，是否存在CircleA对象，此时为空
    Object singletonObject = this.singletonObjects.get(beanName);
    // 判断一级缓存为空并且是否在创建中，我们在知识点中提及，类对象两种状态，实例化、完全状态，此时均未发生，因此不会进入判断
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            //从二级缓存中查找，是否存在CircleA对象，此时为空
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                //从三级缓存中查找，是否存在CircleA对象，此时为空
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    // 直接返回 空对象
    return singletonObject;
}
```

通过上面的注释可以看到，第一次获取的时候首先进入到`getSingleton(String beanName, boolean allowEarlyReference)`方法，这时在三级缓存中都没有查找到，因此返回一个`null`。  此时在`doGetBean`方法中继续向下运行到`getSingleton(beanName,ObjectFactory<?> singletonFactory)`  

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    // 4. 执行对象处理
    synchronized (this.singletonObjects) {
        // 一级缓存中获取A 对象，无
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            //进入判断
            if (this.singletonsCurrentlyInDestruction) {
                //省略代码
            }
            if (logger.isDebugEnabled()) {
               //省略代码
            }
            //省略代码
            if (recordSuppressedExceptions) {
                //省略代码
            }
            try {
                // 获取对象 ObjectFactory<?> 通过匿名内部类执行，匿名内部类 ，获取到A实例化对象，此时A 为实例化状态，创建中
                //这里就是执行传入进来的createBean(beanName, mbd, args)
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            //省略代码
        }
        return singletonObject;
    }
}
```

通过上面的注释可以看到，首先会再次从一级缓存中再次尝试查找，这次依然会查不到。然后代码继续向下运行到`singletonObject = singletonFactory.getObject()`。这里就是执行传入进来的`createBean(beanName, mbd, args)`。从而进入到`createBean(beanName, mbd, args)`方法中  

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    //省略代码

    try {
        //doCreateBean是创建Bean的真正执行者
        beanInstance = this.doCreateBean(beanName, mbdToUse, args);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Finished creating instance of bean '" + beanName + "'");
        }

        return beanInstance;
    } catch (ImplicitlyAppearedSingletonException | BeanCreationException var7) {
        throw var7;
    } catch (Throwable var8) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", var8);
    }
}
```

进入到`createBean`方法之后，主要的逻辑是由其内部的`doCreateBean`来实现的。接下来进入`doCreateBean`方法  

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        //实例CircleA创建
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    //省略代码

    // Allow post-processors to modify the merged bean definition.
    synchronized (mbd.postProcessingLock) {
        //省略代码
    }

    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    // 此处 A对象已创建，会进入判断
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isTraceEnabled()) {
            logger.trace("Eagerly caching bean '" + beanName +
                         "' to allow for resolving potential circular references");
        }
        //三级缓存，放入实例化对象A 此时A 未完全状态 缓存内状态，见代码下图示
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        // 6. 填充bean 
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
        if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
            throw (BeanCreationException) ex;
        }
        else {
            throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
        }
    }

    if (earlySingletonExposure) {
        //省略代码
    }

    // Register bean as disposable.
    //省略代码

    return exposedObject;
}
```

通过上面的注释可以知道，`doCreateBean`方法的主要工作有三个，从上刀下分别是：    

+ 通过反射的方式创建`Bean`  

  `instanceWrapper = createBeanInstance(beanName, mbd, args);`  

+ 三级缓存中方入实例化的对象  

  `addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));`  

+ 填充`Bean`的属性  

  `populateBean(beanName, mbd, instanceWrapper);`  

先来看一下在三级缓存中放入实例化对象  

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(singletonFactory, "Singleton factory must not be null");
        synchronized(this.singletonObjects) {
            if (!this.singletonObjects.containsKey(beanName)) {
                this.singletonFactories.put(beanName, singletonFactory);
                this.earlySingletonObjects.remove(beanName);
                this.registeredSingletons.add(beanName);
            }
        }
    }
```

可以看到，此时三级缓存中放入的是一个函数式接口`() -> getEarlyBeanReference(beanName, mbd, bean)`  

此时三级缓存中的内容如下图  

![三级缓存放入函数式接口](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98%E6%94%BE%E5%85%A5%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3.png) 

三级缓存放入函数式接口之后，就是对`CircleA`对象进行属性的装配，即此时需要对`CircleA`对象填充`CircleB`属性，因此进入到`populateBean(beanName, mbd, instanceWrapper);`  方法  

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 省略

    if (pvs != null) {
        //处理属性内容
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}
```

从上面的注释可以知道继续进入`applyPropertyValues(beanName, mbd, bw, pvs);`方法

```java
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    // 省略代码

    // Create a deep copy, resolving any references for values.
    List<PropertyValue> deepCopy = new ArrayList<>(original.size());
    boolean resolveNecessary = false;
    for (PropertyValue pv : original) {
        if (pv.isConverted()) {
            deepCopy.add(pv);
        }
        else {
            // B对象
            String propertyName = pv.getName();
            // 值为 null
            Object originalValue = pv.getValue();
            if (originalValue == AutowiredPropertyMarker.INSTANCE) {
                Method writeMethod = bw.getPropertyDescriptor(propertyName).getWriteMethod();
                if (writeMethod == null) {
                    throw new IllegalArgumentException("Autowire marker for property without write method: " + pv);
                }
                originalValue = new DependencyDescriptor(new MethodParameter(writeMethod, 0), true);
            }
            // 7. 解决B对象
            Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
            Object convertedValue = resolvedValue;
            //省略代码
        }
```

根据注释，继续进入`valueResolver.resolveValueIfNecessary(pv, originalValue)`方法  

```java
@Nullable
private Object resolveReference(Object argName, RuntimeBeanReference ref) {
    try {
        Object bean;
        Class<?> beanType = ref.getBeanType();
        if (ref.isToParent()) {
            BeanFactory parent = this.beanFactory.getParentBeanFactory();
            if (parent == null) {
                throw new BeanCreationException(
                    this.beanDefinition.getResourceDescription(), this.beanName,
                    "Cannot resolve reference to bean " + ref +
                    " in parent factory: no parent factory available");
            }
            if (beanType != null) {
                bean = parent.getBean(beanType);
            }
            else {
                // A对象依赖B对象，获取B对象
                bean = parent.getBean(String.valueOf(doEvaluate(ref.getBeanName())));
            }
        }
        //省略代码
    }
```

根据注释可以看到会进入到`getBean`方法，这不就是最开始获取对象`CircleA`时进入的方法吗？那么接下来的过程就和上面一样了，这里就不赘述了。

可以预见，在`CircleB`对象初始化之前，也会在三级缓存中放入函数式表达式。那么这时三级缓存的内容会如下图  

![三级缓存放入函数式接口二](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98%E6%94%BE%E5%85%A5%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3%E4%BA%8C.png)  

然后`CircleB`对象开始对属性进行填充，向下运行就会运行到`valueResolver.resolveValueIfNecessary(pv, originalValue)`的`parent.getBean(String.valueOf(doEvaluate(ref.getBeanName())))`。又回到了开始的地方。不过这时是向`CircleB`对象填充属性`circleA`  

重复上面的过程就会进入到`doGetBean`方法。在`doGetBean`方法内首先会进入到`getSingletion(String beanName, boolean allowEarlyReference)`方法。再来看一下这个方法  

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //  
    // 一级缓存中查找，是否存在CircleA对象，此时为空
    Object singletonObject = this.singletonObjects.get(beanName);
    // 判断一级缓存为空并且是否在创建中，我们在知识点中提及，类对象两种状态，实例化、完全状态，此时均未发生，因此不会进入判断
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            //从二级缓存中查找，是否存在CircleA对象，此时为空
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                //从三级缓存中查找，此时三级缓存中不为空，里面有一个函数式表达式
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    /**
                    * 这里执行getEarlyBeanReference(beanName, mbd, bean)
                    * 同时就这个对象放入二级缓存，并存三级缓存中删除
                    */
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

依旧是从三级缓存中获取`Bean`对象，在这个过程中，一级、二级缓存中依旧没有，但是三级缓存中有一个函数式表达式`() -> getEarlyBeanReference(beanName, mbd, bean)`  。所以这里就会去执行`getEarlyBeanReference(beanName, mbd, bean)`方法，实例化一个对象。接下来将这个对象方法二级缓存，并从三级缓存中删除刚刚这个函数式表达式  

这时三级缓存中的内容如下图  

![二级缓存放入半成品对象](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E6%94%BE%E5%85%A5%E5%8D%8A%E6%88%90%E5%93%81%E5%AF%B9%E8%B1%A1.png)  

可以看到这时二级缓存中有一个半成品的对象。这个对象完成了实例化，但是没有完初始化。继续在`doGetBean`方法中向下运行，会进入到另一个`getSingleton(beanName,ObjectFactory<?> singletonFactory)`方法中  

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName,
                                                          "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                                                          "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
            }
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                //三级缓存中已被删除，这里就不执行了，跳过
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                // Has the singleton object implicitly appeared in the meantime ->
                // if yes, proceed with it since the exception indicates that state.
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            catch (BeanCreationException ex) {
                if (recordSuppressedExceptions) {
                    for (Exception suppressedException : this.suppressedExceptions) {
                        ex.addRelatedCause(suppressedException);
                    }
                }
                throw ex;
            }
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                // 8. A对象属性对象 B对象完成初始化后执行
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

此时依然无法从一级缓存查找到对象，而三级缓存中的函数式表达式也已经被删除，因此执行到`addSingleton(beanName, singletonObject)`方法。进入这个方法  

```java
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        // 一级缓存放入CircleB对象 完全态
        this.singletonObjects.put(beanName, singletonObject);
        // 三级缓存删除CircleB对象
        this.singletonFactories.remove(beanName);
        // 二级缓存删除CircleB对象
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```

因为已经有了一个半成品的`CircleA`对象，所以此时`CircleB`对象就是一个完整的对象，即完成了实例化和初始化的对象。根据注释，此时将这个完整的`CircleB`对象放入到一级缓存`SingletonObjects`中，并从二级、三级缓存中删除这个对象不完整的对象  

此时三级缓存中的内容如下图所示  

![一级缓存放入完整对象](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E6%94%BE%E5%85%A5%E5%AE%8C%E6%95%B4%E5%AF%B9%E8%B1%A1.png)  

此时`CircleB`是一个完整的对象了，可以从一级缓存中获取到，因而`CircleA`也就可以完成初始化的过程了。即将`CircleB`赋给`CircleA`的`circleB`属性了。然后执行`addSingleton(beanName, singletonObject)`方法。从二级、三级缓存中删除不完整的`CircleA`对象  

此时三级缓存的内容如下  

![一级缓存放入完整对象二](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E6%94%BE%E5%85%A5%E5%AE%8C%E6%95%B4%E5%AF%B9%E8%B1%A1%E4%BA%8C.png)  

至此循环依赖的问题被解决了  

### 总结  

梳理一下整个循环依赖过程中的函数调用  

![循环依赖方法调用](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8.png)  

