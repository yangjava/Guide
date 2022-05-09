# Dubbo源码

Dubbo是什么?

看官网简介

https://dubbo.apache.org/zh/

## 简介

Apache Dubbo 是一款高性能、轻量级的开源 Java 服务框架

Apache Dubbo |ˈdʌbəʊ| 提供了六大核心能力：面向接口代理的高性能RPC调用，智能容错和负载均衡，服务自动注册和发现，高度可扩展能力，运行期流量调度，可视化的服务治理与运维。

dubbo的Maven配置

```
   <properties>
        <alibaba.dubbo.version>2.7.3</alibaba.dubbo.version>
    </properties>
      
      <dependencies>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-dependencies-bom</artifactId>
                <version>${alibaba.dubbo.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>${alibaba.dubbo.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo</artifactId>
                <version>${alibaba.dubbo.version}</version>
            </dependency>
        </dependencies>
```



看Dubbo源码

主要有以下模块

```
dubbo registry

dubbo cluster

dubbo config 

dubbo rpc

dubbo remoting

dubbo container

dubbo monitor

```

#### dubbo registry **注册中心模块**

官方文档的解释：基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。

这个模块就是封装了dubbo所支持的注册中心的实现。

#### **dubbo-cluster 集群模块**

官方文档的解释：将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。

#### **dubbo-common 公共逻辑模块**

官方文档的解释：包括 Util 类和通用模型。

#### **dubbo-config 配置模块**

官方文档的解释：是 Dubbo 对外的 API，用户通过 Config 使用Dubbo，隐藏 Dubbo 所有细节。

用户都是使用配置来使用dubbo，dubbo也提供了四种配置方式，包括XML配置、属性配置、API配置、注解配置，配置模块就是实现了这四种配置的功能。

#### **dubbo-rpc 远程调用模块**

官方文档的解释：抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。

#### **dubbo-remoting 远程通信模块**

官方文档的解释：相当于 Dubbo 协议的实现，如果 RPC 用 RMI协议则不需要使用此包。

#### **dubbo-container 容器模块**

官方文档的解释：是一个 Standlone 的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。

#### **dubbo-monito 监控模块**

官方文档的解释：统计服务调用次数，调用时间的，调用链跟踪的服务。

#### **dubbo-bootstrap 清理模块**

这个模块只有一个类，是作为dubbo的引导类，并且在停止期间进行清理资源。

#### **dubbo-filter 过滤器模块**

这个模块提供了内置的一些过滤器。

#### **dubbo-plugin 插件模块**

该模块提供了内置的插件。

#### **dubbo-serialization 序列化模块**

该模块中封装了各类序列化框架的支持实现。

## Dubbo的SPI机制

dubbo有大量的spi扩展实现，包括协议扩展、调用拦截扩展、路由扩展等26个扩展，并且spi机制运用到了各个模块设计中。

### JDK的SPI

SPI的全名为Service Provider Interface，面向对象的设计里面，模块之间推荐基于接口编程，而不是对实现类进行硬编码，这样做也是为了模块设计的可拔插原则。为了在模块装配的时候不在程序里指明是哪个实现，就需要一种服务发现的机制，jdk的spi就是为某个接口寻找服务实现。jdk提供了服务实现查找的工具类：java.util.ServiceLoader，它会去加载META-INF/service/目录下的配置文件。实现源码如下

```

```

### Dubbo的SPI

dubbo自己实现了一套SPI机制，改进了JDK标准的SPI机制：

1. JDK标准的SPI只能通过遍历来查找扩展点和实例化，有可能导致一次性加载所有的扩展点，如果不是所有的扩展点都被用到，就会导致资源的浪费。dubbo每个扩展点都有多种实现，例如com.alibaba.dubbo.rpc.Protocol接口有InjvmProtocol、DubboProtocol、RmiProtocol、HttpProtocol、HessianProtocol等实现，如果只是用到其中一个实现，可是加载了全部的实现，会导致资源的浪费。
2. 把配置文件中扩展实现的格式修改，例如META-INF/dubbo/com.xxx.Protocol里的com.foo.XxxProtocol格式改为了xxx = com.foo.XxxProtocol这种以键值对的形式，这样做的目的是为了让我们更容易的定位到问题，比如由于第三方库不存在，无法初始化，导致无法加载扩展名（“A”），当用户配置使用A时，dubbo就会报无法加载扩展名的错误，而不是报哪些扩展名的实现加载失败以及错误原因，这是因为原来的配置格式没有把扩展名的id记录，导致dubbo无法抛出较为精准的异常，这会加大排查问题的难度。所以改成key-value的形式来进行配置。
3. dubbo的SPI机制增加了对IOC、AOP的支持，一个扩展点可以直接通过setter注入到其他扩展点。

#### ExtensionLoader

该类是扩展加载器，这是dubbo实现SPI扩展机制等核心，几乎所有实现的逻辑都被封装在ExtensionLoader中。

属性

```
public class ExtensionLoader<T> {

    private static final Logger logger = LoggerFactory.getLogger(ExtensionLoader.class);
    // dubbo为了兼容jdk的SPI扩展机制思想而设存在的
    private static final String SERVICES_DIRECTORY = "META-INF/services/";
    // 为了给用户自定义的扩展实现配置文件存放。
    private static final String DUBBO_DIRECTORY = "META-INF/dubbo/";
    // dubbo内部提供的扩展的配置文件路径
    private static final String DUBBO_INTERNAL_DIRECTORY = DUBBO_DIRECTORY + "internal/";
```

扩展加载器集合，key为扩展接口，例如Protocol等：

```
private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<>();
```

扩展实现类集合，key为扩展实现类，value为扩展对象，例如key为Class<DubboProtocol>，value为DubboProtocol对象

```
 private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<>();
```

以下属性都是cache开头的，都是出于性能和资源的优化，才做的缓存，读取扩展配置后，会先进行缓存，等到真正需要用到某个实现时，再对该实现类的对象进行初始化，然后对该对象也进行缓存。

```
    //以下提到的扩展名就是在配置文件中的key值，类似于“dubbo”等

    //缓存的扩展名与拓展类映射，和cachedClasses的key和value对换。
    private final ConcurrentMap<Class<?>, String> cachedNames = new ConcurrentHashMap<Class<?>, String>();
    //缓存的扩展实现类集合
    private final Holder<Map<String, Class<?>>> cachedClasses = new Holder<Map<String, Class<?>>>();
    //扩展名与加有@Activate的自动激活类的映射
    private final Map<String, Activate> cachedActivates = new ConcurrentHashMap<String, Activate>();
    //缓存的扩展对象集合，key为扩展名，value为扩展对象
    //例如Protocol扩展，key为dubbo，value为DubboProcotol
    private final ConcurrentMap<String, Holder<Object>> cachedInstances = new ConcurrentHashMap<String, Holder<Object
    //缓存的自适应( Adaptive )扩展对象，例如例如AdaptiveExtensionFactory类的对象
    private final Holder<Object> cachedAdaptiveInstance = new Holder<Object>();
    //缓存的自适应扩展对象的类，例如AdaptiveExtensionFactory类
    private volatile Class<?> cachedAdaptiveClass = null;
    //缓存的默认扩展名，就是@SPI中设置的值
    private String cachedDefaultName;
    //创建cachedAdaptiveInstance异常
    private volatile Throwable createAdaptiveInstanceError;
    //拓展Wrapper实现类集合
    private Set<Class<?>> cachedWrapperClasses;
    //拓展名与加载对应拓展类发生的异常的映射
    private Map<String, IllegalStateException> exceptions = new ConcurrentHashMap<String, IllegalStateException>();
```

根据扩展点接口来获得扩展加载器。源码如下

```
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Extension type == null");
        }
        if (!type.isInterface()) {
            throw new IllegalArgumentException("Extension type (" + type + ") is not an interface!");
        }
        if (!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type (" + type +
                    ") is not an extension, because it is NOT annotated with @" + SPI.class.getSimpleName() + "!");
        }

        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        if (loader == null) {
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
```

#### 注解@SPI

在某个接口上加上@SPI注解后，表明该接口为可扩展接口。比如

```
@SPI
public interface ExtensionFactory {

    /**
     * Get extension.
     *
     * @param type object type.
     * @param name object name.
     * @return object instance.
     */
    <T> T getExtension(Class<T> type, String name);

}
```

例如扩展ExtensionFactory

因为在ExtensionFactory上有@SPI("dubbo"),就会去META-INF/dubbo/internal下面的文件

org.apache.dubbo.common.extension.ExtensionFactory文件中找该key对应的value

该接口是扩展工厂接口类，它本身也是一个扩展接口，有SPI的注解。该工厂接口提供的就是获取实现类的实例，它也有两种扩展实现，分别是SpiExtensionFactory和SpringExtensionFactory代表着两种不同方式去获取实例。而具体选择哪种方式去获取实现类的实例，则在适配器AdaptiveExtensionFactory中制定了规则。

#### 注解@Adaptive

该注解为了保证dubbo在内部调用具体实现的时候不是硬编码来指定引用哪个实现，也就是为了适配一个接口的多种实现，这样做符合模块接口设计的可插拔原则

在实现类上面加上@Adaptive注解，表明该实现类是该接口的适配器。

举个例子dubbo中的ExtensionFactory接口就有一个实现类AdaptiveExtensionFactory，加了@Adaptive注解，AdaptiveExtensionFactory就不提供具体业务支持，用来适配ExtensionFactory的SpiExtensionFactory和SpringExtensionFactory这两种实现。AdaptiveExtensionFactory会根据在运行时的一些状态来选择具体调用ExtensionFactory的哪个实现，具体的选择可以看下文Adaptive的代码解析。

#### getActivateExtension

```
    public List<T> getActivateExtension(URL url, String[] values, String group) {
        List<T> exts = new ArrayList<>();
        List<String> names = values == null ? new ArrayList<>(0) : Arrays.asList(values);
        if (!names.contains(REMOVE_VALUE_PREFIX + DEFAULT_KEY)) {
            getExtensionClasses();
            for (Map.Entry<String, Object> entry : cachedActivates.entrySet()) {
                String name = entry.getKey();
                Object activate = entry.getValue();

                String[] activateGroup, activateValue;

                if (activate instanceof Activate) {
                    activateGroup = ((Activate) activate).group();
                    activateValue = ((Activate) activate).value();
                } else if (activate instanceof com.alibaba.dubbo.common.extension.Activate) {
                    activateGroup = ((com.alibaba.dubbo.common.extension.Activate) activate).group();
                    activateValue = ((com.alibaba.dubbo.common.extension.Activate) activate).value();
                } else {
                    continue;
                }
                if (isMatchGroup(group, activateGroup)) {
                    T ext = getExtension(name);
                    if (!names.contains(name)
                            && !names.contains(REMOVE_VALUE_PREFIX + name)
                            && isActive(activateValue, url)) {
                        exts.add(ext);
                    }
                }
            }
            exts.sort(ActivateComparator.COMPARATOR);
        }
        List<T> usrs = new ArrayList<>();
        for (int i = 0; i < names.size(); i++) {
            String name = names.get(i);
            if (!name.startsWith(REMOVE_VALUE_PREFIX)
                    && !names.contains(REMOVE_VALUE_PREFIX + name)) {
                if (DEFAULT_KEY.equals(name)) {
                    if (!usrs.isEmpty()) {
                        exts.addAll(0, usrs);
                        usrs.clear();
                    }
                } else {
                    T ext = getExtension(name);
                    usrs.add(ext);
                }
            }
        }
        if (!usrs.isEmpty()) {
            exts.addAll(usrs);
        }
        return exts;
    }
```

##### getExtension

```
    public T getExtension(String name) {
        if (StringUtils.isEmpty(name)) {
            throw new IllegalArgumentException("Extension name == null");
        }
        if ("true".equals(name)) {
            return getDefaultExtension();
        }
        final Holder<Object> holder = getOrCreateHolder(name);
        Object instance = holder.get();
        if (instance == null) {
            synchronized (holder) {
                instance = holder.get();
                if (instance == null) {
                    instance = createExtension(name);
                    holder.set(instance);
                }
            }
        }
        return (T) instance;
    }
```

#### loadExtensionClasses

从配置文件中，加载拓展实现类数组

```
    private Map<String, Class<?>> loadExtensionClasses() {
        cacheDefaultExtensionName();

        Map<String, Class<?>> extensionClasses = new HashMap<>();
        loadDirectory(extensionClasses, DUBBO_INTERNAL_DIRECTORY, type.getName());
        loadDirectory(extensionClasses, DUBBO_INTERNAL_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        loadDirectory(extensionClasses, DUBBO_DIRECTORY, type.getName());
        loadDirectory(extensionClasses, DUBBO_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        loadDirectory(extensionClasses, SERVICES_DIRECTORY, type.getName());
        loadDirectory(extensionClasses, SERVICES_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        return extensionClasses;
    }
```

加载配置文件的顺序为

```
META-INF/dubbo/internal/
META-INF/dubbo/
META-INF/services/
```

##### loadDirectory

```
    private void loadDirectory(Map<String, Class<?>> extensionClasses, String dir, String type) {
        String fileName = dir + type;
        try {
            Enumeration<java.net.URL> urls;
            ClassLoader classLoader = findClassLoader();
            if (classLoader != null) {
                urls = classLoader.getResources(fileName);
            } else {
                urls = ClassLoader.getSystemResources(fileName);
            }
            if (urls != null) {
                while (urls.hasMoreElements()) {
                    java.net.URL resourceURL = urls.nextElement();
                    loadResource(extensionClasses, classLoader, resourceURL);
                }
            }
        } catch (Throwable t) {
            logger.error("Exception occurred when loading extension class (interface: " +
                    type + ", description file: " + fileName + ").", t);
        }
    }
```

先获得完整的文件名，遍历每一个文件，在loadResource方法中去加载每个文件的内容。

##### loadResource

```
    private void loadResource(Map<String, Class<?>> extensionClasses, ClassLoader classLoader, java.net.URL resourceURL) {
        try {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(resourceURL.openStream(), StandardCharsets.UTF_8))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    final int ci = line.indexOf('#');
                    if (ci >= 0) {
                        line = line.substring(0, ci);
                    }
                    line = line.trim();
                    if (line.length() > 0) {
                        try {
                            String name = null;
                            int i = line.indexOf('=');
                            if (i > 0) {
                                name = line.substring(0, i).trim();
                                line = line.substring(i + 1).trim();
                            }
                            if (line.length() > 0) {
                                loadClass(extensionClasses, resourceURL, Class.forName(line, true, classLoader), name);
                            }
                        } catch (Throwable t) {
                            IllegalStateException e = new IllegalStateException("Failed to load extension class (interface: " + type + ", class line: " + line + ") in " + resourceURL + ", cause: " + t.getMessage(), t);
                            exceptions.put(line, e);
                        }
                    }
                }
            }
        } catch (Throwable t) {
            logger.error("Exception occurred when loading extension class (interface: " +
                    type + ", class file: " + resourceURL + ") in " + resourceURL, t);
        }
    }
```

读取里面的内容，跳过“#”注释的内容，根据配置文件中的key=value的形式去分割，然后去加载value对应的类。

##### loadClass

```
    private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name) throws NoSuchMethodException {
        if (!type.isAssignableFrom(clazz)) {
            throw new IllegalStateException("Error occurred when loading extension class (interface: " +
                    type + ", class line: " + clazz.getName() + "), class "
                    + clazz.getName() + " is not subtype of interface.");
        }
        if (clazz.isAnnotationPresent(Adaptive.class)) {
            cacheAdaptiveClass(clazz);
        } else if (isWrapperClass(clazz)) {
            cacheWrapperClass(clazz);
        } else {
            clazz.getConstructor();
            if (StringUtils.isEmpty(name)) {
                name = findAnnotationName(clazz);
                if (name.length() == 0) {
                    throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + resourceURL);
                }
            }

            String[] names = NAME_SEPARATOR.split(name);
            if (ArrayUtils.isNotEmpty(names)) {
                cacheActivateClass(clazz, names[0]);
                for (String n : names) {
                    cacheName(clazz, n);
                    saveInExtensionClass(extensionClasses, clazz, n);
                }
            }
        }
    }
```

#### AdaptiveExtensionFactory

```
@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {

    private final List<ExtensionFactory> factories;

    public AdaptiveExtensionFactory() {
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList<ExtensionFactory>();
        for (String name : loader.getSupportedExtensions()) {
            list.add(loader.getExtension(name));
        }
        factories = Collections.unmodifiableList(list);
    }

    @Override
    public <T> T getExtension(Class<T> type, String name) {
        for (ExtensionFactory factory : factories) {
            T extension = factory.getExtension(type, name);
            if (extension != null) {
                return extension;
            }
        }
        return null;
    }

}
```

1. factories是扩展对象的集合，当用户没有自己实现ExtensionFactory接口，则这个属性就只会有两种对象，分别是 SpiExtensionFactory 和 SpringExtensionFactory 。
2. 构造器中是把所有支持的扩展名的扩展对象加入到集合
3. 实现了接口的getExtension方法，通过接口和扩展名来获取扩展对象。

## 注册中心

### 注册中心是什么？

服务治理框架中可以大致分为服务通信和服务管理两个部分，服务管理可以分为服务注册、服务发现以及服务被热加工介入，服务提供者Provider会往注册中心注册服务，而消费者Consumer会从注册中心中订阅相关的服务，并不会订阅全部的服务。

从上图看，可以清晰的看到Registry所起到的作用，我举个例子，Registry类似于一个自动售货机，服务提供者类似于一个商品生产者，他会往这个自动售卖机中添加商品，也就是注册服务，而消费者则会到注册中心中购买自己需要的商品，也就是订阅对应的服务。这样解释应该就可以比较直观的感受到注册中心所担任的是什么角色。

#### RegistryService

```
public interface RegistryService {

    /**
     * Register data, such as : provider service, consumer address, route rule, override rule and other data.
     * <p>
     * Registering is required to support the contract:<br>
     * 1. When the URL sets the check=false parameter. When the registration fails, the exception is not thrown and retried in the background. Otherwise, the exception will be thrown.<br>
     * 2. When URL sets the dynamic=false parameter, it needs to be stored persistently, otherwise, it should be deleted automatically when the registrant has an abnormal exit.<br>
     * 3. When the URL sets category=routers, it means classified storage, the default category is providers, and the data can be notified by the classified section. <br>
     * 4. When the registry is restarted, network jitter, data can not be lost, including automatically deleting data from the broken line.<br>
     * 5. Allow URLs which have the same URL but different parameters to coexist,they can't cover each other.<br>
     *
     * @param url  Registration information , is not allowed to be empty, e.g: dubbo://10.20.153.10/org.apache.dubbo.foo.BarService?version=1.0.0&application=kylin
     */
    void register(URL url);

    /**
     * Unregister
     * <p>
     * Unregistering is required to support the contract:<br>
     * 1. If it is the persistent stored data of dynamic=false, the registration data can not be found, then the IllegalStateException is thrown, otherwise it is ignored.<br>
     * 2. Unregister according to the full url match.<br>
     *
     * @param url Registration information , is not allowed to be empty, e.g: dubbo://10.20.153.10/org.apache.dubbo.foo.BarService?version=1.0.0&application=kylin
     */
    void unregister(URL url);

    /**
     * Subscribe to eligible registered data and automatically push when the registered data is changed.
     * <p>
     * Subscribing need to support contracts:<br>
     * 1. When the URL sets the check=false parameter. When the registration fails, the exception is not thrown and retried in the background. <br>
     * 2. When URL sets category=routers, it only notifies the specified classification data. Multiple classifications are separated by commas, and allows asterisk to match, which indicates that all categorical data are subscribed.<br>
     * 3. Allow interface, group, version, and classifier as a conditional query, e.g.: interface=org.apache.dubbo.foo.BarService&version=1.0.0<br>
     * 4. And the query conditions allow the asterisk to be matched, subscribe to all versions of all the packets of all interfaces, e.g. :interface=*&group=*&version=*&classifier=*<br>
     * 5. When the registry is restarted and network jitter, it is necessary to automatically restore the subscription request.<br>
     * 6. Allow URLs which have the same URL but different parameters to coexist,they can't cover each other.<br>
     * 7. The subscription process must be blocked, when the first notice is finished and then returned.<br>
     *
     * @param url      Subscription condition, not allowed to be empty, e.g. consumer://10.20.153.10/org.apache.dubbo.foo.BarService?version=1.0.0&application=kylin
     * @param listener A listener of the change event, not allowed to be empty
     */
    void subscribe(URL url, NotifyListener listener);

    /**
     * Unsubscribe
     * <p>
     * Unsubscribing is required to support the contract:<br>
     * 1. If don't subscribe, ignore it directly.<br>
     * 2. Unsubscribe by full URL match.<br>
     *
     * @param url      Subscription condition, not allowed to be empty, e.g. consumer://10.20.153.10/org.apache.dubbo.foo.BarService?version=1.0.0&application=kylin
     * @param listener A listener of the change event, not allowed to be empty
     */
    void unsubscribe(URL url, NotifyListener listener);

    /**
     * Query the registered data that matches the conditions. Corresponding to the push mode of the subscription, this is the pull mode and returns only one result.
     *
     * @param url Query condition, is not allowed to be empty, e.g. consumer://10.20.153.10/org.apache.dubbo.foo.BarService?version=1.0.0&application=kylin
     * @return The registered information list, which may be empty, the meaning is the same as the parameters of {@link org.apache.dubbo.registry.NotifyListener#notify(List<URL>)}.
     * @see org.apache.dubbo.registry.NotifyListener#notify(List)
     */
    List<URL> lookup(URL url);

}
```

这个接口就是协定了注册中心的功能，这里统一说明一下URL，又再次提到URL了，在上篇文章中就说明了dubbo是以总线模式来时刻传递和保存配置信息的，也就是配置信息都被放在URL上进行传递，随时可以取得相关配置信息，而这里提到了URL有别的作用，就是作为类似于节点的作用，首先服务提供者（Provider）启动时需要提供服务，就会向注册中心写下自己的URL地址。然后消费者启动时需要去订阅该服务，则会订阅Provider注册的地址，并且消费者也会写下自己的URL。继续拿我上面的例子，商品生产者生产完商品，它会在把该商品放在自动售卖机的某一个栏目内，二消费者需要买该商品的时候，就是通过该地址去购买，并且会留下自己的购买记录。

1. 注册，如果看懂我上面说的url的作用，那么就很清楚该方法的作用了，这里强调一点，就是注释中讲到的允许URI相同但参数不同的URL并存，不能覆盖，也就是说url值必须唯一的，不能有一模一样。

   ```
   void register(URL url);
   ```

2. 取消注册，该方法也很简单，就是取消注册，也就是商品生产者不在销售该商品， 需要把东西从自动售卖机上取下来，栏目也要取出，这里强调按全URL匹配取消注册。

   ```
   void unregister(URL url);
   ```

3. 订阅，这里不是根据全URL匹配订阅的，而是根据条件去订阅，也就是说可以订阅多个服务。listener是用来监听处理注册数据变更的事件。

   ```
   void subscribe(URL url, NotifyListener listener);
   ```

4. 取消订阅，这是按照全URL匹配去取消订阅的。

   ```
   void unsubscribe(URL url, NotifyListener listener);
   ```

5. 查询注册列表，通过url进行条件查询所匹配的所有URL集合。

   ```
   List<URL> lookup(URL url);
   ```

#### Registry

注册中心接口，该接口很好理解，就是把节点以及注册中心服务的方法整合在了这个接口里面。我们来看看源代码：

```
public interface Registry extends Node, RegistryService {
}
```

可以看到该接口并没有自己的方法，就是继承了Node和RegistryService接口。这里的Node是节点的接口，里面协定了关于节点的一些操作方法，我们可以来看看源代码：

```
public interface Node {
    //获得节点地址
    URL getUrl();
    //判断节点是否可用
    boolean isAvailable();
    //销毁节点
    void destroy();

}
```

#### RegistryFactory

这个接口是注册中心的工厂接口，用来返回注册中心的对象。来看看它的源码：

```
@SPI("dubbo")
public interface RegistryFactory {

    @Adaptive({"protocol"})
    Registry getRegistry(URL url);

}
```

该接口是一个可扩展接口，可以看到该接口上有个@SPI注解，并且默认值为dubbo，也就是默认扩展的是DubboRegistryFactory，并且可以在getRegistry方法上可以看到有@Adaptive注解，那么该接口会动态生成一个适配器RegistryFactory$Adaptive，并且会去首先扩展url.protocol的值对应的实现类。

#### NotifyListener

该接口只有一个notify方法，通知监听器。当收到服务变更通知时触发。来看看它的源码：

```
public interface NotifyListener {
    /**
     * 当收到服务变更通知时触发。
     * <p>
     * 通知需处理契约：<br>
     * 1. 总是以服务接口和数据类型为维度全量通知，即不会通知一个服务的同类型的部分数据，用户不需要对比上一次通知结果。<br>
     * 2. 订阅时的第一次通知，必须是一个服务的所有类型数据的全量通知。<br>
     * 3. 中途变更时，允许不同类型的数据分开通知，比如：providers, consumers, routers, overrides，允许只通知其中一种类型，但该类型的数据必须是全量的，不是增量的。<br>
     * 4. 如果一种类型的数据为空，需通知一个empty协议并带category参数的标识性URL数据。<br>
     * 5. 通知者(即注册中心实现)需保证通知的顺序，比如：单线程推送，队列串行化，带版本对比。<br>
     *
     * @param urls 已注册信息列表，总不为空，含义同{@link com.alibaba.dubbo.registry.RegistryService#lookup(URL)}的返回值。
     */
    void notify(List<URL> urls);

}
```

#### AbstractRegistry

AbstractRegistry实现的是Registry接口，是Registry的抽象类。为了减轻注册中心的压力，在该类中实现了把本地URL缓存到property文件中的机制，并且实现了注册中心的注册、订阅等方法。

##### 1.属性

```
    // URL的地址分隔符，在缓存文件中使用，服务提供者的URL分隔
    private static final char URL_SEPARATOR = ' ';
    // URL地址分隔正则表达式，用于解析文件缓存中服务提供者URL列表
    private static final String URL_SPLIT = "\\s+";
    // 日志输出
    protected final Logger logger = LoggerFactory.getLogger(getClass());
    // 本地磁盘缓存，有一个特殊的key值为registies，记录的是注册中心列表，其他记录的都是服务提供者列表
    private final Properties properties = new Properties();
    // 缓存写入执行器
    private final ExecutorService registryCacheExecutor = Executors.newFixedThreadPool(1, new NamedThreadFactory("DubboSaveRegistryCache", true));
    // 是否同步保存文件标志
    private final boolean syncSaveFile;
    //数据版本号
    private final AtomicLong lastCacheChanged = new AtomicLong();
    // 已注册 URL 集合
    // 注册的 URL 不仅仅可以是服务提供者的，也可以是服务消费者的
    private final Set<URL> registered = new ConcurrentHashSet<URL>();
    // 订阅URL的监听器集合
    private final ConcurrentMap<URL, Set<NotifyListener>> subscribed = new ConcurrentHashMap<URL, Set<NotifyListener>>();
    // 某个消费者被通知的某一类型的 URL 集合
    // 第一个key是消费者的URL，对应的就是哪个消费者。
    // value是一个map集合，该map集合的key是分类的意思，例如providers、routes等，value就是被通知的URL集合
    private final ConcurrentMap<URL, Map<String, List<URL>>> notified = new ConcurrentHashMap<URL, Map<String, List<URL>>>();
    // 注册中心 URL
    private URL registryUrl;
    // 本地磁盘缓存文件，缓存注册中心的数据
    private File file;
```

理解属性的含义对于后面去解读方法很有帮助，从上面可以看到除了注册中心相关的一些属性外，可以看到好几个是个属性跟磁盘缓存文件和读写文件有关的，这就是上面提到的把URL缓存到本地property的相关属性这里有几个需要关注的点：

1. properties：properties的数据跟本地文件的数据同步，当启动时，会从文件中读取数据到properties，而当properties中数据变化时，会写入到file。而properties是一个key对应一个列表，比如说key就是消费者的url，而值就是服务提供者列表、路由规则列表、配置规则列表。就是类似属性notified的含义。需要注意的是properties有一个特殊的key为registies，记录的是注册中心列表。
2. lastCacheChanged：因为每次写入file都是全部覆盖的写入，不是增量的去写入到文件，所以需要有这个版本号来避免老版本覆盖新版本。
3. notified：跟properties的区别是第一数据来源不是文件，而是从注册中心中读取，第二个notified根据分类把同一类的值做了聚合。

##### 2.构造方法AbstractRegistry

先来看看源码：

```
    public AbstractRegistry(URL url) {
        // 把url放到registryUrl中
        setUrl(url);
        // Start file save timer
        // 从url中读取是否同步保存文件的配置，如果没有值默认用异步保存文件
        syncSaveFile = url.getParameter(Constants.REGISTRY_FILESAVE_SYNC_KEY, false);
        // 获得file路径
        String filename = url.getParameter(Constants.FILE_KEY, System.getProperty("user.home") + "/.dubbo/dubbo-registry-" + url.getParameter(Constants.APPLICATION_KEY) + "-" + url.getAddress() + ".cache");
        File file = null;
        if (ConfigUtils.isNotEmpty(filename)) {
            //创建文件
            file = new File(filename);
            if (!file.exists() && file.getParentFile() != null && !file.getParentFile().exists()) {
                if (!file.getParentFile().mkdirs()) {
                    throw new IllegalArgumentException("Invalid registry store file " + file + ", cause: Failed to create directory " + file.getParentFile() + "!");
                }
            }
        }
        this.file = file;
        // 把文件里面的数据写入properties
        loadProperties();
        // 通知监听器，URL 变化结果
        notify(url.getBackupUrls());
    }
```

需要关注的几个点：

1. 比如是否同步保存文件、比如保存的文件路径都优先选择URL上的配置，如果没有相关的配置，再选用默认配置。
2. 构造AbstractRegistry会有把文件里面的数据写入到properties的操作以及通知监听器url变化结果，相关方法介绍在下面给出。

##### 3.filterEmpty

```
    protected static List<URL> filterEmpty(URL url, List<URL> urls) {
        if (urls == null || urls.isEmpty()) {
            List<URL> result = new ArrayList<URL>(1);
            result.add(url.setProtocol(Constants.EMPTY_PROTOCOL));
            return result;
        }
        return urls;
    }
```

这个方法的源码都不需要解释了，很简单，就是判断url集合是否为空，如果为空，则把url中key为empty的值加入到集合。该方法只有在notify方法中用到，为了防止通知的URL变化结果为空。

##### 4.doSaveProperties

该方法比较长，我这里不贴源码了，需要的就查看github上的分析，该方法主要是将内存缓存properties中的数据存储到文件中，并且在里面做了版本号的控制，防止老的版本数据覆盖了新版本数据。数据流向是跟loadProperties方法相反。

##### 5.loadProperties

```
    private void loadProperties() {
        if (file != null && file.exists()) {
            InputStream in = null;
            try {
                in = new FileInputStream(file);
                // 把数据写入到内存缓存中
                properties.load(in);
                if (logger.isInfoEnabled()) {
                    logger.info("Load registry store file " + file + ", data: " + properties);
                }
            } catch (Throwable e) {
                logger.warn("Failed to load registry store file " + file, e);
            } finally {
                if (in != null) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        logger.warn(e.getMessage(), e);
                    }
                }
            }
        }
    }
```

该方法就是加载本地磁盘缓存文件到内存缓存，也就是把文件里面的数据写入properties，可以对比doSaveProperties方法，其中关键的实现就是properties.load和properties.store的区别，逻辑并不难。跟doSaveProperties的数据流向相反。

##### 6.getCacheUrls

```
    public List<URL> getCacheUrls(URL url) {
        for (Map.Entry<Object, Object> entry : properties.entrySet()) {
            // key为某个分类，例如服务提供者分类
            String key = (String) entry.getKey();
            // value为某个分类的列表，例如服务提供者列表
            String value = (String) entry.getValue();
            if (key != null && key.length() > 0 && key.equals(url.getServiceKey())
                    && (Character.isLetter(key.charAt(0)) || key.charAt(0) == '_')
                    && value != null && value.length() > 0) {
                //分割出列表的每个值
                String[] arr = value.trim().split(URL_SPLIT);
                List<URL> urls = new ArrayList<URL>();
                for (String u : arr) {
                    urls.add(URL.valueOf(u));
                }
                return urls;
            }
        }
        return null;
    }
```

该方法是获得内存缓存properties中相关value，并且返回为一个集合，从该方法中可以很清楚的看出properties中是存储的什么数据格式。

##### 7.lookup

来看看源码：

```
    @Override
    public List<URL> lookup(URL url) {
        List<URL> result = new ArrayList<URL>();
        // 获得该消费者url订阅的 所有被通知的 服务URL集合
        Map<String, List<URL>> notifiedUrls = getNotified().get(url);
        // 判断该消费者是否订阅服务
        if (notifiedUrls != null && notifiedUrls.size() > 0) {
            for (List<URL> urls : notifiedUrls.values()) {
                for (URL u : urls) {
                    // 判断协议是否为空
                    if (!Constants.EMPTY_PROTOCOL.equals(u.getProtocol())) {
                        // 添加 该消费者订阅的服务URL
                        result.add(u);
                    }
                }
            }
        } else {
            // 原子类 避免在获取注册在注册中心的服务url时能够保证是最新的url集合
            final AtomicReference<List<URL>> reference = new AtomicReference<List<URL>>();
            // 通知监听器。当收到服务变更通知时触发
            NotifyListener listener = new NotifyListener() {
                @Override
                public void notify(List<URL> urls) {
                    reference.set(urls);
                }
            };
            // 订阅服务，就是消费者url订阅已经 注册在注册中心的服务（也就是添加该服务的监听器）
            subscribe(url, listener); // Subscribe logic guarantees the first notify to return
            List<URL> urls = reference.get();
            if (urls != null && !urls.isEmpty()) {
                for (URL u : urls) {
                    if (!Constants.EMPTY_PROTOCOL.equals(u.getProtocol())) {
                        result.add(u);
                    }
                }
            }
        }
        return result;
    }
```

该方法是实现了RegistryService接口的方法，作用是获得消费者url订阅的服务URL列表。该方法有几个地方有些绕我在这里重点讲解一下：

1. URL可能是消费者URL，也可能是注册在注册中心的服务URL，我在注释中在URL加了修饰，为了能更明白的区分。
2. 订阅了的服务URL一定是在注册中心中注册了的。
3. 关于订阅服务subscribe方法和通知监听器NotifyListener，我会在下面解释。

##### 8.register && unregister

这两个方法实现了RegistryService接口的方法，里面的逻辑很简单，所有我就不贴代码了，以免影响篇幅，如果真想看，可以进到我github查看，下面我会贴出这部分注释github的地址。其中注册的逻辑就是把url加入到属性registered，而取消注册的逻辑就是把url从该属性中移除，该属性在上面有介绍。真正的实现是在FailbackRegistry类中，FailbackRegistry类我会在下面介绍。

##### 9.subscribe && unsubscribe

这两个方法实现了RegistryService接口的方法，分别是订阅和取消订阅，我就贴一个订阅的代码：

```
    @Override
    public void subscribe(URL url, NotifyListener listener) {
        if (url == null) {
            throw new IllegalArgumentException("subscribe url == null");
        }
        if (listener == null) {
            throw new IllegalArgumentException("subscribe listener == null");
        }
        if (logger.isInfoEnabled()) {
            logger.info("Subscribe: " + url);
        }
        // 获得该消费者url 已经订阅的服务 的监听器集合
        Set<NotifyListener> listeners = subscribed.get(url);
        if (listeners == null) {
            subscribed.putIfAbsent(url, new ConcurrentHashSet<NotifyListener>());
            listeners = subscribed.get(url);
        }
        // 添加某个服务的监听器
        listeners.add(listener);
    }
```

从源代码可以看到，其实订阅也就是把服务通知监听器加入到subscribed中，具体的实现也是在FailbackRegistry类中。

##### 10.recover

恢复方法，在注册中心断开，重连成功的时候，会恢复注册和订阅。

```
    protected void recover() throws Exception {
        // register
        //把内存缓存中的registered取出来遍历进行注册
        Set<URL> recoverRegistered = new HashSet<URL>(getRegistered());
        if (!recoverRegistered.isEmpty()) {
            if (logger.isInfoEnabled()) {
                logger.info("Recover register url " + recoverRegistered);
            }
            for (URL url : recoverRegistered) {
                register(url);
            }
        }
        // subscribe
        //把内存缓存中的subscribed取出来遍历进行订阅
        Map<URL, Set<NotifyListener>> recoverSubscribed = new HashMap<URL, Set<NotifyListener>>(getSubscribed());
        if (!recoverSubscribed.isEmpty()) {
            if (logger.isInfoEnabled()) {
                logger.info("Recover subscribe url " + recoverSubscribed.keySet());
            }
            for (Map.Entry<URL, Set<NotifyListener>> entry : recoverSubscribed.entrySet()) {
                URL url = entry.getKey();
                for (NotifyListener listener : entry.getValue()) {
                    subscribe(url, listener);
                }
            }
        }
    }
```

##### 11.notify

```
protected void notify(List<URL> urls) {
    if (urls == null || urls.isEmpty()) return;
    // 遍历订阅URL的监听器集合，通知他们
    for (Map.Entry<URL, Set<NotifyListener>> entry : getSubscribed().entrySet()) {
        URL url = entry.getKey();

        // 匹配
        if (!UrlUtils.isMatch(url, urls.get(0))) {
            continue;
        }
        // 遍历监听器集合，通知他们
        Set<NotifyListener> listeners = entry.getValue();
        if (listeners != null) {
            for (NotifyListener listener : listeners) {
                try {
                    notify(url, listener, filterEmpty(url, urls));
                } catch (Throwable t) {
                    logger.error("Failed to notify registry event, urls: " + urls + ", cause: " + t.getMessage(), t);
                }
            }
        }
    }
}
protected void notify(URL url, NotifyListener listener, List<URL> urls) {
    if (url == null) {
        throw new IllegalArgumentException("notify url == null");
    }
    if (listener == null) {
        throw new IllegalArgumentException("notify listener == null");
    }
    if ((urls == null || urls.isEmpty())
            && !Constants.ANY_VALUE.equals(url.getServiceInterface())) {
        logger.warn("Ignore empty notify urls for subscribe url " + url);
        return;
    }
    if (logger.isInfoEnabled()) {
        logger.info("Notify urls for subscribe url " + url + ", urls: " + urls);
    }
    Map<String, List<URL>> result = new HashMap<String, List<URL>>();
    // 将urls进行分类
    for (URL u : urls) {
        if (UrlUtils.isMatch(url, u)) {
            // 按照url中key为category对应的值进行分类，如果没有该值，就找key为providers的值进行分类
            String category = u.getParameter(Constants.CATEGORY_KEY, Constants.DEFAULT_CATEGORY);
            List<URL> categoryList = result.get(category);
            if (categoryList == null) {
                categoryList = new ArrayList<URL>();
                // 分类结果放入result
                result.put(category, categoryList);
            }
            categoryList.add(u);
        }
    }
    if (result.size() == 0) {
        return;
    }
    // 获得某一个消费者被通知的url集合（通知的 URL 变化结果）
    Map<String, List<URL>> categoryNotified = notified.get(url);
    if (categoryNotified == null) {
        // 添加该消费者对应的url
        notified.putIfAbsent(url, new ConcurrentHashMap<String, List<URL>>());
        categoryNotified = notified.get(url);
    }
    // 处理通知监听器URL 变化结果
    for (Map.Entry<String, List<URL>> entry : result.entrySet()) {
        String category = entry.getKey();
        List<URL> categoryList = entry.getValue();
        // 把分类标实和分类后的列表放入notified的value中
        // 覆盖到 `notified`
        // 当某个分类的数据为空时，会依然有 urls 。其中 `urls[0].protocol = empty` ，通过这样的方式，处理所有服务提供者为空的情况。
        categoryNotified.put(category, categoryList);
        // 保存到文件
        saveProperties(url);
        //通知监听器
        listener.notify(categoryList);
    }
}
```

notify方法是通知监听器，url的变化结果，不过变化的是全量数据，全量数据意思就是是以服务接口和数据类型为维度全量通知，即不会通知一个服务的同类型的部分数据，用户不需要对比上一次通知结果。这里要注意几个重点：

1. 发起订阅后，会获取全量数据，此时会调用notify方法。即Registry 获取到了全量数据
2. 每次注册中心发生变更时会调用notify方法虽然变化是增量，调用这个方法的调用方，已经进行处理，传入的urls依然是全量的。
3. listener.notify，通知监听器，例如，有新的服务提供者启动时，被通知，创建新的 Invoker 对象。

##### 12.saveProperties

先来看看源码：

```
private void saveProperties(URL url) {
    if (file == null) {
        return;
    }
    try {
        // 拼接url
        StringBuilder buf = new StringBuilder();
        Map<String, List<URL>> categoryNotified = notified.get(url);
        if (categoryNotified != null) {
            for (List<URL> us : categoryNotified.values()) {
                for (URL u : us) {
                    if (buf.length() > 0) {
                        buf.append(URL_SEPARATOR);
                    }
                    buf.append(u.toFullString());
                }
            }
        }
        // 设置到properties中
        properties.setProperty(url.getServiceKey(), buf.toString());
        // 增加版本号
        long version = lastCacheChanged.incrementAndGet();
        if (syncSaveFile) {
            // 将集合中的数据存储到文件中
            doSaveProperties(version);
        } else {
            //异步开启保存到文件
            registryCacheExecutor.execute(new SaveProperties(version));
        }
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
}
```

该方法是单个消费者url对应在notified中的数据，保存在到文件，而保存到文件的操作是调用了doSaveProperties方法，该方法跟doSaveProperties的区别是doSaveProperties方法将properties数据全部覆盖性的保存到文件，而saveProperties只是保存单个消费者url的数据。

##### 13.destroy

该方法在JVM关闭时调用，进行取消注册和订阅的操作。具体逻辑就是调用了unregister和unsubscribe方法，有需要看源码的可以进入github查看。

#### （六）support包下的FailbackRegistry

我在上面讲AbstractRegistry类的时候已经提到了FailbackRegistry，FailbackRegistry继承了AbstractRegistry，AbstractRegistry中的注册订阅等方法，实际上就是一些内存缓存的变化，而真正的注册订阅的实现逻辑在FailbackRegistry实现，并且FailbackRegistry提供了失败重试的机制。

> 源码注释地址：[https://github.com/CrazyHZM/i...](https://github.com/CrazyHZM/incubator-dubbo/blob/analyze-2.6.x/dubbo-registry/dubbo-registry-api/src/main/java/com/alibaba/dubbo/registry/support/FailbackRegistry.java)

##### 1.属性

```
// Scheduled executor service
// 定时任务执行器
private final ScheduledExecutorService retryExecutor = Executors.newScheduledThreadPool(1, new NamedThreadFactory("DubboRegistryFailedRetryTimer", true));

// Timer for failure retry, regular check if there is a request for failure, and if there is, an unlimited retry
// 失败重试定时器，定时去检查是否有请求失败的，如有，无限次重试。
private final ScheduledFuture<?> retryFuture;

// 注册失败的URL集合
private final Set<URL> failedRegistered = new ConcurrentHashSet<URL>();

// 取消注册失败的URL集合
private final Set<URL> failedUnregistered = new ConcurrentHashSet<URL>();

// 订阅失败的监听器集合
private final ConcurrentMap<URL, Set<NotifyListener>> failedSubscribed = new ConcurrentHashMap<URL, Set<NotifyListener>>();

// 取消订阅失败的监听器集合
private final ConcurrentMap<URL, Set<NotifyListener>> failedUnsubscribed = new ConcurrentHashMap<URL, Set<NotifyListener>>();

// 通知失败的URL集合
private final ConcurrentMap<URL, Map<NotifyListener, List<URL>>> failedNotified = new ConcurrentHashMap<URL, Map<NotifyListener, List<URL>>>();
```

该类的属性比较好理解，也可以很明显看出这些属性都是跟失败重试机制相关。

##### 2.构造函数

```
public FailbackRegistry(URL url) {
    super(url);
    // 从url中读取重试频率，如果为空，则默认5000ms
    this.retryPeriod = url.getParameter(Constants.REGISTRY_RETRY_PERIOD_KEY, Constants.DEFAULT_REGISTRY_RETRY_PERIOD);
    // 创建失败重试定时器
    this.retryFuture = retryExecutor.scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            // Check and connect to the registry
            try {
                //重试
                retry();
            } catch (Throwable t) { // Defensive fault tolerance
                logger.error("Unexpected error occur at failed retry, cause: " + t.getMessage(), t);
            }
        }
    }, retryPeriod, retryPeriod, TimeUnit.MILLISECONDS);
}
```

构造函数主要是创建了失败重试的定时器，重试频率从URL取，如果没有设置，则默认为5000ms。

##### 3.register && unregister && subscribe && unsubscribe

这四个方法就是注册、取消注册、订阅、取消订阅的具体实现，因为代码逻辑极其相似，所以为放在一起，下面为只贴出注册的源码：

```
public void register(URL url) {
    super.register(url);
    //首先从失败的缓存中删除该url
    failedRegistered.remove(url);
    failedUnregistered.remove(url);
    try {
        // Sending a registration request to the server side
        // 向注册中心发送一个注册请求
        doRegister(url);
    } catch (Exception e) {
        Throwable t = e;

        // If the startup detection is opened, the Exception is thrown directly.
        // 如果开启了启动时检测，则直接抛出异常
        boolean check = getUrl().getParameter(Constants.CHECK_KEY, true)
                && url.getParameter(Constants.CHECK_KEY, true)
                && !Constants.CONSUMER_PROTOCOL.equals(url.getProtocol());
        boolean skipFailback = t instanceof SkipFailbackWrapperException;
        if (check || skipFailback) {
            if (skipFailback) {
                t = t.getCause();
            }
            throw new IllegalStateException("Failed to register " + url + " to registry " + getUrl().getAddress() + ", cause: " + t.getMessage(), t);
        } else {
            logger.error("Failed to register " + url + ", waiting for retry, cause: " + t.getMessage(), t);
        }

        // Record a failed registration request to a failed list, retry regularly
        // 把这个注册失败的url放入缓存，并且定时重试。
        failedRegistered.add(url);
    }
}
```

可以看到，逻辑很清晰，就是做了一个doRegister的操作，如果失败抛出异常，则加入到失败的缓存中进行重试。为这里要解释的是doRegister，与之对应的还有doUnregister、doSubscribe、doUnsubscribe三个方法，是FailbackRegistry抽象出来的方法，意图在于每种实现注册中心的方法不一样，相对应的注册、订阅等操作也会有所区别，而把这四个方法抽象出现，为了让子类只去关注这四个的实现，比如说redis实现的注册中心跟zookeeper实现的注册中心方式肯定不一样，那么对应的注册订阅等操作也有所不同，那么各自只要去实现该抽象方法即可。

其他的三个方法有需要的可以查看github上的我写的注释。

##### 4.notify

```
@Override
protected void notify(URL url, NotifyListener listener, List<URL> urls) {
    if (url == null) {
        throw new IllegalArgumentException("notify url == null");
    }
    if (listener == null) {
        throw new IllegalArgumentException("notify listener == null");
    }
    try {
        // 通知 url 数据变化
        doNotify(url, listener, urls);
    } catch (Exception t) {
        // Record a failed registration request to a failed list, retry regularly
        // 放入失败的缓存中，重试
        Map<NotifyListener, List<URL>> listeners = failedNotified.get(url);
        if (listeners == null) {
            failedNotified.putIfAbsent(url, new ConcurrentHashMap<NotifyListener, List<URL>>());
            listeners = failedNotified.get(url);
        }
        listeners.put(listener, urls);
        logger.error("Failed to notify for subscribe " + url + ", waiting for retry, cause: " + t.getMessage(), t);
    }
}

protected void doNotify(URL url, NotifyListener listener, List<URL> urls) {
    super.notify(url, listener, urls);
}
```

可以看到notify不一样，他还是又回去调用了父类AbstractRegistry的notify，与上述四个方法不一样。

##### 5.revocer

```
@Override
protected void recover() throws Exception {
    // register
    // register 恢复注册，添加到 `failedRegistered` ，定时重试
    Set<URL> recoverRegistered = new HashSet<URL>(getRegistered());
    if (!recoverRegistered.isEmpty()) {
        if (logger.isInfoEnabled()) {
            logger.info("Recover register url " + recoverRegistered);
        }
        for (URL url : recoverRegistered) {
            failedRegistered.add(url);
        }
    }
    // subscribe
    // subscribe 恢复订阅，添加到 `failedSubscribed` ，定时重试
    Map<URL, Set<NotifyListener>> recoverSubscribed = new HashMap<URL, Set<NotifyListener>>(getSubscribed());
    if (!recoverSubscribed.isEmpty()) {
        if (logger.isInfoEnabled()) {
            logger.info("Recover subscribe url " + recoverSubscribed.keySet());
        }
        for (Map.Entry<URL, Set<NotifyListener>> entry : recoverSubscribed.entrySet()) {
            URL url = entry.getKey();
            for (NotifyListener listener : entry.getValue()) {
                addFailedSubscribed(url, listener);
            }
        }
    }
}
```

重写了父类的recover，将注册和订阅放入到对应的失败缓存中，然后定时重试。

##### 6.retry

该方法中实现了重试的逻辑，分别对注册失败failedRegistered、取消注册失败failedUnregistered、订阅失败failedSubscribed、取消订阅失败failedUnsubscribed、通知监听器失败failedNotified这五个缓存中的元素进行重试，重试的逻辑就是调用了相关的方法，然后从缓存中删除，例如重试注册，先进行doRegister，然后把该url从failedRegistered移除。具体的注释请到GitHub查看。

#### （七）support包下的AbstractRegistryFactory

该类实现了RegistryFactory接口，抽象了createRegistry方法，它实现了Registry的容器管理。

##### 1.属性

```
// Log output
// 日志记录
private static final Logger LOGGER = LoggerFactory.getLogger(AbstractRegistryFactory.class);

// The lock for the acquisition process of the registry
// 锁，对REGISTRIES访问对竞争控制
private static final ReentrantLock LOCK = new ReentrantLock();

// Registry Collection Map<RegistryAddress, Registry>
// Registry 集合
private static final Map<String, Registry> REGISTRIES = new ConcurrentHashMap<String, Registry>();
```

##### 2.destroyAll

```
public static void destroyAll() {
    if (LOGGER.isInfoEnabled()) {
        LOGGER.info("Close all registries " + getRegistries());
    }
    // Lock up the registry shutdown process
    // 获得锁
    LOCK.lock();
    try {
        for (Registry registry : getRegistries()) {
            try {
                // 销毁
                registry.destroy();
            } catch (Throwable e) {
                LOGGER.error(e.getMessage(), e);
            }
        }
        // 清空缓存
        REGISTRIES.clear();
    } finally {
        // Release the lock
        // 释放锁
        LOCK.unlock();
    }
}
```

该方法作用是销毁所有的Registry对象，并且清除内存缓存，逻辑比较简单，关键就是对REGISTRIES进行同步的操作。

##### 3.getRegistry

```
@Override
public Registry getRegistry(URL url) {
    // 修改url
    url = url.setPath(RegistryService.class.getName())
            .addParameter(Constants.INTERFACE_KEY, RegistryService.class.getName())
            .removeParameters(Constants.EXPORT_KEY, Constants.REFER_KEY);
    // 计算key值
    String key = url.toServiceString();
    // Lock the registry access process to ensure a single instance of the registry
    // 获得锁
    LOCK.lock();
    try {
        Registry registry = REGISTRIES.get(key);
        if (registry != null) {
            return registry;
        }
        // 创建Registry对象
        registry = createRegistry(url);
        if (registry == null) {
            throw new IllegalStateException("Can not create registry " + url);
        }
        // 添加到缓存。
        REGISTRIES.put(key, registry);
        return registry;
    } finally {
        // Release the lock
        // 释放锁
        LOCK.unlock();
    }
}
```

该方法是实现了RegistryFactory接口中的方法，关于key值的计算我会在后续讲解URL的文章中讲到，这里最要注意的是createRegistry，因为AbstractRegistryFactory类把这个方法抽象出来，为了让子类只要关注该方法，比如说redis实现的注册中心和zookeeper实现的注册中心创建方式肯定不同，而他们相同的一些操作都已经在AbstractRegistryFactory中实现。所以只要关注并且实现该抽象方法即可。

#### （八）support包下的ConsumerInvokerWrapper && ProviderInvokerWrapper

这两个类实现了Invoker接口，分别是服务消费者和服务提供者的Invoker的包装器，其中就包装了一些属性，我们来看看源码：

##### 1.ConsumerInvokerWrapper属性

```
// Invoker 对象
private Invoker<T> invoker;
// 原始url
private URL originUrl;
// 注册中心url
private URL registryUrl;
// 消费者url
private URL consumerUrl;
// 注册中心 Directory
private RegistryDirectory registryDirectory;
```

##### 2.ProviderInvokerWrapper属性

```
// Invoker对象
private Invoker<T> invoker;
// 原始url
private URL originUrl;
// 注册中心url
private URL registryUrl;
// 服务提供者url
private URL providerUrl;
// 是否注册
private volatile boolean isReg;
```

这两个类都被运用在Dubbo QOS中，需要了解Dubbo QOS的可以到官方文档里面查看

> QOS网址：[http://dubbo.apache.org/zh-cn...](http://dubbo.apache.org/zh-cn/docs/user/references/qos.html)

#### （九）support包下的ProviderConsumerRegTable

服务提供者和消费者注册表，存储JVM进程中服务提供者和消费者的Invoker，该类也是被运用在QOS中，包括上面的两个类，都跟QOS中的Offline下线服务命令和ls列出消费者和提供者逻辑实现有关系。我们可以看看它的属性：

```
// 服务提供者Invoker集合，key 为服务提供者的url 计算的key，就是url.toServiceString()方法得到的
public static ConcurrentHashMap<String, Set<ProviderInvokerWrapper>> providerInvokers = new ConcurrentHashMap<String, Set<ProviderInvokerWrapper>>();
// 服务消费者的Invoker集合，key 为服务消费者的url 计算的key，url.toServiceString()方法得到的
public static ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>> consumerInvokers = new ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>>();
```

可以看到，其实记录的服务提供者、消费者、注册中心中间的调用链，为了从一方出发能够很直观的找到跟它相关联的所有调用链。

该类中的其他方法请自行查看，这部分跟运维命令的实现相关，所以为不在这里讲解。

#### （十）support包下的SkipFailbackWrapperException

该类是一个dubbo单独创建的异常，在FailbackRegistry中被使用到，自定义的是一个跳过失败重试的异常。

#### （十一）status包下的RegistryStatusChecker

该类实现了StatusChecker，StatusChecker是一个状态校验的接口，RegistryStatusChecker是它的扩展类，做了一些跟注册中心有关的状态检查和设置。我们来看看源码：

```
@Activate
public class RegistryStatusChecker implements StatusChecker {

    @Override
    public Status check() {
        // 获得所有的注册中心对象
        Collection<Registry> registries = AbstractRegistryFactory.getRegistries();
        if (registries.isEmpty()) {
            return new Status(Status.Level.UNKNOWN);
        }
        Status.Level level = Status.Level.OK;
        StringBuilder buf = new StringBuilder();
        // 拼接注册中心url中的地址
        for (Registry registry : registries) {
            if (buf.length() > 0) {
                buf.append(",");
            }
            buf.append(registry.getUrl().getAddress());
            // 如果注册中心的节点不可用，则拼接disconnected，并且状态设置为error
            if (!registry.isAvailable()) {
                level = Status.Level.ERROR;
                buf.append("(disconnected)");
            } else {
                buf.append("(connected)");
            }
        }
        // 返回状态检查结果
        return new Status(level, buf.toString());
    }

}
```

第一个关注点就是@Activate注解，也就是RegistryStatusChecker类会自动激活加载。该类就实现了接口的check方法，作用就是给注册中心进行状态检查，并且返回检查结果。

------

下面讲的是integration下面的两个类RegistryProtocol和RegistryDirectory，这两个类与注册中心核心的逻辑关系没有那么强。RegistryProtocol是对dubbo-rpc-api的依赖集成，RegistryDirectory是对dubbo-cluster的依赖集成。如果看了下面的解析有点糊涂，可以先跳过这部分，等我出了rpc和cluster相关的文章后再回来看就会比较清晰。

#### （十二）integration包下的RegistryProtocol && RegistryDirectory

- RegistryProtocol实现了Protocol接口，也是Protocol接口等扩展类，但是它可以认为并不是一个真正的协议，他是实际的协议（dubbo . rmi）包装者，这样客户端的请求在一开始如果没有服务端的信息，会先从注册中心拉取服务的注册信息，然后再和服务端直连。RegistryProtocol是基于注册中心发现服务提供者的实现协议。
- RegistryDirectory：注册中心服务，维护着所有可用的远程Invoker或者本地的Invoker。 它的Invoker集合是从注册中心获取的， 它实现了NotifyListener接口实现了回调接口notify方法。比如消费方要调用某远程服务，会向注册中心订阅这个服务的所有服务提供方，订阅时和服务提供方数据有变动时回调消费方的NotifyListener服务的notify方法，回调接口传入所有服务的提供方的url地址然后将urls转化为invokers, 也就是refer应用远程服务。

这两个类等我讲解完rpc和cluster模块之后再进行补充源码解析。

# 注册中心——dubbo

#### DubboRegistry

该类继承了FailbackRegistry类，该类里面封装了一个重连机制，而注册中心核心的功能注册、订阅、取消注册、取消订阅，查询注册列表都是调用了我上一篇文章[《dubbo源码解析（三）注册中心——开篇》](https://segmentfault.com/a/1190000016905715)中讲到的实现方法，毕竟这种实现注册中心的方式是dubbo默认的方式，不过dubbo推荐使用zookeeper，这个后续讲解。

##### 1.属性

```
// 日志记录
private final static Logger logger = LoggerFactory.getLogger(DubboRegistry.class);

// Reconnecting detection cycle: 3 seconds (unit:millisecond)
// 重新连接周期：3秒
private static final int RECONNECT_PERIOD_DEFAULT = 3 * 1000;

// Scheduled executor service
// 任务调度器
private final ScheduledExecutorService reconnectTimer = Executors.newScheduledThreadPool(1, new NamedThreadFactory("DubboRegistryReconnectTimer", true));

// Reconnection timer, regular check connection is available. If unavailable, unlimited reconnection.
// 重新连接执行器，定期检查连接可用，如果不可用，则无限制重连
private final ScheduledFuture<?> reconnectFuture;

// The lock for client acquisition process, lock the creation process of the client instance to prevent repeated clients
// 客户端的锁，保证客户端的原子性，可见行，线程安全。
private final ReentrantLock clientLock = new ReentrantLock();

// 注册中心Invoker
private final Invoker<RegistryService> registryInvoker;

// 注册中心服务对象
private final RegistryService registryService;

// 任务调度器reconnectTimer将等待的时间
private final int reconnectPeriod;
```

看上面的源码，可以看到这里的重连是建立了一个计时器，并且会定期检查连接是否可用，如果不可用，就无限重连。只要懂一点线程相关的知识，这里的属性还是比较好理解的。

##### 2.构造函数DubboRegistry

先来看看源码：

```
public DubboRegistry(Invoker<RegistryService> registryInvoker, RegistryService registryService) {
    // 调用父类FailbackRegistry的构造函数
    super(registryInvoker.getUrl());
    this.registryInvoker = registryInvoker;
    this.registryService = registryService;
    // Start reconnection timer
    // 优先取url中key为reconnect.perio的配置，如果没有，则使用默认的3s
    this.reconnectPeriod = registryInvoker.getUrl().getParameter(Constants.REGISTRY_RECONNECT_PERIOD_KEY, RECONNECT_PERIOD_DEFAULT);
    // 每reconnectPeriod秒去连接，首次连接也延迟reconnectPeriod秒
    reconnectFuture = reconnectTimer.scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            // Check and connect to the registry
            try {
                connect();
            } catch (Throwable t) { // Defensive fault tolerance
                logger.error("Unexpected error occur at reconnect, cause: " + t.getMessage(), t);
            }
        }
    }, reconnectPeriod, reconnectPeriod, TimeUnit.MILLISECONDS);
}
```

这个构造方法中有两个关键点：

1. 关于等待时间优先从url配置中取得，如果没有这个值，再设置为默认值3s。
2. 创建了一个重连计时器，一定的间隔时间去检查是否断开，如果断开就进行连接。

##### 3.connect

该方法是连接注册中心的实现，来看看源码：

```
protected final void connect() {
    try {
        // Check whether or not it is connected
        // 检查注册中心是否已连接
        if (isAvailable()) {
            return;
        }
        if (logger.isInfoEnabled()) {
            logger.info("Reconnect to registry " + getUrl());
        }
        // 获得客户端锁
        clientLock.lock();
        try {
            // Double check whether or not it is connected
            // 二次查询注册中心是否已经连接
            if (isAvailable()) {
                return;
            }
            // 恢复注册和订阅
            recover();
        } finally {
            // 释放锁
            clientLock.unlock();
        }
    } catch (Throwable t) { // Ignore all the exceptions and wait for the next retry
        if (getUrl().getParameter(Constants.CHECK_KEY, true)) {
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            }
            throw new RuntimeException(t.getMessage(), t);
        }
        logger.error("Failed to connect to registry " + getUrl().getAddress() + " from provider/consumer " + NetUtils.getLocalHost() + " use dubbo " + Version.getVersion() + ", cause: " + t.getMessage(), t);
    }
}
```

我们可以看到这里的重连机制其实就是调用了父类FailbackRegistry的recover方法，关于recover方法我在[《dubbo源码解析（三）注册中心——开篇》](https://segmentfault.com/a/1190000016905715)中已经讲解过了。还有要关注的就是需要保证客户端线程安全。需要获得锁和释放锁。

##### 4.isAvailable

该方法就是用来检查注册中心是否连接，源码如下：

```
public boolean isAvailable() {
    if (registryInvoker == null)
        return false;
    return registryInvoker.isAvailable();
}
```

##### 5.destroy

该方法是销毁方法，主要是销毁重连计时器、注册中心的Invoker和任务调度器，源码如下：

```
@Override
public void destroy() {
    super.destroy();
    try {
        // Cancel the reconnection timer
        // 取消重新连接计时器
        if (!reconnectFuture.isCancelled()) {
            reconnectFuture.cancel(true);
        }
    } catch (Throwable t) {
        logger.warn("Failed to cancel reconnect timer", t);
    }
    // 销毁注册中心的Invoker
    registryInvoker.destroy();
    // 关闭任务调度器
    ExecutorUtil.gracefulShutdown(reconnectTimer, reconnectPeriod);
}
```

这里用到了ExecutorUtil中的gracefulShutdown，因为ExecutorUtil是common模块中的类，我在第一篇中讲到我会穿插在各个文章中介绍这个模块，所以我咋这里介绍一下这个gracefulShutdown方法，我们可以看一下这个源码：

```
public static void gracefulShutdown(Executor executor, int timeout) {
    if (!(executor instanceof ExecutorService) || isTerminated(executor)) {
        return;
    }
    final ExecutorService es = (ExecutorService) executor;
    try {
        // Disable new tasks from being submitted
        // 停止接收新的任务并且等待已经提交的任务（包含提交正在执行和提交未执行）执行完成
        // 当所有提交任务执行完毕，线程池即被关闭
        es.shutdown();
    } catch (SecurityException ex2) {
        return;
    } catch (NullPointerException ex2) {
        return;
    }
    try {
        // Wait a while for existing tasks to terminate
        // 当等待超过设定时间时，会监测ExecutorService是否已经关闭，如果没关闭，再关闭一次
        if (!es.awaitTermination(timeout, TimeUnit.MILLISECONDS)) {
            // 试图停止所有正在执行的线程，不再处理还在池队列中等待的任务
            // ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。
            es.shutdownNow();
        }
    } catch (InterruptedException ex) {
        es.shutdownNow();
        Thread.currentThread().interrupt();
    }
    if (!isTerminated(es)) {
        newThreadToCloseExecutor(es);
    }
}
```

可以看到这个销毁任务调度器，也就是退出线程池，调用了shutdown、shutdownNow方法，这里也替大家恶补了一下线程池关闭的方法区别。

##### 6.doRegister && doUnregister && doSubscribe && doUnsubscribe && lookup

关于这个五个方法都是调用了RegistryService的方法，读者可自主查看[《dubbo源码解析（三）注册中心——开篇》](https://segmentfault.com/a/1190000016905715)来理解内部实现。

#### （二）DubboRegistryFactory

该类继承了AbstractRegistryFactory类，实现了AbstractRegistryFactory抽象出来的createRegistry方法，是dubbo这种实现的注册中心的工厂类，里面做了一些初始化的处理，以及创建注册中心DubboRegistry的对象实例。因为该类的属性比较好理解，所以下面就不在展开讲解了。

##### 1.getRegistryURL

获取注册中心url，类似于初始化注册中心url的方法。

```
private static URL getRegistryURL(URL url) {
    return url.setPath(RegistryService.class.getName())
            // 移除暴露服务和引用服务的参数
            .removeParameter(Constants.EXPORT_KEY).removeParameter(Constants.REFER_KEY)
            // 添加注册中心服务接口class值
            .addParameter(Constants.INTERFACE_KEY, RegistryService.class.getName())
            // 启用sticky 粘性连接，让客户端总是连接同一提供者
            .addParameter(Constants.CLUSTER_STICKY_KEY, "true")
            // 决定在创建客户端时建立连接
            .addParameter(Constants.LAZY_CONNECT_KEY, "true")
            // 不重连
            .addParameter(Constants.RECONNECT_KEY, "false")
            // 方法调用超时时间为10s
            .addParameterIfAbsent(Constants.TIMEOUT_KEY, "10000")
            // 每个客户端上一个接口的回调服务实例的限制为10000个
            .addParameterIfAbsent(Constants.CALLBACK_INSTANCES_LIMIT_KEY, "10000")
            // 注册中心连接超时时间10s
            .addParameterIfAbsent(Constants.CONNECT_TIMEOUT_KEY, "10000")
            // 添加方法级配置
            .addParameter(Constants.METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(Wrapper.getWrapper(RegistryService.class).getDeclaredMethodNames())), ","))
            //.addParameter(Constants.STUB_KEY, RegistryServiceStub.class.getName())
            //.addParameter(Constants.STUB_EVENT_KEY, Boolean.TRUE.toString()) //for event dispatch
            //.addParameter(Constants.ON_DISCONNECT_KEY, "disconnect")
            .addParameter("subscribe.1.callback", "true")
            .addParameter("unsubscribe.1.callback", "false");
}
```

看上面的源码可以很直白的看书就是对url中的配置做一些初始化设置。几乎每个key对应的意义我都在上面展示了，会比较好理解。

##### 2.createRegistry

该方法就是实现了AbstractRegistryFactory抽象出来的createRegistry方法，该子类就只关注createRegistry方法，其他公共的逻辑都在AbstractRegistryFactory已经实现。看一下源码：

```
@Override
public Registry createRegistry(URL url) {
    // 类似于初始化注册中心
    url = getRegistryURL(url);
    List<URL> urls = new ArrayList<URL>();
    // 移除备用的值
    urls.add(url.removeParameter(Constants.BACKUP_KEY));
    String backup = url.getParameter(Constants.BACKUP_KEY);
    if (backup != null && backup.length() > 0) {
        // 分割备用地址
        String[] addresses = Constants.COMMA_SPLIT_PATTERN.split(backup);
        for (String address : addresses) {
            urls.add(url.setAddress(address));
        }
    }
    // 创建RegistryDirectory，里面有多个Registry的Invoker
    RegistryDirectory<RegistryService> directory = new RegistryDirectory<RegistryService>(RegistryService.class, url.addParameter(Constants.INTERFACE_KEY, RegistryService.class.getName()).addParameterAndEncoded(Constants.REFER_KEY, url.toParameterString()));
    // 将directory中的多个Invoker伪装成一个Invoker
    Invoker<RegistryService> registryInvoker = cluster.join(directory);
    // 代理
    RegistryService registryService = proxyFactory.getProxy(registryInvoker);
    // 创建注册中心对象
    DubboRegistry registry = new DubboRegistry(registryInvoker, registryService);
    directory.setRegistry(registry);
    directory.setProtocol(protocol);
    // 通知监听器
    directory.notify(urls);
    // 订阅
    directory.subscribe(new URL(Constants.CONSUMER_PROTOCOL, NetUtils.getLocalHost(), 0, RegistryService.class.getName(), url.getParameters()));
    return registry;
}
```

在这个方法中做了实例化了DubboRegistry，并且做了通知和订阅的操作。相关集群、代理以及包含了很多Invoker的Directory我会在后续文章中讲到。

# 注册中心——multicast

> 目标：解释以为multicast实现的注册中心原理，理解单播、广播、多播区别，解读duubo-registry-multicast的源码

这是dubbo实现注册中心的第二种方式，也是dubbo的demo模块中用的注册中心实现方式。multicast其实是用到了MulticastSocket来实现的。

我这边稍微补充一点关于多点广播，也就是MulticastSocket的介绍。MulticastSocket类是继承了DatagramSocket类，DatagramSocket只允许把数据报发送给一个指定的目标地址，而MulticastSocket可以将数据报以广播的形式发送给多个客户端。它的思想是MulticastSocket会把一个数据报发送给一个特定的多点广播地址，这个多点广播地址是一组特殊的网络地址，当客户端需要发送或者接收广播信息时，只要加入该组就好。IP协议为多点广播提供了一批特殊的IP地址，地址范围是224.0.0.0至239.255.255.255。MulticastSocket类既可以将数据报发送到多点广播地址，也可以接收其他主机的广播信息。

以上是对multicast背景的简略介绍，接下来让我们具体的来看dubbo怎么把MulticastSocket运用到注册中心的实现中。

可以看到跟默认的注册中心的包结构非常类似。接下来我们就来解读一下这两个类。

## （一）MulticastRegistry

该类继承了FailbackRegistry类，该类就是针对注册中心核心的功能注册、订阅、取消注册、取消订阅，查询注册列表进行展开，利用广播的方式去实现。

### 1.属性

```
// logging output
// 日志记录输出
private static final Logger logger = LoggerFactory.getLogger(MulticastRegistry.class);

// 默认的多点广播端口
private static final int DEFAULT_MULTICAST_PORT = 1234;

// 多点广播的地址
private final InetAddress mutilcastAddress;

// 多点广播
private final MulticastSocket mutilcastSocket;

// 多点广播端口
private final int mutilcastPort;

//收到的URL
private final ConcurrentMap<URL, Set<URL>> received = new ConcurrentHashMap<URL, Set<URL>>();

// 任务调度器
private final ScheduledExecutorService cleanExecutor = Executors.newScheduledThreadPool(1, new NamedThreadFactory("DubboMulticastRegistryCleanTimer", true));

// 定时清理执行器，一定时间清理过期的url
private final ScheduledFuture<?> cleanFuture;

// 清理的间隔时间
private final int cleanPeriod;

// 管理员权限
private volatile boolean admin = false;
```

看上面的属性，需要关注以下几个点：

1. mutilcastSocket，该类是muticast注册中心实现的关键，这里补充一下单播、广播、以及多播的区别，因为下面会涉及到。单播是每次只有两个实体相互通信，发送端和接收端都是唯一确定的；广播目的地址为网络中的全体目标，而多播的目的地址是一组目标，加入该组的成员均是数据包的目的地。
2. 关注任务调度器和清理计时器，该类封装了定时清理过期的服务的策略。

### 2.构造方法

```
public MulticastRegistry(URL url) {
    super(url);
    if (url.isAnyHost()) {
        throw new IllegalStateException("registry address == null");
    }
    if (!isMulticastAddress(url.getHost())) {
        throw new IllegalArgumentException("Invalid multicast address " + url.getHost() + ", scope: 224.0.0.0 - 239.255.255.255");
    }
    try {
        mutilcastAddress = InetAddress.getByName(url.getHost());
        // 如果url携带的配置中没有端口号，则使用默认端口号
        mutilcastPort = url.getPort() <= 0 ? DEFAULT_MULTICAST_PORT : url.getPort();
        mutilcastSocket = new MulticastSocket(mutilcastPort);
        // 禁用多播数据报的本地环回
        mutilcastSocket.setLoopbackMode(false);
        // 加入同一组广播
        mutilcastSocket.joinGroup(mutilcastAddress);
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                byte[] buf = new byte[2048];
                // 实例化数据报
                DatagramPacket recv = new DatagramPacket(buf, buf.length);
                while (!mutilcastSocket.isClosed()) {
                    try {
                        // 接收数据包
                        mutilcastSocket.receive(recv);
                        String msg = new String(recv.getData()).trim();
                        int i = msg.indexOf('\n');
                        if (i > 0) {
                            msg = msg.substring(0, i).trim();
                        }
                        // 接收消息请求，根据消息并相应操作，比如注册，订阅等
                        MulticastRegistry.this.receive(msg, (InetSocketAddress) recv.getSocketAddress());
                        Arrays.fill(buf, (byte) 0);
                    } catch (Throwable e) {
                        if (!mutilcastSocket.isClosed()) {
                            logger.error(e.getMessage(), e);
                        }
                    }
                }
            }
        }, "DubboMulticastRegistryReceiver");
        // 设置为守护进程
        thread.setDaemon(true);
        // 开启线程
        thread.start();
    } catch (IOException e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
    // 优先从url中获取清理延迟配置，若没有，则默认为60s
    this.cleanPeriod = url.getParameter(Constants.SESSION_TIMEOUT_KEY, Constants.DEFAULT_SESSION_TIMEOUT);
    // 如果配置了需要清理
    if (url.getParameter("clean", true)) {
        // 开启计时器
        this.cleanFuture = cleanExecutor.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                try {
                    // 清理过期的服务
                    clean(); // Remove the expired
                } catch (Throwable t) { // Defensive fault tolerance
                    logger.error("Unexpected exception occur at clean expired provider, cause: " + t.getMessage(), t);
                }
            }
        }, cleanPeriod, cleanPeriod, TimeUnit.MILLISECONDS);
    } else {
        this.cleanFuture = null;
    }
}
```

这个构造器最关键的就是一个线程和一个定时清理任务。

1. 线程中做的工作是根据接收到的消息来判定是什么请求，作出对应的操作，只要mutilcastSocket没有断开，就一直接收消息，内部的实现体现在receive方法中，下文会展开讲述。
2. 定时清理任务是清理过期的注册的服务。通过两次socket的尝试来判定是否过期。clean方法下文会展开讲述

### 3.isMulticastAddress

```
private static boolean isMulticastAddress(String ip) {
    int i = ip.indexOf('.');
    if (i > 0) {
        String prefix = ip.substring(0, i);
        if (StringUtils.isInteger(prefix)) {
            int p = Integer.parseInt(prefix);
            return p >= 224 && p <= 239;
        }
    }
    return false;
}
```

该方法很简单，为也没写注释，就是判断是否为多点广播地址，地址范围是224.0.0.0至239.255.255.255。

### 4.clean

```
private void clean() {
    // 当url中携带的服务接口配置为是*时候，才可以执行清理
    if (admin) {
        for (Set<URL> providers : new HashSet<Set<URL>>(received.values())) {
            for (URL url : new HashSet<URL>(providers)) {
                // 判断是否过期
                if (isExpired(url)) {
                    if (logger.isWarnEnabled()) {
                        logger.warn("Clean expired provider " + url);
                    }
                    //取消注册
                    doUnregister(url);
                }
            }
        }
    }
}
```

该方法也比较简单，关机的是如何判断过期以及做的取消注册的操作。下面会展开讲解这几个方法。

### 5.isExpired

```
private boolean isExpired(URL url) {
    // 如果为非动态管理模式或者协议是consumer、route或者override，则没有过期
    if (!url.getParameter(Constants.DYNAMIC_KEY, true)
            || url.getPort() <= 0
            || Constants.CONSUMER_PROTOCOL.equals(url.getProtocol())
            || Constants.ROUTE_PROTOCOL.equals(url.getProtocol())
            || Constants.OVERRIDE_PROTOCOL.equals(url.getProtocol())) {
        return false;
    }
    Socket socket = null;
    try {
        // 利用url携带的主机地址和端口号实例化socket
        socket = new Socket(url.getHost(), url.getPort());
    } catch (Throwable e) {
        // 如果实例化失败，等待100ms重试第二次，如果还失败，则判定已过期
        try {
            // 等待100ms
            Thread.sleep(100);
        } catch (Throwable e2) {
        }
        Socket socket2 = null;
        try {
            socket2 = new Socket(url.getHost(), url.getPort());
        } catch (Throwable e2) {
            return true;
        } finally {
            if (socket2 != null) {
                try {
                    socket2.close();
                } catch (Throwable e2) {
                }
            }
        }
    } finally {
        if (socket != null) {
            try {
                socket.close();
            } catch (Throwable e) {
            }
        }
    }
    return false;
}
```

这个方法就是判断服务是否过期，有两次尝试socket的操作，如果尝试失败，则判断为过期。

### 6.receive

```
private void receive(String msg, InetSocketAddress remoteAddress) {
    if (logger.isInfoEnabled()) {
        logger.info("Receive multicast message: " + msg + " from " + remoteAddress);
    }
    // 如果这个消息是以register、unregister、subscribe开头的，则进行相应的操作
    if (msg.startsWith(Constants.REGISTER)) {
        URL url = URL.valueOf(msg.substring(Constants.REGISTER.length()).trim());
        // 注册服务
        registered(url);
    } else if (msg.startsWith(Constants.UNREGISTER)) {
        URL url = URL.valueOf(msg.substring(Constants.UNREGISTER.length()).trim());
        // 取消注册服务
        unregistered(url);
    } else if (msg.startsWith(Constants.SUBSCRIBE)) {
        URL url = URL.valueOf(msg.substring(Constants.SUBSCRIBE.length()).trim());
        // 获得以及注册的url集合
        Set<URL> urls = getRegistered();
        if (urls != null && !urls.isEmpty()) {
            for (URL u : urls) {
                // 判断是否合法
                if (UrlUtils.isMatch(url, u)) {
                    String host = remoteAddress != null && remoteAddress.getAddress() != null
                            ? remoteAddress.getAddress().getHostAddress() : url.getIp();
                    // 建议服务提供者和服务消费者在不同机器上运行，如果在同一机器上，需设置unicast=false
                    // 同一台机器中的多个进程不能单播单播，或者只有一个进程接收信息，发给消费者的单播消息可能被提供者抢占，两个消费者在同一台机器也一样，
                    // 只有multicast注册中心有此问题
                    if (url.getParameter("unicast", true) // Whether the consumer's machine has only one process
                            && !NetUtils.getLocalHost().equals(host)) { // Multiple processes in the same machine cannot be unicast with unicast or there will be only one process receiving information
                        unicast(Constants.REGISTER + " " + u.toFullString(), host);
                    } else {
                        broadcast(Constants.REGISTER + " " + u.toFullString());
                    }
                }
            }
        }
    }/* else if (msg.startsWith(UNSUBSCRIBE)) {
    }*/
}
```

可以很清楚的看到，根据接收到的消息开头的数据来判断需要做什么类型的操作，重点在于订阅，可以选择单播订阅还是广播订阅，这个取决于url携带的配置是什么。

### 7.broadcast

```
private void broadcast(String msg) {
    if (logger.isInfoEnabled()) {
        logger.info("Send broadcast message: " + msg + " to " + mutilcastAddress + ":" + mutilcastPort);
    }
    try {
        byte[] data = (msg + "\n").getBytes();
        // 实例化数据报,重点是目的地址是mutilcastAddress
        DatagramPacket hi = new DatagramPacket(data, data.length, mutilcastAddress, mutilcastPort);
        // 发送数据报
        mutilcastSocket.send(hi);
    } catch (Exception e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

这是广播的实现方法，重点是数据报的目的地址是mutilcastAddress。代表着一组地址

### 8.unicast

```
private void unicast(String msg, String host) {
    if (logger.isInfoEnabled()) {
        logger.info("Send unicast message: " + msg + " to " + host + ":" + mutilcastPort);
    }
    try {
        byte[] data = (msg + "\n").getBytes();
        // 实例化数据报,重点是目的地址是只是单个地址
        DatagramPacket hi = new DatagramPacket(data, data.length, InetAddress.getByName(host), mutilcastPort);
        // 发送数据报
        mutilcastSocket.send(hi);
    } catch (Exception e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

这是单播的实现，跟广播的区别就只是目的地址不一样，单播的目的地址就只是一个地址，而广播的是一组地址。

### 9.doRegister && doUnregister && doSubscribe && doUnsubscribe

```
@Override
protected void doRegister(URL url) {
    broadcast(Constants.REGISTER + " " + url.toFullString());
}
@Override
protected void doUnregister(URL url) {
    broadcast(Constants.UNREGISTER + " " + url.toFullString());
}
@Override
protected void doSubscribe(URL url, NotifyListener listener) {
    // 当url中携带的服务接口配置为是*时候，才可以执行清理，类似管理员权限
    if (Constants.ANY_VALUE.equals(url.getServiceInterface())) {
        admin = true;
    }
    broadcast(Constants.SUBSCRIBE + " " + url.toFullString());
    // 对监听器进行同步锁
    synchronized (listener) {
        try {
            listener.wait(url.getParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT));
        } catch (InterruptedException e) {
        }
    }
}
@Override
protected void doUnsubscribe(URL url, NotifyListener listener) {
    if (!Constants.ANY_VALUE.equals(url.getServiceInterface())
            && url.getParameter(Constants.REGISTER_KEY, true)) {
        unregister(url);
    }
    broadcast(Constants.UNSUBSCRIBE + " " + url.toFullString());
}
```

这几个方法就是实现了父类FailbackRegistry的抽象方法。都是调用了broadcast方法。

### 10.destroy

```
@Override
public void destroy() {
    super.destroy();
    try {
        // 取消清理任务
        if (cleanFuture != null) {
            cleanFuture.cancel(true);
        }
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
    try {
        // 把该地址从组内移除
        mutilcastSocket.leaveGroup(mutilcastAddress);
        // 关闭mutilcastSocket
        mutilcastSocket.close();
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
    // 关闭线程池
    ExecutorUtil.gracefulShutdown(cleanExecutor, cleanPeriod);
}
```

该方法的逻辑跟dubbo注册中心的destroy方法类似，就多了把该地址从组内移除的操作。gracefulShutdown方法我在[《dubbo源码解析（四）注册中心——dubbo》](https://segmentfault.com/a/1190000016921721)中已经讲到。

### 11.register

```
@Override
public void register(URL url) {
    super.register(url);
    registered(url);
}
protected void registered(URL url) {
    // 遍历订阅的监听器集合
    for (Map.Entry<URL, Set<NotifyListener>> entry : getSubscribed().entrySet()) {
        URL key = entry.getKey();
        // 判断是否合法
        if (UrlUtils.isMatch(key, url)) {
            // 通过消费者url获得接收到的服务url集合
            Set<URL> urls = received.get(key);
            if (urls == null) {
                received.putIfAbsent(key, new ConcurrentHashSet<URL>());
                urls = received.get(key);
            }
            // 加入服务url
            urls.add(url);
            List<URL> list = toList(urls);
            for (NotifyListener listener : entry.getValue()) {
                // 把服务url的变化通知监听器
                notify(key, listener, list);
                synchronized (listener) {
                    listener.notify();
                }
            }
        }
    }
}
```

可以看到该类重写了父类的register方法，不过逻辑没有过多的变化，就是把需要注册的url放入缓存中，如果通知监听器url的变化。

### 12.unregister

```
@Override
public void unregister(URL url) {
    super.unregister(url);
    unregistered(url);
}
protected void unregistered(URL url) {
    // 遍历订阅的监听器集合
    for (Map.Entry<URL, Set<NotifyListener>> entry : getSubscribed().entrySet()) {
        URL key = entry.getKey();
        if (UrlUtils.isMatch(key, url)) {
            Set<URL> urls = received.get(key);
            // 缓存中移除
            if (urls != null) {
                urls.remove(url);
            }
            if (urls == null || urls.isEmpty()){
                if (urls == null){
                    urls = new ConcurrentHashSet<URL>();
                }
                // 设置携带empty协议的url
                URL empty = url.setProtocol(Constants.EMPTY_PROTOCOL);
                urls.add(empty);
            }
            List<URL> list = toList(urls);
            // 通知监听器 服务url变化
            for (NotifyListener listener : entry.getValue()) {
                notify(key, listener, list);
            }
        }
    }
}
```

这个逻辑也比较清晰，把需要取消注册的服务url从缓存中移除，然后如果没有接收的服务url了，就加入一个携带empty协议的url，然后通知监听器服务变化。

### 13.lookup

```
@Override
public List<URL> lookup(URL url) {
    List<URL> urls = new ArrayList<URL>();
    // 通过消费者url获得订阅的服务的监听器
    Map<String, List<URL>> notifiedUrls = getNotified().get(url);
    // 获得注册的服务url集合
    if (notifiedUrls != null && notifiedUrls.size() > 0) {
        for (List<URL> values : notifiedUrls.values()) {
            urls.addAll(values);
        }
    }
    // 如果为空，则从内存缓存properties获得相关value，并且返回为注册的服务
    if (urls.isEmpty()) {
        List<URL> cacheUrls = getCacheUrls(url);
        if (cacheUrls != null && !cacheUrls.isEmpty()) {
            urls.addAll(cacheUrls);
        }
    }
    // 如果还是为空则从缓存registered中获得已注册 服务URL 集合
    if (urls.isEmpty()) {
        for (URL u : getRegistered()) {
            if (UrlUtils.isMatch(url, u)) {
                urls.add(u);
            }
        }
    }
    // 如果url携带的配置服务接口为*，也就是所有服务，则从缓存subscribed获得已注册 服务URL 集合
    if (Constants.ANY_VALUE.equals(url.getServiceInterface())) {
        for (URL u : getSubscribed().keySet()) {
            if (UrlUtils.isMatch(url, u)) {
                urls.add(u);
            }
        }
    }
    return urls;
}
```

该方法是返回注册的服务url列表，可以看到有很多种获得的方法这些缓存都保存在AbstractRegistry类中，相关的介绍可以查看[《dubbo源码解析（三）注册中心——开篇》](https://segmentfault.com/a/1190000016905715)。

### 14.subscribe && unsubscribe

```
@Override
public void subscribe(URL url, NotifyListener listener) {
    super.subscribe(url, listener);
    subscribed(url, listener);
}

@Override
public void unsubscribe(URL url, NotifyListener listener) {
    super.unsubscribe(url, listener);
    received.remove(url);
}
protected void subscribed(URL url, NotifyListener listener) {
    // 查询注册列表
    List<URL> urls = lookup(url);
    // 通知url
    notify(url, listener, urls);
}
```

这两个重写了父类的方法，分别是订阅和取消订阅。逻辑很简单。

## （二）MulticastRegistryFactory

该类继承了AbstractRegistryFactory类，实现了AbstractRegistryFactory抽象出来的createRegistry方法，看一下原代码：

```
public class MulticastRegistryFactory extends AbstractRegistryFactory {

    @Override
    public Registry createRegistry(URL url) {
        return new MulticastRegistry(url);
    }

}
```

可以看到就是实例化了MulticastRegistry而已，所有这里就不解释了。

# 注册中心——redis

# 注册中心——redis

> 目标：解释以为redis实现的注册中心原理，解读duubo-registry-redis的源码

Redis是一个key-value存储系统，交换数据非常快，redis以内存作为数据存储的介质，所以读写数据的效率极高，远远超过数据库。redis支持丰富的数据类型，dubbo就利用了redis的value支持map的数据类型。redis的key为服务名称和服务的类型。map中的key为URL地址，map中的value为过期时间，用于判断脏数据，脏数据由监控中心删除。

dubbo利用JRedis来连接到Redis分布式哈希键-值数据库，因为Jedis实例不是线程安全的,所以不可以多个线程共用一个Jedis实例，但是创建太多的实现也不好因为这意味着会建立很多sokcet连接。 所以dubbo又用了JedisPool，JedisPool是一个线程安全的网络连接池。可以用JedisPool创建一些可靠Jedis实例，可以从池中获取Jedis实例，使用完后再把Jedis实例还回JedisPool。这种方式可以避免创建大量socket连接并且会实现高效的性能。

包结构非常类似。接下来我们就来解读一下这两个类。

## （一）RedisRegistry

该类继承了FailbackRegistry类，该类就是针对注册中心核心的功能注册、订阅、取消注册、取消订阅，查询注册列表进行展开，基于redis来实现。

### 1.属性

```
// 日志记录
private static final Logger logger = LoggerFactory.getLogger(RedisRegistry.class);

// 默认的redis连接端口
private static final int DEFAULT_REDIS_PORT = 6379;

// 默认 Redis 根节点，涉及到的是dubbo的分组配置
private final static String DEFAULT_ROOT = "dubbo";

// 任务调度器
private final ScheduledExecutorService expireExecutor = Executors.newScheduledThreadPool(1, new NamedThreadFactory("DubboRegistryExpireTimer", true));

//Redis Key 过期机制执行器
private final ScheduledFuture<?> expireFuture;

// Redis 根节点
private final String root;

// JedisPool集合，map 的key为 "ip:port"的形式
private final Map<String, JedisPool> jedisPools = new ConcurrentHashMap<String, JedisPool>();

// 通知器集合，key为 Root + Service的形式
// 例如 /dubbo/com.alibaba.dubbo.demo.DemoService
private final ConcurrentMap<String, Notifier> notifiers = new ConcurrentHashMap<String, Notifier>();

// 重连时间间隔，单位：ms
private final int reconnectPeriod;

// 过期周期，单位：ms
private final int expirePeriod;

// 是否通过监控中心，用于判断脏数据，脏数据由监控中心删除
private volatile boolean admin = false;

// 是否复制模式
private boolean replicate;
```

可以从属性中看到基于redis的注册中心可以被监控中心监控，并且对过期的节点有清理的机制。

### 2.构造方法

```
public RedisRegistry(URL url) {
    super(url);
    // 判断地址是否为空
    if (url.isAnyHost()) {
        throw new IllegalStateException("registry address == null");
    }
    // 实例化对象池
    GenericObjectPoolConfig config = new GenericObjectPoolConfig();
    // 如果 testOnBorrow 被设置，pool 会在 borrowObject 返回对象之前使用 PoolableObjectFactory的 validateObject 来验证这个对象是否有效
    // 要是对象没通过验证，这个对象会被丢弃，然后重新选择一个新的对象。
    config.setTestOnBorrow(url.getParameter("test.on.borrow", true));
    // 如果 testOnReturn 被设置， pool 会在 returnObject 的时候通过 PoolableObjectFactory 的validateObject 方法验证对象
    // 如果对象没通过验证，对象会被丢弃，不会被放到池中。
    config.setTestOnReturn(url.getParameter("test.on.return", false));
    // 指定空闲对象是否应该使用 PoolableObjectFactory 的 validateObject 校验，如果校验失败，这个对象会从对象池中被清除。
    // 这个设置仅在 timeBetweenEvictionRunsMillis 被设置成正值（ >0） 的时候才会生效。
    config.setTestWhileIdle(url.getParameter("test.while.idle", false));
    if (url.getParameter("max.idle", 0) > 0)
        // 控制一个pool最多有多少个状态为空闲的jedis实例。
        config.setMaxIdle(url.getParameter("max.idle", 0));
    if (url.getParameter("min.idle", 0) > 0)
        // 控制一个pool最少有多少个状态为空闲的jedis实例。
        config.setMinIdle(url.getParameter("min.idle", 0));
    if (url.getParameter("max.active", 0) > 0)
        // 控制一个pool最多有多少个jedis实例。
        config.setMaxTotal(url.getParameter("max.active", 0));
    if (url.getParameter("max.total", 0) > 0)
        config.setMaxTotal(url.getParameter("max.total", 0));
    if (url.getParameter("max.wait", url.getParameter("timeout", 0)) > 0)
        //表示当引入一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛出JedisConnectionException；
        config.setMaxWaitMillis(url.getParameter("max.wait", url.getParameter("timeout", 0)));
    if (url.getParameter("num.tests.per.eviction.run", 0) > 0)
        // 设置驱逐线程每次检测对象的数量。这个设置仅在 timeBetweenEvictionRunsMillis 被设置成正值（ >0）的时候才会生效。
        config.setNumTestsPerEvictionRun(url.getParameter("num.tests.per.eviction.run", 0));
    if (url.getParameter("time.between.eviction.runs.millis", 0) > 0)
        // 指定驱逐线程的休眠时间。如果这个值不是正数（ >0），不会有驱逐线程运行。
        config.setTimeBetweenEvictionRunsMillis(url.getParameter("time.between.eviction.runs.millis", 0));
    if (url.getParameter("min.evictable.idle.time.millis", 0) > 0)
        // 指定最小的空闲驱逐的时间间隔（空闲超过指定的时间的对象，会被清除掉）。
        // 这个设置仅在 timeBetweenEvictionRunsMillis 被设置成正值（ >0）的时候才会生效。
        config.setMinEvictableIdleTimeMillis(url.getParameter("min.evictable.idle.time.millis", 0));

    // 获取url中的集群配置
    String cluster = url.getParameter("cluster", "failover");
    if (!"failover".equals(cluster) && !"replicate".equals(cluster)) {
        throw new IllegalArgumentException("Unsupported redis cluster: " + cluster + ". The redis cluster only supported failover or replicate.");
    }
    // 设置是否为复制模式
    replicate = "replicate".equals(cluster);

    List<String> addresses = new ArrayList<String>();
    addresses.add(url.getAddress());
    // 备用地址
    String[] backups = url.getParameter(Constants.BACKUP_KEY, new String[0]);
    if (backups != null && backups.length > 0) {
        addresses.addAll(Arrays.asList(backups));
    }

    for (String address : addresses) {
        int i = address.indexOf(':');
        String host;
        int port;
        // 分割地址和端口号
        if (i > 0) {
            host = address.substring(0, i);
            port = Integer.parseInt(address.substring(i + 1));
        } else {
            // 没有端口的设置默认端口
            host = address;
            port = DEFAULT_REDIS_PORT;
        }
        // 创建连接池并加入集合
        this.jedisPools.put(address, new JedisPool(config, host, port,
                url.getParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT), StringUtils.isEmpty(url.getPassword()) ? null : url.getPassword(),
                url.getParameter("db.index", 0)));
    }

    // 设置url携带的连接超时时间，如果没有配置，则设置默认为3s
    this.reconnectPeriod = url.getParameter(Constants.REGISTRY_RECONNECT_PERIOD_KEY, Constants.DEFAULT_REGISTRY_RECONNECT_PERIOD);
    // 获取url中的分组配置，如果没有配置，则默认为dubbo
    String group = url.getParameter(Constants.GROUP_KEY, DEFAULT_ROOT);
    if (!group.startsWith(Constants.PATH_SEPARATOR)) {
        group = Constants.PATH_SEPARATOR + group;
    }
    if (!group.endsWith(Constants.PATH_SEPARATOR)) {
        group = group + Constants.PATH_SEPARATOR;
    }
    // 设置redis 的根节点
    this.root = group;

    // 获取过期周期配置，如果没有，则默认为60s
    this.expirePeriod = url.getParameter(Constants.SESSION_TIMEOUT_KEY, Constants.DEFAULT_SESSION_TIMEOUT);
    // 创建过期机制执行器
    this.expireFuture = expireExecutor.scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            try {
                // 延长到期时间
                deferExpired(); // Extend the expiration time
            } catch (Throwable t) { // Defensive fault tolerance
                logger.error("Unexpected exception occur at defer expire time, cause: " + t.getMessage(), t);
            }
        }
    }, expirePeriod / 2, expirePeriod / 2, TimeUnit.MILLISECONDS);
}
```

构造方法首先是调用了父类的构造函数，然后是对对象池的一些配置进行了初始化，具体的我已经在注释中写明。在构造方法中还做了连接池的创建、过期机制执行器的创建，其中过期会进行延长到期时间的操作具体是在deferExpired方法中实现。还有一个关注点事该执行器的时间是取周期的一半。

### 3.deferExpired

```
private void deferExpired() {
    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
        JedisPool jedisPool = entry.getValue();
        try {
            // 获得连接池中的Jedis实例
            Jedis jedis = jedisPool.getResource();
            try {
                // 遍历已经注册的服务url集合
                for (URL url : new HashSet<URL>(getRegistered())) {
                    // 如果是非动态管理模式
                    if (url.getParameter(Constants.DYNAMIC_KEY, true)) {
                        // 获得分类路径
                        String key = toCategoryPath(url);
                        // 以hash 散列表的形式存储
                        if (jedis.hset(key, url.toFullString(), String.valueOf(System.currentTimeMillis() + expirePeriod)) == 1) {
                            // 发布 Redis 注册事件
                            jedis.publish(key, Constants.REGISTER);
                        }
                    }
                }
                // 如果通过监控中心
                if (admin) {
                    // 删除过时的脏数据
                    clean(jedis);
                }
                // 如果服务器端已同步数据，只需写入单台机器
                if (!replicate) {
                    break;//  If the server side has synchronized data, just write a single machine
                }
            } finally {
                jedis.close();
            }
        } catch (Throwable t) {
            logger.warn("Failed to write provider heartbeat to redis registry. registry: " + entry.getKey() + ", cause: " + t.getMessage(), t);
        }
    }
}
```

该方法实现了延长到期时间的逻辑，遍历了已经注册的服务url，这里会有一个是否为非动态管理模式的判断，也就是判断该节点是否为动态节点，只有动态节点是需要延长过期时间，因为动态节点需要人工删除节点。延长过期时间就是重新注册一次。而其他的节点则会被监控中心清除，也就是调用了clean方法。clean方法下面会讲到。

### 4.clean

```
// The monitoring center is responsible for deleting outdated dirty data
private void clean(Jedis jedis) {
    // 获得所有的服务
    Set<String> keys = jedis.keys(root + Constants.ANY_VALUE);
    if (keys != null && !keys.isEmpty()) {
        // 遍历所有的服务
        for (String key : keys) {
            // 返回hash表key对应的所有域和值
            // redis的key为服务名称和服务的类型。map中的key为URL地址，map中的value为过期时间，用于判断脏数据，脏数据由监控中心删除
            Map<String, String> values = jedis.hgetAll(key);
            if (values != null && values.size() > 0) {
                boolean delete = false;
                long now = System.currentTimeMillis();
                for (Map.Entry<String, String> entry : values.entrySet()) {
                    URL url = URL.valueOf(entry.getKey());
                    // 是否为动态节点
                    if (url.getParameter(Constants.DYNAMIC_KEY, true)) {
                        long expire = Long.parseLong(entry.getValue());
                        // 判断是否过期
                        if (expire < now) {
                            // 删除记录
                            jedis.hdel(key, entry.getKey());
                            delete = true;
                            if (logger.isWarnEnabled()) {
                                logger.warn("Delete expired key: " + key + " -> value: " + entry.getKey() + ", expire: " + new Date(expire) + ", now: " + new Date(now));
                            }
                        }
                    }
                }
                // 取消注册
                if (delete) {
                    jedis.publish(key, Constants.UNREGISTER);
                }
            }
        }
    }
}
```

该方法就是用来清理过期数据的，之前我提到过dubbo在redis存储数据的数据结构形式，就是redis的key为服务名称和服务的类型。map中的key为URL地址，map中的value为过期时间，用于判断脏数据，脏数据由监控中心删除，那么判断过期就是通过map中的value来判别。逻辑就是在redis中先把记录删除，然后在取消订阅。

### 5.isAvailable

```
@Override
public boolean isAvailable() {
    // 遍历连接池集合
    for (JedisPool jedisPool : jedisPools.values()) {
        try {
            // 从连接池中获得jedis实例
            Jedis jedis = jedisPool.getResource();
            try {
                // 判断是否有redis服务器被连接着
                // 只要有一台连接，则算注册中心可用
                if (jedis.isConnected()) {
                    return true; // At least one single machine is available.
                }
            } finally {
                jedis.close();
            }
        } catch (Throwable t) {
        }
    }
    return false;
}
```

该方法是判断注册中心是否可用，通过redis是否连接来判断，只要有一台redis可连接，就算注册中心可用。

### 6.destroy

```
@Override
public void destroy() {
    super.destroy();
    try {
        // 关闭过期执行器
        expireFuture.cancel(true);
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
    try {
        // 关闭通知器
        for (Notifier notifier : notifiers.values()) {
            notifier.shutdown();
        }
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
        JedisPool jedisPool = entry.getValue();
        try {
            // 销毁连接池
            jedisPool.destroy();
        } catch (Throwable t) {
            logger.warn("Failed to destroy the redis registry client. registry: " + entry.getKey() + ", cause: " + t.getMessage(), t);
        }
    }
    // 关闭任务调度器
    ExecutorUtil.gracefulShutdown(expireExecutor, expirePeriod);
}
```

这是销毁的方法，逻辑很清晰，gracefulShutdown方法在[《dubbo源码解析（四）注册中心——dubbo》](https://segmentfault.com/a/1190000016921721)中已经讲到。

### 7.doRegister

```
@Override
public void doRegister(URL url) {
    // 获得分类路径
    String key = toCategoryPath(url);
    // 获得URL字符串作为 Value
    String value = url.toFullString();
    // 计算过期时间
    String expire = String.valueOf(System.currentTimeMillis() + expirePeriod);
    boolean success = false;
    RpcException exception = null;
    // 遍历连接池集合
    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
        JedisPool jedisPool = entry.getValue();
        try {
            Jedis jedis = jedisPool.getResource();
            try {
                // 写入 Redis Map 键
                jedis.hset(key, value, expire);
                // 发布 Redis 注册事件
                // 这样订阅该 Key 的服务消费者和监控中心，就会实时从 Redis 读取该服务的最新数据。
                jedis.publish(key, Constants.REGISTER);
                success = true;
                // 如果服务器端已同步数据，只需写入单台机器
                if (!replicate) {
                    break; //  If the server side has synchronized data, just write a single machine
                }
            } finally {
                jedis.close();
            }
        } catch (Throwable t) {
            exception = new RpcException("Failed to register service to redis registry. registry: " + entry.getKey() + ", service: " + url + ", cause: " + t.getMessage(), t);
        }
    }
    if (exception != null) {
        if (success) {
            logger.warn(exception.getMessage(), exception);
        } else {
            throw exception;
        }
    }
}
```

该方法是实现了父类FailbackRegistry的抽象方法，主要是实现了注册的功能，具体的逻辑是先将需要注册的服务信息保存到redis中，然后发布redis注册事件。

### 8.doUnregister

```
@Override
public void doUnregister(URL url) {
    // 获得分类路径
    String key = toCategoryPath(url);
    // 获得URL字符串作为 Value
    String value = url.toFullString();
    RpcException exception = null;
    boolean success = false;
    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
        JedisPool jedisPool = entry.getValue();
        try {
            Jedis jedis = jedisPool.getResource();
            try {
                // 删除redis中的记录
                jedis.hdel(key, value);
                // 发布redis取消注册事件
                jedis.publish(key, Constants.UNREGISTER);
                success = true;
                // 如果服务器端已同步数据，只需写入单台机器
                if (!replicate) {
                    break; //  If the server side has synchronized data, just write a single machine
                }
            } finally {
                jedis.close();
            }
        } catch (Throwable t) {
            exception = new RpcException("Failed to unregister service to redis registry. registry: " + entry.getKey() + ", service: " + url + ", cause: " + t.getMessage(), t);
        }
    }
    if (exception != null) {
        if (success) {
            logger.warn(exception.getMessage(), exception);
        } else {
            throw exception;
        }
    }
}
```

该方法也是实现了父类的抽象方法，当服务消费者或者提供者关闭时，会调用该方法来取消注册。逻辑就是跟注册方法方法，先从redis中删除服务相关记录，然后发布取消注册的事件，从而实时通知订阅者们。

### 9.doSubscribe

```
@Override
public void doSubscribe(final URL url, final NotifyListener listener) {
    // 返回服务地址
    String service = toServicePath(url);
    // 获得通知器
    Notifier notifier = notifiers.get(service);
    // 如果没有该服务的通知器，则创建一个
    if (notifier == null) {
        Notifier newNotifier = new Notifier(service);
        notifiers.putIfAbsent(service, newNotifier);
        notifier = notifiers.get(service);
        // 保证并发情况下，有且只有一个通知器启动
        if (notifier == newNotifier) {
            notifier.start();
        }
    }
    boolean success = false;
    RpcException exception = null;
    // 遍历连接池集合进行订阅，直到有一个订阅成功，仅仅向一个redis进行订阅
    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
        JedisPool jedisPool = entry.getValue();
        try {
            Jedis jedis = jedisPool.getResource();
            try {
                // 如果服务地址为*结尾，也就是处理所有的服务层发起的订阅
                if (service.endsWith(Constants.ANY_VALUE)) {
                    admin = true;
                    // 获得分类层的集合 例如：/dubbo/com.alibaba.dubbo.demo.DemoService/providers
                    Set<String> keys = jedis.keys(service);
                    if (keys != null && !keys.isEmpty()) {
                        // 按照服务聚合url
                        Map<String, Set<String>> serviceKeys = new HashMap<String, Set<String>>();
                        for (String key : keys) {
                            // 获得服务路径，截掉多余部分
                            String serviceKey = toServicePath(key);
                            Set<String> sk = serviceKeys.get(serviceKey);
                            if (sk == null) {
                                sk = new HashSet<String>();
                                serviceKeys.put(serviceKey, sk);
                            }
                            sk.add(key);
                        }
                        // 按照每个服务层进行发起通知，因为服务地址为*结尾
                        for (Set<String> sk : serviceKeys.values()) {
                            doNotify(jedis, sk, url, Arrays.asList(listener));
                        }
                    }
                } else {
                    // 处理指定的服务层发起的通知
                    doNotify(jedis, jedis.keys(service + Constants.PATH_SEPARATOR + Constants.ANY_VALUE), url, Arrays.asList(listener));
                }
                // 只在一个redis上进行订阅
                success = true;
                break; // Just read one server's data
            } finally {
                jedis.close();
            }
        } catch (Throwable t) { // Try the next server
            exception = new RpcException("Failed to subscribe service from redis registry. registry: " + entry.getKey() + ", service: " + url + ", cause: " + t.getMessage(), t);
        }
    }
    if (exception != null) {
        // 虽然发生异常，但结果仍然成功
        if (success) {
            logger.warn(exception.getMessage(), exception);
        } else {
            throw exception;
        }
    }
}
```

该方法是实现了订阅的功能。注意以下几个点：

1. 服务只会向一个redis进行订阅，只要有一个订阅成功就结束订阅。
2. 根据url携带的服务地址来调用doNotify的两个重载方法。其中一个只是遍历通知了所有服务的监听器，doNotify方法我会在后面讲到。

### 10.doUnsubscribe

```
@Override
public void doUnsubscribe(URL url, NotifyListener listener) {
}
```

该方法本来是取消订阅的实现，不过dubbo中并未实现该逻辑。

### 11.doNotify

```
private void doNotify(Jedis jedis, String key) {
    // 遍历所有的通知器，调用重载方法今天通知
    for (Map.Entry<URL, Set<NotifyListener>> entry : new HashMap<URL, Set<NotifyListener>>(getSubscribed()).entrySet()) {
        doNotify(jedis, Arrays.asList(key), entry.getKey(), new HashSet<NotifyListener>(entry.getValue()));
    }
}

private void doNotify(Jedis jedis, Collection<String> keys, URL url, Collection<NotifyListener> listeners) {
    if (keys == null || keys.isEmpty()
            || listeners == null || listeners.isEmpty()) {
        return;
    }
    long now = System.currentTimeMillis();
    List<URL> result = new ArrayList<URL>();
    // 获得分类集合
    List<String> categories = Arrays.asList(url.getParameter(Constants.CATEGORY_KEY, new String[0]));
    // 通过url获得服务接口
    String consumerService = url.getServiceInterface();
    // 遍历分类路径，例如/dubbo/com.alibaba.dubbo.demo.DemoService/providers
    for (String key : keys) {
        // 判断服务是否匹配
        if (!Constants.ANY_VALUE.equals(consumerService)) {
            String prvoiderService = toServiceName(key);
            if (!prvoiderService.equals(consumerService)) {
                continue;
            }
        }
        // 从分类路径上获得分类名
        String category = toCategoryName(key);
        // 判断订阅的分类是否包含该分类
        if (!categories.contains(Constants.ANY_VALUE) && !categories.contains(category)) {
            continue;
        }
        List<URL> urls = new ArrayList<URL>();
        // 返回所有的URL集合
        Map<String, String> values = jedis.hgetAll(key);
        if (values != null && values.size() > 0) {
            for (Map.Entry<String, String> entry : values.entrySet()) {
                URL u = URL.valueOf(entry.getKey());
                // 判断是否为动态节点，因为动态节点不受过期限制。并且判断是否过期
                if (!u.getParameter(Constants.DYNAMIC_KEY, true)
                        || Long.parseLong(entry.getValue()) >= now) {
                    // 判断url是否合法
                    if (UrlUtils.isMatch(url, u)) {
                        urls.add(u);
                    }
                }
            }
        }
        // 若不存在匹配的url，则创建 `empty://` 的 URL返回，用于清空该服务的该分类。
        if (urls.isEmpty()) {
            urls.add(url.setProtocol(Constants.EMPTY_PROTOCOL)
                    .setAddress(Constants.ANYHOST_VALUE)
                    .setPath(toServiceName(key))
                    .addParameter(Constants.CATEGORY_KEY, category));
        }
        result.addAll(urls);
        if (logger.isInfoEnabled()) {
            logger.info("redis notify: " + key + " = " + urls);
        }
    }
    if (result == null || result.isEmpty()) {
        return;
    }
    // 全部数据完成后，调用通知方法，来通知监听器
    for (NotifyListener listener : listeners) {
        notify(url, listener, result);
    }
}
```

该方法实现了通知的逻辑，有两个重载方法，第二个比第一个多了几个参数，其实唯一的区别就是第一个重载方法是通知了所有的监听器，内部逻辑中调用了getSubscribed方法获取所有的监听器，该方法的解释可以查看[《dubbo源码解析（三）注册中心——开篇》](https://segmentfault.com/a/1190000016905715)中关于subscribed属性的解释。而第二个重载方法就是对一个指定的监听器进行通知。

具体的逻辑在第二个重载的方法中，其中有以下几个需要注意的点：

1. 通知的事件要和监听器匹配。
2. 不同的角色会关注不同的分类，服务消费者会关注providers、configurations、routes这几个分类，而服务提供者会关注consumers分类，监控中心会关注所有分类。
3. 遍历分类路径，分类路径是Root + Service + Type。

### 12.toServiceName

```
private String toServiceName(String categoryPath) {
    String servicePath = toServicePath(categoryPath);
    return servicePath.startsWith(root) ? servicePath.substring(root.length()) : servicePath;
}
```

该方法很简单，就是从服务路径上获得服务名，这里就不多做解释了。

### 13.toCategoryName

```
private String toCategoryName(String categoryPath) {
    int i = categoryPath.lastIndexOf(Constants.PATH_SEPARATOR);
    return i > 0 ? categoryPath.substring(i + 1) : categoryPath;
}
```

该方法的作用是从分类路径上获得分类名。

### 14.toServicePath

```
private String toServicePath(String categoryPath) {
    int i;
    if (categoryPath.startsWith(root)) {
        i = categoryPath.indexOf(Constants.PATH_SEPARATOR, root.length());
    } else {
        i = categoryPath.indexOf(Constants.PATH_SEPARATOR);
    }
    return i > 0 ? categoryPath.substring(0, i) : categoryPath;
}

private String toServicePath(URL url) {
    return root + url.getServiceInterface();
}
```

这两个方法都是获得服务地址，第一个方法主要是截掉多余的部分，第二个方法主要是从url配置中获取关于服务地址的值跟根节点拼接。

### 15.toCategoryPath

```
private String toCategoryPath(URL url) {
    return toServicePath(url) + Constants.PATH_SEPARATOR + url.getParameter(Constants.CATEGORY_KEY, Constants.DEFAULT_CATEGORY);
}
```

该方法是获得分类路径，格式是Root + Service + Type。

### 16.内部类NotifySub

```
private class NotifySub extends JedisPubSub {

    private final JedisPool jedisPool;

    public NotifySub(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    @Override
    public void onMessage(String key, String msg) {
        if (logger.isInfoEnabled()) {
            logger.info("redis event: " + key + " = " + msg);
        }
        // 如果是注册事件或者取消注册事件
        if (msg.equals(Constants.REGISTER)
                || msg.equals(Constants.UNREGISTER)) {
            try {
                Jedis jedis = jedisPool.getResource();
                try {
                    // 通知监听器
                    doNotify(jedis, key);
                } finally {
                    jedis.close();
                }
            } catch (Throwable t) { // TODO Notification failure does not restore mechanism guarantee
                logger.error(t.getMessage(), t);
            }
        }
    }

    @Override
    public void onPMessage(String pattern, String key, String msg) {
        onMessage(key, msg);
    }

    @Override
    public void onSubscribe(String key, int num) {
    }

    @Override
    public void onPSubscribe(String pattern, int num) {
    }

    @Override
    public void onUnsubscribe(String key, int num) {
    }

    @Override
    public void onPUnsubscribe(String pattern, int num) {
    }

}
```

NotifySub是RedisRegistry的一个内部类，继承了JedisPubSub类，JedisPubSub类中定义了publish/subsribe的回调方法。通过继承JedisPubSub类并重新实现这些回调方法，当publish/subsribe事件发生时，我们可以定制自己的处理逻辑。这里实现了onMessage和onPMessage两个方法，当收到注册和取消注册的事件的时候通知相关的监听器数据变化，从而实现实时更新数据。

### 17.内部类Notifier

该类继承 Thread 类，负责向 Redis 发起订阅逻辑。

#### 1.属性

```
// 服务名：Root + Service
private final String service;
// 需要忽略连接的次数
private final AtomicInteger connectSkip = new AtomicInteger();
// 已经忽略连接的次数
private final AtomicInteger connectSkiped = new AtomicInteger();
// 随机数
private final Random random = new Random();
// jedis实例
private volatile Jedis jedis;
// 是否是首次通知
private volatile boolean first = true;
// 是否运行中
private volatile boolean running = true;
// 连接次数随机数
private volatile int connectRandom;
```

上述属性中，部分属性都是为了redis的重连策略，用于在和redis断开链接时，忽略一定的次数和redis的连接，避免空跑。

#### 2.resetSkip

```
private void resetSkip() {
    connectSkip.set(0);
    connectSkiped.set(0);
    connectRandom = 0;
}
```

该方法就是重置忽略连接的信息。

#### 3.isSkip

```
private boolean isSkip() {
    // 获得忽略次数
    int skip = connectSkip.get(); // Growth of skipping times
    // 如果忽略次数超过10次，那么取随机数，加上一个10以内的随机数
    // 连接失败的次数越多，每一轮加大需要忽略的总次数，并且带有一定的随机性。
    if (skip >= 10) { // If the number of skipping times increases by more than 10, take the random number
        if (connectRandom == 0) {
            connectRandom = random.nextInt(10);
        }
        skip = 10 + connectRandom;
    }
    // 自增忽略次数。若忽略次数不够，则继续忽略。
    if (connectSkiped.getAndIncrement() < skip) { // Check the number of skipping times
        return true;
    }
    // 增加需要忽略的次数
    connectSkip.incrementAndGet();
    // 重置已忽略次数和随机数
    connectSkiped.set(0);
    connectRandom = 0;
    return false;
}
```

该方法是用来判断忽略本次对redis的连接。首先获得需要忽略的次数，如果忽略次数不小于10次，则加上一个10以内的随机数，然后判断自增的忽略次数，如果次数不够，则继续忽略，如果次数够了，增加需要忽略的次数，重置已经忽略的次数和随机数。主要的思想是连接失败的次数越多，每一轮加大需要忽略的总次数，并且带有一定的随机性。

#### 4.run

```
@Override
public void run() {
    // 当通知器正在运行中时
    while (running) {
        try {
            // 如果不忽略连接
            if (!isSkip()) {
                try {
                    for (Map.Entry<String, JedisPool> entry : jedisPools.entrySet()) {
                        JedisPool jedisPool = entry.getValue();
                        try {
                            jedis = jedisPool.getResource();
                            try {
                                // 是否为监控中心
                                if (service.endsWith(Constants.ANY_VALUE)) {
                                    // 如果不是第一次通知
                                    if (!first) {
                                        first = false;
                                        Set<String> keys = jedis.keys(service);
                                        if (keys != null && !keys.isEmpty()) {
                                            for (String s : keys) {
                                                // 通知
                                                doNotify(jedis, s);
                                            }
                                        }
                                        // 重置
                                        resetSkip();
                                    }
                                    // 批准订阅
                                    jedis.psubscribe(new NotifySub(jedisPool), service); // blocking
                                } else {
                                    // 如果不是监控中心，并且不是第一次通知
                                    if (!first) {
                                        first = false;
                                        // 单独通知一个服务
                                        doNotify(jedis, service);
                                        // 重置
                                        resetSkip();
                                    }
                                    // 批准订阅
                                    jedis.psubscribe(new NotifySub(jedisPool), service + Constants.PATH_SEPARATOR + Constants.ANY_VALUE); // blocking
                                }
                                break;
                            } finally {
                                jedis.close();
                            }
                        } catch (Throwable t) { // Retry another server
                            logger.warn("Failed to subscribe service from redis registry. registry: " + entry.getKey() + ", cause: " + t.getMessage(), t);
                            // If you only have a single redis, you need to take a rest to avoid overtaking a lot of CPU resources
                            // 发生异常，说明 Redis 连接断开了，需要等待reconnectPeriod时间
                            //通过这样的方式，避免执行，占用大量的 CPU 资源。
                            sleep(reconnectPeriod);
                        }
                    }
                } catch (Throwable t) {
                    logger.error(t.getMessage(), t);
                    sleep(reconnectPeriod);
                }
            }
        } catch (Throwable t) {
            logger.error(t.getMessage(), t);
        }
    }
}
```

该方法是线程的run方法，应该很熟悉，其中做了相关订阅的逻辑，其中根据redis的重连策略做了一些忽略连接的策略，也就是调用了上述讲解的isSkip方法，订阅就是调用了jedis.psubscribe方法，它是订阅给定模式相匹配的所有频道。

#### 4.shutdown

```
public void shutdown() {
    try {
        // 更改状态
        running = false;
        // jedis断开连接
        jedis.disconnect();
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
}
```

该方法是断开连接的方法。

## （二）RedisRegistryFactory

该类继承了AbstractRegistryFactory类，实现了AbstractRegistryFactory抽象出来的createRegistry方法，看一下原代码：

```
public class RedisRegistryFactory extends AbstractRegistryFactory {

    @Override
    protected Registry createRegistry(URL url) {
        return new RedisRegistry(url);
    }

}
```

可以看到就是实例化了RedisRegistry而已，所有这里就不解释了。

# 注册中心——zookeeper

# 注册中心——zookeeper

> 目标：解释以为zookeeper实现的注册中心原理，解读duubo-registry-zookeeper的源码

这篇文章是讲解注册中心的最后一篇文章。这篇文章讲的是dubbo的注册中心用zookeeper来实现。这种实现注册中心的方法也是dubbo推荐的方法。为了能更加理解zookeeper在dubbo中的应用，接下来我先简单的介绍一下zookeeper。

因为dubbo是一个分布式的RPC开源框架，各个服务之间单独部署，就会出现资源之间不一致的问题。而zookeeper就有保证分布式一致性的特性。ZooKeeper是一种为分布式应用所设计的高可用、高性能且一致的开源协调服务。关于dubbo为什么会推荐使用zookeeper作为它的注册中心实现，有很多书籍以及博客讲解了zookeeper的特性以及优势，这不是本章的重点，我要讲的是zookeeper的数据结构，dubbo服务是如何被zookeeper的数据结构存储管理的，因为这影响到下面源码的解读。zookeeper采用的是树形结构来组织数据节点，它类似于一个标准的文件系统。先来看看下面这张图：

该图是官方文档里面的一张图，展示了dubbo在zookeeper中存储的形式以及节点层级，

1. dubbo的Root层是根目录，通过<dubbo:registry group="dubbo" />的“group”来设置zookeeper的根节点，缺省值是“dubbo”。
2. Service层是服务接口的全名。
3. Type层是分类，一共有四种分类，分别是providers（服务提供者列表）、consumers（服务消费者列表）、routes（路由规则列表）、configurations（配置规则列表）。
4. URL层：根据不同的Type目录：可以有服务提供者 URL 、服务消费者 URL 、路由规则 URL 、配置规则 URL 。不同的Type关注的URL不同。

zookeeper以每个斜杠来分割每一层的znode，比如第一层根节点dubbo就是“/dubbo”，而第二层的Service层就是/com.foo.Barservice，zookeeper的每个节点通过路径来表示以及访问，例如服务提供者启动时，向/dubbo/com.foo.Barservice/providers目录下写入自己的URL地址。关于流程调用说明，见官方文档：

> 文档地址：[http://dubbo.apache.org/zh-cn...](http://dubbo.apache.org/zh-cn/docs/user/references/registry/zookeeper.html)

了解了dubbo在zookeeper中的节点层级，就可以看相关的源码了，下图是包的结构：

跟前面三种实现方式一样的目录，也就两个类，看起来非常的舒服，接下来就来解析这两个类。

## （一）ZookeeperRegistry

该类继承了FailbackRegistry类，该类就是针对注册中心核心的功能注册、订阅、取消注册、取消订阅，查询注册列表进行展开，基于zookeeper来实现。

### 1.属性

```
// 日志记录
private final static Logger logger = LoggerFactory.getLogger(ZookeeperRegistry.class);

// 默认的zookeeper端口
private final static int DEFAULT_ZOOKEEPER_PORT = 2181;

// 默认zookeeper根节点
private final static String DEFAULT_ROOT = "dubbo";

// zookeeper根节点
private final String root;

// 服务接口集合
private final Set<String> anyServices = new ConcurrentHashSet<String>();

// 监听器集合
private final ConcurrentMap<URL, ConcurrentMap<NotifyListener, ChildListener>> zkListeners = new ConcurrentHashMap<URL, ConcurrentMap<NotifyListener, ChildListener>>();

// zookeeper客户端实例
private final ZookeeperClient zkClient;
```

其实你会发现zookeeper虽然是最被推荐的，反而它的实现逻辑相对简单，因为调用了zookeeper服务组件，很多的逻辑不需要在dubbo中自己去实现。上面的属性介绍也很简单，不需要多说，更多的是调用zookeeper客户端。

### 2.构造方法

```
public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
    super(url);
    if (url.isAnyHost()) {
        throw new IllegalStateException("registry address == null");
    }
    // 获得url携带的分组配置，并且作为zookeeper的根节点
    String group = url.getParameter(Constants.GROUP_KEY, DEFAULT_ROOT);
    if (!group.startsWith(Constants.PATH_SEPARATOR)) {
        group = Constants.PATH_SEPARATOR + group;
    }
    this.root = group;
    // 创建zookeeper client
    zkClient = zookeeperTransporter.connect(url);
    // 添加状态监听器，当状态为重连的时候调用恢复方法
    zkClient.addStateListener(new StateListener() {
        @Override
        public void stateChanged(int state) {
            if (state == RECONNECTED) {
                try {
                    // 恢复
                    recover();
                } catch (Exception e) {
                    logger.error(e.getMessage(), e);
                }
            }
        }
    });
}
```

这里有以下几个关注点：

1. 参数中ZookeeperTransporter是一个接口，并且在dubbo中有ZkclientZookeeperTransporter和CuratorZookeeperTransporter两个实现类，ZookeeperTransporter还是一个可扩展的接口，基于 Dubbo SPI Adaptive 机制，会根据url中携带的参数去选择用哪个实现类。
2. 上面我说明了dubbo在zookeeper节点层级有一层是root层，该层是通过group属性来设置的。
3. 给客户端添加一个监听器，当状态为重连的时候调用FailbackRegistry的恢复方法

### 3.appendDefaultPort

```
static String appendDefaultPort(String address) {
    if (address != null && address.length() > 0) {
        int i = address.indexOf(':');
        // 如果地址本身没有端口，则使用默认端口2181
        if (i < 0) {
            return address + ":" + DEFAULT_ZOOKEEPER_PORT;
        } else if (Integer.parseInt(address.substring(i + 1)) == 0) {
            return address.substring(0, i + 1) + DEFAULT_ZOOKEEPER_PORT;
        }
    }
    return address;
}
```

该方法是拼接使用默认的zookeeper端口，就是方地址本身没有端口的时候才使用默认端口。

### 4.isAvailable && destroy

```
@Override
public boolean isAvailable() {
    return zkClient.isConnected();
}

@Override
public void destroy() {
    super.destroy();
    try {
        zkClient.close();
    } catch (Exception e) {
        logger.warn("Failed to close zookeeper client " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

这里两个方法分别是检测zookeeper是否连接以及销毁连接，很简单，都是调用了zookeeper客户端封装好的方法。

### 5.doRegister && doUnregister

```
@Override
protected void doRegister(URL url) {
    try {
        // 创建URL节点，也就是URL层的节点
        zkClient.create(toUrlPath(url), url.getParameter(Constants.DYNAMIC_KEY, true));
    } catch (Throwable e) {
        throw new RpcException("Failed to register " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}

@Override
protected void doUnregister(URL url) {
    try {
        // 删除节点
        zkClient.delete(toUrlPath(url));
    } catch (Throwable e) {
        throw new RpcException("Failed to unregister " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

这两个方法分别是注册和取消注册，也很简单，调用都是客户端create和delete方法，一个是创建一个节点，另一个是删除节点，该操作都在URL层。

### 6.doSubscribe

```
@Override
protected void doSubscribe(final URL url, final NotifyListener listener) {
    try {
        // 处理所有Service层发起的订阅，例如监控中心的订阅
        if (Constants.ANY_VALUE.equals(url.getServiceInterface())) {
            // 获得根目录
            String root = toRootPath();
            // 获得url对应的监听器集合
            ConcurrentMap<NotifyListener, ChildListener> listeners = zkListeners.get(url);
            // 不存在就创建监听器集合
            if (listeners == null) {
                zkListeners.putIfAbsent(url, new ConcurrentHashMap<NotifyListener, ChildListener>());
                listeners = zkListeners.get(url);
            }
            // 获得节点监听器
            ChildListener zkListener = listeners.get(listener);
            // 如果该节点监听器为空，则创建
            if (zkListener == null) {
                listeners.putIfAbsent(listener, new ChildListener() {
                    @Override
                    public void childChanged(String parentPath, List<String> currentChilds) {
                        // 遍历现有的节点，如果现有的服务集合中没有该节点，则加入该节点，然后订阅该节点
                        for (String child : currentChilds) {
                            // 解码
                            child = URL.decode(child);
                            if (!anyServices.contains(child)) {
                                anyServices.add(child);
                                subscribe(url.setPath(child).addParameters(Constants.INTERFACE_KEY, child,
                                        Constants.CHECK_KEY, String.valueOf(false)), listener);
                            }
                        }
                    }
                });
                // 重新获取，为了保证一致性
                zkListener = listeners.get(listener);
            }
            // 创建service节点，该节点为持久节点
            zkClient.create(root, false);
            // 向zookeeper的service节点发起订阅，获得Service接口全名数组
            List<String> services = zkClient.addChildListener(root, zkListener);
            if (services != null && !services.isEmpty()) {
                // 遍历Service接口全名数组
                for (String service : services) {
                    service = URL.decode(service);
                    anyServices.add(service);
                    // 发起该service层的订阅
                    subscribe(url.setPath(service).addParameters(Constants.INTERFACE_KEY, service,
                            Constants.CHECK_KEY, String.valueOf(false)), listener);
                }
            }
        } else {
            // 处理指定 Service 层的发起订阅，例如服务消费者的订阅
            List<URL> urls = new ArrayList<URL>();
            // 遍历分类数组
            for (String path : toCategoriesPath(url)) {
                // 获得监听器集合
                ConcurrentMap<NotifyListener, ChildListener> listeners = zkListeners.get(url);
                // 如果没有则创建
                if (listeners == null) {
                    zkListeners.putIfAbsent(url, new ConcurrentHashMap<NotifyListener, ChildListener>());
                    listeners = zkListeners.get(url);
                }
                // 获得节点监听器
                ChildListener zkListener = listeners.get(listener);
                if (zkListener == null) {
                    listeners.putIfAbsent(listener, new ChildListener() {
                        @Override
                        public void childChanged(String parentPath, List<String> currentChilds) {
                            // 通知服务变化 回调NotifyListener
                            ZookeeperRegistry.this.notify(url, listener, toUrlsWithEmpty(url, parentPath, currentChilds));
                        }
                    });
                    // 重新获取节点监听器，保证一致性
                    zkListener = listeners.get(listener);
                }
                // 创建type节点，该节点为持久节点
                zkClient.create(path, false);
                // 向zookeeper的type节点发起订阅
                List<String> children = zkClient.addChildListener(path, zkListener);
                if (children != null) {
                    // 加入到自子节点数据数组
                    urls.addAll(toUrlsWithEmpty(url, path, children));
                }
            }
            // 通知数据变化
            notify(url, listener, urls);
        }
    } catch (Throwable e) {
        throw new RpcException("Failed to subscribe " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

这个方法是订阅，逻辑实现比较多，可以分两段来看，这里的实现把所有Service层发起的订阅以及指定的Service层发起的订阅分开处理。所有Service层类似于监控中心发起的订阅。指定的Service层发起的订阅可以看作是服务消费者的订阅。订阅的大致逻辑类似，不过还是有几个区别：

1. 所有Service层发起的订阅中的ChildListener是在在 Service 层发生变更时，才会做出解码，用anyServices属性判断是否是新增的服务，最后调用父类的subscribe订阅。而指定的Service层发起的订阅是在URL层发生变更的时候，调用notify，回调回调NotifyListener的逻辑，做到通知服务变更。
2. 所有Service层发起的订阅中客户端创建的节点是Service节点，该节点为持久节点，而指定的Service层发起的订阅中创建的节点是Type节点，该节点也是持久节点。这里补充一下zookeeper的持久节点是节点创建后，就一直存在，直到有删除操作来主动清除这个节点，不会因为创建该节点的客户端会话失效而消失。而临时节点的生命周期和客户端会话绑定。也就是说，如果客户端会话失效，那么这个节点就会自动被清除掉。注意，这里提到的是会话失效，而非连接断开。另外，在临时节点下面不能创建子节点。
3. 指定的Service层发起的订阅中调用了两次notify，第一次是增量的通知，也就是只是通知这次增加的服务节点，而第二个是全量的通知。

### 7.doUnsubscribe

```
@Override
protected void doUnsubscribe(URL url, NotifyListener listener) {
    // 获得监听器集合
    ConcurrentMap<NotifyListener, ChildListener> listeners = zkListeners.get(url);
    if (listeners != null) {
        // 获得子节点的监听器
        ChildListener zkListener = listeners.get(listener);
        if (zkListener != null) {
            // 如果为全部的服务接口，例如监控中心
            if (Constants.ANY_VALUE.equals(url.getServiceInterface())) {
                // 获得根目录
                String root = toRootPath();
                // 移除监听器
                zkClient.removeChildListener(root, zkListener);
            } else {
                // 遍历分类数组进行移除监听器
                for (String path : toCategoriesPath(url)) {
                    zkClient.removeChildListener(path, zkListener);
                }
            }
        }
    }
}
```

该方法是取消订阅，也是分为两种情况，所有的Service发起的取消订阅还是指定的Service发起的取消订阅。可以看到所有的Service发起的取消订阅就直接移除了根目录下所有的监听器，而指定的Service发起的取消订阅是移除了该Service层下面的所有Type节点监听器。如果不太明白再回去看看前面的那个节点层级图。

### 8.lookup

```
@Override
public List<URL> lookup(URL url) {
    if (url == null) {
        throw new IllegalArgumentException("lookup url == null");
    }
    try {
        List<String> providers = new ArrayList<String>();
        // 遍历分组类别
        for (String path : toCategoriesPath(url)) {
            // 获得子节点
            List<String> children = zkClient.getChildren(path);
            if (children != null) {
                providers.addAll(children);
            }
        }
        // 获得 providers 中，和 consumer 匹配的 URL 数组
        return toUrlsWithoutEmpty(url, providers);
    } catch (Throwable e) {
        throw new RpcException("Failed to lookup " + url + " from zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

该方法就是查询符合条件的已经注册的服务。调用了toUrlsWithoutEmpty方法，在后面会讲到。

### 9.toServicePath

```
private String toServicePath(URL url) {
    String name = url.getServiceInterface();
    // 如果是包括所有服务，则返回根节点
    if (Constants.ANY_VALUE.equals(name)) {
        return toRootPath();
    }
    return toRootDir() + URL.encode(name);
}
```

该方法是获得服务路径，拼接规则：Root + Type。

### 10.toCategoriesPath

```
private String[] toCategoriesPath(URL url) {
    String[] categories;
    // 如果url携带的分类配置为*，则创建包括所有分类的数组
    if (Constants.ANY_VALUE.equals(url.getParameter(Constants.CATEGORY_KEY))) {
        categories = new String[]{Constants.PROVIDERS_CATEGORY, Constants.CONSUMERS_CATEGORY,
                Constants.ROUTERS_CATEGORY, Constants.CONFIGURATORS_CATEGORY};
    } else {
        // 返回url携带的分类配置
        categories = url.getParameter(Constants.CATEGORY_KEY, new String[]{Constants.DEFAULT_CATEGORY});
    }
    String[] paths = new String[categories.length];
    for (int i = 0; i < categories.length; i++) {
        // 加上服务路径
        paths[i] = toServicePath(url) + Constants.PATH_SEPARATOR + categories[i];
    }
    return paths;
}

private String toCategoryPath(URL url) {
    return toServicePath(url) + Constants.PATH_SEPARATOR + url.getParameter(Constants.CATEGORY_KEY, Constants.DEFAULT_CATEGORY);
}
```

第一个方法是获得分类数组，也就是url携带的服务下的所有Type节点数组。第二个是获得分类路径，分类路径拼接规则：Root + Service + Type

### 11.toUrlPath

```
private String toUrlPath(URL url) {
    return toCategoryPath(url) + Constants.PATH_SEPARATOR + URL.encode(url.toFullString());
}
```

该方法是获得URL路径，拼接规则是Root + Service + Type + URL

### 12.toUrlsWithoutEmpty && toUrlsWithEmpty

```
private List<URL> toUrlsWithoutEmpty(URL consumer, List<String> providers) {
    List<URL> urls = new ArrayList<URL>();
    if (providers != null && !providers.isEmpty()) {
        // 遍历服务提供者
        for (String provider : providers) {
            // 解码
            provider = URL.decode(provider);
            if (provider.contains("://")) {
                // 把服务转化成url的形式
                URL url = URL.valueOf(provider);
                // 判断是否匹配，如果匹配， 则加入到集合中
                if (UrlUtils.isMatch(consumer, url)) {
                    urls.add(url);
                }
            }
        }
    }
    return urls;
}

private List<URL> toUrlsWithEmpty(URL consumer, String path, List<String> providers) {
    // 返回和服务消费者匹配的服务提供者url
    List<URL> urls = toUrlsWithoutEmpty(consumer, providers);
    // 如果不存在，则创建`empty://` 的 URL返回
    if (urls == null || urls.isEmpty()) {
        int i = path.lastIndexOf('/');
        String category = i < 0 ? path : path.substring(i + 1);
        URL empty = consumer.setProtocol(Constants.EMPTY_PROTOCOL).addParameter(Constants.CATEGORY_KEY, category);
        urls.add(empty);
    }
    return urls;
}
```

第一个toUrlsWithoutEmpty方法是获得 providers 中，和 consumer 匹配的 URL 数组，第二个toUrlsWithEmpty方法是调用了第一个方法后增加了若不存在匹配，则创建 `empty://` 的 URL返回。通过这样的方式，可以处理类似服务提供者为空的情况。

## （二）ZookeeperRegistryFactory

该类继承了AbstractRegistryFactory类，实现了AbstractRegistryFactory抽象出来的createRegistry方法，看一下原代码：

```
public class ZookeeperRegistryFactory extends AbstractRegistryFactory {

    private ZookeeperTransporter zookeeperTransporter;

    public void setZookeeperTransporter(ZookeeperTransporter zookeeperTransporter) {
        this.zookeeperTransporter = zookeeperTransporter;
    }

    @Override
    public Registry createRegistry(URL url) {
        return new ZookeeperRegistry(url, zookeeperTransporter);
    }

}
```

可以看到就是实例化了ZookeeperRegistry而已，所有这里就不解释了。

# 远程通讯——开篇

服务治理框架中可以大致分为服务通信和服务管理两个部分，前面我先讲到有关注册中心的内容，也就是服务管理，当然dubbo的服务管理还包括监控中心、 telnet 命令，它们起到的是人工的服务管理作用，这个后续再介绍。接下来我要讲解的就是跟服务通信有关的部分，也就是远程通讯模块。我在[《dubbo源码解析（一）Hello,Dubbo》](https://segmentfault.com/a/1190000016741532?share_user=1030000009586134)的"（六）dubbo-remoting——远程通信模块“中提到过一些内容。该模块中提供了多种客户端和服务端通信的功能，而在对NIO框架选型上，dubbo交由用户选择，它集成了mina、netty、grizzly等各类NIO框架来搭建NIO服务器和客户端，并且利用dubbo的SPI扩展机制可以让用户自定义选择。如果对SPI不太了解的朋友可以查看[《dubbo源码解析（二）Dubbo扩展机制SPI》](https://segmentfault.com/a/1190000016842868)。

接下来我们先来看看dubbo-remoting的包结构：

可以看到，大篇幅的逻辑在dubbo-remoting-api中，所以我对于dubbo-remoting-api的解读会分为下面五个部分来说明，其中第五点会在本文介绍，其他四点会分别用四篇文章来介绍：

1. buffer包：缓冲在NIO框架中是很重要的存在，各个NIO框架都实现了自己相应的缓存操作。这个buffer包下包括了缓冲区的接口以及抽象
2. exchange包：信息交换层，其中封装了请求响应模式，在传输层之上重新封装了 Request-Response 语义，为了满足RPC的需求。这层可以认为专注在Request和Response携带的信息上。该层是RPC调用的通讯基础之一。
3. telnet包：dubbo支持通过telnet命令来进行服务治理，该包下就封装了这些通用指令的逻辑实现。
4. transport包：网络传输层，它只负责单向消息传输，是对 Mina, Netty, Grizzly 的抽象，它也可以扩展 UDP 传输。该层是RPC调用的通讯基础之一。
5. 最外层的源码：该部分我会在下面之间给出介绍。

### 接口Endpoint

dubbo抽象出一个端的概念，也就是Endpoint接口，这个端就是一个点，而点对点之间是可以双向传输。在端的基础上在衍生出通道、客户端以及服务端的概念，也就是下面要介绍的Channel、Client、Server三个接口。在传输层，其实Client和Server的区别只是在语义上区别，并不区分请求和应答职责，在交换层客户端和服务端也是一个点，但是已经是有方向的点，所以区分了明确的请求和应答职责。两者都具备发送的能力，只是客户端和服务端所关注的事情不一样，这个在后面会分开介绍，而Endpoint接口抽象的方法就是它们共同拥有的方法。这也就是它们都能被抽象成端的原因。

来看一下它的源码：

```
public interface Endpoint {

    // 获得该端的url
    URL getUrl();

    // 获得该端的通道处理器
    ChannelHandler getChannelHandler();
    
    // 获得该端的本地地址
    InetSocketAddress getLocalAddress();
    
    // 发送消息
    void send(Object message) throws RemotingException;
    
    // 发送消息，sent是是否已经发送的标记
    void send(Object message, boolean sent) throws RemotingException;
    
    // 关闭
    void close();
    
    // 优雅的关闭，也就是加入了等待时间
    void close(int timeout);
    
    // 开始关闭
    void startClose();
    
    // 判断是否已经关闭
    boolean isClosed();

}
```

1. 前三个方法是获得该端本身的一些属性，
2. 两个send方法是发送消息，其中第二个方法多了一个sent的参数，为了区分是否是第一次发送消息。
3. 后面几个方法是提供了关闭通道的操作以及判断通道是否关闭的操作。

### （二）接口Channel

该接口是通道接口，通道是通讯的载体。还是用自动贩卖机的例子，自动贩卖机就好比是一个通道，消息发送端会往通道输入消息，而接收端会从通道读消息。并且接收端发现通道没有消息，就去做其他事情了，不会造成阻塞。所以channel可以读也可以写，并且可以异步读写。channel是client和server的传输桥梁。channel和client是一一对应的，也就是一个client对应一个channel，但是channel和server是多对一对关系，也就是一个server可以对应多个channel。

```
public interface Channel extends Endpoint {

    // 获得远程地址
    InetSocketAddress getRemoteAddress();

    // 判断通道是否连接
    boolean isConnected();

    // 判断是否有该key的值
    boolean hasAttribute(String key);

    // 获得该key对应的值
    Object getAttribute(String key);

    // 添加属性
    void setAttribute(String key, Object value);

    // 移除属性
    void removeAttribute(String key);

}
```

可以看到Channel继承了Endpoint，也就是端抽象出来的方法也同样是channel所需要的。上面的几个方法很好理解，我就不多介绍了。

### 接口ChannelHandler

```
@SPI
public interface ChannelHandler {

    // 连接该通道
    void connected(Channel channel) throws RemotingException;

    // 断开该通道
    void disconnected(Channel channel) throws RemotingException;

    // 发送给这个通道消息
    void sent(Channel channel, Object message) throws RemotingException;

    // 从这个通道内接收消息
    void received(Channel channel, Object message) throws RemotingException;

    // 从这个通道内捕获异常
    void caught(Channel channel, Throwable exception) throws RemotingException;

}
```

该接口是负责channel中的逻辑处理，并且可以看到这个接口有注解@SPI，是个可扩展接口，到时候都会在下面介绍各类NIO框架的时候会具体讲到它的实现类。

### 接口Client

```
public interface Client extends Endpoint, Channel, Resetable {
    
    // 重连
    void reconnect() throws RemotingException;

    // 重置，不推荐使用
    @Deprecated
    void reset(com.alibaba.dubbo.common.Parameters parameters);

}
```

客户端接口，可以看到它继承了Endpoint、Channel和Resetable接口，继承Endpoint的原因上面我已经提到过了，客户端和服务端其实只是语义上的不同，客户端就是一个点。继承Channel是因为客户端跟通道是一一对应的，所以做了这样的设计，还继承了Resetable接口是为了实现reset方法，该方法，不过已经打上@Deprecated注解，不推荐使用。除了这些客户端就只需要关注一个重连的操作。

这里插播一个公共模块下的接口Resetable：

```
public interface Resetable {

    // 用于根据新传入的 url 属性，重置自己内部的一些属性
    void reset(URL url);

}
```

该方法就是根据新的url来重置内部的属性。

### 接口Server

```
public interface Server extends Endpoint, Resetable {

    // 判断是否绑定到本地端口，也就是该服务器是否启动成功，能够连接、接收消息，提供服务。
    boolean isBound();

    // 获得连接该服务器的通道
    Collection<Channel> getChannels();

    // 通过远程地址获得该地址对应的通道
    Channel getChannel(InetSocketAddress remoteAddress);

    @Deprecated
    void reset(com.alibaba.dubbo.common.Parameters parameters);

}
```

该接口是服务端接口，继承了Endpoint和Resetable，继承Endpoint是因为服务端也是一个点，继承Resetable接口是为了继承reset方法。除了这些以外，服务端独有的是检测是否启动成功，还有事获得连接该服务器上所有通道，这里获得所有通道其实就意味着获得了所有连接该服务器的客户端，因为客户端和通道是一一对应的。

### 接口Codec && Codec2

这两个都是编解码器，那么什么叫做编解码器，在网络中只是讲数据看成是原始的字节序列，但是我们的应用程序会把这些字节组织成有意义的信息，那么网络字节流和数据间的转化就是很常见的任务。而编码器是讲应用程序的数据转化为网络格式，解码器则是讲网络格式转化为应用程序，同时具备这两种功能的单一组件就叫编解码器。在dubbo中Codec是老的编解码器接口，而Codec2是新的编解码器接口，并且dubbo已经用CodecAdapter把Codec适配成Codec2了。所以在这里我就介绍Codec2接口，毕竟人总要往前看。

```
@SPI
public interface Codec2 {
      //编码
    @Adaptive({Constants.CODEC_KEY})
    void encode(Channel channel, ChannelBuffer buffer, Object message) throws IOException;
    //解码
    @Adaptive({Constants.CODEC_KEY})
    Object decode(Channel channel, ChannelBuffer buffer) throws IOException;

    enum DecodeResult {
        // 需要更多输入和忽略一些输入
        NEED_MORE_INPUT, SKIP_SOME_INPUT
    }

}
```

因为是编解码器，所以有两个方法分别是编码和解码，上述有以下几个关注的：

1. Codec2是一个可扩展的接口，因为有@SPI注解。
2. 用到了Adaptive机制，首先去url中寻找key为codec的value，来加载url携带的配置中指定的codec的实现。
3. 该接口中有个枚举类型DecodeResult，因为解码过程中，需要解决 TCP 拆包、粘包的场景，所以增加了这两种解码结果，关于TCP 拆包、粘包的场景我就不多解释，不懂得朋友可以google一下。

### 接口Decodeable

```
public interface Decodeable {

    //解码
    public void decode() throws Exception;

}
```

该接口是可解码的接口，该接口有两个作用，第一个是在调用真正的decode方法实现的时候会有一些校验，判断是否可以解码，并且对解码失败会有一些消息设置；第二个是被用来message核对用的。后面看具体的实现会更了解该接口的作用。

### （八）接口Dispatcher

```
@SPI(AllDispatcher.NAME)
public interface Dispatcher {

    // 调度
    @Adaptive({Constants.DISPATCHER_KEY, "dispather", "channel.handler"})
    // The last two parameters are reserved for compatibility with the old configuration
    ChannelHandler dispatch(ChannelHandler handler, URL url);

}
```

该接口是调度器接口，dispatch是线程池的调度方法，这边有几个注意点：

1. 该接口是一个可扩展接口，并且默认实现AllDispatcher，也就是所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。
2. 用了Adaptive注解，也就是按照URL中配置来加载实现类，后面两个参数是为了兼容老版本，如果这是三个key对应的值都为空，就选择AllDispatcher来实现。

### （九）接口Transporter

```
@SPI("netty")
public interface Transporter {
    
    // 绑定一个服务器
    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})
    Server bind(URL url, ChannelHandler handler) throws RemotingException;
    
    // 连接一个服务器，即创建一个客户端
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;

}
```

该接口是网络传输接口，有以下几个注意点：

1. 该接口是一个可扩展的接口，并且默认实现NettyTransporter。
2. 用了dubbo SPI扩展机制中的Adaptive注解，加载对应的bind方法，使用url携带的server或者transporter属性值，加载对应的connect方法，使用url携带的client或者transporter属性值，不了解SPI扩展机制的可以查看[《dubbo源码解析（二）Dubbo扩展机制SPI》](https://segmentfault.com/a/1190000016842868)。

### （十）Transporters

```
public class Transporters {

    static {
        // check duplicate jar package
        // 检查重复的 jar 包
        Version.checkDuplicate(Transporters.class);
        Version.checkDuplicate(RemotingException.class);
    }

    private Transporters() {
    }

    public static Server bind(String url, ChannelHandler... handler) throws RemotingException {
        return bind(URL.valueOf(url), handler);
    }

    public static Server bind(URL url, ChannelHandler... handlers) throws RemotingException {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        if (handlers == null || handlers.length == 0) {
            throw new IllegalArgumentException("handlers == null");
        }
        ChannelHandler handler;
        // 创建handler
        if (handlers.length == 1) {
            handler = handlers[0];
        } else {
            handler = new ChannelHandlerDispatcher(handlers);
        }
        // 调用Transporter的实现类对象的bind方法。
        // 例如实现NettyTransporter，则调用NettyTransporter的connect，并且返回相应的server
        return getTransporter().bind(url, handler);
    }

    public static Client connect(String url, ChannelHandler... handler) throws RemotingException {
        return connect(URL.valueOf(url), handler);
    }

    public static Client connect(URL url, ChannelHandler... handlers) throws RemotingException {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        ChannelHandler handler;
        if (handlers == null || handlers.length == 0) {
            handler = new ChannelHandlerAdapter();
        } else if (handlers.length == 1) {
            handler = handlers[0];
        } else {
            handler = new ChannelHandlerDispatcher(handlers);
        }
        // 调用Transporter的实现类对象的connect方法。
        // 例如实现NettyTransporter，则调用NettyTransporter的connect，并且返回相应的client
        return getTransporter().connect(url, handler);
    }

    public static Transporter getTransporter() {
        return ExtensionLoader.getExtensionLoader(Transporter.class).getAdaptiveExtension();
    }

}
```

1. 该类用到了设计模式的外观模式，通过该类的包装，我们就不会看到内部具体的实现细节，这样降低了程序的复杂度，也提高了程序的可维护性。比如这个类，包装了调用各种实现Transporter接口的方法，通过getTransporter来获得Transporter的实现对象，具体实现哪个实现类，取决于url中携带的配置信息，如果url中没有相应的配置，则默认选择@SPI中的默认值netty。
2. bind和connect方法分别有两个重载方法，其中的操作只是把把字符串的url转化为URL对象。
3. 静态代码块中检测了一下jar包是否有重复。

### （十一）RemotingException && ExecutionException && TimeoutException

这三个类是远程通信的异常类：

1. RemotingException继承了Exception类，是远程通信的基础异常。
2. ExecutionException继承了RemotingException类，ExecutionException是远程通信的执行异常。
3. TimeoutException继承了RemotingException类，TimeoutException是超时异常。

为了不影响篇幅，这三个类源码我就不介绍了，因为比较简单。

# 远程通讯——Transport层

先预警一下，该文篇幅会很长，做好心理准备。Transport层也就是网络传输层，在远程通信中必然会涉及到传输。它在dubbo 的框架设计中也处于倒数第二层，当然最底层是序列化，这个后面介绍。官方文档对Transport层的解释是抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel、Transporter、Client、Server、Codec。那我们现在先来看这个包下面的类图：

可以看到有四个包继承了AbstractChannel、AbstractServer、AbstractClient。也就是说现在Transport层是抽象mina、netty以及grizzly为统一接口。看完类图，再来看看包结构：

### （一）AbstractPeer

```
public abstract class AbstractPeer implements Endpoint, ChannelHandler {
    private final ChannelHandler handler;

    private volatile URL url;
    /**
     * 是否正在关闭
     */
    // closing closed means the process is being closed and close is finished
    private volatile boolean closing;
    /**
     * 是否关闭完成
     */
    private volatile boolean closed;

    public AbstractPeer(URL url, ChannelHandler handler) {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        if (handler == null) {
            throw new IllegalArgumentException("handler == null");
        }
        this.url = url;
        this.handler = handler;
    }
}
```

该类实现了Endpoint和ChannelHandler两个接口，要关注的两个点：

1. 实现ChannelHandler接口并且有在属性中还有一个handler，下面很多实现方法也是直接调用了handler方法，这种模式叫做装饰模式，这样做可以对装饰对象灵活的增强功能。对装饰模式不懂的朋友可以google一下。有很多例子介绍。
2. 在该类中有closing和closed属性，在Endpoint中有很多关于关闭通道的操作，会有关闭中和关闭完成的状态区分，在该类中就缓存了这两个属性来判断关闭的状态。

下面我就介绍该类中的send方法，其他方法比较好理解，到时候可以直接看源码：

```
@Override
public void send(Object message) throws RemotingException {
    // url中sent的配置项
    send(message, url.getParameter(Constants.SENT_KEY, false));
}
```

该配置项是选择是否等待消息发出：

1. sent值为true，等待消息发出，消息发送失败将抛出异常。
2. sent值为false，不等待消息发出，将消息放入 IO 队列，即刻返回。

对该类还有点糊涂的朋友，记住在ChannelHandler接口，该类就做了装饰模式中装饰角色，在Endpoint接口，只是维护了通道的正在关闭和关闭完成两个状态。

### （二）AbstractEndpoint

```
public abstract class AbstractEndpoint extends AbstractPeer implements Resetable {

    /**
     * 日志记录
     */
    private static final Logger logger = LoggerFactory.getLogger(AbstractEndpoint.class);

    /**
     * 编解码器
     */
    private Codec2 codec;

    /**
     * 超时时间
     */
    private int timeout;

    /**
     * 连接超时时间
     */
    private int connectTimeout;

    public AbstractEndpoint(URL url, ChannelHandler handler) {
        super(url, handler);
        this.codec = getChannelCodec(url);
        // 优先从url配置中取，如果没有，默认为1s
        this.timeout = url.getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
        // 优先从url配置中取，如果没有，默认为3s
        this.connectTimeout = url.getPositiveParameter(Constants.CONNECT_TIMEOUT_KEY, Constants.DEFAULT_CONNECT_TIMEOUT);
    }

    /**
     * 从url中获得编解码器的配置，并且返回该实例
     * @param url
     * @return
     */
    protected static Codec2 getChannelCodec(URL url) {
        String codecName = url.getParameter(Constants.CODEC_KEY, "telnet");
        // 优先从Codec2的扩展类中找
        if (ExtensionLoader.getExtensionLoader(Codec2.class).hasExtension(codecName)) {
            return ExtensionLoader.getExtensionLoader(Codec2.class).getExtension(codecName);
        } else {
            return new CodecAdapter(ExtensionLoader.getExtensionLoader(Codec.class)
                    .getExtension(codecName));
        }
    }

}
```

该类是端点的抽象类，其中封装了编解码器以及两个超时时间。基于dubbo 的SPI机制，获得相应的编解码器实现对象，编解码器优先从Codec2的扩展类中寻找。

下面来看看该类中的reset方法：

```
@Override
public void reset(URL url) {
    if (isClosed()) {
        throw new IllegalStateException("Failed to reset parameters "
                + url + ", cause: Channel closed. channel: " + getLocalAddress());
    }
    try {
        // 判断重置的url中有没有携带timeout，有的话重置
        if (url.hasParameter(Constants.TIMEOUT_KEY)) {
            int t = url.getParameter(Constants.TIMEOUT_KEY, 0);
            if (t > 0) {
                this.timeout = t;
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
    try {
        // 判断重置的url中有没有携带connect.timeout，有的话重置
        if (url.hasParameter(Constants.CONNECT_TIMEOUT_KEY)) {
            int t = url.getParameter(Constants.CONNECT_TIMEOUT_KEY, 0);
            if (t > 0) {
                this.connectTimeout = t;
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
    try {
        // 判断重置的url中有没有携带codec，有的话重置
        if (url.hasParameter(Constants.CODEC_KEY)) {
            this.codec = getChannelCodec(url);
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
}

@Deprecated
public void reset(com.alibaba.dubbo.common.Parameters parameters) {
    reset(getUrl().addParameters(parameters.getParameters()));
}
```

这个方法是Resetable接口中的方法，可以看到以前的reset实现方法都加上了@Deprecated注解，不推荐使用了，因为这种实现方式重置太复杂，需要把所有参数都设置一遍，比如我只想重置一个超时时间，但是其他值不变，如果用以前的reset，我需要在url中把所有值都带上，就会很多余。现在用新的reset，每次只关心我需要重置的值，只更改为需要重置的值。比如上面的代码所示，只想修改超时时间，那我就只在url中携带超时时间的参数。

### （三）AbstractServer

该类继承了AbstractEndpoint并且实现Server接口，是服务器抽象类。重点实现了服务器的公共逻辑，比如发送消息，关闭通道，连接通道，断开连接等。并且抽象了打开和关闭服务器两个方法。

#### 1.属性

```
/**
 * 服务器线程名称
 */
protected static final String SERVER_THREAD_POOL_NAME = "DubboServerHandler";
private static final Logger logger = LoggerFactory.getLogger(AbstractServer.class);
/**
 * 线程池
 */
ExecutorService executor;
/**
 * 服务地址，也就是本地地址
 */
private InetSocketAddress localAddress;
/**
 * 绑定地址
 */
private InetSocketAddress bindAddress;
/**
 * 最大可接受的连接数
 */
private int accepts;
/**
 * 空闲超时时间，单位是s
 */
private int idleTimeout = 600; //600 seconds
```

该类的属性比较好理解，就是稍微注意一下idleTimeout的单位是s。

#### 2.构造函数

```
public AbstractServer(URL url, ChannelHandler handler) throws RemotingException {
    super(url, handler);
    // 从url中获得本地地址
    localAddress = getUrl().toInetSocketAddress();

    // 从url配置中获得绑定的ip
    String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());
    // 从url配置中获得绑定的端口号
    int bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());
    // 判断url中配置anyhost是否为true或者判断host是否为不可用的本地Host
    if (url.getParameter(Constants.ANYHOST_KEY, false) || NetUtils.isInvalidLocalHost(bindIp)) {
        bindIp = NetUtils.ANYHOST;
    }
    bindAddress = new InetSocketAddress(bindIp, bindPort);
    // 从url中获取配置，默认值为0
    this.accepts = url.getParameter(Constants.ACCEPTS_KEY, Constants.DEFAULT_ACCEPTS);
    // 从url中获取配置，默认600s
    this.idleTimeout = url.getParameter(Constants.IDLE_TIMEOUT_KEY, Constants.DEFAULT_IDLE_TIMEOUT);
    try {
        // 开启服务器
        doOpen();
        if (logger.isInfoEnabled()) {
            logger.info("Start " + getClass().getSimpleName() + " bind " + getBindAddress() + ", export " + getLocalAddress());
        }
    } catch (Throwable t) {
        throw new RemotingException(url.toInetSocketAddress(), null, "Failed to bind " + getClass().getSimpleName()
                + " on " + getLocalAddress() + ", cause: " + t.getMessage(), t);
    }
    // 获得线程池
    //fixme replace this with better method
    DataStore dataStore = ExtensionLoader.getExtensionLoader(DataStore.class).getDefaultExtension();
    executor = (ExecutorService) dataStore.get(Constants.EXECUTOR_SERVICE_COMPONENT_KEY, Integer.toString(url.getPort()));
}
```

构造函数大部分逻辑就是从url中取配置，存到缓存中，并且做了开启服务器的操作。具体的看上面的注释，还是比较清晰的。

#### 3.reset方法

```
@Override
public void reset(URL url) {
    if (url == null) {
        return;
    }
    try {
        // 重置accepts的值
        if (url.hasParameter(Constants.ACCEPTS_KEY)) {
            int a = url.getParameter(Constants.ACCEPTS_KEY, 0);
            if (a > 0) {
                this.accepts = a;
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
    try {
        // 重置idle.timeout的值
        if (url.hasParameter(Constants.IDLE_TIMEOUT_KEY)) {
            int t = url.getParameter(Constants.IDLE_TIMEOUT_KEY, 0);
            if (t > 0) {
                this.idleTimeout = t;
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
    try {
        // 重置线程数配置
        if (url.hasParameter(Constants.THREADS_KEY)
                && executor instanceof ThreadPoolExecutor && !executor.isShutdown()) {
            ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
            // 获得url配置中的线程数
            int threads = url.getParameter(Constants.THREADS_KEY, 0);
            // 获得线程池允许的最大线程数
            int max = threadPoolExecutor.getMaximumPoolSize();
            // 返回核心线程数
            int core = threadPoolExecutor.getCorePoolSize();
            // 设置最大线程数和核心线程数
            if (threads > 0 && (threads != max || threads != core)) {
                if (threads < core) {
                    // 如果设置的线程数比核心线程数少，则直接设置核心线程数
                    threadPoolExecutor.setCorePoolSize(threads);
                    if (core == max) {
                        // 当核心线程数和最大线程数相等的时候，把最大线程数也重置
                        threadPoolExecutor.setMaximumPoolSize(threads);
                    }
                } else {
                    // 当大于核心线程数时，直接设置最大线程数
                    threadPoolExecutor.setMaximumPoolSize(threads);
                    // 只有当核心线程数和最大线程数相等的时候才设置核心线程数
                    if (core == max) {
                        threadPoolExecutor.setCorePoolSize(threads);
                    }
                }
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
    // 重置url
    super.setUrl(getUrl().addParameters(url.getParameters()));
}
```

该类中的reset方法做了三个值的重置，分别是最大可连接的客户端数量、空闲超时时间以及线程池的两个配置参数。其中要注意核心线程数和最大线程数的区别。举个例子，核心线程数就像是工厂正式工，最大线程数，就是工厂临时工作量加大，请了一批临时工，临时工加正式工的和就是最大线程数，等这批任务结束后，临时工要辞退的，而正式工会留下。

还有send、close、connected、disconnected等方法比较简单，如果有兴趣，可以到我的GitHub查看，地址文章末尾会给出。

### （四）AbstractClient

该类是客户端的抽象类，继承了AbstractEndpoint类，实现了Client接口，该类中也是做了客户端公用的重连逻辑，抽象了打开客户端、关闭客户端、连接服务器、断开服务器连接以及获得通道方法，让子类去重点关注这几个方法。

#### 1.属性

```
/**
 * 客户端线程名称
 */
protected static final String CLIENT_THREAD_POOL_NAME = "DubboClientHandler";
private static final Logger logger = LoggerFactory.getLogger(AbstractClient.class);
/**
 * 线程池id
 */
private static final AtomicInteger CLIENT_THREAD_POOL_ID = new AtomicInteger();
/**
 * 重连定时任务执行器
 */
private static final ScheduledThreadPoolExecutor reconnectExecutorService = new ScheduledThreadPoolExecutor(2, new NamedThreadFactory("DubboClientReconnectTimer", true));
/**
 * 连接锁
 */
private final Lock connectLock = new ReentrantLock();
/**
 * 发送消息时，若断开，是否重连
 */
private final boolean send_reconnect;
/**
 * 重连次数
 */
private final AtomicInteger reconnect_count = new AtomicInteger(0);
/**
 * 在这之前是否调用重新连接的错误日志
 */
// Reconnection error log has been called before?
private final AtomicBoolean reconnect_error_log_flag = new AtomicBoolean(false);
/**
 * 重连 warning 的间隔.(waring多少次之后，warning一次)，也就是错误多少次后告警一次错误
 */
// reconnect warning period. Reconnect warning interval (log warning after how many times) //for test
private final int reconnect_warning_period;
/**
 * 关闭超时时间
 */
private final long shutdown_timeout;
/**
 * 线程池
 */
protected volatile ExecutorService executor;
/**
 * 重连执行任务
 */
private volatile ScheduledFuture<?> reconnectExecutorFuture = null;
// the last successed connected time
/**
 * 最后成功连接的时间
 */
private long lastConnectedTime = System.currentTimeMillis();
```

上述属性大部分跟重连有关，该类最重要的也是封装了重连的逻辑。

#### 2.构造函数

```
public AbstractClient(URL url, ChannelHandler handler) throws RemotingException {
    super(url, handler);

    // 从url中获得是否重连的配置，默认为false
    send_reconnect = url.getParameter(Constants.SEND_RECONNECT_KEY, false);

    // 从url中获得关闭超时时间，默认为900s
    shutdown_timeout = url.getParameter(Constants.SHUTDOWN_TIMEOUT_KEY, Constants.DEFAULT_SHUTDOWN_TIMEOUT);

    // The default reconnection interval is 2s, 1800 means warning interval is 1 hour.
    // 重连的默认值是2s，重连 warning 的间隔默认是1800，当出错的时候，每隔1800*2=3600s报警一次
    reconnect_warning_period = url.getParameter("reconnect.waring.period", 1800);

    try {
        // 打开客户端
        doOpen();
    } catch (Throwable t) {
        close();
        throw new RemotingException(url.toInetSocketAddress(), null,
                "Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                        + " connect to the server " + getRemoteAddress() + ", cause: " + t.getMessage(), t);
    }
    try {
        // connect.
        // 连接服务器
        connect();
        if (logger.isInfoEnabled()) {
            logger.info("Start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress() + " connect to the server " + getRemoteAddress());
        }
    } catch (RemotingException t) {
        if (url.getParameter(Constants.CHECK_KEY, true)) {
            close();
            throw t;
        } else {
            logger.warn("Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                    + " connect to the server " + getRemoteAddress() + " (check == false, ignore and retry later!), cause: " + t.getMessage(), t);
        }
    } catch (Throwable t) {
        close();
        throw new RemotingException(url.toInetSocketAddress(), null,
                "Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                        + " connect to the server " + getRemoteAddress() + ", cause: " + t.getMessage(), t);
    }

    // 从缓存中获得线程池
    executor = (ExecutorService) ExtensionLoader.getExtensionLoader(DataStore.class)
            .getDefaultExtension().get(Constants.CONSUMER_SIDE, Integer.toString(url.getPort()));
    // 清楚线程池缓存
    ExtensionLoader.getExtensionLoader(DataStore.class)
            .getDefaultExtension().remove(Constants.CONSUMER_SIDE, Integer.toString(url.getPort()));
}
```

该构造函数中做了一些属性值的设置，并且做了打开客户端和连接服务器的操作。

#### 3.wrapChannelHandler

```
protected static ChannelHandler wrapChannelHandler(URL url, ChannelHandler handler) {
    // 加入线程名称
    url = ExecutorUtil.setThreadName(url, CLIENT_THREAD_POOL_NAME);
    // 设置使用的线程池类型
    url = url.addParameterIfAbsent(Constants.THREADPOOL_KEY, Constants.DEFAULT_CLIENT_THREADPOOL);
    // 包装
    return ChannelHandlers.wrap(handler, url);
}
```

该方法是包装通道处理器，设置使用的线程池类型是可缓存线程池。

#### 4.initConnectStatusCheckCommand

```
private synchronized void initConnectStatusCheckCommand() {
    //reconnect=false to close reconnect
    int reconnect = getReconnectParam(getUrl());
    // 有连接频率的值，并且当前没有连接任务
    if (reconnect > 0 && (reconnectExecutorFuture == null || reconnectExecutorFuture.isCancelled())) {
        Runnable connectStatusCheckCommand = new Runnable() {
            @Override
            public void run() {
                try {
                    if (!isConnected()) {
                        // 重连
                        connect();
                    } else {
                        // 记录最后一次重连的时间
                        lastConnectedTime = System.currentTimeMillis();
                    }
                } catch (Throwable t) {
                    String errorMsg = "client reconnect to " + getUrl().getAddress() + " find error . url: " + getUrl();
                    // wait registry sync provider list
                    if (System.currentTimeMillis() - lastConnectedTime > shutdown_timeout) {
                        // 如果之前没有打印过重连的误日志
                        if (!reconnect_error_log_flag.get()) {
                            reconnect_error_log_flag.set(true);
                            // 打印日志
                            logger.error(errorMsg, t);
                            return;
                        }
                    }
                    // 如果到达一次重连日志告警周期，则打印告警日志
                    if (reconnect_count.getAndIncrement() % reconnect_warning_period == 0) {
                        logger.warn(errorMsg, t);
                    }
                }
            }
        };
        // 开启重连定时任务
        reconnectExecutorFuture = reconnectExecutorService.scheduleWithFixedDelay(connectStatusCheckCommand, reconnect, reconnect, TimeUnit.MILLISECONDS);
    }
}
```

该方法是初始化重连线程，其中做了重连失败后的告警日志和错误日志打印策略。

#### 5.reconnect

```
@Override
public void reconnect() throws RemotingException {
    disconnect();
    connect();
}
```

单独放该方法是因为这是该类关注的重点。实现了客户端的重连逻辑。

#### 6.其他

connect、disconnect、close等方法都是调用了对应的抽象方法，而具体的逻辑需要看具体的子类如何去实现相关的抽象方法，这几个方法逻辑比较简单，我不在这里贴出源码，有兴趣可以看我的GitHub，地址文章末尾会给出。

### （四）AbstractChannel

该类是通道的抽象类，该类里面做的逻辑很简单，具体的发送消息逻辑在它 的子类中实现。

```
@Override
public void send(Object message, boolean sent) throws RemotingException {
    // 检测通道是否关闭
    if (isClosed()) {
        throw new RemotingException(this, "Failed to send message "
                + (message == null ? "" : message.getClass().getName()) + ":" + message
                + ", cause: Channel closed. channel: " + getLocalAddress() + " -> " + getRemoteAddress());
    }
}
```

可以看到send方法，其中只做了检测通道是否关闭的状态检测，没有实现具体的发送消息的逻辑。

### （五）ChannelHandlerDelegate

该类继承了ChannelHandler，从它的名字可以看出是ChannelHandler的代表，它就是作为装饰模式中的Component角色，后面讲到的AbstractChannelHandlerDelegate作为装饰模式中的Decorator角色。

```
public interface ChannelHandlerDelegate extends ChannelHandler {
    /**
     * 获得通道
     * @return
     */
    ChannelHandler getHandler();
}
```

### （六）AbstractChannelHandlerDelegate

属性：

```
protected ChannelHandler handler
```

该类实现了ChannelHandlerDelegate接口，并且有一个属性是ChannelHandler，上述已经说到这是装饰模式中的装饰角色，其中的所有实现方法都直接调用被装饰的handler属性的方法。

### （七）DecodeHandler

该类为解码处理器，继承了AbstractChannelHandlerDelegate，对接收到的消息进行解码，在父类处理接收消息的功能上叠加了解码功能。

我们来看看received方法：

```
@Override
public void received(Channel channel, Object message) throws RemotingException {
    // 如果是Decodeable类型的消息，则对整个消息解码
    if (message instanceof Decodeable) {
        decode(message);
    }

    // 如果是Request请求类型消息，则对请求中对请求数据解码
    if (message instanceof Request) {
        decode(((Request) message).getData());
    }

    // 如果是Response返回类型的消息，则对返回消息中对结果进行解码
    if (message instanceof Response) {
        decode(((Response) message).getResult());
    }

    // 继续将消息委托给handler，继续处理
    handler.received(channel, message);
}
```

可以看到做了三次判断，根据消息的不同会对消息的不同数据做解码。可以看到，这里用到装饰模式后，在处理消息的前面做了解码的处理，并且还能继续委托给handler来处理消息，通过组合做到了功能的叠加。

```
private void decode(Object message) {
    // 如果消息类型是Decodeable，进一步调用Decodeable的decode来解码
    if (message != null && message instanceof Decodeable) {
        try {
            ((Decodeable) message).decode();
            if (log.isDebugEnabled()) {
                log.debug("Decode decodeable message " + message.getClass().getName());
            }
        } catch (Throwable e) {
            if (log.isWarnEnabled()) {
                log.warn("Call Decodeable.decode failed: " + e.getMessage(), e);
            }
        } // ~ end of catch
    } // ~ end of if
} // ~ end of method decode
```

可以看到这是解析消息的逻辑，当消息是Decodeable类型，还会继续调用Decodeable的decode方法来进行解析。它的实现类后续会讲解到。

### （八）MultiMessageHandler

该类是多消息处理器的抽象类。同样继承了AbstractChannelHandlerDelegate类，我们来看看它的received方法：

```
@SuppressWarnings("unchecked")
@Override
public void received(Channel channel, Object message) throws RemotingException {
    // 当消息为多消息时 循环交给handler处理接收到当消息
    if (message instanceof MultiMessage) {
        MultiMessage list = (MultiMessage) message;
        for (Object obj : list) {
            handler.received(channel, obj);
        }
    } else {
        // 如果是单消息，就直接交给handler处理器
        handler.received(channel, message);
    }
}
```

逻辑很简单，当消息是多消息类型时，也就是一次性接收到多条消息的情况，循环去处理消息，当消息是单消息时候，直接交给handler去处理。

### （九）WrappedChannelHandler

该类跟AbstractChannelHandlerDelegate的作用类似，都是装饰模式中的装饰角色，其中的所有实现方法都直接调用被装饰的handler属性的方法，该类是为了添加线程池的功能，它的子类都是去关心哪些消息是需要分发到线程池的，哪些消息直接由I / O线程执行，现在版本有四种场景，也就是它的四个子类，下面我一一描述。

```
public WrappedChannelHandler(ChannelHandler handler, URL url) {
    this.handler = handler;
    this.url = url;
    // 创建线程池
    executor = (ExecutorService) ExtensionLoader.getExtensionLoader(ThreadPool.class).getAdaptiveExtension().getExecutor(url);

    // 设置组件的key
    String componentKey = Constants.EXECUTOR_SERVICE_COMPONENT_KEY;
    if (Constants.CONSUMER_SIDE.equalsIgnoreCase(url.getParameter(Constants.SIDE_KEY))) {
        componentKey = Constants.CONSUMER_SIDE;
    }
    // 获得dataStore实例
    DataStore dataStore = ExtensionLoader.getExtensionLoader(DataStore.class).getDefaultExtension();
    // 把线程池放到dataStore中缓存
    dataStore.put(componentKey, Integer.toString(url.getPort()), executor);
}
```

可以看到构造方法除了属性的填充以外，线程池是基于dubbo 的SPI Adaptive机制创建的，在dataStore中把线程池加进去， 该线程池就是AbstractClient 或 AbstractServer 从 DataStore 获得的线程池。

```
public ExecutorService getExecutorService() {
    // 首先返回的不是共享线程池，是该类的线程池
    ExecutorService cexecutor = executor;
    // 如果该类的线程池关闭或者为空，则返回的是共享线程池
    if (cexecutor == null || cexecutor.isShutdown()) {
        cexecutor = SHARED_EXECUTOR;
    }
    return cexecutor;
}
```

该方法是获得线程池的实例，不过该类里面有两个线程池，还加入了一个共享线程池，共享线程池优先级较低。

### （十）ExecutionChannelHandler

该类继承了WrappedChannelHandler，也是增强了功能，处理的是接收请求消息时，把请求消息分发到线程池，而除了请求消息以外，其他消息类型都直接通过I / O线程直接执行。

```
@Override
public void received(Channel channel, Object message) throws RemotingException {
    // 获得线程池实例
    ExecutorService cexecutor = getExecutorService();
    // 如果消息是request类型，才会分发到线程池，其他消息，如响应，连接，断开连接，心跳将由I / O线程直接执行。
    if (message instanceof Request) {
        try {
            // 把请求消息分发到线程池
            cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            // FIXME: when the thread pool is full, SERVER_THREADPOOL_EXHAUSTED_ERROR cannot return properly,
            // therefore the consumer side has to wait until gets timeout. This is a temporary solution to prevent
            // this scenario from happening, but a better solution should be considered later.
            // 当线程池满了，SERVER_THREADPOOL_EXHAUSTED_ERROR错误无法正常返回
            // 因此消费者方必须等到超时。这是一种预防的临时解决方案，所以这里直接返回该错误
            if (t instanceof RejectedExecutionException) {
                Request request = (Request) message;
                if (request.isTwoWay()) {
                    String msg = "Server side(" + url.getIp() + "," + url.getPort()
                            + ") thread pool is exhausted, detail msg:" + t.getMessage();
                    Response response = new Response(request.getId(), request.getVersion());
                    response.setStatus(Response.SERVER_THREADPOOL_EXHAUSTED_ERROR);
                    response.setErrorMessage(msg);
                    channel.send(response);
                    return;
                }
            }
            throw new ExecutionException(message, channel, getClass() + " error when process received event.", t);
        }
    } else {
        // 如果消息不是request类型，则直接处理
        handler.received(channel, message);
    }
}
```

上述就可以都看到对于请求消息的处理，其中有个打补丁的方式是当线程池满了的时候，消费者只能等待请求超时，所以这里直接返回线程池满的错误。

### （十一）AllChannelHandler

该类也继承了WrappedChannelHandler，也是为了增强功能，处理的是连接、断开连接、捕获异常以及接收到的所有消息都分发到线程池。

```
@Override
public void connected(Channel channel) throws RemotingException {
    ExecutorService cexecutor = getExecutorService();
    try {
        // 把连接操作分发到线程池处理
        cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
    } catch (Throwable t) {
        throw new ExecutionException("connect event", channel, getClass() + " error when process connected event .", t);
    }
}

@Override
public void disconnected(Channel channel) throws RemotingException {
    ExecutorService cexecutor = getExecutorService();
    try {
        // 把断开连接操作分发到线程池处理
        cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
    } catch (Throwable t) {
        throw new ExecutionException("disconnect event", channel, getClass() + " error when process disconnected event .", t);
    }
}

@Override
public void received(Channel channel, Object message) throws RemotingException {
    ExecutorService cexecutor = getExecutorService();
    try {
        // 把所有消息分发到线程池处理
        cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
    } catch (Throwable t) {
        //TODO A temporary solution to the problem that the exception information can not be sent to the opposite end after the thread pool is full. Need a refactoring
        //fix The thread pool is full, refuses to call, does not return, and causes the consumer to wait for time out
        // 这里处理线程池满的问题，只有在请求时候会出现。
        //复线程池已满，拒绝调用，不返回，并导致使用者等待超时
       if(message instanceof Request && t instanceof RejectedExecutionException){
          Request request = (Request)message;
          if(request.isTwoWay()){
             String msg = "Server side(" + url.getIp() + "," + url.getPort() + ") threadpool is exhausted ,detail msg:" + t.getMessage();
             Response response = new Response(request.getId(), request.getVersion());
             response.setStatus(Response.SERVER_THREADPOOL_EXHAUSTED_ERROR);
             response.setErrorMessage(msg);
             channel.send(response);
             return;
          }
       }
        throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
    }
}

@Override
public void caught(Channel channel, Throwable exception) throws RemotingException {
    ExecutorService cexecutor = getExecutorService();
    try {
        // 把捕获异常作分发到线程池处理
        cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
    } catch (Throwable t) {
        throw new ExecutionException("caught event", channel, getClass() + " error when process caught event .", t);
    }
}
```

可以看到，所有操作以及消息都分到到线程池中。并且注意操作不同，传入的状态也不同。

### （十二）ConnectionOrderedChannelHandler

该类也是继承了WrappedChannelHandler，增强功能，该类是把连接、取消连接以及接收到的消息都分发到线程池，但是不同的是，该类自己创建了一个跟连接相关的线程池，把连接操作和断开连接操分发到该线程池，而接收到的消息则分发到WrappedChannelHandler的线程池中。来看看具体的实现。

```
/**
 * 连接线程池
 */
protected final ThreadPoolExecutor connectionExecutor;
/**
 * 连接队列大小限制
 */
private final int queuewarninglimit;

public ConnectionOrderedChannelHandler(ChannelHandler handler, URL url) {
    super(handler, url);
    // 获得线程名，默认是Dubbo
    String threadName = url.getParameter(Constants.THREAD_NAME_KEY, Constants.DEFAULT_THREAD_NAME);
    // 创建连接线程池
    connectionExecutor = new ThreadPoolExecutor(1, 1,
            0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>(url.getPositiveParameter(Constants.CONNECT_QUEUE_CAPACITY, Integer.MAX_VALUE)),
            new NamedThreadFactory(threadName, true),
            new AbortPolicyWithReport(threadName, url)
    );  // FIXME There's no place to release connectionExecutor!
    // 设置工作队列限制，默认是1000
    queuewarninglimit = url.getParameter(Constants.CONNECT_QUEUE_WARNING_SIZE, Constants.DEFAULT_CONNECT_QUEUE_WARNING_SIZE);
}
```

可以属性中有一个连接线程池，看到在构造函数里创建了该线程池，而queuewarninglimit是用来限制连接线程池的工作队列长度，比较简单。来看看连接和断开连接到逻辑。

```
@Override
public void connected(Channel channel) throws RemotingException {
    try {
        // 核对工作队列长度
        checkQueueLength();
        // 分发连接操作
        connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
    } catch (Throwable t) {
        throw new ExecutionException("connect event", channel, getClass() + " error when process connected event .", t);
    }
}

@Override
public void disconnected(Channel channel) throws RemotingException {
    try {
        // 核对工作队列长度
        checkQueueLength();
        // 分发断开连接操作
        connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
    } catch (Throwable t) {
        throw new ExecutionException("disconnected event", channel, getClass() + " error when process disconnected event .", t);
    }
}
```

可以看到，这两个操作都是分发到连接线程池connectionExecutor中，和AllChannelHandle类r中的分发的线程池不是同一个。而ConnectionOrderedChannelHandler的received方法跟AllChannelHandle一样，我就不贴出来。

### （十三）MessageOnlyChannelHandler

该类也是继承了WrappedChannelHandler，是WrappedChannelHandler的最后一个子类，也是增强功能，不过该类只是处理了所有的消息分发到线程池。可以看到源码，比较简单：

```
@Override
public void received(Channel channel, Object message) throws RemotingException {
    // 获得线程池实例
    ExecutorService cexecutor = getExecutorService();
    try {
        // 把消息分发到线程池
        cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
    } catch (Throwable t) {
        throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
    }
}
```

下面我讲讲解五种线程池的调度策略，也就是我在[《dubbo源码解析（八）远程通信——开篇》](https://segmentfault.com/a/1190000017274525)中提到的Dispatcher接口的五种实现，分别是AllDispatcher、DirectDispatcher、MessageOnlyDispatcher、ExecutionDispatcher、ConnectionOrderedDispatcher。

### （十四）AllDispatcher

```
public class AllDispatcher implements Dispatcher {

    public static final String NAME = "all";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 线程池调度方法：任何消息以及操作都分发到线程池中
        return new AllChannelHandler(handler, url);
    }

}
```

对照着上述讲到的AllChannelHandler，是不是很清晰这种线程池的调度方法。并且该调度方法是默认的调度方法。

### （十五）ConnectionOrderedDispatcher

```
public class ConnectionOrderedDispatcher implements Dispatcher {

    public static final String NAME = "connection";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 线程池调度方法：连接、断开连接分发到到线程池和其他消息分发到线程池不是同一个
        return new ConnectionOrderedChannelHandler(handler, url);
    }

}
```

对照上述讲到的ConnectionOrderedChannelHandler，也很清晰该线程池调度方法。

### （十六）DirectDispatcher

```
public class DirectDispatcher implements Dispatcher {

    public static final String NAME = "direct";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 直接处理消息，不分发到线程池
        return handler;
    }

}
```

该线程池调度方法是不调度线程池，直接执行。

### （十七）ExecutionDispatcher

```
public class ExecutionDispatcher implements Dispatcher {

    public static final String NAME = "execution";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 线程池调度方法：只有请求消息分发到线程池，其他都直接执行
        return new ExecutionChannelHandler(handler, url);
    }

}
```

对照着上述的ExecutionChannelHandler讲解，也可以很清晰的看出该线程池调度策略。

### （十八）MessageOnlyDispatcher

```
public class MessageOnlyDispatcher implements Dispatcher {

    public static final String NAME = "message";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 只要是接收到的消息，都分发到线程池
        return new MessageOnlyChannelHandler(handler, url);
    }

}
```

对照着上述讲到的MessageOnlyChannelHandler，可以很清晰该线程池调度策略。

### （十九）ChannelHandlers

该类是通道处理器工厂，会对传入的handler进行一次包装，无论是Client还是Server都会做这样的处理，也就是做了一些功能上的增强，就像上述我说到的装饰模式中的那些功能。

我们来看看源码：

```
public static ChannelHandler wrap(ChannelHandler handler, URL url) {
    return ChannelHandlers.getInstance().wrapInternal(handler, url);
}

protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
    // 调用了多消息处理器，对心跳消息进行了功能加强
    return new MultiMessageHandler(new HeartbeatHandler(ExtensionLoader.getExtensionLoader(Dispatcher.class)
            .getAdaptiveExtension().dispatch(handler, url)));
}
```

最关键的是这两个方法，看第二个方法，其实就是包装了MultiMessageHandler功能，增加了多消息处理的功能，以及对心跳消息做了功能增强。

### （二十）AbstractCodec

实现 Codec**2** 接口，，其中实现了一些编解码的公共逻辑。

#### 1.checkPayload

```
protected static void checkPayload(Channel channel, long size) throws IOException {
    // 默认长度
    int payload = Constants.DEFAULT_PAYLOAD;
    if (channel != null && channel.getUrl() != null) {
        // 优先从url中获得消息长度配置，如果没有则用默认长度
        payload = channel.getUrl().getParameter(Constants.PAYLOAD_KEY, Constants.DEFAULT_PAYLOAD);
    }
    // 如果消息长度过长，则报错
    if (payload > 0 && size > payload) {
        ExceedPayloadLimitException e = new ExceedPayloadLimitException("Data length too large: " + size + ", max payload: " + payload + ", channel: " + channel);
        logger.error(e);
        throw e;
    }
}
```

该方法是检验消息长度。

#### 2.getSerialization

```
protected Serialization getSerialization(Channel channel) {
    return CodecSupport.getSerialization(channel.getUrl());
}
```

该方法是获得序列化对象。

#### 3.isClientSide

```
protected boolean isClientSide(Channel channel) {
    // 获得是side对应的value
    String side = (String) channel.getAttribute(Constants.SIDE_KEY);
    if ("client".equals(side)) {
        return true;
    } else if ("server".equals(side)) {
        return false;
    } else {
        InetSocketAddress address = channel.getRemoteAddress();
        URL url = channel.getUrl();
        // 判断url的主机地址是否和远程地址一样，如果是，则判断为client，如果不是，则判断为server
        boolean client = url.getPort() == address.getPort()
                && NetUtils.filterLocalHost(url.getIp()).equals(
                NetUtils.filterLocalHost(address.getAddress()
                        .getHostAddress()));
        // 把value设置进去
        channel.setAttribute(Constants.SIDE_KEY, client ? "client"
                : "server");
        return client;
    }
}
```

该方法是判断是否为客户端侧的通道。

#### 4.isServerSide

```
protected boolean isServerSide(Channel channel) {
    return !isClientSide(channel);
}
```

该方法是判断是否为服务端侧的通道。

### （二十一）TransportCodec

该类是传输编解码器，使用 Serialization 进行序列化/反序列化，直接编解码。关于序列化为会在后续文章中介绍。

```
@Override
public void encode(Channel channel, ChannelBuffer buffer, Object message) throws IOException {
    // 获得序列化的 ObjectOutput 对象
    OutputStream output = new ChannelBufferOutputStream(buffer);
    ObjectOutput objectOutput = getSerialization(channel).serialize(channel.getUrl(), output);
    // 写入 ObjectOutput
    encodeData(channel, objectOutput, message);
    objectOutput.flushBuffer();
    // 释放
    if (objectOutput instanceof Cleanable) {
        ((Cleanable) objectOutput).cleanup();
    }
}

@Override
public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
    // 获得反序列化的 ObjectInput 对象
    InputStream input = new ChannelBufferInputStream(buffer);
    ObjectInput objectInput = getSerialization(channel).deserialize(channel.getUrl(), input);
    // 读取 ObjectInput
    Object object = decodeData(channel, objectInput);
    // 释放
    if (objectInput instanceof Cleanable) {
        ((Cleanable) objectInput).cleanup();
    }
    return object;
}
```

该类关键方法就是编码和解码，比较好理解，直接进行了序列化和反序列化。

### （二十二）CodecAdapter

该类是Codec 的适配器，用到了适配器模式，把Codec适配成Codec2。将Codec的编码和解码方法都适配成Codec2。比如很多时候都只能用Codec2的编解码器，但是有的时候需要用Codec，但是不能满足导致只能加入适配器来完成使用。

```
@Override
public void encode(Channel channel, ChannelBuffer buffer, Object message)
        throws IOException {
    UnsafeByteArrayOutputStream os = new UnsafeByteArrayOutputStream(1024);
    // 调用旧的编解码器的编码
    codec.encode(channel, os, message);
    buffer.writeBytes(os.toByteArray());
}

@Override
public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
    byte[] bytes = new byte[buffer.readableBytes()];
    int savedReaderIndex = buffer.readerIndex();
    buffer.readBytes(bytes);
    UnsafeByteArrayInputStream is = new UnsafeByteArrayInputStream(bytes);
    // 调用旧的编解码器的解码
    Object result = codec.decode(channel, is);
    buffer.readerIndex(savedReaderIndex + is.position());
    return result == Codec.NEED_MORE_INPUT ? DecodeResult.NEED_MORE_INPUT : result;
}
```

可以看到，在编码和解码的方法中都调用了codec的方法。

### （二十三）ChannelDelegate、ServerDelegate、ClientDelegate

ChannelDelegate实现类Channel，ServerDelegate实现了Server，ClientDelegate实现了Client，都用到了装饰模式，都作为装饰模式中的装饰角色，所以类中的所有实现方法都调用了属性的方法。具体代码就不贴了，朋友们可以自行查看。

### （二十四）ChannelHandlerAdapter

该类实现了ChannelHandler接口，是通道处理器适配类，该类中所有实现方法都是空的，所有想实现ChannelHandler接口的类可以直接继承该类，选择需要实现的方法进行实现，不需要实现ChannelHandler接口中所有方法。

### （二十五）ChannelHandlerDispatcher

该类是通道处理器调度器，其中缓存了所有通道处理器，有一个通道处理器集合。并且每个操作都会去遍历该集合，执行相应的操作，例如：

```
@Override
public void connected(Channel channel) {
    // 遍历通道处理器集合
    for (ChannelHandler listener : channelHandlers) {
        try {
            // 连接
            listener.connected(channel);
        } catch (Throwable t) {
            logger.error(t.getMessage(), t);
        }
    }
}
```

### （二十六）CodecSupport

该类是编解码工具类，提供查询 Serialization 的功能。

```
/**
 * 序列化对象集合 key为序列化类型编号
 */
private static Map<Byte, Serialization> ID_SERIALIZATION_MAP = new HashMap<Byte, Serialization>();
/**
 * 序列化扩展名集合 key为序列化类型编号 value为序列化扩展名
 */
private static Map<Byte, String> ID_SERIALIZATIONNAME_MAP = new HashMap<Byte, String>();

static {
    // 利用dubbo 的SPI机制获得序列化扩展名
    Set<String> supportedExtensions = ExtensionLoader.getExtensionLoader(Serialization.class).getSupportedExtensions();
    for (String name : supportedExtensions) {
        // 获得相应扩展名的序列化实现
        Serialization serialization = ExtensionLoader.getExtensionLoader(Serialization.class).getExtension(name);
        byte idByte = serialization.getContentTypeId();
        if (ID_SERIALIZATION_MAP.containsKey(idByte)) {
            logger.error("Serialization extension " + serialization.getClass().getName()
                    + " has duplicate id to Serialization extension "
                    + ID_SERIALIZATION_MAP.get(idByte).getClass().getName()
                    + ", ignore this Serialization extension");
            continue;
        }
        // 缓存序列化实现
        ID_SERIALIZATION_MAP.put(idByte, serialization);
        // 缓存序列化编号和扩展名
        ID_SERIALIZATIONNAME_MAP.put(idByte, name);
    }
}
```

可以看到该类中缓存了所有的序列化对象和序列化扩展名。可以从中拿到Serialization。

### （二十七）ExceedPayloadLimitException

该类是消息长度限制异常。

```
public class ExceedPayloadLimitException extends IOException {
    private static final long serialVersionUID = -1112322085391551410L;

    public ExceedPayloadLimitException(String message) {
        super(message);
    }
}
```

# 远程通讯——Exchange层

> 目标：介绍Exchange层的相关设计和逻辑、介绍dubbo-remoting-api中的exchange包内的源码解析。

## 前言

上一篇文章我讲的是dubbo框架设计中Transport层，这篇文章我要讲的是它的上一层Exchange层，也就是信息交换层。官方文档对这一层的解释是封装请求响应模式，同步转异步，以 Request, Response为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer。

这一层的设计意图是什么？它应该算是在信息传输层上又做了部分装饰，为了适应rpc调用的一些需求，比如rpc调用中一次请求只关心它所对应的响应，这个时候只是一个message消息传输过来，是无法区分这是新的请求还是上一个请求的响应，这种类似于幂等性的问题以及rpc异步处理返回结果、内置事件等特性都是在Transport层无法解决满足的，所有在Exchange层讲message分成了request和response两种类型，并且在这两个模型上增加一些系统字段来处理问题。具体我会在下面讲到。而dubbo把一条消息分为了协议头和内容两部分：协议头包括系统字段，例如编号等，内容包括具体请求的参数和响应的结果等。在exchange层中大量逻辑都是基于协议头的。

现在对这一层的设计意图大致应该有所了解了吧，现在来看看exchange的类图：

![exchange类图](https://segmentfault.com/img/remote/1460000017467346)

我讲解的顺序还是按照类图从上而下，分块讲解，忽略绿色的test类。

## 源码解析

### （一）ExchangeChannel

```
public interface ExchangeChannel extends Channel {

    ResponseFuture request(Object request) throws RemotingException;

    ResponseFuture request(Object request, int timeout) throws RemotingException;
    
    ExchangeHandler getExchangeHandler();
    
    @Override
    void close(int timeout);

}
```

该接口是信息交换通道接口，有四个方法，前两个是发送请求消息，区别就是第二个发送请求有超时的参数，getExchangeHandler方法就是返回一个信息交换处理器，第四个是需要覆写父类的方法。

### （二）HeaderExchangeChannel

该类实现了ExchangeChannel，是基于协议头的信息交换通道。

#### 1.属性

```
private static final Logger logger = LoggerFactory.getLogger(HeaderExchangeChannel.class);

/**
 * 通道的key值
 */
private static final String CHANNEL_KEY = HeaderExchangeChannel.class.getName() + ".CHANNEL";

/**
 * 通道
 */
private final Channel channel;

/**
 * 是否关闭
 */
private volatile boolean closed = false;
```

上述属性比较简单，还是放一下这个类的属性是因为该类中有channel属性，也就是说HeaderExchangeChannel是Channel的装饰器，每个实现方法都会调用channel的方法。

#### 2.静态方法

```
static HeaderExchangeChannel getOrAddChannel(Channel ch) {
    if (ch == null) {
        return null;
    }
    // 获得通道中的HeaderExchangeChannel
    HeaderExchangeChannel ret = (HeaderExchangeChannel) ch.getAttribute(CHANNEL_KEY);
    if (ret == null) {
        // 创建一个HeaderExchangeChannel实例
        ret = new HeaderExchangeChannel(ch);
        // 如果通道连接
        if (ch.isConnected()) {
            // 加入属性值
            ch.setAttribute(CHANNEL_KEY, ret);
        }
    }
    return ret;
}

static void removeChannelIfDisconnected(Channel ch) {
    // 如果通道断开连接
    if (ch != null && !ch.isConnected()) {
        // 移除属性值
        ch.removeAttribute(CHANNEL_KEY);
    }
}
```

该静态方法做了HeaderExchangeChannel的创建和销毁，并且生命周期随channel销毁而销毁。

#### 3.send

```
@Override
public void send(Object message) throws RemotingException {
    send(message, getUrl().getParameter(Constants.SENT_KEY, false));
}

@Override
public void send(Object message, boolean sent) throws RemotingException {
    // 如果通道关闭，抛出异常
    if (closed) {
        throw new RemotingException(this.getLocalAddress(), null, "Failed to send message " + message + ", cause: The channel " + this + " is closed!");
    }
    // 判断消息的类型
    if (message instanceof Request
            || message instanceof Response
            || message instanceof String) {
        // 发送消息
        channel.send(message, sent);
    } else {
        // 新建一个request实例
        Request request = new Request();
        // 设置信息的版本
        request.setVersion(Version.getProtocolVersion());
        // 该请求不需要响应
        request.setTwoWay(false);
        // 把消息传入
        request.setData(message);
        // 发送消息
        channel.send(request, sent);
    }
}
```

该方法是在channel的send方法上加上了request和response模型，最后再调用channel.send，起到了装饰器的作用。

#### 4.request

```
@Override
public ResponseFuture request(Object request) throws RemotingException {
    return request(request, channel.getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT));
}

@Override
public ResponseFuture request(Object request, int timeout) throws RemotingException {
    // 如果通道关闭，则抛出异常
    if (closed) {
        throw new RemotingException(this.getLocalAddress(), null, "Failed to send request " + request + ", cause: The channel " + this + " is closed!");
    }
    // create request.创建请求
    Request req = new Request();
    // 设置版本号
    req.setVersion(Version.getProtocolVersion());
    // 设置需要响应
    req.setTwoWay(true);
    // 把请求数据传入
    req.setData(request);
    // 创建DefaultFuture对象，可以从future中主动获得请求对应的响应信息
    DefaultFuture future = new DefaultFuture(channel, req, timeout);
    try {
        // 发送请求消息
        channel.send(req);
    } catch (RemotingException e) {
        future.cancel();
        throw e;
    }
    return future;
}
```

该方法是请求方法，用Request模型把请求内容装饰起来，然后发送一个Request类型的消息，并且返回DefaultFuture实例，DefaultFuture我会在后面讲到。

cloes方法也重写了，我就不再多说，因为比较简单，没有重点，其他方法都是直接调用channel属性的方法。

### （三）ExchangeClient

该接口继承了Client和ExchangeChannel，是信息交换客户端接口，其中没有定义多余的方法。

### （四）HeaderExchangeClient

　该类实现了ExchangeClient接口，是基于协议头的信息交互客户端类，同样它是Client、Channel的适配器。在该类的源码中可以看到所有的实现方法都是调用了client和channel属性的方法。该类主要的作用就是增加了心跳功能，为什么要增加心跳功能呢，对于长连接，一些拔网线等物理层的断开，会导致TCP的FIN消息来不及发送，对方收不到断开事件，那么就需要用到发送心跳包来检测连接是否断开。consumer和provider断开，处理措施不一样，会分别做出重连和关闭通道的操作。

#### 1.属性

```
private static final Logger logger = LoggerFactory.getLogger(HeaderExchangeClient.class);

/**
 * 定时器线程池
 */
private static final ScheduledThreadPoolExecutor scheduled = new ScheduledThreadPoolExecutor(2, new NamedThreadFactory("dubbo-remoting-client-heartbeat", true));
/**
 * 客户端
 */
private final Client client;
/**
 * 信息交换通道
 */
private final ExchangeChannel channel;
// heartbeat timer
/**
 * 心跳定时器
 */
private ScheduledFuture<?> heartbeatTimer;
// heartbeat(ms), default value is 0 , won't execute a heartbeat.
/**
 * 心跳周期，间隔多久发送心跳消息检测一次
 */
private int heartbeat;
/**
 * 心跳超时时间
 */
private int heartbeatTimeout;
```

该类的属性除了需要适配的属性外，其他都是跟心跳相关属性。

#### 2.构造函数

```
public HeaderExchangeClient(Client client, boolean needHeartbeat) {
    if (client == null) {
        throw new IllegalArgumentException("client == null");
    }
    this.client = client;
    // 创建信息交换通道
    this.channel = new HeaderExchangeChannel(client);
    // 获得dubbo版本
    String dubbo = client.getUrl().getParameter(Constants.DUBBO_VERSION_KEY);
    //获得心跳周期配置，如果没有配置，并且dubbo是1.0版本的，则这只为1分钟，否则设置为0
    this.heartbeat = client.getUrl().getParameter(Constants.HEARTBEAT_KEY, dubbo != null && dubbo.startsWith("1.0.") ? Constants.DEFAULT_HEARTBEAT : 0);
    // 获得心跳超时配置，默认是心跳周期的三倍
    this.heartbeatTimeout = client.getUrl().getParameter(Constants.HEARTBEAT_TIMEOUT_KEY, heartbeat * 3);
    // 如果心跳超时时间小于心跳周期的两倍，则抛出异常
    if (heartbeatTimeout < heartbeat * 2) {
        throw new IllegalStateException("heartbeatTimeout < heartbeatInterval * 2");
    }
    if (needHeartbeat) {
        // 开启心跳
        startHeartbeatTimer();
    }
}
```

构造函数就是对一些属性初始化设置，优先从url中获取。心跳超时时间小于心跳周期的两倍就抛出异常，意思就是至少重试两次心跳检测。

#### 3.startHeartbeatTimer

```
private void startHeartbeatTimer() {
    // 停止现有的心跳线程
    stopHeartbeatTimer();
    // 如果需要心跳
    if (heartbeat > 0) {
        // 创建心跳定时器
        heartbeatTimer = scheduled.scheduleWithFixedDelay(
                // 新建一个心跳线程
                new HeartBeatTask(new HeartBeatTask.ChannelProvider() {
                    @Override
                    public Collection<Channel> getChannels() {
                        // 返回一个只包含HeaderExchangeClient对象的不可变列表
                        return Collections.<Channel>singletonList(HeaderExchangeClient.this);
                    }
                }, heartbeat, heartbeatTimeout),
                heartbeat, heartbeat, TimeUnit.MILLISECONDS);
    }
}
```

该方法就是开启心跳。利用心跳定时器来做到定时检测心跳。因为这是信息交换客户端类，所有这里的只是返回包含HeaderExchangeClient对象的不可变列表，因为客户端跟channel是一一对应的，只有这一个该客户端本身的channel需要心跳。

#### 4.stopHeartbeatTimer

```
private void stopHeartbeatTimer() {
    if (heartbeatTimer != null && !heartbeatTimer.isCancelled()) {
        try {
            // 取消定时器
            heartbeatTimer.cancel(true);
            // 取消大量已排队任务，用于回收空间
            scheduled.purge();
        } catch (Throwable e) {
            if (logger.isWarnEnabled()) {
                logger.warn(e.getMessage(), e);
            }
        }
    }
    heartbeatTimer = null;
}
```

该方法是停止现有心跳，也就是停止定时器，释放空间。

其他方法都是调用channel和client属性的方法。

### （五）HeartBeatTask

该类实现了Runnable接口，实现的是心跳任务，里面包含了核心的心跳策略。

#### 1.属性

```
/**
 * 通道管理
 */
private ChannelProvider channelProvider;

/**
 * 心跳间隔 单位：ms
 */
private int heartbeat;

/**
 * 心跳超时时间 单位：ms
 */
private int heartbeatTimeout;
```

后两个属性跟HeaderExchangeClient中的属性含义一样，第一个是该类自己内部的一个接口：

```
interface ChannelProvider {
    // 获得所有的通道集合，需要心跳的通道数组
    Collection<Channel> getChannels();
}
```

该接口就定义了一个方法，获得需要心跳的通道集合。可想而知，会对集合内的通道都做心跳检测。

#### 2.run

```
@Override
public void run() {
    try {
        long now = System.currentTimeMillis();
        // 遍历所有通道
        for (Channel channel : channelProvider.getChannels()) {
            // 如果通道关闭了，则跳过
            if (channel.isClosed()) {
                continue;
            }
            try {
                // 最后一次接收到消息的时间戳
                Long lastRead = (Long) channel.getAttribute(
                        HeaderExchangeHandler.KEY_READ_TIMESTAMP);
                // 最后一次发送消息的时间戳
                Long lastWrite = (Long) channel.getAttribute(
                        HeaderExchangeHandler.KEY_WRITE_TIMESTAMP);
                // 如果最后一次接收或者发送消息到时间到现在的时间间隔超过了心跳间隔时间
                if ((lastRead != null && now - lastRead > heartbeat)
                        || (lastWrite != null && now - lastWrite > heartbeat)) {
                    // 创建一个request
                    Request req = new Request();
                    // 设置版本号
                    req.setVersion(Version.getProtocolVersion());
                    // 设置需要得到响应
                    req.setTwoWay(true);
                    // 设置事件类型，为心跳事件
                    req.setEvent(Request.HEARTBEAT_EVENT);
                    // 发送心跳请求
                    channel.send(req);
                    if (logger.isDebugEnabled()) {
                        logger.debug("Send heartbeat to remote channel " + channel.getRemoteAddress()
                                + ", cause: The channel has no data-transmission exceeds a heartbeat period: " + heartbeat + "ms");
                    }
                }
                // 如果最后一次接收消息的时间到现在已经超过了超时时间
                if (lastRead != null && now - lastRead > heartbeatTimeout) {
                    logger.warn("Close channel " + channel
                            + ", because heartbeat read idle time out: " + heartbeatTimeout + "ms");
                    // 如果该通道是客户端，也就是请求的服务器挂掉了，客户端尝试重连服务器
                    if (channel instanceof Client) {
                        try {
                            // 重新连接服务器
                            ((Client) channel).reconnect();
                        } catch (Exception e) {
                            //do nothing
                        }
                    } else {
                        // 如果不是客户端，也就是是服务端返回响应给客户端，但是客户端挂掉了，则服务端关闭客户端连接
                        channel.close();
                    }
                }
            } catch (Throwable t) {
                logger.warn("Exception when heartbeat to remote channel " + channel.getRemoteAddress(), t);
            }
        }
    } catch (Throwable t) {
        logger.warn("Unhandled exception when heartbeat, cause: " + t.getMessage(), t);
    }
}
```

该方法中是心跳机制的核心逻辑。注意以下几个点：

1. 如果需要心跳的通道本身如果关闭了，那么跳过，不添加心跳机制。
2. 无论是接收消息还是发送消息，只要超过了设置的心跳间隔，就发送心跳消息来测试是否断开
3. 如果最后一次接收到消息到到现在已经超过了心跳超时时间，那就认定对方的确断开，分两种情况来处理对方断开的情况。分别是服务端断开，客户端重连以及客户端断开，服务端断开这个客户端的连接。，这里要好好品味一下谁是发送方，谁在等谁的响应，苦苦没有等到。

### （六）ResponseFuture

```
public interface ResponseFuture {

    Object get() throws RemotingException;

    Object get(int timeoutInMillis) throws RemotingException;

    void setCallback(ResponseCallback callback);

    boolean isDone();

}
```

该接口是响应future接口，该接口的设计意图跟java.util.concurrent.Future很类似。发送出去的消息，泼出去的水，只有等到对方主动响应才能得到结果，但是请求方需要去主动回去该请求的结果，就显得有些艰难，所有产生了这样一个接口，它能够获取任务执行结果、可以核对请求消息是否被响应，还能设置回调来支持异步。

### （七）DefaultFuture

该类实现了ResponseFuture接口，其中封装了处理响应的逻辑。你可以把DefaultFuture看成是一个中介，买房和卖房都通过这个中介进行沟通，中介拥有着买房者的信息request和卖房者的信息response，并且促成他们之间的买卖。

#### 1.属性

```
private static final Logger logger = LoggerFactory.getLogger(DefaultFuture.class);

/**
 * 通道集合
 */
private static final Map<Long, Channel> CHANNELS = new ConcurrentHashMap<Long, Channel>();

/**
 * Future集合，key为请求编号
 */
private static final Map<Long, DefaultFuture> FUTURES = new ConcurrentHashMap<Long, DefaultFuture>();

// invoke id.
/**
 * 请求编号
 */
private final long id;
/**
 * 通道
 */
private final Channel channel;
/**
 * 请求
 */
private final Request request;
/**
 * 超时
 */
private final int timeout;
/**
 * 锁
 */
private final Lock lock = new ReentrantLock();
/**
 * 完成情况，控制多线程的休眠与唤醒
 */
private final Condition done = lock.newCondition();
/**
 * 创建开始时间
 */
private final long start = System.currentTimeMillis();
/**
 * 发送请求时间
 */
private volatile long sent;
/**
 * 响应
 */
private volatile Response response;
/**
 * 回调
 */
private volatile ResponseCallback callback;
```

可以看到，该类的属性包含了request、response、channel三个实例，在该类中，把请求和响应通过唯一的id一一对应起来。做到异步处理返回结果时能给准确的返回给对应的请求。可以看到属性中有两个集合，分别是通道集合和future集合，也就是该类本身也是所有 DefaultFuture 的管理容器。

#### 2.构造函数

```
public DefaultFuture(Channel channel, Request request, int timeout) {
    this.channel = channel;
    this.request = request;
    // 设置请求编号
    this.id = request.getId();
    this.timeout = timeout > 0 ? timeout : channel.getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
    // put into waiting map.，加入到等待集合中
    FUTURES.put(id, this);
    CHANNELS.put(id, channel);
}
```

构造函数比较简单，每一个DefaultFuture实例都跟每一个请求一一对应，被存入到集合中管理起来。

#### 3.closeChannel

```
public static void closeChannel(Channel channel) {
    // 遍历通道集合
    for (long id : CHANNELS.keySet()) {
        if (channel.equals(CHANNELS.get(id))) {
            // 通过请求id获得future
            DefaultFuture future = getFuture(id);
            if (future != null && !future.isDone()) {
                // 创建一个关闭通道的响应
                Response disconnectResponse = new Response(future.getId());
                disconnectResponse.setStatus(Response.CHANNEL_INACTIVE);
                disconnectResponse.setErrorMessage("Channel " +
                        channel +
                        " is inactive. Directly return the unFinished request : " +
                        future.getRequest());
                // 接收该关闭通道并且请求未完成的响应
                DefaultFuture.received(channel, disconnectResponse);
            }
        }
    }
}
```

该方法是关闭不活跃的通道，并且返回请求未完成。也就是关闭指定channel的请求，返回的是请求未完成。

#### 4.received

```
public static void received(Channel channel, Response response) {
    try {
        // future集合中移除该请求的future，（响应id和请求id一一对应的）
        DefaultFuture future = FUTURES.remove(response.getId());
        if (future != null) {
            // 接收响应结果
            future.doReceived(response);
        } else {
            logger.warn("The timeout response finally returned at "
                    + (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").format(new Date()))
                    + ", response " + response
                    + (channel == null ? "" : ", channel: " + channel.getLocalAddress()
                    + " -> " + channel.getRemoteAddress()));
        }
    } finally {
        // 通道集合移除该请求对应的通道，代表着这一次请求结束
        CHANNELS.remove(response.getId());
    }
}
```

该方法是接收响应，也就是某个请求得到了响应，那么代表这次请求任务完成，所有需要把future从集合中移除。具体的接收响应结果在doReceived方法中实现。

#### 5.doReceived

```
private void doReceived(Response res) {
    // 获得锁
    lock.lock();
    try {
        // 设置响应
        response = res;
        if (done != null) {
            // 唤醒等待
            done.signal();
        }
    } finally {
        // 释放锁
        lock.unlock();
    }
    if (callback != null) {
        // 执行回调
        invokeCallback(callback);
    }
}
```

可以看到，当接收到响应后，会把等待的线程唤醒，然后执行回调来处理该响应结果。

#### 6.invokeCallback

```
private void invokeCallback(ResponseCallback c) {
    ResponseCallback callbackCopy = c;
    if (callbackCopy == null) {
        throw new NullPointerException("callback cannot be null.");
    }
    c = null;
    Response res = response;
    if (res == null) {
        throw new IllegalStateException("response cannot be null. url:" + channel.getUrl());
    }

    // 如果响应成功，返回码是20
    if (res.getStatus() == Response.OK) {
        try {
            // 使用响应结果执行 完成 后的逻辑
            callbackCopy.done(res.getResult());
        } catch (Exception e) {
            logger.error("callback invoke error .reasult:" + res.getResult() + ",url:" + channel.getUrl(), e);
        }
        //超时，回调处理成超时异常
    } else if (res.getStatus() == Response.CLIENT_TIMEOUT || res.getStatus() == Response.SERVER_TIMEOUT) {
        try {
            TimeoutException te = new TimeoutException(res.getStatus() == Response.SERVER_TIMEOUT, channel, res.getErrorMessage());
            // 回调处理异常
            callbackCopy.caught(te);
        } catch (Exception e) {
            logger.error("callback invoke error ,url:" + channel.getUrl(), e);
        }
        // 其他情况处理成RemotingException异常
    } else {
        try {
            RuntimeException re = new RuntimeException(res.getErrorMessage());
            callbackCopy.caught(re);
        } catch (Exception e) {
            logger.error("callback invoke error ,url:" + channel.getUrl(), e);
        }
    }
}
```

该方法是执行回调来处理响应结果。分为了三种情况：

1. 响应成功，那么执行完成后的逻辑。
2. 超时，会按照超时异常来处理
3. 其他，按照RuntimeException异常来处理

具体的处理都在ResponseCallback接口的实现类里执行，后面我会讲到。

#### 7.get

```
@Override
public Object get() throws RemotingException {
    return get(timeout);
}

@Override
public Object get(int timeout) throws RemotingException {
    // 超时时间默认为1s
    if (timeout <= 0) {
        timeout = Constants.DEFAULT_TIMEOUT;
    }
    // 如果请求没有完成，也就是还没有响应返回
    if (!isDone()) {
        long start = System.currentTimeMillis();
        // 获得锁
        lock.lock();
        try {
            // 轮询 等待请求是否完成
            while (!isDone()) {
                // 线程阻塞等待
                done.await(timeout, TimeUnit.MILLISECONDS);
                // 如果请求完成或者超时，则结束
                if (isDone() || System.currentTimeMillis() - start > timeout) {
                    break;
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            // 释放锁
            lock.unlock();
        }
        // 如果没有收到响应，则抛出超时的异常
        if (!isDone()) {
            throw new TimeoutException(sent > 0, channel, getTimeoutMessage(false));
        }
    }
    // 返回响应
    return returnFromResponse();
}
```

该方法是实现了ResponseFuture定义的方法，是获得该future对应的请求对应的响应结果，其实future、请求、响应都是一一对应的。其中如果还没得到响应，则会线程阻塞等待，等到有响应结果或者超时，才返回。返回的逻辑在returnFromResponse中实现。

#### 8.returnFromResponse

```
private Object returnFromResponse() throws RemotingException {
    Response res = response;
    if (res == null) {
        throw new IllegalStateException("response cannot be null");
    }
    // 如果正常返回，则返回响应结果
    if (res.getStatus() == Response.OK) {
        return res.getResult();
    }
    // 如果超时，则抛出超时异常
    if (res.getStatus() == Response.CLIENT_TIMEOUT || res.getStatus() == Response.SERVER_TIMEOUT) {
        throw new TimeoutException(res.getStatus() == Response.SERVER_TIMEOUT, channel, res.getErrorMessage());
    }
    // 其他 抛出RemotingException异常
    throw new RemotingException(channel, res.getErrorMessage());
}
```

这代码跟invokeCallback方法中差不多，都是把响应分了三种情况。

#### 9.cancel

```
public void cancel() {
    // 创建一个取消请求的响应
    Response errorResult = new Response(id);
    errorResult.setErrorMessage("request future has been canceled.");
    response = errorResult;
    // 从集合中删除该请求
    FUTURES.remove(id);
    CHANNELS.remove(id);
}
```

该方法是取消一个请求，可以直接关闭一个请求，也就是值创建一个响应来回应该请求，把response值设置到该请求对于到future中，做到了中断请求的作用。该方法跟closeChannel的区别是closeChannel中对response的状态设置了CHANNEL_INACTIVE，而cancel方法是中途被主动取消的，虽然有response值，但是并没有一个响应状态。

#### 10.RemotingInvocationTimeoutScan

```
private static class RemotingInvocationTimeoutScan implements Runnable {

    @Override
    public void run() {
        while (true) {
            try {
                for (DefaultFuture future : FUTURES.values()) {
                    // 已经完成，跳过扫描
                    if (future == null || future.isDone()) {
                        continue;
                    }
                    // 超时
                    if (System.currentTimeMillis() - future.getStartTimestamp() > future.getTimeout()) {
                        // create exception response.，创建一个超时的响应
                        Response timeoutResponse = new Response(future.getId());
                        // set timeout status.，设置超时状态，是服务端侧超时还是客户端侧超时
                        timeoutResponse.setStatus(future.isSent() ? Response.SERVER_TIMEOUT : Response.CLIENT_TIMEOUT);
                        // 设置错误信息
                        timeoutResponse.setErrorMessage(future.getTimeoutMessage(true));
                        // handle response.，接收创建的超时响应
                        DefaultFuture.received(future.getChannel(), timeoutResponse);
                    }
                }
                // 睡眠
                Thread.sleep(30);
            } catch (Throwable e) {
                logger.error("Exception when scan the timeout invocation of remoting.", e);
            }
        }
    }
}
```

该方法是扫描调用超时任务的线程，每次都会遍历future集合，检测请求是否超时了，如果超时则创建一个超时响应来回应该请求。

```
static {
    // 开启一个后台扫描调用超时任务
    Thread th = new Thread(new RemotingInvocationTimeoutScan(), "DubboResponseTimeoutScanTimer");
    th.setDaemon(true);
    th.start();
}
```

开启一个后台线程进行扫描的逻辑写在了静态代码块里面，只开启一次。

### （八）SimpleFuture

该类实现了ResponseFuture，目前没有用到，很简单的实现，我就不多说了。

### （九）ExchangeHandler

该接口继承了ChannelHandler, TelnetHandler接口，是信息交换处理器接口。

```
public interface ExchangeHandler extends ChannelHandler, TelnetHandler {
    /**
     * reply.
     * 回复请求结果
     * @param channel
     * @param request
     * @return response
     * @throws RemotingException
     */
    Object reply(ExchangeChannel channel, Object request) throws RemotingException;

}
```

该接口只定义了一个回复请求结果的方法，返回的是请求结果。

### （十）ExchangeHandlerDispatcher

该类实现了ExchangeHandler接口， 是信息交换处理器调度器类，也就是对应不同的事件，选择不同的处理器去处理。该类中有三个属性，分别对应了三种事件：

```
/**
 * 回复者调度器
 */
private final ReplierDispatcher replierDispatcher;

/**
 * 通道处理器调度器
 */
private final ChannelHandlerDispatcher handlerDispatcher;

/**
 * Telnet 命令处理器
 */
private final TelnetHandler telnetHandler;
```

如果事件是跟通道处理器有关的，就调用通道处理器来处理，比如：

```
@Override
@SuppressWarnings({"unchecked", "rawtypes"})
public Object reply(ExchangeChannel channel, Object request) throws RemotingException {
    return ((Replier) replierDispatcher).reply(channel, request);
}

@Override
public void connected(Channel channel) {
    handlerDispatcher.connected(channel);
}
@Override
public String telnet(Channel channel, String message) throws RemotingException {
    return telnetHandler.telnet(channel, message);
}
```

可以看到以上三种事件，回复请求结果需要回复者调度器来处理，连接需要通道处理器调度器来处理，telnet消息需要Telnet命令处理器来处理。

### （十一）ExchangeHandlerAdapter

该类继承了TelnetHandlerAdapter，实现了ExchangeHandler，是信息交换处理器的适配器类。

```
public abstract class ExchangeHandlerAdapter extends TelnetHandlerAdapter implements ExchangeHandler {

    @Override
    public Object reply(ExchangeChannel channel, Object msg) throws RemotingException {
        // 直接返回null
        return null;
    }

}
```

该类直接让ExchangeHandler定义的方法reply返回null，交由它的子类选择性的去实现具体的回复请求结果。

### （十二）ExchangeServer

该接口继承了Server接口，定义了两个方法：

```
public interface ExchangeServer extends Server {

    /**
     * get channels.
     * 获得通道集合
     * @return channels
     */
    Collection<ExchangeChannel> getExchangeChannels();

    /**
     * get channel.
     * 根据远程地址获得对应的信息通道
     * @param remoteAddress
     * @return channel
     */
    ExchangeChannel getExchangeChannel(InetSocketAddress remoteAddress);

}
```

该接口比较好理解，并且在Server接口基础上新定义了两个方法。直接来看看它的实现类吧。

### （十三）HeaderExchangeServer

该类实现了ExchangeServer接口，是基于协议头的信息交换服务器实现类，HeaderExchangeServer是Server的装饰器，每个实现方法都会调用server的方法。

#### 1.属性

```
protected final Logger logger = LoggerFactory.getLogger(getClass());

/**
 * 线程池
 */
private final ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(1,
        new NamedThreadFactory(
                "dubbo-remoting-server-heartbeat",
                true));
/**
 * 服务器
 */
private final Server server;
// heartbeat timer
/**
 * 心跳定时器
 */
private ScheduledFuture<?> heartbeatTimer;
// heartbeat timeout (ms), default value is 0 , won't execute a heartbeat.
/**
 * 心跳周期
 */
private int heartbeat;
/**
 * 心跳超时时间
 */
private int heartbeatTimeout;
/**
 * 信息交换服务器是否关闭
 */
private AtomicBoolean closed = new AtomicBoolean(false);
```

该类里面的很多实现跟HeaderExchangeClient差不多，包括心跳检测等逻辑。看得懂上述我讲的HeaderExchangeClient的属性，想必这里的属性应该也很简单了。

#### 2.构造函数

```
public HeaderExchangeServer(Server server) {
    if (server == null) {
        throw new IllegalArgumentException("server == null");
    }
    this.server = server;
    //获得心跳周期配置，如果没有配置，默认设置为0
    this.heartbeat = server.getUrl().getParameter(Constants.HEARTBEAT_KEY, 0);
    // 获得心跳超时配置，默认是心跳周期的三倍
    this.heartbeatTimeout = server.getUrl().getParameter(Constants.HEARTBEAT_TIMEOUT_KEY, heartbeat * 3);
    // 如果心跳超时时间小于心跳周期的两倍，则抛出异常
    if (heartbeatTimeout < heartbeat * 2) {
        throw new IllegalStateException("heartbeatTimeout < heartbeatInterval * 2");
    }
    // 开始心跳
    startHeartbeatTimer();
}

public Server getServer() {
    return server;
}
```

构造函数就是对属性的设置，心跳的机制以及默认值都跟HeaderExchangeClient中的一模一样。

#### 3.isRunning

```
private boolean isRunning() {
    Collection<Channel> channels = getChannels();
    // 遍历所有连接该服务器的通道
    for (Channel channel : channels) {

        /**
         *  If there are any client connections,
         *  our server should be running.
         */

        // 只要有任何一个客户端连接，则服务器还运行着
        if (channel.isConnected()) {
            return true;
        }
    }
    return false;
}
```

该方法是检测服务器是否还运行，只要有一个客户端连接着，就算服务器运行着。

#### 4.close

```
@Override
public void close() {
    // 关闭线程池和心跳检测
    doClose();
    // 关闭服务器
    server.close();
}

@Override
public void close(final int timeout) {
    // 开始关闭
    startClose();
    if (timeout > 0) {
        final long max = (long) timeout;
        final long start = System.currentTimeMillis();
        if (getUrl().getParameter(Constants.CHANNEL_SEND_READONLYEVENT_KEY, true)) {
            // 发送 READONLY_EVENT事件给所有连接该服务器的客户端，表示 Server 不可读了。
            sendChannelReadOnlyEvent();
        }
        // 当服务器还在运行，并且没有超时，睡眠，也就是等待timeout左右时间在进行关闭
        while (HeaderExchangeServer.this.isRunning()
                && System.currentTimeMillis() - start < max) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                logger.warn(e.getMessage(), e);
            }
        }
    }
    // 关闭线程池和心跳检测
    doClose();
    // 延迟关闭
    server.close(timeout);
}
```

两个close方法，第二个close方法是优雅的关闭，有一定的延时来让一些响应或者操作做完。关闭分两个步骤，第一个就是关闭信息交换服务器中的线程池和心跳检测，然后才是关闭服务器。

#### 5.sendChannelReadOnlyEvent

```
private void sendChannelReadOnlyEvent() {
    // 创建一个READONLY_EVENT事件的请求
    Request request = new Request();
    request.setEvent(Request.READONLY_EVENT);
    // 不需要响应
    request.setTwoWay(false);
    // 设置版本
    request.setVersion(Version.getProtocolVersion());

    Collection<Channel> channels = getChannels();
    // 遍历连接的通道，进行通知
    for (Channel channel : channels) {
        try {
            // 通过通道还连接着，则发送通知
            if (channel.isConnected())
                channel.send(request, getUrl().getParameter(Constants.CHANNEL_READONLYEVENT_SENT_KEY, true));
        } catch (RemotingException e) {
            logger.warn("send cannot write message error.", e);
        }
    }
}
```

在关闭服务器中有一个操作就是发送事件READONLY_EVENT，告诉客户端该服务器不可读了，就是该方法实现的，逐个通知连接的客户端该事件。

#### 6.doClose

```
private void doClose() {
    if (!closed.compareAndSet(false, true)) {
        return;
    }
    // 停止心跳检测
    stopHeartbeatTimer();
    try {
        // 关闭线程池
        scheduled.shutdown();
    } catch (Throwable t) {
        logger.warn(t.getMessage(), t);
    }
}
```

该方法就是close方法调用到的停止心跳检测和关闭线程池。

#### 7.getExchangeChannels

```
@Override
public Collection<ExchangeChannel> getExchangeChannels() {
    Collection<ExchangeChannel> exchangeChannels = new ArrayList<ExchangeChannel>();
    // 获得连接该服务器通道集合
    Collection<Channel> channels = server.getChannels();
    if (channels != null && !channels.isEmpty()) {
        // 遍历通道集合，为每个通道都创建信息交换通道，并且加入信息交换通道集合
        for (Channel channel : channels) {
            exchangeChannels.add(HeaderExchangeChannel.getOrAddChannel(channel));
        }
    }
    return exchangeChannels;
}
```

该方法是返回连接该服务器信息交换通道集合。逻辑就是先获得通道集合，在根据通道来创建信息交换通道，然后返回信息通道集合。

#### 8.reset

```
@Override
public void reset(URL url) {
    // 重置属性
    server.reset(url);
    try {
        // 重置的逻辑跟构造函数一样设置
        if (url.hasParameter(Constants.HEARTBEAT_KEY)
                || url.hasParameter(Constants.HEARTBEAT_TIMEOUT_KEY)) {
            int h = url.getParameter(Constants.HEARTBEAT_KEY, heartbeat);
            int t = url.getParameter(Constants.HEARTBEAT_TIMEOUT_KEY, h * 3);
            if (t < h * 2) {
                throw new IllegalStateException("heartbeatTimeout < heartbeatInterval * 2");
            }
            if (h != heartbeat || t != heartbeatTimeout) {
                heartbeat = h;
                heartbeatTimeout = t;
                // 重新开始心跳
                startHeartbeatTimer();
            }
        }
    } catch (Throwable t) {
        logger.error(t.getMessage(), t);
    }
}
```

该方法就是重置属性，重置后，重新开始心跳，设置心跳属性的机制跟构造函数一样。

#### 9.startHeartbeatTimer

```
private void startHeartbeatTimer() {
    // 先停止现有的心跳检测
    stopHeartbeatTimer();
    if (heartbeat > 0) {
        // 创建心跳定时器
        heartbeatTimer = scheduled.scheduleWithFixedDelay(
                new HeartBeatTask(new HeartBeatTask.ChannelProvider() {
                    @Override
                    public Collection<Channel> getChannels() {
                        // 返回一个不可修改的连接该服务器的信息交换通道集合
                        return Collections.unmodifiableCollection(
                                HeaderExchangeServer.this.getChannels());
                    }
                }, heartbeat, heartbeatTimeout),
                heartbeat, heartbeat, TimeUnit.MILLISECONDS);
    }
}
```

该方法是开始心跳，跟HeaderExchangeClient类中的开始心跳方法唯一区别是获得的通道不一样，客户端跟通道是一一对应的，所有只要对一个通道进行心跳检测，而服务端跟通道是一对多的关系，所有需要对该服务器连接的所有通道进行心跳检测。

#### 10.stopHeartbeatTimer

```
private void stopHeartbeatTimer() {
    if (heartbeatTimer != null && !heartbeatTimer.isCancelled()) {
        try {
            // 取消定时器
            heartbeatTimer.cancel(true);
            // 取消大量已排队任务，用于回收空间
            scheduled.purge();
        } catch (Throwable e) {
            if (logger.isWarnEnabled()) {
                logger.warn(e.getMessage(), e);
            }
        }
    }
    heartbeatTimer = null;
}
```

该方法是停止当前的心跳检测。

### （十四）ExchangeServerDelegate

该类实现了ExchangeServer接口，是信息交换服务器装饰者，是ExchangeServer的装饰器。该类就一个属性ExchangeServer server，所有实现方法都调用了server属性的方法。目前只有在p2p中被用到，代码为就不贴了，很简单。

### （十五）Exchanger

```
@SPI(HeaderExchanger.NAME)
public interface Exchanger {

    /**
     * bind.
     * 绑定一个服务器
     * @param url 服务器url
     * @param handler 数据交换处理器
     * @return message server 数据交换服务器
     */
    @Adaptive({Constants.EXCHANGER_KEY})
    ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException;

    /**
     * connect.
     * 连接一个服务器，也就是创建一个客户端
     * @param url 服务器url
     * @param handler 数据交换处理器
     * @return message channel 返回数据交换客户端
     */
    @Adaptive({Constants.EXCHANGER_KEY})
    ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException;

}
```

该接口是数据交换者接口，该接口是一个可扩展接口默认实现的是HeaderExchanger类，并且用到了dubbo SPI的Adaptive机制，优先实现url携带的配置。如果不了解dubbo SPI机制的可以看[《dubbo源码解析（二）Dubbo扩展机制SPI》](https://segmentfault.com/a/1190000016842868)。那么回到该接口定义的方法，定义了绑定和连接两个方法，分别返回信息交互服务器和客户端实例。

### （十六）HeaderExchanger

```
public class HeaderExchanger implements Exchanger {

    public static final String NAME = "header";

    @Override
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        // 用传输层连接返回的client 创建对应的信息交换客户端，默认开启心跳检测
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))), true);
    }

    @Override
    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        // 用传输层绑定返回的server 创建对应的信息交换服务端
        return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

}
```

该类继承了Exchanger接口，是Exchanger接口的默认实现，实现了Exchanger接口定义的两个方法，分别调用的是Transporters的连接和绑定方法，再利用这这两个方法返回的客户端和服务端实例来创建信息交换的客户端和服务端。

### （十七）Replier

我们知道Request对应的是ExchangeHandler接口实现对象来处理，但有些时候我们需要不同数据类型对应不同的处理器，该类就是为了支持这一需求所设计的。

```
public interface Replier<T> {

    /**
     * reply.
     * 回复请求结果
     * @param channel
     * @param request
     * @return response
     * @throws RemotingException
     */
    Object reply(ExchangeChannel channel, T request) throws RemotingException;

}
```

可以看到该接口跟ExchangeHandler定义的方法也一一，只有请求的类型改为了范型。

### （十八）ReplierDispatcher

该类实现了Replier接口，是回复者调度器实现类。

```
/**
 * 默认回复者
 */
private final Replier<?> defaultReplier;

/**
 * 回复者集合
 */
private final Map<Class<?>, Replier<?>> repliers = new ConcurrentHashMap<Class<?>, Replier<?>>();
```

这是该类的两个属性，缓存了回复者集合和默认的回复者。

```
/**
 * 从回复者集合中找到该类型的回复者，并且返回
 * @param type
 * @return
 */
private Replier<?> getReplier(Class<?> type) {
    for (Map.Entry<Class<?>, Replier<?>> entry : repliers.entrySet()) {
        if (entry.getKey().isAssignableFrom(type)) {
            return entry.getValue();
        }
    }
    if (defaultReplier != null) {
        return defaultReplier;
    }
    throw new IllegalStateException("Replier not found, Unsupported message object: " + type);
}

/**
 * 回复请求
 * @param channel
 * @param request
 * @return
 * @throws RemotingException
 */
@Override
@SuppressWarnings({"unchecked", "rawtypes"})
public Object reply(ExchangeChannel channel, Object request) throws RemotingException {
    return ((Replier) getReplier(request.getClass())).reply(channel, request);
}
```

上述是该类中关键的两个方法，reply还是调用实现类的reply。根据请求的数据类型来使用指定的回复者进行回复。

### （十九）MultiMessage

该类实现了实现 Iterable 接口，是多消息的封装，我们直接看它的属性：

```
/**
 * 消息集合
 */
private final List messages = new ArrayList();
```

该类要和[《dubbo源码解析（九）远程通信——Transport层》](https://segmentfault.com/a/1190000017390253)的（八）MultiMessageHandler联合着看。

### （二十）HeartbeatHandler

该类继承了AbstractChannelHandlerDelegate类，是心跳处理器。是用来处理心跳事件的，也接收消息上增加了对心跳消息的处理。该类是

```
@Override
public void received(Channel channel, Object message) throws RemotingException {
    // 设置接收时间的时间戳属性值
    setReadTimestamp(channel);
    // 如果是心跳请求
    if (isHeartbeatRequest(message)) {
        Request req = (Request) message;
        // 如果需要响应
        if (req.isTwoWay()) {
            // 创建一个响应
            Response res = new Response(req.getId(), req.getVersion());
            // 设置为心跳事件的响应
            res.setEvent(Response.HEARTBEAT_EVENT);
            // 发送消息，也就是返回响应
            channel.send(res);
            if (logger.isInfoEnabled()) {
                int heartbeat = channel.getUrl().getParameter(Constants.HEARTBEAT_KEY, 0);
                if (logger.isDebugEnabled()) {
                    logger.debug("Received heartbeat from remote channel " + channel.getRemoteAddress()
                            + ", cause: The channel has no data-transmission exceeds a heartbeat period"
                            + (heartbeat > 0 ? ": " + heartbeat + "ms" : ""));
                }
            }
        }
        return;
    }
    // 如果是心跳响应，则直接return
    if (isHeartbeatResponse(message)) {
        if (logger.isDebugEnabled()) {
            logger.debug("Receive heartbeat response in thread " + Thread.currentThread().getName());
        }
        return;
    }
    handler.received(channel, message);
}
```

该方法是就是在handler处理消息上增加了处理心跳消息的功能，做到了功能增强。

### （二十一）Exchangers

该类跟Transporters的设计意图是一样的，Transporters我在[《dubbo源码解析（八）远程通信——开篇》](https://segmentfault.com/a/1190000017274525)的（十）Transporters已经讲到了。Exchangers也用到了外观模式。代码为就不贴了，可以对照着Transporters来看，很简单。

### （二十二）Request

请求模型类，最重要的肯定是模型的属性，我们来看看属性：

```
/**
 * 心跳事件
 */
public static final String HEARTBEAT_EVENT = null;

/**
 * 只读事件
 */
public static final String READONLY_EVENT = "R";

/**
 * 请求编号自增序列
 */
private static final AtomicLong INVOKE_ID = new AtomicLong(0);

/**
 * 请求编号
 */
private final long mId;

/**
 * dubbo版本
 */
private String mVersion;

/**
 * 是否需要响应
 */
private boolean mTwoWay = true;

/**
 * 是否是事件
 */
private boolean mEvent = false;

/**
 * 是否是异常的请求
 */
private boolean mBroken = false;

/**
 * 请求数据
 */
private Object mData;
```

1. 由于心跳事件比较常用，所有设置为null。
2. 请求编号使用INVOKE_ID生成，是JVM 进程内唯一的。
3. 其他属性比较简单

### （二十三）Response

响应模型，来看看它的属性：

```
/**
 * 心跳事件
 */
public static final String HEARTBEAT_EVENT = null;

/**
 * 只读事件
 */
public static final String READONLY_EVENT = "R";

/**
 * ok.
 * 成功状态码
 */
public static final byte OK = 20;

/**
 * clien side timeout.
 * 客户端侧的超时状态码
 */
public static final byte CLIENT_TIMEOUT = 30;

/**
 * server side timeout.
 * 服务端侧超时的状态码
 */
public static final byte SERVER_TIMEOUT = 31;

/**
 * channel inactive, directly return the unfinished requests.
 * 通道不活跃，返回未完成请求的状态码
 */
public static final byte CHANNEL_INACTIVE = 35;

/**
 * request format error.
 * 请求格式错误状态码
 */
public static final byte BAD_REQUEST = 40;

/**
 * response format error.
 * 响应格式错误状态码
 */
public static final byte BAD_RESPONSE = 50;

/**
 * service not found.
 * 服务找不到状态码
 */
public static final byte SERVICE_NOT_FOUND = 60;

/**
 * service error.
 * 服务错误状态码
 */
public static final byte SERVICE_ERROR = 70;

/**
 * internal server error.
 * 内部服务器错误状态码
 */
public static final byte SERVER_ERROR = 80;

/**
 * internal server error.
 * 客户端错误状态码
 */
public static final byte CLIENT_ERROR = 90;

/**
 * server side threadpool exhausted and quick return.
 * 服务器端线程池耗尽并快速返回状态码
 */
public static final byte SERVER_THREADPOOL_EXHAUSTED_ERROR = 100;

/**
 * 响应编号
 */
private long mId = 0;

/**
 * dubbo 版本
 */
private String mVersion;

/**
 * 状态
 */
private byte mStatus = OK;

/**
 * 是否是事件
 */
private boolean mEvent = false;

/**
 * 错误信息
 */
private String mErrorMsg;

/**
 * 返回结果
 */
private Object mResult;
```

很多属性跟Request模型的属性一样，并且含义也一样，不过该模型多了很多的状态码。关键的是id跟请求一一对应。

### （二十四）ResponseCallback

```
public interface ResponseCallback {

    /**
     * done.
     * 处理请求
     * @param response
     */
    void done(Object response);

    /**
     * caught exception.
     * 处理异常
     * @param exception
     */
    void caught(Throwable exception);

}
```

该接口是回调的接口，定义了两个方法，分别是处理正常的响应结果和处理异常。

### （二十五）ExchangeCodec

该类继承了TelnetCodec，是信息交换编解码器。在本文的开头，我就写到，dubbo将一条消息分成了协议头和协议体，用来解决粘包拆包问题，但是头跟体在编解码上有区别，我们先来看看dubbo 的协议头的配置：

![dubbo_protocol_header](https://segmentfault.com/img/remote/1460000017467347)

上图是官方文档的图片，能够清晰的看出协议中各个数据所占的位数：

1. 0-7位和8-15位：Magic High和Magic Low，类似java字节码文件里的魔数，用来判断是不是dubbo协议的数据包，就是一个固定的数字
2. 16位：Req/Res：请求还是响应标识。
3. 17位：2way：单向还是双向
4. 18位：Event：是否是事件
5. 19-23位：Serialization 编号
6. 24-31位：status状态
7. 32-95位：id编号
8. 96-127位：body数据
9. 128-…位：上图表格内的数据

可以看到一个该协议中前65位是协议头，后面的都是协议体数据。那么在编解码中，协议头是通过 Codec 编解码，而body部分是用Serialization序列化和反序列化的。下面我们就来看看该类对协议头的编解码。

#### 1.属性

```
// header length.
/**
 * 协议头长度：16字节 = 128Bits
 */
protected static final int HEADER_LENGTH = 16;
// magic header.
/**
 * MAGIC二进制：1101101010111011，十进制：55995
 */
protected static final short MAGIC = (short) 0xdabb;
/**
 * Magic High，也就是0-7位：11011010
 */
protected static final byte MAGIC_HIGH = Bytes.short2bytes(MAGIC)[0];
/**
 * Magic Low  8-15位 ：10111011
 */
protected static final byte MAGIC_LOW = Bytes.short2bytes(MAGIC)[1];
// message flag.
/**
 * 128 二进制：10000000
 */
protected static final byte FLAG_REQUEST = (byte) 0x80;
/**
 * 64 二进制：1000000
 */
protected static final byte FLAG_TWOWAY = (byte) 0x40;
/**
 * 32 二进制：100000
 */
protected static final byte FLAG_EVENT = (byte) 0x20;
/**
 * 31 二进制：11111
 */
protected static final int SERIALIZATION_MASK = 0x1f;
```

可以看到 MAGIC是个固定的值，用来判断是不是dubbo协议的数据包，并且MAGIC_LOW和MAGIC_HIGH分别是MAGIC的低位和高位。其他的属性用来干嘛后面会讲到。

#### 2.encode

```
@Override
public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
    if (msg instanceof Request) {
        // 如果消息是Request类型，对请求消息编码
        encodeRequest(channel, buffer, (Request) msg);
    } else if (msg instanceof Response) {
        // 如果消息是Response类型，对响应消息编码
        encodeResponse(channel, buffer, (Response) msg);
    } else {
        // 直接让父类( Telnet ) 处理，目前是 Telnet 命令的结果。
        super.encode(channel, buffer, msg);
    }
}
```

该方法是根据消息的类型来分别进行编码，分为三种情况：Request类型、Response类型以及其他

#### 3.encodeRequest

```
protected void encodeRequest(Channel channel, ChannelBuffer buffer, Request req) throws IOException {
    Serialization serialization = getSerialization(channel);
    // header.
    // 创建16字节的字节数组
    byte[] header = new byte[HEADER_LENGTH];
    // set magic number.
    // 设置前16位数据，也就是设置header[0]和header[1]的数据为Magic High和Magic Low
    Bytes.short2bytes(MAGIC, header);

    // set request and serialization flag.
    // 16-23位为serialization编号，用到或运算10000000|serialization编号，例如serialization编号为11111，则为00011111
    header[2] = (byte) (FLAG_REQUEST | serialization.getContentTypeId());

    // 继续上面的例子，00011111|1000000 = 01011111
    if (req.isTwoWay()) header[2] |= FLAG_TWOWAY;
    // 继续上面的例子，01011111|100000 = 011 11111 可以看到011代表请求标记、双向、是事件，这样就设置了16、17、18位，后面19-23位是Serialization 编号
    if (req.isEvent()) header[2] |= FLAG_EVENT;

    // set request id.
    // 设置32-95位请求id
    Bytes.long2bytes(req.getId(), header, 4);

    // encode request data.
    // // 编码 `Request.data` 到 Body ，并写入到 Buffer
    int savedWriteIndex = buffer.writerIndex();
    buffer.writerIndex(savedWriteIndex + HEADER_LENGTH);
    ChannelBufferOutputStream bos = new ChannelBufferOutputStream(buffer);
    // 对body数据序列化
    ObjectOutput out = serialization.serialize(channel.getUrl(), bos);
    // 如果该请求是事件
    if (req.isEvent()) {
        // 特殊事件编码
        encodeEventData(channel, out, req.getData());
    } else {
        // 正常请求编码
        encodeRequestData(channel, out, req.getData(), req.getVersion());
    }
    // 释放资源
    out.flushBuffer();
    if (out instanceof Cleanable) {
        ((Cleanable) out).cleanup();
    }
    bos.flush();
    bos.close();
    int len = bos.writtenBytes();
    //检验消息长度
    checkPayload(channel, len);
    // 设置96-127位：Body值
    Bytes.int2bytes(len, header, 12);

    // write
    // 把header写入到buffer
    buffer.writerIndex(savedWriteIndex);
    buffer.writeBytes(header); // write header.
    buffer.writerIndex(savedWriteIndex + HEADER_LENGTH + len);
}
```

该方法是对Request类型的消息进行编码，仔细阅读上述我写的注解，结合协议头各个位数的含义，好好品味我举的例子。享受二进制位运算带来的快乐，也可以看到前半部分逻辑是对协议头的编码，后面还有对body值的序列化。

#### 4.encodeResponse

```
protected void encodeRequest(Channel channel, ChannelBuffer buffer, Request req) throws IOException {
    Serialization serialization = getSerialization(channel);
    // header.
    // 创建16字节的字节数组
    byte[] header = new byte[HEADER_LENGTH];
    // set magic number.
    // 设置前16位数据，也就是设置header[0]和header[1]的数据为Magic High和Magic Low
    Bytes.short2bytes(MAGIC, header);

    // set request and serialization flag.
    // 16-23位为serialization编号，用到或运算10000000|serialization编号，例如serialization编号为11111，则为00011111
    header[2] = (byte) (FLAG_REQUEST | serialization.getContentTypeId());

    // 继续上面的例子，00011111|1000000 = 01011111
    if (req.isTwoWay()) header[2] |= FLAG_TWOWAY;
    // 继续上面的例子，01011111|100000 = 011 11111 可以看到011代表请求标记、双向、是事件，这样就设置了16、17、18位，后面19-23位是Serialization 编号
    if (req.isEvent()) header[2] |= FLAG_EVENT;

    // set request id.
    // 设置32-95位请求id
    Bytes.long2bytes(req.getId(), header, 4);

    // encode request data.
    // // 编码 `Request.data` 到 Body ，并写入到 Buffer
    int savedWriteIndex = buffer.writerIndex();
    buffer.writerIndex(savedWriteIndex + HEADER_LENGTH);
    ChannelBufferOutputStream bos = new ChannelBufferOutputStream(buffer);
    // 对body数据序列化
    ObjectOutput out = serialization.serialize(channel.getUrl(), bos);
    // 如果该请求是事件
    if (req.isEvent()) {
        // 特殊事件编码
        encodeEventData(channel, out, req.getData());
    } else {
        // 正常请求编码
        encodeRequestData(channel, out, req.getData(), req.getVersion());
    }
    // 释放资源
    out.flushBuffer();
    if (out instanceof Cleanable) {
        ((Cleanable) out).cleanup();
    }
    bos.flush();
    bos.close();
    int len = bos.writtenBytes();
    //检验消息长度
    checkPayload(channel, len);
    // 设置96-127位：Body值
    Bytes.int2bytes(len, header, 12);

    // write
    // 把header写入到buffer
    buffer.writerIndex(savedWriteIndex);
    buffer.writeBytes(header); // write header.
    buffer.writerIndex(savedWriteIndex + HEADER_LENGTH + len);
}

protected void encodeResponse(Channel channel, ChannelBuffer buffer, Response res) throws IOException {
    int savedWriteIndex = buffer.writerIndex();
    try {
        Serialization serialization = getSerialization(channel);
        // header.
        // 创建16字节大小的字节数组
        byte[] header = new byte[HEADER_LENGTH];
        // set magic number.
        // 设置前0-15位为魔数
        Bytes.short2bytes(MAGIC, header);
        // set request and serialization flag.
        // 设置响应标志和序列化id
        header[2] = serialization.getContentTypeId();
        // 如果是心跳事件，则设置第18位为事件
        if (res.isHeartbeat()) header[2] |= FLAG_EVENT;
        // set response status.
        // 设置24-31位为状态码
        byte status = res.getStatus();
        header[3] = status;
        // set request id.
        // 设置32-95位为请求id
        Bytes.long2bytes(res.getId(), header, 4);

        // 写入数据
        buffer.writerIndex(savedWriteIndex + HEADER_LENGTH);
        ChannelBufferOutputStream bos = new ChannelBufferOutputStream(buffer);
        // 对body进行序列化
        ObjectOutput out = serialization.serialize(channel.getUrl(), bos);
        // encode response data or error message.
        if (status == Response.OK) {
            if (res.isHeartbeat()) {
                // 对心跳事件编码
                encodeHeartbeatData(channel, out, res.getResult());
            } else {
                // 对普通响应编码
                encodeResponseData(channel, out, res.getResult(), res.getVersion());
            }
        } else out.writeUTF(res.getErrorMessage());
        // 释放
        out.flushBuffer();
        if (out instanceof Cleanable) {
            ((Cleanable) out).cleanup();
        }
        bos.flush();
        bos.close();

        int len = bos.writtenBytes();
        checkPayload(channel, len);
        Bytes.int2bytes(len, header, 12);
        // write
        buffer.writerIndex(savedWriteIndex);
        buffer.writeBytes(header); // write header.
        buffer.writerIndex(savedWriteIndex + HEADER_LENGTH + len);
    } catch (Throwable t) {
        // clear buffer
        buffer.writerIndex(savedWriteIndex);
        // send error message to Consumer, otherwise, Consumer will wait till timeout.
        //如果在写入数据失败，则返回响应格式错误的返回码
        if (!res.isEvent() && res.getStatus() != Response.BAD_RESPONSE) {
            Response r = new Response(res.getId(), res.getVersion());
            r.setStatus(Response.BAD_RESPONSE);

            if (t instanceof ExceedPayloadLimitException) {
                logger.warn(t.getMessage(), t);
                try {
                    r.setErrorMessage(t.getMessage());
                    // 发送响应
                    channel.send(r);
                    return;
                } catch (RemotingException e) {
                    logger.warn("Failed to send bad_response info back: " + t.getMessage() + ", cause: " + e.getMessage(), e);
                }
            } else {
                // FIXME log error message in Codec and handle in caught() of IoHanndler?
                logger.warn("Fail to encode response: " + res + ", send bad_response info instead, cause: " + t.getMessage(), t);
                try {
                    r.setErrorMessage("Failed to send response: " + res + ", cause: " + StringUtils.toString(t));
                    channel.send(r);
                    return;
                } catch (RemotingException e) {
                    logger.warn("Failed to send bad_response info back: " + res + ", cause: " + e.getMessage(), e);
                }
            }
        }

        // Rethrow exception
        if (t instanceof IOException) {
            throw (IOException) t;
        } else if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new RuntimeException(t.getMessage(), t);
        }
    }
}
```

该方法是对Response类型的消息进行编码，该方法里面我没有举例子演示如何进行编码，不过过程跟encodeRequest类似。

#### 5.decode

```
@Override
public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
    int readable = buffer.readableBytes();
    // 读取前16字节的协议头数据，如果数据不满16字节，则读取全部
    byte[] header = new byte[Math.min(readable, HEADER_LENGTH)];
    buffer.readBytes(header);
    // 解码
    return decode(channel, buffer, readable, header);
}

@Override
protected Object decode(Channel channel, ChannelBuffer buffer, int readable, byte[] header) throws IOException {
    // check magic number.
    // 核对魔数（该数字固定）
    if (readable > 0 && header[0] != MAGIC_HIGH
            || readable > 1 && header[1] != MAGIC_LOW) {
        int length = header.length;
        // 将 buffer 完全复制到 `header` 数组中
        if (header.length < readable) {
            header = Bytes.copyOf(header, readable);
            buffer.readBytes(header, length, readable - length);
        }
        for (int i = 1; i < header.length - 1; i++) {
            if (header[i] == MAGIC_HIGH && header[i + 1] == MAGIC_LOW) {
                buffer.readerIndex(buffer.readerIndex() - header.length + i);
                header = Bytes.copyOf(header, i);
                break;
            }
        }
        return super.decode(channel, buffer, readable, header);
    }
    // check length.
    // Header 长度不够，返回需要更多的输入，解决拆包现象
    if (readable < HEADER_LENGTH) {
        return DecodeResult.NEED_MORE_INPUT;
    }

    // get data length.
    int len = Bytes.bytes2int(header, 12);
    // 检查信息头长度
    checkPayload(channel, len);

    int tt = len + HEADER_LENGTH;
    // 总长度不够，返回需要更多的输入，解决拆包现象
    if (readable < tt) {
        return DecodeResult.NEED_MORE_INPUT;
    }

    // limit input stream.
    ChannelBufferInputStream is = new ChannelBufferInputStream(buffer, len);

    try {
        // 对body反序列化
        return decodeBody(channel, is, header);
    } finally {
        // 如果不可用
        if (is.available() > 0) {
            try {
                // 打印错误日志
                if (logger.isWarnEnabled()) {
                    logger.warn("Skip input stream " + is.available());
                }
                // 跳过未读完的流
                StreamUtils.skipUnusedStream(is);
            } catch (IOException e) {
                logger.warn(e.getMessage(), e);
            }
        }
    }
}
```

该方法就是解码前的一些核对过程，包括检测是否为dubbo协议，是否有拆包现象等，具体的解码在decodeBody方法。

#### 6.decodeBody

```
protected Object decodeBody(Channel channel, InputStream is, byte[] header) throws IOException {
    // 用并运算符
    byte flag = header[2], proto = (byte) (flag & SERIALIZATION_MASK);
    // get request id.
    // 获得请求id
    long id = Bytes.bytes2long(header, 4);
    // 如果第16位为0，则说明是响应
    if ((flag & FLAG_REQUEST) == 0) {
        // decode response.
        Response res = new Response(id);
        // 如果第18位不是0，则说明是心跳事件
        if ((flag & FLAG_EVENT) != 0) {
            res.setEvent(Response.HEARTBEAT_EVENT);
        }
        // get status.
        byte status = header[3];
        res.setStatus(status);
        try {
            ObjectInput in = CodecSupport.deserialize(channel.getUrl(), is, proto);
            // 如果响应是成功的
            if (status == Response.OK) {
                Object data;
                if (res.isHeartbeat()) {
                    // 如果是心跳事件，则心跳事件的解码
                    data = decodeHeartbeatData(channel, in);
                } else if (res.isEvent()) {
                    // 如果是事件，则事件的解码
                    data = decodeEventData(channel, in);
                } else {
                    // 否则执行普通解码
                    data = decodeResponseData(channel, in, getRequestData(id));
                }
                // 重新设置响应结果
                res.setResult(data);
            } else {
                res.setErrorMessage(in.readUTF());
            }
        } catch (Throwable t) {
            res.setStatus(Response.CLIENT_ERROR);
            res.setErrorMessage(StringUtils.toString(t));
        }
        return res;
    } else {
        // decode request.
        // 对请求类型解码
        Request req = new Request(id);
        // 设置版本号
        req.setVersion(Version.getProtocolVersion());
        // 如果第17位不为0，则是双向
        req.setTwoWay((flag & FLAG_TWOWAY) != 0);
        // 如果18位不为0，则是心跳事件
        if ((flag & FLAG_EVENT) != 0) {
            req.setEvent(Request.HEARTBEAT_EVENT);
        }
        try {
            // 反序列化
            ObjectInput in = CodecSupport.deserialize(channel.getUrl(), is, proto);
            Object data;
            if (req.isHeartbeat()) {
                // 如果请求是心跳事件，则心跳事件解码
                data = decodeHeartbeatData(channel, in);
            } else if (req.isEvent()) {
                // 如果是事件，则事件解码
                data = decodeEventData(channel, in);
            } else {
                // 否则，用普通解码
                data = decodeRequestData(channel, in);
            }
            // 把重新设置请求数据
            req.setData(data);
        } catch (Throwable t) {
            // bad request
            // 设置是异常请求
            req.setBroken(true);
            req.setData(t);
        }
        return req;
    }
}
```

该方法就是解码的过程，并且对协议头和协议体分开解码，协议头编码是做或运算，而解码则是做并运算，协议体用反序列化的方式解码，同样也是分为了Request类型、Response类型进行解码。

# 远程通讯——Buffer

> 目标：介绍Buffer的相关实现逻辑、介绍dubbo-remoting-api中的buffer包内的源码解析。

## 前言

缓存区在NIO框架中非常重要，它作为字节容器，每个NIO框架都有自己的相应的设计实现。比如Java NIO有ByteBuffer的设计，Mina有IoBuffer的设计，Netty4有ByteBuf的设计。 那么在本文讲到的内容是dubbo对于缓冲区做的一些接口定义，并且做了不同的框架实现缓冲区公共的逻辑。下面是本文要讲到的类图：

![buffer类图](https://segmentfault.com/img/remote/1460000017483892)

接下来我就按照类图上的一个一个分析，忽略test类。

## 源码分析

### （一）ChannelBuffer

该接口继承了Comparable接口，该接口是通道缓存接口，是字节容器，在netty中也有通道缓存的设计，也就是io.netty.buffer.ByteBuf，该接口的方法定义和设计跟ByteBuf几乎一样，连注释都一样，所以我就不再细说了。

### （二）AbstractChannelBuffer

该类实现了ChannelBuffer接口，是通道缓存的抽象类，它实现了ChannelBuffer所有方法，但是它实现的方法都是需要被重写的方法，具体的实现都是需要子类来实现。现在我们来恶补一下这个通道缓存的原理，当然这个原理跟netty的ByteBuf原理是分不开的。AbstractChannelBuffer维护了两个索引，一个用于读取，另一个用于写入当你从通道缓存中读取时，readerIndex将会被递增已经被读取的字节数，同样的当你写入的时候writerIndex也会被递增。

```
/**
 * 读索引
 */
private int readerIndex;

/**
 * 写索引
 */
private int writerIndex;

/**
 * 标记读索引
 */
private int markedReaderIndex;

/**
 * 标记写索引
 */
private int markedWriterIndex;
```

可以看到该类有四个属性，读索引和写索引的作用就是我上述介绍的，读索引和写索引的起始位置都为索引位置0。而标记读索引和标记写索引是为了做备份回滚，当对缓冲区进行读写操作时，可能需要对之前的操作进行回滚，我们就需要将当前的读写索引备份到相应的标记索引中。

该类的其他方法都是利用四个属性来操作，无非就是检测是否有数据可读或者还是否有空间可写等方法，做一些前置条件的校验以及索引的设置，具体的实现都是需要子类来实现，所以我就不贴代码，因为逻辑比较简单。

### （三）DynamicChannelBuffer

该类继承了AbstractChannelBuffer类，该类是动态的通道缓存区类，也就是该类是从ChannelBufferFactory工厂中动态的生成缓冲区，默认使用的工厂是HeapChannelBufferFactory。

#### 1.属性和构造方法

```
/**
 * 通道缓存区工厂
 */
private final ChannelBufferFactory factory;

/**
 * 通道缓存区
 */
private ChannelBuffer buffer;

public DynamicChannelBuffer(int estimatedLength) {
    // 默认是HeapChannelBufferFactory
    this(estimatedLength, HeapChannelBufferFactory.getInstance());
}

public DynamicChannelBuffer(int estimatedLength, ChannelBufferFactory factory) {
    // 如果预计长度小于0 则抛出异常
    if (estimatedLength < 0) {
        throw new IllegalArgumentException("estimatedLength: " + estimatedLength);
    }
    // 如果工厂为空，则抛出空指针异常
    if (factory == null) {
        throw new NullPointerException("factory");
    }
    // 设置工厂
    this.factory = factory;
    // 创建缓存区
    buffer = factory.getBuffer(estimatedLength);
}
```

可以看到，该类有两个属性，所有的实现方法都是调用了buffer的方法，不过该buffer产生是通过工厂动态生成的。并且从构造方法来看，默认使用HeapChannelBufferFactory。

#### 2.ensureWritableBytes

```
@Override
public void ensureWritableBytes(int minWritableBytes) {
    // 如果最小写入的字节数不大于可写的字节数，则结束
    if (minWritableBytes <= writableBytes()) {
        return;
    }

    // 新增容量
    int newCapacity;
    // 此缓冲区可包含的字节数等于0。
    if (capacity() == 0) {
        // 新增容量设置为1
        newCapacity = 1;
    } else {
        // 新增容量设置为缓冲区可包含的字节数
        newCapacity = capacity();
    }
    // 最小新增容量 = 当前的写索引+最小写入的字节数
    int minNewCapacity = writerIndex() + minWritableBytes;
    // 当新增容量比最小新增容量小
    while (newCapacity < minNewCapacity) {
        // 新增容量左移1位，也就是加倍
        newCapacity <<= 1;
    }

    // 通过工厂创建该容量大小当缓冲区
    ChannelBuffer newBuffer = factory().getBuffer(newCapacity);
    // 从buffer中读取数据到newBuffer中
    newBuffer.writeBytes(buffer, 0, writerIndex());
    // 替换原来到缓冲区
    buffer = newBuffer;
}
```

该方法是确保数组有可写的容量，该方法是重写了父类的方法，通过传入一个最小写入的字节数，来对缓冲区进行扩容，可以看到，当现有的缓冲区不够大的时候，会对缓冲区进行加倍对扩容，直到buffer的大小大于传入的最小可写字节数。

#### 3.copy

```
@Override
public ChannelBuffer copy(int index, int length) {
    // 创建缓冲区，预计长度最小为64，或者更大
    DynamicChannelBuffer copiedBuffer = new DynamicChannelBuffer(Math.max(length, 64), factory());
    // 复制数据
    copiedBuffer.buffer = buffer.copy(index, length);
    // 设置索引，读索引设置为0，写索引设置为copy的数据长度
    copiedBuffer.setIndex(0, length);
    // 返回缓存区
    return copiedBuffer;
}
```

该方法是复制数据，在创建缓冲区的时候，预计长度最小是64，，然后重新设置读索引写索引。

其他方法都调用了buffer的方法或者调用了父类的方法，所以不再这里多说。

### （四）ByteBufferBackedChannelBuffer

该方法继承AbstractChannelBuffer，该类是基于 Java NIO中的ByteBuffer来实现相关的读写数据等操作。

```
/**
 * ByteBuffer实例
 */
private final ByteBuffer buffer;

/**
 * 容量
 */
private final int capacity;

public ByteBufferBackedChannelBuffer(ByteBuffer buffer) {
    if (buffer == null) {
        throw new NullPointerException("buffer");
    }

    // 创建一个新的字节缓冲区，新缓冲区的大小将是此缓冲区的剩余容量
    this.buffer = buffer.slice();
    // 返回buffer的剩余容量
    capacity = buffer.remaining();
    // 设置写索引
    writerIndex(capacity);
}
```

上述就是该类的属性和构造函数，可以看到它有一个ByteBuffer类型的实例，并且capacity是buffer的剩余容量。

还有其他的方法比如getByte方法是从buffer中读取数据方法，setBytes方法是把数据写入buffer，它们都有很多重载方法，为就不一一讲解了，它们都是调用了ByteBuffer中的一些方法，如果对于Java NIO中的ByteBuffer方法不是很熟悉的朋友，需要先了解一下Java NIO中的ByteBuffer。

### （五）HeapChannelBuffer

该方法继承了AbstractChannelBuffer，该类中buffer是基于字节数组实现

```
/**
 * The underlying heap byte array that this buffer is wrapping.
 * 此缓冲区包装的基础堆字节数组。
 */
protected final byte[] array;

/**
 * Creates a new heap buffer with a newly allocated byte array.
 * 使用新分配的字节数组创建新的堆缓冲区。
 *
 * @param length the length of the new byte array
 */
public HeapChannelBuffer(int length) {
    this(new byte[length], 0, 0);
}

/**
 * Creates a new heap buffer with an existing byte array.
 * 使用现有字节数组创建新的堆缓冲区。
 *
 * @param array the byte array to wrap
 */
public HeapChannelBuffer(byte[] array) {
    this(array, 0, array.length);
}

/**
 * Creates a new heap buffer with an existing byte array.
 * 使用现有字节数组创建新的堆缓冲区。
 *
 * @param array       the byte array to wrap
 * @param readerIndex the initial reader index of this buffer
 * @param writerIndex the initial writer index of this buffer
 */
protected HeapChannelBuffer(byte[] array, int readerIndex, int writerIndex) {
    if (array == null) {
        throw new NullPointerException("array");
    }
    this.array = array;
    setIndex(readerIndex, writerIndex);
}
```

该类有好几个构造函数，都是基于字节数组的，也就是在该类中包装了一个字节数组，把构造函数传入的字节数组传入到该属性中。其他方法逻辑比较简单。

### （六）ChannelBufferFactory

```
public interface ChannelBufferFactory {

    /**
     * 获得缓冲区实例
     * @param capacity
     * @return
     */
    ChannelBuffer getBuffer(int capacity);

    ChannelBuffer getBuffer(byte[] array, int offset, int length);

    ChannelBuffer getBuffer(ByteBuffer nioBuffer);

}
```

该接口是通道缓冲区工厂，其中就只定义了获得通道缓冲区的方法，比较好理解，它有两个实现类，我后续会讲到。

### （七）HeapChannelBufferFactory

该类实现了ChannelBufferFactory，该类就是基于字节数组来创建缓冲区的工厂。

```
public class HeapChannelBufferFactory implements ChannelBufferFactory {

    /**
     * 单例
     */
    private static final HeapChannelBufferFactory INSTANCE = new HeapChannelBufferFactory();

    public HeapChannelBufferFactory() {
        super();
    }

    public static ChannelBufferFactory getInstance() {
        return INSTANCE;
    }

    @Override
    public ChannelBuffer getBuffer(int capacity) {
        // 创建一个capacity容量的缓冲区
        return ChannelBuffers.buffer(capacity);
    }

    @Override
    public ChannelBuffer getBuffer(byte[] array, int offset, int length) {
        return ChannelBuffers.wrappedBuffer(array, offset, length);
    }

    @Override
    public ChannelBuffer getBuffer(ByteBuffer nioBuffer) {
        // 判断该缓冲区是否有字节数组支持
        if (nioBuffer.hasArray()) {
            // 使用
            return ChannelBuffers.wrappedBuffer(nioBuffer);
        }

        // 创建一个nioBuffer剩余容量的缓冲区
        ChannelBuffer buf = getBuffer(nioBuffer.remaining());
        // 记录下nioBuffer的位置
        int pos = nioBuffer.position();
        // 写入数据到buf
        buf.writeBytes(nioBuffer);
        // 把nioBuffer的位置重置到pos
        nioBuffer.position(pos);
        return buf;
    }

}
```

该类利用了单例模式，其中的方法比较简单，就是调用了ChannelBuffers中的方法，调用的方法实际上还是使用了HeapChannelBuffer中创建缓冲区的方法。

### （八）DirectChannelBufferFactory

该类实现了ChannelBufferFactory接口，是直接缓冲区工厂，用来创建直接缓冲区。

```
public class DirectChannelBufferFactory implements ChannelBufferFactory {

    /**
     * 单例
     */
    private static final DirectChannelBufferFactory INSTANCE = new DirectChannelBufferFactory();

    public DirectChannelBufferFactory() {
        super();
    }

    public static ChannelBufferFactory getInstance() {
        return INSTANCE;
    }

    @Override
    public ChannelBuffer getBuffer(int capacity) {
        if (capacity < 0) {
            throw new IllegalArgumentException("capacity: " + capacity);
        }
        if (capacity == 0) {
            return ChannelBuffers.EMPTY_BUFFER;
        }
        // 生成直接缓冲区
        return ChannelBuffers.directBuffer(capacity);
    }

    @Override
    public ChannelBuffer getBuffer(byte[] array, int offset, int length) {
        if (array == null) {
            throw new NullPointerException("array");
        }
        if (offset < 0) {
            throw new IndexOutOfBoundsException("offset: " + offset);
        }
        if (length == 0) {
            return ChannelBuffers.EMPTY_BUFFER;
        }
        if (offset + length > array.length) {
            throw new IndexOutOfBoundsException("length: " + length);
        }

        ChannelBuffer buf = getBuffer(length);
        buf.writeBytes(array, offset, length);
        return buf;
    }

    @Override
    public ChannelBuffer getBuffer(ByteBuffer nioBuffer) {
        // 如果nioBuffer不是只读，并且它是直接缓冲区
        if (!nioBuffer.isReadOnly() && nioBuffer.isDirect()) {
            // 创建一个缓冲区
            return ChannelBuffers.wrappedBuffer(nioBuffer);
        }

        // 创建一个nioBuffer剩余容量的缓冲区
        ChannelBuffer buf = getBuffer(nioBuffer.remaining());
        // 记录下nioBuffer的位置
        int pos = nioBuffer.position();
        // 写入数据到buf
        buf.writeBytes(nioBuffer);
        // 把nioBuffer的位置重置到pos
        nioBuffer.position(pos);
        return buf;
    }
```

该类中的实现方式与HeapChannelBufferFactory中的实现方式差不多，唯一的区别就是它创建的是一个直接缓冲区。

### （九）ChannelBuffers

该类是缓冲区的工具类，提供创建、比较 ChannelBuffer 等公用方法。我在这里举两个方法来讲：

```
public static ChannelBuffer wrappedBuffer(ByteBuffer buffer) {
    // 如果缓冲区没有剩余容量
    if (!buffer.hasRemaining()) {
        return EMPTY_BUFFER;
    }
    // 如果是字节数组生成的缓冲区
    if (buffer.hasArray()) {
        // 使用buffer的字节数组生成一个新的缓冲区
        return wrappedBuffer(buffer.array(), buffer.arrayOffset() + buffer.position(), buffer.remaining());
    } else {
        // 基于ByteBuffer创建一个缓冲区（利用buffer的剩余容量创建）
        return new ByteBufferBackedChannelBuffer(buffer);
    }
}
```

该方法通过buffer来创建一个新的缓冲区。可以看出来调用的就是上述生成缓冲区的三个类中的方法，ChannelBuffers中很多方法都是这样去实现的，逻辑比较简单。

```
public static boolean equals(ChannelBuffer bufferA, ChannelBuffer bufferB) {
    // 获得bufferA的可读数据
    final int aLen = bufferA.readableBytes();
    // 如果两个缓冲区的可读数据大小不一样，则不是同一个
    if (aLen != bufferB.readableBytes()) {
        return false;
    }

    final int byteCount = aLen & 7;

    // 获得两个比较的缓冲区的读索引
    int aIndex = bufferA.readerIndex();
    int bIndex = bufferB.readerIndex();

    // 最多比较缓冲区中的7个数据
    for (int i = byteCount; i > 0; i--) {
        // 一旦有一个数据不一样，则不是同一个
        if (bufferA.getByte(aIndex) != bufferB.getByte(bIndex)) {
            return false;
        }
        aIndex++;
        bIndex++;
    }

    return true;
}
```

该方法就是比较两个缓冲区是否为同一个，重写了equals。

### （十）ChannelBufferOutputStream

该类继承了OutputStream

#### 1.属性和构造方法

```
/**
 * 缓冲区
 */
private final ChannelBuffer buffer;
/**
 * 记录开始写入的索引
 */
private final int startIndex;

public ChannelBufferOutputStream(ChannelBuffer buffer) {
    if (buffer == null) {
        throw new NullPointerException("buffer");
    }
    this.buffer = buffer;
    // 把开始写入数据的索引记录下来
    startIndex = buffer.writerIndex();
}
```

该类中包装了一个缓冲区对象和startIndex，startIndex是记录开始写入的索引。

#### 2.writtenBytes

```
public int writtenBytes() {
    return buffer.writerIndex() - startIndex;
}
```

该方法是返回写入了多少数据。

该类里面还有write方法，都是调用了buffer.writeBytes。

### （十一）ChannelBufferInputStream

该类继承了InputStream

#### 1.属性和构造函数

```
/**
 * 缓冲区
 */
private final ChannelBuffer buffer;
/**
 * 记录开始读数据的索引
 */
private final int startIndex;
/**
 * 结束读数据的索引
 */
private final int endIndex;

public ChannelBufferInputStream(ChannelBuffer buffer) {
    this(buffer, buffer.readableBytes());
}

public ChannelBufferInputStream(ChannelBuffer buffer, int length) {
    if (buffer == null) {
        throw new NullPointerException("buffer");
    }
    if (length < 0) {
        throw new IllegalArgumentException("length: " + length);
    }
    if (length > buffer.readableBytes()) {
        throw new IndexOutOfBoundsException();
    }

    this.buffer = buffer;
    // 记录开始读数据的索引
    startIndex = buffer.readerIndex();
    // 设置结束读数据的索引
    endIndex = startIndex + length;
    // 标记读索引
    buffer.markReaderIndex();
}
```

该类里面包装了读开始索引和结束索引，并且在构造方法中初始化这些属性。

#### 2.readBytes

```
public int readBytes() {
    return buffer.readerIndex() - startIndex;
}
```

该方法是返回读了多少数据。

#### 3.available

```
@Override
public int available() throws IOException {
    return endIndex - buffer.readerIndex();
}
```

该方法是返回还剩多少数据没读

#### 4.read

```
@Override
public int read() throws IOException {
    if (!buffer.readable()) {
        return -1;
    }
    return buffer.readByte() & 0xff;
}

@Override
public int read(byte[] b, int off, int len) throws IOException {
    // 判断是否还有数据可读
    int available = available();
    if (available == 0) {
        return -1;
    }

    // 获得需要读取的数据长度
    len = Math.min(available, len);
    buffer.readBytes(b, off, len);
    return len;
}
```

该方法是读数据，返回读了数据长度。

#### 5.skip

```
@Override
public long skip(long n) throws IOException {
    if (n > Integer.MAX_VALUE) {
        return skipBytes(Integer.MAX_VALUE);
    } else {
        return skipBytes((int) n);
    }
}

private int skipBytes(int n) throws IOException {
    int nBytes = Math.min(available(), n);
    // 跳过一些数据
    buffer.skipBytes(nBytes);
    return nBytes;
}
```

该方法是跳过n长度来读数据。

# 远程通讯——Http

## 前言

本文我们讲解的是如何基于Tomcat或者Jetty实现HTTP服务器。Tomcat和Jetty都是一种servlet引擎，Jetty要比Tomcat的架构更简单一些。关于它们之间的比较，我觉得google一些更加方便，我就不多废话了 。

下面是dubbo-remoting-http包下的类图：

![dubbo-remoting-http类图](https://segmentfault.com/img/remote/1460000017508552)

可以看到这个包下只提供了服务端实现，并没有客户端实现。

## 源码分析

### （一）HttpServer

```
public interface HttpServer extends Resetable {

    /**
     * get http handler.
     * 获得http的处理类
     * @return http handler.
     */
    HttpHandler getHttpHandler();

    /**
     * get url.
     * 获得url
     * @return url
     */
    URL getUrl();

    /**
     * get local address.
     * 获得本地服务器地址
     * @return local address.
     */
    InetSocketAddress getLocalAddress();

    /**
     * close the channel.
     * 关闭通道
     */
    void close();

    /**
     * Graceful close the channel.
     * 优雅的关闭通道
     */
    void close(int timeout);

    /**
     * is bound.
     * 是否绑定
     * @return bound
     */
    boolean isBound();

    /**
     * is closed.
     * 服务器是否关闭
     * @return closed
     */
    boolean isClosed();

}
```

该接口是http服务器的接口，定义了服务器相关的方法，都比较好理解。

### （二）AbstractHttpServer

该类实现接口HttpServer，是http服务器接口的抽象类。

```
/**
 * url
 */
private final URL url;

/**
 * http服务器处理器
 */
private final HttpHandler handler;

/**
 * 该服务器是否关闭
 */
private volatile boolean closed;

public AbstractHttpServer(URL url, HttpHandler handler) {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    this.url = url;
    this.handler = handler;
}
```

该类中有三个属性，关键是看在后面怎么用到，因为该类是抽象类，所以该类中的方法并没有实现具体的逻辑，我们继续往下看它的三个子类。

### （三）TomcatHttpServer

该类是基于Tomcat来实现服务器的实现类，它继承了AbstractHttpServer。

#### 1.属性

```
/**
 * 内嵌的tomcat对象
 */
private final Tomcat tomcat;

/**
 * url对象
 */
private final URL url;
```

该类的两个属性，关键是内嵌的tomcat。

#### 2.构造方法

```
    public TomcatHttpServer(URL url, final HttpHandler handler) {
        super(url, handler);

        this.url = url;
        // 添加处理器
        DispatcherServlet.addHttpHandler(url.getPort(), handler);
        // 获得java.io.tmpdir的绝对路径目录
        String baseDir = new File(System.getProperty("java.io.tmpdir")).getAbsolutePath();
        // 创建内嵌的tomcat对象
        tomcat = new Tomcat();
        // 设置根目录
        tomcat.setBaseDir(baseDir);
        // 设置端口号
        tomcat.setPort(url.getPort());
        // 给默认的http连接器。设置最大线程数
        tomcat.getConnector().setProperty(
                "maxThreads", String.valueOf(url.getParameter(Constants.THREADS_KEY, Constants.DEFAULT_THREADS)));
//        tomcat.getConnector().setProperty(
//                "minSpareThreads", String.valueOf(url.getParameter(Constants.THREADS_KEY, Constants.DEFAULT_THREADS)));

        // 设置最大的连接数
        tomcat.getConnector().setProperty(
                "maxConnections", String.valueOf(url.getParameter(Constants.ACCEPTS_KEY, -1)));

        // 设置URL编码格式
        tomcat.getConnector().setProperty("URIEncoding", "UTF-8");
        // 设置连接超时事件为60s
        tomcat.getConnector().setProperty("connectionTimeout", "60000");

        // 设置最大长连接个数为不限制个数
        tomcat.getConnector().setProperty("maxKeepAliveRequests", "-1");
        // 设置将由连接器使用的Coyote协议。
        tomcat.getConnector().setProtocol("org.apache.coyote.http11.Http11NioProtocol");

        // 添加上下文
        Context context = tomcat.addContext("/", baseDir);
        // 添加servlet，把servlet添加到context
        Tomcat.addServlet(context, "dispatcher", new DispatcherServlet());
        // 添加servlet映射
        context.addServletMapping("/*", "dispatcher");
        // 添加servlet上下文
        ServletManager.getInstance().addServletContext(url.getPort(), context.getServletContext());

        try {
            // 开启tomcat
            tomcat.start();
        } catch (LifecycleException e) {
            throw new IllegalStateException("Failed to start tomcat server at " + url.getAddress(), e);
        }
    }
```

该方法的构造函数中就启动了tomcat，前面很多都是对tomcat的启动参数以及配置设置。如果有过tomcat配置经验的朋友应该看起来很简单。

#### 3.close

```
@Override
public void close() {
    super.close();

    // 移除相关的servlet上下文
    ServletManager.getInstance().removeServletContext(url.getPort());

    try {
        // 停止tomcat
        tomcat.stop();
    } catch (Exception e) {
        logger.warn(e.getMessage(), e);
    }
}
```

该方法是关闭服务器的方法。调用了tomcat.stop

### （四）JettyHttpServer

该类是基于Jetty来实现服务器的实现类，它继承了AbstractHttpServer。

#### 1.属性

```
/**
 * 内嵌的Jetty服务器对象
 */
private Server server;

/**
 * url对象
 */
private URL url;
```

该类的两个属性，关键是内嵌的sever，它是内嵌的etty服务器对象。

#### 2.构造方法

```
   public JettyHttpServer(URL url, final HttpHandler handler) {
        super(url, handler);
        this.url = url;
        // TODO we should leave this setting to slf4j
        // we must disable the debug logging for production use
        // 设置日志
        Log.setLog(new StdErrLog());
        // 禁用调试用的日志
        Log.getLog().setDebugEnabled(false);

        // 添加http服务器处理器
        DispatcherServlet.addHttpHandler(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()), handler);

        // 获得线程数
        int threads = url.getParameter(Constants.THREADS_KEY, Constants.DEFAULT_THREADS);
        // 创建线程池
        QueuedThreadPool threadPool = new QueuedThreadPool();
        // 设置线程池配置
        threadPool.setDaemon(true);
        threadPool.setMaxThreads(threads);
        threadPool.setMinThreads(threads);

        // 创建选择NIO连接器
        SelectChannelConnector connector = new SelectChannelConnector();

        // 获得绑定的ip
        String bindIp = url.getParameter(Constants.BIND_IP_KEY, url.getHost());
        if (!url.isAnyHost() && NetUtils.isValidLocalHost(bindIp)) {
            // 设置主机地址
            connector.setHost(bindIp);
        }
        // 设置端口号
        connector.setPort(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()));

        // 创建Jetty服务器对象
        server = new Server();
        // 设置线程池
        server.setThreadPool(threadPool);
        // 设置连接器
        server.addConnector(connector);

        // 添加DispatcherServlet到jetty
        ServletHandler servletHandler = new ServletHandler();
        ServletHolder servletHolder = servletHandler.addServletWithMapping(DispatcherServlet.class, "/*");
        servletHolder.setInitOrder(2);

        // dubbo's original impl can't support the use of ServletContext
//        server.addHandler(servletHandler);
        // TODO Context.SESSIONS is the best option here?
        Context context = new Context(server, "/", Context.SESSIONS);
        context.setServletHandler(servletHandler);
        // 添加 ServletContext 对象，到 ServletManager 中
        ServletManager.getInstance().addServletContext(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()), context.getServletContext());

        try {
            // 启动jetty服务器
            server.start();
        } catch (Exception e) {
            throw new IllegalStateException("Failed to start jetty server on " + url.getParameter(Constants.BIND_IP_KEY) + ":" + url.getParameter(Constants.BIND_PORT_KEY) + ", cause: "
                    + e.getMessage(), e);
        }
    }
```

可以看到它跟TomcatHttpServer中构造函数不同的是API的不同，不过思路差不多，先设置启动参数和配置，然后启动jetty服务器。

#### 3.close

```
@Override
public void close() {
    super.close();

    // 移除 ServletContext 对象
    ServletManager.getInstance().removeServletContext(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()));

    if (server != null) {
        try {
            // 停止服务器
            server.stop();
        } catch (Exception e) {
            logger.warn(e.getMessage(), e);
        }
    }
}
```

该方法是关闭服务器的方法，调用的是server的stop方法。

### （五）ServletHttpServer

该类继承了AbstractHttpServer，是基于 Servlet 的服务器实现类。

```
public class ServletHttpServer extends AbstractHttpServer {

    public ServletHttpServer(URL url, HttpHandler handler) {
        super(url, handler);
        // /把 HttpHandler 到 DispatcherServlet 中，默认端口为8080
        DispatcherServlet.addHttpHandler(url.getParameter(Constants.BIND_PORT_KEY, 8080), handler);
    }

}
```

该类就一个构造方法。就是把服务器处理器注册到DispatcherServlet上。

### （六）HttpBinder

```
@SPI("jetty")
public interface HttpBinder {

    /**
     * bind the server.
     * 绑定到服务器
     * @param url server url.
     * @return server.
     */
    @Adaptive({Constants.SERVER_KEY})
    HttpServer bind(URL url, HttpHandler handler);

}
```

该接口是http绑定器接口，其中就定义了一个方法就是绑定方法，并且返回服务器对象。该接口是一个可扩展接口，默认实现JettyHttpBinder。它有三个实现类，请往下看。

### （七）TomcatHttpBinder

```
public class TomcatHttpBinder implements HttpBinder {

    @Override
    public HttpServer bind(URL url, HttpHandler handler) {
        // 创建一个TomcatHttpServer
        return new TomcatHttpServer(url, handler);
    }

}
```

一眼就看出来，就是创建了个基于Tomcat实现的服务器TomcatHttpServer对象。具体的就往上看TomcatHttpServer里面的实现。

### （八）JettyHttpBinder

```
public class JettyHttpBinder implements HttpBinder {

    @Override
    public HttpServer bind(URL url, HttpHandler handler) {
        // 创建JettyHttpServer实例
        return new JettyHttpServer(url, handler);
    }

}
```

一眼就看出来，就是创建了个基于Jetty实现的服务器JettyHttpServer对象。具体的就往上看JettyHttpServer里面的实现。

### （九）ServletHttpBinder

```
public class ServletHttpBinder implements HttpBinder {

    @Override
    @Adaptive()
    public HttpServer bind(URL url, HttpHandler handler) {
        // 创建ServletHttpServer对象
        return new ServletHttpServer(url, handler);
    }

}
```

创建了个基于servlet实现的服务器ServletHttpServer对象。并且方法上加入了Adaptive，用到了dubbo SPI机制。

### （十）BootstrapListener

```
public class BootstrapListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        // context创建的时候，把ServletContext添加到ServletManager
        ServletManager.getInstance().addServletContext(ServletManager.EXTERNAL_SERVER_PORT, servletContextEvent.getServletContext());
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        // context销毁到时候，把servletContextEvent移除
        ServletManager.getInstance().removeServletContext(ServletManager.EXTERNAL_SERVER_PORT);
    }
}
```

该类实现了ServletContextListener，是 启动监听器，当context创建和销毁的时候对ServletContext做处理。不过需要配置BootstrapListener到web.xml，通过这样的方式，让外部的 ServletContext 对象，添加到 ServletManager 中。

### （十一）DispatcherServlet

```
public class DispatcherServlet extends HttpServlet {

    private static final long serialVersionUID = 5766349180380479888L;
    /**
     * http服务器处理器
     */
    private static final Map<Integer, HttpHandler> handlers = new ConcurrentHashMap<Integer, HttpHandler>();
    /**
     * 单例
     */
    private static DispatcherServlet INSTANCE;

    public DispatcherServlet() {
        DispatcherServlet.INSTANCE = this;
    }

    /**
     * 添加处理器
     * @param port
     * @param processor
     */
    public static void addHttpHandler(int port, HttpHandler processor) {
        handlers.put(port, processor);
    }

    public static void removeHttpHandler(int port) {
        handlers.remove(port);
    }

    public static DispatcherServlet getInstance() {
        return INSTANCE;
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 获得处理器
        HttpHandler handler = handlers.get(request.getLocalPort());
        // 如果处理器不存在
        if (handler == null) {// service not found.
            // 返回404
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "Service not found.");
        } else {
            // 处理请求
            handler.handle(request, response);
        }
    }

}
```

该类继承了HttpServlet，是服务请求调度servlet类，主要是service方法，根据请求来调度不同的处理器去处理请求，如果没有该处理器，则报错404

### （十二）ServletManager

```
public class ServletManager {

    /**
     * 外部服务器端口，用于 `servlet` 的服务器端口
     */
    public static final int EXTERNAL_SERVER_PORT = -1234;

    /**
     * 单例
     */
    private static final ServletManager instance = new ServletManager();

    /**
     * ServletContext 集合
     */
    private final Map<Integer, ServletContext> contextMap = new ConcurrentHashMap<Integer, ServletContext>();

    public static ServletManager getInstance() {
        return instance;
    }

    /**
     * 添加ServletContext
     * @param port
     * @param servletContext
     */
    public void addServletContext(int port, ServletContext servletContext) {
        contextMap.put(port, servletContext);
    }

    /**
     * 移除ServletContext
     * @param port
     */
    public void removeServletContext(int port) {
        contextMap.remove(port);
    }

    /**
     * 获得ServletContext对象
     * @param port
     * @return
     */
    public ServletContext getServletContext(int port) {
        return contextMap.get(port);
    }
}
```

该类是servlet的管理器，管理着ServletContext。

### （十三）HttpHandler

```
public interface HttpHandler {

    /**
     * invoke.
     * HTTP请求处理
     * @param request  request.
     * @param response response.
     * @throws IOException
     * @throws ServletException
     */
    void handle(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException;

}
```

该接口是HTTP 处理器接口，就定义了一个处理请求的方法。

# 远程通讯——Netty4

# 远程通讯——Netty4

> 目标：介绍基于netty4的来实现的远程通信、介绍dubbo-remoting-netty4内的源码解析。

## 前言

netty4对netty3兼容性不是很好，并且netty4在很多的术语和api也发生了改变，导致升级netty4会很艰辛，网上应该有很多相关文章，高版本的总有高版本的优势所在，所以dubbo也需要与时俱进，又新增了基于netty4来实现远程通讯模块。下面讲解的，如果跟上一篇文章有重复的地方我就略过去了。关键还是要把远程通讯的api那几篇看懂，看这几篇实现才会很简单。

下面是包的结构：

![dubbo-remoting-netty4包结构](https://segmentfault.com/img/remote/1460000017553205)

## 源码分析

### （一）NettyChannel

该类继承了AbstractChannel，是基于netty4的通道实现类

#### 1.属性

```
/**
 * 通道集合
 */
private static final ConcurrentMap<Channel, NettyChannel> channelMap = new ConcurrentHashMap<Channel, NettyChannel>();

/**
 * 通道
 */
private final Channel channel;

/**
 * 属性集合
 */
private final Map<String, Object> attributes = new ConcurrentHashMap<String, Object>();
```

属性跟netty3实现的通道类属性几乎一样，我就不讲解了。

#### 2.getOrAddChannel

```
static NettyChannel getOrAddChannel(Channel ch, URL url, ChannelHandler handler) {
    if (ch == null) {
        return null;
    }
    // 首先从集合中取通道
    NettyChannel ret = channelMap.get(ch);
    // 如果为空，则新建
    if (ret == null) {
        NettyChannel nettyChannel = new NettyChannel(ch, url, handler);
        // 如果通道还活跃着
        if (ch.isActive()) {
            // 加入集合
            ret = channelMap.putIfAbsent(ch, nettyChannel);
        }
        if (ret == null) {
            ret = nettyChannel;
        }
    }
    return ret;
}
```

该方法是获得通道，如果集合中没有找到对应通道，则创建一个，然后加入集合。

#### 3.send

```
@Override
public void send(Object message, boolean sent) throws RemotingException {
    super.send(message, sent);

    boolean success = true;
    int timeout = 0;
    try {
        // 写入数据，发送消息
        ChannelFuture future = channel.writeAndFlush(message);
        // 如果已经发送过
        if (sent) {
            // 获得超时时间
            timeout = getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            // 等待timeout的连接时间后查看是否发送成功
            success = future.await(timeout);
        }
        // 获得异常
        Throwable cause = future.cause();
        // 如果异常不为空，则抛出异常
        if (cause != null) {
            throw cause;
        }
    } catch (Throwable e) {
        throw new RemotingException(this, "Failed to send message " + message + " to " + getRemoteAddress() + ", cause: " + e.getMessage(), e);
    }

    if (!success) {
        throw new RemotingException(this, "Failed to send message " + message + " to " + getRemoteAddress()
                + "in timeout(" + timeout + "ms) limit");
    }
}
```

该方法是发送消息，调用了channel.writeAndFlush方法，与netty3的实现只是调用的api不同。

#### 4.close

```
@Override
public void close() {
    try {
        super.close();
    } catch (Exception e) {
        logger.warn(e.getMessage(), e);
    }
    try {
        // 移除通道
        removeChannelIfDisconnected(channel);
    } catch (Exception e) {
        logger.warn(e.getMessage(), e);
    }
    try {
        // 清理属性集合
        attributes.clear();
    } catch (Exception e) {
        logger.warn(e.getMessage(), e);
    }
    try {
        if (logger.isInfoEnabled()) {
            logger.info("Close netty channel " + channel);
        }
        // 关闭通道
        channel.close();
    } catch (Exception e) {
        logger.warn(e.getMessage(), e);
    }
}
```

该方法就是操作了四个步骤，比较清晰。

### （二）NettyClientHandler

该类继承了ChannelDuplexHandler，是基于netty4实现的客户端通道处理实现类。这里的设计与netty3实现的通道处理器有所不同，netty3实现的通道处理器是被客户端和服务端统一使用的，而在这里服务端和客户端使用了两个不同的Handler来处理。并且netty3的NettyHandler是基于netty3的SimpleChannelHandler设计的，而这里是基于netty4的ChannelDuplexHandler。

```
/**
 * url对象
 */
private final URL url;

/**
 * 通道
 */
private final ChannelHandler handler;
```

该类的属性只有两个，下面实现的方法也都是调用了handler的方法，我就举一个例子：

```
@Override
public void disconnect(ChannelHandlerContext ctx, ChannelPromise future)
        throws Exception {
    // 获得通道
    NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
    try {
        // 断开连接
        handler.disconnected(channel);
    } finally {
        // 从集合中移除
        NettyChannel.removeChannelIfDisconnected(ctx.channel());
    }
}
```

可以看到分了三部，获得通道对象，调用handler方法，最后检测一下通道是否活跃。其他方法也是差不多。

### （三）NettyServerHandler

该类继承了ChannelDuplexHandler，是基于netty4实现的服务端通道处理实现类。

```
/**
 * 连接该服务器的通道数 key为ip:port
 */
private final Map<String, Channel> channels = new ConcurrentHashMap<String, Channel>(); // <ip:port, channel>

/**
 * url对象
 */
private final URL url;

/**
 * 通道处理器
 */
private final ChannelHandler handler;
```

该类有三个属性，比NettyClientHandler多了一个属性channels，下面的实现方法也是一样的，都是调用了handler方法，来看一个例子：

```
@Override
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    // 激活事件
    ctx.fireChannelActive();

    // 获得通道
    NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
    try {
        // 如果通道不为空，则加入集合中
        if (channel != null) {
            channels.put(NetUtils.toAddressString((InetSocketAddress) ctx.channel().remoteAddress()), channel);
        }
        // 连接该通道
        handler.connected(channel);
    } finally {
        // 如果通道不活跃，则移除通道
        NettyChannel.removeChannelIfDisconnected(ctx.channel());
    }
}
```

该方法是通道活跃的时候调用了handler.connected，差不多也是常规套路，就多了激活事件和加入到通道中。其他方法也差不多。

### （四）NettyClient

该类继承了AbstractClient，是基于netty4实现的客户端实现类。

#### 1.属性

```
/**
 * NioEventLoopGroup对象
 */
private static final NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup(Constants.DEFAULT_IO_THREADS, new DefaultThreadFactory("NettyClientWorker", true));

/**
 * 客户端引导类
 */
private Bootstrap bootstrap;

/**
 * 通道
 */
private volatile Channel channel; // volatile, please copy reference to use
```

属性里的NioEventLoopGroup对象是netty4中的对象，什么用处请看netty的解析。

#### 2.doOpen

```
@Override
protected void doOpen() throws Throwable {
    // 创建一个客户端的通道处理器
    final NettyClientHandler nettyClientHandler = new NettyClientHandler(getUrl(), this);
    // 创建一个引导类
    bootstrap = new Bootstrap();
    // 设置可选项
    bootstrap.group(nioEventLoopGroup)
            .option(ChannelOption.SO_KEEPALIVE, true)
            .option(ChannelOption.TCP_NODELAY, true)
            .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
            //.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, getTimeout())
            .channel(NioSocketChannel.class);

    // 如果连接超时时间小于3s，则设置为3s，也就是说最低的超时时间为3s
    if (getConnectTimeout() < 3000) {
        bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000);
    } else {
        bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, getConnectTimeout());
    }

    // 创建一个客户端
    bootstrap.handler(new ChannelInitializer() {

        @Override
        protected void initChannel(Channel ch) throws Exception {
            // 编解码器
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyClient.this);
            // 加入责任链
            ch.pipeline()//.addLast("logging",new LoggingHandler(LogLevel.INFO))//for debug
                    .addLast("decoder", adapter.getDecoder())
                    .addLast("encoder", adapter.getEncoder())
                    .addLast("handler", nettyClientHandler);
        }
    });
}
```

该方法还是做了创建客户端，并且打开的操作，其中很多的参数设置操作。

其他方法跟[ dubbo源码解析（十六）远程通信——Netty3](https://segmentfault.com/a/1190000017530167)中写到的NettyClient实现一样。

### （五）NettyServer

该类继承了AbstractServer，实现了Server。是基于netty4实现的服务器类

```
private static final Logger logger = LoggerFactory.getLogger(NettyServer.class);


/**
 * 连接该服务器的通道集合 key为ip:port
 */
private Map<String, Channel> channels; // <ip:port, channel>

/**
 * 服务器引导类
 */
private ServerBootstrap bootstrap;

/**
 * 通道
 */
private io.netty.channel.Channel channel;

/**
 * boss线程组
 */
private EventLoopGroup bossGroup;
/**
 * worker线程组
 */
private EventLoopGroup workerGroup;
```

属性相较netty3而言，新增了两个线程组，同样也是因为netty3和netty4的设计不同。

#### 2.doOpen

```
@Override
protected void doOpen() throws Throwable {
    // 创建服务引导类
    bootstrap = new ServerBootstrap();

    // 创建boss线程组
    bossGroup = new NioEventLoopGroup(1, new DefaultThreadFactory("NettyServerBoss", true));
    // 创建worker线程组
    workerGroup = new NioEventLoopGroup(getUrl().getPositiveParameter(Constants.IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS),
            new DefaultThreadFactory("NettyServerWorker", true));

    // 创建服务器处理器
    final NettyServerHandler nettyServerHandler = new NettyServerHandler(getUrl(), this);
    // 获得通道集合
    channels = nettyServerHandler.getChannels();

    // 设置ventLoopGroup还有可选项
    bootstrap.group(bossGroup, workerGroup)
            .channel(NioServerSocketChannel.class)
            .childOption(ChannelOption.TCP_NODELAY, Boolean.TRUE)
            .childOption(ChannelOption.SO_REUSEADDR, Boolean.TRUE)
            .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
            .childHandler(new ChannelInitializer<NioSocketChannel>() {
                @Override
                protected void initChannel(NioSocketChannel ch) throws Exception {
                    // 编解码器
                    NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
                    // 增加责任链
                    ch.pipeline()//.addLast("logging",new LoggingHandler(LogLevel.INFO))//for debug
                            .addLast("decoder", adapter.getDecoder())
                            .addLast("encoder", adapter.getEncoder())
                            .addLast("handler", nettyServerHandler);
                }
            });
    // bind 绑定
    ChannelFuture channelFuture = bootstrap.bind(getBindAddress());
    // 等待绑定完成
    channelFuture.syncUninterruptibly();
    // 设置通道
    channel = channelFuture.channel();

}
```

该方法是创建服务器，并且开启。如果熟悉netty4点朋友应该觉得还是很好理解的。其他方法跟[《 dubbo源码解析（十六）远程通信——Netty3》](https://segmentfault.com/a/1190000017530167)中写到的NettyClient实现一样，处理close中要多关闭两个线程组

### （六）NettyTransporter

该类跟[ 《dubbo源码解析（十六）远程通信——Netty3》](https://segmentfault.com/a/1190000017530167)中的NettyTransporter一样的实现。

### （七）NettyCodecAdapter

该类是基于netty4的编解码器。

#### 1.属性

```
/**
 * 编码器
 */
private final ChannelHandler encoder = new InternalEncoder();

/**
 * 解码器
 */
private final ChannelHandler decoder = new InternalDecoder();

/**
 * 编解码器
 */
private final Codec2 codec;

/**
 * url对象
 */
private final URL url;

/**
 * 通道处理器
 */
private final com.alibaba.dubbo.remoting.ChannelHandler handler;
```

属性跟基于netty3实现的编解码一样。

#### 2.InternalEncoder

```
private class InternalEncoder extends MessageToByteEncoder {

    @Override
    protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
        // 创建缓冲区
        com.alibaba.dubbo.remoting.buffer.ChannelBuffer buffer = new NettyBackedChannelBuffer(out);
        // 获得通道
        Channel ch = ctx.channel();
        // 获得netty通道
        NettyChannel channel = NettyChannel.getOrAddChannel(ch, url, handler);
        try {
            // 编码
            codec.encode(channel, buffer, msg);
        } finally {
            // 检测通道是否活跃
            NettyChannel.removeChannelIfDisconnected(ch);
        }
    }
}
```

该内部类是编码器的抽象，主要的编码还是调用了codec.encode。

#### 3.InternalDecoder

```
private class InternalDecoder extends ByteToMessageDecoder {

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf input, List<Object> out) throws Exception {

        // 创建缓冲区
        ChannelBuffer message = new NettyBackedChannelBuffer(input);

        // 获得通道
        NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);

        Object msg;

        int saveReaderIndex;

        try {
            // decode object.
            do {
                // 记录读索引
                saveReaderIndex = message.readerIndex();
                try {
                    // 解码
                    msg = codec.decode(channel, message);
                } catch (IOException e) {
                    throw e;
                }
                // 拆包
                if (msg == Codec2.DecodeResult.NEED_MORE_INPUT) {
                    message.readerIndex(saveReaderIndex);
                    break;
                } else {
                    //is it possible to go here ?
                    if (saveReaderIndex == message.readerIndex()) {
                        throw new IOException("Decode without read data.");
                    }
                    // 读取数据
                    if (msg != null) {
                        out.add(msg);
                    }
                }
            } while (message.readable());
        } finally {
            NettyChannel.removeChannelIfDisconnected(ctx.channel());
        }
    }
}
```

该内部类是解码器的抽象类，其中关键的是调用了codec.decode。

### （八）NettyBackedChannelBuffer

该类是缓冲区类。

```
/**
 * 缓冲区
 */
private ByteBuf buffer;
```

其中的方法几乎都调用了该属性的方法。而ByteBuf是netty4中的字节数据的容器。

### （九）FormattingTuple和MessageFormatter

这两个类是用于用于格式化的，是从netty4中复制出来的，其中并且稍微做了一下改动。我就不再讲解了。

# 远程调用——开篇

其中dubbo-rpc-api是对协议、暴露、引用、代理等的抽象和实现，是rpc整个设计的核心内容。其他的包则是dubbo支持的9种协议，在官方文档也能查看介绍，并且包括一种本地调用injvm。那么我们再来看看dubbo-rpc-api中包结构：

1. filter包：在进行服务引用时会进行一系列的过滤。其中包括了很多过滤器。
2. listener包：看上面两张服务引用和服务暴露的时序图，发现有两个listener，其中的逻辑实现就在这个包内
3. protocol包：这个包实现了协议的一些公共逻辑
4. proxy包：实现了代理的逻辑。
5. service包：其中包含了一个需要调用的方法等封装抽象。
6. support包：包括了工具类
7. 最外层的实现。

下面的篇幅设计，本文会讲解最外层的源码和service下的源码，support包下的源码我会穿插在其他用到的地方一并讲解，filter、listener、protocol、proxy以及各类协议的实现各自用一篇来讲。

## 源码分析

### （一）Invoker

```
public interface Invoker<T> extends Node {

    /**
     * get service interface.
     * 获得服务接口
     * @return service interface.
     */
    Class<T> getInterface();

    /**
     * invoke.
     * 调用下一个会话域
     * @param invocation
     * @return result
     * @throws RpcException
     */
    Result invoke(Invocation invocation) throws RpcException;

}
```

该接口是实体域，它是dubbo的核心模型，其他模型都向它靠拢，或者转化成它，它代表了一个可执行体，可以向它发起invoke调用，这个有可能是一个本地的实现，也可能是一个远程的实现，也可能是一个集群的实现。它代表了一次调用

### （二）Invocation

```
public interface Invocation {

    /**
     * get method name.
     * 获得方法名称
     * @return method name.
     * @serial
     */
    String getMethodName();

    /**
     * get parameter types.
     * 获得参数类型
     * @return parameter types.
     * @serial
     */
    Class<?>[] getParameterTypes();

    /**
     * get arguments.
     * 获得参数
     * @return arguments.
     * @serial
     */
    Object[] getArguments();

    /**
     * get attachments.
     * 获得附加值集合
     * @return attachments.
     * @serial
     */
    Map<String, String> getAttachments();

    /**
     * get attachment by key.
     * 获得附加值
     * @return attachment value.
     * @serial
     */
    String getAttachment(String key);

    /**
     * get attachment by key with default value.
     * 获得附加值
     * @return attachment value.
     * @serial
     */
    String getAttachment(String key, String defaultValue);

    /**
     * get the invoker in current context.
     * 获得当前上下文的invoker
     * @return invoker.
     * @transient
     */
    Invoker<?> getInvoker();

}
```

Invocation 是会话域，它持有调用过程中的变量，比如方法名，参数等。

### （三）Exporter

```
public interface Exporter<T> {

    /**
     * get invoker.
     * 获得对应的实体域invoker
     * @return invoker
     */
    Invoker<T> getInvoker();

    /**
     * unexport.
     * 取消暴露
     * <p>
     * <code>
     * getInvoker().destroy();
     * </code>
     */
    void unexport();

}
```

该接口是暴露服务的接口，定义了两个方法分别是获得invoker和取消暴露服务。

### （四）ExporterListener

```
@SPI
public interface ExporterListener {

    /**
     * The exporter exported.
     * 暴露服务
     * @param exporter
     * @throws RpcException
     * @see com.alibaba.dubbo.rpc.Protocol#export(Invoker)
     */
    void exported(Exporter<?> exporter) throws RpcException;

    /**
     * The exporter unexported.
     * 取消暴露
     * @param exporter
     * @throws RpcException
     * @see com.alibaba.dubbo.rpc.Exporter#unexport()
     */
    void unexported(Exporter<?> exporter);

}
```

该接口是服务暴露的监听器接口，定义了两个方法是暴露和取消暴露，参数都是Exporter类型的。

### （五）Protocol

```
@SPI("dubbo")
public interface Protocol {

    /**
     * Get default port when user doesn't config the port.
     * 获得默认的端口
     * @return default port
     */
    int getDefaultPort();

    /**
     * Export service for remote invocation: <br>
     * 1. Protocol should record request source address after receive a request:
     * RpcContext.getContext().setRemoteAddress();<br>
     * 2. export() must be idempotent, that is, there's no difference between invoking once and invoking twice when
     * export the same URL<br>
     * 3. Invoker instance is passed in by the framework, protocol needs not to care <br>
     * 暴露服务方法，
     * @param <T>     Service type 服务类型
     * @param invoker Service invoker 服务的实体域
     * @return exporter reference for exported service, useful for unexport the service later
     * @throws RpcException thrown when error occurs during export the service, for example: port is occupied
     */
    @Adaptive
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

    /**
     * Refer a remote service: <br>
     * 1. When user calls `invoke()` method of `Invoker` object which's returned from `refer()` call, the protocol
     * needs to correspondingly execute `invoke()` method of `Invoker` object <br>
     * 2. It's protocol's responsibility to implement `Invoker` which's returned from `refer()`. Generally speaking,
     * protocol sends remote request in the `Invoker` implementation. <br>
     * 3. When there's check=false set in URL, the implementation must not throw exception but try to recover when
     * connection fails.
     * 引用服务方法
     * @param <T>  Service type 服务类型
     * @param type Service class 服务类名
     * @param url  URL address for the remote service
     * @return invoker service's local proxy
     * @throws RpcException when there's any error while connecting to the service provider
     */
    @Adaptive
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

    /**
     * Destroy protocol: <br>
     * 1. Cancel all services this protocol exports and refers <br>
     * 2. Release all occupied resources, for example: connection, port, etc. <br>
     * 3. Protocol can continue to export and refer new service even after it's destroyed.
     */
    void destroy();

}
```

该接口是服务域接口，也是协议接口，它是一个可扩展的接口，默认实现的是dubbo协议。定义了四个方法，关键的是服务暴露和引用两个方法。

### （六）Filter

```
@SPI
public interface Filter {

    /**
     * do invoke filter.
     * <p>
     * <code>
     * // before filter
     * Result result = invoker.invoke(invocation);
     * // after filter
     * return result;
     * </code>
     *
     * @param invoker    service
     * @param invocation invocation.
     * @return invoke result.
     * @throws RpcException
     * @see com.alibaba.dubbo.rpc.Invoker#invoke(Invocation)
     */
    Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException;

}
```

该接口是invoker调用时过滤器接口，其中就只有一个invoke方法。在该方法中对调用进行过滤

### （七）InvokerListener

```
@SPI
public interface InvokerListener {

    /**
     * The invoker referred
     * 在服务引用的时候进行监听
     * @param invoker
     * @throws RpcException
     * @see com.alibaba.dubbo.rpc.Protocol#refer(Class, com.alibaba.dubbo.common.URL)
     */
    void referred(Invoker<?> invoker) throws RpcException;

    /**
     * The invoker destroyed.
     * 销毁实体域
     * @param invoker
     * @see com.alibaba.dubbo.rpc.Invoker#destroy()
     */
    void destroyed(Invoker<?> invoker);

}
```

该接口是实体域的监听器，定义了两个方法，分别是服务引用和销毁的时候执行的方法。

### （八）Result

该接口是实体域执行invoke的结果接口，里面定义了获得结果异常以及附加值等方法。比较好理解我就不贴代码了。

### （九）ProxyFactory

```
@SPI("javassist")
public interface ProxyFactory {

    /**
     * create proxy.
     * 创建一个代理
     * @param invoker
     * @return proxy
     */
    @Adaptive({Constants.PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker) throws RpcException;

    /**
     * create proxy.
     * 创建一个代理
     * @param invoker
     * @return proxy
     */
    @Adaptive({Constants.PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker, boolean generic) throws RpcException;

    /**
     * create invoker.
     * 创建一个实体域
     * @param <T>
     * @param proxy
     * @param type
     * @param url
     * @return invoker
     */
    @Adaptive({Constants.PROXY_KEY})
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;

}
```

该接口是代理工厂接口，它也是个可扩展接口，默认实现javassist，dubbo提供两种动态代理方法分别是javassist/jdk，该接口定义了三个方法，前两个方法是通过invoker创建代理，最后一个是通过代理来获得invoker。

### （十）RpcContext

该类就是远程调用的上下文，贯穿着整个调用，例如A调用B，然后B调用C。在服务B上，RpcContext在B之前将调用信息从A保存到B。开始调用C，并在B调用C后将调用信息从B保存到C。RpcContext保存了调用信息。

```
public class RpcContext {

    /**
     * use internal thread local to improve performance
     * 本地上下文
     */
    private static final InternalThreadLocal<RpcContext> LOCAL = new InternalThreadLocal<RpcContext>() {
        @Override
        protected RpcContext initialValue() {
            return new RpcContext();
        }
    };
    /**
     * 服务上下文
     */
    private static final InternalThreadLocal<RpcContext> SERVER_LOCAL = new InternalThreadLocal<RpcContext>() {
        @Override
        protected RpcContext initialValue() {
            return new RpcContext();
        }
    };

    /**
     * 附加值集合
     */
    private final Map<String, String> attachments = new HashMap<String, String>();
    /**
     * 上下文值
     */
    private final Map<String, Object> values = new HashMap<String, Object>();
    /**
     * 线程结果
     */
    private Future<?> future;

    /**
     * url集合
     */
    private List<URL> urls;

    /**
     * 当前的url
     */
    private URL url;

    /**
     * 方法名称
     */
    private String methodName;

    /**
     * 参数类型集合
     */
    private Class<?>[] parameterTypes;

    /**
     * 参数集合
     */
    private Object[] arguments;

    /**
     * 本地地址
     */
    private InetSocketAddress localAddress;

    /**
     * 远程地址
     */
    private InetSocketAddress remoteAddress;
    /**
     * 实体域集合
     */
    @Deprecated
    private List<Invoker<?>> invokers;
    /**
     * 实体域
     */
    @Deprecated
    private Invoker<?> invoker;
    /**
     * 会话域
     */
    @Deprecated
    private Invocation invocation;

    // now we don't use the 'values' map to hold these objects
    // we want these objects to be as generic as possible
    /**
     * 请求
     */
    private Object request;
    /**
     * 响应
     */
    private Object response;
```

该类中最重要的是它的一些属性，因为该上下文就是用来保存信息的。方法我就不介绍了，因为比较简单。

### （十一）RpcException

```
/**
 * 不知道异常
 */
public static final int UNKNOWN_EXCEPTION = 0;
/**
 * 网络异常
 */
public static final int NETWORK_EXCEPTION = 1;
/**
 * 超时异常
 */
public static final int TIMEOUT_EXCEPTION = 2;
/**
 * 基础异常
 */
public static final int BIZ_EXCEPTION = 3;
/**
 * 禁止访问异常
 */
public static final int FORBIDDEN_EXCEPTION = 4;
/**
 * 序列化异常
 */
public static final int SERIALIZATION_EXCEPTION = 5;
```

该类是rpc调用抛出的异常类，其中封装了五种通用的错误码。

### （十二）RpcInvocation

```
/**
 * 方法名称
 */
private String methodName;

/**
 * 参数类型集合
 */
private Class<?>[] parameterTypes;

/**
 * 参数集合
 */
private Object[] arguments;

/**
 * 附加值
 */
private Map<String, String> attachments;

/**
 * 实体域
 */
private transient Invoker<?> invoker;
```

该类实现了Invocation接口，是rpc的会话域，其中的方法比较简单，主要是封装了上述的属性。

### （十三）RpcResult

```
/**
 * 结果
 */
private Object result;

/**
 * 异常
 */
private Throwable exception;

/**
 * 附加值
 */
private Map<String, String> attachments = new HashMap<String, String>();
```

该类实现了Result接口，是rpc的结果实现类，其中关键是封装了以上三个属性。

### （十四）RpcStatus

该类是rpc的一些状态监控，其中封装了许多的计数器，用来记录rpc调用的状态。

#### 1.属性

```
/**
 * uri对应的状态集合，key为uri，value为RpcStatus对象
 */
private static final ConcurrentMap<String, RpcStatus> SERVICE_STATISTICS = new ConcurrentHashMap<String, RpcStatus>();

/**
 * method对应的状态集合，key是uri，第二个key是方法名methodName
 */
private static final ConcurrentMap<String, ConcurrentMap<String, RpcStatus>> METHOD_STATISTICS = new ConcurrentHashMap<String, ConcurrentMap<String, RpcStatus>>();
/**
 * 已经没用了
 */
private final ConcurrentMap<String, Object> values = new ConcurrentHashMap<String, Object>();
/**
 * 活跃状态
 */
private final AtomicInteger active = new AtomicInteger();
/**
 * 总的数量
 */
private final AtomicLong total = new AtomicLong();
/**
 * 失败的个数
 */
private final AtomicInteger failed = new AtomicInteger();
/**
 * 总调用时长
 */
private final AtomicLong totalElapsed = new AtomicLong();
/**
 * 总调用失败时长
 */
private final AtomicLong failedElapsed = new AtomicLong();
/**
 * 最大调用时长
 */
private final AtomicLong maxElapsed = new AtomicLong();
/**
 * 最大调用失败时长
 */
private final AtomicLong failedMaxElapsed = new AtomicLong();
/**
 * 最大调用成功时长
 */
private final AtomicLong succeededMaxElapsed = new AtomicLong();

/**
 * Semaphore used to control concurrency limit set by `executes`
 * 信号量用来控制`execution`设置的并发限制
 */
private volatile Semaphore executesLimit;
/**
 * 用来控制`execution`设置的许可证
 */
private volatile int executesPermits;
```

以上是该类的属性，可以看到保存了很多的计数器，分别用来记录了失败调用成功调用等累计数。

#### 2.beginCount

```
/**
 * 开始计数
 * @param url
 */
public static void beginCount(URL url, String methodName) {
    // 对该url对应对活跃计数器加一
    beginCount(getStatus(url));
    // 对该方法对活跃计数器加一
    beginCount(getStatus(url, methodName));
}

/**
 * 以原子方式加1
 * @param status
 */
private static void beginCount(RpcStatus status) {
    status.active.incrementAndGet();
}
```

该方法是增加计数。

#### 3.endCount

```
public static void endCount(URL url, String methodName, long elapsed, boolean succeeded) {
    // url对应的状态中计数器减一
    endCount(getStatus(url), elapsed, succeeded);
    // 方法对应的状态中计数器减一
    endCount(getStatus(url, methodName), elapsed, succeeded);
}

private static void endCount(RpcStatus status, long elapsed, boolean succeeded) {
    // 活跃计数器减一
    status.active.decrementAndGet();
    // 总计数器加1
    status.total.incrementAndGet();
    // 总调用时长加上调用时长
    status.totalElapsed.addAndGet(elapsed);
    // 如果最大调用时长小于elapsed，则设置最大调用时长
    if (status.maxElapsed.get() < elapsed) {
        status.maxElapsed.set(elapsed);
    }
    // 如果rpc调用成功
    if (succeeded) {
        // 如果成最大调用成功时长小于elapsed，则设置最大调用成功时长
        if (status.succeededMaxElapsed.get() < elapsed) {
            status.succeededMaxElapsed.set(elapsed);
        }
    } else {
        // 失败计数器加一
        status.failed.incrementAndGet();
        // 失败的过期数加上elapsed
        status.failedElapsed.addAndGet(elapsed);
        // 总调用失败时长小于elapsed，则设置总调用失败时长
        if (status.failedMaxElapsed.get() < elapsed) {
            status.failedMaxElapsed.set(elapsed);
        }
    }
}
```

该方法是计数器减少。

### （十五）StaticContext

该类是系统上下文，仅供内部使用。

```
/**
 * 系统名称
 */
private static final String SYSTEMNAME = "system";
/**
 * 系统上下文集合，仅供内部使用
 */
private static final ConcurrentMap<String, StaticContext> context_map = new ConcurrentHashMap<String, StaticContext>();
/**
 * 系统上下文名称
 */
private String name;
```

上面是该类的属性，它还记录了所有的系统上下文集合。

### （十六）EchoService

```
public interface EchoService {

    /**
     * echo test.
     * 回声测试
     * @param message message.
     * @return message.
     */
    Object $echo(Object message);

}
```

该接口是回声服务接口，定义了一个一个回声测试的方法，回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调用是否通畅，可用于监控，所有服务自动实现该接口，只需将任意服务强制转化为EchoService，就可以用了。

### （十七）GenericException

该方法是通用的异常类。

```
/**
 * 异常类名
 */
private String exceptionClass;

/**
 * 异常信息
 */
private String exceptionMessage;
```

比较简单，就封装了两个属性。

### （十八）GenericService

```
public interface GenericService {

    /**
     * Generic invocation
     * 通用的会话域
     * @param method         Method name, e.g. findPerson. If there are overridden methods, parameter info is
     *                       required, e.g. findPerson(java.lang.String)
     * @param parameterTypes Parameter types
     * @param args           Arguments
     * @return invocation return value
     * @throws Throwable potential exception thrown from the invocation
     */
    Object $invoke(String method, String[] parameterTypes, Object[] args) throws GenericException;

}
```

该接口是通用的服务接口，同样定义了一个类似invoke的方法

# 远程调用——Filter

## 源码分析

### （一）AccessLogFilter

该过滤器是对记录日志的过滤器，它所做的工作就是把引用服务或者暴露服务的调用链信息写入到文件中。日志消息先被放入日志集合，然后加入到日志队列，然后被放入到写入文件到任务中，最后进入文件。

#### 1.属性

```
private static final Logger logger = LoggerFactory.getLogger(AccessLogFilter.class);

/**
 * 日志访问名称，默认的日志访问名称
 */
private static final String ACCESS_LOG_KEY = "dubbo.accesslog";

/**
 * 日期格式
 */
private static final String FILE_DATE_FORMAT = "yyyyMMdd";

private static final String MESSAGE_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";

/**
 * 日志队列大小
 */
private static final int LOG_MAX_BUFFER = 5000;

/**
 * 日志输出的频率
 */
private static final long LOG_OUTPUT_INTERVAL = 5000;

/**
 * 日志队列 key为访问日志的名称，value为该日志名称对应的日志集合
 */
private final ConcurrentMap<String, Set<String>> logQueue = new ConcurrentHashMap<String, Set<String>>();

/**
 * 日志线程池
 */
private final ScheduledExecutorService logScheduled = Executors.newScheduledThreadPool(2, new NamedThreadFactory("Dubbo-Access-Log", true));

/**
 * 日志记录任务
 */
private volatile ScheduledFuture<?> logFuture = null;
```

按照我上面讲到日志流向，日志先进入到是日志队列中的日志集合，再进入logQueue，在进入logFuture，最后落地到文件。

#### 2.init

```
private void init() {
    // synchronized是一个重操作消耗性能，所有加上判空
    if (logFuture == null) {
        synchronized (logScheduled) {
            // 为了不重复初始化
            if (logFuture == null) {
                // 创建日志记录任务
                logFuture = logScheduled.scheduleWithFixedDelay(new LogTask(), LOG_OUTPUT_INTERVAL, LOG_OUTPUT_INTERVAL, TimeUnit.MILLISECONDS);
            }
        }
    }
}
```

该方法是初始化方法，就创建了日志记录任务。

#### 3.log

```
private void log(String accesslog, String logmessage) {
    init();
    Set<String> logSet = logQueue.get(accesslog);
    if (logSet == null) {
        logQueue.putIfAbsent(accesslog, new ConcurrentHashSet<String>());
        logSet = logQueue.get(accesslog);
    }
    if (logSet.size() < LOG_MAX_BUFFER) {
        logSet.add(logmessage);
    }
}
```

该方法是增加日志信息到日志集合中。

#### 4.invoke

```
@Override
public Result invoke(Invoker<?> invoker, Invocation inv) throws RpcException {
    try {
        // 获得日志名称
        String accesslog = invoker.getUrl().getParameter(Constants.ACCESS_LOG_KEY);
        if (ConfigUtils.isNotEmpty(accesslog)) {
            // 获得rpc上下文
            RpcContext context = RpcContext.getContext();
            // 获得调用的接口名称
            String serviceName = invoker.getInterface().getName();
            // 获得版本号
            String version = invoker.getUrl().getParameter(Constants.VERSION_KEY);
            // 获得组，是消费者侧还是生产者侧
            String group = invoker.getUrl().getParameter(Constants.GROUP_KEY);
            StringBuilder sn = new StringBuilder();
            sn.append("[").append(new SimpleDateFormat(MESSAGE_DATE_FORMAT).format(new Date())).append("] ").append(context.getRemoteHost()).append(":").append(context.getRemotePort())
                    .append(" -> ").append(context.getLocalHost()).append(":").append(context.getLocalPort())
                    .append(" - ");
            // 拼接组
            if (null != group && group.length() > 0) {
                sn.append(group).append("/");
            }
            // 拼接服务名称
            sn.append(serviceName);
            // 拼接版本号
            if (null != version && version.length() > 0) {
                sn.append(":").append(version);
            }
            sn.append(" ");
            // 拼接方法名
            sn.append(inv.getMethodName());
            sn.append("(");
            // 拼接参数类型
            Class<?>[] types = inv.getParameterTypes();
            // 拼接参数类型
            if (types != null && types.length > 0) {
                boolean first = true;
                for (Class<?> type : types) {
                    if (first) {
                        first = false;
                    } else {
                        sn.append(",");
                    }
                    sn.append(type.getName());
                }
            }
            sn.append(") ");
            // 拼接参数
            Object[] args = inv.getArguments();
            if (args != null && args.length > 0) {
                sn.append(JSON.toJSONString(args));
            }
            String msg = sn.toString();
            // 如果用默认的日志访问名称
            if (ConfigUtils.isDefault(accesslog)) {
                LoggerFactory.getLogger(ACCESS_LOG_KEY + "." + invoker.getInterface().getName()).info(msg);
            } else {
                // 把日志加入集合
                log(accesslog, msg);
            }
        }
    } catch (Throwable t) {
        logger.warn("Exception in AcessLogFilter of service(" + invoker + " -> " + inv + ")", t);
    }
    // 调用下一个调用链
    return invoker.invoke(inv);
}
```

该方法是最重要的方法，从拼接了日志信息，把日志加入到集合，并且调用下一个调用链。

#### 4.LogTask

```
private class LogTask implements Runnable {
    @Override
    public void run() {
        try {
            if (logQueue != null && logQueue.size() > 0) {
                // 遍历日志队列
                for (Map.Entry<String, Set<String>> entry : logQueue.entrySet()) {
                    try {
                        // 获得日志名称
                        String accesslog = entry.getKey();
                        // 获得日志集合
                        Set<String> logSet = entry.getValue();
                        // 如果文件不存在则创建文件
                        File file = new File(accesslog);
                        File dir = file.getParentFile();
                        if (null != dir && !dir.exists()) {
                            dir.mkdirs();
                        }
                        if (logger.isDebugEnabled()) {
                            logger.debug("Append log to " + accesslog);
                        }
                        if (file.exists()) {
                            // 获得现在的时间
                            String now = new SimpleDateFormat(FILE_DATE_FORMAT).format(new Date());
                            // 获得文件最后一次修改的时间
                            String last = new SimpleDateFormat(FILE_DATE_FORMAT).format(new Date(file.lastModified()));
                            // 如果文件最后一次修改的时间不等于现在的时间
                            if (!now.equals(last)) {
                                // 获得重新生成文件名称
                                File archive = new File(file.getAbsolutePath() + "." + last);
                                // 因为都是file的绝对路径，所以没有进行移动文件，而是修改文件名
                                file.renameTo(archive);
                            }
                        }
                        // 把日志集合中的日志写入到文件
                        FileWriter writer = new FileWriter(file, true);
                        try {
                            for (Iterator<String> iterator = logSet.iterator();
                                 iterator.hasNext();
                                 iterator.remove()) {
                                writer.write(iterator.next());
                                writer.write("\r\n");
                            }
                            writer.flush();
                        } finally {
                            writer.close();
                        }
                    } catch (Exception e) {
                        logger.error(e.getMessage(), e);
                    }
                }
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
    }
}
```

该内部类实现了Runnable，是把日志消息落地到文件到线程。

### （二）ActiveLimitFilter

该类时对于每个服务的每个方法的最大可并行调用数量限制的过滤器，它是在服务消费者侧的过滤。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获得url对象
    URL url = invoker.getUrl();
    // 获得方法名称
    String methodName = invocation.getMethodName();
    // 获得并发调用数（单个服务的单个方法），默认为0
    int max = invoker.getUrl().getMethodParameter(methodName, Constants.ACTIVES_KEY, 0);
    // 通过方法名来获得对应的状态
    RpcStatus count = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName());
    if (max > 0) {
        // 获得该方法调用的超时次数
        long timeout = invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.TIMEOUT_KEY, 0);
        // 获得系统时间
        long start = System.currentTimeMillis();
        long remain = timeout;
        // 获得该方法的调用数量
        int active = count.getActive();
        // 如果活跃数量大于等于最大的并发调用数量
        if (active >= max) {
            synchronized (count) {
                // 当活跃数量大于等于最大的并发调用数量时一直循环
                while ((active = count.getActive()) >= max) {
                    try {
                        // 等待超时时间
                        count.wait(remain);
                    } catch (InterruptedException e) {
                    }
                    // 获得累计时间
                    long elapsed = System.currentTimeMillis() - start;
                    remain = timeout - elapsed;
                    // 如果累计时间大于超时时间，则抛出异常
                    if (remain <= 0) {
                        throw new RpcException("Waiting concurrent invoke timeout in client-side for service:  "
                                + invoker.getInterface().getName() + ", method: "
                                + invocation.getMethodName() + ", elapsed: " + elapsed
                                + ", timeout: " + timeout + ". concurrent invokes: " + active
                                + ". max concurrent invoke limit: " + max);
                    }
                }
            }
        }
    }
    try {
        // 获得系统时间作为开始时间
        long begin = System.currentTimeMillis();
        // 开始计数
        RpcStatus.beginCount(url, methodName);
        try {
            // 调用后面的调用链，如果没有抛出异常，则算成功
            Result result = invoker.invoke(invocation);
            // 结束计数，记录时间
            RpcStatus.endCount(url, methodName, System.currentTimeMillis() - begin, true);
            return result;
        } catch (RuntimeException t) {
            RpcStatus.endCount(url, methodName, System.currentTimeMillis() - begin, false);
            throw t;
        }
    } finally {
        if (max > 0) {
            synchronized (count) {
                // 唤醒count
                count.notify();
            }
        }
    }
}
```

该类只有这一个方法。该过滤器是用来限制调用数量，先进行调用数量的检测，如果没有到达最大的调用数量，则先调用后面的调用链，如果在后面的调用链失败，则记录相关时间，如果成功也记录相关时间和调用次数。

### （三）ClassLoaderFilter

该过滤器是做类加载器切换的。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获得当前的类加载器
    ClassLoader ocl = Thread.currentThread().getContextClassLoader();
    // 设置invoker携带的服务的类加载器
    Thread.currentThread().setContextClassLoader(invoker.getInterface().getClassLoader());
    try {
        // 调用下面的调用链
        return invoker.invoke(invocation);
    } finally {
        // 最后切换回原来的类加载器
        Thread.currentThread().setContextClassLoader(ocl);
    }
}
```

可以看到先切换成当前的线程锁携带的类加载器，然后调用结束后，再切换回原先的类加载器。

### （四）CompatibleFilter

该过滤器是做兼容性的过滤器。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 调用下一个调用链
    Result result = invoker.invoke(invocation);
    // 如果方法前面没有$或者结果没有异常
    if (!invocation.getMethodName().startsWith("$") && !result.hasException()) {
        Object value = result.getValue();
        if (value != null) {
            try {
                // 获得方法
                Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
                // 获得返回的数据类型
                Class<?> type = method.getReturnType();
                Object newValue;
                // 序列化方法
                String serialization = invoker.getUrl().getParameter(Constants.SERIALIZATION_KEY);
                // 如果是json或者fastjson形式
                if ("json".equals(serialization)
                        || "fastjson".equals(serialization)) {
                    // 获得方法的泛型返回值类型
                    Type gtype = method.getGenericReturnType();
                    // 把数据结果进行类型转化
                    newValue = PojoUtils.realize(value, type, gtype);
                    // 如果value不是type类型
                } else if (!type.isInstance(value)) {
                    // 如果是pojo，则，转化为type类型，如果不是，则进行兼容类型转化。
                    newValue = PojoUtils.isPojo(type)
                            ? PojoUtils.realize(value, type)
                            : CompatibleTypeUtils.compatibleTypeConvert(value, type);

                } else {
                    newValue = value;
                }
                // 重新设置RpcResult的result
                if (newValue != value) {
                    result = new RpcResult(newValue);
                }
            } catch (Throwable t) {
                logger.warn(t.getMessage(), t);
            }
        }
    }
    return result;
}
```

可以看到对于调用链的返回结果，如果返回值类型和返回值不一样的时候，就需要做兼容类型的转化。重新把结果放入RpcResult，返回。

### （五）ConsumerContextFilter

该过滤器做的是在当前的RpcContext中记录本地调用的一次状态信息。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 设置rpc上下文
    RpcContext.getContext()
            .setInvoker(invoker)
            .setInvocation(invocation)
            .setLocalAddress(NetUtils.getLocalHost(), 0)
            .setRemoteAddress(invoker.getUrl().getHost(),
                    invoker.getUrl().getPort());
    // 如果该会话域是rpc会话域
    if (invocation instanceof RpcInvocation) {
        // 设置实体域
        ((RpcInvocation) invocation).setInvoker(invoker);
    }
    try {
        // 调用下个调用链
        RpcResult result = (RpcResult) invoker.invoke(invocation);
        // 设置附加值
        RpcContext.getServerContext().setAttachments(result.getAttachments());
        return result;
    } finally {
        // 情况附加值
        RpcContext.getContext().clearAttachments();
    }
}
```

可以看到RpcContext记录了一次调用状态信息，然后先调用后面的调用链，再回来把附加值设置到RpcContext中。然后返回RpcContext，再清空，这样是因为后面的调用链中的附加值对前面的调用链是不可见的。

### （六）ContextFilter

该过滤器做的是初始化rpc上下文。

```
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 获得会话域的附加值
        Map<String, String> attachments = invocation.getAttachments();
        // 删除异步属性以避免传递给以下调用链
        if (attachments != null) {
            attachments = new HashMap<String, String>(attachments);
            attachments.remove(Constants.PATH_KEY);
            attachments.remove(Constants.GROUP_KEY);
            attachments.remove(Constants.VERSION_KEY);
            attachments.remove(Constants.DUBBO_VERSION_KEY);
            attachments.remove(Constants.TOKEN_KEY);
            attachments.remove(Constants.TIMEOUT_KEY);
            attachments.remove(Constants.ASYNC_KEY);// Remove async property to avoid being passed to the following invoke chain.
        }
        // 在rpc上下文添加上一个调用链的信息
        RpcContext.getContext()
                .setInvoker(invoker)
                .setInvocation(invocation)
//                .setAttachments(attachments)  // merged from dubbox
                .setLocalAddress(invoker.getUrl().getHost(),
                        invoker.getUrl().getPort());

        // mreged from dubbox
        // we may already added some attachments into RpcContext before this filter (e.g. in rest protocol)
        if (attachments != null) {
            // 把会话域中的附加值全部加入RpcContext中
            if (RpcContext.getContext().getAttachments() != null) {
                RpcContext.getContext().getAttachments().putAll(attachments);
            } else {
                RpcContext.getContext().setAttachments(attachments);
            }
        }

        // 如果会话域属于rpc的会话域，则设置实体域
        if (invocation instanceof RpcInvocation) {
            ((RpcInvocation) invocation).setInvoker(invoker);
        }
        try {
            // 调用下一个调用链
            RpcResult result = (RpcResult) invoker.invoke(invocation);
            // pass attachments to result 把附加值加入到RpcResult
            result.addAttachments(RpcContext.getServerContext().getAttachments());
            return result;
        } finally {
            // 移除本地的上下文
            RpcContext.removeContext();
            // 清空附加值
            RpcContext.getServerContext().clearAttachments();
        }
    }
```

在[《 dubbo源码解析（十九）远程调用——开篇》](https://segmentfault.com/a/1190000017787521)中我已经介绍了RpcContext的作用，角色。该过滤器就是做了初始化RpcContext的作用。

### （七）DeprecatedFilter

该过滤器的作用是调用了废弃的方法时打印错误日志。

```
private static final Logger LOGGER = LoggerFactory.getLogger(DeprecatedFilter.class);

/**
 * 日志集合
 */
private static final Set<String> logged = new ConcurrentHashSet<String>();

@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获得key 服务+方法
    String key = invoker.getInterface().getName() + "." + invocation.getMethodName();
    // 如果集合中没有该key
    if (!logged.contains(key)) {
        // 则加入集合
        logged.add(key);
        // 如果该服务方法是废弃的，则打印错误日志
        if (invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.DEPRECATED_KEY, false)) {
            LOGGER.error("The service method " + invoker.getInterface().getName() + "." + getMethodSignature(invocation) + " is DEPRECATED! Declare from " + invoker.getUrl());
        }
    }
    // 调用下一个调用链
    return invoker.invoke(invocation);
}

/**
 * 获得方法定义
 * @param invocation
 * @return
 */
private String getMethodSignature(Invocation invocation) {
    // 方法名
    StringBuilder buf = new StringBuilder(invocation.getMethodName());
    buf.append("(");
    // 参数类型
    Class<?>[] types = invocation.getParameterTypes();
    // 拼接参数
    if (types != null && types.length > 0) {
        boolean first = true;
        for (Class<?> type : types) {
            if (first) {
                first = false;
            } else {
                buf.append(", ");
            }
            buf.append(type.getSimpleName());
        }
    }
    buf.append(")");
    return buf.toString();
}
```

该过滤器比较简单。

### （八）EchoFilter

该过滤器是处理回声测试的方法。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation inv) throws RpcException {
    // 如果调用的方法是回声测试的方法 则直接返回结果，否则 调用下一个调用链
    if (inv.getMethodName().equals(Constants.$ECHO) && inv.getArguments() != null && inv.getArguments().length == 1)
        return new RpcResult(inv.getArguments()[0]);
    return invoker.invoke(inv);
}
```

如果调用的方法是回声测试的方法 则直接返回结果，否则 调用下一个调用链。

### （九）ExceptionFilter

该过滤器是作用是对异常的处理。

```
private final Logger logger;

public ExceptionFilter() {
    this(LoggerFactory.getLogger(ExceptionFilter.class));
}

public ExceptionFilter(Logger logger) {
    this.logger = logger;
}

@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    try {
        // 调用下一个调用链，返回结果
        Result result = invoker.invoke(invocation);
        // 如果结果有异常，并且该服务不是一个泛化调用
        if (result.hasException() && GenericService.class != invoker.getInterface()) {
            try {
                // 获得异常
                Throwable exception = result.getException();

                // directly throw if it's checked exception
                // 如果这是一个checked的异常，则直接返回异常，也就是接口上声明的Unchecked的异常
                if (!(exception instanceof RuntimeException) && (exception instanceof Exception)) {
                    return result;
                }
                // directly throw if the exception appears in the signature
                // 如果已经在接口方法上声明了该异常，则直接返回
                try {
                    // 获得方法
                    Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
                    // 获得异常类型
                    Class<?>[] exceptionClassses = method.getExceptionTypes();
                    for (Class<?> exceptionClass : exceptionClassses) {
                        if (exception.getClass().equals(exceptionClass)) {
                            return result;
                        }
                    }
                } catch (NoSuchMethodException e) {
                    return result;
                }

                // for the exception not found in method's signature, print ERROR message in server's log.
                // 打印错误 该异常没有在方法上申明
                logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                        + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                        + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

                // directly throw if exception class and interface class are in the same jar file.
                // 如果异常类和接口类在同一个jar包里面，则抛出异常
                String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());
                String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());
                if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)) {
                    return result;
                }
                // directly throw if it's JDK exception
                // 如果是jdk中定义的异常，则直接抛出
                String className = exception.getClass().getName();
                if (className.startsWith("java.") || className.startsWith("javax.")) {
                    return result;
                }
                // directly throw if it's dubbo exception
                // 如果 是dubbo的异常，则直接抛出
                if (exception instanceof RpcException) {
                    return result;
                }

                // otherwise, wrap with RuntimeException and throw back to the client
                // 如果不是以上的异常，则包装成为RuntimeException并且抛出
                return new RpcResult(new RuntimeException(StringUtils.toString(exception)));
            } catch (Throwable e) {
                logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost()
                        + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                        + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
                return result;
            }
        }
        return result;
    } catch (RuntimeException e) {
        logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
        throw e;
    }
}
```

可以看到除了接口上声明的Unchecked的异常和有定义的异常外，都会包装成RuntimeException来返回，为了防止客户端反序列化失败。

### （十）ExecuteLimitFilter

该过滤器是限制最大可并行执行请求数，该过滤器是服务提供者侧，而上述讲到的ActiveLimitFilter是在消费者侧的限制。

```
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 获得url对象
        URL url = invoker.getUrl();
        // 方法名称
        String methodName = invocation.getMethodName();
        Semaphore executesLimit = null;
        boolean acquireResult = false;
        int max = url.getMethodParameter(methodName, Constants.EXECUTES_KEY, 0);
        // 如果该方法设置了executes并且值大于0
        if (max > 0) {
            // 获得该方法对应的RpcStatus
            RpcStatus count = RpcStatus.getStatus(url, invocation.getMethodName());
//            if (count.getActive() >= max) {
            /**
             * http://manzhizhen.iteye.com/blog/2386408
             * use semaphore for concurrency control (to limit thread number)
             */
            // 获得信号量
            executesLimit = count.getSemaphore(max);
            // 如果不能获得许可，则抛出异常
            if(executesLimit != null && !(acquireResult = executesLimit.tryAcquire())) {
                throw new RpcException("Failed to invoke method " + invocation.getMethodName() + " in provider " + url + ", cause: The service using threads greater than <dubbo:service executes=\"" + max + "\" /> limited.");
            }
        }
        long begin = System.currentTimeMillis();
        boolean isSuccess = true;
        // 计数加1
        RpcStatus.beginCount(url, methodName);
        try {
            // 调用下一个调用链
            Result result = invoker.invoke(invocation);
            return result;
        } catch (Throwable t) {
            isSuccess = false;
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new RpcException("unexpected exception when ExecuteLimitFilter", t);
            }
        } finally {
            // 计数减1
            RpcStatus.endCount(url, methodName, System.currentTimeMillis() - begin, isSuccess);
            if(acquireResult) {
                executesLimit.release();
            }
        }
    }
```

为什么这里需要用到信号量来控制，可以看一下以下链接的介绍：[http://manzhizhen.iteye.com/b...](http://manzhizhen.iteye.com/blog/2386408)

### （十一）GenericFilter

该过滤器就是对于泛化调用的请求和结果进行反序列化和序列化的操作，它是服务提供者侧的。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation inv) throws RpcException {
    // 如果是泛化调用
    if (inv.getMethodName().equals(Constants.$INVOKE)
            && inv.getArguments() != null
            && inv.getArguments().length == 3
            && !invoker.getInterface().equals(GenericService.class)) {
        // 获得请求名字
        String name = ((String) inv.getArguments()[0]).trim();
        // 获得请求参数类型
        String[] types = (String[]) inv.getArguments()[1];
        // 获得请求参数
        Object[] args = (Object[]) inv.getArguments()[2];
        try {
            // 获得方法
            Method method = ReflectUtils.findMethodByMethodSignature(invoker.getInterface(), name, types);
            // 获得该方法的参数类型
            Class<?>[] params = method.getParameterTypes();
            if (args == null) {
                args = new Object[params.length];
            }
            // 获得附加值
            String generic = inv.getAttachment(Constants.GENERIC_KEY);

            // 如果附加值为空，在用上下文携带的附加值
            if (StringUtils.isBlank(generic)) {
                generic = RpcContext.getContext().getAttachment(Constants.GENERIC_KEY);
            }

            // 如果附加值还是为空或者是默认的泛化序列化类型
            if (StringUtils.isEmpty(generic)
                    || ProtocolUtils.isDefaultGenericSerialization(generic)) {
                // 直接进行类型转化
                args = PojoUtils.realize(args, params, method.getGenericParameterTypes());
            } else if (ProtocolUtils.isJavaGenericSerialization(generic)) {
                for (int i = 0; i < args.length; i++) {
                    if (byte[].class == args[i].getClass()) {
                        try {
                            UnsafeByteArrayInputStream is = new UnsafeByteArrayInputStream((byte[]) args[i]);
                            // 使用nativejava方式反序列化
                            args[i] = ExtensionLoader.getExtensionLoader(Serialization.class)
                                    .getExtension(Constants.GENERIC_SERIALIZATION_NATIVE_JAVA)
                                    .deserialize(null, is).readObject();
                        } catch (Exception e) {
                            throw new RpcException("Deserialize argument [" + (i + 1) + "] failed.", e);
                        }
                    } else {
                        throw new RpcException(
                                "Generic serialization [" +
                                        Constants.GENERIC_SERIALIZATION_NATIVE_JAVA +
                                        "] only support message type " +
                                        byte[].class +
                                        " and your message type is " +
                                        args[i].getClass());
                    }
                }
            } else if (ProtocolUtils.isBeanGenericSerialization(generic)) {
                for (int i = 0; i < args.length; i++) {
                    if (args[i] instanceof JavaBeanDescriptor) {
                        // 用JavaBean方式反序列化
                        args[i] = JavaBeanSerializeUtil.deserialize((JavaBeanDescriptor) args[i]);
                    } else {
                        throw new RpcException(
                                "Generic serialization [" +
                                        Constants.GENERIC_SERIALIZATION_BEAN +
                                        "] only support message type " +
                                        JavaBeanDescriptor.class.getName() +
                                        " and your message type is " +
                                        args[i].getClass().getName());
                    }
                }
            }
            // 调用下一个调用链
            Result result = invoker.invoke(new RpcInvocation(method, args, inv.getAttachments()));
            if (result.hasException()
                    && !(result.getException() instanceof GenericException)) {
                return new RpcResult(new GenericException(result.getException()));
            }
            if (ProtocolUtils.isJavaGenericSerialization(generic)) {
                try {
                    UnsafeByteArrayOutputStream os = new UnsafeByteArrayOutputStream(512);
                    // 用nativejava方式序列化
                    ExtensionLoader.getExtensionLoader(Serialization.class)
                            .getExtension(Constants.GENERIC_SERIALIZATION_NATIVE_JAVA)
                            .serialize(null, os).writeObject(result.getValue());
                    return new RpcResult(os.toByteArray());
                } catch (IOException e) {
                    throw new RpcException("Serialize result failed.", e);
                }
            } else if (ProtocolUtils.isBeanGenericSerialization(generic)) {
                // 使用JavaBean方式序列化返回结果
                return new RpcResult(JavaBeanSerializeUtil.serialize(result.getValue(), JavaBeanAccessor.METHOD));
            } else {
                // 直接转化为pojo类型然后返回
                return new RpcResult(PojoUtils.generalize(result.getValue()));
            }
        } catch (NoSuchMethodException e) {
            throw new RpcException(e.getMessage(), e);
        } catch (ClassNotFoundException e) {
            throw new RpcException(e.getMessage(), e);
        }
    }
    // 调用下一个调用链
    return invoker.invoke(inv);
}
```

### （十二）GenericImplFilter

该过滤器也是对于泛化调用的序列化检查和处理，它是消费者侧的过滤器。

```
private static final Logger logger = LoggerFactory.getLogger(GenericImplFilter.class);

/**
 * 参数集合
 */
private static final Class<?>[] GENERIC_PARAMETER_TYPES = new Class<?>[]{String.class, String[].class, Object[].class};

@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获得泛化的值
    String generic = invoker.getUrl().getParameter(Constants.GENERIC_KEY);
    // 如果该值是nativejava或者bean或者true，并且不是一个返回调用
    if (ProtocolUtils.isGeneric(generic)
            && !Constants.$INVOKE.equals(invocation.getMethodName())
            && invocation instanceof RpcInvocation) {
        RpcInvocation invocation2 = (RpcInvocation) invocation;
        // 获得方法名称
        String methodName = invocation2.getMethodName();
        // 获得参数类型集合
        Class<?>[] parameterTypes = invocation2.getParameterTypes();
        // 获得参数集合
        Object[] arguments = invocation2.getArguments();

        // 把参数类型的名称放入集合
        String[] types = new String[parameterTypes.length];
        for (int i = 0; i < parameterTypes.length; i++) {
            types[i] = ReflectUtils.getName(parameterTypes[i]);
        }

        Object[] args;
        // 对参数集合进行序列化
        if (ProtocolUtils.isBeanGenericSerialization(generic)) {
            args = new Object[arguments.length];
            for (int i = 0; i < arguments.length; i++) {
                args[i] = JavaBeanSerializeUtil.serialize(arguments[i], JavaBeanAccessor.METHOD);
            }
        } else {
            args = PojoUtils.generalize(arguments);
        }

        // 重新把序列化的参数放入
        invocation2.setMethodName(Constants.$INVOKE);
        invocation2.setParameterTypes(GENERIC_PARAMETER_TYPES);
        invocation2.setArguments(new Object[]{methodName, types, args});
        // 调用下一个调用链
        Result result = invoker.invoke(invocation2);

        if (!result.hasException()) {
            Object value = result.getValue();
            try {
                Method method = invoker.getInterface().getMethod(methodName, parameterTypes);
                if (ProtocolUtils.isBeanGenericSerialization(generic)) {
                    if (value == null) {
                        return new RpcResult(value);
                    } else if (value instanceof JavaBeanDescriptor) {
                        // 用javabean方式反序列化
                        return new RpcResult(JavaBeanSerializeUtil.deserialize((JavaBeanDescriptor) value));
                    } else {
                        throw new RpcException(
                                "The type of result value is " +
                                        value.getClass().getName() +
                                        " other than " +
                                        JavaBeanDescriptor.class.getName() +
                                        ", and the result is " +
                                        value);
                    }
                } else {
                    // 直接转化为pojo类型
                    return new RpcResult(PojoUtils.realize(value, method.getReturnType(), method.getGenericReturnType()));
                }
            } catch (NoSuchMethodException e) {
                throw new RpcException(e.getMessage(), e);
            }
            // 如果调用链中有异常抛出，并且是GenericException类型的异常
        } else if (result.getException() instanceof GenericException) {
            GenericException exception = (GenericException) result.getException();
            try {
                // 获得异常类名
                String className = exception.getExceptionClass();
                Class<?> clazz = ReflectUtils.forName(className);
                Throwable targetException = null;
                Throwable lastException = null;
                try {
                    targetException = (Throwable) clazz.newInstance();
                } catch (Throwable e) {
                    lastException = e;
                    for (Constructor<?> constructor : clazz.getConstructors()) {
                        try {
                            targetException = (Throwable) constructor.newInstance(new Object[constructor.getParameterTypes().length]);
                            break;
                        } catch (Throwable e1) {
                            lastException = e1;
                        }
                    }
                }
                if (targetException != null) {
                    try {
                        Field field = Throwable.class.getDeclaredField("detailMessage");
                        if (!field.isAccessible()) {
                            field.setAccessible(true);
                        }
                        field.set(targetException, exception.getExceptionMessage());
                    } catch (Throwable e) {
                        logger.warn(e.getMessage(), e);
                    }
                    result = new RpcResult(targetException);
                } else if (lastException != null) {
                    throw lastException;
                }
            } catch (Throwable e) {
                throw new RpcException("Can not deserialize exception " + exception.getExceptionClass() + ", message: " + exception.getExceptionMessage(), e);
            }
        }
        return result;
    }

    // 如果是泛化调用
    if (invocation.getMethodName().equals(Constants.$INVOKE)
            && invocation.getArguments() != null
            && invocation.getArguments().length == 3
            && ProtocolUtils.isGeneric(generic)) {

        Object[] args = (Object[]) invocation.getArguments()[2];
        if (ProtocolUtils.isJavaGenericSerialization(generic)) {

            for (Object arg : args) {
                // 如果调用消息不是字节数组类型，则抛出异常
                if (!(byte[].class == arg.getClass())) {
                    error(generic, byte[].class.getName(), arg.getClass().getName());
                }
            }
        } else if (ProtocolUtils.isBeanGenericSerialization(generic)) {
            for (Object arg : args) {
                if (!(arg instanceof JavaBeanDescriptor)) {
                    error(generic, JavaBeanDescriptor.class.getName(), arg.getClass().getName());
                }
            }
        }

        // 设置附加值
        ((RpcInvocation) invocation).setAttachment(
                Constants.GENERIC_KEY, invoker.getUrl().getParameter(Constants.GENERIC_KEY));
    }
    return invoker.invoke(invocation);
}

/**
 * 抛出错误异常
 * @param generic
 * @param expected
 * @param actual
 * @throws RpcException
 */
private void error(String generic, String expected, String actual) throws RpcException {
    throw new RpcException(
            "Generic serialization [" +
                    generic +
                    "] only support message type " +
                    expected +
                    " and your message type is " +
                    actual);
}
```

### （十三）TimeoutFilter

该过滤器是当服务调用超时的时候，记录告警日志。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获得开始时间
    long start = System.currentTimeMillis();
    // 调用下一个调用链
    Result result = invoker.invoke(invocation);
    // 获得调用使用的时间
    long elapsed = System.currentTimeMillis() - start;
    // 如果服务调用超时，则打印告警日志
    if (invoker.getUrl() != null
            && elapsed > invoker.getUrl().getMethodParameter(invocation.getMethodName(),
            "timeout", Integer.MAX_VALUE)) {
        if (logger.isWarnEnabled()) {
            logger.warn("invoke time out. method: " + invocation.getMethodName()
                    + " arguments: " + Arrays.toString(invocation.getArguments()) + " , url is "
                    + invoker.getUrl() + ", invoke elapsed " + elapsed + " ms.");
        }
    }
    return result;
}
```

### （十四）TokenFilter

该过滤器提供了token的验证功能，关于token的介绍可以查看官方文档。

```
@Override
public Result invoke(Invoker<?> invoker, Invocation inv)
        throws RpcException {
    // 获得token值
    String token = invoker.getUrl().getParameter(Constants.TOKEN_KEY);
    if (ConfigUtils.isNotEmpty(token)) {
        // 获得服务类型
        Class<?> serviceType = invoker.getInterface();
        // 获得附加值
        Map<String, String> attachments = inv.getAttachments();
        String remoteToken = attachments == null ? null : attachments.get(Constants.TOKEN_KEY);
        // 如果令牌不一样，则抛出异常
        if (!token.equals(remoteToken)) {
            throw new RpcException("Invalid token! Forbid invoke remote service " + serviceType + " method " + inv.getMethodName() + "() from consumer " + RpcContext.getContext().getRemoteHost() + " to provider " + RpcContext.getContext().getLocalHost());
        }
    }
    // 调用下一个调用链
    return invoker.invoke(inv);
}
```

### （十五）TpsLimitFilter

该过滤器的作用是对TPS限流。

```
/**
 * TPS 限制器对象
 */
private final TPSLimiter tpsLimiter = new DefaultTPSLimiter();

@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {

    // 如果限流器不允许，则抛出异常
    if (!tpsLimiter.isAllowable(invoker.getUrl(), invocation)) {
        throw new RpcException(
                "Failed to invoke service " +
                        invoker.getInterface().getName() +
                        "." +
                        invocation.getMethodName() +
                        " because exceed max service tps.");
    }

    // 调用下一个调用链
    return invoker.invoke(invocation);
}
```

其中关键是TPS 限制器对象，请看下面的分析。

### （十六）TPSLimiter

```
public interface TPSLimiter {

    /**
     * judge if the current invocation is allowed by TPS rule
     * 是否允许通过
     * @param url        url
     * @param invocation invocation
     * @return true allow the current invocation, otherwise, return false
     */
    boolean isAllowable(URL url, Invocation invocation);

}
```

该接口是tps限流器的接口，只定义了一个是否允许通过的方法。

### （十七）StatItem

该类是统计的数据结构。

```
class StatItem {

    /**
     * 服务名
     */
    private String name;

    /**
     * 最后一次重置的时间
     */
    private long lastResetTime;

    /**
     * 周期
     */
    private long interval;

    /**
     * 剩余多少流量
     */
    private AtomicInteger token;

    /**
     * 限制大小
     */
    private int rate;

    StatItem(String name, int rate, long interval) {
        this.name = name;
        this.rate = rate;
        this.interval = interval;
        this.lastResetTime = System.currentTimeMillis();
        this.token = new AtomicInteger(rate);
    }

    public boolean isAllowable() {
        long now = System.currentTimeMillis();
        // 如果限制的时间大于最后一次时间加上周期，则重置
        if (now > lastResetTime + interval) {
            token.set(rate);
            lastResetTime = now;
        }

        int value = token.get();
        boolean flag = false;
        // 直到有流量
        while (value > 0 && !flag) {
            flag = token.compareAndSet(value, value - 1);
            value = token.get();
        }

        // 返回flag
        return flag;
    }

    long getLastResetTime() {
        return lastResetTime;
    }

    int getToken() {
        return token.get();
    }

    @Override
    public String toString() {
        return new StringBuilder(32).append("StatItem ")
                .append("[name=").append(name).append(", ")
                .append("rate = ").append(rate).append(", ")
                .append("interval = ").append(interval).append("]")
                .toString();
    }

}
```

可以看到该类中记录了一些访问的流量，并且设置了周期重置机制。

### （十八）DefaultTPSLimiter

该类实现了TPSLimiter，是默认的tps限流器实现。

```
public class DefaultTPSLimiter implements TPSLimiter {

    /**
     * 统计项集合
     */
    private final ConcurrentMap<String, StatItem> stats
            = new ConcurrentHashMap<String, StatItem>();

    @Override
    public boolean isAllowable(URL url, Invocation invocation) {
        // 获得tps限制大小，默认-1，不限制
        int rate = url.getParameter(Constants.TPS_LIMIT_RATE_KEY, -1);
        // 获得限流周期
        long interval = url.getParameter(Constants.TPS_LIMIT_INTERVAL_KEY,
                Constants.DEFAULT_TPS_LIMIT_INTERVAL);
        String serviceKey = url.getServiceKey();
        // 如果限制
        if (rate > 0) {
            // 从集合中获得统计项
            StatItem statItem = stats.get(serviceKey);
            // 如果为空，则新建
            if (statItem == null) {
                stats.putIfAbsent(serviceKey,
                        new StatItem(serviceKey, rate, interval));
                statItem = stats.get(serviceKey);
            }
            // 返回是否允许
            return statItem.isAllowable();
        } else {
            StatItem statItem = stats.get(serviceKey);
            if (statItem != null) {
                // 移除该服务的统计项
                stats.remove(serviceKey);
            }
        }

        return true;
    }

}
```

是否允许的逻辑还是调用了统计项中的isAllowable方法。

本文介绍了很多的过滤器，哪些过滤器是在服务引用的，哪些服务器是服务暴露的，可以查看相应源码过滤器的实现上的注解，

例如ActiveLimitFilter上：

```
@Activate(group = Constants.CONSUMER, value = Constants.ACTIVES_KEY)
```

可以看到group为consumer组的，也就是服务消费者侧的，则是服务引用过程中的的过滤器。

例如ExecuteLimitFilter上：

```
@Activate(group = Constants.PROVIDER, value = Constants.EXECUTES_KEY)
```

可以看到group为provider组的，也就是服务提供者侧的，则是服务暴露过程中的的过滤器。

# 远程调用——Listener

> 目标：介绍dubbo-rpc-api中的各种listener监听器的实现逻辑，内容略少，随便撇两眼，不是重点。

## 前言

本文介绍监听器的相关逻辑。在服务引用和服务发现中监听器处于的位置请看下面的图：

1. 服务暴露：

![dubbo-export](https://segmentfault.com/img/remote/1460000017827076)

1. 服务引用：

![dubbo-refer](https://segmentfault.com/img/remote/1460000017827077)

这两个监听器所做的工作不是很多，来看看源码理解一下。

## 源码分析

### （一）ListenerInvokerWrapper

该类实现了Invoker，是服务引用监听器的包装类。

### 1.属性

```
/**
 * invoker对象
 */
private final Invoker<T> invoker;

/**
 * 监听器集合
 */
private final List<InvokerListener> listeners;
```

用到了装饰模式，其中很多实现方法直接调用了invoker的方法。

### 2.构造方法

```
public ListenerInvokerWrapper(Invoker<T> invoker, List<InvokerListener> listeners) {
    // 如果invoker为空则抛出异常
    if (invoker == null) {
        throw new IllegalArgumentException("invoker == null");
    }
    this.invoker = invoker;
    this.listeners = listeners;
    if (listeners != null && !listeners.isEmpty()) {
        // 遍历监听器
        for (InvokerListener listener : listeners) {
            if (listener != null) {
                try {
                    // 调用在服务引用的时候进行监听
                    listener.referred(invoker);
                } catch (Throwable t) {
                    logger.error(t.getMessage(), t);
                }
            }
        }
    }
}
```

构造方法中直接调用了监听器的服务引用。

#### 3.destroy

```
@Override
public void destroy() {
    try {
        // 销毁invoker
        invoker.destroy();
    } finally {
        // 销毁所有监听的实体域
        if (listeners != null && !listeners.isEmpty()) {
            for (InvokerListener listener : listeners) {
                if (listener != null) {
                    try {
                        listener.destroyed(invoker);
                    } catch (Throwable t) {
                        logger.error(t.getMessage(), t);
                    }
                }
            }
        }
    }
}
```

该方法是把服务引用的监听器销毁。

### （二）InvokerListenerAdapter

```
public abstract class InvokerListenerAdapter implements InvokerListener {

    /**
     * 引用服务
     * @param invoker
     * @throws RpcException
     */
    @Override
    public void referred(Invoker<?> invoker) throws RpcException {
    }

    /**
     * 销毁
     * @param invoker
     */
    @Override
    public void destroyed(Invoker<?> invoker) {
    }

}
```

该类是服务引用监听器的适配类，没有做实际的操作。

### （三）DeprecatedInvokerListener

```
@Activate(Constants.DEPRECATED_KEY)
public class DeprecatedInvokerListener extends InvokerListenerAdapter {

    private static final Logger LOGGER = LoggerFactory.getLogger(DeprecatedInvokerListener.class);

    @Override
    public void referred(Invoker<?> invoker) throws RpcException {
        // 当该引用的服务被废弃时，打印错误日志
        if (invoker.getUrl().getParameter(Constants.DEPRECATED_KEY, false)) {
            LOGGER.error("The service " + invoker.getInterface().getName() + " is DEPRECATED! Declare from " + invoker.getUrl());
        }
    }

}
```

该类是当调用废弃的服务时候打印错误日志。

### （四）ListenerExporterWrapper

该类是服务暴露监听器包装类。

#### 1.属性

```
/**
 * 服务暴露者
 */
private final Exporter<T> exporter;

/**
 * 服务暴露监听者集合
 */
private final List<ExporterListener> listeners;
```

用到了装饰模式，其中很多实现方法直接调用了exporter的方法。

#### 2.构造方法

```
public ListenerExporterWrapper(Exporter<T> exporter, List<ExporterListener> listeners) {
    if (exporter == null) {
        throw new IllegalArgumentException("exporter == null");
    }
    this.exporter = exporter;
    this.listeners = listeners;
    if (listeners != null && !listeners.isEmpty()) {
        RuntimeException exception = null;
        // 遍历服务暴露监听集合
        for (ExporterListener listener : listeners) {
            if (listener != null) {
                try {
                    // 暴露服务监听
                    listener.exported(this);
                } catch (RuntimeException t) {
                    logger.error(t.getMessage(), t);
                    exception = t;
                }
            }
        }
        if (exception != null) {
            throw exception;
        }
    }
}
```

该方法中对于每个服务暴露进行监听。

#### 3.unexport

```
@Override
public void unexport() {
    try {
        // 取消暴露
        exporter.unexport();
    } finally {
        if (listeners != null && !listeners.isEmpty()) {
            RuntimeException exception = null;
            // 遍历监听集合
            for (ExporterListener listener : listeners) {
                if (listener != null) {
                    try {
                        // 监听取消暴露
                        listener.unexported(this);
                    } catch (RuntimeException t) {
                        logger.error(t.getMessage(), t);
                        exception = t;
                    }
                }
            }
            if (exception != null) {
                throw exception;
            }
        }
    }
}
```

该方法是对每个取消服务暴露的监听。

### （五）ExporterListenerAdapter

```
public abstract class ExporterListenerAdapter implements ExporterListener {

    /**
     * 暴露服务
     * @param exporter
     * @throws RpcException
     */
    @Override
    public void exported(Exporter<?> exporter) throws RpcException {
    }

    /**
     * 取消暴露服务
     * @param exporter
     * @throws RpcException
     */
    @Override
    public void unexported(Exporter<?> exporter) throws RpcException {
    }

}
```

该类是服务暴露监听器的适配类，没有做实际的操作。

# 远程调用——Protocol

# 远程调用——Protocol

> 目标：介绍远程调用中协议的设计和实现，介绍dubbo-rpc-api中的各种protocol包的源码，是重点内容。

## 前言

在远程调用中协议是非常重要的一层，看下面这张图：

![dubbo-framework](https://segmentfault.com/img/remote/1460000017854957?w=900&h=674)

该层是在信息交换层之上，分为了并且夹杂在服务暴露和服务引用中间，为了有一个约定的方式进行调用。

dubbo支持不同协议的扩展，比如http、thrift等等，具体的可以参照官方文档。本文讲解的源码大部分是对于公共方法的实现，而具体的服务暴露和服务引用会在各个协议实现中讲到。

下面是该包下面的类图：

![protocol包类图](https://segmentfault.com/img/remote/1460000017854958?w=1896&h=688)

## 源码分析

### （一）AbstractProtocol

该类是协议的抽象类，实现了Protocol接口，其中实现了一些公共的方法，抽象方法在它的子类AbstractProxyProtocol中定义。

#### 1.属性

```
/**
 * 服务暴露者集合
 */
protected final Map<String, Exporter<?>> exporterMap = new ConcurrentHashMap<String, Exporter<?>>();

/**
 * 服务引用者集合
 */
//TODO SOFEREFENCE
protected final Set<Invoker<?>> invokers = new ConcurrentHashSet<Invoker<?>>();
```

#### 2.serviceKey

```
protected static String serviceKey(URL url) {
    // 获得绑定的端口号
    int port = url.getParameter(Constants.BIND_PORT_KEY, url.getPort());
    return serviceKey(port, url.getPath(), url.getParameter(Constants.VERSION_KEY),
            url.getParameter(Constants.GROUP_KEY));
}

protected static String serviceKey(int port, String serviceName, String serviceVersion, String serviceGroup) {
    return ProtocolUtils.serviceKey(port, serviceName, serviceVersion, serviceGroup);
}
```

该方法是为了得到服务key group+"/"+serviceName+":"+serviceVersion+":"+port

#### 3.destroy

```
@Override
public void destroy() {
    // 遍历服务引用实体
    for (Invoker<?> invoker : invokers) {
        if (invoker != null) {
            // 从集合中移除
            invokers.remove(invoker);
            try {
                if (logger.isInfoEnabled()) {
                    logger.info("Destroy reference: " + invoker.getUrl());
                }
                // 销毁
                invoker.destroy();
            } catch (Throwable t) {
                logger.warn(t.getMessage(), t);
            }
        }
    }
    // 遍历服务暴露者
    for (String key : new ArrayList<String>(exporterMap.keySet())) {
        // 从集合中移除
        Exporter<?> exporter = exporterMap.remove(key);
        if (exporter != null) {
            try {
                if (logger.isInfoEnabled()) {
                    logger.info("Unexport service: " + exporter.getInvoker().getUrl());
                }
                // 取消暴露
                exporter.unexport();
            } catch (Throwable t) {
                logger.warn(t.getMessage(), t);
            }
        }
    }
}
```

该方法是对invoker和exporter的销毁。

### （二）AbstractProxyProtocol

该类继承了AbstractProtocol类，其中利用了代理工厂对AbstractProtocol中的两个集合进行了填充，并且对异常做了处理。

#### 1.属性

```
/**
 * rpc的异常类集合
 */
private final List<Class<?>> rpcExceptions = new CopyOnWriteArrayList<Class<?>>();

/**
 * 代理工厂
 */
private ProxyFactory proxyFactory;
```

#### 2.export

```
@Override
@SuppressWarnings("unchecked")
public <T> Exporter<T> export(final Invoker<T> invoker) throws RpcException {
    // 获得uri
    final String uri = serviceKey(invoker.getUrl());
    // 获得服务暴露者
    Exporter<T> exporter = (Exporter<T>) exporterMap.get(uri);
    if (exporter != null) {
        return exporter;
    }
    // 新建一个线程
    final Runnable runnable = doExport(proxyFactory.getProxy(invoker, true), invoker.getInterface(), invoker.getUrl());
    exporter = new AbstractExporter<T>(invoker) {
        /**
         * 取消暴露
         */
        @Override
        public void unexport() {
            super.unexport();
            // 移除该key对应的服务暴露者
            exporterMap.remove(uri);
            if (runnable != null) {
                try {
                    // 启动线程
                    runnable.run();
                } catch (Throwable t) {
                    logger.warn(t.getMessage(), t);
                }
            }
        }
    };
    // 加入集合
    exporterMap.put(uri, exporter);
    return exporter;
}
```

其中分为两个步骤，创建一个exporter，放入到集合汇中。在创建exporter时对unexport方法进行了重写。

#### 3.refer

```
@Override
public <T> Invoker<T> refer(final Class<T> type, final URL url) throws RpcException {
    // 通过代理获得实体域
    final Invoker<T> target = proxyFactory.getInvoker(doRefer(type, url), type, url);
    Invoker<T> invoker = new AbstractInvoker<T>(type, url) {
        @Override
        protected Result doInvoke(Invocation invocation) throws Throwable {
            try {
                // 获得调用结果
                Result result = target.invoke(invocation);
                Throwable e = result.getException();
                // 如果抛出异常，则抛出相应异常
                if (e != null) {
                    for (Class<?> rpcException : rpcExceptions) {
                        if (rpcException.isAssignableFrom(e.getClass())) {
                            throw getRpcException(type, url, invocation, e);
                        }
                    }
                }
                return result;
            } catch (RpcException e) {
                // 抛出未知异常
                if (e.getCode() == RpcException.UNKNOWN_EXCEPTION) {
                    e.setCode(getErrorCode(e.getCause()));
                }
                throw e;
            } catch (Throwable e) {
                throw getRpcException(type, url, invocation, e);
            }
        }
    };
    // 加入集合
    invokers.add(invoker);
    return invoker;
}
```

该方法是服务引用，先从代理工厂中获得Invoker对象target，然后创建了真实的invoker在重写方法中调用代理的方法，最后加入到集合。

```
protected abstract <T> Runnable doExport(T impl, Class<T> type, URL url) throws RpcException;

protected abstract <T> T doRefer(Class<T> type, URL url) throws RpcException;
```

可以看到其中抽象了服务引用和暴露的方法，让各类协议各自实现。

### （三）AbstractInvoker

该类是invoker的抽象方法，因为协议被夹在服务引用和服务暴露中间，无论什么协议都有一些通用的Invoker和exporter的方法实现，而该类就是实现了Invoker的公共方法，而把doInvoke抽象出来，让子类只关注这个方法。

#### 1.属性

```
/**
 * 服务类型
 */
private final Class<T> type;

/**
 * url对象
 */
private final URL url;

/**
 * 附加值
 */
private final Map<String, String> attachment;

/**
 * 是否可用
 */
private volatile boolean available = true;

/**
 *  是否销毁
 */
private AtomicBoolean destroyed = new AtomicBoolean(false);
```

#### 2.convertAttachment

```
private static Map<String, String> convertAttachment(URL url, String[] keys) {
    if (keys == null || keys.length == 0) {
        return null;
    }
    Map<String, String> attachment = new HashMap<String, String>();
    // 遍历key，把值放入附加值集合中
    for (String key : keys) {
        String value = url.getParameter(key);
        if (value != null && value.length() > 0) {
            attachment.put(key, value);
        }
    }
    return attachment;
}
```

该方法是转化为附加值，把url中的值转化为服务调用invoker的附加值。

#### 3.invoke

```
@Override
public Result invoke(Invocation inv) throws RpcException {
    // if invoker is destroyed due to address refresh from registry, let's allow the current invoke to proceed
    // 如果服务引用销毁，则打印告警日志，但是通过
    if (destroyed.get()) {
        logger.warn("Invoker for service " + this + " on consumer " + NetUtils.getLocalHost() + " is destroyed, "
                + ", dubbo version is " + Version.getVersion() + ", this invoker should not be used any longer");
    }

    RpcInvocation invocation = (RpcInvocation) inv;
    // 会话域中加入该调用链
    invocation.setInvoker(this);
    // 把附加值放入会话域
    if (attachment != null && attachment.size() > 0) {
        invocation.addAttachmentsIfAbsent(attachment);
    }
    // 把上下文的附加值放入会话域
    Map<String, String> contextAttachments = RpcContext.getContext().getAttachments();
    if (contextAttachments != null && contextAttachments.size() != 0) {
        /**
         * invocation.addAttachmentsIfAbsent(context){@link RpcInvocation#addAttachmentsIfAbsent(Map)}should not be used here,
         * because the {@link RpcContext#setAttachment(String, String)} is passed in the Filter when the call is triggered
         * by the built-in retry mechanism of the Dubbo. The attachment to update RpcContext will no longer work, which is
         * a mistake in most cases (for example, through Filter to RpcContext output traceId and spanId and other information).
         */
        invocation.addAttachments(contextAttachments);
    }
    // 如果开启的是异步调用，则把该设置也放入附加值
    if (getUrl().getMethodParameter(invocation.getMethodName(), Constants.ASYNC_KEY, false)) {
        invocation.setAttachment(Constants.ASYNC_KEY, Boolean.TRUE.toString());
    }
    // 加入编号
    RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);


    try {
        // 执行调用链
        return doInvoke(invocation);
    } catch (InvocationTargetException e) { // biz exception
        Throwable te = e.getTargetException();
        if (te == null) {
            return new RpcResult(e);
        } else {
            if (te instanceof RpcException) {
                ((RpcException) te).setCode(RpcException.BIZ_EXCEPTION);
            }
            return new RpcResult(te);
        }
    } catch (RpcException e) {
        if (e.isBiz()) {
            return new RpcResult(e);
        } else {
            throw e;
        }
    } catch (Throwable e) {
        return new RpcResult(e);
    }
}
```

该方法做了一些公共的操作，比如服务引用销毁的检测，加入附加值，加入调用链实体域到会话域中等。然后执行了doInvoke抽象方法。各协议自己去实现。

### （四）AbstractExporter

该类和AbstractInvoker类似，也是在服务暴露中实现了一些公共方法。

#### 1.属性

```
/**
 * 实体域
 */
private final Invoker<T> invoker;

/**
 * 是否取消暴露服务
 */
private volatile boolean unexported = false;
```

#### 2.unexport

```
@Override
public void unexport() {
    // 如果已经消取消暴露，则之间返回
    if (unexported) {
        return;
    }
    // 设置为true
    unexported = true;
    // 销毁该实体域
    getInvoker().destroy();
}
```

### （五）InvokerWrapper

该类是Invoker的包装类，其中用到类装饰模式，不过并没有实现实际的功能增强。

```
public class InvokerWrapper<T> implements Invoker<T> {

    /**
     * invoker对象
     */
    private final Invoker<T> invoker;

    private final URL url;

    public InvokerWrapper(Invoker<T> invoker, URL url) {
        this.invoker = invoker;
        this.url = url;
    }

    @Override
    public Class<T> getInterface() {
        return invoker.getInterface();
    }

    @Override
    public URL getUrl() {
        return url;
    }

    @Override
    public boolean isAvailable() {
        return invoker.isAvailable();
    }

    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        return invoker.invoke(invocation);
    }

    @Override
    public void destroy() {
        invoker.destroy();
    }

}
```

### （六）ProtocolFilterWrapper

该类实现了Protocol接口，其中也用到了装饰模式，是对Protocol的装饰，是在服务引用和暴露的方法上加上了过滤器功能。

#### 1.buildInvokerChain

```
private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
    Invoker<T> last = invoker;
    // 获得过滤器的所有扩展实现类实例集合
    List<Filter> filters = ExtensionLoader.getExtensionLoader(Filter.class).getActivateExtension(invoker.getUrl(), key, group);
    if (!filters.isEmpty()) {
        // 从最后一个过滤器开始循环，创建一个带有过滤器链的invoker对象
        for (int i = filters.size() - 1; i >= 0; i--) {
            final Filter filter = filters.get(i);
            // 记录last的invoker
            final Invoker<T> next = last;
            // 新建last
            last = new Invoker<T>() {

                @Override
                public Class<T> getInterface() {
                    return invoker.getInterface();
                }

                @Override
                public URL getUrl() {
                    return invoker.getUrl();
                }

                @Override
                public boolean isAvailable() {
                    return invoker.isAvailable();
                }

                /**
                 * 关键在这里，调用下一个filter代表的invoker，把每一个过滤器串起来
                 * @param invocation
                 * @return
                 * @throws RpcException
                 */
                @Override
                public Result invoke(Invocation invocation) throws RpcException {
                    return filter.invoke(next, invocation);
                }

                @Override
                public void destroy() {
                    invoker.destroy();
                }

                @Override
                public String toString() {
                    return invoker.toString();
                }
            };
        }
    }
    return last;
}
```

该方法就是创建带 Filter 链的 Invoker 对象。倒序的把每一个过滤器串连起来，形成一个invoker。

#### 2.export

```
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    // 如果是注册中心，则直接暴露服务
    if (Constants.REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
        return protocol.export(invoker);
    }
    // 服务提供侧暴露服务
    return protocol.export(buildInvokerChain(invoker, Constants.SERVICE_FILTER_KEY, Constants.PROVIDER));
}
```

该方法是在服务暴露上做了过滤器链的增强，也就是加上了过滤器。

#### 3.refer

```
@Override
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    // 如果是注册中心，则直接引用
    if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
        return protocol.refer(type, url);
    }
    // 消费者侧引用服务
    return buildInvokerChain(protocol.refer(type, url), Constants.REFERENCE_FILTER_KEY, Constants.CONSUMER);
}
```

该方法是在服务引用上做了过滤器链的增强，也就是加上了过滤器。

### （七）ProtocolListenerWrapper

该类也实现了Protocol，也是装饰了Protocol接口，但是它是在服务引用和暴露过程中加上了监听器的功能。

#### 1.export

```
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    // 如果是注册中心，则暴露该invoker
    if (Constants.REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
        return protocol.export(invoker);
    }
    // 创建一个暴露者监听器包装类对象
    return new ListenerExporterWrapper<T>(protocol.export(invoker),
            Collections.unmodifiableList(ExtensionLoader.getExtensionLoader(ExporterListener.class)
                    .getActivateExtension(invoker.getUrl(), Constants.EXPORTER_LISTENER_KEY)));
}
```

该方法是在服务暴露上做了监听器功能的增强，也就是加上了监听器。

#### 2.refer

```
@Override
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    // 如果是注册中心。则直接引用服务
    if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
        return protocol.refer(type, url);
    }
    // 创建引用服务监听器包装类对象
    return new ListenerInvokerWrapper<T>(protocol.refer(type, url),
            Collections.unmodifiableList(
                    ExtensionLoader.getExtensionLoader(InvokerListener.class)
                            .getActivateExtension(url, Constants.INVOKER_LISTENER_KEY)));
}
```

该方法是在服务引用上做了监听器功能的增强，也就是加上了监听器。