## 1 Annotation（自定义配置）模块

Annotation的代码实现我们还是沿用Mini版本的，保持不变，复制过来便可。

### 1.1 @GPService

@GPService代码如下：

```java
package com.tom.spring.formework.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 业务逻辑，注入接口
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface GPService {

   String value() default "";
	 
}
```

### 1.2 @GPAutowired

@GPAutowired代码如下：

```java
package com.tom.spring.formework.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 自动注入
 */
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface GPAutowired {

   String value() default "";
	 
}
```

### 1.3 @GPController

@GPController代码如下：

```java
package com.tom.spring.formework.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 页面交互
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface GPController {

   String value() default "";
	 
}
```

### 1.4 @GPRequestMapping

@GPRequestMapping代码如下：

```java
package com.tom.spring.formework.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 请求URL
 */
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface GPRequestMapping {

   String value() default "";
	 
}
```

### 1.5 @GPRequestParam

@GPRequestParam代码如下：

```java
package com.tom.spring.formework.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 请求参数映射
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface GPRequestParam {

   String value() default "";
	 
}
```

## 2 core（顶层接口）模块

### 2.1 GPFactoryBean

关于顶层接口设计，通过前面的学习我们了解了FactoryBean的基本作用，在此不做过多解释。

```java
package com.tom.spring.formework.core;

public interface GPFactoryBean {

}
```

### 2.2 GPBeanFactory

作为所有IoC容器的顶层设计，前面也已经详细介绍了BeanFactory的作用。

```java
package com.tom.spring.formework.core;

/**
 * 单例工厂的顶层设计
 */
public interface GPBeanFactory {

    /**
     * 根据beanName从IoC容器中获得一个实例Bean
     * @param beanName
     * @return
     */
    Object getBean(String beanName) throws Exception;

    public Object getBean(Class<?> beanClass) throws Exception;

}
```

## 3 beans（配置封装）模块

### 3.1 GPBeanDefinition

BeanDefinition主要用于保存Bean相关的配置信息。

```java
package com.tom.spring.formework.beans.config;

//用来存储配置文件中的信息
//相当于保存在内存中的配置
public class GPBeanDefinition {

    private String beanClassName;  //原生Bean的全类名
    private boolean lazyInit = false; //标记是否延时加载
    private String factoryBeanName;  //保存beanName，在IoC容器中存储的key
		
    public String getBeanClassName() {
        return beanClassName;
    }
		
    public void setBeanClassName(String beanClassName) {
        this.beanClassName = beanClassName;
    }
		
    public boolean isLazyInit() {
        return lazyInit;
    }
		
    public void setLazyInit(boolean lazyInit) {
        this.lazyInit = lazyInit;
    }
		
    public String getFactoryBeanName() {
        return factoryBeanName;
    }
		
    public void setFactoryBeanName(String factoryBeanName) {
        this.factoryBeanName = factoryBeanName;
    }
		
}
```

### 3.2 GPBeanWrapper

BeanWrapper主要用于封装创建后的对象实例，代理对象（Proxy Object）或者原生对象（Original Object）都由BeanWrapper来保存。

```java
package com.tom.spring.formework.beans;

public class GPBeanWrapper {

    private Object wrappedInstance;
    private Class<?> wrappedClass;

    public GPBeanWrapper(Object wrappedInstance){
        this.wrappedInstance = wrappedInstance;
    }

    public Object getWrappedInstance(){
        return this.wrappedInstance;
    }

    //返回代理以后的Class
    //可能会是这个 $Proxy0
    public Class<?> getWrappedClass(){
        return this.wrappedInstance.getClass();
    }
}
```

## 4 context（IoC容器）模块

### 4.1 GPAbstractApplicationContext

IoC容器实现类的顶层抽象类，实现IoC容器相关的公共逻辑。为了尽可能地简化，在这个Mini版本中，暂时只设计了一个refresh()方法。

```java
package com.tom.spring.formework.context.support;

/**
 * IoC容器实现的顶层设计
 */
public abstract class GPAbstractApplicationContext {

    //受保护，只提供给子类重写
    public void refresh() throws Exception {}
		
}
```

### 4.2 GPDefaultListableBeanFactory

DefaultListableBeanFactory是众多IoC容器子类的典型代表。在Mini版本中我只做了一个简单的设计，就是定义顶层的IoC缓存，也就是一个Map，属性名字也和原生Spring保持一致，定义为beanDefinitionMap，以方便大家对比理解。

