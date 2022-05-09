# Javaagent

## 背景介绍

### 需求说明

需求是在程序运行期间，向某个类的某个方法前、后加入某段业务代码，或者直接替换整个方法的业务逻辑，即业务方法客制化。注意是运行期间动态更改，做到无侵入，而不是事先在代码中写死切入点或逻辑。

拿到这个需求，首先想到的是使用 spring aop 技术，但这种方式需要事先在方法上加注解进行拦截，可我们在服务启动前并不知道要拦截哪些方法。或者直接拦截所有方法，但这样或多或少都会有一些性能问题，每次方法调用时，都会进入切面，需要判断是否需要对这个方法做客制化，而判断的规则以及客制化代码一般存储在缓存中，这时还会涉及缓存查询，性能肯定会有所降低。鉴于以上考虑，选择 Java 动态字节码技术 来实现。

### 动态字节码技术

Java 代码都是要被编译成字节码后才能放到 JVM 里执行的，而字节码一旦被加载到虚拟机中，就可以被解释执行。字节码文件（.class）就是普通的二进制文件，它是通过 Java 编译器生成的。而只要是文件就可以被改变，如果我们用特定的规则解析了原有的字节码文件，对它进行修改或者干脆重新定义，这不就可以改变代码行为了么。动态字节码技术优势在于 Java 字节码生成之后，对其进行修改，增强其功能，这种方式相当于对应用程序的二进制文件进行修改。

Java 生态里有很多可以动态处理字节码的技术，比较流行的有两个，一个是 ASM，一个是 Javassist 。

ASM：直接操作字节码指令，执行效率高，但涉及到JVM的操作和指令，要求使用者掌握Java类字节码文件格式及指令，对使用者的要求比较高。

Javassist：提供了更高级的API，执行效率相对较差，但无需掌握字节码指令的知识，简单、快速，对使用者要求较低。

考虑到简单易用性，这里选择 Javassist 工具来实现。

### 技术设计

① 首先需要一个扫描服务类及方法的功能，这样我们才能选择某个方法切入。

调用客户端服务扫描切入点接口，需要扫描出服务中的包名、类名、方法名、以及方法参数列表。

② 维护规则，配置切入的位置、业务代码。

位置可以是前置、后置、替换。客制化的代码类需要实现 ICustomizeHandler 接口的 execute 方法，目的是固定结构。

在切入方法时，只需要创建这个 handler 的实例对象，然后执行 execute 方法即可。这种方式比较简单，但也有一定的局限性。

在 execute 方法中如果要引用 spring 容器中的其它对象，需要通过 ApplicationContext 上下文获取，不能使用依赖注入，如果要使用依赖注入，还需要处理类的属性。

③ 维护切入点与规则之间的关系，因为一个切入点可以维护多个规则。

维护好规则和关系之后，就需要应用规则，即调用客户端客制化接口，动态应用规则。

### 准备工作

① 切入点、客制化代码、以及关系 已经维护好了，客制化 test-service 服务中 org.test.demo.app.service.impl.DemoServiceImpl 类的 selectOrder 方法，在方法前、后加一段代码，打印一些东西。

② OrderServiceImpl 的代码，之后通过观察控制台打印内容来确认客制化效果。

```
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.test.demo.app.service.DemoService;
import org.test.demo.domain.entity.Order;
import org.test.demo.domain.repository.OrderRepository;

@Service
public class DemoServiceImpl implements DemoService {

    private static final Logger LOGGER = LoggerFactory.getLogger(DemoServiceImpl.class);

    private final OrderRepository orderRepository;

    public DemoServiceImpl(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public List<Order> selectOrder(String orderNumber, String status) {

        Order params = new Order();
        params.setOrderNumber(orderNumber);
        params.setStatus(status);
        List<Order> orders = orderRepository.select(params);
        LOGGER.info("order size is {}", orders.size());

        return orders;
    }

}
```

背景及准备工作介绍完了，下面就来看看如何一步步实现动态切面的能力。接下来首先对一些必备知识做简要介绍，然后对实现过程中一些核心逻辑做介绍。

## 简介

对于Java 程序员来说，Java Intrumentation、Java agent 这些技术可能平时接触的很少。实际上，我们日常应用的各种工具中，有很多都是基于他们实现的，例如常见的热部署（JRebel, spring-loaded）、IDE debug、各种线上诊断工具（btrace,、Arthas）等等。

### Instrumentation(插桩)

使用 java.lang.instrument.Instrumentation，使得开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义。有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和 Java 类操作，这样的特性实际上提供了一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些 AOP 的功能了。Instrumentation 的最大作用，就是类定义动态改变和操作。

Instrumentation的一些主要方法如下：

