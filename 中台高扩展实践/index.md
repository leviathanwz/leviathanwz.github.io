# 中台高扩展实践

# 中台高扩展实战
## 扩展点
### 基于Spring Bean的扩展点
> 使用流程：
>
> 1. 编写扩展点类
>
>       * 该类实现扩展点接口
>
>       * 该类添加扩展点注解（可能包括了扩展点定位信息）来标识扩展点
> 2. 执行扩展点类

* 定义扩展点坐标类：用于定位扩展点的位置
    * 包含扩展点名和ID 
    * 扩展点名是接口，用于分别不同的扩展点业务
    * 扩展点ID是其对应的业务区分点，用于分别此处使用的该扩展点业务的方法
```java 
package com.alibaba.cola.extension;

/**
 * Extension Coordinate(扩展坐标) is used to uniquely position an Extension
 */
public class ExtensionCoordinate {

    private final String extensionPointName;
    private final String bizScenarioUniqueIdentity;

    /**
     * Wrapper
     */
    private Class<?> extensionPointClass;
    private BizScenario bizScenario;

    public Class getExtensionPointClass() {
        return extensionPointClass;
    }

    public BizScenario getBizScenario() {
        return bizScenario;
    }

    public static ExtensionCoordinate valueOf(Class<?> extPtClass, BizScenario bizScenario){
        return new ExtensionCoordinate(extPtClass, bizScenario);
    }

    public ExtensionCoordinate(Class<?> extPtClass, BizScenario bizScenario){
        this.extensionPointClass = extPtClass;
        this.extensionPointName = extPtClass.getName();
        this.bizScenario = bizScenario;
        this.bizScenarioUniqueIdentity = bizScenario.getUniqueIdentity();
    }

    public ExtensionCoordinate(String extensionPoint, String bizScenario){
        this.extensionPointName = extensionPoint;
        this.bizScenarioUniqueIdentity = bizScenario;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((bizScenarioUniqueIdentity == null) ? 0 : bizScenarioUniqueIdentity.hashCode());
        result = prime * result + ((extensionPointName == null) ? 0 : extensionPointName.hashCode());
        return result;
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
          return true;
        }

        if (obj == null) {
          return false;
        }

        if (getClass() != obj.getClass()) {
          return false;
        }

        ExtensionCoordinate other = (ExtensionCoordinate) obj;
        if (bizScenarioUniqueIdentity == null) {
            if (other.bizScenarioUniqueIdentity != null) {
              return false;
            }
        } else if (!bizScenarioUniqueIdentity.equals(other.bizScenarioUniqueIdentity)) {
          return false;
        }
        if (extensionPointName == null) {
          return other.extensionPointName == null;
        } else {
          return extensionPointName.equals(other.extensionPointName);
        }
    }

    @Override
    public String toString() {
        return "ExtensionCoordinate [extensionPointName=" + extensionPointName + ", bizScenarioUniqueIdentity=" + bizScenarioUniqueIdentity + "]";
    }

}
```
* 定义扩展点注解，用于后续自动注册扩展点类，同时也要包含一些参数用于区分不同的扩展点和生成扩展点坐标
```java
package com.alibaba.cola.extension;

import org.springframework.stereotype.Component;

import java.lang.annotation.*;

/**
 * Extension
 */
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Component
public @interface Extension {
    String bizId()  default BizScenario.DEFAULT_BIZ_ID;
    String useCase() default BizScenario.DEFAULT_USE_CASE;
    String scenario() default BizScenario.DEFAULT_SCENARIO;
}
```

```java
package com.alibaba.cola.extension;

import org.springframework.stereotype.Component;

import java.lang.annotation.*;

/**
 * because {@link Extension} only supports single coordinates,
 * this annotation is a supplement to {@link Extension} and supports multiple coordinates
 *
 * @version 1.0
 * @see Extension
 */
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Component
public @interface Extensions {

    String[] bizId() default BizScenario.DEFAULT_BIZ_ID;

    String[] useCase() default BizScenario.DEFAULT_USE_CASE;

    String[] scenario() default BizScenario.DEFAULT_SCENARIO;

}
```

* 定义扩展点仓库类，使用Map，Key代表扩展点坐标，Value代表扩展点类
```java
package com.alibaba.cola.extension;

import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

/**
 * ExtensionRepository 
 */
@Component
public class ExtensionRepository {

    public Map<ExtensionCoordinate, ExtensionPointI> getExtensionRepo() {
        return extensionRepo;
    }

    private Map<ExtensionCoordinate, ExtensionPointI> extensionRepo = new HashMap<>();


}

```
* 定义扩展点接口
```java
package com.alibaba.cola.extension;

/**
 *  ExtensionPointI is the parent interface of all ExtensionPoints
 *  扩展点表示一块逻辑在不同的业务有不同的实现，使用扩展点做接口申明，然后用Extension（扩展）去实现扩展点。 
 */
public interface ExtensionPointI {

}


```

