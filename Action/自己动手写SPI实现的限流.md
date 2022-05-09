## 前言

之前我们聊了一下[聊聊如何实现一个带有拦截器功能的SPI](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI1MTY1Njk4NQ%3D%3D%26mid%3D2247499884%26idx%3D1%26sn%3D0c945f47737e73920fbab179ebe6cd0d%26chksm%3De9ed2e1ade9aa70c635d6fb53bc38bc77c2c9342d9a9d567f3d99a40fa49ffdef12d4b4e6c39%26token%3D1779395205%26lang%3Dzh_CN%23rd)。当时我们实现的核心思路是利用了责任链+动态代理。今天我们再聊下通过动态代理如何去整合sentinel实现熔断限流

## 前置知识

### alibaba sentinel简介

Sentinel 是面向分布式服务架构的流量控制组件，主要以流量为切入点，从限流、流量整形、熔断降级、系统负载保护、热点防护等多个维度来帮助开发者保障微服务的稳定性。

### sentinel工作流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc98a255949643ed94798c04c328afd4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGlueWLmnoHlrqLkuYvot68=,size_20,color_FFFFFF,t_70,g_se,x_16)

### sentinel关键词

资源 + 规则

### sentinel实现模板套路

```java
Entry entry = null;
// 务必保证 finally 会被执行
try {
  // 资源名可使用任意有业务语义的字符串，注意数目不能太多（超过 1K），超出几千请作为参数传入而不要直接作为资源名
  // EntryType 代表流量类型（inbound/outbound），其中系统规则只对 IN 类型的埋点生效
  entry = SphU.entry("自定义资源名");
  // 被保护的业务逻辑
  // do something...
} catch (BlockException ex) {
  // 资源访问阻止，被限流或被降级
  // 进行相应的处理操作
} catch (Exception ex) {
  // 若需要配置降级规则，需要通过这种方式记录业务异常
  Tracer.traceEntry(ex, entry);
} finally {
  // 务必保证 exit，务必保证每个 entry 与 exit 配对
  if (entry != null) {
    entry.exit();
  }
}
```

### sentinel wiki

[https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E9%A1%B5](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falibaba%2FSentinel%2Fwiki%2F%E4%B8%BB%E9%A1%B5)

## 实现思路

整体实现思路：动态代理 + sentinel实现套路模板

### 核心代码

#### 动态代理部分

> 1、定义动态代理接口

```java
public interface CircuitBreakerProxy {

    Object getProxy(Object target);

    Object getProxy(Object target,@Nullable ClassLoader classLoader);
}
```

> 2、定义JDK或者cglib具体动态实现

以jdk动态代理为例子

```java
public class CircuitBreakerJdkProxy implements CircuitBreakerProxy, InvocationHandler {

    private Object target;

    @Override
    public Object getProxy(Object target) {
        this.target = target;
        return getProxy(target,Thread.currentThread().getContextClassLoader());
    }

    @Override
    public Object getProxy(Object target, ClassLoader classLoader) {
        this.target = target;
        return Proxy.newProxyInstance(classLoader,target.getClass().getInterfaces(),this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        CircuitBreakerInvocation invocation = new CircuitBreakerInvocation(target,method,args);
        try {
            return new CircuitBreakerInvoker().proceed(invocation);
            //用InvocationTargetException包裹是java.lang.reflect.UndeclaredThrowableException问题
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        }

    }
}
```

> 3、动态代理具体调用

```java
public class CircuitBreakerProxyFactory implements ProxyFactory{
    @Override
    public Object createProxy(Object target) {
        if(target.getClass().isInterface() || Proxy.isProxyClass(target.getClass())){
            return new CircuitBreakerJdkProxy().getProxy(target);
        }
        return new CircuitBreakerCglibProxy().getProxy(target);
    }
}
```

**ps：** 上面的动态代理实现思路是参考spring aop动态代理实现

具体参考类如下

```java
org.springframework.aop.framework.AopProxy
org.springframework.aop.framework.DefaultAopProxyFactory
```

#### sentinel实现部分

