# Kettle

## 概述

### 什么是ETL?

```
ETL（Extract-Transform-Load的缩写，即数据抽取、转换、装载的过程），对于企业或行业应用来说，我们经常会遇到各种数据的处理，转换，迁移，所以了解并掌握一种etl工具的使用，必不可少，这里我要学习的ETL工具是Kettle！
```

### 什么是Kettle？

```
Kettle是一款国外开源的ETL工具，纯java编写，可以在Window、Linux、Unix上运行，绿色无需安装，数据抽取高效稳定。
Kettle 中文名称叫水壶，该项目的主程序员MATT 希望把各种数据放到一个壶里，然后以一种指定的格式流出。
Kettle这个ETL工具集，它允许你管理来自不同数据库的数据，通过提供一个图形化的用户环境来描述你想做什么，而不是你想怎么做。
Kettle中有两种脚本文件，transformation和job，transformation完成针对数据的基础转换，job则完成整个工作流的控制。
Kettle(现在已经更名为PDI，Pentaho Data Integration-Pentaho数据集成)。
```

### kettle的核心组件

| 名称    | 描述                                                 |
| ------- | ---------------------------------------------------- |
| Spoon   | 通过图形接口,用于编辑作业和转换的桌面应用            |
| Pan     | 一个独立的命令行程序,用于执行由Spoon编辑的转换和作业 |
| Kitchen | 一个独立的命令行程序,用于执行由Spoon编辑的作业       |
| Carte   | 一个轻量级的web容器,用于建立专用,远程ETL Server      |

### Kettle的下载

Kettle官方网址

```
https://community.hitachivantara.com/s/article/data-integration-kettle
```

Kettle的国内镜像

```
http://mirror.bit.edu.cn/pentaho/Data%20Integration/
```

### Kettle的部署

```
由于Kettle是Java语言开发的，该软件的允许需要Java运行环境的依赖。需要先安装JDK,准备好Java软件的运行环境。
在Window10环境下，双击Spoon.bat即可运行了。
```

## 使用kettle

Kettle实现，把数据从CSV文件复制到Excel文件。

```
首先，创建一个转换，找到核心对象，找到输入里面的CVS文件输入图元，拖拽到工作区域，双击CVS文件输入。

可以修改步骤的名称，点击浏览，选择到CVS文件，其他参数可以默认，点击获取字段，最后点击确定。

CVS文件输入配置完毕以后，可以配置Excel输出

此时，可以 按住shift拖动鼠标，划线，将CVS文件输入和Excel输出连到一起。

最后，点击Excel输出，选择字段，点击获取字段，将输出到Excel的字段进行映射，最后点击确定即可。
```

Java使用

配置Maven仓库

```
<repositories><!-- kettle中央仓库 -->
	<repository>
		<id>pentaho-public</id>
		<name>Pentaho Public</name>
		<url>http://nexus.pentaho.org/content/groups/omni</url>
		<releases>
			<enabled>true</enabled>
			<updatePolicy>always</updatePolicy>
		</releases>
		<snapshots>
			<enabled>true</enabled>
			<updatePolicy>always</updatePolicy>
		</snapshots>
	</repository>
</repositories>
```



pom文件

```
<kettle.version>9.3.0.0-13</kettle.version>

```



```
       <dependency>
            <groupId>org.pentaho</groupId>
            <artifactId>kettle-core</artifactId>
            <version>${kettle.version}</version>
        </dependency>
        <dependency>
            <groupId>org.pentaho</groupId>
            <artifactId>kettle-engine</artifactId>
            <version>${kettle.version}</version>
        </dependency>
        <dependency>
            <groupId>org.pentaho</groupId>
            <artifactId>metastore</artifactId>
            <version>${kettle.version}</version>
        </dependency>
        <dependency>
            <groupId>org.pentaho</groupId>
            <artifactId>pentaho-vfs-browser</artifactId>
            <version>${kettle.version}</version>
        </dependency>
```

代码

```
public class kettleDemo {

    public static void main(String[] args) throws KettleException {

        KettleEnvironment.init();
        // 创建DB资源库
        KettleDatabaseRepository repository = new KettleDatabaseRepository();
        DatabaseMeta databaseMeta = setDatabaseMeta();
        // 选择资源库
        KettleDatabaseRepositoryMeta kettleDatabaseRepositoryMeta = new KettleDatabaseRepositoryMeta("Kettle", "Kettle", "Transformation description", databaseMeta);
        repository.init(kettleDatabaseRepositoryMeta);
        // 连接资源库
        repository.connect("root", "root");
        RepositoryDirectoryInterface directoryInterface = repository.loadRepositoryDirectoryTree();
        System.out.println(directoryInterface);// 根节点是 “/”
        List<RepositoryDirectoryInterface> children = directoryInterface.getChildren();
        for (RepositoryDirectoryInterface re : children) {
            System.out.println(re);
        }
        execJob(repository);

    }

    /**
     * 执行 作业
     *
     * @param repository
     * @throws KettleException
     */
    private static void execJob(KettleDatabaseRepository repository) throws KettleException {
        RepositoryDirectoryInterface dir = repository.findDirectory("/201811_JOB");
        JobMeta jobMeta = repository.loadJob(repository.getJobId("A_邮件job", dir), null);
        Job job = new Job(repository, jobMeta);
        // 设置参数
        // jobMeta.setParameterValue("method", "update");

        job.setLogLevel(LogLevel.DETAILED);
        // 启动执行指定的job
        job.start();

        job.waitUntilFinished();// 等待job执行完；
        job.setFinished(true);
        System.out.println(job.getResult());

    }

    /**
     * 执行转换
     *
     * @param repository
     * @throws KettleException
     */
    @SuppressWarnings("unused")
    private static void execTran(KettleDatabaseRepository repository) throws KettleException {
        // 根据指定的字符串路径 找到目录
        RepositoryDirectoryInterface dir = repository.findDirectory("/201811_KTR/A_KTR");
        // 在 指定目录下找转换
        TransMeta tmeta = repository.loadTransformation(repository.getTransformationID("A_TRAN", dir), null);
        // 设置参数
        // tmeta.setParameterValue("", "");
        Trans trans = new Trans(tmeta);
        trans.execute(null);// 执行trans
        trans.waitUntilFinished();// 等待直到数据结束
        if (trans.getErrors() > 0) {
            System.out.println("transformation error");
        } else {
            System.out.println("transformation successfully");
        }
    }

    /**
     * 设置kettle资源库连接信息
     *
     * @return
     */
    private static DatabaseMeta setDatabaseMeta() {
        return new DatabaseMeta(PropUtils.getConfigValByKey("name"), PropUtils.getConfigValByKey("type"), PropUtils.getConfigValByKey("access"), PropUtils.getConfigValByKey("host"), PropUtils.getConfigValByKey("db"), PropUtils.getConfigValByKey("port"), PropUtils.getConfigValByKey("user"), PropUtils.getConfigValByKey("pass"));
    }
}
```