* 扩展点类自动注册：
    * 利用Spring应用上下文来获取所有带有特殊注解的Bean对象，也就是扩展点类
    * 定义扩展点注册类
      1. 通过反射来获取Bean对象对应的扩展点类
      2. 利用反射来获取该类对应的接口和以及注解中的值，生成扩展点坐标类
      3. 将扩展点坐标和扩展点类放入扩展点仓库

```java
package com.alibaba.cola.extension.register;

import com.alibaba.cola.extension.Extension;
import com.alibaba.cola.extension.ExtensionPointI;
import com.alibaba.cola.extension.Extensions;
import jakarta.annotation.PostConstruct;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;


import java.util.Map;

/**
 * ExtensionBootstrap
 *
 * @author Frank Zhang
 * @date 2020-06-18 7:55 PM
 */
@Slf4j
@Component
public class ExtensionBootstrap implements ApplicationContextAware {

    @Resource
    private ExtensionRegister extensionRegister;

    private ApplicationContext applicationContext;

    @PostConstruct
    public void init(){
        Map<String, ExtensionPointI> extMap = applicationContext.getBeansOfType(ExtensionPointI.class);
        for (ExtensionPointI ext : extMap.values()) {
            if (ext.getClass().isAnnotationPresent(Extension.class)) {
                extensionRegister.doRegistration(ext);
            }else if (ext.getClass().isAnnotationPresent(Extensions.class)){
                extensionRegister.doRegistrationExtensions(ext);
            }else {
                log.error("There is no annotation for @Extension or @Extension on this extension class:{}" , ext.getClass());
            }
        }
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

```java
package com.alibaba.cola.extension.register;

import com.alibaba.cola.extension.*;
import jakarta.annotation.Resource;
import org.springframework.aop.support.AopUtils;
import org.springframework.core.annotation.AnnotatedElementUtils;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.stereotype.Component;
import org.springframework.util.ClassUtils;
import org.springframework.util.ObjectUtils;

/**
 * ExtensionRegister
 *
 */
@Component
public class ExtensionRegister {

    /**
     * 扩展点接口名称不合法
     */
    private static final String EXTENSION_INTERFACE_NAME_ILLEGAL = "extension_interface_name_illegal";
    /**
     * 扩展点不合法
     */
    private static final String EXTENSION_ILLEGAL = "extension_illegal";
    /**
     * 扩展点定义重复
     */
    private static final String EXTENSION_DEFINE_DUPLICATE = "extension_define_duplicate";

    @Resource
    private ExtensionRepository extensionRepository;

    public final static String EXTENSION_EXTPT_NAMING = "ExtPt";


    public void doRegistration(ExtensionPointI extensionObject) {
        Class<?> extensionClz = extensionObject.getClass();
        if (AopUtils.isAopProxy(extensionObject)) {
            extensionClz = AopUtils.getTargetClass(extensionObject);
        }
        Extension extensionAnn = AnnotatedElementUtils.findMergedAnnotation(extensionClz, Extension.class);
        BizScenario bizScenario = BizScenario.valueOf(extensionAnn.bizId(), extensionAnn.useCase(), extensionAnn.scenario());
        ExtensionCoordinate extensionCoordinate = new ExtensionCoordinate(calculateExtensionPoint(extensionClz), bizScenario.getUniqueIdentity());
        ExtensionPointI preVal = extensionRepository.getExtensionRepo().put(extensionCoordinate, extensionObject);
        if (preVal != null) {
            String errMessage = "Duplicate registration is not allowed for :" + extensionCoordinate;
            throw new ExtensionException(EXTENSION_DEFINE_DUPLICATE, errMessage);
        }
    }