```
package java.lang.instrument;

import  java.io.File;
import  java.io.IOException;
import  java.util.jar.JarFile;

/*
 * Copyright 2003 Wily Technology, Inc.
 */

/**
 * This class provides services needed to instrument Java
 * programming language code.
 * Instrumentation is the addition of byte-codes to methods for the
 * purpose of gathering data to be utilized by tools.
 * Since the changes are purely additive, these tools do not modify
 * application state or behavior.
 * Examples of such benign tools include monitoring agents, profilers,
 * coverage analyzers, and event loggers.
 *
 * <P>
 * There are two ways to obtain an instance of the
 * <code>Instrumentation</code> interface:
 *
 * <ol>
 *   <li><p> When a JVM is launched in a way that indicates an agent
 *     class. In that case an <code>Instrumentation</code> instance
 *     is passed to the <code>premain</code> method of the agent class.
 *     </p></li>
 *   <li><p> When a JVM provides a mechanism to start agents sometime
 *     after the JVM is launched. In that case an <code>Instrumentation</code>
 *     instance is passed to the <code>agentmain</code> method of the
 *     agent code. </p> </li>
 * </ol>
 * <p>
 * These mechanisms are described in the
 * {@linkplain java.lang.instrument package specification}.
 * <p>
 * Once an agent acquires an <code>Instrumentation</code> instance,
 * the agent may call methods on the instance at any time.
 *
 * @since   1.5
 */
public interface Instrumentation {
    /**
     * Registers the supplied transformer. All future class definitions
     * will be seen by the transformer, except definitions of classes upon which any
     * registered transformer is dependent.
     * The transformer is called when classes are loaded, when they are
     * {@linkplain #redefineClasses redefined}. and if <code>canRetransform</code> is true,
     * when they are {@linkplain #retransformClasses retransformed}.
     * See {@link java.lang.instrument.ClassFileTransformer#transform
     * ClassFileTransformer.transform} for the order
     * of transform calls.
     * If a transformer throws
     * an exception during execution, the JVM will still call the other registered
     * transformers in order. The same transformer may be added more than once,
     * but it is strongly discouraged -- avoid this by creating a new instance of
     * transformer class.
     * <P>
     * This method is intended for use in instrumentation, as described in the
     * {@linkplain Instrumentation class specification}.
     *
     * @param transformer          the transformer to register
     * @param canRetransform       can this transformer's transformations be retransformed
     * @throws java.lang.NullPointerException if passed a <code>null</code> transformer
     * @throws java.lang.UnsupportedOperationException if <code>canRetransform</code>
     * is true and the current configuration of the JVM does not allow
     * retransformation ({@link #isRetransformClassesSupported} is false)
     * @since 1.6
     */
     
      //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void
    addTransformer(ClassFileTransformer transformer, boolean canRetransform);

    /**
     * Registers the supplied transformer.
     * <P>
     * Same as <code>addTransformer(transformer, false)</code>.
     *
     * @param transformer          the transformer to register
     * @throws java.lang.NullPointerException if passed a <code>null</code> transformer
     * @see    #addTransformer(ClassFileTransformer,boolean)
     */
     
     //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    /**
     * 注册一个Transformer，从此之后的类加载都会被 transformer 拦截。
     * ClassFileTransformer 的 transform 方法可以直接对类的字节码进行修改，但是只能修改方法体，不能变更方法签名、增加和删除方法/类的成员属性
     */
    void
    addTransformer(ClassFileTransformer transformer);

    /**
     * Unregisters the supplied transformer. Future class definitions will
     * not be shown to the transformer. Removes the most-recently-added matching
     * instance of the transformer. Due to the multi-threaded nature of
     * class loading, it is possible for a transformer to receive calls
     * after it has been removed. Transformers should be written defensively
     * to expect this situation.
     *
     * @param transformer          the transformer to unregister
     * @return  true if the transformer was found and removed, false if the
     *           transformer was not found
     * @throws java.lang.NullPointerException if passed a <code>null</code> transformer
     */
     
     //删除一个类转换器
    boolean
    removeTransformer(ClassFileTransformer transformer);

    /**
     * Returns whether or not the current JVM configuration supports retransformation
     * of classes.
     * The ability to retransform an already loaded class is an optional capability
     * of a JVM.
     * Retransformation will only be supported if the
     * <code>Can-Retransform-Classes</code> manifest attribute is set to
     * <code>true</code> in the agent JAR file (as described in the
     * {@linkplain java.lang.instrument package specification}) and the JVM supports
     * this capability.
     * During a single instantiation of a single JVM, multiple calls to this
     * method will always return the same answer.
     * @return  true if the current JVM configuration supports retransformation of
     *          classes, false if not.
     * @see #retransformClasses
     * @since 1.6
     */
     
     //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    boolean
    isRetransformClassesSupported();

    /**
     * Retransform the supplied set of classes.
     *
     * <P>
     * This function facilitates the instrumentation
     * of already loaded classes.
     * When classes are initially loaded or when they are
     * {@linkplain #redefineClasses redefined},
     * the initial class file bytes can be transformed with the
     * {@link java.lang.instrument.ClassFileTransformer ClassFileTransformer}.
     * This function reruns the transformation process
     * (whether or not a transformation has previously occurred).
     * This retransformation follows these steps:
     *  <ul>
     *    <li>starting from the initial class file bytes
     *    </li>
     *    <li>for each transformer that was added with <code>canRetransform</code>
     *      false, the bytes returned by
     *      {@link java.lang.instrument.ClassFileTransformer#transform transform}
     *      during the last class load or redefine are
     *      reused as the output of the transformation; note that this is
     *      equivalent to reapplying the previous transformation, unaltered;
     *      except that
     *      {@link java.lang.instrument.ClassFileTransformer#transform transform}
     *      is not called
     *    </li>
     *    <li>for each transformer that was added with <code>canRetransform</code>
     *      true, the
     *      {@link java.lang.instrument.ClassFileTransformer#transform transform}
     *      method is called in these transformers
     *    </li>
     *    <li>the transformed class file bytes are installed as the new
     *      definition of the class
     *    </li>
     *  </ul>
     * <P>
     *
     * The order of transformation is described in the
     * {@link java.lang.instrument.ClassFileTransformer#transform transform} method.
     * This same order is used in the automatic reapplication of retransformation
     * incapable transforms.
     * <P>
     *
     * The initial class file bytes represent the bytes passed to
     * {@link java.lang.ClassLoader#defineClass ClassLoader.defineClass} or
     * {@link #redefineClasses redefineClasses}
     * (before any transformations
     *  were applied), however they might not exactly match them.
     *  The constant pool might not have the same layout or contents.
     *  The constant pool may have more or fewer entries.
     *  Constant pool entries may be in a different order; however,
     *  constant pool indices in the bytecodes of methods will correspond.
     *  Some attributes may not be present.
     *  Where order is not meaningful, for example the order of methods,
     *  order might not be preserved.
     *
     * <P>
     * This method operates on
     * a set in order to allow interdependent changes to more than one class at the same time
     * (a retransformation of class A can require a retransformation of class B).
     *
     * <P>
     * If a retransformed method has active stack frames, those active frames continue to
     * run the bytecodes of the original method.
     * The retransformed method will be used on new invokes.
     *
     * <P>
     * This method does not cause any initialization except that which would occur
     * under the customary JVM semantics. In other words, redefining a class
     * does not cause its initializers to be run. The values of static variables
     * will remain as they were prior to the call.
     *
     * <P>
     * Instances of the retransformed class are not affected.
     *
     * <P>
     * The retransformation may change method bodies, the constant pool and attributes.
     * The retransformation must not add, remove or rename fields or methods, change the
     * signatures of methods, or change inheritance.  These restrictions maybe be
     * lifted in future versions.  The class file bytes are not checked, verified and installed
     * until after the transformations have been applied, if the resultant bytes are in
     * error this method will throw an exception.
     *
     * <P>
     * If this method throws an exception, no classes have been retransformed.
     * <P>
     * This method is intended for use in instrumentation, as described in the
     * {@linkplain Instrumentation class specification}.
     *
     * @param classes array of classes to retransform;
     *                a zero-length array is allowed, in this case, this method does nothing
     * @throws java.lang.instrument.UnmodifiableClassException if a specified class cannot be modified
     * ({@link #isModifiableClass} would return <code>false</code>)
     * @throws java.lang.UnsupportedOperationException if the current configuration of the JVM does not allow
     * retransformation ({@link #isRetransformClassesSupported} is false) or the retransformation attempted
     * to make unsupported changes
     * @throws java.lang.ClassFormatError if the data did not contain a valid class
     * @throws java.lang.NoClassDefFoundError if the name in the class file is not equal to the name of the class
     * @throws java.lang.UnsupportedClassVersionError if the class file version numbers are not supported
     * @throws java.lang.ClassCircularityError if the new classes contain a circularity
     * @throws java.lang.LinkageError if a linkage error occurs
     * @throws java.lang.NullPointerException if the supplied classes  array or any of its components
     *                                        is <code>null</code>.
     *
     * @see #isRetransformClassesSupported
     * @see #addTransformer
     * @see java.lang.instrument.ClassFileTransformer
     * @since 1.6
     */
     
     /**
     * 对JVM已经加载的类重新触发类加载，使用上面注册的 ClassFileTransformer 重新对类进行修饰。
     */
    void
    retransformClasses(Class<?>... classes) throws UnmodifiableClassException;

    /**
     * Returns whether or not the current JVM configuration supports redefinition
     * of classes.
     * The ability to redefine an already loaded class is an optional capability
     * of a JVM.
     * Redefinition will only be supported if the
     * <code>Can-Redefine-Classes</code> manifest attribute is set to
     * <code>true</code> in the agent JAR file (as described in the
     * {@linkplain java.lang.instrument package specification}) and the JVM supports
     * this capability.
     * During a single instantiation of a single JVM, multiple calls to this
     * method will always return the same answer.
     * @return  true if the current JVM configuration supports redefinition of classes,
     * false if not.
     * @see #redefineClasses
     */
    boolean
    isRedefineClassesSupported();

    /**
     * Redefine the supplied set of classes using the supplied class files.
     *
     * <P>
     * This method is used to replace the definition of a class without reference
     * to the existing class file bytes, as one might do when recompiling from source
     * for fix-and-continue debugging.
     * Where the existing class file bytes are to be transformed (for
     * example in bytecode instrumentation)
     * {@link #retransformClasses retransformClasses}
     * should be used.
     *
     * <P>
     * This method operates on
     * a set in order to allow interdependent changes to more than one class at the same time
     * (a redefinition of class A can require a redefinition of class B).
     *
     * <P>
     * If a redefined method has active stack frames, those active frames continue to
     * run the bytecodes of the original method.
     * The redefined method will be used on new invokes.
     *
     * <P>
     * This method does not cause any initialization except that which would occur
     * under the customary JVM semantics. In other words, redefining a class
     * does not cause its initializers to be run. The values of static variables
     * will remain as they were prior to the call.
     *
     * <P>
     * Instances of the redefined class are not affected.
     *
     * <P>
     * The redefinition may change method bodies, the constant pool and attributes.
     * The redefinition must not add, remove or rename fields or methods, change the
     * signatures of methods, or change inheritance.  These restrictions maybe be
     * lifted in future versions.  The class file bytes are not checked, verified and installed
     * until after the transformations have been applied, if the resultant bytes are in
     * error this method will throw an exception.
     *
     * <P>
     * If this method throws an exception, no classes have been redefined.
     * <P>
     * This method is intended for use in instrumentation, as described in the
     * {@linkplain Instrumentation class specification}.
     *
     * @param definitions array of classes to redefine with corresponding definitions;
     *                    a zero-length array is allowed, in this case, this method does nothing
     * @throws java.lang.instrument.UnmodifiableClassException if a specified class cannot be modified
     * ({@link #isModifiableClass} would return <code>false</code>)
     * @throws java.lang.UnsupportedOperationException if the current configuration of the JVM does not allow
     * redefinition ({@link #isRedefineClassesSupported} is false) or the redefinition attempted
     * to make unsupported changes
     * @throws java.lang.ClassFormatError if the data did not contain a valid class
     * @throws java.lang.NoClassDefFoundError if the name in the class file is not equal to the name of the class
     * @throws java.lang.UnsupportedClassVersionError if the class file version numbers are not supported
     * @throws java.lang.ClassCircularityError if the new classes contain a circularity
     * @throws java.lang.LinkageError if a linkage error occurs
     * @throws java.lang.NullPointerException if the supplied definitions array or any of its components
     * is <code>null</code>
     * @throws java.lang.ClassNotFoundException Can never be thrown (present for compatibility reasons only)
     *
     * @see #isRedefineClassesSupported
     * @see #addTransformer
     * @see java.lang.instrument.ClassFileTransformer
     */
     
     /**
     * 重新定义类，不是使用 transformer 修饰，而是把处理结果(bytecode)直接给JVM。
     * 调用此方法同样只能修改方法体，不能变更方法签名、增加和删除方法/类的成员属性
     */
    void
    redefineClasses(ClassDefinition... definitions)
        throws  ClassNotFoundException, UnmodifiableClassException;


    /**
     * Determines whether a class is modifiable by
     * {@linkplain #retransformClasses retransformation}
     * or {@linkplain #redefineClasses redefinition}.
     * If a class is modifiable then this method returns <code>true</code>.
     * If a class is not modifiable then this method returns <code>false</code>.
     * <P>
     * For a class to be retransformed, {@link #isRetransformClassesSupported} must also be true.
     * But the value of <code>isRetransformClassesSupported()</code> does not influence the value
     * returned by this function.
     * For a class to be redefined, {@link #isRedefineClassesSupported} must also be true.
     * But the value of <code>isRedefineClassesSupported()</code> does not influence the value
     * returned by this function.
     * <P>
     * Primitive classes (for example, <code>java.lang.Integer.TYPE</code>)
     * and array classes are never modifiable.
     *
     * @param theClass the class to check for being modifiable
     * @return whether or not the argument class is modifiable
     * @throws java.lang.NullPointerException if the specified class is <code>null</code>.
     *
     * @see #retransformClasses
     * @see #isRetransformClassesSupported
     * @see #redefineClasses
     * @see #isRedefineClassesSupported
     * @since 1.6
     */
    boolean
    isModifiableClass(Class<?> theClass);

    /**
     * Returns an array of all classes currently loaded by the JVM.
     *
     * @return an array containing all the classes loaded by the JVM, zero-length if there are none
     */
    @SuppressWarnings("rawtypes")
    
    // 获取当前被JVM加载的所有类对象
    Class[]
    getAllLoadedClasses();

    /**
     * Returns an array of all classes for which <code>loader</code> is an initiating loader.
     * If the supplied loader is <code>null</code>, classes initiated by the bootstrap class
     * loader are returned.
     *
     * @param loader          the loader whose initiated class list will be returned
     * @return an array containing all the classes for which loader is an initiating loader,
     *          zero-length if there are none
     */
    @SuppressWarnings("rawtypes")
    
    
    Class[]
    getInitiatedClasses(ClassLoader loader);

    /**
     * Returns an implementation-specific approximation of the amount of storage consumed by
     * the specified object. The result may include some or all of the object's overhead,
     * and thus is useful for comparison within an implementation but not between implementations.
     *
     * The estimate may change during a single invocation of the JVM.
     *
     * @param objectToSize     the object to size
     * @return an implementation-specific approximation of the amount of storage consumed by the specified object
     * @throws java.lang.NullPointerException if the supplied Object is <code>null</code>.
     */
     
     //获取一个对象的大小
    long
    getObjectSize(Object objectToSize);


    /**
     * Specifies a JAR file with instrumentation classes to be defined by the
     * bootstrap class loader.
     *
     * <p> When the virtual machine's built-in class loader, known as the "bootstrap
     * class loader", unsuccessfully searches for a class, the entries in the {@link
     * java.util.jar.JarFile JAR file} will be searched as well.
     *
     * <p> This method may be used multiple times to add multiple JAR files to be
     * searched in the order that this method was invoked.
     *
     * <p> The agent should take care to ensure that the JAR does not contain any
     * classes or resources other than those to be defined by the bootstrap
     * class loader for the purpose of instrumentation.
     * Failure to observe this warning could result in unexpected
     * behavior that is difficult to diagnose. For example, suppose there is a
     * loader L, and L's parent for delegation is the bootstrap class loader.
     * Furthermore, a method in class C, a class defined by L, makes reference to
     * a non-public accessor class C$1. If the JAR file contains a class C$1 then
     * the delegation to the bootstrap class loader will cause C$1 to be defined
     * by the bootstrap class loader. In this example an <code>IllegalAccessError</code>
     * will be thrown that may cause the application to fail. One approach to
     * avoiding these types of issues, is to use a unique package name for the
     * instrumentation classes.
     *
     * <p>
     * <cite>The Java&trade; Virtual Machine Specification</cite>
     * specifies that a subsequent attempt to resolve a symbolic
     * reference that the Java virtual machine has previously unsuccessfully attempted
     * to resolve always fails with the same error that was thrown as a result of the
     * initial resolution attempt. Consequently, if the JAR file contains an entry
     * that corresponds to a class for which the Java virtual machine has
     * unsuccessfully attempted to resolve a reference, then subsequent attempts to
     * resolve that reference will fail with the same error as the initial attempt.
     *
     * @param   jarfile
     *          The JAR file to be searched when the bootstrap class loader
     *          unsuccessfully searches for a class.
     *
     * @throws  NullPointerException
     *          If <code>jarfile</code> is <code>null</code>.
     *
     * @see     #appendToSystemClassLoaderSearch
     * @see     java.lang.ClassLoader
     * @see     java.util.jar.JarFile
     *
     * @since 1.6
     */
     
         /**
     * 将一个jar加入到bootstrap classloader 的 classpath 里
     */
    void
    appendToBootstrapClassLoaderSearch(JarFile jarfile);

    /**
     * Specifies a JAR file with instrumentation classes to be defined by the
     * system class loader.
     *
     * When the system class loader for delegation (see
     * {@link java.lang.ClassLoader#getSystemClassLoader getSystemClassLoader()})
     * unsuccessfully searches for a class, the entries in the {@link
     * java.util.jar.JarFile JarFile} will be searched as well.
     *
     * <p> This method may be used multiple times to add multiple JAR files to be
     * searched in the order that this method was invoked.
     *
     * <p> The agent should take care to ensure that the JAR does not contain any
     * classes or resources other than those to be defined by the system class
     * loader for the purpose of instrumentation.
     * Failure to observe this warning could result in unexpected
     * behavior that is difficult to diagnose (see
     * {@link #appendToBootstrapClassLoaderSearch
     * appendToBootstrapClassLoaderSearch}).
     *
     * <p> The system class loader supports adding a JAR file to be searched if
     * it implements a method named <code>appendToClassPathForInstrumentation</code>
     * which takes a single parameter of type <code>java.lang.String</code>. The
     * method is not required to have <code>public</code> access. The name of
     * the JAR file is obtained by invoking the {@link java.util.zip.ZipFile#getName
     * getName()} method on the <code>jarfile</code> and this is provided as the
     * parameter to the <code>appendToClassPathForInstrumentation</code> method.
     *
     * <p>
     * <cite>The Java&trade; Virtual Machine Specification</cite>
     * specifies that a subsequent attempt to resolve a symbolic
     * reference that the Java virtual machine has previously unsuccessfully attempted
     * to resolve always fails with the same error that was thrown as a result of the
     * initial resolution attempt. Consequently, if the JAR file contains an entry
     * that corresponds to a class for which the Java virtual machine has
     * unsuccessfully attempted to resolve a reference, then subsequent attempts to
     * resolve that reference will fail with the same error as the initial attempt.
     *
     * <p> This method does not change the value of <code>java.class.path</code>
     * {@link java.lang.System#getProperties system property}.
     *
     * @param   jarfile
     *          The JAR file to be searched when the system class loader
     *          unsuccessfully searches for a class.
     *
     * @throws  UnsupportedOperationException
     *          If the system class loader does not support appending a
     *          a JAR file to be searched.
     *
     * @throws  NullPointerException
     *          If <code>jarfile</code> is <code>null</code>.
     *
     * @see     #appendToBootstrapClassLoaderSearch
     * @see     java.lang.ClassLoader#getSystemClassLoader
     * @see     java.util.jar.JarFile
     * @since 1.6
     */
     
      /**
     * 将一个jar加入到 system classloader 的 classpath 里
     */
    void
    appendToSystemClassLoaderSearch(JarFile jarfile);

    /**
     * Returns whether the current JVM configuration supports
     * {@linkplain #setNativeMethodPrefix(ClassFileTransformer,String)
     * setting a native method prefix}.
     * The ability to set a native method prefix is an optional
     * capability of a JVM.
     * Setting a native method prefix will only be supported if the
     * <code>Can-Set-Native-Method-Prefix</code> manifest attribute is set to
     * <code>true</code> in the agent JAR file (as described in the
     * {@linkplain java.lang.instrument package specification}) and the JVM supports
     * this capability.
     * During a single instantiation of a single JVM, multiple
     * calls to this method will always return the same answer.
     * @return  true if the current JVM configuration supports
     * setting a native method prefix, false if not.
     * @see #setNativeMethodPrefix
     * @since 1.6
     */
    boolean
    isNativeMethodPrefixSupported();

    /**
     * This method modifies the failure handling of
     * native method resolution by allowing retry
     * with a prefix applied to the name.
     * When used with the
     * {@link java.lang.instrument.ClassFileTransformer ClassFileTransformer},
     * it enables native methods to be
     * instrumented.
     * <p>
     * Since native methods cannot be directly instrumented
     * (they have no bytecodes), they must be wrapped with
     * a non-native method which can be instrumented.
     * For example, if we had:
     * <pre>
     *   native boolean foo(int x);</pre>
     * <p>
     * We could transform the class file (with the
     * ClassFileTransformer during the initial definition
     * of the class) so that this becomes:
     * <pre>
     *   boolean foo(int x) {
     *     <i>... record entry to foo ...</i>
     *     return wrapped_foo(x);
     *   }
     *
     *   native boolean wrapped_foo(int x);</pre>
     * <p>
     * Where <code>foo</code> becomes a wrapper for the actual native
     * method with the appended prefix "wrapped_".  Note that
     * "wrapped_" would be a poor choice of prefix since it
     * might conceivably form the name of an existing method
     * thus something like "$$$MyAgentWrapped$$$_" would be
     * better but would make these examples less readable.
     * <p>
     * The wrapper will allow data to be collected on the native
     * method call, but now the problem becomes linking up the
     * wrapped method with the native implementation.
     * That is, the method <code>wrapped_foo</code> needs to be
     * resolved to the native implementation of <code>foo</code>,
     * which might be:
     * <pre>
     *   Java_somePackage_someClass_foo(JNIEnv* env, jint x)</pre>
     * <p>
     * This function allows the prefix to be specified and the
     * proper resolution to occur.
     * Specifically, when the standard resolution fails, the
     * resolution is retried taking the prefix into consideration.
     * There are two ways that resolution occurs, explicit
     * resolution with the JNI function <code>RegisterNatives</code>
     * and the normal automatic resolution.  For
     * <code>RegisterNatives</code>, the JVM will attempt this
     * association:
     * <pre>{@code
     *   method(foo) -> nativeImplementation(foo)
     * }</pre>
     * <p>
     * When this fails, the resolution will be retried with
     * the specified prefix prepended to the method name,
     * yielding the correct resolution:
     * <pre>{@code
     *   method(wrapped_foo) -> nativeImplementation(foo)
     * }</pre>
     * <p>
     * For automatic resolution, the JVM will attempt:
     * <pre>{@code
     *   method(wrapped_foo) -> nativeImplementation(wrapped_foo)
     * }</pre>
     * <p>
     * When this fails, the resolution will be retried with
     * the specified prefix deleted from the implementation name,
     * yielding the correct resolution:
     * <pre>{@code
     *   method(wrapped_foo) -> nativeImplementation(foo)
     * }</pre>
     * <p>
     * Note that since the prefix is only used when standard
     * resolution fails, native methods can be wrapped selectively.
     * <p>
     * Since each <code>ClassFileTransformer</code>
     * can do its own transformation of the bytecodes, more
     * than one layer of wrappers may be applied. Thus each
     * transformer needs its own prefix.  Since transformations
     * are applied in order, the prefixes, if applied, will
     * be applied in the same order
     * (see {@link #addTransformer(ClassFileTransformer,boolean) addTransformer}).
     * Thus if three transformers applied
     * wrappers, <code>foo</code> might become
     * <code>$trans3_$trans2_$trans1_foo</code>.  But if, say,
     * the second transformer did not apply a wrapper to
     * <code>foo</code> it would be just
     * <code>$trans3_$trans1_foo</code>.  To be able to
     * efficiently determine the sequence of prefixes,
     * an intermediate prefix is only applied if its non-native
     * wrapper exists.  Thus, in the last example, even though
     * <code>$trans1_foo</code> is not a native method, the
     * <code>$trans1_</code> prefix is applied since
     * <code>$trans1_foo</code> exists.
     *
     * @param   transformer
     *          The ClassFileTransformer which wraps using this prefix.
     * @param   prefix
     *          The prefix to apply to wrapped native methods when
     *          retrying a failed native method resolution. If prefix
     *          is either <code>null</code> or the empty string, then
     *          failed native method resolutions are not retried for
     *          this transformer.
     * @throws java.lang.NullPointerException if passed a <code>null</code> transformer.
     * @throws java.lang.UnsupportedOperationException if the current configuration of
     *           the JVM does not allow setting a native method prefix
     *           ({@link #isNativeMethodPrefixSupported} is false).
     * @throws java.lang.IllegalArgumentException if the transformer is not registered
     *           (see {@link #addTransformer(ClassFileTransformer,boolean) addTransformer}).
     *
     * @since 1.6
     */
    void
    setNativeMethodPrefix(ClassFileTransformer transformer, String prefix);
}
```

