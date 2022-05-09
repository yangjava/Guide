# HttpClient

在Java的世界中，Http客户端之前一直是Apache家的HttpClient占据主导，但是由于此包较为庞大，API又比较难用，因此并不使用很多场景。而新兴的OkHttp、Jodd-http固然好用，但是面对一些场景时，学习成本还是有一些的。很多时候，我们想追求轻量级的Http客户端，并且追求简单易用。而JDK自带的HttpUrlConnection可以满足大部分需求。Hutool针对此类做了一层封装，使Http请求变得无比简单。





HttpClient相比传统JDK自带的URLConnection，增加了易用性和灵活性，它不仅使客户端发送Http请求变得容易，而且也方便开发人员测试接口（基于Http协议的），提高了开发的效率，也方便提高代码的健壮性。因此熟练掌握HttpClient是很重要的必修内容，掌握HttpClient后，相信对于Http协议的了解会更加深入。

org.apache.commons.httpclient.HttpClient与org.apache.http.client.HttpClient的区别

Commons的HttpClient项目现在是生命的尽头，不再被开发,  已被Apache HttpComponents项目HttpClient和HttpCore  模组取代，提供更好的性能和更大的灵活性。  

一、简介
HttpClient是Apache Jakarta Common下的子项目，用来提供高效的、最新的、功能丰富的支持HTTP协议的客户端编程工具包，并且它支持HTTP协议最新的版本和建议。HttpClient已经应用在很多的项目中，比如Apache Jakarta上很著名的另外两个开源项目Cactus和HTMLUnit都使用了HttpClient。

下载地址: http://hc.apache.org/downloads.cgi

二、特性
1. 基于标准、纯净的java语言。实现了Http1.0和Http1.1

2. 以可扩展的面向对象的结构实现了Http全部的方法（GET, POST, PUT, DELETE, HEAD, OPTIONS, and TRACE）。

3. 支持HTTPS协议。

4. 通过Http代理建立透明的连接。

5. 利用CONNECT方法通过Http代理建立隧道的https连接。

6. Basic, Digest, NTLMv1, NTLMv2, NTLM2 Session, SNPNEGO/Kerberos认证方案。

7. 插件式的自定义认证方案。

8. 便携可靠的套接字工厂使它更容易的使用第三方解决方案。

9. 连接管理器支持多线程应用。支持设置最大连接数，同时支持设置每个主机的最大连接数，发现并关闭过期的连接。

10. 自动处理Set-Cookie中的Cookie。

11. 插件式的自定义Cookie策略。

12. Request的输出流可以避免流中内容直接缓冲到socket服务器。

13. Response的输入流可以有效的从socket服务器直接读取相应内容。

14. 在http1.0和http1.1中利用KeepAlive保持持久连接。

15. 直接获取服务器发送的response code和 headers。

16. 设置连接超时的能力。

17. 实验性的支持http1.1 response caching。

18. 源代码基于Apache License 可免费获取。

三、使用方法
使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可。

1. 创建HttpClient对象。

2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。

3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HttpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。

4. 调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。

5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。

6. 释放连接。无论执行方法是否成功，都必须释放连接

相关jar包

commons-cli-1.2.jar  
commons-codec-1.9.jar  
commons-logging-1.2.jar  
fluent-hc-4.5.1.jar  
httpclient-4.5.1.jar  
httpclient-cache-4.5.1.jar  
httpclient-win-4.5.1.jar  
httpcore-4.4.3.jar  
httpcore-ab-4.4.3.jar  
httpcore-nio-4.4.3.jar  
httpmime-4.5.1.jar  
jna-4.1.0.jar  
jna-platform-4.1.0.jar  
最简单post请求, 源自 http://my.oschina.net/xinxingegeya/blog/282683

package a;  

import java.io.FileInputStream;  
import java.io.IOException;  
import java.util.ArrayList;  
import java.util.List;  
import java.util.Properties;  

import org.apache.http.HttpEntity;  
import org.apache.http.HttpResponse;  
import org.apache.http.NameValuePair;  
import org.apache.http.client.HttpClient;  
import org.apache.http.client.config.RequestConfig;  
import org.apache.http.client.entity.UrlEncodedFormEntity;  
import org.apache.http.client.methods.HttpPost;  
import org.apache.http.impl.client.DefaultHttpClient;  
import org.apache.http.message.BasicNameValuePair;  
import org.apache.http.util.EntityUtils;  

public class First {  
    public static void main(String[] args) throws Exception{  
        List<NameValuePair> formparams = new ArrayList<NameValuePair>();  
        formparams.add(new BasicNameValuePair("account", ""));  
        formparams.add(new BasicNameValuePair("password", ""));  
        HttpEntity reqEntity = new UrlEncodedFormEntity(formparams, "utf-8");  
    
        RequestConfig requestConfig = RequestConfig.custom()  
        .setConnectTimeout(5000)//一、连接超时：connectionTimeout-->指的是连接一个url的连接等待时间  
                .setSocketTimeout(5000)// 二、读取数据超时：SocketTimeout-->指的是连接上一个url，获取response的返回等待时间  
                .setConnectionRequestTimeout(5000)  
                .build();  
    
        HttpClient client = new DefaultHttpClient();  
        HttpPost post = new HttpPost("http://cnivi.com.cn/login");  
        post.setEntity(reqEntity);  
        post.setConfig(requestConfig);  
        HttpResponse response = client.execute(post);  
    
        if (response.getStatusLine().getStatusCode() == 200) {  
            HttpEntity resEntity = response.getEntity();  
            String message = EntityUtils.toString(resEntity, "utf-8");  
            System.out.println(message);  
        } else {  
            System.out.println("请求失败");  
        }  
    }  

}  
四、实例
主文件
package com.test;  
      
