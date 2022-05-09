# spring之RestTemplate

一、RestTemplate 语法
二、配置类的使用
步骤一：编写配置类，将需要new的对象交于spring进行管理
步骤二：在任意位置，通过@Resource进行注入
创建 RestTemplate
一些其他设置
1. 拦截器配置
2. ErrorHandler 配置
3. HttpMessageConverter 配置
4.路由配置
5 实例化
参考地址
RestTemplate是spring的一个rest客户端，在spring-web这个包下，spring boot的依赖如下

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
1
2
3
4
点进去可以看到spring-web

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>5.0.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
1
2
3
4
5
6
所以如果在非spring的框架下直接引入spring-web这个包即可

RestTemplate只是对其它Rest客户端的一个封装，本身并没有自己的实现。
很多人都说Spring Boot 2.0之前RestTestTemplate的默认实现是HttpClient，2.+为OKHttp3，其实这种说法并不完全正确，如果没有引入这些客户端的jar包，其默认实现是HttpURLConnection（集成了URLConnection），这是JDK自带的REST客户端实现。

RestTemplate有三个实现

RestTemplate()
RestTemplate(List<HttpMessageConverter<?>> messageConverters)
RestTemplate(ClientHttpRequestFactory requestFactory)。
RestTemplate(ClientHttpRequestFactory requestFactory)是要说的重点，跟踪ClientHttpRequestFactory查看其实现，如下图

先看默认实现的测试代码

RestConfig

package com.pk.client.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * Created by pangkunkun on 2018/8/25.
 */
 @Configuration
 public class RestConfig {
    @Bean
    public RestTemplate restTemplate(){
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate;
    }
 }
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
 11
 12
 13
 14
 15
 16
 17
 MyRestController

package com.pk.client.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * Created by pangkunkun on 2018/8/25.
 */
    @RestController
    public class MyRestController {

    private static final Logger logger = LoggerFactory.getLogger(MyRestController.class);

    @Autowired
    private RestTemplate restTemplate;

    private static final String URL = "http://localhost:8081/server";

    @GetMapping("/default")
    public void defaultRestClient(){
        String result = restTemplate.getForObject(URL,String.class);
        logger.info("result = {}",result);
    }

}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
Server端代码

package com.pk.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by pangkunkun on 2018/8/25.
 */
 @SpringBootApplication
 @RestController
 public class RestServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(RestServerApplication.class,args);
    }

    @GetMapping("/server")
    public String restServer(){
        System.out.println("This is rest server");
        return "success";
    }
 }
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19
 20
 21
 22
 23
 24
 在浏览器调用client之后会追踪到下面这里

在这里就可以看到其实现方式是HttpURLConnection（URLConnection）。
所以，在没有第三方依赖的情况下其默认实现是URLConnection。

现在来看下其它几种客户端的引入。在ClientHttpRequestFactory的实现那张图中列出了RestTemplate的几种REST Client的封装。其中最常用的有以下三种：

SimpleClientHttpRequestFactory（封装URLConnection）
HttpComponentsClientHttpRequestFactory（封装HttpClient）
OkHttp3ClientHttpRequestFactory(封装OKHttp)
其实Netty4ClientHttpRequestFactory这个为在测试的时候也测试过，不过spring boot 2.0之后就不推荐使用了，而且我测试的时候性能上明显不如其它几个，所以这里没加上

其切换与使用也很简单，在pom中引入相应依赖

<dependencies>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.6</version>
    </dependency>
    <dependency>
        <groupId>com.squareup.okhttp3</groupId>
        <artifactId>okhttp</artifactId>
        <version>3.11.0</version>
    </dependency>
</dependencies>
1
2
3
4
5
6
7
8
9
10
11
12
在Config中的配置

package com.pk.client.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.http.client.OkHttp3ClientHttpRequestFactory;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

/**
 * Created by pangkunkun on 2018/8/25.
 */
 @Configuration
 public class RestConfig {

    @Bean
    public RestTemplate restTemplate(){
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate;
    }

    @Bean("urlConnection")
    public RestTemplate urlConnectionRestTemplate(){
        RestTemplate restTemplate = new RestTemplate(new SimpleClientHttpRequestFactory());
        return restTemplate;
    }

    @Bean("httpClient")
    public RestTemplate httpClientRestTemplate(){
        RestTemplate restTemplate = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
        return restTemplate;
    }

    @Bean("OKHttp3")
    public RestTemplate OKHttp3RestTemplate(){
        RestTemplate restTemplate = new RestTemplate(new OkHttp3ClientHttpRequestFactory());
        return restTemplate;
    }
 }
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19
 20
 21
 22
 23
 24
 25
 26
 27
 28
 29
 30
 31
 32
 33
 34
 35
 36
 37
 38
 39
 一、RestTemplate 语法
 //1 创建核心类
 RestTemplate restTemplate = new RestTemplate();

