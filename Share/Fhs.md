# [Fhs 快速开发平台 3.1 发布，借鉴 lombok，玩点新花样](https://www.oschina.net/news/185642/fhs-framework-3-1-released)

来源: 投稿

作者: [shuaizai88](https://my.oschina.net/u/3072142)

2022-03-09

[ 0](https://www.oschina.net/news/185642/fhs-framework-3-1-released#comments)

# 1 本次更新

**1.1 前言**

  相信Myabtis Plus的wrapper大家已经很熟悉了，尤其是 LambdaQueryWrapper比JPA的 Specification好用了不知道多少倍，但是LambdaQueryWrapper写起来还不够行云流水，我一直在想能不能更简化它，于是出现了本次fhs版本更新带来的PO Wrapper增强器功能，他类似lombok，不需要写很多代码就可以对PO进行wrapper化增强配合ActiveRecord简直不要太爽。

**1.2 上DEMO**

 首先我们定义一个PO，并且使用 @Wrapperable注解标记要增强。

```java
@Data
@Wrapperable
@TableName("user")
public class User {

    @TableId("user_id")
    private Integer userId;

    @TableField("name")
    private String name;

    @TableField("age")
    private Integer age;

    @TableField("sex")
    private String sex;


}
```

 然后使用activeRecord愉快的对User表进行CRUD操作。

```java
@RestController
public class UserController {

    @GetMapping("/one")
    public User one() {
        return User.newOBJ().nameLike("小").one();
    }

    @GetMapping("/oneField")
    public User oneField() {
        return User.newOBJ().nameLike("小").one(new String[]{User.USERID, User.NAME});
    }

    @GetMapping("/list")
    public List<User> list() {
        return User.newOBJ().ageBetween(10, 25).list();
    }

    @GetMapping("/listField")
    public List<User> listField() {
        return User.newOBJ().ageBetween(10, 20).list(new String[]{User.USERID, User.NAME});
    }

    @GetMapping("/delete")
    public int delete() {
        return User.newOBJ().ageBetween(50, 80).delete();
    }

    @GetMapping("/count")
    public Long count() {
        return  User.newOBJ().ageBetween(10,26).count();
    }

   
    @GetMapping("/update")
    public int update() {
        User user = User.newOBJ();
        user.setAge(19);
        return user.nameEQ("小明").update();
    }
}
```

支持的方法(注意XX代表属性名，如果有多个属性的话，则会生成多个方法)

| 方法名称              | 方法用途                                                   |
| --------------------- | ---------------------------------------------------------- |
| bean2Wrapper          | 把po转换为一个wrapper                                      |
| list                  | 返回集合                                                   |
| list(String[] fields) | 返回集合 并且指定查询字段                                  |
| one                   | 返回单个，如果条件能查询出多个返回第一个                   |
| one (String[] fields) | 返回单个，如果条件能查询出多个返回第一个，并且指定查询字段 |
| count                 | 总数                                                       |
| delete                | 删除                                                       |
| update                | 更新                                                       |
| newOBJ                | 调用无参构造方法返回一个对象                               |
| xxEQ                  | =                                                          |
| xxNE                  | !=                                                         |
| xxLike                | like                                                       |
| xxLikeRight           | like xx%                                                   |
| xxLikeLeft            | like %xx                                                   |
| xxNotLike             | not like                                                   |
| xxIn                  | in                                                         |
| xxNotIn               | not in                                                     |
| xxBetween             | between                                                    |
| xxNotBetween          | not betwen                                                 |
| xxGE                  | >=                                                         |
| xxGT                  | >                                                          |
| xxLT                  | <                                                          |
| xxLE                  | <=                                                         |
| xxIsNull              | is null                                                    |
| xxNotNull             | not null                                                   |
| xxOrderByAsc          | order by xx asc                                            |
| xxOrderByDesc         | order by xx desc                                           |

除了生成方法外还会生成常量，方便引用属性名。

比如属性中有一个name(小写) 会自动生成一个public static final String NAME="name".

**1.3 所用技术简介**

- Java APT+AST 在编译期对PO的class进行修改，添加我们需要的方法
- idea plugin 生成的class增加了方法和常量后，idea并不能索引到，所以需要开发插件告诉idea我们其实有这些方法，idea就不会报错并且可以自动提示了。
- QueryWrapper 当你执行了nameLike后其实框架是自动调用了 QueryWrapper.like("name","xx")，所以底层还是 QueryWrapper

**1.4 觉得上述特性很好用，但是又不想用我的fhs framework怎么办？**

 我们对此模块进行了单独抽离，https://gitee.com/fhs-opensource/fhs_mp maven中央仓库最新是1.0.2版本，

```xml
 <dependency>
            <groupId>com.fhs-opensource</groupId>
            <artifactId>mp_ext</artifactId>
            <version>1.0.2</version>
 </dependency>
```

 为了方便大家折腾自己的插件，我们也对idea进行了开源：

https://gitee.com/fhs-opensource/myabtis_plus_ext_idea_plugin

 和mp作者聊过，本插件孵化好后可能会合并到mp官方 作为mp4新特性发布。

 

# 2 fhs framework相关

**2.1 FHS V3介绍**

  优秀的国内快速开发平台非常多，FHS V2发布后我们单位内部也做了一番讨论，要不要坚持用自己的轮子，最终决定还是要做。一番讨论后，我们确定了以下几点目标：

-  全面拥抱vue,但是前端技术栈不求最新，但求最快。
-  前端简单的页面使用JSON驱动，简单的CRUD功能代码就算是后端也能开发和维护
-  全面拥抱微服务，但是不能因为SpringCloud的引入带来很大的学习成本(我们的项目有一些新手和实习生在干)
-  提供AllInOne模式，本地开发实现只启动一个SpringBoot 应用即可完成开发调试，test prod环境又可以支持微服务模式部署(俺们单位还有部分机器是E3 1230V2+8G 内存)
-  把部分组件提出来作为单独的开源项目发布，因为我们接的很多项目甲方要求使用他们的平台，我们可以引入几个组件但是不能换平台
-  提供高级查询API，简单的单表查询不要让程序员手动写一行代码。
-  提供基础的组织，用户，角色，菜单，字典，日志，前后端代码生成器
-  在不增加学习成本的情况下，尽可能使用国产开源组件

 

**2.2 FHS V3差异化**

-   **翻译组件** 

​    ![img](https://oscimg.oschina.net/oscnet/up-821f1095527e02eb32c796a301881ac16e8.jpg)

​    字典码 sex 0 需要翻译成男 给前端，userid 1 需要翻译成张三给前端，使用翻译组件，配合Mybatis Plus，无需自己写SQL加一个注解即可实现。

​    翻译组件支持一个项目多个数据源，以及跨微服务进行翻译，还支持传统的枚举翻译。

​    此组件已经单独开源：https://gitee.com/fhs-opensource/easy_trans

-  **简化远程调用**

​    Feign 做远程调用需要首先写一个service方法，然后用controller把service方法包一下，接着写一个feign接口，最后使用，而且对SpringCloud依赖性比较强，无法实现我们的All In One目标，我们希望把这个过程简化掉，于是EasyCloud模块出现了，只需要在service层加个注解就可以把普通的service方法暴漏出去给别人调用。

```java
@CloudApi(serviceName="producer")//producer 是服务提供者的服务名称
public interface UserService {

    @CloudMethod  //加此注解意思是此方法提供给其他微服务调用
    List<UserDto> listByIds(String[] ids);
}
```

  我要使用的时候只需要依赖User微服务的接口和pojo，然后进行注入即可。

```java
 @Autowired
 private UserService userService;
```

   通过EasyCloud，我们的开发者只需要多记2个注解即可，没有额外学习成本，本组件已经单独抽离开源：https://gitee.com/fhs-opensource/easy_cloud

-  **JSON驱动VUE插件集-PAGEX**

   常年逛开源中国的开发者应该对avue和amis比较熟悉，他们的star数量就能看出越来越多的前端开发者已经接受了JSON驱动，我们对组件的理解是：高度抽象，搭配后台API设计规范，开发者通过JSON告诉组件，我需要一个下拉框/多选框/Other，你的数据绑定到表单对象的哪个属性上，你这个组件的数据源URL是什么，你这个组件数据校验规则是什么于是我们写一个下拉tree是如下代码：

```json
{
    type: "treeSelect",
    name: "organizationId",
    label: "部门",
    rule: "required",
    query: {},
    api: '/basic/ms/sysOrganization/tree',
    selectOn: (node) => {
        this.changeRoleSelect(node.id);
    }
}
```

  又比如列表的过滤条件，我只需要告诉组件，我需要input，他的title是分组编码，对应字段是code，我需要模糊匹配。

```json
filters: [
    {label: '分组名称:', name: 'groupName', placeholder: "分组名称", type: 'text', operation: 'like'},
    {label: '分组编码:', name: 'groupCode', placeholder: "分组编码", type: 'text', operation: '='}
  ]
```

  配合高级查询API规范，需求变动加过滤字段，更改数据过滤操作符前端自己就搞了。当然我们也考虑了很多扩展，这里就不示例了。

  一个JSON驱动的CRUD 的代码demo。 https://gitee.com/fhs-opensource/fhs-framework/blob/v3.x/vue_ui/src/views/system/dict/index.vue

-  **AllInOne 模式**

  all in one的好处是省内存，调试方便，在新建微服务的时候需要注意业务代码和springboot启动类分离开，比如我有用户中心，订单2个微服务，那么使用all in one需要user,userApp,order,orderApp,allinoneApp 这么几个子模块，其中spingboot相关依赖和配置放到userApp和orderApp上，业务代码和springcloud 相关类解耦即可。

-  **高级查询API**

   高级查询API很多项目已经有了，配合前端PAGEX的CRUD组件，可以更简单灵活的构造列表查询参数。

```json
 {
	"groupRelation": "AND",
	"pagerInfo": {
		"page": 1,  //第几页，从1开始
		"pageSize":10//每页多少条
	},
	"params": {},//扩展参数，具体要和后端约定
	"querys": [
		{
			"group": "main",//分组
			"operation": "=",//操作符
			"property": "userName",//Java属性名
			"relation": "AND",//是and 还是or
			"value": "wanglei" // 属性username=wanglei的查询出来
		}
	],
	"sorter": [
		{
			"direction": "ASC",//ASC 从小到大排序，DESC 从大到小排序
			"property": "createTime"//排序的字段名
		}
	]
}
```

-  **Mybatis Plus+PO增强 简化查询条件构造**

   为本次主要更新内容，这里不多说了

 

**4 体验地址**

[http://82.157.62.164/login](https://gitee.com/link?target=http%3A%2F%2F82.157.62.164%2Flogin) admin 123456

**5 预览图**

![img](https://gitee.com/fhs-opensource/fhs-framework/raw/v3.x/img/fhs.jpg)

**5 用到的国产组件集**

- Mybatis Plus
- Sa-Token
- Validate-Springboot-Starter
- SpringCloud Alibaba
- ip2region
- knife4j