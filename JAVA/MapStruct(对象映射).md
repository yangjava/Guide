#### 前言

在我们日常开发的分层结构的应用程序中，为了各层之间互相解耦，一般都会定义不同的对象用来在不同层之间传递数据，因此，就有了各种 `XXXDTO`、`XXXVO`、`XXXBO` 等基于数据库对象派生出来的对象，当在不同层之间传输数据时，不可避免地经常需要将这些对象进行相互转换。

此时一般处理两种处理方式：① 直接使用 `Setter` 和 `Getter` 方法转换、② 使用一些工具类进行转换（e.g. `BeanUtil.copyProperties`）。第一种方式如果对象属性比较多时，需要写很多的 `Getter/Setter` 代码。第二种方式看起来虽然比第一种方式要简单很多，但是因为其使用了反射，性能不太好，而且在使用中也有很多陷阱。而今天要介绍的主角 [MapStruct](https://mapstruct.org/) 在不影响性能的情况下，同时解决了这两种方式存在的缺点。



#### MapStruct 是什么

`MapStruct` 是一个代码生成器，它基于**约定优于配置**方法极大地简化了 `Java bean` 类型之间映射的实现。自动生成的映射转换代码只使用简单的方法调用，因此速度快、类型安全而且易于理解阅读，源码仓库 `Github` 地址 [MapStruct](https://github.com/mapstruct/mapstruct)。总的来说，有如下三个特点：

1. 基于注解
2. 在编译期自动生成映射转换代码
3. 类型安全、高性能、无依赖性

#### MapStruct 使用步骤

`MapStruct` 的使用比较简单，只需如下三步即可。

###### ① 引入依赖（这里以 `Gradle` 方式为例）

复制

```
dependencies {
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
}
```

###### ② 创建相关转换对象

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Doctor {

  private Integer id;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class DoctorDTO {

  private Integer id;

  private String name;
}
```

###### ③ 创建转换器类（Mapper）

需要注意的是，转换器不一定都要使用 `Mapper` 作为结尾，只是官方示例推荐以 `XXXMapper` 格式命名转换器名称，这里举例的是最简单的映射情况（字段名称和类型都完全匹配），只需要在转换器类上添加 `@Mapper` 注解即可，转换器代码如下所示：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Mapper
public interface DoctorMapper {

  DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

  DoctorDTO toDTO(Doctor doctor);
}
```

通过下面这个简单的测试来校验转换结果是否正确，测试代码如下：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
public class DoctorTest {

  @Test
  public void testToDTO() {
    Integer doctorId = 9527;
    String doctorName = "mghio";

    Doctor doctor = new Doctor();
    doctor.setId(doctorId);
    doctor.setName(doctorName);

    DoctorDTO doctorDTO = DoctorMapper.INSTANCE.toDTO(doctor);

    assertEquals(doctorId, doctorDTO.getId());
    assertEquals(doctorName, doctorDTO.getName());
  }

}
```

测试结果正常通过，说明使用 `DoctorMapper` 转换器达到我们的预期结果。

[![1.png](https://i.loli.net/2021/08/08/hMJpBU93vbkIqKG.png)](https://i.loli.net/2021/08/08/hMJpBU93vbkIqKG.png)

#### MapStruct 实现浅析

在以上示例中，使用 `MapStruct` 通过简单的三步就实现了 `Doctor` 到 `DoctorDTO` 的转换，那么，`MapStruct` 是如何做到的呢？其实通过我们定义的转换器可以发现，转换器是接口类型的，而我们知道在 `Java` 中，接口是无法提供功能的，只是定义规范，具体干活的还是它的实现类。

因此我们可以大胆猜想，`MapStruct` 肯定给我们定义的转换器接口（`DoctorMapper`）生成了实现类，而通过 `Mappers.getMapper(DoctorMapper.class)` 获取到的转换器实际上是获取到了转化器接口的实现类。下面通过在测试类中 `debug` 来验证一下：

[![2.png](https://i.loli.net/2021/08/08/hbI7SAVXcN18xWR.png)](https://i.loli.net/2021/08/08/hbI7SAVXcN18xWR.png)

通过 `debug` 可以看出，`DoctorMapper.INSTANCE` 获取到的是接口的实现类 `DoctorMapperImpl`。这个转换器接口实现类是在**编译期**自动生成的，`Gradle` 项目是在 `build/generated/sources/anotationProcessor/Java` 下（`Maven` 项目在 `target/generated-sources/annotations` 目录下），生成以上示例转换器接口的实现类源码如下：

[![4.png](https://i.loli.net/2021/08/08/7q4Slnz8ETCsdx2.png)](https://i.loli.net/2021/08/08/7q4Slnz8ETCsdx2.png)

可以发现，自动生成的代码和我们平时手写的差不多，简单易懂，代码完全在编译期间生成，没有运行时依赖。和使用反射的实现方式相比还有一个有点就是，出错时很容易去 `debug` 实现源码来定位，而反射相对来说定位问题就要困难得多了。

#### 常见使用场景介绍

###### ① 对象属性名称和类型完全相同

从上文的示例可以看出，当属性名称和类型完全一致时，我们只需要定义一个转换器接口并添加 `@Mapper` 注解即可，然后 `MapStruct` 会自动生成实现类完成转换。示例代码如下:

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Source {

  private Integer id;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Target {

  private Integer id;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Mapper
public interface SourceMapper {

  SourceMapper INSTANCE = Mappers.getMapper(SourceMapper.class);

  Target toTarget(Source source);
}
```

###### ② 对象属性类型相同但是名称不同

当对象属性类型相同但是属性名称不一样时，通过 `@Mapping` 注解来手动指定转换。示例代码如下：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Source {

  private Integer id;

  private String sourceName;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Target {

  private Integer id;

  private String targetName;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Mapper
public interface SourceMapper {

  SourceMapper INSTANCE = Mappers.getMapper(SourceMapper.class);

  @Mapping(source = "sourceName", target = "targetName")
  Target toTarget(Source source);
}
```

###### ③ 在 Mapper 中使用自定义转换方法

有时候，对于某些类型（比如：一个类的属性是自定义的类），无法以自动生成代码的形式进行处理。此时我们需要自定义类型转换的方法，在 `JDK 7` 之前的版本，就需要使用抽象类来定义转换 `Mapper` 了，在 `JDK 8` 以上的版本可以使用接口的**默认方法**来自定义类型转换的方法。示例代码如下：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Source {

  private Integer id;

  private String sourceName;

  private InnerSource innerSource;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class InnerSource {

  private Integer deleted;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Target {

  private Integer id;

  private String targetName;

  private InnerTarget innerTarget;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class InnerTarget {

  private Boolean isDeleted;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Mapper
public interface SourceMapper {

  SourceMapper INSTANCE = Mappers.getMapper(SourceMapper.class);

  @Mapping(source = "sourceName", target = "targetName")
  Target toTarget(Source source);

  default InnerTarget innerTarget2InnerSource(InnerSource innerSource) {
    if (innerSource == null) {
      return null;
    }
    InnerTarget innerTarget = new InnerTarget();
    innerTarget.setIsDeleted(innerSource.getDeleted() == 1);
    innerTarget.setName(innerSource.getName());
    return innerTarget;
  }
}
```

###### ④ 多个对象转换成一个对象返回

在一些实际业务编码的过程中，不可避免地需要将多个对象转化为一个对象的场景，`MapStruct` 也能很好的支持，对于这种最终返回信息来源于多个类，我们可以通过配置来实现多对一的转换。示例代码如下：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Data
public class Doctor {

  private Integer id;

  private String name;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-09
 */
@Data
public class Address {

  private String street;

  private Integer zipCode;
}
```

复制

```
/**
 * @author mghio
 * @since 2021-08-09
 */
@Mapper
public interface AddressMapper {

  AddressMapper INSTANCE = Mappers.getMapper(AddressMapper.class);

  @Mapping(source = "doctor.id", target = "personId")
  @Mapping(source = "address.street", target = "streetDesc")
  DeliveryAddressDTO doctorAndAddress2DeliveryAddressDTO(Doctor doctor, Address address);
}
```

从这个示例中的转换器（`AddressMapper`）可以看出，当属性名称和类型完全匹配时同样可以自动转换，但是当来源对象有多个属性名称及类型完全和目标对象相同时，还是需要手动配置指定的，因为此时 `MapStruct` 也无法准确判断应该使用哪个属性转换。

#### 获取转换器（Mapper）的几种方式

获取转换器的方式根据 `@Mapper` 注解的 `componentModel` 属性不同而不同，支持以下四种不同的取值：

1. **default** 默认方式，默认方式，使用工厂方式（`Mappers.getMapper(Class)`）来获取
2. **cdi** 此时生成的映射器是一个应用程序范围的 `CDI bean`，使用 `@Inject` 注解来获取
3. **spring** `Spring` 的方式，可以通过 `@Autowired` 注解来获取，在 `Spring` 框架中推荐使用此方式
4. **jsr330** 生成的映射器用 `@javax.inject.Named` 和 `@Singleton` 注解，通过 `@Inject` 来获取

###### ① 通过工厂方式获取

上文的示例中都是通过工厂方式获取的，也就是使用 `MapStruct` 提供的 `Mappers.getMapper(Class<T> clazz)` 方法来获取指定类型的 `Mapper`。然后在调用的时候就不需要反复创建对象了，方法的最终实现是通过我们定义接口的类加载器加载 `MapStruct` 生成的实现类（类名称规则为：接口名称 + `Impl`），然后调用该类的无参构造器创建对象。核心源码如下所示：

[![5.png](https://i.loli.net/2021/08/09/5CwVsSo3bUNyF7B.png)](https://i.loli.net/2021/08/09/5CwVsSo3bUNyF7B.png)

###### ② 使用依赖注入方式获取

对于依赖注入（`dependency injection`），使用 `Spring` 框架开发的朋友们应该很熟悉了，工作中经常使用。`MapStruct` 也支持依赖注入的使用方式，并且官方也推荐使用依赖注入的方式获取。使用 `Spring` 依赖注入的方式只需要指定 `@Mapper` 注解的 `componentModel = "spring"` 即可，示例代码如下：

复制

```
/**
 * @author mghio
 * @since 2021-08-08
 */
@Mapper(componentModel = "spring")
public interface SourceMapper {

  SourceMapper INSTANCE = Mappers.getMapper(SourceMapper.class);

  @Mapping(source = "sourceName", target = "targetName")
  Target toTarget(Source source);

}
```

我们可以使用 `@Autowired` 获取的原因是 `SourceMapper` 接口的实现类已经被注册为容器中一个 `Bean` 了，通过如下生成的接口实现类的代码也可以看到，在类上自动加上了 `@Component` 注解。

[![6.png](https://i.loli.net/2021/08/09/FCGxQvZUWXeHm5h.png)](https://i.loli.net/2021/08/09/FCGxQvZUWXeHm5h.png)

最后还有两个注意事项：① 当两个转换对象的属性不一致时（比如 `DoctorDTO` 中不存在 `Doctor` 对象中的某个字段），编译时会出现警告提示。可以在`@Mapping` 注解中配置 `ignore = true`，或者当不一致字段比较多时，可以直接设置 `@Mapper` 注解的 `unmappedTargetPolicy` 属性或`unmappedSourcePolicy` 属性设置为 `ReportingPolicy.IGNORE`。② 如果你项目中也使用了 [Lombok](https://projectlombok.org/)，需要注意一下 `Lombok` 的版本至少是 `1.18.10` 或者以上才行，否则会出现编译失败的情况。刚开始用的时候我也踩到这个坑了。。。

#### 总结

本文介绍了对象转换工具 `Mapstruct` 库，以安全优雅的方式来减少我们的转换代码。从文中的示例中可以看出，`Mapstruct` 提供了大量的功能和配置，使我们能够以简单快捷的方式创建从简单到复杂的映射器。文中所介绍到的只是 `Mapstruct` 库的冰山一角，还有很多强大的功能文中没有提到，感兴趣的朋友可以自行查看 [官方使用指南](https://mapstruct.org/documentation/stable/reference/html)。