```java
package com.tom.spring.formework.beans.support;

import com.tom.spring.formework.beans.config.GPBeanDefinition;
import com.tom.spring.formework.context.support.GPAbstractApplicationContext;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class GPDefaultListableBeanFactory extends GPAbstractApplicationContext{

    //存储注册信息的BeanDefinition
    protected final Map<String, GPBeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, GPBeanDefinition>();
		
}
```

### 4.3 GPApplicationContext

ApplicationContext是直接接触用户的入口，主要实现DefaultListableBeanFactory的refresh()方法和BeanFactory的getBean()方法，完成IoC、DI、AOP的衔接。

```java
package com.tom.spring.formework.context;

import com.tom.spring.formework.annotation.GPAutowired;
import com.tom.spring.formework.annotation.GPController;
import com.tom.spring.formework.annotation.GPService;
import com.tom.spring.formework.beans.GPBeanWrapper;
import com.tom.spring.formework.beans.config.GPBeanPostProcessor;
import com.tom.spring.formework.core.GPBeanFactory;
import com.tom.spring.formework.beans.config.GPBeanDefinition;
import com.tom.spring.formework.beans.support.GPBeanDefinitionReader;
import com.tom.spring.formework.beans.support.GPDefaultListableBeanFactory;
import java.lang.reflect.Field;
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 按之前源码分析的套路，IoC、DI、MVC、AOP
 */
public class GPApplicationContext extends GPDefaultListableBeanFactory implements GPBeanFactory {

    private String [] configLoactions;
    private GPBeanDefinitionReader reader;

    //单例的IoC容器缓存
    private Map<String,Object> factoryBeanObjectCache = new ConcurrentHashMap<String, Object>();
		
    //通用的IoC容器
    private Map<String,GPBeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<String, GPBeanWrapper>();

    public GPApplicationContext(String... configLoactions){
		
        this.configLoactions = configLoactions;
				
        try {
            refresh();
        } catch (Exception e) {
            e.printStackTrace();
        }
				
    }


    @Override
    public void refresh() throws Exception{
		
        //1. 定位，定位配置文件
        reader = new GPBeanDefinitionReader(this.configLoactions);

        //2. 加载配置文件，扫描相关的类，把它们封装成BeanDefinition
        List<GPBeanDefinition> beanDefinitions = reader.loadBeanDefinitions();

        //3. 注册，把配置信息放到容器里面（伪IoC容器）
        doRegisterBeanDefinition(beanDefinitions);

        //4. 把不是延时加载的类提前初始化
        doAutowrited();
				
    }

    //只处理非延时加载的情况
    private void doAutowrited() {
		
        for (Map.Entry<String, GPBeanDefinition> beanDefinitionEntry : super.beanDefinitionMap.entrySet()) {
				
           String beanName = beanDefinitionEntry.getKey();
					 
           if(!beanDefinitionEntry.getValue().isLazyInit()) {
					 
               try {
							 
                   getBean(beanName);
									 
               } catch (Exception e) {
							 
                   e.printStackTrace();
									 
               }
							 
           }
					 
        }
				
    }

    private void doRegisterBeanDefinition(List<GPBeanDefinition> beanDefinitions) throws Exception {

        for (GPBeanDefinition beanDefinition: beanDefinitions) {
				
            if(super.beanDefinitionMap.containsKey(beanDefinition.getFactoryBeanName())){
                throw new Exception("The “" + beanDefinition.getFactoryBeanName() + "” is exists!!");
            }
            super.beanDefinitionMap.put(beanDefinition.getFactoryBeanName(),beanDefinition);
						
        }
				
        //到这里为止，容器初始化完毕
    }
		
    public Object getBean(Class<?> beanClass) throws Exception {
		
        return getBean(beanClass.getName());
				
    }

    //依赖注入，从这里开始，读取BeanDefinition中的信息
    //然后通过反射机制创建一个实例并返回
    //Spring做法是，不会把最原始的对象放出去，会用一个BeanWrapper来进行一次包装
    //装饰器模式：
    //1. 保留原来的OOP关系
    //2. 需要对它进行扩展、增强（为了以后的AOP打基础）
    public Object getBean(String beanName) throws Exception {

        return null;
    }

    public String[] getBeanDefinitionNames() {
		
        return this.beanDefinitionMap.keySet().toArray(new String[this.beanDefinitionMap. size()]);
				
    }

    public int getBeanDefinitionCount(){
		
        return this.beanDefinitionMap.size();
				
    }

    public Properties getConfig(){
		
        return this.reader.getConfig();
				
    }
		
}
```