import java.io.File;  
import java.io.FileInputStream;  
import java.io.IOException;  
import java.io.UnsupportedEncodingException;  
import java.security.KeyManagementException;  
import java.security.KeyStore;  
import java.security.KeyStoreException;  
import java.security.NoSuchAlgorithmException;  
import java.security.cert.CertificateException;  
import java.util.ArrayList;  
import java.util.List;  
import javax.net.ssl.SSLContext;  
import org.apache.http.HttpEntity;  
import org.apache.http.NameValuePair;  
import org.apache.http.ParseException;  
import org.apache.http.client.ClientProtocolException;  
import org.apache.http.client.entity.UrlEncodedFormEntity;  
import org.apache.http.client.methods.CloseableHttpResponse;  
import org.apache.http.client.methods.HttpGet;  
import org.apache.http.client.methods.HttpPost;  
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;  
import org.apache.http.conn.ssl.SSLContexts;  
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;  
import org.apache.http.entity.ContentType;  
import org.apache.http.entity.mime.MultipartEntityBuilder;  
import org.apache.http.entity.mime.content.FileBody;  
import org.apache.http.entity.mime.content.StringBody;  
import org.apache.http.impl.client.CloseableHttpClient;  
import org.apache.http.impl.client.HttpClients;  
import org.apache.http.message.BasicNameValuePair;  
import org.apache.http.util.EntityUtils;  
import org.apache.http.client.config.RequestConfig;  
import org.junit.Test;  
public class HttpClientTest {  
　　//方法见下........  
}  
HttpClientUtils工具类
package com.bobo.code.web.controller.technology.httpcomponents;  


import org.apache.http.HttpEntity;  
import org.apache.http.HttpHost;  
import org.apache.http.HttpResponse;  
import org.apache.http.NameValuePair;  
import org.apache.http.client.HttpClient;  
import org.apache.http.client.config.RequestConfig;  
import org.apache.http.client.methods.HttpUriRequest;  
import org.apache.http.client.methods.RequestBuilder;  
import org.apache.http.conn.routing.HttpRoute;  
import org.apache.http.impl.client.CloseableHttpClient;  
import org.apache.http.impl.client.HttpClientBuilder;  
import org.apache.http.impl.client.HttpClients;  
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;  
import org.apache.http.message.BasicNameValuePair;  
import org.apache.http.util.EntityUtils;  
    
import java.io.IOException;  
import java.util.*;  
    
public class HttpClientUtils {  
    
    private static PoolingHttpClientConnectionManager connectionManager = null;  
    private static HttpClientBuilder httpBuilder = null;  
    private static RequestConfig requestConfig = null;  
    
    private static int MAXCONNECTION = 10;  
    
    private static int DEFAULTMAXCONNECTION = 5;  
    
    private static String IP = "cnivi.com.cn";  
    private static int PORT = 80;  
    
    static {  
        //设置http的状态参数  
        requestConfig = RequestConfig.custom()  
                .setSocketTimeout(5000)  
                .setConnectTimeout(5000)  
                .setConnectionRequestTimeout(5000)  
                .build();  
    
        HttpHost target = new HttpHost(IP, PORT);  
        connectionManager = new PoolingHttpClientConnectionManager();  
        connectionManager.setMaxTotal(MAXCONNECTION);//客户端总并行链接最大数  
        connectionManager.setDefaultMaxPerRoute(DEFAULTMAXCONNECTION);//每个主机的最大并行链接数  
        connectionManager.setMaxPerRoute(new HttpRoute(target), 20);  
        httpBuilder = HttpClients.custom();  
        httpBuilder.setConnectionManager(connectionManager);  
    }  
    
    public static CloseableHttpClient getConnection() {  
        CloseableHttpClient httpClient = httpBuilder.build();  
        return httpClient;  
    }  


​    
​    public static HttpUriRequest getRequestMethod(Map<String, String> map, String url, String method) {  
​        List<NameValuePair> params = new ArrayList<NameValuePair>();  
​        Set<Map.Entry<String, String>> entrySet = map.entrySet();  
​        for (Map.Entry<String, String> e : entrySet) {  
​            String name = e.getKey();  
​            String value = e.getValue();  
​            NameValuePair pair = new BasicNameValuePair(name, value);  
​            params.add(pair);  
​        }  
​        HttpUriRequest reqMethod = null;  
​        if ("post".equals(method)) {  
​            reqMethod = RequestBuilder.post().setUri(url)  
​                    .addParameters(params.toArray(new BasicNameValuePair[params.size()]))  
​                    .setConfig(requestConfig).build();  
​        } else if ("get".equals(method)) {  
​            reqMethod = RequestBuilder.get().setUri(url)  
​                    .addParameters(params.toArray(new BasicNameValuePair[params.size()]))  
​                    .setConfig(requestConfig).build();  
​        }  
​        return reqMethod;  
​    }  
​    
​    public static void main(String args[]) throws IOException {  
​        Map<String, String> map = new HashMap<String, String>();  
​        map.put("account", "");  
​        map.put("password", "");  
​    
        HttpClient client = getConnection();  
        HttpUriRequest post = getRequestMethod(map, "http://cnivi.com.cn/login", "post");  
        HttpResponse response = client.execute(post);  
    
        if (response.getStatusLine().getStatusCode() == 200) {  
            HttpEntity entity = response.getEntity();  
            String message = EntityUtils.toString(entity, "utf-8");  
            System.out.println(message);  
        } else {  
            System.out.println("请求失败");  
        }  
    }  
}  
get方式
/** 
     * 发送 get请求 
     */   
    public void get() {   
        CloseableHttpClient httpclient = HttpClients.createDefault();   
        try {   
            // 创建httpget.     
            HttpGet httpget = new HttpGet("http://www.baidu.com/");   
            System.out.println("executing request " + httpget.getURI());   
            // 执行get请求.     
            CloseableHttpResponse response = httpclient.execute(httpget);   
            try {   
                // 获取响应实体     
                HttpEntity entity = response.getEntity();   
                System.out.println("--------------------------------------");   
                // 打印响应状态     
                System.out.println(response.getStatusLine());   
                if (entity != null) {   
                    // 打印响应内容长度     
                    System.out.println("Response content length: " + entity.getContentLength());   
                    // 打印响应内容     
                    System.out.println("Response content: " + EntityUtils.toString(entity));   
                }   
                System.out.println("------------------------------------");   
            } finally {   
                response.close();   
            }   
        } catch (ClientProtocolException e) {   
            e.printStackTrace();   
        } catch (ParseException e) {   
            e.printStackTrace();   
        } catch (IOException e) {   
            e.printStackTrace();   
        } finally {   
            // 关闭连接,释放资源     
            try {   
                httpclient.close();   
            } catch (IOException e) {   
                e.printStackTrace();   
            }   
        }   
    }  