//get请求
ResponseEntity<返回值类型> entity =
restTemplate.getForEntity(‘请求路径’, 返回值类型.class );

//post请求
ResponseEntity<返回值类型> entity =
restTemplate.postForEntity(‘请求路径’,请求参数对象,返回值类型.class);

//put请求
restTemplate.put(‘请求路径’, 请求参数);

//delete请求
restTemplate.delete(‘请求路径’)

二、配置类的使用
步骤一：编写配置类，将需要new的对象交于spring进行管理
@Configuration    //配置类，spring boot会自动扫描
public class 类{
	@Bean		//spring将管理此方法创建的对象
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}
}
1
2
3
4
5
6
7
步骤二：在任意位置，通过@Resource进行注入
public class AdminClient{
    @Resource			//因为配置类已经构建对象，此处自动注入
    public RestTemplate restTemplate;
    //..使用即可
}
1
2
3
4
5
创建 RestTemplate
@Bean
public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
    RestTemplate restTemplate = new RestTemplate(factory);
    return restTemplate;
}

@Bean
public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
    HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectionRequestTimeout(connectionRequestTimeout); //连接请求超时时间
        httpRequestFactory.setConnectTimeout(connectionTimeout);//连接超时时间
        httpRequestFactory.setReadTimeout(readTimeout);//读取超时时间
	  // 设置代理
    //factory.setProxy(null);
    return factory;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
创建 RestTemplate 时需要一个 ClientHttpRequestFactory，通过这个请求工厂，我们可以统一设置请求的超时时间，设置代理以及一些其他细节。通过上面代码配置后，我们直接在代码中注入 RestTemplate 就可以使用了。
有时候我们还需要通过 ClientHttpRequestFactory 配置最大链接数，忽略SSL证书等，大家需要的时候可以自己查看代码设置

一些其他设置
1. 拦截器配置
RestTemplate 也可以设置拦截器做一些统一处理。这个功能感觉和 Spring MVC 的拦截器类似。配置也很简单：

class MyInterceptor implements ClientHttpRequestInterceptor{

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
            logger.info("enter interceptor...");
            return execution.execute(request,body);
        }
    }

 @Bean
public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
    RestTemplate restTemplate = new RestTemplate(factory);
    MyInterceptor myInterceptor = new MyInterceptor();
    List<ClientHttpRequestInterceptor> list = new ArrayList<>();
    list.add(myInterceptor);
    restTemplate.setInterceptors(list);
    return restTemplate;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
2. ErrorHandler 配置
ErrorHandler 用来对调用错误对统一处理。

public class MyResponseErrorHandler extends DefaultResponseErrorHandler {

        @Override
        public boolean hasError(ClientHttpResponse response) throws IOException {
            return super.hasError(response);
        }
    
        @Override
        public void handleError(ClientHttpResponse response) throws IOException {
            HttpStatus statusCode = HttpStatus.resolve(response.getRawStatusCode());
            if (statusCode == null) {
                throw new UnknownHttpStatusCodeException(response.getRawStatusCode(), response.getStatusText(),
                        response.getHeaders(), getResponseBody(response), getCharset(response));
            }
            handleError(response, statusCode);
        }
        @Override
        protected void handleError(ClientHttpResponse response, HttpStatus statusCode) throws IOException {
            switch (statusCode.series()) {
                case CLIENT_ERROR:
                    HttpClientErrorException exp1 = new HttpClientErrorException(statusCode, response.getStatusText(), response.getHeaders(), getResponseBody(response), getCharset(response));
                    logger.error("客户端调用异常",exp1);
                    throw  exp1;
                case SERVER_ERROR:
                    HttpServerErrorException exp2 = new HttpServerErrorException(statusCode, response.getStatusText(),
                            response.getHeaders(), getResponseBody(response), getCharset(response));
                    logger.error("服务端调用异常",exp2);
                    throw exp2;
                default:
                    UnknownHttpStatusCodeException exp3 = new UnknownHttpStatusCodeException(statusCode.value(), response.getStatusText(),
                            response.getHeaders(), getResponseBody(response), getCharset(response));
                    logger.error("网络调用未知异常");
                    throw exp3;
            }
        }
    
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
@Bean
public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
    RestTemplate restTemplate = new RestTemplate(factory);
    MyResponseErrorHandler errorHandler = new MyResponseErrorHandler();
    restTemplate.setErrorHandler(errorHandler);
    List<HttpMessageConverter<?>> messageConverters = restTemplate.getMessageConverters();
    // 通过下面代码可以添加新的 HttpMessageConverter
    //messageConverters.add(new );
    return restTemplate;
}
1
2
3
4
5
6
7
8
9
10
3. HttpMessageConverter 配置
RestTemplate 也可以配置 HttpMessageConverter，配置的原理和 Spring MVC 中类似。

