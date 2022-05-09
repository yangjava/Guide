Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库，例如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。

Guava 是Java的工具集，提供了一些常用的便利的操作工具类，减少因为 空指针、异步操作等引起的问题BUG，提高开发效率。

Google Guava中包含大概七大模块，分别如下:

1. Guava Utils:在Guava中封装了很多关于字符串，join，split，断言等工具，可以极大的方便我们在开发中进行使用
2. Functional Programming：在JDK8 以前，Java对函数式编程几乎没有任何支持，Guava提供了一系列的函数式编程接口，可以很方便的使用函数式（陈述式）编写优雅灵活的代码
3. Collections： 相比较Java的Collections以及Apache Commons的Collections，Guava的Collections显然要强大很多，在Google Guava中支持了几乎你能想到的任何数据结构  ，这对对程序员来说无路是使用，还是研习代码都有很大的裨益
4. Concurrency： 在Guava中对并发编程也提供了不少的支持，比如Monitor（类似于条件锁），支持回调的Future接口，异步函数接口以及RateLimte，使用RateLimte我们可以很容易的实现令牌桶，漏桶等高并发算法
5. Guava Cache: Guava的Cache功能同样非常强大，通过Google Guava我们可以轻而易举的实现基于JVM进程级别的Cache功能
6. EventBus：事件总线，是一个非常好的程序解耦合解决方案，使用EventBus，就像使用消息中间件一样，让Event的消费者只专注于Event本身
7. Guava IO：在Guava中提供了很多source，sink，encoding工具集，可以很方便的操作文件，以及字节流



```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>28.0-jre</version>
</dependency>
```

# 1 使用和避免null

> 大多数情况下，使用null表明的是某种缺失情况
>
> Guava引入 Optional<T>表明可能为null的T类型引用，Optional实例可能包含非null的引用（引用存在），也可能什么也不包括（引用缺失）
>
> 正是受到Guava的启发，Java**8**将Optional类做为一个新特性引入进Java8的类库

## 1.1 学习Java8中的Optional使用方法



```css
import java.util.Optional;
```



```tsx
/** * 三种创建Optional对象方式 */
// 1.创建空的Optional对象Optional<Object> optional1 = Optional.empty();
// 2.使用非null值创建Optional对象Optional<String> optional2 = Optional.of("蔡徐坤");
// 3.使用任意值创建Optional对象Optional optional3 = Optional.ofNullable("乔碧萝");

Optional optional = Optional.ofNullable("蔡徐坤");/** * 判断是否引用缺失的方法(建议不直接使用) */boolean isNull = optional.isPresent();/** * 当optional引用存在时执行 * 类似的方法：map filter flatMap */optional.ifPresent(System.out::println);

Optional optional = Optional.empty();/** * 当optional引用缺失时执行 */// 1.引用缺失时的替换值Object obj1 = optional.orElse("引用缺失");System.out.println(obj1); // 2.自定义引用缺失时的返回值Object obj2 = optional.orElseGet(() -> {    return "自定义引用缺失";});System.out.println(obj2); // 3.抛出异常Object obj3 = optional.orElseThrow(() -> {    throw new RuntimeException("引用缺失异常");});

public static void stream(List<String> list) {    // list.stream().forEach(System.out::println);    Optional.ofNullable(list)            .map(List::stream)            //创建一个空的流            .orElseGet(Stream::empty)            .forEach(System.out::println); }public static void main(String[] args) {    List<String> list = new ArrayList(){{        add("蔡徐坤");        add("乔碧萝");        add("卢本伟");    }};    stream(list);    //stream(null);}

@Dataclass User {    private String id;    private String name;    public User(){}    public User(String id,String name){this.id=id;this.name=name;}} public class OptionalTest {    public static void main(String[] args) {        User user = new User();        user.setName("蔡徐坤");        user  = null;        String name = Optional.ofNullable(user).map(User::getName).orElse("默认值");        System.out.println(name);    }}
```

------

# 2 不可变集合

> - 创建对象的不可变拷贝是一项很好的防御性编程技巧
> - Guava为所有JDK标准集合类型和Guava新集合类型都提供了简单易用的不可变版本

## 2.1 不可变对象的优点

> 1. 当对象被不可信的库调用时，不可变形式是安全的
> 2. 不可变对象被多个线程调用时，不存在竞态条件问题
> 3. 不可变集合不需要考虑变化，因此可以节省时间和空间
> 4. 不可变对象因为有固定不变，可以作为常量来安全使用

## 2.2 JDK提供的unmodifiableXxx方法

> 1. 笨重而且累赘
> 2. 不安全
> 3. 低效



