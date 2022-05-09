JMH简介
JMH即Java Microbenchmark Harness，是Java用来做基准测试的一个工具，该工具由OpenJDK提供并维护，测试结果可信度高。

基准测试Benchmark是测量、评估软件性能指标的一种测试，对某个特定目标场景的某项性能指标进行定量的和可对比的测试。

项目中添加依赖
创建一个基准测试项目，在项目中引入JMH的jar包，目前JMH的最新版本为1.23。以maven为例，依赖配置如下。

<dependencies>
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-core</artifactId>
        <version>1.23</version>
    </dependency>

    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-generator-annprocess</artifactId>
        <version>1.23</version>
    </dependency>
</dependencies>
复制代码
另外，我们也可以直接用在需要进行基准测试的项目中，以单元测试方式使用。

注解方式使用
在运行时，注解配置被用于解析生成BenchmarkListEntry配置类实例。

一个方法对应一个@Benchmark注解，一个@Benchmark注解对应一个基准测试方法。

注释在类上的注解，或者注释在类的字段上的注解，则是类中所有基准测试方法共用的配置。

@Benchmark
声明一个public方法为基准测试方法。

public class JsonBenchmark {
    
    @Benchmark
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@BenchmarkMode
通过JMH我们可以轻松的测试出某个接口的吞吐量、平均执行时间等指标的数据。

假设我想测试testGson方法的平均耗时，那么可以使用@BenchmarkMode注解指定测试维度为Mode.AverageTime。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@Measurement
假设我想测量testGson方法五次，那么可以使用@Measurement注解。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@Measurement注解有三个配置项：

iterations：测量次数；
time与timeUnit：每次测量的持续时间，timeUnit指定时间单位，本例中：每次测量持续1秒，1秒内执行的testGson方法的次数是不固定的，由方法执行耗时和time决定。
@Warmup
为了数据准确，我们可能需要让testGson方法做下热身运动。如在方法中创建GsonParser对象，预热可以避免首次创建GsonParser时因多了类加载的耗时而导致测试结果不准备的情况。jvm使用JIT即时编译器，一定的预热次数可让JIT对testGson方法的调用链路完成编译，去掉解释执行对测试结果的影响。

@Warmup注解用于配置预热参数。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@Warmup注解有三个配置项：

iterations：预热次数；
time与timeUnit：每次预热的持续时间，timeUnit指定时间单位。
假设@Measurement指定iterations为100，time为10s，则： 每个线程实际执行基准测试方法的次数等于time除以基准测试方法单次执行的耗时，假设基准测试方法执行耗时为1s,那么一次测量最多只执行10（time为10s / 方法执行耗时1s）次基准测试方法，而iterations为100指的是测试100次（不是执行100次基准测试方法）。

@OutputTimeUnit
OutputTimeUnit注解用于指定输出的方法执行耗时的单位。如果方法执行耗时为秒级别，为了便于 观察结果，我们可以使用@OutputTimeUnit指定输出的耗时时间单位为秒；如果方法执行耗时为毫秒级别，为了便于观察结果，我们可以使用@OutputTimeUnit指定输出的耗时时间单位为毫秒，否则使用默认的秒做单位，会输出10的负几次方这样的数字，不太直观。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @OutputTimeUnit(TimeUnit.NANOSECONDS) // 指定输出的耗时时长的单位
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@Fork
@Fork用于指定fork出多少个子进程来执行同一基准测试方法。假设我们不需要多个进程，那么 可以使用@Fork指定为进程数为1。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @Fork(1)
    @OutputTimeUnit(TimeUnit.NANOSECONDS) // 指定输出的耗时时长的单位
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
@Threads
@Threads注解用于指定使用多少个线程来执行基准测试方法，如果使用@Threads指定线程数为2，那么每次测量都会创建两个线程来执行基准测试方法。

public class JsonBenchmark {
    
    @BenchmarkMode(Mode.AverageTime) // 指定mode为Mode.AverageTime
    @Benchmark
    @Fork(1)
    @Threads(2)
    @OutputTimeUnit(TimeUnit.NANOSECONDS) // 指定输出的耗时时长的单位
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Test // 因为这是一个单元测试类，所以多了一个@Test注解，可以忽略
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }

}
复制代码
如果@Measurement注解指定time为1s，基准测试方法的执行耗时为1s，那么如果只使用单个线程，一次测量只会执行一次基准测试方法，如果使用10个线程，一次测量就能执行10次基准测试方法。

公共注解
假设我们需要在JsonBenchmark类中创建两个基准测试方法，一个是testGson，另一个是testJackson，用于对比Gson与Jackson这两个框架解析json的性能。那么我们可以将除@Benchmark注解外的其它注解都声明到类上，让两个基准测试方法都使用同样的配置。