```java
public class CircuitBreakerInvoker {

    public Object proceed(CircuitBreakerInvocation circuitBreakerInvocation) throws Throwable {

       Method method = circuitBreakerInvocation.getMethod();

        if ("equals".equals(method.getName())) {
            try {
                Object otherHandler = circuitBreakerInvocation.getArgs().length > 0 && circuitBreakerInvocation.getArgs()[0] != null
                        ? Proxy.getInvocationHandler(circuitBreakerInvocation.getArgs()[0]) : null;
                return equals(otherHandler);
            } catch (IllegalArgumentException e) {
                return false;
            }
        } else if ("hashCode".equals(method.getName())) {
            return hashCode();
        } else if ("toString".equals(method.getName())) {
            return toString();
        }


        Object result = null;

        String contextName = "spi_circuit_breaker：";

        String className = ClassUtils.getClassName(circuitBreakerInvocation.getTarget());
        String resourceName = contextName + className + "." + method.getName();


        Entry entry = null;
        try {
            ContextUtil.enter(contextName);
            entry = SphU.entry(resourceName, EntryType.OUT, 1, circuitBreakerInvocation.getArgs());
            result = circuitBreakerInvocation.proceed();
        } catch (Throwable ex) {
            return doFallBack(ex, entry, circuitBreakerInvocation);
        } finally {
            if (entry != null) {
                entry.exit(1, circuitBreakerInvocation.getArgs());
            }
            ContextUtil.exit();
        }

        return result;
    }
    }
```

**ps：** 如果心细的朋友，就会发现这个逻辑就是sentinel原生的实现套路逻辑

## 示例演示

### 示例准备

> 1、定义接口实现类并加上熔断注解

```java
@CircuitBreakerActivate(spiKey = "sqlserver",fallbackFactory = SqlServerDialectFallBackFactory.class)
public class SqlServerDialect implements SqlDialect {
    @Override
    public String dialect() {
        return "sqlserver";
    }

}
```

**ps：** @CircuitBreakerActivate这个是自定义熔断注解，用过springcloud openfeign的@FeignClient注解大概就会有种熟悉感，都是在注解上配置fallbackFactory或者fallbackBack

> 2、定义接口熔断工厂

```java
@Slf4j
@Component
public class SqlServerDialectFallBackFactory implements FallbackFactory<SqlDialect> {

    @Override
    public SqlDialect create(Throwable ex) {
        return () -> {
            log.error("{}",ex);
            return "SqlServerDialect FallBackFactory";
        };
    }
}
```

**ps：** 看这个是不是也很熟悉，这不就是springcloud hystrix的熔断工厂吗

> 3、项目中配置sentinel dashbord地址

```java
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080
      filter:
        enabled: false
```

### 示例验证

> 1、浏览器访问[http://localhost:8082/test/ciruitbreak](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flocalhost%3A8082%2Ftest%2Fciruitbreak)

![img](https://img-blog.csdnimg.cn/img_convert/b79018edaf8c26250fc723458bc6e191.png)

此时访问正常，看下sentinel-dashbord

![img](https://img-blog.csdnimg.cn/img_convert/d0417069e4c4860410d0fcde9dc627e5.png)

> 2、配置限流规则

![img](https://img-blog.csdnimg.cn/img_convert/37328e6f543ecbe59502953dcbcbfb40.png)

> 3、再次快速访问

![img](https://img-blog.csdnimg.cn/img_convert/f4d725c093bb4ac5c9d5ad8bc59c2c87.png)

由图可以看出已经触发熔断

本示例项目，如果不配置fallback或者fallbackFactory，当触发限流时，它也会有个默认fallback

把示例注解的fallbackFactory去掉，如下

```java
@CircuitBreakerActivate(spiKey = "sqlserver")
public class SqlServerDialect implements SqlDialect {
    @Override
    public String dialect() {
        return "sqlserver";
    }

}
```

再重复上面的访问流程，当触发限流时，会提示如下

![img](https://img-blog.csdnimg.cn/img_convert/e8ed762ded2efbb0299127fa092c2485.png)

## 总结

自定义spi的实现系列接近尾声了，其实这个小demo并没有多少原创的东西，大都从dubbo、shenyu、mybatis、spring、sentinel源码中抽出一些比较好玩的东西，东拼西凑出来的一个demo。

平时我们大部分时间还是在做业务开发，可能有些人会觉得天天crud，感觉没啥成长空间，但只要稍微留意一下我们平时经常使用的框架，说不定就会发现一些不一样的风景