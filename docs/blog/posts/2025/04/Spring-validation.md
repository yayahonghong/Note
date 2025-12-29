---
title: Spring Validation使用指南
authors:
  - ysh
date: 2025-12-12
categories:
  - Backend
slug: bakend
---

Spring Validation 是 Spring 框架中用于数据验证的强大工具。它允许你通过注解来定义验证规则，从而简化了数据验证的过程。本文将详细介绍如何在 Spring 项目中使用 Spring Validation。
<!-- more -->

## 引入依赖
首先，你需要在你的项目中引入 Spring Validation 的依赖。如果你使用的是 Maven 或 Gradle，可以在 `pom.xml` 或 `build.gradle` 文件中添加以下依赖：
=== "Maven"

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
    </dependencies>
    ```

=== "Gradle (Groovy DSL)"

    ```groovy
    dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    }
    ```

=== "Gradle (Kotlin DSL)"

    ```kotlin
    dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    }
    ```
    
## DTO中使用注解
在 DTO（Data Transfer Object）中，你可以使用 Spring Validation 提供的注解来定义验证规则,其`message`属性用于自定义错误提示信息。以下是一些常用的注解：

```java
@Data
@Schema(description = "用户DTO")
public class UserDTO {
    
    // 非空校验
    @NotNull(message = "用户名不能为空")
    @NotBlank(message = "用户名不能为空字符串") // 不允许空格
    private String username;
    
    // 长度校验
    @Size(min = 6, max = 128, message = "密码长度必须在6-128之间")
    private String password;
    
    // 手机号校验
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
    
    // 邮箱校验
    @Email(message = "邮箱格式不正确")
    private String email;
    
    // 数值范围
    @Min(value = 0, message = "年龄不能小于0")
    @Max(value = 150, message = "年龄不能大于150")
    private Integer age;
    
    // 正数校验
    @Positive(message = "金额必须大于0")
    private BigDecimal amount;
    
    // 日期校验
    @Past(message = "出生日期必须是过去的时间")
    private Date birth;
    
    @Future(message = "预约时间必须是未来的时间")
    private LocalDateTime appointmentTime;
    
    // 集合校验
    @NotEmpty(message = "列表不能为空")
    @Size(min = 1, max = 10, message = "最多选择10项")
    private List<String> items;

    @Valid //嵌套校验，会同时校验Address对象的属性
    private Address address;
}

```

## Controller中使用启用校验

在Controller中，可以在类上使用@Validated 注解启用查询参数和路径参数的校验。

@Valid 用于嵌套对象的校验。

```java
@Validated 
@RestController
@RequestMapping("/user")
public class UserController {
    
    // @RequestBody 参数校验
    @PostMapping("/login")
    public Result<String> login(@Valid @RequestBody LoginDTO dto) {
        // 校验失败会自动抛出 MethodArgumentNotValidException
        return Result.success(userService.login(dto));
    }
    
    // @RequestParam 参数校验（需要在类上加 @Validated）
    @GetMapping("/info")
    public Result<UserVO> getUser(
        @NotNull(message = "用户ID不能为空") 
        @RequestParam Long userId) {
        return Result.success(userService.getById(userId));
    }
    
    // 嵌套对象校验
    @PostMapping("/complex")
    public Result<Void> complexValidation(@Valid @RequestBody ComplexDTO dto) {
        // 需要在嵌套对象字段上加 @Valid
        return Result.success();
    }
}

```

## 异常处理

为了更好地处理验证失败的情况，你可以创建一个统一的异常处理类，使用 `@ExceptionHandler` 注解来捕获并处理 `MethodArgumentNotValidException` 异常。

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotNotException.class)
    public Result<String> handleValidationException(MethodArgumentNotValidException e) {
        // 获取所有验证错误信息
        List<String> errorMessages = e.getBindingResult().getAllErrors().stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .collect(Collectors.toList());
        
        // 将错误信息返回给客户端
        return Result.fail(String.join(", ", errorMessages));
    }
}
```