@BenchmarkMode(Mode.AverageTime)
@Fork(1)
@Threads(2)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
public class JsonBenchmark {

    @Benchmark
    @Test
    public void testGson() {
        new GsonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
     
    @Benchmark
    @Test
    public void testJackson() {
        new JacksonParser().fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
}
复制代码
如果不想每执行一次方法都创建一个GsonParser或JacksonParser实例，我们可以将GsonParser与JacksonParser对象声明为JsonBenchmark的字段。（GsonParser与JacksonParser是笔者封装的工具类，配合设计模式使用，为项目提供随时切换解析框架的功能）。

@BenchmarkMode(Mode.AverageTime)
@Fork(1)
@Threads(2)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@State(Scope.Benchmark)
public class JsonBenchmark {

     private GsonParser gsonParser = new GsonParser();
     private JacksonParser jacksonParser = new JacksonParser();
     
    @Benchmark
    @Test
    public void testGson() {
        gsonParser.fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
     
    @Benchmark
    @Test
    public void testJackson() {
        jacksonParser.fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
}
复制代码
还需要使用@State注解指定字段的共享域。在本例中，我们使用@Threads注解声明创建两个线程来执行基准测试方法，假设我们配置@State(Scope.Thread)，那么在不同线程中，gsonParser、jacksonParser这两个字段都是不同的实例。

以testGson方法为例，我们可以认为JMH会为每个线程克隆出一个gsonParser对象。如果在testGson方法中打印gsonParser对象的hashCode，你会发现，相同线程打印的结果相同，不同线程打印的结果不同。例如：

@BenchmarkMode(Mode.AverageTime)
@Fork(1)
@Threads(2)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@State(Scope.Benchmark)
public class JsonBenchmark {

     private GsonParser gsonParser = new GsonParser();
     private JacksonParser jacksonParser = new JacksonParser();
     
    @Benchmark
    @Test
    public void testGson() {
        System.out.println("current Thread:" + Thread.currentThread().getName() + "==>" + gsonParser.hashCode());
        gsonParser.fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
     
    @Benchmark
    @Test
    public void testJackson() {
        jacksonParser.fromJson("{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}", JsonTestModel.class);
    }
}
复制代码
执行testGson方法输出的结果如下：

current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-1==>2063684770
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-2==>1629232880
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-1==>2063684770
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-2==>1629232880
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-1==>2063684770
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-2==>1629232880
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-1==>2063684770
current Thread:com.msyc.common.JsonBenchmark.testGson-jmh-worker-2==>1629232880
......
复制代码
@Param
使用@Param注解可指定基准方法执行参数，@Param注解只能指定String类型的值，可以是一个数组，参数值将在运行期间按给定顺序遍历。假设@Param注解指定了多个参数值，那么JMH会为每个参数值进行一次测量。

例如，我们想测试不同复杂度的json字符串使用Gson框架与使用Jackson框架解析的性能对比，代码如下。

@BenchmarkMode(Mode.AverageTime)
@Fork(1)
@Threads(2)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@State(Scope.Thread)
public class JsonBenchmark {

    private GsonParser gsonParser = new GsonParser();
    private JacksonParser jacksonParser = new JacksonParser();
     
    // 指定参数有三个值
    @Param(value = 
                {"{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 13:00:00\",\"flag\":true,\"threads\":5,\"shardingIndex\":0}",
                "{\"startDate\":\"2020-04-01 16:00:00\",\"endDate\":\"2020-05-20 14:00:00\"}",
                "{\"flag\":true,\"threads\":5,\"shardingIndex\":0}"})
    private String jsonStr;
     
    @Benchmark
    @Test
    public void testGson() {
        gsonParser.fromJson(jsonStr, JsonTestModel.class);
    }
     
    @Benchmark
    @Test
    public void testJackson() {
        jacksonParser.fromJson(jsonStr, JsonTestModel.class);
    }

}
复制代码
测试结果如下：

Benchmark                                                                                                                      (jsonStr)  Mode  Cnt      Score       Error  Units
JsonBenchmark.testGson     {"startDate":"2020-04-01 16:00:00","endDate":"2020-05-20 13:00:00","flag":true,"threads":5,"shardingIndex":0}  avgt    5  12180.763 ±  2481.973  ns/op
JsonBenchmark.testGson                                               {"startDate":"2020-04-01 16:00:00","endDate":"2020-05-20 14:00:00"}  avgt    5   8154.709 ±  3393.881  ns/op
JsonBenchmark.testGson                                                                       {"flag":true,"threads":5,"shardingIndex":0}  avgt    5   9994.171 ±  5737.958  ns/op
JsonBenchmark.testJackson  {"startDate":"2020-04-01 16:00:00","endDate":"2020-05-20 13:00:00","flag":true,"threads":5,"shardingIndex":0}  avgt    5  15663.060 ±  9042.672  ns/op
JsonBenchmark.testJackson                                            {"startDate":"2020-04-01 16:00:00","endDate":"2020-05-20 14:00:00"}  avgt    5  13776.828 ± 11006.412  ns/op
JsonBenchmark.testJackson                                                                    {"flag":true,"threads":5,"shardingIndex":0}  avgt    5   9824.283 ±   311.555  ns/op
复制代码
非注解使用
使用注解与不使用注解其实都是一样，只不过使用注解更加方便。在运行时，注解配置被用于解析生成BenchmarkListEntry配置类实例，而在代码中使用Options配置也是被解析成一个个BenchmarkListEntry配置类实例（每个方法对应一个）。

非注解方式我们可以使用OptionsBuilder构造一个Options，例如，非注解方式实现上面的例子。

public class BenchmarkTest{

    @Test
    public void test() throws RunnerException {
        Options options = new OptionsBuilder()
                .include(JsonBenchmark.class.getSimpleName())
                .exclude("testJackson")
                .forks(1)
                .threads(2)
                .timeUnit(TimeUnit.NANOSECONDS)
                .warmupIterations(5)
                .warmupTime(TimeValue.seconds(1))
                .measurementIterations(5)
                .measurementTime(TimeValue.seconds(1))
                .mode(Mode.AverageTime)
                .build();
        new Runner(options).run();
    }

}
复制代码
include：导入一个基准测试类。调用方法传递的是类的简单名称，不含包名。
exclude：排除哪些方法。默认JMH会为include导入的类的每个public方法都生成一个BenchmarkListEntry配置类实例，也就是把每个public方法都当成是基准测试方法，这时我们就可以使用exclude排除不需要参与基准测试的方法。例如本例中使用exclude排除了testJackson方法。
打jar包放服务器上执行
对于大型的测试，需要测试时间比较久、线程比较多的情况，我们可以将写好的基准测试项目打包成jar包丢到linux服务器上执行。对于吞吐量基准测试，建议放到服务器上执行，其结果会更准确一些，硬件、系统贴近线上环境、也不受本机开启的应用数、硬件配置等因素影响。

java -jar my-benchmarks.jar
复制代码
在IDEA中执行
对于一般的方法执行耗时测试，我们不需要把测试放到服务器上执行，例如测试对比几个json解析框架的性能。

在idea中，我们可以编写一个单元测试方法，在单元测试方法中创建一个org.openjdk.jmh.runner.Runner，调用Runner的run方法执行基准测试。但JMH不会去扫描包，不会执行每个基准测试方法，这需要我们通过配置项来告知JMH需要执行哪些基准测试方法。

public class BenchmarkTest{

    @Test
    public void test() throws RunnerException {
        Options options = null; // 创建Options
        new Runner(options).run();
    }

}
复制代码
完整例子如下：

public class BenchmarkTest{
     @Test
     public void test() throws RunnerException {
        Options options = new OptionsBuilder()
                 .include(JsonBenchmark.class.getSimpleName())
                 // .output("/tmp/json_benchmark.log")
                 .build();
        new Runner(options).run();
     }
}
复制代码
Options在前面已经介绍过了，由于本例中JsonBenchmark这个类已经使用了注解，因此Options只需要配置需要执行基准测试的类。如果需要执行多个基准测试类，include方法可以多次调用。

如果需要将测试结果输出到文件，可调用output方法配置文件路径，不配置则输出到控制台。

在IDEA中使用插件JMH Plugin执行
插件源码地址：https://github.com/artyushov/idea-jmh-plugin。

安装：在IDEA中搜索JMH Plugin，安装后重启即可使用。

1、只执行单个Benchmark方法
在方法名称所在行，IDEA会有一个▶️执行符号，右键点击运行即可。如果写的是单元测试方法， IDEA会提示你选择执行单元测试还是基准测试。

2、执行一个类中的所有Benchmark方法
在类名所在行，IDEA会有一个▶️执行符号，右键点击运行，该类下的所有被@Benchmark注解注释的方法都会执行。如果写的是单元测试方法，IDEA会提示你选择执行单元测试还是基准测试。

官方提供的JMH使用例子
官方的demo：
http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/
复制代码
翻译的demo：
https://github.com/Childe-Chen/goodGoodStudy/tree/master/src/main/java/com/cxd/benchmark
复制代码
 更多参考：

https://dunwu.github.io/javatech/test/jmh.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-jmh
