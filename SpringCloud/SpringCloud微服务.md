# 概述

## Springcloud

Spring Cloud 是一系列框架的有序集合，它利用 Spring Boot 的开发便利性简化了分布式系统的开发，比如服务发现、服务网关、服务路由、链路追踪等。Spring Cloud 并不重复造轮子，而是将市面上开发得比较好的模块集成进去，进行封装，从而减少了各模块的开发成本。换句话说：Spring Cloud 提供了构建分布式系统所需的“全家桶”



## 微服务

微服务架构（Microservice Architecture）是一种架构概念，旨在通过将功能分解到各个离散的服务中以实现对解决方案的解耦。你可以将其看作是在架构层次而非获取服务的

类上应用很多SOLID原则。微服务架构是个很有趣的概念，它的主要作用是将功能分解到离散的各个服务当中，从而降低系统的耦合性，并提供更加灵活的服务支持。

**概念：** 把一个大型的单个应用程序和服务拆分为数个甚至数十个的支持微服务，它可扩展单个组件而不是整个的应用程序堆栈，从而满足服务等级协议。



# MyBatis-Plus

## 快速入门

1. 引入依赖(maven)：

```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
            <version>3.5.9</version>
        </dependency>
        <!-- springBoot3依赖 -->
```



2. 自定义的Mapper继承MybatisPlus提供的BaseMapper接口：

```java
public interface UserMapper extends BaseMapper<User> {}
```



### 常见注解

> MyBatisPlus通过扫描实体类，并基于反射获取实体类信息作为数据库表信息。

约定大于配置：

- 类名驼峰转下划线作为表名

- 名为id的字段作为主键

- 变量名驼峰转下划线作为表的字段名
  
  

MybatisPlus中比较常用的几个注解如下(主要用于不满足约定的情况)：

- `@TableName`：用来指定表名

- `@TableId`：用来指定表中的主键字段信息

- `@TableField`：用来指定表中的普通字段信息

```java
@TableName("user")
public class User {

    /**
     * 用户id
     */
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    /**
     * 用户名
     */
    @TableField("username")
    private String username;
}
```

IdType枚举（建议指定）：

- `AUTO`：数据库自增长

- `INPUT`：通过set方法自行输入

- `ASSIGN_ID`：分配 ID，接口`IdentifierGenerator`的方法nextId来生成id，默认实现类为`DefaultIdentifierGenerator`雪花算法
  
  

使用@TableField的常见场景：

- 成员变量名与数据库字段名不一致

- 成员变量名<font color=red>以is开头，且是布尔值</font>

- 成员变量名与数据库**关键字冲突**    如@TableField("\`order\`")  反引号

- 成员变量不是数据库字段
  
  

### 常用配置

```yml
mybatis-plus:
  type-aliases-package: com.ysh.entity #别名扫描包
  mapper-locations: classpath:mapper/**/*.xml #mapper扫描路径
  configuration:
    map-underscore-to-camel-case: true #开启驼峰命名规则
    cache-enabled: false #关闭Mybatis二级缓存
  global-config:
    db-config:
      id-type: assign_id #主键类型为雪花算法（全局配置）注解配置高于全局配置
      update-strategy: not_null #更新策略:只更新不为null的值
```

