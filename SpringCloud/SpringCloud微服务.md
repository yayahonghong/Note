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

### 通用枚举


