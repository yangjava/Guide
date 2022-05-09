# Java 8 新特性

Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本。 Oracle 公司于 2014 年 3 月 18 日发布 Java 8 ，它支持函数式编程，新的 JavaScript 引擎，新的日期 API，新的Stream API 等。

------

## 新特性

Java8 新增了非常多的特性，我们主要讨论以下几个：

- **Lambda 表达式** − Lambda 允许把函数作为一个方法的参数（函数作为参数传递到方法中）。
- **方法引用** − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
- **默认方法** − 默认方法就是一个在接口里面有了一个实现的方法。
- **新工具** − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。
- **Stream API** −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。
- **Date Time API** − 加强对日期与时间的处理。
- **Optional 类** − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。
- **Nashorn, JavaScript 引擎** − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。

## Lambda表达式和函数式接口

Lambda表达式（也称为闭包）是Java 8中最大和最令人期待的语言改变。它允许我们将函数当成参数传递给某个方法，或者把代码本身当作数据处理：函数式开发者非常熟悉这些概念。很多JVM平台上的语言（Groovy、Scala等）从诞生之日就支持Lambda表达式，但是Java开发者没有选择，只能使用匿名内部类代替Lambda表达式。

Lambda的设计耗费了很多时间和很大的社区力量，最终找到一种折中的实现方案，可以实现简洁而紧凑的语言结构。最简单的Lambda表达式可由逗号分隔的参数列表、->符号和语句块组成

### Lambda演变过程

```
@Data
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    //名字
    private String name;
    //性别
    private String sex;
    //薪水
    private int salary;
    //年龄
    private int age;
    //星座
    private String star;
}
```

#### **普通筛选**

将这个集合遍历，然后依次的判断，这是最为普通的一种方式。

```
@Test
public void test1(){
    //首先创建一个
    List<Student> list = Arrays.asList(
            new Student("九天","男",5000,18,"天秤座"),
            new Student("十夜","男",4000,16,"双鱼座"),
            new Student("十一郎","男",3000,24,"水瓶座")
    );

    List<Student> result = new ArrayList<>();
    for (Student student:list){
        if ("天秤座".equals(student.getStar())){
            result.add(student);
        }
    }
    System.out.println(result);
}
```

#### 匿名内部类筛选

通过匿名内部类的方法，在内部类中添加判断条件进行筛选,首先创建一个公共接口：

```
public interface FilterProcess<T> {
    boolean process(T t);
}
```

 接下来通过一个公共函数，对集合以及筛选条件做一个共同方法，筛选到班级里星座是天秤星座的学生

```
public List<Student> filterStudent(List<Student> students, FilterProcess<Student> mp){
    List<Student> list = new ArrayList<>();

    for (Student student : students) {
        if(mp.process(student)){
            list.add(student);
        }
    }
    return list;
}
```

   最后是通过匿名内部类和该方法得到结果：

```
@Test
public void test2(){
    List<Student> students = Arrays.asList(
            new Student("九天","男",5000,18,"天秤座"),
            new Student("十夜","男",4000,16,"双鱼座"),
            new Student("十一郎","男",3000,24,"水瓶座")
    );

    List<Student> list = filterStudent(students, new FilterProcess<Student>() {
        @Override
        public boolean process(Student student) {
            return student.getStar().equals("天秤座");
        }
    });
    for (Student student : list) {
        System.out.println(student);
    }
}
```

结果如图：

```

```

#### 半Lambda方法

   但是通过这两种代码都是很多，所以java8在这一点上提供了对集合筛选最大程度的删减代码，就是第三种方法。第三种方法：通过Lambda直接判断，一步到位，不需要在写其他的方法。

```
@Test
public void test3(){
    List<Student> list = Arrays.asList(
            new Student("九天","男",5000,18,"天秤座"),
            new Student("十夜","男",4000,16,"双鱼座"),
            new Student("十一郎","男",3000,24,"水瓶座")
    );
    List<Student> result = filterStudent(list,(e)->e.getStar().equals("天秤座"));
    System.out.println(result);
}
```

   测试结果：

```

```

   但是现在又会有人会问这个问题，我的那个方法中是这样子的