具体可参考官方文档：[使用配置](https://www.baomidou.com/pages/56bac0/) [|](https://www.baomidou.com/pages/56bac0/) [MyBatis](https://www.baomidou.com/pages/56bac0/)[-Plus(baomidou.com)](https://www.baomidou.com/pages/56bac0/)



## 核心功能

### 条件构造器

![f17d017e-40ff-4b5b-a00b-d03ee45aefbe](./images/f17d017e-40ff-4b5b-a00b-d03ee45aefbe.png)

需求：

1. 查询出名字中带o的，存款大于等于1000元的人的id、username、info、balance字段
   
   ```sql
   # 原始SQL语句
   SELECT id,username,info,balance
   FROM user 
   WHERE username LIKE ? AND balance >= ?
   ```

```java
        // 构建查询条件
        QueryWrapper<User> wrapper = new QueryWrapper<User>()
                .select("id", "username", "info", "balance")
                .like("username", "o")
                .ge("balance", 1000);

        // 执行查询
        userMapper.selectList(wrapper);
```



2. 更新用户名为jack的用户的余额为2000
   
   ```sql
   UPDATE user
   SET balance = 2000
   WHERE (username = "jack")
   ```

```java
        User user = new User();
        user.setBalance(2000);

        QueryWrapper<User> wrapper = new QueryWrapper<User>()
                .eq("username", "jack");

        userMapper.update(user, wrapper);
```



3. 更新id为1,2,4的用户的余额，扣200

```sql
UPDATE user
SET balance = balance - 200
WHERE id in (1, 2, 4)
```

```java
        List<Long> ids = List.of(1L, 2L, 4L);
        UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
                .setSql("balance = balance - 200")
                .in("id", ids);
        userMapper.update(null,wrapper);
```



4. LambdaWrapper

```java
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
                .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
                .like(User::getUsername, "o")
                .ge(User::getBalance, 1000);
        userMapper.selectList(wrapper).forEach(System.out::println);
```



总结：

条件构造器的用法：

- QueryWrapper和LambdaQueryWrapper通常用来构建select、delete、update的where条件部分

- UpdateWrapper和LambdaUpdateWrapper通常只有在set语句比较特殊才使用

- 尽量使用LambdaQueryWrapper和LambdaUpdateWrapper，避免硬编码
  
  

### 自定义SQl

我们可以利用MyBatisPlus的Wrapper来构建复杂的Where条件，然后自己定义SQL语句中剩下的部分。



需求：将id在指定范围的用户（例如1、2、4 ）的余额扣减指定值

1. 基于Wrapper构建where条件

```java
        List<Long> ids = List.of(1L, 2L, 4L);
        int amount = 200;
        // 1.构建条件
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>().in(User::getId, ids);
        //2.自定义SQL方法调用
        userMapper.updateBalanceByIds(wrapper, amount);
```

2. 在mapper方法参数中用Param注解声明wrapper变量名称，必须是ew

```java
void updateBalanceByIds(@Param("ew") LambdaQueryWrapper<User> wrapper, @Param("amount") int amount);
```

> Wrapper参数必须指定为“ew”

3. 自定义SQL，并使用Wrapper条件

```xml
    <update id="updateBalanceByIds">
        UPDATE tb_user SET balance = balance - #{amount} ${ew.customSqlSegment}
    </update>
```





### Service接口

#### 基本使用

![cf661e6a-b8e4-4e89-89f4-9e0b77b4e330](./images/cf661e6a-b8e4-4e89-89f4-9e0b77b4e330.png)

   使用步骤：

1. 自定义Service接口继承IService接口

```java
public interface IUserService extends IService<User> {
}
```

2. 自定义Service实现类，实现自定义接口并继承ServiceImpl类

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
}
```



#### Lambda

IService中还提供了**Lambda**功能来简化我们的复杂查询及更新功能。

**案例一**：实现一个根据复杂条件查询用户的接口，查询条件如下：

* name：用户名关键字，可以为空

* status：用户状态，可以为空

* minBalance：最小余额，可以为空

* maxBalance：最大余额，可以为空

可以理解成一个用户的后台管理界面，管理员可以自己选择条件来筛选用户，因此上述条件不一定存在，需要做判断。



```java
    List<User> users = userService.lambdaQuery()
            .like(username != null, User::getUsername, username)
            .eq(status != null, User::getStatus, status)
            .ge(minBalance != null, User::getBalance, minBalance)
            .le(maxBalance != null, User::getBalance, maxBalance)
            .list();
```

> 在组织查询条件的时候，我们加入了 `username != null` 这样的参数，意思就是当条件成立时才会添加这个查询条件

可以发现lambdaQuery方法中除了可以构建条件，还需要在链式编程的最后添加一个`list()`，这是在告诉MP我们的调用结果需要是一个list集合。这里不仅可以用`list()`，可选的方法有：

* `.one()`：最多1个结果

* `.list()`：返回集合结果

* `.count()`：返回计数结果

MybatisPlus会根据链式编程的最后一个方法来判断最终的返回结果。



根据id修改用户余额

```java
lambdaUpdate()
            .set(User::getBalance, remainBalance) // 更新余额
            .set(remainBalance == 0, User::getStatus, 2) // 动态判断，是否更新status
            .eq(User::getId, id)
            .eq(User::getBalance, user.getBalance()) // 乐观锁，解决并发问题
            .update();
