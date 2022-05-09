# Java实现shell脚本

如何使用Java实现Shell脚本功能?

我们思考:首先应该有个控制台能够输入某个命令,然后shell后台会执行

## pom

```

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <jcommander.version>1.30</jcommander.version>
        <jline.version>2.12</jline.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.beust</groupId>
            <artifactId>jcommander</artifactId>
        </dependency>
        <dependency>
            <groupId>jline</groupId>
            <artifactId>jline</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.beust</groupId>
                <artifactId>jcommander</artifactId>
                <version>${jcommander.version}</version>
            </dependency>
            <dependency>
                <groupId>jline</groupId>
                <artifactId>jline</artifactId>
                <version>${jline.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## jline 初步了解

jline是用java开发的仿shell终端模拟工具包，模拟shell终端，以命令行的方式读取输入信息。

我们这里使用的是*jline*2

读取终端代码如下

```
ConsoleReader consoleReader  = new ConsoleReader();
// 获取命令标识符
consoleReader.setPrompt(prompt);
// 读取行
String result = consoleReader.readLine(prompt);
```

## JCommander

JCommander的网页上指出：“因为生命太短，无法解析命令行参数”，并且概述将JCommander引入为“一个很小的Java框架，使得解析命令行参数变得微不足道。

JCommander使用批注来实现命令行处理的“定义”阶段。

如下

```

public class Main
{
   @Parameter(names={"-v","--verbose"},
              description="Enable verbose logging")
   private boolean verbose;
 
   @Parameter(names={"-f","--file"},
              description="Path and name of file to use",
              required=true)
   private String file;
 
   @Parameter(names={"-h", "--help"},
              description="Help/Usage",
              help=true)
   private boolean help;
 
   // . . .
 
final JCommander commander
   = JCommander.newBuilder()
              .programName("JCommander Demonstration")
             .addObject(this)
             .build();
```

使用JCommander进行“解析”阶段

```
commander.parse(arguments);
```

JCommander的“审讯”阶段

```
if (help)
{
   commander.usage();
}
else
{
   out.println(
      "The file name provided is '" + file + "' and verbosity is set to " + verbose);
}
```

现在是完整的实现Shell的代码

定义shell命令

```
public interface JShellCommand extends Runnable {
}
```

实现shell命令定义

```
@Parameters(commandDescription = "help can show all command ", commandNames = "help")
public class Help implements JShellCommand {

    private JCommander jCommander;

    public Help(JCommander jCommander) {
        this.jCommander = jCommander;
    }

    @Override
    public void run() {
        jCommander.usage();
    }
}
```

实现shell功能

```


    public void test1() throws Exception{

        String prompt=PROMPT_TOKEN;
        jline.TerminalFactory.registerFlavor(TerminalFactory.Flavor.WINDOWS, jline.UnsupportedTerminal.class);
        ConsoleReader consoleReader  = new ConsoleReader();
        consoleReader.setPrompt(prompt);
        while (true){
            String result = consoleReader.readLine(prompt);
            if(result==null){
                System.out.println("no result !!!");
            }
            if(result.isEmpty()){
                continue;
            }
            String[] commandArgs = TOKENIZER.split(result);
            if(commandArgs.length==0){
                System.out.println("no commandArgs !!!");
            }
            JCommander jCommander = new JCommander();
            jCommander.addCommand(new Help(jCommander));
            consoleReader.addCompleter(new StringsCompleter(jCommander.getCommands().keySet()));
            String commandName = commandArgs[0];
            JCommander commandCommander = jCommander.getCommands().get(commandName);
            if (commandCommander == null) {
                System.out.println("no commandCommander !!!");
                return;
            }
            List<Object> objects = commandCommander.getObjects();
            if (objects.isEmpty()) {
                throw new IllegalStateException("Returned object doesn't contain a shell command!");
            }

            if (objects.size() > 1) {
                throw new IllegalStateException("More than one object found in the commander object");
            }

            Object command = objects.get(0);
            jCommander.parse(commandArgs);
            ((Runnable) command).run();
        }

    }
```

参考文档:

commander-shell Java实现的shell命令工具    https://github.com/nikolavp/commander-shell

java编写的模拟shell https://gitee.com/Buynow96/Z_Shell

数据库命令行工具 https://www.oschina.net/p/henplus