post方式 
/** 
     * 发送 post请求访问本地应用并根据传递参数不同返回不同结果 
     */   
    public void post() {   
        // 创建默认的httpClient实例.     
        CloseableHttpClient httpclient = HttpClients.createDefault();   
        // 创建httppost     
        HttpPost httppost = new HttpPost("http://localhost:8080/myDemo/Ajax/serivceJ.action");   
        // 创建参数队列     
        List<NameValuePair> formparams = new ArrayList<NameValuePair>();   
        formparams.add(new BasicNameValuePair("type", "house"));   
        UrlEncodedFormEntity uefEntity;   
        try {   
            uefEntity = new UrlEncodedFormEntity(formparams, "UTF-8");   
            httppost.setEntity(uefEntity);   
            System.out.println("executing request " + httppost.getURI());   
            CloseableHttpResponse response = httpclient.execute(httppost);   
            try {   
                HttpEntity entity = response.getEntity();   
                if (entity != null) {   
                    System.out.println("--------------------------------------");   
                    System.out.println("Response content: " + EntityUtils.toString(entity, "UTF-8"));   
                    System.out.println("--------------------------------------");   
                }   
            } finally {   
                response.close();   
            }   
        } catch (ClientProtocolException e) {   
            e.printStackTrace();   
        } catch (UnsupportedEncodingException e1) {   
            e1.printStackTrace();   
        } catch (IOException e) {   
            e.printStackTrace();   
        } finally {   
            // 关闭连接,释放资源     
            try {   
                httpclient.close();   
            } catch (IOException e) {   
                e.printStackTrace();   
            }   
        }   
    }  
post方式乱码补充　

如果有乱码,可以偿试使用 StringEntity 来替换HttpEntity:

StringEntity content =new StringEntity(soapRequestData.toString(), Charset.forName("UTF-8"));// 第二个参数，设置后才会对，内容进行编码  
        content.setContentType("application/soap+xml; charset=UTF-8");  
        content.setContentEncoding("UTF-8");  
        httppost.setEntity(content);  