```
filterStudent(List<Student> students, FilterProcess<Student> mp)
```

   为什么我的代码参数却是这样子的呢

```
filterStudent(list,(e)->e.getStar().equals("天秤座")
```

   其实 -> 这个是一个连接符，左边代表参数，而右边代表函数体（也就是我们说的条件），这个e就是代表 FilterProcess<Student> mp 这个参数的，只不过我们得java8 中lambda可以给这个参数附加上了条件，这些条件筛选都是封装到jdk8中内部类中自己实现的，所以我们只要附加条件就可以了，那个(e)就代表传了参数。

#### 真正运用lambda方法

```
@Test
public void test1() {
    List<Student> list = Arrays.asList(
            new Student("九天","男",5000,18,"天秤座"),
            new Student("十夜","男",4000,16,"双鱼座"),
            new Student("十一郎","男",3000,24,"水瓶座")
    );

    list.stream().filter((e) -> e.getStar().equals("天秤座"))
            .forEach(System.out::println);
}
```

   结果依然是相同的答案，直到第4个方法出来，对比前三个方法，简单了很多，这就是我们lambda演练的过程。

   总结：lambda主要是针对集合中条件的筛选，包括数组等等。接下来我们介绍Stream API ,这个和Lambda息息相关，论重要性，lambda只是基础，Stream API 才是真正的升级版

### StreamAPI详解

#### 功能 

   父类：BasicStream

   子类：Stream、IntStream、LongStream、DoubleStream

   包含两个类型，中间操作(intermediate operations)和结束操作(terminal operations)

   然后准备一个测试类，和一个静态变量，图下：

```
public class JdkTest {

    public static List<Student> list = Arrays.asList(
            new Student("九天", "男", 5000, 18, "天秤座"),
            new Student("十夜", "男", 4000, 16, "双鱼座"),
            new Student("十一郎", "男", 3000, 24, "水瓶座")
    );
}
```

   接下来我们一个一个方法解析他们的作用

##### stream

   将集合转换成流,一般会使用流继续后续操作。

```
@Test
public void test0() {
    list.stream();
}
```

##### forEach遍历

   forEach遍历集合，System.out::println等同于System.out.println()

```
@Test
public void test1() {
    list.forEach(System.out::println);
}
```

   结果为：

##### filter过滤

   该方法中是一个筛选条件，等同于sql查询的where后面的筛选。

```
@Test
public void test2() {
    list.stream().filter((e) -> e.getStar().equals("天秤座"))
            .forEach(System.out::println);
}
```

##### map转换集合

   将List<Student> 转换为List<String>, collect是将结果转换为List

```
@Test
public void test3() {
    List<String> names = list.stream().map(Student::getName).collect(Collectors.toList());
    names.stream().forEach(System.out::println);
}
```

   结果：

##### mapToInt转换数值流

   转换数值流，等同mapToLong、mapToDouble，如下这个是取最大值

```
@Test
public void test4() {
    IntStream intStream = list.stream().mapToInt(Student::getAge);
    Stream<Integer> integerStream = intStream.boxed();
    Optional<Integer> max   = integerStream.max(Integer::compareTo);
    System.out.println(max.get());
}
```

   结果为：

```
24
```

##### flatMap合并成一个流

   将流中的每一个元素 T 映射为一个流，再把每一个流连接成为一个流

```
@Test
public void test5() {
    List<String> list2 = new ArrayList<>();
    list2.add("aaa bbb ccc");
    list2.add("ddd eee fff");
    list2.add("ggg hhh iii");
    list2 = list2.stream().map(s -> s.split(" ")).flatMap(Arrays::stream).collect(Collectors.toList());
    System.out.println(list2);
}
```

   结果为：

```
[aaa, bbb, ccc, ddd, eee, fff, ggg, hhh, iii]
```

### distinct去重

```
@Test
public void test6() {
    List<String> list2 = new ArrayList<>();
    list2.add("aaa bbb ccc");
    list2.add("ddd eee fff");
    list2.add("ggg hhh iii");
    list2.add("ggg hhh iii");

    list2.stream().distinct().forEach(System.out::println);
}
```

   结果：

```
aaa bbb ccc
ddd eee fff
ggg hhh iii
```



