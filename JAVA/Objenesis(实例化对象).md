### `objenesis简介：`

```
objenesis是一个小型Java类库用来实例化一个特定class的对象。
```

### `使用场合：`

```
Java已经支持使用Class.newInstance()动态实例化类的实例。但是类必须拥有一个合适的构造器。有很多场景下不能使用这种方式实例化类，比如：
```

- `构造器需要参数`
- `构造器有side effects`
- `构造器会抛异常`

```
因此，在类库中经常会有类必须拥有一个默认构造器的限制。Objenesis通过绕开对象实例构造器来克服这个限制。
```

### `典型使用`

```
实例化一个对象而不调用构造器是一个特殊的任务，然而在一些特定的场合是有用的：
```

- `序列化，远程调用和持久化 -对象需要实例化并存储为到一个特殊的状态，而没有调用代码。`
- `代理，AOP库和Mock对象 -类可以被子类继承而子类不用担心父类的构造器`
- `容器框架 -对象可以以非标准的方式被动态实例化。`

### `快速入门`

```
Objenesis有两个主要的接口：
ObjectInstantiator -实例化一个类的多个实例
interface ObjectInstantiator{



    Object new Instance();



}



  123



  123
```

InstantiatorStrategy -对一个类的实例化进行特殊的策略处理（随着类的不同而不同）

```csharp
interface InstantiatorStrategy{



    ObjectInstantiator newInstantiaorOf(Class type);    



}123123
```

注意：所有的Objenesis类都在org.objenesis包中

### 使用方法

根据jvm供应商，版本和类的安全管理和类型的不同, Objenesis实例化对象有许多不同的策略。 
有两种不同种类的实例化是必需的：

- Stardard -没有构造器会被调用
- Serilizable compliant -与java标准序列化方式实例一个对象的行为一致。即第一个不可序列化的父类构造器将被调用。 但是，readResolve不会被调用并且不检查对象是否是可序列化的。

最简单的使用Objenesis的方法是使用ObjenesisStd(Standard) 和ObjenesisSerializer(Serializable compliant)。这两种方式会自动选择最好的策略。

```csharp
Objenesis objenesis = new ObjenesisStd(); // or ObjenesisSerialializer11
```

实现Objenesis接口后，就可以为一个特定的类型创建一个ObjectInstantiator

```python
ObjectInstantiator thingInstantiator = objenesis.getInstantiatorOf(MyThingy.class);11
```

最后，可以使用这个去新建实例

```java
MyThingy thingy1 = (MyThingy)thingyInstantiator.newInstnace();



MyThingy thingy2 = (MyThingy)thingInstantiator.newInstance();1212
```

### 性能和多线程

为了提高性能，最好尽可能多的使用ObjectInstantiator 对象。 例如，如果要实例化一个类的多个对象，请使用相同的ObjectInstantiator。 
InstantiatorStrategy和ObjectInstantiator都可以在多线程中共享并发使用，它们是线程安全的。

### 代码示例：

```ruby
Objenesis objenesis = new ObjenesisStd(); // or ObjenesisSerializer



MyThingy thingy1 = (MyThingy) objenesis.newInstance(MyThingy.class);



 



// or (a little bit more efficient if you need to create many objects)



 



Objenesis objenesis = new ObjenesisStd(); // or ObjenesisSerializer



ObjectInstantiator thingyInstantiator = objenesis.getInstantiatorOf(MyThingy.class);



 



MyThingy thingy2 = (MyThingy)thingyInstantiator.newInstance();



MyThingy thingy3 = (MyThingy)thingyInstantiator.newInstance();



MyThingy thingy4 = (MyThingy)thingyInstantiator.newInstance();12345678910111234567891011
```

（译自 http://objenesis.org/）