具体SOAP协议代码如下:

    package com.isoftstone.core.service.impl;  
      
    import java.io.BufferedReader;  
    import java.io.File;  
    import java.io.FileInputStream;  
    import java.io.FileReader;  
    import java.io.IOException;  
    import java.io.InputStreamReader;  
    import java.nio.charset.Charset;  
    import java.util.Scanner;  
      
    import org.apache.http.HttpEntity;  
    import org.apache.http.HttpResponse;  
    import org.apache.http.client.ClientProtocolException;  
    import org.apache.http.client.HttpClient;  
    import org.apache.http.client.entity.EntityBuilder;  
    import org.apache.http.client.methods.HttpPost;  
    import org.apache.http.entity.ContentType;  
    import org.apache.http.entity.StringEntity;  
    import org.apache.http.impl.client.HttpClients;  
    import org.apache.http.util.EntityUtils;  
    import org.apache.log4j.Logger;  
    import org.jdom.Document;  
    import org.jdom.Element;  
      
    import com.isoftstone.core.common.constant.RequestConstants;  
    import com.isoftstone.core.common.tools.XmlTool;  
    import com.isoftstone.core.service.intf.ServiceOfStringPara;  
    /** 
     * 
     * 
     */  
    public class DeloittePricingSingleCarImpl implements ServiceOfStringPara {  
        private  String serviceUrl = "http://10.30.0.35:7001/ZSInsUW/Auto/PricingService";  
      
        private static Logger log = Logger.getLogger(DeloittePricingSingleCarImpl.class.getName());  
      
        public String invoke(String sRequest) {  
              
            StringBuffer soapRequestData = new StringBuffer();  
            soapRequestData.append("<soapenv:Envelope");  
            soapRequestData.append("  xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" ");  
            soapRequestData.append("  xmlns:prov=\"http://provider.webservice.zsins.dtt.com/\">");  
            soapRequestData.append(" <soapenv:Header/> ");  
            soapRequestData.append("<soapenv:Body>");  
            soapRequestData.append("<prov:executePrvPricing>");  
            soapRequestData.append("<arg0>");  
            soapRequestData.append("<![CDATA[" + sRequest + "]]>");  
            soapRequestData.append("</arg0>");  
            soapRequestData.append("</prov:executePrvPricing>");  
            soapRequestData.append(" </soapenv:Body>");  
            soapRequestData.append("</soapenv:Envelope>");  
      
            HttpClient httpclient = HttpClients.createDefault();  
            HttpPost httppost = new HttpPost(serviceUrl);  
      
            StringEntity content =new StringEntity(soapRequestData.toString(), Charset.forName("UTF-8"));// 第二个参数，设置后才会对，内容进行编码  
            content.setContentType("application/soap+xml; charset=UTF-8");  
            content.setContentEncoding("UTF-8");  
            httppost.setEntity(content);  
              
            //用下面的服务器端以UTF-8接收到的报文会乱码,原因未知  
    //        HttpEntity reqEntity = EntityBuilder.create().setContentType(  
    //                ContentType.TEXT_PLAIN) // .TEXT_PLAIN  
    //                .setText(soapRequestData.toString()).build();  
    //        httppost.setEntity(reqEntity);  
    //        httppost.addHeader("Content-Type",  
    //                "application/soap+xml; charset=utf-8");  
            HttpResponse response = null;  
            Document doc = null;  
            String returnXml = null;  
            String sentity = null;  
            try {  
                response = httpclient.execute(httppost);  
                HttpEntity resEntity = response.getEntity();  
                if (resEntity != null) {  
                    sentity = EntityUtils.toString(resEntity, "UTF-8");  
                    doc = XmlTool.getDocument(sentity, RequestConstants.ENCODE);  
                    System.out.println(doc.toString());  
                    Element eRoot = doc.getRootElement();  
                    Element body = eRoot.getChild("Body", eRoot.getNamespace());  
                    Element resp = (Element) body.getChildren().get(0);  
                    Element returnele = resp.getChild("return");  
                    returnXml = returnele.getText().toString();  
                }  
            } catch (ClientProtocolException e) {  
                e.printStackTrace();  
            } catch (IOException e) {  
                e.printStackTrace();  
            } catch (Exception e) {  
                e.printStackTrace();  
            } finally {  
                log.info("发送给系统的请求报文：\n" + soapRequestData.toString());  
                log.info("系统返回的响应报文：\n" + sentity);  
                log.info("返回给核心的的报文:\n" + returnXml);  
            }  
            return returnXml;  
        }  


​          
​        public String getServiceUrl() {  
​            return serviceUrl;  
​        }  


​      
​        public void setServiceUrl(String serviceUrl) {  
​            this.serviceUrl = serviceUrl;  
​        }  


​      
​        public static void main(String[] args) throws Exception{  
​            File file = new File("D:/test.txt");  
​            System.out.println(file.exists());  
​              
​            String temp2 = null;  
​            StringBuilder sb2 = new StringBuilder();  
​            InputStreamReader isr = new InputStreamReader(new FileInputStream(file),"GBK");  
​            BufferedReader br = new BufferedReader(isr);  
​            temp2 = br.readLine();  
​              
            while( temp2 != null ){  
                sb2.append(temp2);  
                temp2 = br.readLine();  
            }  
            String sss = sb2.toString();  
    //        System.out.println(sss.toString());  
            new DeloittePricingSingleCarImpl().invoke(sss);  
        }  
    }  

 




post提交表单
/** 
     * post方式提交表单（模拟用户登录请求） 
     */   
    public void postForm() {   
        // 创建默认的httpClient实例.     
        CloseableHttpClient httpclient = HttpClients.createDefault();   
        // 创建httppost     
        HttpPost httppost = new HttpPost("http://localhost:8080/myDemo/Ajax/serivceJ.action");   
        // 创建参数队列     
        List<NameValuePair> formparams = new ArrayList<NameValuePair>();   
        formparams.add(new BasicNameValuePair("username", "admin"));   
        formparams.add(new BasicNameValuePair("password", "123456"));   
        UrlEncodedFormEntity uefEntity;   
        try {   
            uefEntity = new UrlEncodedFormEntity(formparams, "UTF-8");   
            httppost.setEntity(uefEntity);   
            System.out.println("executing request " + httppost.getURI());   
            CloseableHttpResponse response = httpclient.execute(httppost);   
            try {   
                HttpEntity entity = response.getEntity();   
                if (entity != null) {   
                    System.out.println("--------------------------------------");   
                    System.out.println("Response content: " + EntityUtils.toString(entity, "UTF-8"));   
                    System.out.println("--------------------------------------");   
                }   
            } finally {   
                response.close();   
            }   
        } catch (ClientProtocolException e) {   
            e.printStackTrace();   
        } catch (UnsupportedEncodingException e1) {   
            e1.printStackTrace();   
        } catch (IOException e) {   
            e.printStackTrace();   
        } finally {   
            // 关闭连接,释放资源     
            try {   
                httpclient.close();   
            } catch (IOException e) {   
                e.printStackTrace();   
            }   
        }   
    }  
文件上传
/** 
     * 上传文件 
     */   
    public void upload() {   
        CloseableHttpClient httpclient = HttpClients.createDefault();   
        try {   
            HttpPost httppost = new HttpPost("http://localhost:8080/myDemo/Ajax/serivceFile.action");   
     
            FileBody bin = new FileBody(new File("F:\\image\\sendpix0.jpg"));   
            StringBody comment = new StringBody("A binary file of some kind", ContentType.TEXT_PLAIN);   
     
            HttpEntity reqEntity = MultipartEntityBuilder.create().addPart("bin", bin).addPart("comment", comment).build();   
     
            httppost.setEntity(reqEntity);   
     
            System.out.println("executing request " + httppost.getRequestLine());   
            CloseableHttpResponse response = httpclient.execute(httppost);   
            try {   
                System.out.println("----------------------------------------");   
                System.out.println(response.getStatusLine());   
                HttpEntity resEntity = response.getEntity();   
                if (resEntity != null) {   
                    System.out.println("Response content length: " + resEntity.getContentLength());   
                }   
                EntityUtils.consume(resEntity);   
            } finally {   
                response.close();   
            }   
        } catch (ClientProtocolException e) {   
            e.printStackTrace();   
        } catch (IOException e) {   
            e.printStackTrace();   
        } finally {   
            try {   
                httpclient.close();   
            } catch (IOException e) {   
                e.printStackTrace();   
            }   
        }   
    } 
ssl连接
/** 
     * HttpClient连接SSL 
     */   
    public void ssl() {   
        CloseableHttpClient httpclient = null;   
        try {   
            KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());   
            FileInputStream instream = new FileInputStream(new File("d:\\tomcat.keystore"));   
            try {   
                // 加载keyStore d:\\tomcat.keystore     
                trustStore.load(instream, "123456".toCharArray());   
            } catch (CertificateException e) {   
                e.printStackTrace();   
            } finally {   
                try {   
                    instream.close();   
                } catch (Exception ignore) {   
                }   
            }   
            // 相信自己的CA和所有自签名的证书   
            SSLContext sslcontext = SSLContexts.custom().loadTrustMaterial(trustStore, new TrustSelfSignedStrategy()).build();   
            // 只允许使用TLSv1协议   
            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslcontext, new String[] { "TLSv1" }, null,   
                    SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);   
            httpclient = HttpClients.custom().setSSLSocketFactory(sslsf).build();   
            // 创建http请求(get方式)   
            HttpGet httpget = new HttpGet("https://localhost:8443/myDemo/Ajax/serivceJ.action");   
            System.out.println("executing request" + httpget.getRequestLine());   
            CloseableHttpResponse response = httpclient.execute(httpget);   
            try {   
                HttpEntity entity = response.getEntity();   
                System.out.println("----------------------------------------");   
                System.out.println(response.getStatusLine());   
                if (entity != null) {   
                    System.out.println("Response content length: " + entity.getContentLength());   
                    System.out.println(EntityUtils.toString(entity));   
                    EntityUtils.consume(entity);   
                }   
            } finally {   
                response.close();   
            }   
        } catch (ParseException e) {   
            e.printStackTrace();   
        } catch (IOException e) {   
            e.printStackTrace();   
        } catch (KeyManagementException e) {   
            e.printStackTrace();   
        } catch (NoSuchAlgorithmException e) {   
            e.printStackTrace();   
        } catch (KeyStoreException e) {   
            e.printStackTrace();   
        } finally {   
            if (httpclient != null) {   
                try {   
                    httpclient.close();   
                } catch (IOException e) {   
                    e.printStackTrace();   
                }   
            }   
        }   
    }   