### sorted排序

```
@Test
public void test7() {
    //asc排序
    list.stream().sorted(Comparator.comparingInt(Student::getAge)).forEach(System.out::println);
    System.out.println("------------------------------------------------------------------");
    //desc排序
    list.stream().sorted(Comparator.comparingInt(Student::getAge).reversed()).forEach(System.out::println);
}
```

   结果：

```
Student(name=十夜, sex=男, salary=4000, age=16, star=双鱼座)
Student(name=九天, sex=男, salary=5000, age=18, star=天秤座)
Student(name=十一郎, sex=男, salary=3000, age=24, star=水瓶座)
------------------------------------------------------------------
Student(name=十一郎, sex=男, salary=3000, age=24, star=水瓶座)
Student(name=九天, sex=男, salary=5000, age=18, star=天秤座)
Student(name=十夜, sex=男, salary=4000, age=16, star=双鱼座)
```

### skip跳过前n个

```
@Test
public void test8() {
    list.stream().skip(1).forEach(System.out::println);
}
```

### limit截取前n个

```
@Test
public void test10() {
    list.stream().limit(1).forEach(System.out::println);
}
```

   结果为：

```
Student(name=九天, sex=男, salary=5000, age=18, star=天秤座)
```



### anyMatch

   只要有其中任意一个符合条件

```
@Test
public void test11() {
    boolean isHave = list.stream().anyMatch(student -> student.getAge() == 16);
    System.out.println(isHave);
}
```



### allMatch

   全部符合

```
@Test
public void test12() {
    boolean isHave = list.stream().allMatch(student -> student.getAge() == 16);
    System.out.println(isHave);
}
```



### noneMatch

   是否满足没有符合的

```
@Test
public void test13() {
    boolean isHave = list.stream().noneMatch(student -> student.getAge() == 16);
    System.out.println(isHave);
}
```



### findAny

   找到其中一个元素 （使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素）

```
@Test
public void test14() {
    Optional<Student> student = list.stream().findAny();
    System.out.println(student.get());
}
```



### findFirst

   找到第一个元素

```
@Test
public void test15() {
    Optional<Student> student = list.stream().findFirst();
    System.out.println(student.get());
}
```



### count计数

```
@Test
public void test17() {
    long count = list.stream().count();
    System.out.println(count);
}
```



### of

   生成一个字符串流

```
@Test
public void test18() {
    Stream<String> stringStream = Stream.of("i","love","you");
}
```



### empty

   生成一个空流

```
@Test
public void test19() {
    Stream<String> stringStream = Stream.empty();
}
```



### iterate

```
@Test
public void test20() {
    List<String> list = Arrays.asList("a", "b", "c", "c", "d", "f", "a");
    Stream.iterate(0, i -> i + 1).limit(list.size()).forEach(i -> {
        System.out.println(String.valueOf(i) + list.get(i));
    });
}
```

### collect：averagingLong

求平均值

```
@Test
public void test1(){
    // 求年龄平均值
Double average = list.stream().collect(Collectors.averagingLong(Student::getAge));
}
```

### collect：collectingAndThen

两步结束，先如何，在如何

```
@Test
public void test1(){
    // 求年龄平均值
String average = list.stream().collect(Collectors.collectingAndThen(Collectors.averagingInt(Student::getAge), a->"哈哈，平均年龄"+a));
System.out.println(average);
}
```

结果：

```
哈哈，平均年龄20.5
```



### collect：counting

求个数

```
@Test
public void test1(){
    // 求数量
Long num = list.stream().collect(Collectors.counting());
System.out.println(num);
}
```



### collect: groupingBy(Function)

接下来所有的都是用下面的新List数据测试使用

```
public static List<Student> list = Arrays.asList(
        new Student("九天", "男", 5000, 18, "天秤座"),
        new Student("十夜", "男", 4000, 16, "双鱼座"),
        new Student("十一郎", "男", 3000, 24, "水瓶座"),
        new Student("十二郎", "男", 3000, 24, "水瓶座")
);
@Test
public void test1(){
    Map<Integer,List<Student>> result = list.stream().collect(Collectors.groupingBy(Student::getAge));
    for (Integer age:result.keySet()){
        System.out.println(result.get(age));
    }
}
```