### Javaagent

Java agent 是一种特殊的Java程序（Jar文件），它是 Instrumentation 的客户端。与普通 Java 程序通过main方法启动不同，agent 并不是一个可以单独启动的程序，而必须依附在一个Java应用程序（JVM）上，与它运行在同一个进程中，通过 Instrumentation API 与虚拟机交互。

Java agent 与 Instrumentation 密不可分，二者也需要在一起使用。因为JVM 会把 Instrumentation 的实例会作为参数注入到 Java agent 的启动方法中。因此如果想使用 Instrumentation 功能，拿到 Instrumentation 实例，我们必须通过Java agent。

Java agent 有两个启动时机，一个是在程序启动时通过 -javaagent 参数启动代理程序，一个是在程序运行期间通过 Java Tool API 中的 attach api 动态启动代理程序。

javaagent的主要功能如下：

- 可以在加载class文件之前做拦截，对字节码做修改
- 可以在运行期对已加载类的字节码做变更，但是这种情况下会有很多的限制，后面会详细说
- 还有其他一些小众的功能
  - 获取所有已经加载过的类
  - 获取所有已经初始化过的类（执行过clinit方法，是上面的一个子集）
  - 获取某个对象的大小
  - 将某个jar加入到bootstrap classpath里作为高优先级被bootstrapClassloader加载
  - 将某个jar加入到classpath里供AppClassloard去加载
  - 设置某些native方法的前缀，主要在查找native方法的时候做规则匹配

#### ① JVM启动时静态加载

对于VM启动时加载的 agent，Instrumentation 会通过 premain 方法传入代理程序，premain 方法会在程序 main 方法执行之前被调用。此时大部分Java类都没有被加载（“大部分”是因为，agent类本身和它依赖的类还是无法避免的会先加载的），是一个对类加载埋点做手脚（addTransformer）的好机会。但这种方式有很大的局限性，Instrumentation 仅限于 main 函数执行前，此时有很多类还没有被加载，如果想为其注入 Instrumentation 就无法办到。

```
/**
 * agentArgs 是 premain 函数得到的程序参数，通过 -javaagent 传入。这个参数是个字符串，如果程序参数有多个，需要程序自行解析这个字符串。
 * inst 是一个 java.lang.instrument.Instrumentation 的实例，由 JVM 自动传入。
 */
public static void premain(String agentArgs, Instrumentation inst) {

}

/**
 * 带有 Instrumentation 参数的 premain 优先级高于不带此参数的 premain。
 * 如果存在带 Instrumentation 参数的 premain，不带此参数的 premain 将被忽略。
 */
public static void premain(String agentArgs) {

}
```

这种方式的应用比如在 IDEA 启动 debug 模式时，就是以 -javaagent 的形式启动 debug 代理程序实现的。

Javaagent是java命令的一个参数。参数 javaagent 可以用于指定一个 jar 包，并且对该 java 包有2个要求：

1. 这个 jar 包的 MANIFEST.MF 文件必须指定 Premain-Class 项。
2. Premain-Class 指定的那个类必须实现 premain() 方法。

premain 方法，从字面上理解，就是运行在 main 函数之前的的类。当Java 虚拟机启动时，在执行 main 函数之前，JVM 会先运行`-javaagent`所指定 jar 包内 Premain-Class 这个类的 premain 方法 。

在命令行输入 `java`可以看到相应的参数，其中有 和 java agent相关的：

```shell
Copy-agentlib:<libname>[=<选项>] 加载本机代理库 <libname>, 例如 -agentlib:hprof
	另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
-agentpath:<pathname>[=<选项>]
	按完整路径名加载本机代理库
-javaagent:<jarpath>[=<选项>]
	加载 Java 编程语言代理, 请参阅 java.lang.instrument
```

在上面`-javaagent`参数中提到了参阅`java.lang.instrument`，这是在`rt.jar` 中定义的一个包，该路径下有两个重要的类：

```
java.lang.instrument.Instrumentation
java.lang.instrument.ClassFileTransformer
```

该包提供了一些工具帮助开发人员在 Java 程序运行时，动态修改系统中的 Class 类型。其中，使用该软件包的一个关键组件就是 Javaagent。从名字上看，似乎是个 Java 代理之类的，而实际上，他的功能更像是一个Class 类型的转换器，他可以在运行时接受重新外部请求，对Class类型进行修改。

从本质上讲，Java Agent 是一个遵循一组严格约定的常规 Java 类。 上面说到 javaagent命令要求指定的类中必须要有premain()方法，并且对premain方法的签名也有要求，签名必须满足以下两种格式：

```java
Copypublic static void premain(String agentArgs, Instrumentation inst)
    
public static void premain(String agentArgs)
```

JVM 会优先加载 带 `Instrumentation` 签名的方法，加载成功忽略第二种，如果第一种没有，则加载第二种方法。这个逻辑在sun.instrument.InstrumentationImpl 类中：

```
    private void loadClassAndStartAgent(String var1, String var2, String var3) throws Throwable {
        ClassLoader var4 = ClassLoader.getSystemClassLoader();
        Class var5 = var4.loadClass(var1);
        Method var6 = null;
        NoSuchMethodException var7 = null;
        boolean var8 = false;

        try {
            var6 = var5.getDeclaredMethod(var2, String.class, Instrumentation.class);
            var8 = true;
        } catch (NoSuchMethodException var13) {
            var7 = var13;
        }

        if (var6 == null) {
            try {
                var6 = var5.getDeclaredMethod(var2, String.class);
            } catch (NoSuchMethodException var12) {
            }
        }

        if (var6 == null) {
            try {
                var6 = var5.getMethod(var2, String.class, Instrumentation.class);
                var8 = true;
            } catch (NoSuchMethodException var11) {
            }
        }

        if (var6 == null) {
            try {
                var6 = var5.getMethod(var2, String.class);
            } catch (NoSuchMethodException var10) {
                throw var7;
            }
        }

        setAccessible(var6, true);
        if (var8) {
            var6.invoke((Object)null, var3, this);
        } else {
            var6.invoke((Object)null, var3);
        }

        setAccessible(var6, false);
    }
```

最为重要的是上面注释的几个方法，下面我们会用到。

如何使用javaagent？

使用 javaagent 需要几个步骤：

1. 定义一个 MANIFEST.MF 文件，必须包含 Premain-Class 选项，通常也会加入Can-Redefine-Classes 和 Can-Retransform-Classes 选项。
2. 创建一个Premain-Class 指定的类，类中包含 premain 方法，方法逻辑由用户自己确定。
3. 将 premain 的类和 MANIFEST.MF 文件打成 jar 包。
4. 使用参数 -javaagent: jar包路径 启动要代理的方法。

在执行以上步骤后，JVM 会先执行 premain 方法，大部分类加载都会通过该方法，注意：是大部分，不是所有。当然，遗漏的主要是系统类，因为很多系统类先于 agent 执行，而用户类的加载肯定是会被拦截的。也就是说，这个方法是在 main 方法启动前拦截大部分类的加载活动，既然可以拦截类的加载，那么就可以去做重写类这样的操作，结合第三方的字节码编译工具，比如ASM，javassist，cglib等等来改写实现类。

通过上面的步骤我们用代码实现来实现。实现 javaagent 你需要搭建两个工程，一个工程是用来承载 javaagent类，单独的打成jar包；一个工程是javaagent需要去代理的类。即javaagent会在这个工程中的main方法启动之前去做一些事情。

1.首先来实现javaagent工程。

工程目录结构如下：

```txt
Copy-java-agent
----src
--------main
--------|------java
--------|----------com.demo.learn
--------|------------PreMainTraceAgent
--------|resources
-----------META-INF
--------------MANIFEST.MF
```

第一步是需要创建一个类，包含premain 方法：

```java
Copyimport java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.instrument.Instrumentation;
import java.security.ProtectionDomain;


public class PreMainTraceAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("agentArgs : " + agentArgs);
        inst.addTransformer(new DefineTransformer(), true);
    }

    static class DefineTransformer implements ClassFileTransformer{

        @Override
        public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
            System.out.println("premain load Class:" + className);
            return classfileBuffer;
        }
    }
}
```

上面就是我实现的一个类，实现了带Instrumentation参数的premain()方法。调用addTransformer()方法对启动时所有的类进行拦截。

然后在 resources 目录下新建目录：META-INF，在该目录下新建文件：MANIFREST.MF：

```txt
CopyManifest-Version: 1.0
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Premain-Class: PreMainTraceAgent
```

注意到第5行有空行。

说一下MANIFREST.MF文件的作用，这里如果你不去手动指定的话，直接 打包，默认会在打包的文件中生成一个MANIFREST.MF文件：

```txt
CopyManifest-Version: 1.0
Implementation-Title: test-agent
Implementation-Version: 0.0.1-SNAPSHOT
Built-By: yangyue
Implementation-Vendor-Id: com.demo.learn
Spring-Boot-Version: 2.0.9.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.rickiyang.learn.LearnApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Created-By: Apache Maven 3.5.2
Build-Jdk: 1.8.0_151
Implementation-URL: https://projects.spring.io/spring-boot/#/spring-bo
 ot-starter-parent/test-agent
```

这是默认的文件，包含当前的一些版本信息，当前工程的启动类，它还有别的参数允许你做更多的事情，可以用上的有：

> Premain-Class ：包含 premain 方法的类（类的全路径名）
>
> Agent-Class ：包含 agentmain 方法的类（类的全路径名）
>
> Boot-Class-Path ：设置引导类加载器搜索的路径列表。查找类的特定于平台的机制失败后，引导类加载器会搜索这些路径。按列出的顺序搜索路径。列表中的路径由一个或多个空格分开。路径使用分层 URI 的路径组件语法。如果该路径以斜杠字符（“/”）开头，则为绝对路径，否则为相对路径。相对路径根据代理 JAR 文件的绝对路径解析。忽略格式不正确的路径和不存在的路径。如果代理是在 VM 启动之后某一时刻启动的，则忽略不表示 JAR 文件的路径。（可选）
>
> Can-Redefine-Classes ：true表示能重定义此代理所需的类，默认值为 false（可选）
>
> Can-Retransform-Classes ：true 表示能重转换此代理所需的类，默认值为 false （可选）
>
> Can-Set-Native-Method-Prefix： true表示能设置此代理所需的本机方法前缀，默认值为 false（可选）

即在该文件中主要定义了程序运行相关的配置信息，程序运行前会先检测该文件中的配置项。

一个java程序中`-javaagent`参数的个数是没有限制的，所以可以添加任意多个javaagent。所有的java agent会按照你定义的顺序执行，例如：

```shell
Copyjava -javaagent:agent1.jar -javaagent:agent2.jar -jar MyProgram.jar
```

程序执行的顺序将会是：

MyAgent1.premain -> MyAgent2.premain -> MyProgram.main

**说回上面的 javaagent工程，接下来将该工程打成jar包，我在打包的时候发现打完包之后的 MANIFREST.MF文件被默认配置替换掉了。所以我是手动将上面我的配置文件替换到jar包中的文件，这里你需要注意。**

另外的再说一种不去手动写MANIFREST.MF文件的方式，使用maven插件：

