# MyBatis-Plus 数据访问增强

MyBatis-Plus 是 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变

## 快速入门

### 依赖引入

Maven 依赖配置：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
<!-- springBoot3依赖 -->
```

!!! tip "版本说明"
    以上为Spring Boot 3.x.x版本依赖，3.x.x以下版本略有不同

### 基本使用

自定义的 Mapper 继承 MyBatis-Plus 提供的 BaseMapper 接口：

```java
public interface UserMapper extends BaseMapper<User> {}
```

## 核心注解

!!! info "约定大于配置"
    MyBatis-Plus 通过扫描实体类，并基于反射获取实体类信息作为数据库表信息。

### 默认约定
- 类名驼峰转下划线作为表名
- 名为id的字段作为主键
- 变量名驼峰转下划线作为表的字段名

### 常用注解

MyBatis-Plus 中比较常用的几个注解（主要用于不满足约定的情况）：

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

### IdType 枚举

- `AUTO`：数据库自增长
- `INPUT`：通过set方法自行输入
- `ASSIGN_ID`：分配 ID，使用雪花算法

### @TableField 常见场景

- 成员变量名与数据库字段名不一致
- 成员变量名以is开头，且是布尔值
- 成员变量名与数据库关键字冲突，如：`@TableField("`order`")`
- 成员变量不是数据库字段

## 常用配置

```yaml
mybatis-plus:
  type-aliases-package: com.ysh.entity # 别名扫描包
  mapper-locations: classpath:mapper/**/*.xml # mapper扫描路径
  configuration:
    map-underscore-to-camel-case: true # 开启驼峰命名规则
    cache-enabled: false # 关闭Mybatis二级缓存
  global-config:
    db-config:
      id-type: assign_id # 主键类型为雪花算法
      update-strategy: not_null # 更新策略:只更新不为null的值
```

具体可参考官方文档：[使用配置](https://www.baomidou.com)

## 条件构造器

### QueryWrapper 基本用法

需求：查询出名字中带o的，存款大于等于1000元的人的信息

```java
// 构建查询条件
QueryWrapper<User> wrapper = new QueryWrapper<User>()
    .select("id", "username", "info", "balance")
    .like("username", "o")
    .ge("balance", 1000);

// 执行查询
userMapper.selectList(wrapper);
```

### UpdateWrapper 用法

需求：更新id为1,2,4的用户的余额，扣200

```java
List<Long> ids = List.of(1L, 2L, 4L);
UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
    .setSql("balance = balance - 200")
    .in("id", ids);
userMapper.update(null, wrapper);
```

### LambdaWrapper 用法（推荐）

```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
    .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
    .like(User::getUsername, "o")
    .ge(User::getBalance, 1000);
userMapper.selectList(wrapper);
```

!!! tip "最佳实践"
    - 尽量使用 LambdaQueryWrapper 和 LambdaUpdateWrapper，避免硬编码
    - QueryWrapper 通常用来构建 select、delete、update 的 where 条件部分
    - UpdateWrapper 通常只有在 set 语句比较特殊才使用

## 自定义SQL

结合 Wrapper 构建复杂查询条件，同时自定义SQL语句的其他部分。

### 使用步骤

1. 构建条件：
```java
List<Long> ids = List.of(1L, 2L, 4L);
int amount = 200;
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
    .in(User::getId, ids);
userMapper.updateBalanceByIds(wrapper, amount);
```

2. Mapper 方法声明：
```java
void updateBalanceByIds(@Param("ew") LambdaQueryWrapper<User> wrapper, 
                       @Param("amount") int amount);
```

!!! warning "注意"
    Wrapper 参数必须指定为 "ew"

3. 自定义SQL：
```xml
<update id="updateBalanceByIds">
    UPDATE tb_user SET balance = balance - #{amount} ${ew.customSqlSegment}
</update>
```

## Service接口

### 基本使用

1. 自定义Service接口继承IService接口：
```java
public interface IUserService extends IService<User> {
}
```

2. 自定义Service实现类：
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> 
    implements IUserService {
}
```

### Lambda查询

实现复杂条件查询：

```java
List<User> users = userService.lambdaQuery()
    .like(username != null, User::getUsername, username)
    .eq(status != null, User::getStatus, status)
    .ge(minBalance != null, User::getBalance, minBalance)
    .le(maxBalance != null, User::getBalance, maxBalance)
    .list();
```

链式编程结尾方法：
- `.one()`：最多1个结果
- `.list()`：返回集合结果
- `.count()`：返回计数结果

### Lambda更新

