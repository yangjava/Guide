GraphQL

GraphQL 是一种针对 Graph（图状数据）进行查询特别有优势的 Query Language（查询语言），所以叫做 GraphQL。它跟 SQL 的关系是共用 QL 后缀，就好像「汉语」和「英语」共用后缀一样，但他们本质上是不同的语言。GraphQL 跟用作存储的 NoSQL 没有必然联系，虽然 GraphQL 背后的实际存储可以选择 NoSQL 类型的数据库，但也可以用 SQL 类型的数据库，或者任意其它存储方式（例如文本文件、存内存里等等）。

GraphQL 最大的优势是查询图状数据。GraphQL 是 Facebook 发明的，我可以用 Facebook 做例子。例如说，你要在 Facebook 上打开我的页面查看我的信息，你需要请求如下信息：

- 我的名字

- 我的头像

- 我的好友（按他们跟你的亲疏程度排序取前 6）：

- - 好友 1 的名字、头像及链接
  - 好友 2 的名字、头像及链接
  - ……

- 我的照片（按时间倒序排序取前 6）：

- - [照片](https://www.zhihu.com/search?q=照片&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A949588861}) 1 及其链接
  - 照片 2 及其链接
  - ……

- 我的帖子（按时间倒序排序）：

- - 帖子 1：

  - - 帖子 1 内容

    - 帖子 1 评论：

    - - 帖子 1 评论 1：

      - - 帖子 1 评论 1 内容
        - 帖子 1 评论 1 作者名字
        - 帖子 1 评论 1 作者头像

      - 帖子 1 评论 2：

      - - ……

      - ……

  - 帖子 2：

  - - 帖子 2 内容

    - 帖子 2 评论：

    - - ……

  - ……

这是一个超级复杂的树状结构，如果我们用常见的 RESTful API 涉及，每个 API 负责请求一种类型的对象，例如用户是一个类型，帖子是另一个类型，那就需要非常多个请求才能把这个页面所需的所有数据拿回来。而且这些请求直接还存在依赖关系，不能平行地发多个请求，例如说在获得帖子数据之前，无法请求评论数据；在获得评论数据之后，才能开始请求评论作者数据。

如何解决这种问题？一个简单粗暴的办法是专门写一个 RESTful API，请求上述树状复杂数据。但很快新问题就会出现。现在 Facebook 想要做一个新的产品，例如说是宠物，然后要在我的页面上显示我的宠物信息，那这个 RESTful API 的实现就要跟着改。

GraphQL 能够很好地解决这个问题，但前提是数据已经以图的数据结构进行保存。例如上面说到的用户、帖子、评论是顶点，而用户跟用户发过的帖子存在边的关系，帖子跟帖子评论存在一对多的边，评论跟评论作者存在一对一的边。这时候如果新产品引入了新的对象类型（也就是顶点类型）和新的边类型，那没有关系。在查询数据时用 GraphQL 描述一下要查询的这些边和顶点就行，不需要去改 API 实现。

------

说完了 GraphQL 是什么和能解决什么问题，说说不够好的地方吧。

第一，Facebook 从来没有公开自己的 GraphQL 后端设计，使得大家必需要用第三方的，但体验显然不如我们在 Facebook 内部使用 GraphQL 好。我上面说了，数据必需已经以图的数据结构进行存储才有优势。Facebook 内部有非常好的后端做好了这件事情，而且还内置了基于隐私设置的访问控制。例如说你发的帖子有些是所有人可见的、有些是好友可见的、有些是仅同事可见的，我在打开你的页面时 Facebook 有一个[中间层](https://www.zhihu.com/search?q=中间层&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A949588861})保证了根据我和你的关系我只能看到我该看到的帖子。GraphQL 在这一层之上，所以无论 GraphQL 怎么写我都不可能看到我不该看到的信息。

第二，并不是所有场景都适用于 GraphQL 的，有些很简单的事情就应该用 RESTful API 来实现。Facebook 内部用户增长部门的很多 API 都还不是 GraphQL，因为没必要迁移到 GraphQL。用户增长部门的 API 处理新用户注册、填写短信验证码之类的事情，这些事情都是围绕着一个用户的具体某项或多项信息发生的，根本没有任何图的概念。可以强行写作 GraphQL，但得不到显著的好处。既然老的 API 早就写好了，需要的时候做一些小改动，但没必要重写。

第三，GraphQL 尽管查询的数据是图状数据结构，但实际获得的数据视图是树状数据结构。每一个 GraphQL 查询或更新都有自己的根节点，然后所有的数据都是从根结点展开出去的。查询后获得的数据如果要在前端重新变回图的状态，那前端就不能简单地缓存查询得到的数据，必须用对用的 GraphQL 存储库，然后通过顶点的 ID 把不同节点之间的某些边重新连接起来。

# 开源项目/GraphQL(2)-spring Boot 实现

# spring Boot 实现

GraphQL标准提供了多种主流编程语言的实现，本文介绍其中的Java实现graphql-java。所谓GraphQL标准的实现，就是一个GraphQL服务器的实现，通过一组软件组件，以支持请求的解析、验证和执行GraphQL查询。

graphql-java就是GraphQL标准的Java语言实现之一。在Maven项目中配置如下：

```
<dependency>
			<groupId>com.graphql-java</groupId>
			<artifactId>graphql-java</artifactId>
			<version>15.0</version>
</dependency>
```

官网提供了一个Demo

```
import graphql.ExecutionResult;
import graphql.GraphQL;
import graphql.schema.GraphQLSchema;
import graphql.schema.StaticDataFetcher;
import graphql.schema.idl.RuntimeWiring;
import graphql.schema.idl.SchemaGenerator;
import graphql.schema.idl.SchemaParser;
import graphql.schema.idl.TypeDefinitionRegistry;

import static graphql.schema.idl.RuntimeWiring.newRuntimeWiring;

public class HelloWorld {

    public static void main(String[] args) {
        //定义schema文件，直接写在了代码中，包含一个hello的查询方法
        String schema = "type Query{hello: String} schema{query: Query}";
        SchemaParser schemaParser = new SchemaParser();
        //直接加载schema，初始化GraphQL
        TypeDefinitionRegistry typeDefinitionRegistry = schemaParser.parse(schema);
        //加载一份服务端数据
        RuntimeWiring runtimeWiring = new RuntimeWiring()
                .type("Query", builder -> builder.dataFetcher("hello", new StaticDataFetcher("world")))
                .build();

        SchemaGenerator schemaGenerator = new SchemaGenerator();
        GraphQLSchema graphQLSchema = schemaGenerator.makeExecutableSchema(typeDefinitionRegistry, runtimeWiring);
        // 构建一个GraphQL实例，执行graphql脚本
        GraphQL build = GraphQL.newGraphQL(graphQLSchema).build();
        ExecutionResult executionResult = build.execute("{hello}");

        System.out.println(executionResult.getData().toString());
        // Prints: {hello=world}
    }
}

```

## graphql-java的核心

### GraphQL Schema

就类似于数据库服务器要响应用户对数据库的访问，首先要定义数据库Schema一样。GraphQL服务器要是响应用户对REST数据的查询请求，也要定义一个描述服务数据的结构，就是GraphQL Schema。

为了定义GraphQL Schema，graphql-java提供了如下两种定义的方式：

代码中编程定义动态GraphQL Schema

```
GraphQLObjectType fooType = newObject()
    .name("Ci")
    .field(newFieldDefinition()
            .name("product_number")
            .type(GraphQLString))
    .build();
```

如代码所示，运行中创建了新类型Ci，并为其定义了属性product_number，该属性的类型为Scalar的GraphQLString。

- 使用SDL (Schema Definition Language)在独立的资源文件中定义静态GraphQL Schema


创建src/main/resources/myschema.graphqls文件，定义类型Ci如下：

```
type Ci {
    product_number: String
}
```

为了示例，两种方式定义的GraphQL Schema完全等价。

### DataFetcher/TypeResolver

GraphQL Schema中的每个字段类型，都关联一个DataFetcher（在Node.js实现或Python实现中被称为resolver函数），用以为GraphQL Schema中的字段类型获取数据。数据源可以是数据库或REST服务，并将数据表示为Java POJO对象。DataFetcher对象可以直接读取Java POJO对象的属性，再使用Jackson或GSON转换为JSON格式的数据。

```
DataFetcher pnDataFetcher = new DataFetcher() {
    @Override
    public Object get(DataFetchingEnvironment environment) {
        Map response = fetchPnFromDatabase(environment.getArgument("product_number"));
		List<GraphQLError> errors = ((List)response.get("errors")).stream()
			.map(MyMapGraphQLError::new)
			.collect(Collectors.toList());
		return new DataFetcherResult(response.get("data"), errors);
    }
};

```



```
GraphQLObjectType objectType = newObject()
        .name("Ci")
        .field(newFieldDefinition()
                .name("product_number")
                .type(GraphQLString)
                .dataFetcher(pnDataFetcher))
        .build();
```

与DataFetcher获取数据的作用相似，TypeResolver提供了更强大的功能，能够根据获取的数据转换为对应的Schema中定义的字段类型，这对面向接口的编程有很大的意义。在一个获取数据的方法中，定义返回的类型是接口，而实际返回可能是实现该接口的任何对象，根据获取的数据转换为对应的对象，返回即可。

### Runtime Wiring

创建完毕GraphQL Schema还只是开始，还要将其应用在GraphQL服务器上。这就类似于定义数据库结构的SDL脚本已经准备就绪，但是真正在数据库服务器上执行后才生效。

在GraphQL服务器上真正执行GraphQL Schema使其运行起来，是通过Runtime Wiring，这样才能够得到一个可执行的GraphQL Schema。典型代码如下：


```
File schemaFile = loadSchema("myschema.graphqls");
 
SchemaParser schemaParser = new SchemaParser();
TypeDefinitionRegistry typeRegistry = schemaParser.parse(schemaFile);
 
RuntimeWiring wiring = RuntimeWiring.newRuntimeWiring()....build();
SchemaGenerator schemaGenerator = new SchemaGenerator();
GraphQLSchema graphQLSchema = schemaGenerator.makeExecutableSchema(typeRegistry, wiring);

```

### GraphQL服务

```
GraphQL build = GraphQL.newGraphQL(graphQLSchema).build();
```

### Execution请求GraphQL服务

请求参数

在请求应用中，可以直接将请求参数以字符串的形式传递给GraphQL服务，如下所示：

```
ExecutionResult executionResult = build.execute("query { hero { name } }");

```

也可以先构建ExecutionInput对象（适合复杂的请求参数），然后再传递给GraphQL服务，如下所示：

```
ExecutionInput executionInput = ExecutionInput.newExecutionInput().query("query { hero { name } }")
        .build();
ExecutionResult executionResult = build.execute(executionInput);
```

处理结果

GraphQL服务的响应结果在data中，如果出错则在

```
executionResult.getData().toString();
List<GraphQLError> errors = executionResult.getErrors();
```

如果要将得到的响应数据序列化为JSON格式，为了完全兼容GraphQL标准，建议首先将响应结果进行规范化转换，然后再调用Jackson或GSON等JSON库进行JSON格式转换。

```
Map<String, Object> toSpecificationResult = executionResult.toSpecification();
doWithJson(toSpecificationResult);
```