```xml
Copy<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <archive>
            <!--自动添加META-INF/MANIFEST.MF -->
            <manifest>
                <addClasspath>true</addClasspath>
            </manifest>
            <manifestEntries>
                <Premain-Class>com.rickiyang.learn.PreMainTraceAgent</Premain-Class>
                <Agent-Class>com.rickiyang.learn.PreMainTraceAgent</Agent-Class>
                <Can-Redefine-Classes>true</Can-Redefine-Classes>
                <Can-Retransform-Classes>true</Can-Retransform-Classes>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

用这种插件的方式也可以自动生成该文件。

agent代码就写完了，下面再重新开一个工程，你只需要写一个带 main 方法的类即可：

```java
Copypublic class TestMain {

    public static void main(String[] args) {
        System.out.println("main start");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("main end");
    }
}
```

很简单，然后需要做的就是将上面的 代理类 和 这个测试类关联起来。有两种方式：

如果你用的是idea，那么你可以点击菜单： run-debug configuration，然后将你的代理类包 指定在 启动参数中即可：

另一种方式是不用 编译器，采用命令行的方法。与上面大致相同，将 上面的测试类编译成 class文件，然后 运行该类即可：

```shell
Copy #将该类编译成class文件
 > javac TestMain.java
 
 #指定agent程序并运行该类
 > java -javaagent:c:/alg.jar TestMain
```

使用上面两种方式都可以运行,输出结果如下：

```txt
CopyD:\soft\jdk1.8\bin\java.exe -javaagent:c:/alg.jar "-javaagent:D:\soft\IntelliJ IDEA 2019.1.1\lib\idea_rt.jar=54274:D:\soft\IntelliJ IDEA 2019.1.1\bin" -Dfile.encoding=UTF-8 -classpath D:\soft\jdk1.8\jre\lib\charsets.jar;D:\soft\jdk1.8\jre\lib\deploy.jar;D:\soft\jdk1.8\jre\lib\ext\access-bridge-64.jar;D:\soft\jdk1.8\jre\lib\ext\cldrdata.jar;D:\soft\jdk1.8\jre\lib\ext\dnsns.jar;D:\soft\jdk1.8\jre\lib\ext\jaccess.jar;D:\soft\jdk1.8\jre\lib\ext\jfxrt.jar;D:\soft\jdk1.8\jre\lib\ext\localedata.jar;D:\soft\jdk1.8\jre\lib\ext\nashorn.jar;D:\soft\jdk1.8\jre\lib\ext\sunec.jar;D:\soft\jdk1.8\jre\lib\ext\sunjce_provider.jar;D:\soft\jdk1.8\jre\lib\ext\sunmscapi.jar;D:\soft\jdk1.8\jre\lib\ext\sunpkcs11.jar;D:\soft\jdk1.8\jre\lib\ext\zipfs.jar;D:\soft\jdk1.8\jre\lib\javaws.jar;D:\soft\jdk1.8\jre\lib\jce.jar;D:\soft\jdk1.8\jre\lib\jfr.jar;D:\soft\jdk1.8\jre\lib\jfxswt.jar;D:\soft\jdk1.8\jre\lib\jsse.jar;D:\soft\jdk1.8\jre\lib\management-agent.jar;D:\soft\jdk1.8\jre\lib\plugin.jar;D:\soft\jdk1.8\jre\lib\resources.jar;D:\soft\jdk1.8\jre\lib\rt.jar;D:\workspace\demo1\target\classes;E:\.m2\repository\org\springframework\boot\spring-boot-starter-aop\2.1.1.RELEASE\spring-
...
...
...
1.8.11.jar;E:\.m2\repository\com\google\guava\guava\20.0\guava-20.0.jar;E:\.m2\repository\org\apache\commons\commons-lang3\3.7\commons-lang3-3.7.jar;E:\.m2\repository\com\alibaba\fastjson\1.2.54\fastjson-1.2.54.jar;E:\.m2\repository\org\springframework\boot\spring-boot\2.1.0.RELEASE\spring-boot-2.1.0.RELEASE.jar;E:\.m2\repository\org\springframework\spring-context\5.1.3.RELEASE\spring-context-5.1.3.RELEASE.jar com.springboot.example.demo.service.TestMain
agentArgs : null
premain load Class     :java/util/concurrent/ConcurrentHashMap$ForwardingNode
premain load Class     :sun/nio/cs/ThreadLocalCoders
premain load Class     :sun/nio/cs/ThreadLocalCoders$1
premain load Class     :sun/nio/cs/ThreadLocalCoders$Cache
premain load Class     :sun/nio/cs/ThreadLocalCoders$2
premain load Class     :java/util/jar/Attributes
premain load Class     :java/util/jar/Manifest$FastInputStream
...
...
...
premain load Class     :java/lang/Class$MethodArray
premain load Class     :java/lang/Void
main start
premain load Class     :sun/misc/VMSupport
premain load Class     :java/util/Hashtable$KeySet
premain load Class     :sun/nio/cs/ISO_8859_1$Encoder
premain load Class     :sun/nio/cs/Surrogate$Parser
premain load Class     :sun/nio/cs/Surrogate
...
...
...
premain load Class     :sun/util/locale/provider/LocaleResources$ResourceReference
main end
premain load Class     :java/lang/Shutdown
premain load Class     :java/lang/Shutdown$Lock

Process finished with exit code 0
```

上面的输出结果我们能够发现：

1. 执行main方法之前会加载所有的类，包括系统类和自定义类；
2. 在ClassFileTransformer中会去拦截系统类和自己实现的类对象；
3. 如果你有对某些类对象进行改写，那么在拦截的时候抓住该类使用字节码编译工具即可实现。

下面是使用javassist来动态将某个方法替换掉：

```java
Copypackage com.demo.learn;

import javassist.*;

import java.io.IOException;
import java.lang.instrument.ClassFileTransformer;
import java.security.ProtectionDomain;


public class MyClassTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(final ClassLoader loader, final String className, final Class<?> classBeingRedefined,final ProtectionDomain protectionDomain, final byte[] classfileBuffer) {
        // 操作Date类
        if ("java/util/Date".equals(className)) {
            try {
                // 从ClassPool获得CtClass对象
                final ClassPool classPool = ClassPool.getDefault();
                final CtClass clazz = classPool.get("java.util.Date");
                CtMethod convertToAbbr = clazz.getDeclaredMethod("convertToAbbr");
                //这里对 java.util.Date.convertToAbbr() 方法进行了改写，在 return之前增加了一个 打印操作
                String methodBody = "{sb.append(Character.toUpperCase(name.charAt(0)));" +
                        "sb.append(name.charAt(1)).append(name.charAt(2));" +
                        "System.out.println(\"sb.toString()\");" +
                        "return sb;}";
                convertToAbbr.setBody(methodBody);

                // 返回字节码，并且detachCtClass对象
                byte[] byteCode = clazz.toBytecode();
                //detach的意思是将内存中曾经被javassist加载过的Date对象移除，如果下次有需要在内存中找不到会重新走javassist加载
                clazz.detach();
                return byteCode;
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
        // 如果返回null则字节码不会被修改
        return null;
    }
}
```

#### ② JVM 启动后动态加载

对于VM启动后动态加载的 agent，Instrumentation 会通过 agentmain 方法传入代理程序，agentmain 在 main 函数开始运行后才被调用。

```
/**
 * agentArgs 是 agentmain 函数得到的程序参数，在 attach 时传入。这个参数是个字符串，如果程序参数有多个，需要程序自行解析这个字符串。
 * inst 是一个 java.lang.instrument.Instrumentation 的实例，由 JVM 自动传入。
 */
public static void agentmain(String agentArgs, Instrumentation inst) {

}

/**
 * 带有 Instrumentation 参数的 agentmain 优先级高于不带此参数的 agentmain。
 * 如果存在带 Instrumentation 参数的 agentmain，不带此参数的 agentmain 将被忽略。
 */
public static void agentmain(String agentArgs) {

}
```

这种方式的应用比如在启用 Arthas 来诊断线上问题时，通过 attach api，来动态加载代理程序到目标VM。

上面介绍的Instrumentation是在 JDK 1.5中提供的，开发者只能在main加载之前添加手脚，在 Java SE 6 的 Instrumentation 当中，提供了一个新的代理操作方法：agentmain，可以在 main 函数开始运行之后再运行。

跟`premain`函数一样， 开发者可以编写一个含有`agentmain`函数的 Java 类：

```java
Copy//采用attach机制，被代理的目标程序VM有可能很早之前已经启动，当然其所有类已经被加载完成，这个时候需要借助Instrumentation#retransformClasses(Class<?>... classes)让对应的类可以重新转换，从而激活重新转换的类执行ClassFileTransformer列表中的回调
public static void agentmain (String agentArgs, Instrumentation inst)

public static void agentmain (String agentArgs)
```

同样，agentmain 方法中带Instrumentation参数的方法也比不带优先级更高。开发者必须在 manifest 文件里面设置“Agent-Class”来指定包含 agentmain 函数的类。

在Java6 以后实现启动后加载的新实现是Attach api。Attach API 很简单，只有 2 个主要的类，都在 `com.sun.tools.attach` 包里面：

1. `VirtualMachine` 字面意义表示一个Java 虚拟机，也就是程序需要监控的目标虚拟机，提供了获取系统信息(比如获取内存dump、线程dump，类信息统计(比如已加载的类以及实例个数等)， loadAgent，Attach 和 Detach （Attach 动作的相反行为，从 JVM 上面解除一个代理）等方法，可以实现的功能可以说非常之强大 。该类允许我们通过给attach方法传入一个jvm的pid(进程id)，远程连接到jvm上 。

   代理类注入操作只是它众多功能中的一个，通过`loadAgent`方法向jvm注册一个代理程序agent，在该agent的代理程序中会得到一个Instrumentation实例，该实例可以 在class加载前改变class的字节码，也可以在class加载后重新加载。在调用Instrumentation实例的方法时，这些方法会使用ClassFileTransformer接口中提供的方法进行处理。

2. `VirtualMachineDescriptor` 则是一个描述虚拟机的容器类，配合 VirtualMachine 类完成各种功能。

attach实现动态注入的原理如下：

通过VirtualMachine类的`attach(pid)`方法，便可以attach到一个运行中的java进程上，之后便可以通过`loadAgent(agentJarPath)`来将agent的jar包注入到对应的进程，然后对应的进程会调用agentmain方法。

既然是两个进程之间通信那肯定的建立起连接，VirtualMachine.attach动作类似TCP创建连接的三次握手，目的就是搭建attach通信的连接。而后面执行的操作，例如vm.loadAgent，其实就是向这个socket写入数据流，接收方target VM会针对不同的传入数据来做不同的处理。

我们来测试一下agentmain的使用：

工程结构和 上面premain的测试一样，编写AgentMainTest，然后使用maven插件打包 生成MANIFEST.MF。

```java
Copypackage com.demo.learn;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.instrument.Instrumentation;
import java.security.ProtectionDomain;

/**
 * @author rickiyang
 * @date 2019-08-16
 * @Desc
 */
public class AgentMainTest {

    public static void agentmain(String agentArgs, Instrumentation instrumentation) {
        instrumentation.addTransformer(new DefineTransformer(), true);
    }
    
    static class DefineTransformer implements ClassFileTransformer {

        @Override
        public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
            System.out.println("premain load Class:" + className);
            return classfileBuffer;
        }
    }
}
```

将agent打包之后，就是编写测试main方法。

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.1.0</version>
  <configuration>
    <archive>
      <!--自动添加META-INF/MANIFEST.MF -->
      <manifest>
        <addClasspath>true</addClasspath>
      </manifest>
      <manifestEntries>
        <Agent-Class>com.demo.learn.AgentMainTest</Agent-Class>
        <Can-Redefine-Classes>true</Can-Redefine-Classes>
        <Can-Retransform-Classes>true</Can-Retransform-Classes>
      </manifestEntries>
    </archive>
  </configuration>
</plugin>
```

从一个attach JVM去探测目标JVM，如果目标JVM存在则向它发送agent.jar。我测试写的简单了些，找到当前JVM并加载agent.jar。

```java
Copypackage com.demo.learn.job;

import com.sun.tools.attach.*;

import java.io.IOException;
import java.util.List;


public class TestAgentMain {

    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        //获取当前系统中所有 运行中的 虚拟机
        System.out.println("running JVM start ");
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for (VirtualMachineDescriptor vmd : list) {
            //如果虚拟机的名称为 xxx 则 该虚拟机为目标虚拟机，获取该虚拟机的 pid
            //然后加载 agent.jar 发送给该虚拟机
            System.out.println(vmd.displayName());
            if (vmd.displayName().endsWith("com.rickiyang.learn.job.TestAgentMain")) {
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                virtualMachine.loadAgent("/Users/yangyue/Documents/java-agent.jar");
                virtualMachine.detach();
            }
        }
    }

}
```

list()方法会去寻找当前系统中所有运行着的JVM进程，你可以打印`vmd.displayName()`看到当前系统都有哪些JVM进程在运行。因为main函数执行起来的时候进程名为当前类名，所以通过这种方式可以去找到当前的进程id。

注意：在mac上安装了的jdk是能直接找到 VirtualMachine 类的，但是在windows中安装的jdk无法找到，如果你遇到这种情况，请手动将你jdk安装目录下：lib目录中的tools.jar添加进当前工程的Libraries中。

运行main方法的输出为：

可以看到实际上是启动了一个socket进程去传输agent.jar。先打印了“running JVM start”表名main方法是先启动了，然后才进入代理类的transform方法。

### MANIFEST.MF

写好的代理类想要运行，在打 jar 包前，还需要要在 MANIFEST.MF 中指定代理程序入口。

①、MANIFEST.MF

大多数 JAR 文件会包含一个 META-INF 目录，它用于存储包和扩展的配置数据，如安全性和版本信息。其中会有一个 MANIFEST.MF 文件，该文件包含了该 Jar 包的版本、创建人和类搜索路径等信息，如果是可执行Jar 包，会包含Main-Class属性，表明 Main 方法入口。

例如下面是通过 mvn clean package 命令打包后的 Jar 包中的 MANIFEST.MF 文件，从中可以看出 jar 的版本、创建者、SpringBoot 版本、程序入口、类搜索路径等信息。

