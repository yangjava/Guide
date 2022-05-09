# [Apache Commons CLI命令行启动](https://www.cnblogs.com/xing901022/p/5608823.html)

> 今天又看了下Hangout的源码，一般来说一个开源项目有好几种启动方式——比如可以从命令行启动，也可以从web端启动。今天就看看如何设计命令行启动...

## Apache Commons CLI

Apache Commons CLI是开源的命令行解析工具，它可以帮助开发者快速构建启动命令，并且帮助你组织命令的参数、以及输出列表等。

CLI分为三个过程：

- 定义阶段：在Java代码中定义Optin参数，定义参数、是否需要输入值、简单的描述等
- 解析阶段：应用程序传入参数后，CLI进行解析
- 询问阶段：通过查询CommandLine询问进入到哪个程序分支中

## 举个栗子

#### 定义阶段:

```vbnet
Options options = new Options();
Option opt = new Option("h", "help", false, "Print help");
opt.setRequired(false);
options.addOption(opt);

opt = new Option("c", "configFile", true, "Name server config properties file");
opt.setRequired(false);
options.addOption(opt);

opt = new Option("p", "printConfigItem", false, "Print all config item");
opt.setRequired(false);
options.addOption(opt);
```

其中Option的参数：

- 第一个参数：参数的简单形式
- 第二个参数：参数的复杂形式
- 第三个参数：是否需要额外的输入
- 第四个参数：对参数的描述信息

#### 解析阶段

通过解析器解析参数

```java
CommandLine commandLine = null;
CommandLineParser parser = new PosixParser();
try {
	commandLine = parser.parse(options, args);
}catch(Exception e){
	//TODO xxx
}
```

#### 询问阶段

根据commandLine查询参数，提供服务

```java
HelpFormatter hf = new HelpFormatter();
hf.setWidth(110);

if (commandLine.hasOption('h')) {
	// 打印使用帮助
	hf.printHelp("testApp", options, true);
}
```

## 全部代码样例

```java
package hangout.study;

import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.HelpFormatter;
import org.apache.commons.cli.Option;
import org.apache.commons.cli.Options;
import org.apache.commons.cli.ParseException;
import org.apache.commons.cli.PosixParser;

public class CLITest {
	public static void main(String[] args) {
        String[] arg = { "-h", "-c", "config.xml" };
        testOptions(arg);
    }
    public static void testOptions(String[] args) {
        Options options = new Options();
        Option opt = new Option("h", "help", false, "Print help");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("c", "configFile", true, "Name server config properties file");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("p", "printConfigItem", false, "Print all config item");
        opt.setRequired(false);
        options.addOption(opt);

        HelpFormatter hf = new HelpFormatter();
        hf.setWidth(110);
        CommandLine commandLine = null;
        CommandLineParser parser = new PosixParser();
        try {
            commandLine = parser.parse(options, args);
            if (commandLine.hasOption('h')) {
                // 打印使用帮助
                hf.printHelp("testApp", options, true);
            }

            // 打印opts的名称和值
            System.out.println("--------------------------------------");
            Option[] opts = commandLine.getOptions();
            if (opts != null) {
                for (Option opt1 : opts) {
                    String name = opt1.getLongOpt();
                    String value = commandLine.getOptionValue(name);
                    System.out.println(name + "=>" + value);
                }
            }
        }
        catch (ParseException e) {
            hf.printHelp("testApp", options, true);
        }
    }
}
```

#### 运行结果

```lua
usage: testApp [-c <arg>] [-h] [-p]
 -c,--configFile <arg>   Name server config properties file
 -h,--help               Print help
 -p,--printConfigItem    Print all config item
--------------------------------------
help=>null
configFile=>config.xml
```

## Hangout中的应用

#### 源码片段

```java
package hangout.study;

import org.apache.commons.cli.BasicParser;
import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.HelpFormatter;
import org.apache.commons.cli.Options;
import org.apache.commons.cli.ParseException;

public class HangoutMainTest {
	/**
	 * 解析Hangout参数
	 * 
	 * @param args
	 * @return
	 * @throws ParseException
	 */
	private static CommandLine parseArg(String[] args) throws ParseException {
		//定义阶段
		Options options = new Options();
		options.addOption("h", false, "usage help");
		options.addOption("help", false, "usage help");
		options.addOption("f", true, "configuration file");
		options.addOption("l", true, "log file");
		options.addOption("w", true, "filter worker number");
		options.addOption("v", false, "print info log");
		options.addOption("vv", false, "print debug log");
		options.addOption("vvvv", false, "print trace log");
		//解析阶段
		CommandLineParser paraer = new BasicParser();
		CommandLine cmdLine = paraer.parse(options, args);
		//询问阶段
		if (cmdLine.hasOption("help") || cmdLine.hasOption("h")) {
			/*usage(); //这里作者自定义了帮助信息，其实可以使用helpFormat直接输出的*/
			
			HelpFormatter hf = new HelpFormatter();
	        hf.setWidth(110);
	        hf.printHelp("testApp", options, true);
	        
			System.exit(-1);
		}
		
		// TODO need process invalid arguments
		if (!cmdLine.hasOption("f")) {
			throw new IllegalArgumentException("Required -f argument to specify config file");
		}

		return cmdLine;
	}

	public static void main(String[] args) throws Exception {
		String[] arg = {"-h","xx.file"};//输入参数
		CommandLine cmdLine = parseArg(arg);//解析参数
		System.out.println(cmdLine.getOptionValue("f"));//拿到重要参数
		//TODO
	}
}
```

## 参考

1 [Apache Commons CLI 下载地址](http://mvnrepository.com/artifact/commons-cli/commons-cli/1.3.1)
2 [Apache Commons CLI 官方指南](http://commons.apache.org/proper/commons-cli/usage.html)
3 [IBM 开发者文档](http://www.ibm.com/developerworks/cn/java/j-lo-commonscli/)
4 [CSDN Commons CLI 使用详解](http://blog.csdn.net/faye0412/article/details/2949753)