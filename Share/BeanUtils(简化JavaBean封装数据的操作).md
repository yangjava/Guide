

# BeanUtils手册

## 简介

### 概述

BeanUtils 是 Apache commons组件的成员之一，主要用于简化JavaBean封装数据的操作。它可以给JavaBean封装一个字符串数据，也可以将一个表单提交的所有数据封装到JavaBean中。

### 包

BeanUtils一共分4个包：

> org.apache.commons.beanutils
>
> org.apache.commons.beanutils.converters
>
> org.apache.commons.beanutils.locale
>
> org.apache.commons.beanutils.locale.converters

其中需要我们特别关注的是这个**org.apache.commons.beanutils**包，其他包都是起辅助作用的。其中上面两个没有针对本地化的任何处理，执行效率高。

## 入门

### 配置

Maven

```xml
<dependency>
	<groupId>commons-beanutils</groupId>
	<artifactId>commons-beanutils</artifactId>
	<version>1.8.3</version>
</dependency>
```

### 常用API

```
// 把orig对象copy到dest对象中.
public void copyProperties (Object dest, Object orig)

// 把Bean的属性值放入到一个Map里面
public Map describe(Object bean)

// 把map里面的值放入bean中
public void populate (Object bean, Map map)

// 设置Bean对象中名称为name的属性值赋值为value.	
public void setProperty(Object bean,String name, Object value)

// 取得bean对象中名为name的属性的值
public String getProperty(Object bean, String name)	
```

### 代码示例

用户模型

```
package com.demo.beanutils;

import lombok.Data;
import lombok.ToString;
@Data
@ToString
public class User {

    private Long id;

    private String name;

    private Integer age;


}

```

用户表单模型

```
package com.demo.beanutils;

import lombok.Data;
import lombok.ToString;

import java.util.Date;

@Data
@ToString
public class UserForm {

    private Long id;

    private String name;

    private String desc;
}

```

#### 同类之间不同对象要求进行数据复制

```java
package com.demo.beanutils;

import org.apache.commons.beanutils.BeanUtils;

public class HelloWorld {


    public static void main(String[] args) throws Exception {
        //同类之间不同对象要求进行数据复制
        User user1 = new User();
        user1.setId(1L);
        user1.setName("张三");
        user1.setAge(18);
        User user2 = new User();
        BeanUtils.copyProperties(user2, user1);
        System.out.println(user2.toString());
    }

}

```

注意: BeanUtils.copyProperties(p,d);

p是等待被赋值的对象,d是源对象,将d中属性值赋值的p中对应的字段,d中有的属性p中必须有,p可以有更多属性

#### 不同类不同对象之间的数据复制

```
        //不同类不同对象之间的数据复制
        UserForm userForm=new UserForm();
        BeanUtils.copyProperties(userForm, user1);
        System.out.println(userForm.toString());
```

注意: 如果目标类存在的属性会赋值,不存在的会出现null

结果如下

```
UserForm(id=1, name=张三, desc=null)
```

#### 对象转Map

```
        // 将对象转换为Map
        Map map = BeanUtils.describe(user1);
        System.out.println(map);
```

#### Map转对象

```
        // 将Map转换为对象
        User userInfo = new User();
        BeanUtils.populate(userInfo,map);
        System.out.println(userInfo);
```

#### 拷贝指定属性

```
        //拷贝指定属性
        User user3 = new User();
        BeanUtils.copyProperty(user3, "name", "李四");
        System.out.println(user3);
```

结果如下

```
User(id=null, name=李四, age=null)
```

#### 设置指定属性

```
        //设置指定的属性
        User user4 = new User();
        BeanUtils.setProperty(user4, "name", "王五");
        System.out.println(user4);
```

#### 克隆对象

```
        //克隆对象
        Object object = BeanUtils.cloneBean(user1);
        System.out.println(object);
```

#### 获取List中的对象

```
        //获取List中的对象
        List<UserForm> list = new ArrayList<>();
        list.add(userForm);
        user1.setUserForms(list);
        BeanUtils.getProperty(user1, "userForms[0].name");
        System.out.println("获取List中的对象==>" + BeanUtils.getProperty(user1, "userForms[0].name"));
```

## 核心

### BeanUtils

主要提供了对于JavaBean进行各种操作，比如对象,属性复制等等。

BeanUtils设置属性值的时候，如果属性是基本数据类型，那么BeanUtils会自动帮我们进行数据类型的转换，并且BeanUtils设置属性的时候也是依赖于底层的getter和setter方法。如果设置的属性值是其他的引用数据类型，此时必须要注册一个类型转换器才能实现自动的转换（参考下面的ConvertUtils）

| 主要方法                                                     | 描述                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| cloneBean(Object bean)                                       | 对象的克隆                                                   |
| copyProperties(Object dest, Object orig)                     | 对象的拷贝                                                   |
| copyProperty(Object bean, String name, Object value)         | 拷贝指定的属性值到指定的bean                                 |
| setProperty(Object bean, String name, Object value)          | 设置指定属性的值                                             |
| populate(Object bean, Map<String,? extends Object> properties) | 将map数据拷贝到javabean中（map的key要与javabean的属性名称一致） |

> 注：copyProperty与setProperty，日常使用时推荐copyProperty。如果我们需要自定义实现populate()方法,那么我们可以override setProperty()方法.

### ConvertUtils

这个工具类的职能是在字符串和指定类型的实例之间进行转换。 实际上，BeanUtils是依赖ConvertUtils来完成类型转换，但是有时候我们可能需要自定义转换器来完成特殊需求的类型转换；

| 主要方法                                   | 描述                               |
| ------------------------------------------ | ---------------------------------- |
| convert(Object value, Class<?> targetType) | 将给定的value转换成指定的Class类型 |