关于RequestConfig的配置:  
源自:  

http://segmentfault.com/a/1190000000587944

http://blog.csdn.net/walkerjong/article/details/51710945

public void requestConfig(){  
//      新建一个RequestConfig：  
        RequestConfig defaultRequestConfig = RequestConfig.custom()  
            //一、连接目标服务器超时时间：ConnectionTimeout-->指的是连接一个url的连接等待时间  
            .setConnectTimeout(5000)  
            //二、读取目标服务器数据超时时间：SocketTimeout-->指的是连接上一个url，获取response的返回等待时间  
            .setSocketTimeout(5000)  
            //三、从连接池获取连接的超时时间:ConnectionRequestTimeout  
            .setConnectionRequestTimeout(5000)  
            .build();  
           
//      这个超时可以设置为客户端级别,作为所有请求的默认值：  
        CloseableHttpClient httpclient = HttpClients.custom()  
            .setDefaultRequestConfig(defaultRequestConfig)  
            .build();  
//       httpclient.execute(httppost);的时候可以让httppost直接享受到httpclient中的默认配置.  
           
//      Request不会继承客户端级别的请求配置，所以在自定义Request的时候，需要将客户端的默认配置拷贝过去：  
        HttpGet httpget = new HttpGet("http://www.apache.org/");  
        RequestConfig requestConfig = RequestConfig.copy(defaultRequestConfig)  
            .setProxy(new HttpHost("myotherproxy", 8080))  
            .build();  
        httpget.setConfig(requestConfig);  
//      httpget可以单独地使用新copy的requestConfig请求配置,不会对别的request请求产生影响  
    }  
httpGet或httpPost 的abort()和releaseConnection()差异

//httpPost.abort();//中断请求,接下来可以开始另一段请求,所以个人理应,用这个应该可以在session中虚拟登录  
 //httpPost.releaseConnection();//释放请求.如果释放了相当于要清空session  
可知模拟登录可以如下:  源自 http://bbs.csdn.net/topics/390195343

package com.bms.core;  
     
import java.io.IOException;  
import java.util.ArrayList;  
import java.util.List;  
     
import org.apache.http.Consts;  
import org.apache.http.HttpEntity;  
import org.apache.http.HttpResponse;  
import org.apache.http.NameValuePair;  
import org.apache.http.client.ClientProtocolException;  
import org.apache.http.client.entity.UrlEncodedFormEntity;  
import org.apache.http.client.methods.HttpGet;  
import org.apache.http.client.methods.HttpPost;  
import org.apache.http.impl.client.DefaultHttpClient;  
import org.apache.http.message.BasicNameValuePair;  
import org.apache.http.util.EntityUtils;  
     
import com.bms.util.CommonUtil;  
     
public class Test2 {  
     
    /** 
     * @param args 
     * @throws IOException 
     * @throws ClientProtocolException 
     */  
    public static void main(String[] args) throws ClientProtocolException, IOException {  
        DefaultHttpClient httpclient = new DefaultHttpClient();  
     
         HttpGet httpGet = new HttpGet("http://www.baidu.com");  
         String body = "";  
         HttpResponse response;  
         HttpEntity entity;  
         response = httpclient.execute(httpGet);  
         entity = response.getEntity();  
         body = EntityUtils.toString(entity);//这个就是页面源码了  
         httpGet.abort();//中断请求,接下来可以开始另一段请求  
         System.out.println(body);  
         //httpGet.releaseConnection();//释放请求.如果释放了相当于要清空session  
         //以下是post方法  
         HttpPost httpPost = new HttpPost("http://www.baidu.com");//一定要改成可以提交的地址,这里用百度代替  
         List <NameValuePair> nvps = new ArrayList <NameValuePair>();  
         nvps.add(new BasicNameValuePair("name", "1"));//名值对  
         nvps.add(new BasicNameValuePair("account", "xxxx"));  
         httpPost.setEntity(new UrlEncodedFormEntity(nvps, Consts.UTF_8));  
         response = httpclient.execute(httpPost);  
         entity = response.getEntity();  
         body = EntityUtils.toString(entity);  
         System.out.println("Login form get: " + response.getStatusLine());//这个可以打印状态  
         httpPost.abort();  
         System.out.println(body);  
         httpPost.releaseConnection();  
    }  

}  
源自  http://blog.csdn.net/wangpeng047/article/details/19624529#reply