    public void doRegistrationExtensions(ExtensionPointI extensionObject){
        Class<?> extensionClz = extensionObject.getClass();
        if (AopUtils.isAopProxy(extensionObject)) {
            extensionClz = ClassUtils.getUserClass(extensionObject);
        }

        Extensions extensionsAnnotation = AnnotationUtils.findAnnotation(extensionClz, Extensions.class);

        //Support multiple extensions registration
        String[] bizIds = extensionsAnnotation.bizId();
        String[] useCases = extensionsAnnotation.useCase();
        String[] scenarios = extensionsAnnotation.scenario();
        for (String bizId : bizIds) {
            for (String useCase : useCases) {
                for (String scenario : scenarios) {
                    BizScenario bizScenario = BizScenario.valueOf(bizId, useCase, scenario);
                    ExtensionCoordinate extensionCoordinate = new ExtensionCoordinate(calculateExtensionPoint(extensionClz), bizScenario.getUniqueIdentity());
                    ExtensionPointI preVal = extensionRepository.getExtensionRepo().put(extensionCoordinate, extensionObject);
                    if (preVal != null) {
                        String errMessage = "Duplicate registration is not allowed for :" + extensionCoordinate;
                        throw new ExtensionException(EXTENSION_DEFINE_DUPLICATE, errMessage);
                    }
                }
            }
        }
    }

    /**
     * @param targetClz
     * @return
     */
    private String calculateExtensionPoint(Class<?> targetClz) {
        Class<?>[] interfaces = ClassUtils.getAllInterfacesForClass(targetClz);
        if (interfaces == null || interfaces.length == 0) {
            throw new ExtensionException(EXTENSION_ILLEGAL, "Please assign a extension point interface for " + targetClz);
        }
        for (Class intf : interfaces) {
            String extensionPoint = intf.getSimpleName();
            if (extensionPoint.contains(EXTENSION_EXTPT_NAMING)) {
                return intf.getName();
            }
        }
        String errMessage = "Your name of ExtensionPoint for " + targetClz +
                " is not valid, must be end of " + EXTENSION_EXTPT_NAMING;
        throw new ExtensionException(EXTENSION_INTERFACE_NAME_ILLEGAL, errMessage);
    }

}
```

* 定义扩展点执行器（本质上是一种策略模式）
    * 通过接口和业务模式，生成扩展点坐标，通过不同的扩展点坐标，定位到不同的扩展点类
    * 使用函数式接口来执行不同的扩展点类的方法
> 抽象扩展点执行器，包含了函数式接口的执行方法
```java
package com.alibaba.cola.extension.register;

import com.alibaba.cola.extension.BizScenario;
import com.alibaba.cola.extension.ExtensionCoordinate;

import java.util.function.Consumer;
import java.util.function.Function;

/**
 * @author fulan.zjf
 * @date 2017/12/21
 */
public abstract class AbstractComponentExecutor {

    /**
     * Execute extension with Response
     *
     * @param targetClz
     * @param bizScenario
     * @param exeFunction
     * @param <R> Response Type
     * @param <T> Parameter Type
     * @return
     */
    public <R, T> R execute(Class<T> targetClz, BizScenario bizScenario, Function<T, R> exeFunction) {
        T component = locateComponent(targetClz, bizScenario);
        return exeFunction.apply(component);
    }

    public <R, T> R execute(ExtensionCoordinate extensionCoordinate, Function<T, R> exeFunction){
        return execute((Class<T>) extensionCoordinate.getExtensionPointClass(), extensionCoordinate.getBizScenario(), exeFunction);
    }

    /**
     * Execute extension without Response
     *
     * @param targetClz
     * @param context
     * @param exeFunction
     * @param <T> Parameter Type
     */
    public <T> void executeVoid(Class<T> targetClz, BizScenario context, Consumer<T> exeFunction) {
        T component = locateComponent(targetClz, context);
        exeFunction.accept(component);
    }

    public <T> void executeVoid(ExtensionCoordinate extensionCoordinate, Consumer<T> exeFunction){
        executeVoid(extensionCoordinate.getExtensionPointClass(), extensionCoordinate.getBizScenario(), exeFunction);
    }

    protected abstract <C> C locateComponent(Class<C> targetClz, BizScenario context);
}
```

> 定义具体扩展点执行器，包含了寻找扩展点的逻辑。
```java
package com.alibaba.cola.extension;

import com.alibaba.cola.extension.register.AbstractComponentExecutor;
import jakarta.annotation.Resource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;


/**
 * ExtensionExecutor
 *
 */
@Component
public class ExtensionExecutor extends AbstractComponentExecutor {

    private static final String EXTENSION_NOT_FOUND = "extension_not_found";

    private Logger logger = LoggerFactory.getLogger(ExtensionExecutor.class);

    @Resource
    private ExtensionRepository extensionRepository;

    @Override
    protected <C> C locateComponent(Class<C> targetClz, BizScenario bizScenario) {
        C extension = locateExtension(targetClz, bizScenario);
        logger.debug("[Located Extension]: " + extension.getClass().getSimpleName());
        return extension;
    }