根据id修改用户余额：

```java
lambdaUpdate()
    .set(User::getBalance, remainBalance) // 更新余额
    .set(remainBalance == 0, User::getStatus, 2) // 动态判断
    .eq(User::getId, id)
    .eq(User::getBalance, user.getBalance()) // 乐观锁
    .update();
```

### 批量新增

高效批量插入：

```java
@Test
void testSaveBatch() {
    List<User> list = new ArrayList<>(1000);
    for (int i = 1; i <= 100000; i++) {
        list.add(buildUser(i));
        // 每1000条批量插入一次
        if (i % 1000 == 0) {
            userService.saveBatch(list);
            list.clear();
        }
    }
}
```

!!! tip "性能优化"
    在 MySQL 连接参数中添加 `rewriteBatchedStatements=true` 可以将多条插入语句合并为一条，大幅提升性能。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mp?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=UTF-8
```

## 扩展功能

### 代码生成器

推荐使用 IDEA 插件 "MyBatisX" 进行图形化代码生成。

### 静态工具类

使用 `Db` 静态工具类避免Service循环依赖：

```java
@Test
void testDbOperations() {
    // 根据ID查询
    User user = Db.getById(1L, User.class);
    
    // 复杂条件查询
    List<User> list = Db.lambdaQuery(User.class)
        .like(User::getUsername, "o")
        .ge(User::getBalance, 1000)
        .list();
    
    // 更新操作
    Db.lambdaUpdate(User.class)
        .set(User::getBalance, 2000)
        .eq(User::getUsername, "Rose");
}
```

### 逻辑删除

1. 添加逻辑删除字段：
```sql
alter table address add deleted bit default b'0' null comment '逻辑删除';
```

2. 配置逻辑删除：
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted # 逻辑删除字段名
      logic-delete-value: 1 # 已删除值
      logic-not-delete-value: 0 # 未删除值
```

!!! warning "注意事项"
    - 逻辑删除会导致垃圾数据增多，影响查询效率
    - 自定义SQL需要手动处理逻辑删除逻辑
    - 建议采用数据迁移方案替代逻辑删除

### 通用枚举

通用枚举可以简化枚举字段的读写操作。

1. 定义枚举类：
```java
@Getter
public enum UserStatus {
    NORMAL(1, "正常"),
    FREEZE(2, "冻结");
    
    @EnumValue
    private final int value;
    private final String desc;

    UserStatus(int value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}
```

2. 配置枚举处理器：
```yaml
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

### JSON类型处理器

处理JSON字段的自动转换：

```java
@TableField(typeHandler = JacksonTypeHandler.class)
private UserInfo info;
```

## 插件功能

### 分页插件

1. 配置分页插件：
```java
@Configuration
public class MybatisConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MySQL));
        return interceptor;
    }
}
```

2. 使用分页查询：
```java
@Test
void testPageQuery() {
    // 分页查询，参数：页码、每页大小
    Page<User> p = userService.page(new Page<>(2, 2));
    System.out.println("total = " + p.getTotal());
    System.out.println("pages = " + p.getPages());
    List<User> records = p.getRecords();
}
```

### 通用分页实体

创建通用的分页查询和返回实体，简化分页开发：

!!!example "分页实体封装"
    
    **PageQuery.java**
    ```java
    @Data
    public class PageQuery {
        private Integer pageNo;
        private Integer pageSize;
        private String sortBy;
        private Boolean isAsc;

        public <T> Page<T> toMpPage(OrderItem... orders) {
            Page<T> p = Page.of(pageNo, pageSize);
            if (sortBy != null) {
                p.addOrder(new OrderItem(sortBy, isAsc));
                return p;
            }
            if (orders != null) {
                p.addOrder(orders);
            }
            return p;
        }

        public <T> Page<T> toMpPageDefaultSortByUpdateTimeDesc() {
            return toMpPage("update_time", false);
        }
    }
    ```

    **PageDTO.java**
    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class PageDTO<V> {
        private Long total;
        private Long pages;
        private List<V> list;

        public static <V, P> PageDTO<V> of(Page<P> p, Class<V> voClass) {
            List<P> records = p.getRecords();
            if (records == null || records.size() <= 0) {
                return new PageDTO<>(p.getTotal(), p.getPages(), Collections.emptyList());
            }
            List<V> vos = BeanUtil.copyToList(records, voClass);
            return new PageDTO<>(p.getTotal(), p.getPages(), vos);
        }
    }
    ```

---

**下一节：** [微服务基础设施](SpringCloud-Discovery.md)