结果：

```
[Student(name=十夜, sex=男, salary=4000, age=16, star=双鱼座)]
[Student(name=九天, sex=男, salary=5000, age=18, star=天秤座)]
[Student(name=十一郎, sex=男, salary=3000, age=24, star=水瓶座), Student(name=十二郎, sex=男, salary=3000, age=24, star=水瓶座)]
```



### collect：groupingBy(Function,Collector)

```
@Test
public void test1(){
    // 先分组，在计算每组的个数
Map<Integer,Long> num = list.stream().collect(Collectors.groupingBy(Student::getAge,Collectors.counting()));
System.out.println(num);
}
```

结果：

```
{16=1, 18=1, 24=2}
```



### collect：groupingBy(Function, Supplier, Collector)

```
@Test
public void test1(){
    // 先分组，在计算每组的个数,然后排序
Map<Integer,Long> num = list.stream().collect(Collectors.groupingBy(Student::getAge, TreeMap::new,Collectors.counting()));
System.out.println(num);
}
```



### collect：groupingByConcurrent

同上，不过这个Concurrent是并发的，也有3个方法，和上面非并发一个效果

```
groupingByConcurrent(Function)

groupingByConcurrent(Function, Collector)

groupingByConcurrent(Function, Supplier, Collector)
```



### collect：joining()

```
@Test
public void test1(){
    // 名字拼接
String result = list.stream().map(Student::getName).collect(Collectors.joining());
System.out.println(result);
}
```

结果：

```
九天十夜十一郎十二郎
```



### collect：joining(str)

```
@Test
public void test1(){
    // 名字拼接,用逗号隔开
String result = list.stream().map(Student::getName).collect(Collectors.joining(","));
System.out.println(result);
}
```

结果：

```
九天,十夜,十一郎,十二郎
```



### collect：joining(str, prefix, suffix)

```
@Test
public void test1(){
    // 名字拼接,包含前缀、后缀
String result = list.stream().map(Student::getName).collect(Collectors.joining(",","hello","world"));
System.out.println(result);
}
```

结果：

```
hello九天,十夜,十一郎,十二郎world
```



### collect：summarizingDouble

```
@Test
public void test1(){
    // 求年龄的最大值、最小值、平均值、综合以及人数
DoubleSummaryStatistics result = list.stream().collect(Collectors.summarizingDouble(Student::getAge));
System.out.println(result);
}
```

结果：

```
DoubleSummaryStatistics{count=4, sum=82.000000, min=16.000000, average=20.500000, max=24.000000}
```



### collect：toCollection

有很多如

## Date Time API

### JDK7 Date缺点

```
1、所有的日期类都是可变的，因此他们都不是线程安全的，这是Java日期类最大的问题之一

2、Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义

3、java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。对于时间、时间戳、格式化以及解析，并没有一些明确定义的类。对于格式化和解析的需求，我们有java.text.DateFormat抽象类，但通常情况下，SimpleDateFormat类被用于此类需求

4、日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题
```

### JDK8 Date优势

```
1、不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程环境有好处。

2、关注点分离：新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。

3、清晰：在所有的类中，方法都被明确定义用以完成相同的行为。举个例子，要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。

4、实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。

5、可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。
```

### JDK8 Date新增字段

   Java.time包中的是类是不可变且线程安全的。新的时间及日期API位于java.time中，java8 time包下关键字段解读。

| 属性          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| Instant       | 代表的是时间戳                                               |
| LocalDate     | 代表日期，比如2020-01-14                                     |
| LocalTime     | 代表时刻，比如12:59:59                                       |
| LocalDateTime | 代表具体时间 2020-01-12 12:22:26                             |
| ZonedDateTime | 代表一个包含时区的完整的日期时间，偏移量是以UTC/  格林威治时间为基准的 |
| Period        | 代表时间段                                                   |
| ZoneOffset    | 代表时区偏移量，比如：+8:00                                  |
| Clock         | 代表时钟，比如获取目前美国纽约的时间                         |



### 获取当前时间