    /**
     * if the bizScenarioUniqueIdentity is "ali.tmall.supermarket"
     * <p>
     * the search path is as below:
     * 1、first try to get extension by "ali.tmall.supermarket", if get, return it.
     * 2、loop try to get extension by "ali.tmall", if get, return it.
     * 3、loop try to get extension by "ali", if get, return it.
     * 4、if not found, try the default extension
     *
     * @param targetClz
     */
    protected <Ext> Ext locateExtension(Class<Ext> targetClz, BizScenario bizScenario) {
        checkNull(bizScenario);

        Ext extension;

        logger.debug("BizScenario in locateExtension is : " + bizScenario.getUniqueIdentity());

        // first try with full namespace
        extension = firstTry(targetClz, bizScenario);
        if (extension != null) {
            return extension;
        }

        // second try with default scenario
        extension = secondTry(targetClz, bizScenario);
        if (extension != null) {
            return extension;
        }

        // third try with default use case + default scenario
        extension = defaultUseCaseTry(targetClz, bizScenario);
        if (extension != null) {
            return extension;
        }

        String errMessage = "Can not find extension with ExtensionPoint: " +
                targetClz + " BizScenario:" + bizScenario.getUniqueIdentity();
        throw new ExtensionException(EXTENSION_NOT_FOUND, errMessage);
    }

    /**
     * first try with full namespace
     * <p>
     * example:  biz1.useCase1.scenario1
     */
    private <Ext> Ext firstTry(Class<Ext> targetClz, BizScenario bizScenario) {
        logger.debug("First trying with " + bizScenario.getUniqueIdentity());
        return locate(targetClz.getName(), bizScenario.getUniqueIdentity());
    }

    /**
     * second try with default scenario
     * <p>
     * example:  biz1.useCase1.#defaultScenario#
     */
    private <Ext> Ext secondTry(Class<Ext> targetClz, BizScenario bizScenario) {
        logger.debug("Second trying with " + bizScenario.getIdentityWithDefaultScenario());
        return locate(targetClz.getName(), bizScenario.getIdentityWithDefaultScenario());
    }

    /**
     * third try with default use case + default scenario
     * <p>
     * example:  biz1.#defaultUseCase#.#defaultScenario#
     */
    private <Ext> Ext defaultUseCaseTry(Class<Ext> targetClz, BizScenario bizScenario) {
        logger.debug("Third trying with " + bizScenario.getIdentityWithDefaultUseCase());
        return locate(targetClz.getName(), bizScenario.getIdentityWithDefaultUseCase());
    }

    private <Ext> Ext locate(String name, String uniqueIdentity) {
        final Ext ext = (Ext) extensionRepository.getExtensionRepo().
                get(new ExtensionCoordinate(name, uniqueIdentity));
        return ext;
    }

    private void checkNull(BizScenario bizScenario) {
        if (bizScenario == null) {
            throw new IllegalArgumentException("BizScenario can not be null for extension");
        }
    }

}
```

* 扩展点自动配置
```java
package com.alibaba.cola.extension;

import com.alibaba.cola.extension.register.ExtensionBootstrap;
import com.alibaba.cola.extension.register.ExtensionRegister;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ExtensionAutoConfiguration {

    @Bean(initMethod = "init")
    @ConditionalOnMissingBean(ExtensionBootstrap.class)
    public ExtensionBootstrap bootstrap() {
        return new ExtensionBootstrap();
    }

    @Bean
    @ConditionalOnMissingBean(ExtensionRepository.class)
    public ExtensionRepository repository() {
        return new ExtensionRepository();
    }

    @Bean
    @ConditionalOnMissingBean(ExtensionExecutor.class)
    public ExtensionExecutor executor() {
        return new ExtensionExecutor();
    }

    @Bean
    @ConditionalOnMissingBean(ExtensionRegister.class)
    public ExtensionRegister register() {
        return new ExtensionRegister();
    }

}
```
> 一些思考：
> * 为什么要定义扩展点坐标类，使用接口和业务区分点？
> 方便扩展，具体应用的时候我们只需要关注**接口**和对应的**业务方式**，通过这两个参数来调用扩展点执行器，之后如何找寻具体的扩展点类，不用应用层去考虑
>    
> * 为什么不使用类似于Spring的模式，在扩展点仓库中直接new出一个对象？
> 暂时未知，有可能因为多线程的并发问题，扩展点对象可能包含有一定可读取和改变的状态
### 基于SPI的扩展点（一般用于软件的插件）
### 基于Dubbo的扩展点实现方式（详细查看文档）
## 模板方法
* 定义抽象扩展点类，包含一系列流程，作为模板方法
* 具体的扩展点类继承抽象扩展点类，进行重写方法和实现抽象方法
## 策略模式
* 扩展点执行器实际上是策略模式的一种

