### Spring Bean的生命周期  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/16deb7fd22aa8e16.png)  

#### 文字描述  

1. Bean容器在配置文件中找到Spring Bean的定义  

2. Bean容器使用Java Refection API创建Bean实例  

3. 如果声明了任何属性，声明的属性会被设置设置。如果属性本身是Bean，则将对其进行解析和设置 

4. 如果Bean类实现了BeanNameAware接口，则将通过传递Bean的名称来调用setBeanName()方法  

5. 如果Bean类实现了BeanFactoryAware接口，则将通过传递Bean的名称来调用setBeanFactory()方法  

6. 如果任何有关BeanFactory关联的BeanPostProcessors对象已加载Bean，则将在设置Bean属性之前调用postProcessorBeforeInitialization()方法  

7. 如果Bean类实现了InitializingBean接口，则在设置了配置文件中定义的所有Bean属性后，将调用afterPropertiesSet()方法  

8. 如果配置文件中包含int-method属性，则该属性的值将解析为Bean类中的方法名称，并将调用该方法  

9. 如果Bean Factory对象附加了任何Bean后置处理器，则将调用postProcessorAfterInitialization()方法  

10. 如果Bean类实现DisosableBean接口，则当Application不再需要Bean引用时，将调用destory()方法  

11. 如果配置文件中的Bean定义包含destory-method属性，那么将调用Bean类中的相应方法定义  

#### 实例演示

定义一个person类，实现BeanNameAware、BeanFactoryAware、ApplicationContextAware、InitializaingBean、DisposableBean接口，同时还定义了int-method、destroy-method

```java
package com.sogou;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class Person implements BeanNameAware, BeanFactoryAware, InitializingBean, ApplicationContextAware, DisposableBean {

    private String name;

    public Person() {
        System.out.println("person的构造方法");
    }

    public void setName(String name) {
        System.out.println("person的set方法");
        this.name = name;
    }

    public void init(){
        System.out.println("person的init方法");
    }

    public void destroyMethod(){
        System.out.println("person的destroy方法");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {

        System.out.println("BeanFactoryAware的setBeanFactory");

    }

    @Override
    public void setBeanName(String s) {

        System.out.println("BeanNameAware的setBeanName");

    }

    @Override
    public void destroy() throws Exception {

        System.out.println("DisposableBean的destroy");

    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean的afterPropertiesSet方法");

    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("ApplicationContextAware的setApplicationContext方法");

    }
}

```

定义一个MyPostProcessor类，实现BeanPostProcessor接口  

```java
package com.sogou;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("post processor的postProcessBeforeInitialization");
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("post processor的postProcessAfterInitialization");
        return null;
    }
}

```

配置文件，指定init-method和destroy-method属性

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="post" class="com.sogou.MyPostProcessor"></bean>
    <bean id="person" class="com.sogou.Person" init-method="init" destroy-method="destroyMethod">
        <property name="name" value="hh"></property>
    </bean>
</beans>
```

启动容器、销毁容器  

```java
package com.sogou;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class applicationContext {

    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("ApplicationContext.xml");
//        Person person = applicationContext.getBean("person", Person.class);
        classPathXmlApplicationContext.destroy();

    }
}

```

输出  

![image](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/image-20210319104149373.png)  