其它相关资料: 非CloseableHttpClient  HTTPClient模块的HttpGet和HttpPost

HttpClient 4.3教程

httpclient异常情况分析

我项目中用到的HttpClientUtil (2016/12/17)
package com.isoftstone.pcis.isc.util;  

import java.io.IOException;  
import java.io.InterruptedIOException;  
import java.net.UnknownHostException;  

import javax.net.ssl.SSLException;  
import javax.net.ssl.SSLHandshakeException;  

import org.apache.http.HttpEntityEnclosingRequest;  
import org.apache.http.HttpRequest;  
import org.apache.http.NoHttpResponseException;  
import org.apache.http.client.HttpRequestRetryHandler;  
import org.apache.http.client.protocol.HttpClientContext;  
import org.apache.http.config.Registry;  
import org.apache.http.config.RegistryBuilder;  
import org.apache.http.conn.ConnectTimeoutException;  
import org.apache.http.conn.socket.ConnectionSocketFactory;  
import org.apache.http.conn.socket.LayeredConnectionSocketFactory;  
import org.apache.http.conn.socket.PlainConnectionSocketFactory;  
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;  
import org.apache.http.impl.client.CloseableHttpClient;  
import org.apache.http.impl.client.HttpClients;  
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;  
import org.apache.http.protocol.HttpContext;  
import com.isoftstone.pcis.isc.util.ProperUtil;  

public class HttpClientUtil {  
      
    private static CloseableHttpClient httpclient = null;   
      
    static final int maxTotal=Integer.valueOf(ProperUtil.get("maxTotal")).intValue();//总最大连接数  
    static final int defaultMaxPerRoute=Integer.valueOf(ProperUtil.get("corePoolSize")).intValue();//每条线路最大连接数 = 本系统核心线程数 , 这样永远不会超过最大连接  
      
     public static CloseableHttpClient getHttpClient() {  
              
            if (null == httpclient) {  
                synchronized (HttpClientUtil.class) {  
                    if (null == httpclient) {  
                        httpclient = getNewHttpClient();  
                    }  
                }  
            }  
              
            return httpclient;  
        }  
       
       private static CloseableHttpClient getNewHttpClient() {  


​            
​            // 设置连接池  
​            ConnectionSocketFactory plainsf = PlainConnectionSocketFactory.getSocketFactory();  
​            LayeredConnectionSocketFactory sslsf = SSLConnectionSocketFactory.getSocketFactory();  
​            Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory> create().register("http", plainsf).register("https", sslsf).build();  
​            PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(registry);  
​            // 配置最大连接数  
​            cm.setMaxTotal(maxTotal);  
​            // 配置每条线路的最大连接数  
​            cm.setDefaultMaxPerRoute(defaultMaxPerRoute);  
​              
​            // 请求重试处理  
​            HttpRequestRetryHandler httpRequestRetryHandler = new HttpRequestRetryHandler() {  
​                @Override  
​                public boolean retryRequest(IOException exception,  
​                        int executionCount, HttpContext context) {  
​                    if (executionCount >= 2) {// 如果已经重试了2次，就放弃  
​                        return false;  
​                    }  
​                    if (exception instanceof NoHttpResponseException) {// 如果服务器丢掉了连接，那么就重试  
​                        return true;  
​                    }  
​                    if (exception instanceof SSLHandshakeException) {// 不要重试SSL握手异常  
​                        return false;  
​                    }  
​                    if (exception instanceof InterruptedIOException) {// 超时  
​                        return false;  
​                    }  
​                    if (exception instanceof UnknownHostException) {// 目标服务器不可达  
​                        return false;  
​                    }  
​                    if (exception instanceof ConnectTimeoutException) {// 连接被拒绝  
​                        return false;  
​                    }  
​                    if (exception instanceof SSLException) {// SSL握手异常  
​                        return false;  
​                    }  
​      
                    HttpClientContext clientContext = HttpClientContext.adapt(context);  
                    HttpRequest request = clientContext.getRequest();  
                      
                    if (!(request instanceof HttpEntityEnclosingRequest)) {  
                        return true;  
                    }  
                    return false;  
                }  
      
            };  
              
            CloseableHttpClient newHttpclient=null;  
          
                newHttpclient = HttpClients.custom()  
                                           .setConnectionManager(cm)  
//                                           .setDefaultRequestConfig(requestConfig)  
                                           .setRetryHandler(httpRequestRetryHandler)  
                                           .build();  
                  
             return newHttpclient;  
       }  
}  
我自己整理的HttpClientTool
package com.isoftstone.core.util;  

import java.io.IOException;  
import java.io.InterruptedIOException;  
import java.net.UnknownHostException;  

import javax.net.ssl.SSLException;  
import javax.net.ssl.SSLHandshakeException;  

import org.apache.http.HttpEntityEnclosingRequest;  
import org.apache.http.HttpHost;  
import org.apache.http.HttpRequest;  
import org.apache.http.NoHttpResponseException;  
import org.apache.http.client.HttpRequestRetryHandler;  
import org.apache.http.client.config.RequestConfig;  
import org.apache.http.client.protocol.HttpClientContext;  
import org.apache.http.config.Registry;  
import org.apache.http.config.RegistryBuilder;  
import org.apache.http.conn.ConnectTimeoutException;  
import org.apache.http.conn.routing.HttpRoute;  
import org.apache.http.conn.socket.ConnectionSocketFactory;  
import org.apache.http.conn.socket.LayeredConnectionSocketFactory;  
import org.apache.http.conn.socket.PlainConnectionSocketFactory;  
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;  
import org.apache.http.impl.client.CloseableHttpClient;  
import org.apache.http.impl.client.HttpClients;  
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;  
import org.apache.http.protocol.HttpContext;  