```csharp
public static void test(List<Integer> list) {    list.remove(0);}public static void main(String[] args) {    List<Integer> list = new ArrayList<Integer>() {{        add(1);        add(2);        add(3);    }};    //创建不可变集合    List<Integer> newList = Collections.unmodifiableList(list);    test(newList);    System.out.println(newList);}
```

> Exception in thread "main" java.lang.UnsupportedOperationException
>  at java.util.Collections$UnmodifiableList.remove(Collections.java:1317)
>  at com.imooc.zhangxiaoxi.guava.ImmutableTest.test(ImmutableTest.java:16)
>  at com.imooc.zhangxiaoxi.guava.ImmutableTest.main(ImmutableTest.java:26)

## 2.3 Guava构造不可变集合对象



```csharp
import com.google.common.collect.ImmutableSet;import com.google.common.collect.Sets;

List<Integer> list = new ArrayList<Integer>() {{    add(1);    add(2);    add(3);}};
// 1.通过已经存在的集合创建ImmutableSet<Integer> immutable = ImmutableSet.copyOf(list);

// 2.通过初始值，直接创建不可变集合
ImmutableSet immutableSet = ImmutableSet.of(1, 2, 3);

// 3.以builder方式创建
ImmutableSet.builder()        .add(1)        .addAll(Sets.newHashSet(2, 3))        .add(4)        .build();

List<Integer> list = new ArrayList<Integer>() {{    
add(1);   
 add(2);   
 add(3);}};
ImmutableSet<Integer> immutableList = ImmutableSet.copyOf(list);
//immutableList里面的添加删除方法时不支持的
for (Integer num : immutableList) {    System.out.println(num);}
```

------

# 3 新集合类型

> Guava引入了很多JDK没有的、但明显有用的新集合类型。这些新类型是为了和JDK集合框架共存，而没有往JDK集合抽象中硬塞其他概念。

## 3.1 Multiset