```
Manifest-Version: 1.0
Premain-Class:MyAgent1
Created-By:1.6.0_06
```

② 与 agent 相关的参数

- **Premain-Class** ：JVM 启动时指定了代理，此属性指定代理类，**即包含 premain 方法的类**。
- **Agent-Class** ：JVM动态加载代理，此属性指定代理类，**即包含 agentmain 方法的类**。
- **Boot-Class-Path** ：设置引导类加载器搜索的路径列表，列表中的路径由一个或多个空格分开。
- **Can-Redefine-Classes** ：布尔值（true 或 false）。**是否能重定义此代理所需的类**。
- **Can-Retransform-Classes** ：布尔值（true 或 false）。**是否能重转换此代理所需的类**。
- **Can-Set-Native-Method-Prefix** ：布尔值（true 或 false）。**是否能设置此代理所需的本机方法前缀**。

### Attach API

Java agent 可以在JVM启动后再加载，就是通过 Attach API 实现的。当然，Attach API 不仅仅是为了实现动态加载 agent，Attach API 其实是跨JVM进程通讯的工具，能够将某种指令从一个JVM进程发送给另一个JVM进程。

加载 agent 只是 Attach API 发送的各种指令中的一种， 诸如 jstack 打印线程栈、jps 列出Java进程、jmap 做内存dump等功能，都属于Attach API 可以发送的指令。

Attach API不是Java的标准API，而是Sun公司提供的一套扩展API，用来向目标JVM"附着"(Attach)代理工具程序的。有了它，开发者可以方便的监控一个JVM，运行一个外加的代理程序。

① 引入 Attach API

在使用 Attach API时，需要引入 tools.jar 

```
<dependency>
    <groupId>jdk.tools</groupId>
    <artifactId>jdk.tools</artifactId>
    <version>1.8</version>
    <scope>system</scope>
    <systemPath>${env.JAVA_HOME}/lib/tools.jar</systemPath>
</dependency>
```

打包运行时，需要将 tools.jar 打包进去

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <includeSystemScope>true</includeSystemScope>
            </configuration>
        </plugin>
    </plugins>
</build>
```

② attach agent

```
// VirtualMachine等相关Class位于JDK的tools.jar
VirtualMachine vm = VirtualMachine.attach("1234");  // 1234表示目标JVM进程pid
try {
    vm.loadAgent(".../agent.jar");    // 指定agent的jar包路径，发送给目标进程
} finally {
    vm.detach();
}
```

#### instrument原理

`instrument`的底层实现依赖于`JVMTI(JVM Tool Interface)`，它是JVM暴露出来的一些供用户扩展的接口集合，JVMTI是基于事件驱动的，JVM每执行到一定的逻辑就会调用一些事件的回调接口（如果有的话），这些接口可以供开发者去扩展自己的逻辑。`JVMTIAgent`是一个利用`JVMTI`暴露出来的接口提供了代理启动时加载(agent on load)、代理通过attach形式加载(agent on attach)和代理卸载(agent on unload)功能的动态库。而`instrument agent`可以理解为一类`JVMTIAgent`动态库，别名是`JPLISAgent(Java Programming Language Instrumentation Services Agent)`，也就是**专门为java语言编写的插桩服务提供支持的代理**。

##### 启动时加载instrument agent过程：

1. 创建并初始化 JPLISAgent；
2. 监听 `VMInit` 事件，在 JVM 初始化完成之后做下面的事情：
   1. 创建 InstrumentationImpl 对象 ；
   2. 监听 ClassFileLoadHook 事件 ；
   3. 调用 InstrumentationImpl 的`loadClassAndCallPremain`方法，在这个方法里会去调用 javaagent 中 MANIFEST.MF 里指定的Premain-Class 类的 premain 方法 ；
3. 解析 javaagent 中 MANIFEST.MF 文件的参数，并根据这些参数来设置 JPLISAgent 里的一些内容。

##### 运行时加载instrument agent过程：

通过 JVM 的attach机制来请求目标 JVM 加载对应的agent，过程大致如下：

1. 创建并初始化JPLISAgent；
2. 解析 javaagent 里 MANIFEST.MF 里的参数；
3. 创建 InstrumentationImpl 对象；
4. 监听 ClassFileLoadHook 事件；
5. 调用 InstrumentationImpl 的`loadClassAndCallAgentmain`方法，在这个方法里会去调用javaagent里 MANIFEST.MF 里指定的`Agent-Class`类的`agentmain`方法。

#### Instrumentation的局限性

大多数情况下，我们使用Instrumentation都是使用其字节码插桩的功能，或者笼统说就是类重定义(Class Redefine)的功能，但是有以下的局限性：

1. premain和agentmain两种方式修改字节码的时机都是类文件加载之后，也就是说必须要带有Class类型的参数，不能通过字节码文件和自定义的类名重新定义一个本来不存在的类。
2. 类的字节码修改称为类转换(Class Transform)，类转换其实最终都回归到类重定义Instrumentation#redefineClasses()方法，此方法有以下限制：
   1. 新类和老类的父类必须相同；
   2. 新类和老类实现的接口数也要相同，并且是相同的接口；
   3. 新类和老类访问符必须一致。 新类和老类字段数和字段名要一致；
   4. 新类和老类新增或删除的方法必须是private static/final修饰的；
   5. 可以修改方法体。

除了上面的方式，如果想要重新定义一个类，可以考虑基于类加载器隔离的方式：创建一个新的自定义类加载器去通过新的字节码去定义一个全新的类，不过也存在只能通过反射调用该全新类的局限性。

# JVMTI

 [JVMTI](http://docs.oracle.com/javase/7/docs/platform/jvmti/jvmti.html)全称JVM Tool Interface，是JVM暴露出来的一些供用户扩展的接口集合。JVMTI是基于事件驱动的，JVM每执行到一定的逻辑就会调用一些事件的回调接口（如果有的话），这些接口可以供开发者扩展自己的逻辑。

比如最常见的，我们想在某个类的字节码文件读取之后、类定义之前修改相关的字节码，从而使创建的class对象是我们修改之后的字节码内容，那就可以实现一个回调函数赋给jvmtiEnv（JVMTI的运行时，通常一个JVMTIAgent对应一个jvmtiEnv，但是也可以对应多个）的回调方法集合里的ClassFileLoadHook，这样在接下来的类文件加载过程中都会调用到这个函数中，大致实现如下:，

```
  jvmtiEventCallbacks callbacks;
 
    jvmtiEnv *          jvmtienv = jvmti(agent);
 
    jvmtiError          jvmtierror;
 
    memset(&callbacks, 0, sizeof(callbacks));
 
    callbacks.ClassFileLoadHook = &eventHandlerClassFileLoadHook;
 
    jvmtierror = (*jvmtienv)->SetEventCallbacks( jvmtienv,
 
                                                 &callbacks,
 
                                                 sizeof(callbacks));

```

# JVMTIAgent

JVMTIAgent其实就是一个动态库，利用JVMTI暴露出来的一些接口来干一些我们想做、但是正常情况下又做不到的事情，不过为了和普通的动态库进行区分，它一般会实现如下的一个或者多个函数：

```
JNIEXPORT jint JNICALL
Agent_OnLoad(JavaVM *vm, char *options, void *reserved);
 
JNIEXPORT jint JNICALL
Agent_OnAttach(JavaVM* vm, char* options, void* reserved);
 
JNIEXPORT void JNICALL
Agent_OnUnload(JavaVM *vm); 
```

- Agent_OnLoad函数，如果agent是在启动时加载的，也就是在vm参数里通过-agentlib来指定的，那在启动过程中就会去执行这个agent里的Agent_OnLoad函数。
- Agent_OnAttach函数，如果agent不是在启动时加载的，而是我们先attach到目标进程上，然后给对应的目标进程发送load命令来加载，则在加载过程中会调用Agent_OnAttach函数。
- Agent_OnUnload函数，在agent卸载时调用，不过貌似基本上很少实现它。

其实我们每天都在和JVMTIAgent打交道，只是你可能没有意识到而已，比如我们经常使用Eclipse等工具调试Java代码，其实就是利用JRE自带的jdwp agent实现的，只是Eclipse等工具在没让你察觉的情况下将相关参数(类似-agentlib:jdwp=transport=dt_socket,suspend=y,address=localhost:61349)自动加到程序启动参数列表里了，其中agentlib参数就用来跟要加载的agent的名字，比如这里的jdwp(不过这不是动态库的名字，JVM会做一些名称上的扩展，比如在Linux下会去找libjdwp.so的动态库进行加载，也就是在名字的基础上加前缀lib，再加后缀.so)，接下来会跟一堆相关的参数，将这些参数传给Agent_OnLoad或者Agent_OnAttach函数里对应的options。

## instrument agent

instrument agent实现了Agent_OnLoad和Agent_OnAttach两方法，也就是说在使用时，agent既可以在启动时加载，也可以在运行时动态加载。其中启动时加载还可以通过类似-javaagent:myagent.jar的方式来间接加载instrument agent，运行时动态加载依赖的是JVM的attach机制（[JVM Attach机制实现](http://lovestblog.cn/blog/2014/06/18/jvm-attach/)），通过发送load命令来加载agent。

instrument agent的核心数据结构如下：

```
struct _JPLISAgent {
    JavaVM *                mJVM;                   /* handle to the JVM */
    JPLISEnvironment        mNormalEnvironment;     /* for every thing but retransform stuff */
    JPLISEnvironment        mRetransformEnvironment;/* for retransform stuff only */
    jobject                 mInstrumentationImpl;   /* handle to the Instrumentation instance */
    jmethodID               mPremainCaller;         /* method on the InstrumentationImpl that does the premain stuff (cached to save lots of lookups) */
    jmethodID               mAgentmainCaller;       /* method on the InstrumentationImpl for agents loaded via attach mechanism */
    jmethodID               mTransform;             /* method on the InstrumentationImpl that does the class file transform */
    jboolean                mRedefineAvailable;     /* cached answer to "does this agent support redefine" */
    jboolean                mRedefineAdded;         /* indicates if can_redefine_classes capability has been added */
    jboolean                mNativeMethodPrefixAvailable; /* cached answer to "does this agent support prefixing" */
    jboolean                mNativeMethodPrefixAdded;     /* indicates if can_set_native_method_prefix capability has been added */
    char const *            mAgentClassName;        /* agent class name */
    char const *            mOptionsString;         /* -javaagent options string */
};
 
struct _JPLISEnvironment {
    jvmtiEnv *              mJVMTIEnv;              /* the JVM TI environment */
    JPLISAgent *            mAgent;                 /* corresponding agent */
    jboolean                mIsRetransformer;       /* indicates if special environment */
};
```

这里解释一下几个重要项：

- mNormalEnvironment：主要提供正常的类transform及redefine功能。
- mRetransformEnvironment：主要提供类retransform功能。
- mInstrumentationImpl：这个对象非常重要，也是我们Java agent和JVM进行交互的入口，或许写过javaagent的人在写`premain`以及`agentmain`方法的时候注意到了有个Instrumentation参数，该参数其实就是这里的对象。
- mPremainCaller：指向`sun.instrument.InstrumentationImpl.loadClassAndCallPremain`方法，如果agent是在启动时加载的，则该方法会被调用。
- mAgentmainCaller：指向`sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain`方法，该方法在通过attach的方式动态加载agent的时候调用。
- mTransform：指向`sun.instrument.InstrumentationImpl.transform`方法。
- mAgentClassName：在我们javaagent的MANIFEST.MF里指定的`Agent-Class`。
- mOptionsString：传给agent的一些参数。
- mRedefineAvailable：是否开启了redefine功能，在javaagent的MANIFEST.MF里设置`Can-Redefine-Classes:true`。
- mNativeMethodPrefixAvailable：是否支持native方法前缀设置，同样在javaagent的MANIFEST.MF里设置`Can-Set-Native-Method-Prefix:true`。
- mIsRetransformer：如果在javaagent的MANIFEST.MF文件里定义了`Can-Retransform-Classes:true`，将会设置mRetransformEnvironment的mIsRetransformer为true。

## 在启动时加载instrument agent

正如前面“概述”里提到的方式，就是启动时加载instrument agent，具体过程都在`InvocationAdapter.c`的`Agent_OnLoad`方法里，这里简单描述下过程：

- 创建并初始化JPLISAgent
- 监听VMInit事件，在vm初始化完成之后做下面的事情：
  - 创建InstrumentationImpl对象
  - 监听ClassFileLoadHook事件
  - 调用InstrumentationImpl的`loadClassAndCallPremain`方法，在这个方法里会调用javaagent里MANIFEST.MF里指定的`Premain-Class`类的premain方法
- 解析javaagent里MANIFEST.MF里的参数，并根据这些参数来设置JPLISAgent里的一些内容

## 在运行时加载instrument agent

在运行时加载的方式，大致按照下面的方式来操作：

```cpp
VirtualMachine vm = VirtualMachine.attach(pid); 
vm.loadAgent(agentPath, agentArgs); 
```

上面会通过JVM的attach机制来请求目标JVM加载对应的agent，过程大致如下：

- 创建并初始化JPLISAgent
- 解析javaagent里MANIFEST.MF里的参数
- 创建InstrumentationImpl对象
- 监听ClassFileLoadHook事件
- 调用InstrumentationImpl的loadClassAndCallAgentmain方法，在这个方法里会调用javaagent里MANIFEST.MF里指定的Agent-Class类的agentmain方法

## instrument agent的ClassFileLoadHook回调实现

不管是启动时还是运行时加载的instrument agent，都关注着同一个jvmti事件——ClassFileLoadHook，这个事件是在读取字节码文件之后回调时用的，这样可以对原来的字节码做修改，那这里面究竟是怎样实现的呢？

```
void JNICALL
 