示例代码

```
package com.demo.beanutils;

import org.apache.commons.beanutils.ConvertUtils;

import java.util.Date;

public class ConvertDemo {

    public static void main(String[] args) {
        // 将整型转换成字符串
        Object object = ConvertUtils.convert(123, String.class);
        String typeName = object.getClass().getTypeName();
        System.out.println(typeName);

        // 将日期转换成字符串
        Object object2 = ConvertUtils.convert(new Date(), String.class);
        String typeName2 = object2.getClass().getTypeName();
        System.out.println(typeName2);

        // 将字符串转换成Double
        Object object3 = ConvertUtils.convert("123", Double.class);
        String typeName3 = object3.getClass().getTypeName();
        System.out.println(typeName3);
    }
}

```

#### 自定义类型转换：

自定义转换器，实现Converter接口，重写**convert**方法

```
package com.demo.beanutils;

import org.apache.commons.beanutils.ConversionException;
import org.apache.commons.beanutils.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;

class MyConverter implements Converter {
    private static SimpleDateFormat format;

    public MyConverter(String pattern) {
        format = new SimpleDateFormat(pattern);
    }

    @Override
    public Object convert(Class type, Object value) {
        if (value == null) {
            return null;
        }

        if (value instanceof String) {
            String tmp = (String) value;
            if (tmp.trim().length() == 0) {
                return null;
            } else {
                try {
                    return format.parse(tmp);
                } catch (ParseException e) {
                    e.printStackTrace();
                }
            }
        } else {
            throw new ConversionException("not String");
        }
        return value;
    }
}

```

使用自定义类型转换器

```
package com.demo.beanutils;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.commons.beanutils.ConvertUtils;

import java.util.Date;

public class MyConvertDemo {

    public static void main(String[] args) throws Exception {

        MyConverter converter = new MyConverter("yyyy-MM-dd HH:mm:ss");
        // 注册该转换器
        ConvertUtils.register(converter, Date.class);
        Object object3 = ConvertUtils.convert("2019-03-13 14:04:00", Date.class);
        System.out.println(object3.getClass().getTypeName());
        System.out.println(object3);


        // BeanUtils设置属性时，自动进行类型转换
        MyConverter myConverter = new MyConverter("yyyy-MM-dd HH:mm:ss");
        // 注册该转换器
        ConvertUtils.register(myConverter, Date.class);
        UserDate userDate = new UserDate();
        BeanUtils.copyProperty(userDate, "birthDay", "2019-03-13 14:04:00");
        System.out.println(userDate);
    }
}

```

### PropertyUtils

BeanUtils与PropertyUtils这两个类几乎有一摸一样的功能，唯一的区别是：BeanUtils在对Bean赋值是会进行类型转化。
举例来说也就是在copyProperty时只要属性名相同，就算类型不同，BeanUtils也可以进行copy；而PropertyBean则可能会报错！！当然2个Bean之间的同名属性的类型必须是可以转化的，否则用BeanUtils一样会报错。若实现了org.apache.commons.beanutils.Converter接口则可以自定义类型之间的转化。由于不做类型转化，用PropertyUtils在速度上会有很大提高！

```
        // 获取集合
        String userForms0 = BeanUtils.getIndexedProperty(user1, "userForms", 0);
        String userForm0 = BeanUtils.getIndexedProperty(user1, "userForms[0]");
        System.out.println("userForms0==>"+userForms0);
        System.out.println("userForm0==>"+userForm0);
```

### BeanCompartor 

还是通过反射，动态设定Bean按照哪个属性来排序，而不再需要在实现bean的Compare接口进行复杂的条件判断。

```
 List<UserForm> list = new ArrayList<>();
 Collections.sort(list, new BeanComparator("age"));
```

如果要支持多个属性的复合排序，如"Order By lastName,firstName"

```
ArrayList sortFields = new ArrayList();
sortFields.add(new BeanComparator("lastName"));
sortFields.add(new BeanComparator("firstName"));
ComparatorChain multiSort = new ComparatorChain(sortFields);
Collections.sort(rows,multiSort);
```

其中ComparatorChain属于jakata commons-collections包。
如果age属性不是普通类型，构造函数需要再传入一个comparator对象为age变量排序。
另外, BeanCompartor本身的ComparebleComparator, 遇到属性为null就会抛出异常, 也不能设定升序还是降序。这个时候又要借助commons-collections包的ComparatorUtils.

```
   Comparator mycmp = ComparableComparator.getInstance();
   mycmp = ComparatorUtils.nullLowComparator(mycmp);  //允许null
   mycmp = ComparatorUtils.reversedComparator(mycmp); //逆序
   Comparator cmp = new BeanComparator(sortColumn, mycmp);
```

### ConstructorUtils

动态创建对象

```
public static Object invokeConstructor(Class klass, Object arg)
```

### MethodUtils

动态调用方法

```
MethodUtils.invokeMethod(bean, methodName, parameter);
```

# 附录

现在不建议使用`beanutils`进行属性赋值，因为由于源码中采用反射机制，性能太低

BeanUtils的copyProperties()方法的确存在着浅拷贝的情况，持久化操作实体类的时候是很大的一个坑。

建议使用`Spring BeanUtils`或者`Cglib BeanCopier`。

BeanUtils的copyProperties()方法并不是完全的深度克隆，在包含有引用类型的对象拷贝上就可能会出现引用对象指向同一个的情况，且该方法的性能低下，项目中一定要谨慎使用。

要实现高性能且安全的深度克隆方法还是实现Serializable接口,多层克隆时，引用类型均要实现Serializable接口。