/** 
 * org.apache.http.impl.client.CloseableHttpClient链接池生成工具 
 * @reference http://www.cnblogs.com/whatlonelytear/articles/4835538.html 
 * @author King 
 * @date 20170601 
 */  
   public class HttpClientTool {  
  
    // org.apache.http.impl.client.CloseableHttpClient  
    private static CloseableHttpClient httpclient = null;  
  
    // 这里就直接默认固定了,因为以下三个参数在新建的method中仍然可以重新配置并被覆盖.  
    static final int connectionRequestTimeout = 5000;// ms毫秒,从池中获取链接超时时间  
    static final int connectTimeout = 5000;// ms毫秒,建立链接超时时间  
    static final int socketTimeout = 30000;// ms毫秒,读取超时时间  
  
    // 总配置,主要涉及是以下两个参数,如果要作调整没有用到properties会比较后麻烦,但鉴于一经粘贴,随处可用的特点,就不再做依赖性配置化处理了.  
    // 而且这个参数同一家公司基本不会变动.  
    static final int maxTotal = 500;// 最大总并发,很重要的参数  
    static final int maxPerRoute = 100;// 每路并发,很重要的参数  
  
    // 正常情况这里应该配成MAP或LIST  
    // 细化配置参数,用来对每路参数做精细化处理,可以管控各ip的流量,比如默认配置请求baidu:80端口最大100个并发链接,  
    static final String detailHostName = "http://www.baidu.com";// 每个细化配置之ip(不重要,在特殊场景很有用)  
    static final int detailPort = 80;// 每个细化配置之port(不重要,在特殊场景很有用)  
    static final int detailMaxPerRoute = 100;// 每个细化配置之最大并发数(不重要,在特殊场景很有用)  
  
    public static CloseableHttpClient getHttpClient() {  
        if (null == httpclient) {  
            synchronized (HttpClientTool.class) {  
                if (null == httpclient) {  
                    httpclient = init();  
                }  
            }  
        }  
        return httpclient;  
    }  
  
    /** 
     * 链接池初始化 这里最重要的一点理解就是. 让CloseableHttpClient 一直活在池的世界里, 但是HttpPost却一直用完就消掉. 
     * 这样可以让链接一直保持着. 
     *  
     * @return 
       */  
       private static CloseableHttpClient init() {  
        CloseableHttpClient newHttpclient = null;  
   
        // 设置连接池  
        ConnectionSocketFactory plainsf = PlainConnectionSocketFactory.getSocketFactory();  
        LayeredConnectionSocketFactory sslsf = SSLConnectionSocketFactory.getSocketFactory();  
        Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory> create().register("http", plainsf).register("https", sslsf).build();  
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(registry);  
        // 将最大连接数增加  
        cm.setMaxTotal(maxTotal);  
        // 将每个路由基础的连接增加  
        cm.setDefaultMaxPerRoute(maxPerRoute);  
   
        // 细化配置开始,其实这里用Map或List的for循环来配置每个链接,在特殊场景很有用.  
        // 将每个路由基础的连接做特殊化配置,一般用不着  
        HttpHost httpHost = new HttpHost(detailHostName, detailPort);  
        // 将目标主机的最大连接数增加  
        cm.setMaxPerRoute(new HttpRoute(httpHost), detailMaxPerRoute);  
        // cm.setMaxPerRoute(new HttpRoute(httpHost2),  
        // detailMaxPerRoute2);//可以有细化配置2  
        // cm.setMaxPerRoute(new HttpRoute(httpHost3),  
        // detailMaxPerRoute3);//可以有细化配置3  
        // 细化配置结束  
   
        // 请求重试处理  
        HttpRequestRetryHandler httpRequestRetryHandler = new HttpRequestRetryHandler() {  
            @Override  
            public boolean retryRequest(IOException exception, int executionCount, HttpContext context) {  
                if (executionCount >= 2) {// 如果已经重试了2次，就放弃  
                    return false;  
                }  
                if (exception instanceof NoHttpResponseException) {// 如果服务器丢掉了连接，那么就重试  
                    return true;  
                }  
                if (exception instanceof SSLHandshakeException) {// 不要重试SSL握手异常  
                    return false;  
                }  
                if (exception instanceof InterruptedIOException) {// 超时  
                    return false;  
                }  
                if (exception instanceof UnknownHostException) {// 目标服务器不可达  
                    return false;  
                }  
                if (exception instanceof ConnectTimeoutException) {// 连接被拒绝  
                    return false;  
                }  
                if (exception instanceof SSLException) {// SSL握手异常  
                    return false;  
                }  
   
                HttpClientContext clientContext = HttpClientContext.adapt(context);  
                HttpRequest request = clientContext.getRequest();  
                // 如果请求是幂等的，就再次尝试  
                if (!(request instanceof HttpEntityEnclosingRequest)) {  
                    return true;  
                }  
                return false;  
            }  
        };  
   
        // 配置请求的超时设置  
        RequestConfig requestConfig = RequestConfig.custom().setConnectionRequestTimeout(connectionRequestTimeout).setConnectTimeout(connectTimeout).setSocketTimeout(socketTimeout).build();  
        newHttpclient = HttpClients.custom().setConnectionManager(cm).setDefaultRequestConfig(requestConfig).setRetryHandler(httpRequestRetryHandler).build();  
        return newHttpclient;  
       }  
         } 
         package com.alqsoft.utils;


import java.io.BufferedReader;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;