![img](https:////upload-images.jianshu.io/upload_images/23166491-f7e3eb28fa73224e?imageMogr2/auto-orient/strip|imageView2/2/w/883/format/webp)

image

### 3.1.1 Multiset两种视角

> 没有元素顺序限制的ArrayList(E)

![img](https:////upload-images.jianshu.io/upload_images/23166491-eccfc7790c93ca31.png?imageMogr2/auto-orient/strip|imageView2/2/w/777/format/webp)

image

> Map<E, Integer> ，键为元素，值为计数

![img](https:////upload-images.jianshu.io/upload_images/23166491-c0a1577a0bd2b1c4.png?imageMogr2/auto-orient/strip|imageView2/2/w/762/format/webp)

image

### 3.1.2 Multiset与Map的区别

![img](https:////upload-images.jianshu.io/upload_images/23166491-c3a7fb3dc32fdd59?imageMogr2/auto-orient/strip|imageView2/2/w/687/format/webp)

image

### 3.1.3 多种Multiset的实现

![img](https:////upload-images.jianshu.io/upload_images/23166491-da4dc57eb95d0894?imageMogr2/auto-orient/strip|imageView2/2/w/716/format/webp)

image

### 3.1.4 Coding



```java
import com.google.common.collect.HashMultiset;import com.google.common.collect.Multiset;import com.google.common.primitives.Chars;import org.junit.Test;import java.util.Arrays;/** * 实现：使用Multiset统计一首古诗的文字出现频率 */public class MultisetTest {    private static final String text =            "《南陵别儿童入京》" +                    "白酒新熟山中归，黄鸡啄黍秋正肥。" +                    "呼童烹鸡酌白酒，儿女嬉笑牵人衣。" +                    "高歌取醉欲自慰，起舞落日争光辉。" +                    "游说万乘苦不早，著鞭跨马涉远道。" +                    "会稽愚妇轻买臣，余亦辞家西入秦。" +                    "仰天大笑出门去，我辈岂是蓬蒿人。";    @Test    public void handle() {        // multiset创建        Multiset<Character> multiset = HashMultiset.create();        // string 转换成 char 数组        char[] chars = text.toCharArray();        System.out.println(Arrays.toString(chars));        // 遍历数组，添加到multiset中        //multiset.addAll(Chars.asList(chars));        Chars.asList(chars)                .stream()                .forEach(charItem -> {                    multiset.add(charItem);                });        System.out.println("size : " + multiset.size());        System.out.println("count : " + multiset.count('人'));    }}
```

------

# 4 集合工具类



```css
import com.alibaba.fastjson.JSON;import com.google.common.collect.Lists;import com.google.common.collect.Sets;import org.junit.Test;
```



```dart
/** * Lists / Sets 使用 */public class SetsTest {    /**     * Sets工具类的常用方法     * 并集 / 交集 / 差集 / 分解集合中的所有子集 / 求两个集合的笛卡尔积     *     * Lists工具类的常用方式     * 反转 / 拆分     */    private static final Set set1 = Sets.newHashSet(1, 2);    private static final Set set2 = Sets.newHashSet(4);
```

## 4.1 Sets的使用

### 4.1.1 并集



```bash
// 并集@Testpublic void union() {    Set<Integer> set = Sets.union(set1, set2);    //[1, 2, 4]    System.out.println(set);}
```

### 4.1.2 交集



```bash
// 交集@Testpublic void intersection() {    Set<Integer> set = Sets.intersection(set1, set2);    //[]    System.out.println(set);}
```

### 4.1.3 差集



```bash
@Testpublic void difference() {    // 差集：如果元素属于A而且不属于B    Set<Integer> set = Sets.difference(set1, set2);    //[1, 2]    System.out.println(set);     // 相对差集：属于A而且不属于B 或者 属于B而且不属于A    // 两个集合之和-两个集合的交集    set = Sets.symmetricDifference(set1, set2);    //[1, 2, 4]    System.out.println(set);}
```

### 4.1.4 拆分所有子集合



```cpp
// 拆分所有子集合@Testpublic void powerSet() {    Set<Set<Integer>> powerSet = Sets.powerSet(set1);    //[[],[1],[2],[1,2]]    System.out.println(JSON.toJSONString(powerSet));}
```

### 4.1.5 计算两个集合笛卡尔积



```cpp
// 计算两个集合笛卡尔积@Testpublic void cartesianProduct() {    Set<List<Integer>> product = Sets.cartesianProduct(set1, set2);    //[[1,4],[2,4]]    System.out.println(JSON.toJSONString(product));}
```

## 4.2 Lists的使用

### 4.2.1 拆分



```cpp
/** * 拆分 */@Testpublic void partition() {    List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7);    //拆分成3个3个一组    List<List<Integer>> partition = Lists.partition(list, 3);    //[[1,2,3],[4,5,6],[7]]    System.out.println(JSON.toJSONString(partition));}
```

### 4.2.2 反转



```xml
// 反转@Testpublic void reverse() {    List<Integer> list = Lists.newLinkedList();    list.add(1);    list.add(2);    list.add(3);    List<Integer> newList = Lists.reverse(list);    //[3, 2, 1]    System.out.println(newList);}
```

------

# 5 I/O

# 5.1 对字节流/字符流提供的工具方法

![img](https:////upload-images.jianshu.io/upload_images/23166491-d8c9325ff739feb5.png?imageMogr2/auto-orient/strip|imageView2/2/w/970/format/webp)

image

> 需要手动关闭流

## 5.2 对源（Source）与汇（Sink）的抽象

![img](https:////upload-images.jianshu.io/upload_images/23166491-873e80c74f12d45a.png?imageMogr2/auto-orient/strip|imageView2/2/w/713/format/webp)

image

> 不需要手动关闭流

## 5.3 Coding



```css
import com.google.common.base.Charsets;import com.google.common.io.CharSink;import com.google.common.io.CharSource;import com.google.common.io.Files;import org.junit.Test;import java.io.File;import java.io.IOException;
```



```cpp
/** * 演示如何使用流(Source)与汇(Sink)来对文件进行常用操作 */public class IOTest {    @Test    public void copyFile() throws IOException {        /**         * 创建对应的Source和Sink         */        CharSource charSource = Files.asCharSource(new File("SourceText.txt"), Charsets.UTF_8);        CharSink charSink = Files.asCharSink(new File("TargetText.txt"), Charsets.UTF_8);        /**         * 拷贝         */        charSource.copyTo(charSink);    }}
```

------

# 6 异常工具类

## 6.1 获得字符串类型的堆栈信息



```csharp
public class ExceptionStackTest {    public static void main(String[] args) {        try {            controller();        } catch (Throwable e) {            String stackTrace = Throwables.getStackTraceAsString(e);            System.out.println(stackTrace);        }    }    private static void controller() {        service();    }    private static void service() {        dao();    }    private static void dao() {        int a = 1 / 0;    }}//java.lang.ArithmeticException: / by zero//   at cn.yuanyu.guava.ExceptionStackTest.dao(ExceptionStackTest.java:24)// at cn.yuanyu.guava.ExceptionStackTest.service(ExceptionStackTest.java:21)// at cn.yuanyu.guava.ExceptionStackTest.controller(ExceptionStackTest.java:18)//  at cn.yuanyu.guava.ExceptionStackTest.main(ExceptionStackTest.java:11)
```



5人点赞



[Java]()





作者：梅西爱骑车
链接：https://www.jianshu.com/p/d78705dcc513
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。