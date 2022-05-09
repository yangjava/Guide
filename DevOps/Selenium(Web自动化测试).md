# Selenium

## Selenium是什么？

　　用官网的一句话来讲：Selenium automates browsers. That's it！简单来讲，Selenium是一个用于Web应用程序自动化测试工具。Selenium测试直接运行在浏览器中，就像真正的用户在操作浏览器一样。支持的浏览器包括IE，Firefox，Safari，Chrome等。Selenium 不仅仅是一个工具或 API，它还组成了许多工具

(以上用了翻译软件，有些翻译不准确，阅读时请自行斟酌)

- WebDriver

  如果你开始使用桌面网站或移动网站测试自动化，那么你将使用 webdriverapi。 Webdriver 使用浏览器厂商提供的浏览器自动化 api 来控制浏览器和运行测试。 这就好像是一个真正的用户在操作浏览器。 由于 WebDriver 不需要使用应用程序代码编译其 API，因此它不具有侵入性。 因此，您测试的应用程序与实时推送的应用程序相同。

- IDE

  Ide (集成开发环境)是您用来开发 Selenium 测试用例的工具。 它是一个易于使用的 Chrome 和 Firefox 扩展，并且通常是开发测试用例的最有效的方法。 它使用现有的 Selenium 命令记录用户在浏览器中的操作，参数由该元素的上下文定义。 这不仅是一个节省时间的方法，也是学习 Selenium 脚本语法的一个很好的方法。

- Grid

  Selenium Grid 允许您跨不同平台在不同的机器上运行测试用例。 触发测试用例的控制位于本地端，当触发测试用例时，它们将由远程端自动执行。

  在 WebDriver 测试开发之后，您可能需要在多个浏览器和操作系统组合上运行测试。 这就是Grid出现的地方。

## Selenium History

　　2004年，诞生了Selenium Core，Selenium Core是基于浏览器并且采用JavaScript编程语言的测试工具，运行在浏览器的安全沙箱中，设计理念是将待测试产品、Selenium Core和测试脚本均部署到同一台服务器上来完成自动化测试的工作。

　　2005年，Selenium RC诞生，就是selenium1 ，这个时候，Selenium Core其实是Selenium RC的核心。Selenium RC让待测试产品、Selenium Core和测试脚本三者分散在不同的服务器上。（测试脚本只关心将HTTP请求发送到指定的URL上，selenium本身不需要关心HTTP请求由于什么程序编程语言编写而成），Selenium RC包括两部分：一个是Selenium RC Server，一个是提供各种编程语言的客户端驱动来编写测试脚本

 　　2007年，Webdriver诞生，WebDriver的设计理念是将端到端测试与底层具体的测试工具分隔离，并采用设计模式Adapter适配器来达到目标。WebDriver的API组织更多的是面向对象。

　　2008年，selenium2诞生，selenium2其实是selenium rc和webdriver的合并，合并的根本原因是相互补充各自的缺点

　　2009年，selenium3诞生，这个版本剔除了selenium rc ， 主要由 selenium webdriver和selenium Grid组成， 我们日常使用的其实就是selenium webdriver，至于selenium grid是一个分布式实现自动化测试的工具

## Selenium原理

本文所讲的Selenium是指Selenium Webdriver，Selenium WebDriver与RC的功能相同，并且包含原始的1.x绑定。它涉及语言绑定和单个浏览器控制代码的实现。这通常被称为“WebDriver”，有时也被称为Selenium 2。Selenium 1.0 + WebDriver = Selenium 2.0

- WebDriver被设计在一个更简单和更简洁的编程接口中，同时解决了Selenium-RC API中的一些限制。

- 与Selenium1.0相比，WebDriver是一个紧凑的面向对象的API

- 它更有效地驱动浏览器，并克服了Selenium 1.x的限制，这影响了我们的功能测试覆盖范围，如文件上传或下载，弹出框和对话框

  在用Selenium进行自动化测试时必须引入相应jar包，比如selenium-server-standalone-2.46.0.jar，selenium-java-2.47.1.jar，3+以上版本可能有所不同，我们看到有个sever这么一个jar包，这个jar包就是Selenium服务，server端可以是任何浏览器作为remote server，职责就是处理client的请求并作出相应操作，client就是我们运行的脚本，response的具体内容根据请求的内容而定，我们以firefox为例，如下图所示

## Selenium工作过程

- selenium client(Java等语言编写的自动化测试脚本)初始化一个service服务，通过Webdriver启动浏览器驱动程序
- 通过RemoteWebDriver向浏览器驱动程序发送HTTP请求，浏览器驱动程序解析请求，打开浏览器，并获得sessionid，如果再次对浏览器操作需携带此id
- 打开浏览器，绑定特定的端口，把启动后的浏览器作为webdriver的remote server
- 打开浏览器后，所有的selenium的操作(访问地址，查找元素等)均通过RemoteConnection链接到remote server，然后使用execute方法调用_request方法通过urlib3向remote server发送请求
- 浏览器通过请求的内容执行对应动作
- 浏览器再把执行的动作结果通过浏览器驱动程序返回给测试脚本

## remote server端的这些功能是如何实现的呢？

　　浏览器实现了webdriver的统一接口，client就可以通过统一的restful的接口去进行浏览器的自动化操作。

　　目前webdriver支持ie, chrome, firefox等主流浏览器，其主要原因是这些浏览器实现了webdriver约定的各种接口。

举个打开浏览器的栗子：

```
package com.Demo;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

public class ExampleForFirefox {
    public static void main(String[] args) {

        System.setProperty("webdriver.firefox.bin", "D:\\Program Files\\Mozilla Firefox 24\\firefox.exe");
        WebDriver driver = new FirefoxDriver();
        System.out.println("https://www.cnblogs.com/mrjade/");
        driver.get("https://www.cnblogs.com/mrjade/");
    
    }
}
```