import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.ParseException;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.CookieSpecs;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.config.Registry;
import org.apache.http.config.RegistryBuilder;
import org.apache.http.cookie.Cookie;
import org.apache.http.cookie.CookieOrigin;
import org.apache.http.cookie.CookieSpec;
import org.apache.http.cookie.CookieSpecProvider;
import org.apache.http.cookie.MalformedCookieException;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.cookie.BestMatchSpecFactory;
import org.apache.http.impl.cookie.BrowserCompatSpec;
import org.apache.http.impl.cookie.BrowserCompatSpecFactory;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.protocol.HttpContext;
import org.apache.http.util.EntityUtils;


public class HttpClientObject {
    private static Log logger = LogFactory.getLog(HttpClientObject.class);
    private CloseableHttpClient httpClient = null;
    private HttpResponse response;

    private HttpPost httpPost = null;
     
    private HttpGet httpGet = null;
     
    private String paramKey = "";
     
    private String paramValue = "";
     
    private String responseString;
     
    public void setParamKey(String paramKey) {
        this.paramKey = paramKey;
    }
     
    public void setParamValue(String paramValue) {
        this.paramValue = paramValue;
    }
     
    public String getResponseString() {
        return responseString;
    }
     
    public HttpClientObject() {
        this.getHttpClient();
    }
     
    private List<NameValuePair> getRequestBody() {
        NameValuePair pair1 = new BasicNameValuePair(paramKey, paramValue);
        List<NameValuePair> pairList = new ArrayList<NameValuePair>();
        pairList.add(pair1);
        return pairList;
    }

 



    public void submit() {
        try {
            if (httpPost != null) {
                response = httpClient.execute(httpPost);
                httpPost = null;
            }
            if (httpGet != null) {
                response = httpClient.execute(httpGet);
                httpGet = null;
            }
            this.response();
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

 



    private void response() {
        String result = "";
        BufferedReader in = null;
        try {
            HttpEntity httpEntity = response.getEntity();
            responseString = EntityUtils.toString(httpEntity);
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    in.close();
                }
            } catch (Exception e2) {
                e2.printStackTrace();
            }
        }

 



    }

 



    public void setPost(String httpUrl) {
        try {
            HttpEntity requestHttpEntity = new UrlEncodedFormEntity(this.getRequestBody());
            httpPost = new HttpPost(httpUrl);
            httpPost.addHeader("Content-Type", "”application/json;charset=UTF-8");
            httpPost.setEntity(requestHttpEntity);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }

 



    public void setGet(String httpUrl) {
        httpGet = new HttpGet(httpUrl);
        httpGet.addHeader("Content-Type", "text/html;charset=UTF-8");
    }

 



    private void getHttpClient() {
        BasicCookieStore cookieStore = new BasicCookieStore();
        CookieSpecProvider easySpecProvider = new CookieSpecProvider() {
            public CookieSpec create(HttpContext context) {

 



                return new BrowserCompatSpec() {
                    @Override
                    public void validate(Cookie cookie, CookieOrigin origin)
                            throws MalformedCookieException {
// Oh, I am easy
                    }
                };
            }


        };
        Registry<CookieSpecProvider> r = RegistryBuilder.<CookieSpecProvider>create()
                .register(CookieSpecs.BEST_MATCH, new BestMatchSpecFactory())
                .register(CookieSpecs.BROWSER_COMPATIBILITY, new BrowserCompatSpecFactory())
                .register("easy", easySpecProvider).build();

 



        RequestConfig requestConfig = RequestConfig.custom().setCookieSpec("easy")
                .setSocketTimeout(20000).setConnectTimeout(20000).build();

 



        httpClient = HttpClients.custom().setDefaultCookieSpecRegistry(r)
                .setDefaultRequestConfig(requestConfig).setDefaultCookieStore(cookieStore).build();
    }

 



    /**
     * httpclient发送http get请求；
     *
     * @return
     * @throws URISyntaxException
     * @throws IOException
     * @throws ClientProtocolException
     */
    public static String sendGet(String uriPath, List<NameValuePair> ns)
            throws URISyntaxException, ClientProtocolException, IOException {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        URIBuilder uri = new URIBuilder();
        uri.setPath(uriPath);
        uri.addParameters(ns);
        URI u = uri.build();
        HttpGet httpget = new HttpGet(u);
        CloseableHttpResponse response = httpclient.execute(httpget);
        return EntityUtils.toString(response.getEntity());
    }

 



    /**
     * httpclient发送http post请求；
     *
     * @return
     * @throws URISyntaxException
     * @throws IOException
     * @throws ClientProtocolException
     */
    public static String sendPost(String uriPath, List<NameValuePair> ns)
            throws URISyntaxException, ClientProtocolException, IOException {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        URIBuilder uri = new URIBuilder();
        uri.setPath(uriPath);
        uri.addParameters(ns);
        URI u = uri.build();
        HttpPost httpPost = new HttpPost(u);
        CloseableHttpResponse response = httpclient.execute(httpPost);
        return EntityUtils.toString(response.getEntity());
    }

}


使用方法如下：

        String signUrl= ”路径“；
     
        try{
        List<NameValuePair> nameValuePairs=new ArrayList<NameValuePair>();
        nameValuePairs.add(new BasicNameValuePair("phone",phone));
        nameValuePairs.add(new BasicNameValuePair("xytId",String.valueOf(xytId)));
        nameValuePairs.add(new BasicNameValuePair("balance",balance));
     
        String result=HttpClientObject.sendPost(signUrl,nameValuePairs);
        logger.info("result: = "+result);
        JSONObject jsonObject=JSON.parseObject(result);
     
        Integer code=Integer.valueOf(jsonObject.getString("code"));
        if(code.equals(0)){
        code1=0L;
        }else if(code.equals(1)){
        code1=1L;
        }else if(code.equals(3)){
        code1=3L;
        }
        }catch(Exception e){
        e.printStackTrace();
        }