eventHandlerClassFileLoadHook(  jvmtiEnv *              jvmtienv,
                                JNIEnv *                jnienv,
                                jclass                  class_being_redefined,
                                jobject                 loader,
                                const char*             name,
                                jobject                 protectionDomain,
                                jint                    class_data_len,
                                const unsigned char*    class_data,
                                jint*                   new_class_data_len,
                                unsigned char**         new_class_data) {
 
    JPLISEnvironment * environment  = NULL;
 
    environment = getJPLISEnvironment(jvmtienv);
 
    /* if something is internally inconsistent (no agent), just silently return without touching the buffer */
 
    if ( environment != NULL ) {
 
        jthrowable outstandingException = preserveThrowable(jnienv);
        transformClassFile( environment->mAgent,
                            jnienv,
                            loader,
                            name,
                            class_being_redefined,
                            protectionDomain,
                            class_data_len,
                            class_data,
                            new_class_data_len,
                            new_class_data,
                            environment->mIsRetransformer);
 
        restoreThrowable(jnienv, outstandingException);
    }
 
}
```

先根据jvmtiEnv取得对应的JPLISEnvironment，因为上面我已经说到其实有两个JPLISEnvironment（并且有两个jvmtiEnv），其中一个是专门做retransform的，而另外一个用来做其他事情，根据不同的用途，在注册具体的ClassFileTransformer时也是分开的，对于作为retransform用的ClassFileTransformer，我们会注册到一个单独的TransformerManager里。

接着调用transformClassFile方法，由于函数实现比较长，这里就不贴代码了，大致意思就是调用InstrumentationImpl对象的transform方法，根据最后那个参数来决定选哪个TransformerManager里的ClassFileTransformer对象们做transform操作。

```
private byte[]
    transform(  ClassLoader         loader,
                String              classname,
                Class               classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer,
                boolean             isRetransformer) {
 
        TransformerManager mgr = isRetransformer?
 
                                        mRetransfomableTransformerManager :
                                        mTransformerManager;
 
        if (mgr == null) {
 
            return null; // no manager, no transform
 
        } else {
 
            return mgr.transform(   loader,
                                    classname,
                                    classBeingRedefined,
                                    protectionDomain,
                                    classfileBuffer);
 
        }
 
    }
 
 
  public byte[]
 
    transform(  ClassLoader         loader,
                String              classname,
                Class               classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer) {
 
        boolean someoneTouchedTheBytecode = false;
        TransformerInfo[]  transformerList = getSnapshotTransformerList();
        byte[]  bufferToUse = classfileBuffer;
 
        // order matters, gotta run 'em in the order they were added
 
        for ( int x = 0; x < transformerList.length; x++ ) {
 
            TransformerInfo         transformerInfo = transformerList[x];
            ClassFileTransformer    transformer = transformerInfo.transformer();
            byte[]                  transformedBytes = null;
 
            try {
 
                transformedBytes = transformer.transform(   loader,
                                                            classname,
                                                            classBeingRedefined,
                                                            protectionDomain,
                                                            bufferToUse);
 
            }
 
            catch (Throwable t) {
 
                // don't let any one transformer mess it up for the others.
                // This is where we need to put some logging. What should go here? FIXME
 
            }
 
 
            if ( transformedBytes != null ) {
                someoneTouchedTheBytecode = true;
                bufferToUse = transformedBytes;
            }
 
        }
 
 
        // if someone modified it, return the modified buffer.
        // otherwise return null to mean "no transforms occurred"
 
        byte [] result;
 
        if ( someoneTouchedTheBytecode ) {
            result = bufferToUse;
        }
        else {
            result = null;
        }
 
        return result;
 
    } 
```

以上是最终调到的java代码，可以看到已经调用到我们自己编写的javaagent代码里了，我们一般是实现一个ClassFileTransformer类，然后创建一个对象注册到对应的TransformerManager里。

# Class Transform的实现

这里说的class transform其实是狭义的，主要是针对第一次类文件加载时就要求被transform的场景，在加载类文件的时候发出ClassFileLoad事件，然后交给instrumenat agent来调用javaagent里注册的ClassFileTransformer实现字节码的修改。

# Class Redefine的实现

类重新定义，这是Instrumentation提供的基础功能之一，主要用在已经被加载过的类上，想对其进行修改，要做这件事，我们必须要知道两个东西，一个是要修改哪个类，另外一个是想将那个类修改成怎样的结构，有了这两个信息之后就可以通过InstrumentationImpl下面的redefineClasses方法操作了：

```
public void redefineClasses(ClassDefinition[]   definitions) throws  ClassNotFoundException {
 
        if (!isRedefineClassesSupported()) {
 
            throw new UnsupportedOperationException("redefineClasses is not supported in this environment");
 
        }
 
        if (definitions == null) {
 
            throw new NullPointerException("null passed as 'definitions' in redefineClasses");
 
        }
 
        for (int i = 0; i < definitions.length; ++i) {
 
            if (definitions[i] == null) {
 
                throw new NullPointerException("element of 'definitions' is null in redefineClasses");
 
            }
 
        }
 
        if (definitions.length == 0) {
 
            return; // short-circuit if there are no changes requested
 
        }
 
 
        redefineClasses0(mNativeAgent, definitions);
 
    }
```

在JVM里对应的实现是创建一个VM_RedefineClasses的VM_Operation，注意执行它的时候会stop-the-world：

```
jvmtiError
 
JvmtiEnv::RedefineClasses(jint class_count, const jvmtiClassDefinition* class_definitions) {
 
//TODO: add locking
 
  VM_RedefineClasses op(class_count, class_definitions, jvmti_class_load_kind_redefine);
 
  VMThread::execute(&op);
 
  return (op.check_error());
 
} /* end RedefineClasses */
```

这个过程我尽量用语言来描述清楚，不详细贴代码了，因为代码量实在有点大：

- 挨个遍历要批量重定义的jvmtiClassDefinition
- 然后读取新的字节码，如果有关注ClassFileLoadHook事件的，还会走对应的transform来对新的字节码再做修改
- 字节码解析好，创建一个klassOop对象
- 对比新老类，并要求如下：
  - 父类是同一个
  - 实现的接口数也要相同，并且是相同的接口
  - 类访问符必须一致
  - 字段数和字段名要一致
  - 新增的方法必须是private static/final的
  - 可以删除修改方法
- 对新类做字节码校验
- 合并新老类的常量池
- 如果老类上有断点，那都清除掉
- 对老类做JIT去优化
- 对新老方法匹配的方法的jmethodId做更新，将老的jmethodId更新到新的method上
- 新类的常量池的holer指向老的类
- 将新类和老类的一些属性做交换，比如常量池，methods，内部类
- 初始化新的vtable和itable
- 交换annotation的method、field、paramenter
- 遍历所有当前类的子类，修改他们的vtable及itable

上面是基本的过程，总的来说就是只更新了类里的内容，相当于只更新了指针指向的内容，并没有更新指针，避免了遍历大量已有类对象对它们进行更新所带来的开销。

# Class Retransform的实现

retransform class可以简单理解为回滚操作，具体回滚到哪个版本，这个需要看情况而定，下面不管那种情况都有一个前提，那就是javaagent已经要求要有retransform的能力了：

- 如果类是在第一次加载的的时候就做了transform，那么做retransform的时候会将代码回滚到transform之后的代码
- 如果类是在第一次加载的的时候没有任何变化，那么做retransform的时候会将代码回滚到最原始的类文件里的字节码
- 如果类已经加载了，期间类可能做过多次redefine(比如被另外一个agent做过)，但是接下来加载一个新的agent要求有retransform的能力了，然后对类做redefine的动作，那么retransform的时候会将代码回滚到上一个agent最后一次做redefine后的字节码

我们从InstrumentationImpl的retransformClasses方法参数看猜到应该是做回滚操作，因为我们只指定了class：

```
   public void retransformClasses(Class<?>[] classes) {
 
        if (!isRetransformClassesSupported()) {
 
            throw new UnsupportedOperationException( "retransformClasses is not supported in this environment");
 
        }
 
        retransformClasses0(mNativeAgent, classes);
 
    }
```

不过retransform的实现其实也是通过redefine的功能来实现，在类加载的时候有比较小的差别，主要体现在究竟会走哪些transform上，如果当前是做retransform的话，那将忽略那些注册到正常的TransformerManager里的ClassFileTransformer，而只会走专门为retransform而准备的TransformerManager的ClassFileTransformer，不然想象一下字节码又被无声无息改成某个中间态了。

```
private:
 
  void post_all_envs() {
 
    if (_load_kind != jvmti_class_load_kind_retransform) {
 
      // for class load and redefine,
 
      // call the non-retransformable agents
 
      JvmtiEnvIterator it;
 
      for (JvmtiEnv* env = it.first(); env != NULL; env = it.next(env)) {
 
        if (!env->is_retransformable() && env->is_enabled(JVMTI_EVENT_CLASS_FILE_LOAD_HOOK)) {
 
          // non-retransformable agents cannot retransform back,
 
          // so no need to cache the original class file bytes
 
          post_to_env(env, false);
 
        }
 
      }
 
    }
 
    JvmtiEnvIterator it;
 
    for (JvmtiEnv* env = it.first(); env != NULL; env = it.next(env)) {
 
      // retransformable agents get all events
 
      if (env->is_retransformable() && env->is_enabled(JVMTI_EVENT_CLASS_FILE_LOAD_HOOK)) {
 
        // retransformable agents need to cache the original class file
 
        // bytes if changes are made via the ClassFileLoadHook
 
        post_to_env(env, true);
 
      }
 
    }
 
  }