4.路由配置
5 实例化
RestTemplate的实例化
RestTemplate实例通常需要自己进行定制，SpringBoot相关的自动配置bean。但是，SpringBoot提供了自动配置的RestTemplateBuilder，可以用它来创建RestTemplate实例。

那么，SpringBoot是如何提供RestTemplateBuilder的呢？我们先来看一下源代码。自动配置的源码位于spring-boot-autoconfigure的RestTemplateAutoConfiguration当中。看一下部分源码：

@Configuration(
    proxyBeanMethods = false
)
@AutoConfigureAfter({HttpMessageConvertersAutoConfiguration.class})
@ConditionalOnClass({RestTemplate.class})
@Conditional({RestTemplateAutoConfiguration.NotReactiveWebApplicationCondition.class})
public class RestTemplateAutoConfiguration {
    @Bean
    @Lazy
    @ConditionalOnMissingBean
    public RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer restTemplateBuilderConfigurer) {
        RestTemplateBuilder builder = new RestTemplateBuilder(new RestTemplateCustomizer[0]);
        return restTemplateBuilderConfigurer.configure(builder);
    }
    // 省略其他代码
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
通过上面代码可以看出springboot默认会自动实例化这么一个RestTemplateBuilder，并且bean的名称为restTemplateBuilder。那么，此时我们就可以直接注入来进行使用了。

RestTemplate的实例化配置，有两种方案可选，方案一：

@Configuration
public class RestTemplateConfig {
    @Autowired
    private RestTemplateBuilder restTemplateBuilder;
    @Bean
    public RestTemplate restTemplate() {
        restTemplateBuilder.setConnectTimeout(Duration.ofSeconds(5));
        restTemplateBuilder.setReadTimeout(Duration.ofSeconds(5));
        return restTemplateBuilder.build();
    }
}
1
2
3
4
5
6
7
8
9
10
11
此种方案很直观明白，直接注入RestTemplateBuilder，然后在实例化RestTemplate时进行使用即可。

另外一种方案，更简化一些。直接作为restTemplate方法的参数传入，SpringBoot在实例化RestTemplate时会自动匹配注入，使用示例如下：

@Component
public class RestConfig {undefined
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {undefined
builder.setConnectTimeout(Duration.ofSeconds(5));
builder.setReadTimeout(Duration.ofSeconds(5));
return builder.build();
}
}
当然，根据你的具体需要可以调用RestTemplateBuilder的其他方法，进行参数项的设置，也就是定制化你的RestTemplate，最后调用build方法进行构建。

1、以restTemplate.getForObject为例


2、private ClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();

@Override
public ClientHttpRequest createRequest(URI uri, HttpMethod httpMethod) throws IOException {
	HttpURLConnection connection = openConnection(uri.toURL(), this.proxy);
	prepareConnection(connection, httpMethod.name());

	if (this.bufferRequestBody) {
		return new SimpleBufferingClientHttpRequest(connection, this.outputStreaming);
	}
	else {
		return new SimpleStreamingClientHttpRequest(connection, this.chunkSize, this.outputStreaming);
	}
}
1
2
3
4
5
6
7
8
9
10
11
12
默认是使用HttpURLConnection 。可以指定ClientHttpRequestFactory。可选择OkHttp3ClientHttpRequestFactory、HttpComponentsClientHttpRequestFactory

参考地址
https://www.cnblogs.com/lay2017/p/11741206.html

https://blog.csdn.net/weixin_36146358/article/details/102511685
https://ifeve.com/httpclient-2-3/
————————————————
版权声明：本文为CSDN博主「Mr.Debug」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_34614236/article/details/118002606