### 4.4 GPBeanDefinitionReader

根据约定，BeanDefinitionReader主要完成对application.properties配置文件的解析工作，实现逻辑非常简单。通过构造方法获取从ApplicationContext传过来的locations配置文件路径，然后解析，扫描并保存所有相关的类并提供统一的访问入口。

```java
package com.tom.spring.formework.beans.support;

import com.tom.spring.formework.beans.config.GPBeanDefinition;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

//对配置文件进行查找、读取、解析
public class GPBeanDefinitionReader {

    private List<String> registyBeanClasses = new ArrayList<String>();

    private Properties config = new Properties();

    //固定配置文件中的key，相对于XML的规范
    private final String SCAN_PACKAGE = "scanPackage";

    public GPBeanDefinitionReader(String... locations){
		
        //通过URL定位找到其所对应的文件，然后转换为文件流
        InputStream is = this.getClass().getClassLoader().getResourceAsStream(locations[0]. replace("classpath:",""));
        try {
            config.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(null != is){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        doScanner(config.getProperty(SCAN_PACKAGE));
				
    }

    private void doScanner(String scanPackage) {
		
        //转换为文件路径，实际上就是把.替换为/
        URL url = this.getClass().getClassLoader().getResource("/" + scanPackage.replaceAll ("\\.","/"));
        File classPath = new File(url.getFile());
        for (File file : classPath.listFiles()) {
            if(file.isDirectory()){
                doScanner(scanPackage + "." + file.getName());
            }else{
                if(!file.getName().endsWith(".class")){ continue;}
                String className = (scanPackage + "." + file.getName().replace(".class",""));
                registyBeanClasses.add(className);
            }
        }
				
    }

    public Properties getConfig(){
		
        return this.config;
				
    }

    //把配置文件中扫描到的所有配置信息转换为GPBeanDefinition对象，以便于之后的IoC操作
    public List<GPBeanDefinition> loadBeanDefinitions(){
		
        List<GPBeanDefinition> result = new ArrayList<GPBeanDefinition>();
        try {
				
            for (String className : registyBeanClasses) {
                Class<?> beanClass = Class.forName(className);
                if(beanClass.isInterface()) { continue; }

                result.add(doCreateBeanDefinition(toLowerFirstCase(beanClass.getSimpleName()), beanClass.getName()));

                Class<?> [] interfaces = beanClass.getInterfaces();
                for (Class<?> i : interfaces) {
                    result.add(doCreateBeanDefinition(i.getName(),beanClass.getName()));
                }

            }
						
        }catch (Exception e){
            e.printStackTrace();
        }
				
        return result;
				
    }


    //把每一个配置信息解析成一个BeanDefinition
    private GPBeanDefinition doCreateBeanDefinition(String factoryBeanName,String beanClassName){
		
        GPBeanDefinition beanDefinition = new GPBeanDefinition();
        beanDefinition.setBeanClassName(beanClassName);
        beanDefinition.setFactoryBeanName(factoryBeanName);
        return beanDefinition;
				
    }

    //将类名首字母改为小写
    //为了简化程序逻辑，就不做其他判断了，大家了解就好
    private String toLowerFirstCase(String simpleName) {
		
        char [] chars = simpleName.toCharArray();
        //因为大小写字母的ASCII码相差32
        //而且大写字母的ASCII码要小于小写字母的ASCII码
        //在Java中，对char做算术运算，实际上就是对ASCII码做算术运算
        chars[0] += 32;
        return String.valueOf(chars);
				
    }

}
```

### 4.5 GPApplicationContextAware

相信很多“小伙伴”都用过ApplicationContextAware接口，主要是通过实现侦听机制得到一个回调方法，从而得到IoC容器的上下文，即ApplicationContext。在这个Mini版本中只是做了一个顶层设计，告诉大家这样一种现象，并没有做具体实现。这不是本书的重点，感兴趣的“小伙伴”可以自行尝试。

```java
package com.tom.spring.formework.context;

/**
 * 通过解耦方式获得IoC容器的顶层设计
 * 后面将通过一个监听器去扫描所有的类，只要实现了此接口，
 * 将自动调用setApplicationContext()方法，从而将IoC容器注入目标类中
 */
public interface GPApplicationContextAware {

    void setApplicationContext(GPApplicationContext applicationContext);
		
}
```