```

# javaagent的其他小众功能

javaagent除了做字节码上面的修改之外，其实还有一些小功能，有时候还是挺有用的

- 获取所有已经被加载的类：Class[] getAllLoadedClasses(); 
- 获取所有已经初始化了的类： Class[] getInitiatedClasses(ClassLoader loader); 
- 获取某个对象的大小： long getObjectSize(Object objectToSize); 
- 将某个jar加入到bootstrap classpath里优先其他jar被加载： void appendToBootstrapClassLoaderSearch(JarFile jarfile); 
- 将某个jar加入到classpath里供appclassloard去加载：void appendToSystemClassLoaderSearch(JarFile jarfile); 
- 设置某些native方法的前缀，主要在找native方法的时候做规则匹配： void setNativeMethodPrefix(ClassFileTransformer transformer, String prefix)。

# 参考资料

`bTrace`，`Arthas`

### 5、资料参考

其它更详细的相关知识请参考如下文章

[基于Java Instrument的Agent实现](https://juejin.im/post/5ac32eba5188255c313af0dd#heading-0)

[谈谈Java Intrumentation和相关应用](http://www.fanyilun.me/2017/07/18/谈谈Java Intrumentation和相关应用/)

[聊一聊 JAR 文件和 MANIFEST.MF](https://juejin.im/post/5d16cc8cf265da1b8d163237)

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 四、知识准备：JVM类加载器



### 1、类加载器简介

类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class 类的一个实例。每个这样的实例用来表示一个 Java 类。

基本上所有的类加载器都是 java.lang.ClassLoader 类的一个实例。java.lang.ClassLoader 类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class 类的一个实例。

Java 中的类加载器大致可以分成两类，一类是系统提供的，另外一类则是由 Java 应用开发人员编写的，开发人员可以通过继承 java.lang.ClassLoader 类的方式实现自定义类加载器，以满足一些特殊的需求。

系统提供的类加载器主要有下面三个：

- **引导类加载器(Bootstrap ClassLoader)**：负责将 $JAVA_HOME/lib 或者 -Xbootclasspath 参数指定路径下面的文件(按照文件名识别，如 rt.jar) 加载到虚拟机内存中。它用来加载 Java 的核心库，是用原生代码实现的，并不继承自 java.lang.ClassLoader，引导类加载器无法直接被 java 代码引用。
- **扩展类加载器(Extension ClassLoader)**：负责加载 $JAVA_HOME/lib/ext 目录中的文件，或者 java.ext.dirs 系统变量所指定的路径的类库，它用来加载 Java 的扩展库。
- **应用程序类加载器(Application ClassLoader)**：一般是系统的默认加载器，它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般 Java 应用的类都是由它来完成加载的，可以通过 ClassLoader.getSystemClassLoader() 来获取它。



### 2、类加载过程 — 双亲委派模型

① 类加载器结构

除了引导类加载器之外，所有的类加载器都有一个父类加载器。应用程序类加载器的父类加载器是扩展类加载器，扩展类加载器的父类加载器是引导类加载器。一般来说，开发人员自定义的类加载器的父类加载器是应用程序类加载器。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190828195112778-243066691.png)

② 双亲委派模型

类加载器在尝试去查找某个类的字节代码并定义它时，会先代理给其父类加载器，由父类加载器先去尝试加载这个类，如果父类加载器没有，继续寻找父类加载器，依次类推，如果到引导类加载器都没找到才从自身查找。这个类加载过程就是双亲委派模型。

首先要明白，Java 虚拟机判定两个 Java 类是否相同，不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样(可以通过 class.getClassLoader() 获得)。只有两个类来源于同一个Class文件，并且被同一个类加载器加载，这两个类才相等。不同类加载器加载的类之间是不兼容的。

双亲委派模型就是为了保证 Java 核心库的类型安全的。所有 Java 应用都至少需要引用 java.lang.Object 类，也就是说在运行的时候，java.lang.Object 这个类需要被加载到 Java 虚拟机中。如果这个加载过程由 Java 应用自己的类加载器来完成的话，很可能就存在多个版本的 java.lang.Object 类，而这些类之间是不兼容的。通过双亲委派模型，对于 Java 核心库的类加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。

类加载器在成功加载某个类之后，会把得到的 java.lang.Class 类的实例缓存起来。下次再请求加载该类的时候，类加载器会直接使用缓存的类的实例，而不会尝试再次加载。



### 3、线程上下文类加载器

线程上下文类加载器可通过 java.lang.Thread 中的方法 getContextClassLoader() 获得，可以通过 setContextClassLoader(ClassLoader cl) 来设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl) 方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是应用程序类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。



### 4、SpringBoot 类加载器

由于我是使用 SpringBoot (2.0.x) 开发，且打包成 jar 的形式部署在服务器上，这里有必要了解下 Spring boot 相关的类加载机制，遇到的很多问题就是由于 Spring Boot 的类加载机制导致的。

SpringBoot 的可执行jar包又称 fat jar ，是包含所有第三方依赖的 jar 包，jar 包中嵌入了除 java 虚拟机以外的所有依赖，是一个 all-in-one jar 包。普通插件 maven-jar-plugin 生成的包和 spring-boot-maven-plugin 生成的包之间的直接区别是， fat jar 中主要增加了两部分，第一部分是 lib 目录，存放的是Maven依赖的jar包文件，第二部分是spring boot 类加载器相关的类。

使用 spring-boot-maven-plugin 插件打包出来的结构

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

MANIFEST.MF 的内容 

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190828234021826-291962376.png)

从生成的 MANIFEST.MF 文件中，可以看到两个关键信息 Main-Class 和 Start-Class。说明程序的启动入口并不是我们 SpringBoot 中定义的启动类的 main，而是 JarLauncher#main。

为了不解压就能启动 SpringBoot 程序，在 JarLauncher 内部，会读取 /BOOT-INF/lib/ 下的 jar 文件以及 /BOOT-INF/classes/ 构造一个 URL 数组，并用这个数组来构造 SpringBoot 的自定义类加载器 LaunchedURLClassLoader，该类继承了 java.net.URLClassLoader，其父类加载器是应用程序类加载器。

LaunchedURLClassLoader 创建好之后，会通过反射来启动我们写的启动类中的 main 函数，并设置当前线程上下文类加载器为 LaunchedURLClassLoader。



### 5、Javaagent 类加载器

javaagent 的代码永远都是被应用类加载器( Application ClassLoader)所加载，和应用代码的真实加载器无关。比如，当前运行在 undertow 中的代码是 LaunchedURLClassLoader  加载的，如果启动参数加上 -javaagent，这个 javaagent 还是在 Application ClassLoader 中加载的。



### 6、资料参考

其它一些深入详细的资料可以参考下面的一些文章：

[深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)

[深入理解Java ClassLoader及在 JavaAgent 中的应用](https://juejin.im/post/5c70e1bae51d45125433246f)

[Java类加载机制](https://juejin.im/post/5a810b0e5188257a5c606a85)

[真正理解线程上下文类加载器](https://blog.csdn.net/yangcheng33/article/details/52631940)

[彻底透析SpringBoot jar可执行原理](https://juejin.im/post/5d2d6812e51d45777b1a3e5a)

[spring boot应用启动原理分析](http://hengyunabc.github.io/spring-boot-application-start-analysis/)



## 五、使用 Javassist 扫描类方法

首先我们来看下如何扫描出服务中指定包下的类及方法信息的。由于源码不开放，只贴出部分核心代码逻辑。



### 1、读取资源

要在程序运行期间读取资源文件，可以注入 ResourceLoader 来读取，MetadataReaderFactory 可以用来从 Resource 中读取元数据信息。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class DefaultApiScanService implements ResourceLoaderAware, InitializingBean {
 2 
 3     private ResourceLoader resourceLoader;
 4     private ResourcePatternResolver resolver;
 5     private MetadataReaderFactory metadataReader;
 6 
 7     @Override
 8     public void setResourceLoader(@NotNull ResourceLoader resourceLoader) {
 9         this.resourceLoader = resourceLoader;
10     }
11 
12     @Override
13     public void afterPropertiesSet() throws Exception {
14         Assert.notNull(this.resourceLoader, "resourceLoader should not be null");
15         this.resolver = ResourcePatternUtils.getResourcePatternResolver(resourceLoader);
16         this.metadataReader = new MethodsMetadataReaderFactory(resourceLoader);
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

读取类元数据信息

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 String packagePattern = "org.test.demo.app.service.impl";
2 // 读取资源文件
3 Resource[] resources = resolver.getResources("classpath*:" + packagePattern + "/**/*.class");
4 for (Resource resource : resources) {
5     MetadataReader reader = metadataReader.getMetadataReader(resource);
6     // 读取类元数据信息
7     ClassMetadata classMetadata = reader.getClassMetadata();
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 2、使用 Javassist 解析方法信息

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 创建新的 ClassPool，避免内存溢出
ClassPool classPool = new ClassPool(true);
// 将当前类加载路径加入 ClassPool 的 ClassPath 中，避免找不到类
classPool.insertClassPath(new ClassClassPath(this.getClass()));
// 使用 ClassPool 加载类
CtClass ctClass = classPool.get(classMetadata.getClassName());
// 去除接口、注解、枚举、原生、数组等类型的类，以及代理类不解析
if (ctClass.isInterface() || ctClass.isAnnotation() || ctClass.isEnum() || ctClass.isPrimitive() || ctClass.isArray() || ctClass.getSimpleName().contains("$")) {
    return;
}
// 获取所有声明的方法
CtMethod[] methods = ctClass.getDeclaredMethods();
for (CtMethod method : methods) {
    // 代理方法不解析
    if (method.getName().contains("$")) {
        continue;
    }
    // 包名
    String packageName = ctClass.getPackageName();
    // 类名
    String className = ctClass.getSimpleName();
    // 方法名
    String methodName = method.getName();
    // 参数：method.getLongName() 返回格式：com.test.TestService.selectOrder(java.lang.String,java.util.List,com.test.Order)，所以截取括号中的即可
    String methodSignature = StringUtils.defaultIfBlank(StringUtils.substringBetween(method.getLongName(), "(", ")"), null);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 六、动态编译源码

我们在最开始的规则中，已经维护好了一个业务处理类的源代码，但首先需要将其编译成字节码才能被使用，所以就涉及到如何动态编译源码了。



### 1、Java Compile API

JavaCompiler：表示java编译器，run方法执行编译操作.，还有一种编译方式是先生成编译任务(CompilationTask)，然后调用 CompilationTask 的 call 方法执行编译任务

JavaFileObject：表示一个java源文件对象

JavaFileManager：Java源文件管理类, 管理一系列JavaFileObject

Diagnostic：表示一个诊断信息

DiagnosticListener：诊断信息监听器，编译过程触发

动态编译相关的API在 tools.jar 包里，所以需要在 pom 中引入 tools.jar 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 <dependency>
2     <groupId>jdk.tools</groupId>
3     <artifactId>jdk.tools</artifactId>
4     <version>1.8</version>
5     <scope>system</scope>
6     <systemPath>${env.JAVA_HOME}/lib/tools.jar</systemPath>
7 </dependency>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 2、动态编译

下面这段代码先从源码中解析出包名和类名，并将源码文件写入到磁盘中，然后使用 JavaCompiler 编译源码。注意再次编译同样类名时，名称不能相同，否则编译不通过，因为 JVM 已经加载过此实例了，类名可以加上个随机数避免重复。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void createAndCompileJavaFile(String sourceCode) throws Exception {
 2     // 从源码中解析包名
 3     String packageName = StringUtils.trim(StringUtils.substringBetween(sourceCode, "package", ";"));
 4     // 从源码中解析类名
 5     String className = StringUtils.trim(StringUtils.substringBetween(sourceCode, "class", "implements"));
 6     // 类全名
 7     String classFullName = packageName + "." + className;
 8 
 9     // 将源码写入 java 文件
10     File javaFile = new File(CUSTOMIZE_SRC_DIR + StringUtils.replace(classFullName, ".", File.separator) + ".java");
11     FileUtils.writeByteArrayToFile(javaFile, sourceCode.getBytes());
12 
13     // 使用 JavaCompiler 编译java文件
14     JavaCompiler javac = ToolProvider.getSystemJavaCompiler();
15     // 编译，实际上底层就是调用 javac 命令执行编译工作
16     int result = javac.run(null, null, null, javaFile.getAbsolutePath());
17 
18     if (result != 0) {
19         System.out.println("compile failure.");
20     } else {
21         System.out.println("compile success.");
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 在IDEA中启动服务，这段代码没有任何问题，可以正常编译通过，可以看到 class 文件也编译出来了。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829095529439-1517365641.png)

但是，一旦打成 jar 包运行，就不能正常编译了，会出现如下错误：程序包 xxx 不存在、找不到符号等。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829100630575-836700323.png)

实际上，这个错误也很好理解，javac.run(null, null, null, javaFile.getAbsolutePath()) 这行代码可以看成直接使用 javac 命令编译源文件一样，如果不指定 classpath ，肯定无法找到代码中引用的其它类。

那为何IDEA中可以，jar 包运行就不可以呢？这实际上是因为 springboot jar 的特殊性，springboot jar 是 all-in-one，classes 和 lib 都在 jar 包内，IDEA 中的 classes 都在 target 包下，能够直接被访问到。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829103034116-1010535422.png)



### 3、基于 classpath 编译

如果是这样，那我们可以将 /BOOT-INF/classes/ 以及 /BOOT-INF/lib/ 下的文件加入到编译时的 classpath 路径下就没问题了。

首先，jar 包中的内容无法直接访问，比较次的方法就是将 jar 包解压，然后将路径拼接好之后再编译。

① 解压缩包

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 File file = new File("app.jar");
 2 // 得到 JarFile
 3 JarFile jarFile = new JarFile(file);
 4 
 5 // 解压 jar 包
 6 for (Enumeration<JarEntry> e = jarFile.entries(); e.hasMoreElements();) {
 7     JarEntry je = e.nextElement();
 8     String outFileName = CUSTOMIZE_LIB_DIR + je.getName();
 9     File f = new File(outFileName);
10 
11     if(je.isDirectory()){
12         if (!f.exists()) {
13             f.mkdirs();
14         }
15     } else{
16         File pf = f.getParentFile();
17         if(!pf.exists()){
18             pf.mkdirs();
19         }
20 
21         try (InputStream in = jarFile.getInputStream(je);
22              OutputStream out = new BufferedOutputStream(new FileOutputStream(f))) {
23             byte[] buffer = new byte[2048];
24             int b = 0;
25             while ((b = in.read(buffer)) > 0) {
26                 out.write(buffer, 0, b);
27             }
28             out.flush();
29         }
30     }
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

② 拼接 classpath

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 String bootLib = StringUtils.join(CUSTOMIZE_LIB_DIR, "BOOT-INF", File.separator, "lib");
 2 String bootLibPath = StringUtils.join(bootLib, File.separator);
 3 String bootClasses = StringUtils.join(CUSTOMIZE_LIB_DIR, "BOOT-INF", File.separator, "classes");
 4 
 5 File libDir = new File(bootLib);
 6 File[] libs = libDir.listFiles();
 7 // 拼接 classpath
 8 StringBuilder classpath = new StringBuilder(StringUtils.join(bootClasses, File.pathSeparator));
 9 for (File lib : libs) {
10     classpath.append(bootLibPath).append(lib.getName()).append(File.pathSeparator);
11 }
12 return classpath.toString();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

③ 编译

javac 命令只需通过 -cp 参数指定 classpath 即可，这样就可以编译成功了。

```
1 // 使用 JavaCompiler 编译java文件
2 JavaCompiler javac = ToolProvider.getSystemJavaCompiler();
3 // 编译，实际上底层就是调用 javac 命令执行编译工作
4 int result = javac.run(null, null, null, "-cp", classpath, javaFile.getAbsolutePath());
```



### 4、优雅的动态编译

① Arthas 内存编译

上面的方式需要解压 jar 包得到 classpath，否则无法编译，很不优雅，只能算是一种备选方案。通过参考 Arthas 的源码发现，其中有一个内存编译模块，可以轻松的实现动态编译的能力。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829110939933-1620077198.png)

通过学习它的源码发现，底层还是使用 JavaCompiler 相关的API完成编译工作，不同的是它在获取源码中引用类的方式上。

首先继承 ForwardingJavaFileManager 实现自定义查找 JavaFileObject。然后可以看到它会使用自定义的 PackageInternalsFinder 来查找类，可以看出，它还是会从 jar 包中去查找相关的类。更多的大家可以自行阅读其源码。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829113624389-1771866330.png)

② 使用

首先在 pom 中引入 arthas-memorycompiler 的依赖。

```
1 <dependency>
2     <groupId>com.taobao.arthas</groupId>
3     <artifactId>arthas-memorycompiler</artifactId>
4     <version>3.1.1</version>
5 </dependency>
```

使用方式

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 使用 Arthas 动态编译
 2 DynamicCompiler dynamicCompiler = new DynamicCompiler(Thread.currentThread().getContextClassLoader());
 3 dynamicCompiler.addSource(className, sourceCode);
 4 Map<String, byte[]> byteCodes = dynamicCompiler.buildByteCodes();
 5 
 6 File outputDir = new File(CUSTOMIZE_CLZ_DIR);
 7 
 8 for (Map.Entry<String, byte[]> entry : byteCodes.entrySet()) {
 9     File byteCodeFile = new File(outputDir, StringUtils.replace(entry.getKey(), ".", File.separator) + ".class");
10     FileUtils.writeByteArrayToFile(byteCodeFile, entry.getValue());
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 5、资料参考

[Arthas Github](https://github.com/alibaba/arthas)

[Java 类运行时动态编译技术](https://seanwangjs.github.io/2018/03/13/java-runtime-compile.html)

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 七、代码切入方法

 源代码编译成字节码已经完成了，接下来就看如何切入到要拦截的方法中。



### 1、 加载字节码，定义 Class 实例

首先，需要将字节码加载到 JVM 中，创建 Class 实例这个类才能被使用。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 读取字节码文件
 2 CtClass executeClass = classPool.makeClass(new FileInputStream("..../DemoBeforeHandler.class"));
 3 // 当前上下文类加载器
 4 System.out.println("----> current thread context classLoader : " + Thread.currentThread().getContextClassLoader().toString());
 5 // 当前上下文类加载器的父类加载器
 6 System.out.println("----> current thread context classLoader's parent classLoader : " + Thread.currentThread().getContextClassLoader().getParent().toString());
 7 // 应用程序类加载器
 8 System.out.println("----> application classLoader : " + ClassLoader.getSystemClassLoader().toString());
 9 // 定义 Class 实例
10 Class clazz = executeClass.toClass();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

toClass 不传参数时，其内部实际是使用当前上下文类加载器来加载字节码的，也可以自己传入类加载器。

不同的容器，这个当前上下文类加载器可能不同。我这里使用的是 undertow 容器，上下文类加载器是 LaunchedURLClassLoader；当使用 tomcat 容器时，运行时上下文类加载器是 TomcatEmbeddedWebappClassLoader，其父类加载器是 LaunchedURLClassLoader。在IDEA中运行时，上下文类加载器是 AppClassLoader，即应用程序类加载器。

```
----> current thread context classLoader : org.springframework.boot.loader.LaunchedURLClassLoader@20ad9418
----> current thread context classLoader's parent classLoader : sun.misc.Launcher$AppClassLoader@5c647e05
----> application classLoader : sun.misc.Launcher$AppClassLoader@5c647e05
```

这里有个坑需要注意，在程序启动期间调用时，这里的上下文类加载器是 LaunchedURLClassLoader；但是在运行期间调用时，如果使用 tomcat 容器，这里的上下文类加载器是 TomcatEmbeddedWebappClassLoader，是个代理类加载器。

此时如果使用这个类加载器来定义 Class 实例，能定义成功，但是在后面使用的时候，就会发现报错：NoClassDefFoundError。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190829141159005-365268001.png)

这是因为实际请求时，上下文类加载器是 LaunchedURLClassLoader，是 TomcatEmbeddedWebappClassLoader 的父类加载器，类定义在子类加载器中定义，在父类加载器中使用肯定就找不到咯。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
----> current thread context classLoader : TomcatEmbeddedWebappClassLoader
  context: ROOT
  delegate: true
----------> Parent Classloader:
org.springframework.boot.loader.LaunchedURLClassLoader@20ad9418

----> current thread context classLoader's parent classLoader : org.springframework.boot.loader.LaunchedURLClassLoader@20ad9418
----> application classLoader : sun.misc.Launcher$AppClassLoader@5c647e05
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

因此在调用 toClass 时需要传入 LaunchedURLClassLoader 类加载器，不能使用子类加载器。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 final String LAUNCHED_CLASS_LOADER = "org.springframework.boot.loader.LaunchedURLClassLoader";
 2 // 上下文类加载器
 3 ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
 4 if (!LAUNCHED_CLASS_LOADER.equals(contextClassLoader.getClass().getName())) {
 5     if (LAUNCHED_CLASS_LOADER.equals(contextClassLoader.getParent().getClass().getName())) {
 6         contextClassLoader = contextClassLoader.getParent();
 7     } else {
 8         contextClassLoader = ClassLoader.getSystemClassLoader();
 9     }
10 }
11 // 传入类加载器
12 executeClass.toClass(contextClassLoader, null);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 2、 构建代码块

最简单的方式，就是直接创建 handler 的一个实例对象，然后切入到方法中去使用。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void builderExecuteBody(CtClass executeClass, boolean returnValue) {
 2     StringBuilder executeBody = new StringBuilder("{").append("\r\n");
 3     // 效果：org.test.DemoHandler DemoHandler = new org.test.DemoHandler();
 4     executeBody
 5             .append(executeClass.getName()) // 类型
 6             .append(" ")
 7             .append(executeClass.getSimpleName()) // 变量名称
 8             .append(" = ")
 9             .append("new ").append(executeClass.getName()).append("();")
10             .append("\r\n");
11     // 如果有返回值，则使用临时变量存储
12     if (returnValue) {
13         executeBody.append("Object result = ");
14     }
15     // 效果：DemoHandler.execute($$);
16     executeBody
17             .append(executeClass.getSimpleName()).append(".execute($args);")
18             .append("\r\n");
19     if (returnValue) {
20         executeBody.append("return ($r) result;").append("\r\n");
21     }
22     executeBody.append("}").append("\r\n");
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

生成的效果：

```
1 {
2 org.test.demo.app.service.impl.DemoBeforeHandler DemoBeforeHandler = new org.test.demo.app.service.impl.DemoBeforeHandler();
3 DemoBeforeHandler.execute($args);
4 }
```



### 3、 插入代码块

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 String targetClassName = point.getPackageName() + "." + point.getClassName();
 2 // 目标类
 3 CtClass targetClass = classPool.get(targetClassName);
 4 // 根据参数类型构建 CtClass 数组
 5 CtClass[] params = buildParams(classPool, point);
 6 // 目标方法
 7 CtMethod targetMethod = targetClass.getDeclaredMethod(point.getMethodName(), params);
 8 
 9 // 前置规则
10 String beforeCode = "{...}";
11 
12 // 替换规则
13 String replaceCode = "{...}";
14 
15 // 后置规则
16 String afterCode = "{...}";
17 
18 // 替换方法
19 targetMethod.setBody(replaceCode);
20 
21 // 在方法体前插入代码块
22 targetMethod.insertBefore(beforeCode);
23 
24 // 在方法体后插入代码块
25 targetMethod.insertAfter(afterCode);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 八、动态创建代理程序、实现类重载

方法体已经修改好了，剩下的就是怎么让 JVM 重载这个类，达到动态更改源码的目的。



### 1、Javassist HotSwapAgent

javassist 提供了一个 HotSwapAgent 的代理，可以使用它的 redefine 方法重新定义类，但是这个工具基本无法使用。

```
javassist.util.HotSwapAgent.redefine(Class.forName(targetClassName), targetClass);
```

首先我们来看下 javassist.util.HotSwapAgent 重载类的原理。

在 redefine 方法里面，首先会调用 startAgent 方法动态加载代理程序，然后通过 instrumentation 来重定义类，instrumentation 即 java.lang.instrument.Instrumentation。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831031625196-1254478113.png)

在 startAgent 方法内，首先判断如果 instrumentation 已经有了，就不再加载动态代理程序。如果没有，首先动态创建代理程序 jar 包，然后使用 VirtualMachine attach 到当前虚拟机上，然后加载代理程序。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831034508301-2094473671.png)

在创建代理程序包内，首先创建 MAINFEST 文件，并指定 Premain-Class、Agent-Class 为 javassist.util.HotSwapAgent，接着将 javassist.util.HotSwapAgent.java 的字节码文件写入 javassist.util.HotSwapAgent.class，最终打成 agent.jar 。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831041851670-665894377.png)

HotSwapAgent premain 与 agentmain，代理程序都是为了能够得到 Instrumentation 的实例，这个实例在加载代理程序时由虚拟机传入。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831050309327-1028196339.png)

默认生成的 agent.jar 是在用户目录下一个临时目录下，agent.jar 目录结构如下。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831052140816-1861695762.png)

以上就是动态创建代理程序并加载代理程序的过程，这个功能要是能直接用就完美了，可惜不能。



### 2、IDEA 方式运行

首先我们来看看 IDEA 中使用 javassist.util.HotSwapAgent 的问题，在开始之前，先来看看 IDEA 中启动服务的几种方式。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831091833744-1655009716.png)

**user-local** 和 **none** 是一样的：这是默认选项，这种方式会把所有依赖的 jar 包拼接后通过 -classpath 参数指定，命令行参数会很长。如果命令行参数长度超出了OS限制，会报错：Command line is too long。

**JAR manifest**：将所有依赖的 jar 包拼接好后，创建一个临时的 jar 文件，写入 META-INF/MANIFEST.MF 文件的 Class-Path 参数中，然后通过 -classpath 参数指定这个 jar 包，这样的目的是缩短命令行。

**classpath file**：将所有依赖的 jar 包拼接好后，将其写入一个临时的文件，然后通过 com.intellij.rt.execution.CommandLineWrapper 来启动。

这三种方式的一个区别就是他们启动时的上线文类加载器不一样：

user-local：sun.misc.Launcher$AppClassLoader@18b4aac2，应用程序类加载器

JAR manifest：sun.misc.Launcher$AppClassLoader@18b4aac2，应用程序类加载器

classpath file：java.net.URLClassLoader@7cbd213e，URLClassLoader

之前说过，代理程序在加载时使用的是应用程序类加载器，所以，使用 user-local、JAR manifest 方式启动时，可以正确加载代理程序，他们的类加载器是一样的。如果使用 classpath 启动时，就会报 NoClassDefFoundError。这是因为在 javassist 这个 jar 包的类是由 URLClassLoader 类加载器加载，而应用程序类加载器是加载不到这个lib 的类的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
java.lang.NoClassDefFoundError: javassist/NotFoundException
    at java.lang.Class.getDeclaredMethods0(Native Method)
    at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
    at java.lang.Class.getDeclaredMethod(Class.java:2128)
    at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:327)
    at sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain(InstrumentationImpl.java:411)
Caused by: java.lang.ClassNotFoundException: javassist.NotFoundException
    at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ... 5 more
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 3、JAR 包方式运行

当我们打成 jar 包运行时，实际上还是会报同样的错误，因为 JAR 包运行时上下文类加载器是 org.springframework.boot.loader.LaunchedURLClassLoader@20ad9418，其父类加载是应用程序类加载器。

总结起来就是，根据双亲委派模型，子类加载器可以加载父类加载器中的类；但父类加载器无法加载子类加载器中的类。



### 4、自定义代理程序加载

由于 javassist.util.HotSwapAgent 在加载时使用的是应用程序类加载器，所以代理程序在进入 agentmain 方法设置 instrumentation 变量时，实际是在应用程序类加载器中。

而程序启动时使用的是其子类加载器加载的 HotSwapAgent，所以这里实际上有两个不同的 HotSwapAgent 类实例，虽然类名一样，但是使用的类加载器是不一样的。所以在程序运行期间还是得不到 instrumentation 实例对象。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831161836784-1391437854.png)

我这里用了一个比较简单粗暴的方法解决这个问题：

① 覆盖 javassist.util.HotSwapAgent 方法(我是重写的，将 startAgent 和 agentmain 分开)

② 增加静态的 setInstrumentation 方法

③ 在 agentmain 方法中，得到程序运行时的上下文类加载器(LaunchedURLClassLoader)

④ 通过 LaunchedURLClassLoader 找到程序中加载的 javassist.util.HotSwapAgent 类实例

⑤ 通过反射的方式调用 setInstrumentation 方法将JVM传进来的 Instrumentation 设置进去。

⑥ 之后我们就可以在程序运行期间调用 HotSwapClient.redefine 来重载类了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private static final String CLASS_LOADER = "org.springframework.boot.loader.LaunchedURLClassLoader";
 2 private static final String SWAP_CLIENT = "javassist.util.HotSwapClient";
 3 private static final String SWAP_CLIENT_SETTER = "setInstrumentation";
 4 
 5 // 增加一个静态 setInstrumentation 方法，由 agentmain 设置 Instrumentation
 6 public static void setInstrumentation(Instrumentation instrumentation) {
 7     HotSwapAgent.instrumentation = instrumentation;
 8 }
 9 
10 public static void agentmain(String agentArgs, Instrumentation inst) throws Throwable {
11     if (!inst.isRedefineClassesSupported())
12         throw new RuntimeException("this JVM does not support redefinition of classes");
13 
14     instrumentation = inst;
15 
16     // 得到程序类加载器 LaunchedURLClassLoader
17     ClassLoader classLoader = getClassLoader(inst);
18     // 得到 LaunchedURLClassLoader 类加载器中的 javassist.util.HotSwapClient 类实例
19     Class<?> clientClass = classLoader.loadClass(SWAP_CLIENT);
20     // 通过反射方式设置 Instrumentation
21     clientClass.getMethod(SWAP_CLIENT_SETTER, Instrumentation.class).invoke(null, inst);
22 }
23 
24 private static ClassLoader getClassLoader(Instrumentation inst) {
25     // 获取所有已加载的类
26     Class[] loadedClasses = inst.getAllLoadedClasses();
27     // 找出 LaunchedURLClassLoader
28     return Arrays.stream(loadedClasses)
29             .filter(c -> c.getClassLoader() != null && c.getClassLoader().getClass() != null)
30             .filter(c -> CLASS_LOADER.equals(c.getClassLoader().getClass().getName()))
31             .map(Class::getClassLoader)
32             .findFirst()
33             .orElse(Thread.currentThread().getContextClassLoader());
34 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 九、结果验证及局限性



### 1、结果验证

可以看到已经成功在要拦截的方法前后加入了定制化的代码逻辑了，也可以动态地再次更新代码，再重新应用规则。至此，动态切面的功能基本就实现了。

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190831174326802-695307688.png)



### 2、局限性

① 客制化代码时，由于是创建的一个对象，然后通过方法调用的形式插入方法体中的，所以客制化代码的结构必须固定。

② 客制化代码中，不能使用 @Autowired 等方式直接注入 Spring 容器对象，目前没有处理这种情况。

③ **由于 Instrumentation 本身的局限性，我们只能更改方法体，不能更改方法的定义，不能向类中增加方法、字段，否则重载失败。**

[回到顶部](https://www.cnblogs.com/chiangchou/p/javassist.html#_labelTop)

## 十、附：使用 Arthas 诊断Java问题

在开发这个功能的过程中，简单了解了下 Arthas 的源码原理，以及如何使用 Arthas 来诊断一些线上问题，这里仅列出官方的一些文档，看文档很容易上手。 

Arthas 是 阿里巴巴开源出来的一个针对 java 的工具，主要是针对 java 的问题进行诊断！详细内容可以参考官方文档。

① IDEA 安装插件：[通过Cloud Toolkit插件使用Arthas一键诊断远程服务器](https://github.com/alibaba/arthas/issues/570)

② 使用入门：[Arthas 快速入门](https://alibaba.github.io/arthas/quick-start.html)

③ 命令列表：[Arthas 命令列表](https://alibaba.github.io/arthas/commands.html)

④ Attach 失败：[Attach 时报 ERROR](https://github.com/alibaba/arthas/issues/490)

⑤ Github 源码：[Arthas Github](https://github.com/alibaba/arthas)