```
Instant instant = Instant.now(); //获取当前时间戳

LocalDate localDate = LocalDate.now();  //获取当前日期

LocalTime localTime = LocalTime.now();  //获取当前时刻

LocalDateTime localDateTime = LocalDateTime.now();  //获取当前具体时间

ZonedDateTime zonedDateTime = ZonedDateTime.now();   //获取带有时区的时间
```

### 字符串转换

```
jdk8：
String str = "2019-01-11";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate localDate = LocalDate.parse(str, formatter);

jdk7:
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
try {
    Date date = simpleDateFormat.parse(str); 
} catch (ParseException e){ 
    e.printStackTrace();
}
```



### Date转换LocalDate

```
import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;
import java.util.Date;

public class Test {

    public static void main(String[] args) {
        Date date = new Date();
        Instant instant = date.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();

        // atZone()方法返回在指定时区从此Instant生成的ZonedDateTime。
        LocalDate localDate = instant.atZone(zoneId).toLocalDate();
        System.out.println("Date = " + date);
        System.out.println("LocalDate = " + localDate);
    }
}
```



### LocalDate转Date

```
import java.time.LocalDate;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.util.Date;

public class Test {

    public static void main(String[] args) {
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDate localDate = LocalDate.now();
        ZonedDateTime zdt = localDate.atStartOfDay(zoneId);

        Date date = Date.from(zdt.toInstant());

        System.out.println("LocalDate = " + localDate);
        System.out.println("Date = " + date);

    }
}
```



### 时间戳转LocalDateTime

```
long timestamp = System.currentTimeMillis();

Instant instant = Instant.ofEpochMilli(timestamp);

LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
```



### LocalDateTime转时间戳

```
LocalDateTime dateTime = LocalDateTime.now();

dateTime.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();

dateTime.toInstant(ZoneOffset.of("+08:00")).toEpochMilli();

dateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
```



### LocalDate方法总结

```
getYear()                         int        获取当前日期的年份
getMonth()                        Month      获取当前日期的月份对象
getMonthValue()                   int        获取当前日期是第几月
getDayOfWeek()                    DayOfWeek  表示该对象表示的日期是星期几
getDayOfMonth()                   int        表示该对象表示的日期是这个月第几天
getDayOfYear()                    int        表示该对象表示的日期是今年第几天
withYear(int year)                LocalDate  修改当前对象的年份
withMonth(int month)              LocalDate  修改当前对象的月份
withDayOfMonth(intdayOfMonth)     LocalDate  修改当前对象在当月的日期
isLeapYear()                      boolean    是否是闰年
lengthOfMonth()                   int        这个月有多少天
lengthOfYear()                    int        该对象表示的年份有多少天（365或者366）
plusYears(longyearsToAdd)         LocalDate  当前对象增加指定的年份数
plusMonths(longmonthsToAdd)       LocalDate  当前对象增加指定的月份数
plusWeeks(longweeksToAdd)         LocalDate  当前对象增加指定的周数
plusDays(longdaysToAdd)           LocalDate  当前对象增加指定的天数
minusYears(longyearsToSubtract)   LocalDate  当前对象减去指定的年数
minusMonths(longmonthsToSubtract) LocalDate  当前对象减去注定的月数
minusWeeks(longweeksToSubtract)   LocalDate  当前对象减去指定的周数
minusDays(longdaysToSubtract)     LocalDate  当前对象减去指定的天数
compareTo(ChronoLocalDateother)   int        比较当前对象和other对象在时间上的大小，返回值如果为正，则当前对象时间较晚，
isBefore(ChronoLocalDateother)    boolean    比较当前对象日期是否在other对象日期之前
isAfter(ChronoLocalDateother)     boolean    比较当前对象日期是否在other对象日期之后
isEqual(ChronoLocalDateother)     boolean    比较两个日期对象是否相等
```



# Java 9 新特性

Java 9 发布于 2017 年 9 月 22 日，带来了很多新特性，其中最主要的变化是已经实现的模块化系统。接下来我们会详细介绍 Java 9 的新特性。

## Java 9 新特性

