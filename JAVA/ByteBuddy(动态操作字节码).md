# ByteBuddy

## 背景

  JAVA为什么要动态操作字节码？除Byte Buddy说的因为JAVA有严格的类型校验，在开发接口交互的过程中限制了类型编译，其实还有其他很多方面的应用。比如说通用抽象，将脚本描述动态生成JAVA代码，这种流程泛华动态生成代码的应用场景十分普遍。
  在JAVA字节码操作有几种常规方式，在技术选型时，我们主要考虑如下几点：

- 适用范围
- 受欢迎程度
- 开发难度
- 维护成本

## 增强选型

- java.lang.reflect.Proxy：[https://docs.oracle.com/javas...](https://link.segmentfault.com/?enc=BjQsdYzX54FjZcpGJPQ9pQ%3D%3D.gkiBjUTQBgC1alTzOvhTh79kpnDOG0rsnZsjGz2%2BYlNusiM2zryY89F4cEPoTDgDQ806aAJEm2nqp6ITE8F6C75Pivsl5rKd0id3ZPksaQo%3D)
- javassist: [https://github.com/jboss-java...](https://link.segmentfault.com/?enc=8RUj1aMXeScn4BE3uS00Pg%3D%3D.PV2Z75SKclMtpk5ieiBBFUYqnX7BrZqGYHL4F97ZxYS%2FQ%2FNvzSLp8i9ZrM9KZl4k)
- byte buddy：[https://bytebuddy.net/#/tutorial](https://link.segmentfault.com/?enc=QIur8avpaMJeaJeTnzwICQ%3D%3D.zV1RPxCC5VBKec3gUfLeisNhcmTT7zXmMUbzVxF6%2FhO%2BHR9T7gwnI2mL958WBhHk)
- cglib：[https://github.com/cglib/cglib](https://link.segmentfault.com/?enc=NdrGPSgPS0%2BD60Mbghc%2FUQ%3D%3D.4zAv03VRdp%2FpT4Usi3xnN%2FFeOFOZKNCTuXtkorEtIJI%3D)
- java agent：[https://docs.oracle.com/javas...](https://link.segmentfault.com/?enc=9lL4l%2BQocfFfrJVBsPH2qw%3D%3D.ijMVTtrXF%2FI4hOurfpenrxBw3tv%2FGTy2wwkDz7nY4LAEsS64Ziy4ZUq6bh1y0CQlY8iOY4WDBqcskh9XVJmfRWuBPrkPPEH9%2FqDz%2FZGTY8qqcxJTd3J6qHkrmgOPrMtl)
- groovy：

# 3、Byte Buddy

## 3.1、三种增强方式

### 3.1.1、subclass（创建）

  通过继承已有的类，动态创建一个新类。
  subclass可以自定义属性、方法，也可以重写现有方法。
  subclass的一个好处是，类是新建，运行时加载不存在类冲突的问题；缺点是，对已加载的类不能增强，因为编译时没有任何类会依赖新增类。

```java
/**
 * 创建一个空类 
 * 
 */@Runner
public class SimpleCreateRunner implements Runnable {
 static String newClassName = "net.bytepuppy.subclass.HelloWorld";
 @SneakyThrows
 @Override public void run() {
 // DynamicType.Unloaded，顾名思义，创建了字节码，但未加载到虚拟机
 DynamicType.Unloaded<?> dynamicType = new ByteBuddy()
 // 继承Object.class
 .subclass(Object.class)
 // 指定固定的名字
 .name(newClassName)
 // 创建字节码
 .make();
 // 将字节码保存到指定文件
 dynamicType.saveIn(Consts.newFile(Consts.CLASS_OUTPUT_BASE_DIR));
 System.out.println("save class: " + Consts.CLASS_OUTPUT_BASE_DIR + newClassName);
 }
}
```

### 3.1.2、redefine（重写）

  重写顾明思议就是可以对一个现有类的属性、方法进行增、删、改。
  重写的前提是redefine后的类名不变，如果重命名redefine后的类，其实跟subclass效果相当。
  属性、方法被redefine后，原定义（属性、方法）会丢失，好像类被重写了一样，这也是我将redefine翻译成重写的原因。
  JVM runtime redefine一个类，不能被加载到JVM中，因为会报错：java.lang.IllegalStateException: Class already loaded: class xxx
  JVM runtime类替换的的方法之一，是JVM热加载。byte buddy通过ByteBuddyAgent.install() + ClassReloadingStrategy.fromInstalledAgent()封装了简洁的热加载调用。
  但遗憾的是，JVM 热加载不允许增减原class的schema（比如增减属性、方法），因此使用场景非常受限。修改Schema后热加载报错：UnsupportedOperationException: class redefinition failed: attempted to change the schema (add/remove fields)

```java
/**
 * 重写一个类，并在类加载前替换类，然后再实例化
 */
public class RedefineMain2 {
    public static void main(String[] args) throws Exception {
        DynamicType.Unloaded unloaded = createWithoutTriggerClassLoad();

        unloaded.saveIn(Consts.newFile(Consts.CLASS_OUTPUT_BASE_DIR));
        Object demoService = unloaded.load(ClassLoader.getSystemClassLoader(), ClassLoadingStrategy.Default.WRAPPER)
                .getLoaded().newInstance();
        Object o = demoService.getClass()
                .getMethod("report", String.class, int.class)
                .invoke(demoService, "reallx", 12);
        System.out.println(
                        o.toString());
        System.out.println(demoService.getClass().getDeclaredField("qux"));
    }

    private static DynamicType.Unloaded createWithoutTriggerClassLoad() {
        TypePool typePool = TypePool.Default.ofSystemLoader();
        DynamicType.Unloaded unloaded = new ByteBuddy()
                // try rebase
                .redefine(typePool.describe("net.bytepuppy.redefine.delegate.DemoService").resolve(),
                        ClassFileLocator.ForClassLoader.ofSystemLoader())
                // 如果用ClassLoadingStrategy.Default.WRAPPER，那必须为新类指定一个名字，否则在相同ClassLoader中名字冲突
                // ClassLoadingStrategy.Default.CHILD_FIRST，name定义可以省略
                .name("WhatEver")
                .defineField("qux", String.class)
                .method(ElementMatchers.named("report"))
                .intercept(FixedValue.value("Hello World!"))
                .make();

        return unloaded;
    }
}
```

### 3.1.3、rebase（增强）

  rebase功能与redefine相当，也可以已有类的方法、属性自定义增删改。
  rebase与redefine的区别，redefine后的原属性、原方法丢失；rebase后的原属性、原方法被拷贝 + 重命名保留在class内。
  rebase可以实现一些类似java.lang.reflect.Proxy的代理功能。但rebase与redefine一样，热加载类的问题依然存在。见：[https://github.com/raphw/byte-buddy/issues/104](https://link.segmentfault.com/?enc=KBAvdRk7AMZHyCPW4Qjbrw%3D%3D.DJ1%2FU62jd%2FH7XrUvEF4rIcQ%2FXRSJYi5NWG4W9CbIMK3vbDLFegQrPYPgXi%2Bd2Sd4)

```java
// 将redefine示例中的语句，new ByteBuddy().redefine()替换为new ByteBuddy().rebase()即可。
```

## 3.2、加载创建类

  类加载器参考：[https://blog.csdn.net/briblue/article/details/54973413](https://link.segmentfault.com/?enc=IQ9cqcewyHC7a6mG5nz9%2BQ%3D%3D.HvSk%2BPcPPkVS8Y8zGbQLI3s0VnUENycBeD3NW%2BIL4IOwjG%2B4FoE%2FiEOCfmPFNuWHziKqIjUrIS8GUcfCQs9ekQ%3D%3D)。自定义类加载器，一般重写findClass即可，loadClass不重写。
  byte buddy增强后创建的类，如果类名是新的，都可以通过ClassLoader加载。
  鉴于ClassLoader的双亲委派模式：AppClassLoader -> ExtClassLoader -> BootstrapClassLoader，新创建的类可以直接使用AppClassLoader来加载，新类在整个JVM中都是可见的。

  byte buddy封装了几个常用的ClassLoader相关调用：

- **ClassLoadingStrategy.BOOTSTRAP_LOADER**: 在Byte Buddy中代表BootstrapClassLoader，但赋值为Null，不能直接使用。ClassLoadingStrategy.BOOTSTRAP_LOADER的作用是用于构建ByteArrayClassLoader。（BootstrapClassLoader是用C++编写，Java中没有直接类可以引用）
- **ByteArrayClassLoader**：byte buddy自定义类加载器，继承自ClassLoader，未重写loadClass方法，符合双亲委派模式。即用ByteArrayClassLoader加载的类，在JVM中全局可见。ChildFirst
- **ByteArrayClassLoader.ChildFirst**: ChildFirst继承了ByteArrayClassLoader，但是重写了loadClass方法，破坏了双亲委派模式。

  ClassLoader与ByteArrayClassLoader.ChildFirst代码区别对比如下：

- ClassLoader：

  ```java
  protected Class<?> loadClass(String name, boolean resolve)
              throws ClassNotFoundException
      {
          synchronized (getClassLoadingLock(name)) {
              // First, check if the class has already been loaded
  // 在本地loader已加载内容中查找类
              Class<?> c = findLoadedClass(name);
              if (c == null) {
                  long t0 = System.nanoTime();
                  try {
  // 找不到，尝试从父加载器中找。找到了就返回。
                      if (parent != null) {
                          c = parent.loadClass(name, false);
                      } else {
                          c = findBootstrapClassOrNull(name);
                      }
                  } catch (ClassNotFoundException e) {
                      // ClassNotFoundException thrown if class not found
                      // from the non-null parent class loader
                  }
  // 父加载器也找不到此类
                  if (c == null) {
                      // If still not found, then invoke findClass in order
                      // to find the class.
                      long t1 = System.nanoTime();
  // 自定义Loader重写此方法，自定义findClass逻辑。
  // 如果没有自定义实现，原方法抛异常：ClassNotFoundException
                      c = findClass(name);
                      // this is the defining class loader; record the stats
                      sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                     sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                      sun.misc.PerfCounter.getFindClasses().increment();
                  }
              }
              if (resolve) {
                  resolveClass(c);
              }
              return c;
          }
      }
  ```

- ByteArrayClassLoader.ChildFirst

  ```java
  protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
      synchronized (SYNCHRONIZATION_STRATEGY.initialize().getClassLoadingLock(this, name)) {
  // 在本地loader已加载内容中查找类
          Class<?> type = findLoadedClass(name);
          if (type != null) {
              return type;
          }
  // 找不到，跳过父类尝试，直接从本地loader的findClass中找
  // 这里有一个问题：如果用这个ChildFirstClassLoader加载的类，不会尝试父类查找。换句话讲，同名类可以被父loader和子loader同时加载，但class却不相等
          try {
              type = findClass(name);
              if (resolve) {
                  resolveClass(type);
              }
              return type;
          } catch (ClassNotFoundException exception) {
              // If an unknown class is loaded, this implementation causes the findClass method of this instance
              // to be triggered twice. This is however of minor importance because this would result in a
              // ClassNotFoundException what does not alter the outcome.
  // 本地loader找不到，在尝试从父loader中找
              return super.loadClass(name, resolve);
          }
      }
  }
  ```

    结合Byte Buddy使用情况：

- ClassLoadingStrategy.Default.WRAPPER：构建一个ByteArrayClassLoader，符合双亲委派规则。

- ClassLoadingStrategy.Default.CHILD_FIRST：构建一个ByteArrayClassLoader.ChildFirst，rebase、redefine的类，可以被加载，但在AppClassLoader中不认识同名类。

## 3.3、替换加载类

  **byte buddy增强后创建的类，如果想类名不变，就不可以通过ClassLoader加载了，因为JVM拒绝重复加载相同类：java.lang.IllegalStateException: Class already loaded。**
  替换已加载的类有两种方式：

### 3.3.1、热加载

参考：[https://developer.aliyun.com/article/65023](https://link.segmentfault.com/?enc=Gs5GRK71Mzz9mVkRiPIHcg%3D%3D.laLDH1cWRAdSY2B5PiVyXRLIUXPg%2FDpaF9hLofMReHdCGRqyPucu%2BaVxkVXF5GP1)

  热加载需要借助java agent的Instrumentation.redefineClasses。
Byte Buddy提供了便捷的热加载实现：ByteBuddyAgent.install()配合ClassReloadingStrategy.fromInstalledAgent()。
  热加载有极大的使用限制：不允许修改已有的属性、方法，如果class schema变化，JVM拒绝重载修改类。这意味着我们很难用热加载的方式编写切面逻辑。

```java
// 加载类时使用热加载ClassLoader策略。
DynamicType.Unloaded<?>.load(ClassLoader.getSystemClassLoader(), ClassReloadingStrategy.fromInstalledAgent());
```

### 3.3.2、懒加载

  JVM有一个特性，启动后，类要到使用的时候才加载。**这意味着，如果我们不直接引用类，触发类加载，那么在此之前我们都可以自由替换增强类。**
  增强类的生成发生在构建时，因为构建是在另一个JVM中完成的，所以不影响运行时类加载。
为了实现懒加载，byte buddy构建了几个有用的类：

> TypeDescription
>   类型描述对象。**用此对象包装的类，不会触发类加载**，但可以获得包装类的各种信息。
> TypePool
>   TypeDescription类型池。用TypePool.Default.ofSystemLoader()语句可以获得但前ClassLoader下所有的类描述，但不会触发类加载。
>   typePool.describe("{your_class_string_name}").resolve()可以获得对应类的TypeDescription。

```java
//TypePool, TypeDescription用法，参考3.1 redefine示例
```

## 3.4、java agent

  参考：[https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html](https://link.segmentfault.com/?enc=%2BzCtO%2BiSDYpqZqw3%2BixofQ%3D%3D.YuPrn3o7NdotBnZEOjy6cTIka2%2F22r4av%2BJ23RT%2B6tmZR0CPPTjwoSKLOj5iflqWfOmqzl6TtE8uTirdIPcuGz3PK5c6NEmn5%2Ffb2frHWeR7PyLMPMJ6IMpyMRMicZgV)

  在项目规模庞大的时候，上面替换加载类的方式都限制太多，不适用。

  java agent可以在类加载前，修改（transform）类，避免热加载，同时还能对业务逻辑进行增强。

  替换增强类，主要是通过java.lang.instrument.Instrumentation，Java Agent之所以能有用，主要还是Java Agent的两个入口提供了java.lang.instrument.Instrumentation的访问入口：

- premain
    JVM初始化后被调用，方法为： public static void premain(String arguments, Instrumentation instrumentation) {...}
      -javaagent:jarpath[=options]方式添加JVM启动参数，permain被调用，agentmain即便实现也不会调用。
- agentmain
    JVM启动后被调用，方法为：public static void agentmain(String arguments, Instrumentation instrumentation) {...}

  agentmain被调用有三个条件：
  1）agent jar的manifest必须显式定义属性Agent-Class；
  2）Agent-Class指定类，必须定义agentmain方法；
  3）agent jar在JVM启动的classpath路径中。

  例如：byte buddy定义了net.bytebuddy.agent.Installer就是用agentmain的方式获取Instrumentation类。我们解压byte-buddy-agent-1.10.23-SNAPSHOT.jar，cat META-INF/MANIFEST.MF，可以找到Agent-Class定义：

```yaml
../META-INF$ cat MANIFEST.MF 
Manifest-Version: 1.0
Bundle-Description: The Byte Buddy agent offers convenience for attach
 ing an agent to the local or a remote VM.
Bundle-License: http://www.apache.org/licenses/LICENSE-2.0.txt
Bundle-SymbolicName: net.bytebuddy.byte-buddy-agent
Built-By: liuh
**Agent-Class: net.bytebuddy.agent.Installer**
Bnd-LastModified: 1616574091622
Bundle-ManifestVersion: 2
Can-Redefine-Classes: true
Import-Package: com.sun.tools.attach;resolution:=optional,com.ibm.tool
 s.attach;resolution:=optional
Require-Capability: osgi.ee;filter:="(&(osgi.ee=JavaSE)(version=1.5))"
Can-Set-Native-Method-Prefix: true
Tool: Bnd-3.5.0.201709291849
Export-Package: net.bytebuddy.agent;version="1.10.23"
Premain-Class: net.bytebuddy.agent.Installer
Bundle-Name: Byte Buddy agent
Bundle-Version: 1.10.23.SNAPSHOT
Multi-Release: true
Can-Retransform-Classes: true
Created-By: Apache Maven Bundle Plugin
Build-Jdk: 1.8.0_144
```

# 4、byte buddy增强类的两种方式

  很多时候，我们修改一个类，可能不会改变原方法的逻辑，而是只会在方法调用前后做一个拦截，对业务无关的日志、安全、监控等需求织入一个切面，而不会影响原代码逻辑变更。Byte Buddy提供如下两种方式增强一个类。
  这里再强调一下，这里的增强都涉及到运行时代码重载，但代码热加载不允许增、删原有类的属性、方法，使用场景有很大限制。使用时，字节码的生产，尽量安排在加载前，这样可以最大限度的自由编辑已有类库。

## 4.1、net.bytebuddy.implementation.MethodDelegation

参考：[https://bytebuddy.net/#/tutorial](https://link.segmentfault.com/?enc=3CSulgeaMs9oO3%2BQ1LAY8w%3D%3D.zRD%2FEDp7b1P24E5N8p3yEGbVk44A6WlUYcQU%2B3WZVB8x2u7OUkqWeMhWpcsjf8E7) #Delegating a method call#
  MethodDelegation作用是将一个方法调用，重定向（代理）到另一个方法调用上，无论这个方法是否静态，也无论这些方法是否属于同一个类。
  Apache SkyWalking字节码增强，就是用了MethodDelegate的方式。下面代码是SkyWalking源码的一个简洁版。
  我们首先抽象拦截切面的变化点，并实现拦截操作的公共流程。

```java
/**
 * 代理拦截接口，抽象方法调用前后的两个切面
 */
public interface InstMethodAroundInterceptor {

    /**
     * 拦截点前
     * @param inst: 被增强类实例
     * @param interceptPoint：被增强方法
     * @param allArguments：被增强方法入参
     * @param argumentsTypes：被增强方法入参类型
     * @param result：result 包装类
     */
    void beforeMethod(Object inst, Method interceptPoint,
                      Object[] allArguments, Class<?>[] argumentsTypes,
                      ResultWrapper result);

    Object afterMethod(Object inst, Method interceptPoint,
                       Object[] allArguments, Class<?>[] argumentsTypes,
                       Object ret);

    void handleMethodException(Object inst, Method method, Object[] allArguments,
                               Class<?>[] argumentsTypes, Throwable t);
}
/**
 * 统一代理模板
 */
public class DelegateTemplate {

    private InstMethodAroundInterceptor interceptor;

    public DelegateTemplate(InstMethodAroundInterceptor interceptor) {
        this.interceptor = interceptor;
    }

    /**
     * 拦截增强主方法
     *
     * @param inst: 被拦截对象本身
     * @param allArguments：被代理方法原参数
     * @param zuper：被代理方法的包装对象，zuper.call()调用原方法
     * @param method：原方法对象
     * @return
     */
    public Object interceptor(@This Object inst, @AllArguments Object[] allArguments,
                              @SuperCall Callable<?> zuper, @Origin Method method) {
        ResultWrapper rw = new ResultWrapper();
        if (this.interceptor != null) {
            try {
                // 调用前拦截处理
                this.interceptor.beforeMethod(inst, method,
                        allArguments, method.getParameterTypes(), rw);
            } catch (Throwable t) {
                t.printStackTrace();
            }
        }

        if (!rw.isContinue()) {
            return rw.getResult();
        }

        Object result = null;
        try {
            // 被代理方法调用
            result = zuper.call();

            if (this.interceptor != null) {
                try {
                    // 调用后拦截处理
                    result = this.interceptor.afterMethod(inst, method,
                            allArguments, method.getParameterTypes(), result);
                } catch (Throwable t) {
                    t.printStackTrace();
                }
            }
        } catch (Exception e) {
            if (this.interceptor != null) {
                try {
                    // 调用异常拦截处理
                    this.interceptor.handleMethodException(inst, method,
                            allArguments, method.getParameterTypes(), e);
                } catch (Throwable t) {
                    t.printStackTrace();
                }
            }
        }

        return result;
    }
    
    @Data
    public class ResultWrapper {
        private boolean isContinue;
        private Object result;
    }
}
```

  接下来我们开始利用拦截切口 + 拦截模板来增强一个自定义的方法。

```java
/**
 * 被增强类。模拟一个业务类，有report和compute两个方法。
 */
public class DemoService {

    public String report(String name, int value) {
        return String.format("name: %s, value: %s", name, value);
    }

    public void compute(List<Integer> values) {
        System.out.println("compute result:" + values.stream().mapToInt(v -> v.intValue()).sum());
    }
}
/**
 * DemoService增强切面，实现切面接口InstMethodAroundInterceptor
 */
public class DemoServiceInterceptor implements InstMethodAroundInterceptor {
    @Override
    public void beforeMethod(Object inst, Method interceptPoint, Object[] allArguments,
                             Class<?>[] argumentsTypes, ResultWrapper result) {
        System.out.println("DemoService Interceptor in ...");
    }

    @Override
    public Object afterMethod(Object inst, Method interceptPoint, Object[] allArguments,
                              Class<?>[] argumentsTypes, Object ret) {
        System.out.println("DemoService Interceptor out ...");
        return ret;
    }

    @Override
    public void handleMethodException(Object inst, Method method, Object[] allArguments,
                                      Class<?>[] argumentsTypes, Throwable t) {
        System.out.println("DemoService Interceptor error handle ...");
    }
}
/**
 * 模拟java agent增强类
 */
public class JavaAgentMain {

    public static void premain(String agentArgs, Instrumentation instrumentation) {
        new AgentBuilder.Default()
                // 增强类通过类名匹配
                .type(ElementMatchers.named("net.bytepuppy.redefine.delegate.DemoService"))
                // 自定义Transformer
                .transform(new AgentBuilder.Transformer() {
                    @Override
                    public DynamicType.Builder<?> transform(DynamicType.Builder<?> builder,
                                                            TypeDescription typeDescription,
                                                            ClassLoader classLoader, JavaModule module) {
                        // 实例化自己的拦截实例DemoServiceInterceptor
                        // 将拦截实例传入拦截模板，并完成实例化
                        // 将DemoService实例的report方法，拦截代理到DelegateTemplate的interceptor
                        return builder.method(ElementMatchers.named("report"))
                                .intercept(MethodDelegation.to(new DelegateTemplate(new DemoServiceInterceptor())));
                    }
                })
                // 增强
                .installOn(instrumentation);
    }
}
```

  上面的实例，用JAVA Agent在加载前增强类，而不是在运行是用热加载的增强类，是有原因的：

> redefine和rebase都是对已有类进行修改。
> 在正常JVM启动（这里是main函数，也就是运行时）中，DemoService已被加载
> 被redefine的DemoService，不能再被加载，会报错ClassAlreadyExist.
> 另外，JVM热加载时，禁止修改已有类的schema（方法、属性，但可以修改逻辑片段）
>
> - redefine：重新定义一个类，被增强的方法、属性，会丢失原方法、属性
> - rebase：与redefine相似，但被增强的方法、属性不会丢失，而是会已拷贝 + 重命名的方式被保留
>
> 因此：
>
> - redefine：MethodDelegate无效。因为redefine会丢失原方法，@SuperCall调用父类方法找不着了。
> - rebase：MethodDelegate无效。因为rebase拷贝、重命名原有方法，会新增方法，破坏了热加载规则，代理失效。参考{@link RedefineMain2}
>
> 问题参考： [https://github.com/raphw/byte...](https://link.segmentfault.com/?enc=slacorW40WOJ3rseQ2ABtA%3D%3D.p3KbrZYUSP1HLuXirI1P8DV0kcjXxuZYeHotCkTQpsvoO0OytnC1%2BnyI5UyHWxRJ)

## 4.2、net.bytebuddy.asm.Advice

参考：[https://blog.csdn.net/wanxiao...](https://link.segmentfault.com/?enc=HVLqQd6sHBo7u5oAK0os1Q%3D%3D.yCnOnoUmPjd9eGRaniSPPbmuPnLWXBAoT%2BTMEAgW5VNlayNAbm1wk9PDOdZ34vdDiMVxybVXSfC6P3VIsl093g%3D%3D)
 [https://medium.com/@lnishada/...](https://link.segmentfault.com/?enc=oNT%2FhXP%2B895keOcczCLurw%3D%3D.O7WLuuPOjWedNoSaSWCACo6bISGLfLmCjNv0QVDctgm36cb4JRtsJ%2BRo%2B0AD0tBRaajXBv8A%2BqUFEw3WDijya3lxSO1uVgjuvwwGkZLwUJiqhmI96EsfRl%2FjzygCMP%2F%2B)
  在MethodDelegation中有个问题，一个类被反复增强，会导致新的字节码实例方法调用堆栈变化。例如：[byte buddy issue 829](https://link.segmentfault.com/?enc=WOYQwoUoYFu1oTQ812bHPw%3D%3D.95Pg1uVZnG3QyXw%2BedKAUviBeHzOT7kN80zXvnCaowY4gIp5V1AZIC0znM%2F4kEX%2F)。
  Byte Buddy在其官方文档只字未提。试用了一下，稍微复杂的逻辑增强Advise就失效，原因未名，而且Advise会导致断点失效，对于复杂业务开发并不友好。
  简单示例如下：

```java
// 模拟一个业务逻辑类，及方法
public class ComputeService {

    public String compute(String name, List<Integer> values) {
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) { }

        return String.format("compute name: %s, compute result: %s",
                name, values.stream().mapToInt(v -> v.intValue()).sum());
    }
}
// 抽象Advise 公共模板
public class AdviceTemplate {

// 引入LogInterceptor，导致@Advice.OnMethodEnter失效
//    private static LogInterceptor logInterceptor;
//    static {
//        logInterceptor = new LogInterceptor();
//    }

    /**
     * @Advice.OnMethodEnter 必须是静态方法
     *
     * @param thisObject
     * @param origin
     * @param detaildOrigin
     * @param args
     * @return
     */
    @Advice.OnMethodEnter(suppress = Throwable.class)
    public static long beforeMethod(@Advice.This Object thisObject,
                                    @Advice.Origin String origin,
                                    @Advice.Origin("#t #m") String detaildOrigin,
                                    @Advice.AllArguments Object[] args) {

        StringBuilder logBuilder = new StringBuilder();

        if(args != null) {
            for(int i =0 ; i < args.length ; i++) {
                logBuilder.append("Argument- " + i + " is: " + args[i] + ", ");
            }
            logBuilder.delete(logBuilder.length() - 2, logBuilder.length());
        }

//        LogInterceptor.log(logBuilder.toString());
// 调用内部静态log方法，导致@Advice.OnMethodEnter失效
//        log(logBuilder.toString());
        long startTime = System.currentTimeMillis();
        System.out.println("start time: " + startTime);
        return startTime;
    }

    /**
     * @Advice.OnMethodExit 必须是静态方法
     *
     * @param time
     * @param ret
     */
    @Advice.OnMethodExit(suppress = Throwable.class, onThrowable = Throwable.class)
    public static void afterMethod(@Advice.Enter long time, @Advice.Return Object ret) {
        long endTime = System.currentTimeMillis();
        System.out.println("end time: " + endTime);
        System.out.println("Method Execution Cost Time: " + (endTime - time) + " mills");
    }

    private static void log(String log) {
        System.out.println("advised log:" + log);
    }
}
public class AdviceRedefineMain {
    public static void main(String[] args) throws Exception {
        ByteBuddyAgent.install();
        DynamicType.Unloaded dtu = new ByteBuddy()
                .redefine(ComputeService.class)
                // advise 区别于delegation的核心语句
                .visit(Advice.to(AdviceTemplate.class)
                        .on(ElementMatchers.named("compute")))
                .make();

        Class<?> clazz = dtu.load(ClassLoadingStrategy.BOOTSTRAP_LOADER,
                ClassLoadingStrategy.Default.WRAPPER)
                .getLoaded();

        Object service = clazz.newInstance();

        Object result = clazz.getMethod("compute", String.class, List.class)
                .invoke(service, "AdviceDemo", Lists.newArrayList(1, 2, 4));
        System.out.println(result);

//        ((ComputeService) service).compute("AdviceDemo", Lists.newArrayList(1, 2, 4));

    }
}
```

# 5、Byte Buddy使用限制

- JVM类热加载，不能修改类的Schema，否则报错UnsupportedOperationException
- 运行时，尽量避免使用Byte Buddy对某个类的refine和rebase，因为这两个操作都涉及到操作类的Schema。但如果是基于某个类创建新类，则没有此限制。
- Byte Buddy配合JAVA Agent最优解，将类修改放到JVM类真实加载前。（permain，顾名思义在JVM Main方法执行前执行，此时所有类还未加载）