```



#### 批量新增

测试逐条插入数据，效率极低。



然后再试试MybatisPlus的批处理：

```java
@Test
void testSaveBatch() {
    // 准备10万条数据
    List<User> list = new ArrayList<>(1000);
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        list.add(buildUser(i));
        // 每1000条批量插入一次
        if (i % 1000 == 0) {
            userService.saveBatch(list);
            list.clear();
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("耗时：" + (e - b));
}
```

> 比逐条新增效率提高了10倍左右



`MybatisPlus`的批处理是基于`PrepareStatement`的预编译模式，然后批量提交，最终在数据库执行时还是会有多条insert语句，逐条插入数据。SQL类似这样：

```sql
Preparing: INSERT INTO user ( username, password, phone, info, balance, create_time, update_time ) VALUES ( ?, ?, ?, ?, ?, ?, ? )
Parameters: user_1, 123, 18688190001, "", 2000, 2023-07-01, 2023-07-01
Parameters: user_2, 123, 18688190002, "", 2000, 2023-07-01, 2023-07-01
Parameters: user_3, 123, 18688190003, "", 2000, 2023-07-01, 2023-07-01
```

而如果想要得到最佳性能，最好是将多条SQL合并为一条，像这样：

```sql
INSERT INTO user ( username, password, phone, info, balance, create_time, update_time )
VALUES 
(user_1, 123, 18688190001, "", 2000, 2023-07-01, 2023-07-01),
(user_2, 123, 18688190002, "", 2000, 2023-07-01, 2023-07-01),
(user_3, 123, 18688190003, "", 2000, 2023-07-01, 2023-07-01),
(user_4, 123, 18688190004, "", 2000, 2023-07-01, 2023-07-01);
```



**解决方案：**

**MySQL**的客户端连接参数中有这样的一个参数：`rewriteBatchedStatements`。顾名思义，就是重写批处理的`statement`语句。

> 这个参数的默认值是false，我们需要修改连接参数，将其配置为true

修改项目中的application.yml文件，在jdbc的url后面添加参数`&rewriteBatchedStatements=true`

```yml
spring:
  datasource:
    url: jdbc:mysql://8.138.186.154:3306/mp?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
```



## 扩展功能

### 代码生成

在使用MybatisPlus以后，基础的`Mapper`、`Service`、`PO`代码相对固定，重复编写也比较麻烦。因此MybatisPlus官方提供了代码生成器根据数据库表结构生成`PO`、`Mapper`、`Service`等相关代码。只不过代码生成器同样要编码使用，也很麻烦。

推荐使用一款`MybatisPlus`的插件(IDEA插件)，它可以基于图形化界面完成`MybatisPlus`的代码生成，非常简单。

![c228f3a6-b4a3-4209-9bf1-e9da9ca24eb7](./images/c228f3a6-b4a3-4209-9bf1-e9da9ca24eb7.png)



### 静态工具

有的时候Service之间也会相互调用，为了**避免出现循环依赖问题**，MybatisPlus提供一个静态工具类：`Db`，其中的一些静态方法与`IService`中方法签名基本一致，也可以帮助我们实现CRUD功能：

![246f37d2-f85c-4e93-a656-72380da78ba9](./images/246f37d2-f85c-4e93-a656-72380da78ba9.png)

示例：

```java
@Test
void testDbGet() {
    User user = Db.getById(1L, User.class);
    System.out.println(user);
}

@Test
void testDbList() {
    // 利用Db实现复杂条件查询
    List<User> list = Db.lambdaQuery(User.class)
            .like(User::getUsername, "o")
            .ge(User::getBalance, 1000)
            .list();
    list.forEach(System.out::println);
}

@Test
void testDbUpdate() {
    Db.lambdaUpdate(User.class)
            .set(User::getBalance, 2000)
            .eq(User::getUsername, "Rose");
}
```



### 逻辑删除

对于一些比较重要的数据，我们往往会采用逻辑删除的方案，即：

* 在表中添加一个字段标记数据是否被删除

* 当删除数据时把标记置为true

* 查询时过滤掉标记为true的数据
  
  > 一旦采用了逻辑删除，所有的查询和删除逻辑都要跟着变化，非常麻烦。
  
  

  为了解决这个问题，MybatisPlus就添加了对逻辑删除的支持。

> **注意**，只有MybatisPlus生成的SQL语句才支持自动的逻辑删除，自定义SQL需要自己手动处理逻辑删除。





例如，我们给`address`表添加一个逻辑删除字段：

```sql
alter table address add deleted bit default b'0' null comment '逻辑删除';
```



然后给`Address`实体添加`deleted`字段(Boolean类型)：



接下来，我们要在`application.yml`中配置逻辑删除字段：

```yml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```



方法与普通删除一模一样，但是底层的SQL逻辑变了：

![](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NzFkMGFjMGU5MGFkYmZlYWY4MWUxMDkyMDdjYjc2ZDNfNEd4NTQzY1UxRTJ2SnN6NHRlU2t3V0ZHNGlJUDR1UTlfVG9rZW46VHdXaWJOTUlrb1BkNnl4WGYxbWNvamRqbnNiXzE3MzM1Nzk0MjE6MTczMzU4MzAyMV9WNA)



综上， 开启了逻辑删除功能以后，我们就可以像普通删除一样做CRUD，基本不用考虑代码逻辑问题。还是非常方便的。

**注意**：逻辑删除本身也有自己的问题，比如：

* 会导致数据库表垃圾数据越来越多，从而影响查询效率

* SQL中全都需要对逻辑删除字段做判断，影响查询效率

因此，我不太推荐采用逻辑删除功能，如果数据不能删除，可以采用把**数据迁移到其它表**的办法。



### 通用枚举

User类中有一个用户状态字段：

```java
private Integer status;//使用状态（1正常 2冻结）
```

像这种字段我们一般会定义一个**枚举**，做业务判断的时候就可以直接基于枚举做比较。但是我们数据库采用的是`int`类型，对应的PO也是`Integer`。因此业务操作时必须手动把`枚举`与`Integer`转换，非常麻烦。

因此，MybatisPlus提供了一个处理枚举的类型转换器，可以帮我们**把枚举类型与数据库类型自动转换**。





定义枚举类：

> 要让`MybatisPlus`处理枚举与数据库类型自动转换，我们必须告诉`MybatisPlus`，枚举中的哪个字段的值作为数据库值。 `MybatisPlus`提供了`@EnumValue`注解来标记枚举属性

```java
import com.baomidou.mybatisplus.annotation.EnumValue;
import lombok.Getter;

@Getter
public enum UserStatus {
    NORMAL(1, "正常"),
    FREEZE(2, "冻结")
    ;
    @EnumValue
    private final int value;
    private final String desc;

    UserStatus(int value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}
```



然后把`User`类中的`status`字段改为`UserStatus` 类型



配置枚举处理器,在application.yaml文件中添加配置：

```yml
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```



同时，为了使页面查询结果也是枚举格式，我们需要修改UserVO中的status属性

并且，在UserStatus枚举中通过`@JsonValue`注解标记JSON序列化时展示的字段



### JSON类型处理器

数据库的user表中有一个`info`字段，是JSON类型

而目前`User`实体类中却是`String`类型



这样一来，我们要读取info中的属性时就非常不方便。如果要方便获取，info的类型最好是一个`Map`或者实体类。

而一旦我们把`info`改为`对象`类型，就需要在写入数据库时手动转为`String`，再读取数据库时，手动转换为`对象`，这会非常麻烦。



因此MybatisPlus提供了很多特殊类型字段的类型处理器，解决特殊字段类型与数据库类型转换的问题。例如处理JSON就可以使用`JacksonTypeHandler`处理器。



使用步骤：

1. 定义实体

首先，我们定义一个单独实体类来与info字段的属性匹配

```java
public class UserInfo{
    //...
}
```

2. 使用类型处理器

接下来，将User类的info字段修改为UserInfo类型，并声明类型处理器

```java
@TableField(typeHandler = JacksonTypeHandler.class)
private UserInfo info;
```





## 插件功能

MybatisPlus提供了很多的插件功能，进一步拓展其功能。目前已有的插件有：

* `PaginationInnerInterceptor`：自动分页

* `TenantLineInnerInterceptor`：多租户

* `DynamicTableNameInnerInterceptor`：动态表名

* `OptimisticLockerInnerInterceptor`：乐观锁

* `IllegalSQLInnerInterceptor`：sql 性能规范

* `BlockAttackInnerInterceptor`：防止全表更新与删除
  
  

### 分页插件

在项目中新建一个配置类

```java
@Configuration
public class MybatisConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        // 初始化核心插件
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 添加分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```



编写一个分页查询的测试

```java
@Test
void testPageQuery() {
    // 1.分页查询，new Page()的两个参数分别是：页码、每页大小
    Page<User> p = userService.page(new Page<>(2, 2));
    // 2.总条数
    System.out.println("total = " + p.getTotal());
    // 3.总页数
    System.out.println("pages = " + p.getPages());
    // 4.数据
    List<User> records = p.getRecords();
    records.forEach(System.out::println);
}
```

这里用到了分页参数，Page，即可以支持分页参数，也可以支持排序参数。常见的API如下：

```java
int pageNo = 1, pageSize = 5;
// 分页参数
Page<User> page = Page.of(pageNo, pageSize);
// 排序参数, 通过OrderItem来指定
page.addOrder(new OrderItem("balance", false));

userService.page(page);
```



### 通用分页实体

现在要实现一个用户分页查询的接口，接口规范如下：

| **参数** | **说明**                                                                                                                                                                                                                                                                                                                                          |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 请求方式   | GET                                                                                                                                                                                                                                                                                                                                             |
| 请求路径   | /users/page                                                                                                                                                                                                                                                                                                                                     |
| 请求参数   | `{    "pageNo": 1,    "pageSize": 5,    "sortBy": "balance",    "isAsc": false,    "name": "o",    "status": 1}`                                                                                                                                                                                                                                |
| 返回值    | `{    "total": 100006,    "pages": 50003,    "list": [        {            "id": 1685100878975279298,            "username": "user_9****",            "info": {                "age": 24,                "intro": "英文老师",                "gender": "female"            },            "status": "正常",            "balance": 2000        }    ]}` |
| 特殊说明   | 如果排序字段为空，默认按照更新时间排序<br>排序字段不为空，则按照排序字段排序                                                                                                                                                                                                                                                                                                        |

这里需要定义3个实体：

* `UserQuery`：分页查询条件的实体，包含分页、排序参数、过滤条件

* `PageDTO`：分页结果实体，包含总条数、总页数、当前页数据

* `UserVO`：用户页面视图实体
  
  

`PageQuery`是前端提交的查询参数，一般包含四个属性：

* `pageNo`：页码

* `pageSize`：每页数据条数

* `sortBy`：排序字段

* `isAsc`：是否升序

```java
@Data
@ApiModel(description = "分页查询实体")
public class PageQuery {
    @ApiModelProperty("页码")
    private Long pageNo;
    @ApiModelProperty("页码")
    private Long pageSize;
    @ApiModelProperty("排序字段")
    private String sortBy;
    @ApiModelProperty("是否升序")
    private Boolean isAsc;
}
```

> 其他类可继承该类实现扩展



`PageDTO`

```java
@Data
public class PageDTO<T> {
    private Long total;
    private Long pages;
    private List<T> list;
}
```



测试代码：

```java
@Override
public PageDTO<UserVO> queryUsersPage(PageQuery query) {
    // 1.构建条件
    // 1.1.分页条件
    Page<User> page = Page.of(query.getPageNo(), query.getPageSize());
    // 1.2.排序条件
    if (query.getSortBy() != null) {
        page.addOrder(new OrderItem(query.getSortBy(), query.getIsAsc()));
    }else{
        // 默认按照更新时间排序
        page.addOrder(new OrderItem("update_time", false));
    }
    // 2.查询
    this.page(page);
    // 3.数据非空校验
    List<User> records = page.getRecords();
    if (records == null || records.size() <= 0) {
        // 无数据，返回空结果
        return new PageDTO<>(page.getTotal(), page.getPages(), Collections.emptyList());
    }
    // 4.有数据，转换
    List<UserVO> list = BeanUtil.copyToList(records, UserVO.class);
    // 5.封装返回
    return new PageDTO<UserVO>(page.getTotal(), page.getPages(), list);
}
```





### 通用分页实体封装



在刚才的代码中，从`PageQuery`到`MybatisPlus`的`Page`之间转换的过程还是比较麻烦的。

我们完全可以在`PageQuery`这个实体中定义一个工具方法，简化开发。

```java
import com.baomidou.mybatisplus.core.metadata.OrderItem;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import lombok.Data;

@Data
public class PageQuery {
    private Integer pageNo;
    private Integer pageSize;
    private String sortBy;
    private Boolean isAsc;

    public <T>  Page<T> toMpPage(OrderItem ... orders){
        // 1.分页条件
        Page<T> p = Page.of(pageNo, pageSize);
        // 2.排序条件
        // 2.1.先看前端有没有传排序字段
        if (sortBy != null) {
            p.addOrder(new OrderItem(sortBy, isAsc));
            return p;
        }
        // 2.2.再看有没有手动指定排序字段
        if(orders != null){
            p.addOrder(orders);
        }
        return p;
    }

    public <T> Page<T> toMpPage(String defaultSortBy, boolean isAsc){
        return this.toMpPage(new OrderItem(defaultSortBy, isAsc));
    }

    public <T> Page<T> toMpPageDefaultSortByCreateTimeDesc() {
        return toMpPage("create_time", false);
    }

    public <T> Page<T> toMpPageDefaultSortByUpdateTimeDesc() {
        return toMpPage("update_time", false);
    }
}
```



在查询出分页结果后，数据的非空校验，数据的vo转换都是模板代码，编写起来很麻烦。

我们完全可以将其封装到PageDTO的工具方法中，简化整个过程

```java
import cn.hutool.core.bean.BeanUtil;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Collections;
import java.util.List;
import java.util.function.Function;
import java.util.stream.Collectors;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class PageDTO<V> {
    private Long total;
    private Long pages;
    private List<V> list;

    /**
     * 返回空分页结果
     * @param p MybatisPlus的分页结果
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> empty(Page<P> p){
        return new PageDTO<>(p.getTotal(), p.getPages(), Collections.emptyList());
    }

    /**
     * 将MybatisPlus分页结果转为 VO分页结果
     * @param p MybatisPlus的分页结果
     * @param voClass 目标VO类型的字节码
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> of(Page<P> p, Class<V> voClass) {
        // 1.非空校验
        List<P> records = p.getRecords();
        if (records == null || records.size() <= 0) {
            // 无数据，返回空结果
            return empty(p);
        }
        // 2.数据转换
        List<V> vos = BeanUtil.copyToList(records, voClass);
        // 3.封装返回
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);
    }

    /**
     * 将MybatisPlus分页结果转为 VO分页结果，允许用户自定义PO到VO的转换方式
     * @param p MybatisPlus的分页结果
     * @param convertor PO到VO的转换函数
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> of(Page<P> p, Function<P, V> convertor) {
        // 1.非空校验
        List<P> records = p.getRecords();
        if (records == null || records.size() <= 0) {
            // 无数据，返回空结果
            return empty(p);
        }
        // 2.数据转换
        List<V> vos = records.stream().map(convertor).collect(Collectors.toList());
        // 3.封装返回
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);
    }
}
```





---

# Docker（详见笔记《Docker》）

[Docker](../Docker.md)



---



# 微服务

## 单体架构

单体架构（monolithic structure）：顾名思义，整个项目中所有功能模块都在一个工程中开发；项目部署时需要对所有模块一起编译、打包；项目的架构设计、开发模式都非常简单。

当项目规模较小时，这种模式上手快，部署、运维也都很方便，因此早期很多小型项目都采用这种模式。



但随着项目的业务规模越来越大，团队开发人员也不断增加，单体架构就呈现出越来越多的问题：

* **团队协作成本高**：试想一下，你们团队数十个人同时协作开发同一个项目，由于所有模块都在一个项目中，不同模块的代码之间物理边界越来越模糊。最终要把功能合并到一个分支，你绝对会陷入到解决冲突的泥潭之中。

* **系统发布效率低**：任何模块变更都需要发布整个系统，而系统发布过程中需要多个模块之间制约较多，需要对比各种文件，任何一处出现问题都会导致发布失败，往往一次发布需要数十分钟甚至数小时。

* **系统可用性差**：单体架构各个功能模块是作为一个服务部署，相互之间会互相影响，一些热点功能会耗尽系统资源，导致其它服务低可用。
  
  

## 微服务架构

微服务架构，首先是服务化，就是将单体架构中的功能模块从单体应用中拆分出来，独立部署为多个服务。同时要满足下面的一些特点：

* **单一职责**：一个微服务负责一部分业务功能，并且其核心数据不依赖于其它模块。

* **团队自治**：每个微服务都有自己独立的开发、测试、发布、运维人员，团队人员规模不超过10人（2张披萨能喂饱）

* **服务自治**：每个微服务都独立打包部署，访问自己独立的数据库。并且要做好服务隔离，避免对其它服务产生影响

![967d1852-e970-4d02-abee-fcc47a2bd5d6](./images/967d1852-e970-4d02-abee-fcc47a2bd5d6.png)



那么，单体架构存在的问题有没有解决呢？

* 团队协作成本高？
  
  * 由于服务拆分，每个服务代码量大大减少，参与开发的后台人员在1~3名，协作成本大大降低

* 系统发布效率低？
  
  * 每个服务都是独立部署，当有某个服务有代码变更时，只需要打包部署该服务即可

* 系统可用性差？
  
  * 每个服务独立部署，并且做好服务隔离，使用自己的服务器资源，不会影响到其它服务。
    
    

综上所述，微服务架构解决了单体架构存在的问题，特别适合大型互联网项目的开发，因此被各大互联网公司普遍采用。大家以前可能听说过分布式架构，分布式就是服务拆分的过程，其实微服务架构正式分布式架构的一种最佳实践的方案。



当然，微服务架构虽然能解决单体架构的各种问题，但在拆分的过程中，还会面临很多其它问题。比如：

* 如果出现跨服务的业务该如何处理？

* 页面请求到底该访问哪个服务？

* 如何实现各个服务之间的服务隔离？
  
  

## Spring Cloud

微服务拆分以后碰到的各种问题都有对应的解决方案和微服务组件，而SpringCloud框架可以说是目前Java领域最全面的微服务组件的集合了。

![6343fe7b-17ae-4129-8ca5-fe0792b1e587](./images/6343fe7b-17ae-4129-8ca5-fe0792b1e587.png)

而且SpringCloud依托于SpringBoot的自动装配能力，大大降低了其项目搭建、组件使用的成本。对于没有自研微服务组件能力的中小型企业，使用SpringCloud全家桶来实现微服务开发可以说是最合适的选择了！

目前SpringCloud最新版本为`2022.0.x`版本，对应的SpringBoot版本为`3.x`版本，但它们全部依赖于JDK17，目前在企业中使用相对较少。

| **SpringCloud版本**                                                                                                   | **SpringBoot版本**                      |
| ------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| [2022.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2022.0-Release-Notes) aka Kilburn | 3.0.x                                 |
| [2021.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2021.0-Release-Notes) aka Jubilee | 2.6.x, 2.7.x (Starting with 2021.0.3) |
| [2020.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes) aka Ilford  | 2.4.x, 2.5.x (Starting with 2020.0.3) |
| [Hoxton](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-Hoxton-Release-Notes)               | 2.2.x, 2.3.x (Starting with SR5)      |
| [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes)              | 2.1.x                                 |
| [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes)                | 2.0.x                                 |
| [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes)                  | 1.5.x                                 |
| [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes)                  | 1.5.x                                 |

因此，我们推荐使用次新版本：Spring Cloud 2021.0.x以及Spring Boot 2.7.x版本。



另外，Alibaba的微服务产品SpringCloudAlibaba目前也成为了SpringCloud组件中的一员，我们课堂中也会使用其中的部分组件。



## 服务拆分

### 拆分原则

什么时候拆？

    对于**大多数小型项目来说，一般是先采用单体架构**，随着用户规模扩大、业务复杂后**再逐渐拆分为微服务架构**。这样初期成本会比较低，可以快速试错。但是，这么做的问题就在于后期做服务拆分时，可能会遇到很多代码耦合带来的问题，拆分比较困难（**前易后难**）。

    而对于一些大型项目，在立项之初目的就很明确，为了长远考虑，在架构设计时就直接选择微服务架构。虽 然前期投入较多，但后期就少了拆分服务的烦恼（**前难后易**）。



怎么拆？

* **高内聚**：每个微服务的职责要尽量单一，包含的业务相互关联度高、完整度高。

* **低耦合**：每个微服务的功能要相对独立，尽量减少对其它微服务的依赖，或者依赖接口的稳定性要强。
  
  

明确了拆分目标，接下来就是拆分方式了。我们在做服务拆分时一般有两种方式：

* **纵向**拆分

* **横向**拆分
1. 所谓**纵向拆分**，就是按照项目的功能模块来拆分。例如黑马商城中，就有用户管理功能、订单管理功能、购物车功能、商品管理功能、支付功能等。那么按照功能模块将他们拆分为一个个服务，就属于纵向拆分。这种拆分模式可以尽可能提高服务的内聚性。

2. 而**横向拆分**，是看各个功能模块之间有没有公共的业务部分，如果有将其抽取出来作为通用服务。例如用户登录是需要发送消息通知，记录风控数据，下单时也要发送短信，记录风控数据。因此消息发送、风控数据记录就是通用的业务功能，因此可以将他们分别抽取为公共服务：消息中心服务、风控管理服务。这样可以提高业务的复用性，避免重复开发。同时通用业务一般接口稳定性较强，也不会使服务之间过分耦合。
   
   

### 服务调用

服务拆分前，多个服务间可以相互进行本地调用，但是拆分后就行不通了

因此要想解决这个问题，我们就必须改造其中的代码，把原本本地方法调用，改造成跨微服务的**远程调用**（RPC，即**R**emote **P**roduce **C**all）



解决方案：通过网络调用接口



Spring给我们提供了一个`RestTemplate`的API，可以方便的实现Http请求的发送。



首先将`Restemplate`交给spring管理

```java
@Configuration
public class RemoteCallConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

使用`Restemplate`

```java
    // 发起请求
    ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
            "http://localhost:8081/items?ids={ids}",
            HttpMethod.GET,
            null,
            new ParameterizedTypeReference<List<ItemDTO>>() {
            },
            Map.of("ids", CollUtil.join(itemIds, ","))
    );
    // 解析响应
    if(!response.getStatusCode().is2xxSuccessful()){
        // 查询失败，直接结束
        return;
    }
```



## 服务治理

### 注册中心原理

在上一节实现了微服务拆分，并且通过Http请求实现了跨微服务的远程调用。不过这种手动发送Http请求的方式存在一些问题。

试想一下，假如商品微服务被调用较多，为了应对更高的并发，我们进行了**多实例部署**



服务的主机和端口都不固定，如何解决？



在微服务远程调用的过程中，包括两个角色：

* 服务提供者：提供接口供其它微服务访问

* 服务消费者：调用其它微服务提供的接口
  
  

在大型微服务项目中，服务提供者的数量会非常多，为了管理这些服务就引入了**注册中心**的概念。注册中心、服务提供者、服务消费者三者间关系如下：

![2f3e8d9e-5040-4680-a84d-9bfcbee1d6af](./images/2f3e8d9e-5040-4680-a84d-9bfcbee1d6af.png)

流程如下：

* 服务启动时就会注册自己的服务信息（服务名、IP、端口）到注册中心

* 调用者可以从注册中心订阅想要的服务，获取服务对应的实例列表（1个服务可能多实例部署）

* 调用者自己对实例列表负载均衡，挑选一个实例

* 调用者向该实例发起远程调用
  
  

当服务提供者的实例宕机或者启动新实例时，调用者如何得知呢？

* 服务提供者会定期向注册中心发送请求，报告自己的健康状态（心跳请求）

* 当注册中心长时间收不到提供者的心跳时，会认为该实例宕机，将其从服务的实例列表中剔除

* 当服务有新实例启动时，会发送注册服务请求，其信息会被记录在注册中心的服务实例列表

* 当注册中心服务列表变更时，会主动通知微服务，更新本地服务列表
  
  

### Nacos注册中心

[Nacos官网| Nacos 配置中心 | Nacos 下载| Nacos 官方社区 | Nacos 官网](https://nacos.io/)

目前开源的注册中心框架有很多，国内比较常见的有：

* Eureka：Netflix公司出品，目前被集成在SpringCloud当中，一般用于Java应用

* Nacos：Alibaba公司出品，目前被集成在SpringCloudAlibaba中，一般用于Java应用

* Consul：HashiCorp公司出品，目前集成在SpringCloud中，不限制微服务语言
  
  

下载nacos镜像

```bash
docker pull nacos/nacos-server
```



初始化数据库信息

> 找到nacos的安装目录，打开conf目录下的nacos-mysql.sql文件



配置数据库信息(编写配置文件，并在docker中挂载)

```properties
PREFER_HOST_MODE=hostname
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_SERVICE_HOST=IP
MYSQL_SERVICE_DB_NAME=nacos
MYSQL_SERVICE_PORT=3306
MYSQL_SERVICE_USER=root
MYSQL_SERVICE_PASSWORD=123
MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
```



启动容器

```bash
docker run -d \
--name nacos \
--env-file ./nacos/custom.env \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--restart=always \
nacos/nacos-server
```



访问页面查看是否启动成功

```url
http://localhost:8848/nacos
```



### 服务注册

服务发现除了要引入nacos依赖以外，由于还需要负载均衡，因此要引入SpringCloud提供的LoadBalancer依赖。

我们在`pom.xml`中添加下面的依赖：

```xml
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

在application.yml`中添加nacos地址配置：

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

启动服务即可自动注册



### 服务发现

接下来，服务调用者就可以去订阅服务了。不过服务可能有多个实例，而真正发起调用时只需要知道一个实例的地址。

因此，服务调用者必须利用负载均衡的算法，从多个实例中挑选一个去访问。常见的负载均衡算法有：

* 随机

* 轮询

* IP的hash

* 最近最少访问

* ...

这里我们可以选择最简单的随机负载均衡。



1. 引入依赖并配置nacos服务地址（同上一小节）



2. 服务发现需要用到一个工具，DiscoveryClient，SpringCloud已经帮我们自动装配，我们可以直接注入使用：

```java
    @Autowired
    private final DiscoveryClient discoveryClient;
```



3. 使用服务

```java
        // 根据服务名称获取服务实例列表
        List<ServiceInstance> instances = discoveryClient.getInstances("item-service");
        if (CollUtils.isEmpty(instances)) {
            return;
        }
        // 根据负载均衡算法选择一个服务实例
        ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));
        // 得到服务实例的URI
        URI uri = instance.getUri();
        ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
                uri + "/items?ids={ids}",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<ItemDTO>>() {
                },
                Map.of("ids", CollUtil.join(itemIds, ","))
        );
```





## OpenFeign

OpenFeign客户端是一个web声明式http远程调用工具，直接可以根据服务名称去注册中心拿到指定的服务IP集合，提供了接口和注解方式进行调用，内嵌集成了Ribbon本地负载均衡器（现在流行使用loadbalancer）

### 快速入门

上一节远程调用代码仍然比较复杂



而且这种调用方式，与原本的本地方法调用差异太大，编程时的体验也不统一，一会儿远程调用，一会儿本地调用。

因此，我们必须想办法改变远程调用的开发模式，让**远程调用像本地方法调用一样简单**。而这就要用到OpenFeign组件了。

其实远程调用的关键点就在于四个：

* 请求方式

* 请求路径

* 请求参数

* 返回值类型

所以，OpenFeign就利用SpringMVC的相关注解来声明上述4个参数，然后基于动态代理帮我们生成远程调用的代码，而无需我们手动再编写，非常方便。



1. 引入依赖

```xml
  <!--openFeign-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  <!--负载均衡器-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  </dependency>
```

2. 启用OpenFeign

接下来，我们在启动类上添加注解，启动OpenFeign功能

```java
@EnableFeignClients
```

3. 编写OpenFeign客户端

```java
@FeignClient("item-service")
public interface ItemClient {
    
    // 注解指定请求方式、请求路径、参数类型、返回类型
    @GetMapping("/items")
    List<ItemDTO> queryItemsByIds(@RequestParam("ids") List<Long> ids);
}

```

> 这里只需要声明接口，无需实现方法。接口中的几个关键信息：
> 
> * `@FeignClient("item-service")` ：声明服务名称
> 
> * `@GetMapping` ：声明请求方式
> 
> * `@GetMapping("/items")` ：声明请求路径
> 
> * `@RequestParam("ids") Collection<Long> ids` ：声明请求参数
> 
> * `List<ItemDTO>` ：返回值类型



4. 使用FeignClient

注入并调用接口中的方法即可



### 连接池

> OpenFeign默认使用JDK提供的`HttpURLConnection`, 效率较低

Feign底层发起http请求，依赖于其它的框架。其底层支持的http客户端实现包括：

* HttpURLConnection：默认实现，不支持连接池

* Apache HttpClient ：支持连接池

* OKHttp：支持连接池

因此我们通常会使用带有连接池的客户端来代替默认的HttpURLConnection。比如，我们使用OK Http



1. 引入依赖

```xml
<!--OK http 的依赖 -->
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-okhttp</artifactId>
</dependency>
```

2. 开启连接池

> 在`application.yml`配置文件中开启Feign的连接池功能：

```yml
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```

3. 重启生效



### 最佳使用方案

如果在其他微服务中也需要调用该接口，需要重新编写相同的`Client`，导致代码重复

相信大家都能想到，避免重复编码的办法就是**抽取**。不过这里有两种抽取思路：

* 思路1：抽取到微服务之外的公共module

* 思路2：每个微服务自己抽取一个module（推荐）

如图：

![a3767cd1-7a6d-4063-8dbb-81740f1d8aa1](./images/a3767cd1-7a6d-4063-8dbb-81740f1d8aa1.png)



> 方案1抽取更加简单，工程结构也比较清晰，但缺点是整个项目耦合度偏高。
> 
> 方案2抽取相对麻烦，工程结构相对更复杂，但服务之间耦合度降低。



需要调用接口的服务引入相关API模块的依赖，并且在启动类的注解`@EnableFeignClients`中**指定扫描的Client即可**



### 日志配置

OpenFeign只会在FeignClient所在包的日志级别为**DEBUG**时，才会输出日志。而且其日志级别有4级：

* **NONE**：不记录任何日志信息，这是默认值。

* **BASIC**：仅记录请求的方法，URL以及响应状态码和执行时间

* **HEADERS**：在BASIC的基础上，额外记录了请求和响应的头信息

* **FULL**：记录所有请求和响应的明细，包括头信息、请求体、元数据。

Feign默认的日志级别就是NONE，所以默认我们看不到请求日志。



在api模块下新建一个配置类，定义Feign的日志级别：

```java
public class DefaultFeignConfig {
    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

接下来，要让日志级别生效，还需要配置这个类。有两种方式：

* **局部**生效：在某个`FeignClient`中配置，只对当前`FeignClient`生效

```java
@FeignClient(value = "item-service", configuration = DefaultFeignConfig.class)
```

* **全局**生效：在`@EnableFeignClients`中配置，针对所有`FeignClient`生效。

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfig.class)
```



> 一般情况下无需开启日志，调试时才需开启