- **模块系统**：模块是一个包的容器，Java 9 最大的变化之一是引入了模块系统（Jigsaw 项目）。
- **REPL (JShell)**：交互式编程环境。
- **HTTP 2 客户端**：HTTP/2标准是HTTP协议的最新版本，新的 HTTPClient API 支持 WebSocket 和 HTTP2 流以及服务器推送特性。
- **改进的 Javadoc**：Javadoc 现在支持在 API 文档中的进行搜索。另外，Javadoc 的输出现在符合兼容 HTML5 标准。
- **多版本兼容 JAR 包**：多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本。
- **集合工厂方法**：List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例。
- **私有接口方法**：在接口中使用private私有方法。我们可以使用 private 访问修饰符在接口中编写私有方法。
- **进程 API**: 改进的 API 来控制和管理操作系统进程。引进 java.lang.ProcessHandle 及其嵌套接口 Info 来让开发者逃离时常因为要获取一个本地进程的 PID 而不得不使用本地代码的窘境。
- **改进的 Stream API**：改进的 Stream API 添加了一些便利的方法，使流处理更容易，并使用收集器编写复杂的查询。
- **改进 try-with-resources**：如果你已经有一个资源是 final 或等效于 final 变量,您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。
- **改进的弃用注解 @Deprecated**：注解 @Deprecated 可以标记 Java API 状态，可以表示被标记的 API 将会被移除，或者已经破坏。
- **改进钻石操作符(Diamond Operator)** ：匿名类可以使用钻石操作符(Diamond Operator)。
- **改进 Optional 类**：java.util.Optional 添加了很多新的有用方法，Optional 可以直接转为 stream。
- **多分辨率图像 API**：定义多分辨率图像API，开发者可以很容易的操作和展示不同分辨率的图像了。
- **改进的 CompletableFuture API** ： CompletableFuture 类的异步机制可以在 ProcessHandle.onExit 方法退出时执行操作。
- **轻量级的 JSON API**：内置了一个轻量级的JSON API
- **响应式流（Reactive Streams) API**: Java 9中引入了新的响应式流 API 来支持 Java 9 中的响应式编程。

更多的新特性可以参阅官网：[What's New in JDK 9](https://docs.oracle.com/javase/9/whatsnew/toc.htm)

JDK 9 下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk9-doc-downloads-3850606.html

在关于 Java 9 文章的实例，我们均使用 jdk 1.9 环境，你可以使用以下命令查看当前 jdk 的版本：

```
$ java -version
java version "9-ea"
Java(TM) SE Runtime Environment (build 9-ea+163)
Java HotSpot(TM) 64-Bit Server VM (build 9-ea+163, mixed mode)
```

接下来我们将详细为大家简介 Java 9 的新特性：

| 序号 | 特性                                                         |
| :--- | :----------------------------------------------------------- |
| 1    | [模块系统](https://www.runoob.com/java/java9-module-system.html) |
| 2    | [REPL (JShell)](https://www.runoob.com/java/java9-repl.html) |
| 3    | [改进的 Javadoc](https://www.runoob.com/java/java9-improved-javadocs.html) |
| 4    | [多版本兼容 JAR 包](https://www.runoob.com/java/java9-multirelease-jar.html) |
| 5    | [集合工厂方法](https://www.runoob.com/java/java9-collection-factory-methods.html) |
| 6    | [私有接口方法](https://www.runoob.com/java/java9-private-interface-methods.html) |
| 7    | [进程 API](https://www.runoob.com/java/java9-process-api-improvements.html) |
| 8    | [Stream API](https://www.runoob.com/java/java9-stream-api-improvements.html) |
| 9    | [try-with-resources](https://www.runoob.com/java/java9-try-with-resources-improvement.html) |
| 10   | [@Deprecated](https://www.runoob.com/java/java9-enhanced-deprecated-annotation.html) |
| 11   | [内部类的钻石操作符(Diamond Operator)](https://www.runoob.com/java/java9-inner-class-diamond-operator.html) |
| 12   | [Optional 类](https://www.runoob.com/java/java9-optional-class-improvements.html) |
| 13   | [多分辨率图像 API](https://www.runoob.com/java/java9-multiresolution-image_api.html) |
| 14   | [CompletableFuture API](https://www.runoob.com/java/java9-completablefuture-api-improvements.html) |
