# Spring常用参数校验注解

## 目录
- [Spring Validation (spring-boot-starter-validation)](#spring-validation)
- [Hibernate Validator](#hibernate-validator)
- [自定义校验注解](#自定义校验注解)
- [分组校验](#分组校验)
- [嵌套校验](#嵌套校验)

<a id="spring-validation"></a>
## Spring Validation (spring-boot-starter-validation)

Spring Boot 2.3.0之后，校验相关的依赖需要单独引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 常用注解

| 注解 | 说明 |
| --- | --- |
| @NotNull | 被注解的元素不能为 `null` |
| @NotEmpty | 被注解的字符串、集合、Map、数组不能为 `null` 且长度或大小必须大于0 |
| @NotBlank | 被注解的字符串不能为 `null` 且去除前后空格后长度必须大于0 |
| @Size | 被注解的元素的大小必须在指定的范围内，适用于字符串、集合、Map、数组等 |
| @Min | 被注解的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max | 被注解的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Range | 被注解的元素必须在合适的范围内 |
| @Email | 被注解的元素必须是电子邮箱地址，也可以通过regexp和flags指定自定义的email格式 |
| @Pattern | 被注解的元素必须符合指定的正则表达式 |
| @Positive | 被注解的元素必须是正数（大于0） |
| @PositiveOrZero | 被注解的元素必须是正数或零 |
| @Negative | 被注解的元素必须是负数（小于0） |
| @NegativeOrZero | 被注解的元素必须是负数或零 |
| @Past | 被注解的元素必须是一个过去的日期 |
| @PastOrPresent | 被注解的元素必须是一个过去或现在的日期 |
| @Future | 被注解的元素必须是一个将来的日期 |
| @FutureOrPresent | 被注解的元素必须是一个将来或现在的日期 |
| @Valid | 用于对象属性的递归验证，常用于嵌套校验 |

### 使用示例

```java
@RestController
@Validated
public class UserController {
    
    @PostMapping("/user")
    public String createUser(@RequestBody @NotNull User user) {
        // 处理逻辑
        return "success";
    }
}

public class User {
    // 字符串相关校验
    @NotBlank(message = "用户名不能为空白")
    private String username;
    
    @Size(min = 6, max = 20, message = "密码长度必须在6到20之间")
    private String password;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phoneNumber;
    
    // 集合相关校验
    @NotEmpty(message = "角色列表不能为空")
    @Size(max = 10, message = "角色数量不能超过10个")
    private List<String> roles;
    
    // 数字相关校验
    @PositiveOrZero(message = "年龄不能为负数")
    private Integer age;
    
    // 日期相关校验
    @Past(message = "出生日期必须是过去的日期")
    private Date birthDate;
    
    @FutureOrPresent(message = "预约时间必须是现在或将来的时间")
    private LocalDateTime appointmentTime;
    
    // getter 和 setter
}

public class Product {
    @NotNull(message = "商品ID不能为空")
    private Long id;
    
    @Min(value = 0, message = "价格不能为负数")
    @Max(value = 100000, message = "价格不能超过100000")
    private BigDecimal price;
    
    @PositiveOrZero(message = "库存不能为负数")
    private Integer stock;
    
    // getter 和 setter
}

public class Account {
    @Negative(message = "欠款金额必须为负数")
    private BigDecimal debt;
    
    @NegativeOrZero(message = "欠款金额必须为负数或零")
    private BigDecimal maxDebt;
    
    // getter 和 setter
}

public class Order {
    @NotNull(message = "订单号不能为空")
    private String orderNo;
    
    @PastOrPresent(message = "下单时间必须是过去或现在的时间")
    private LocalDateTime orderTime;
    
    @NotNull(message = "用户信息不能为空")
    @Valid // 开启嵌套校验
    private User user;
    
    // getter 和 setter
}

<a id="hibernate-validator"></a>
## Hibernate Validator

Hibernate Validator 是 Bean Validation 的参考实现，提供了更多的校验注解。

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>7.0.1.Final</version>
</dependency>
```

### 常用注解

| 注解 | 说明 |
| --- | --- |
| @URL | 被注解的元素必须是一个有效的URL |
| @CreditCardNumber | 被注解的字符串必须是有效的信用卡号码（使用Luhn算法校验） |
| @Length | 被注解的字符串的长度必须在指定的范围内 |
| @EAN | 被注解的字符串必须是有效的EAN商品编码 |
| @ISBN | 被注解的字符串必须是有效的ISBN书籍编号 |
| @SafeHtml | 被注解的字符串必须是安全的HTML（防止XSS攻击） |

### 使用示例

```java
public class WebsiteInfo {
    @URL(message = "网址格式不正确")
    private String url;
    
    @SafeHtml(message = "HTML内容不安全")
    private String content;
    
    // getter 和 setter
}

public class Payment {
    @CreditCardNumber(message = "信用卡号码不正确")
    private String creditCardNumber;
    
    // getter 和 setter
}

public class UserProfile {
    @Length(min = 6, max = 20, message = "密码长度必须在6到20之间")
    private String password;
    
    // getter 和 setter
}

public class Product {
    @EAN(message = "EAN编码不正确")
    private String eanCode;
    
    // getter 和 setter
}

public class Book {
    @ISBN(message = "ISBN编号不正确")
    private String isbn;
    
    // getter 和 setter
}
```

## 自定义校验注解

当内置的校验注解无法满足需求时，可以自定义校验注解。

### 示例：自定义手机号校验注解

```java
// 1. 定义注解
@Documented
@Constraint(validatedBy = PhoneNumberValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface PhoneNumber {
    
    String message() default "手机号格式不正确";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
}

// 2. 实现校验器
public class PhoneNumberValidator implements ConstraintValidator<PhoneNumber, String> {
    
    private static final String PHONE_REGEX = "^1[3-9]\\d{9}$";
    private static final Pattern PATTERN = Pattern.compile(PHONE_REGEX);
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true; // 如果需要非空校验，应该结合@NotNull使用
        }
        return PATTERN.matcher(value).matches();
    }
}

// 3. 使用自定义注解
public class User {
    
    @PhoneNumber(message = "手机号格式不正确")
    private String phoneNumber;
    
    // getter 和 setter
}
```
## 分组校验

分组校验可以在不同的场景下使用不同的校验规则。

### 示例

```java
// 1. 定义分组接口
// 新增用户分组
public interface AddGroup {}

// 更新用户分组
public interface UpdateGroup {}

// 2. 在校验注解中指定分组
public class User {
    
    @NotNull(groups = {UpdateGroup.class}, message = "更新时ID不能为空")
    private Long id;
    
    @NotBlank(groups = {AddGroup.class, UpdateGroup.class}, message = "用户名不能为空")
    private String username;
    
    @NotBlank(groups = {AddGroup.class}, message = "新增时密码不能为空")
    private String password;
    
    // getter 和 setter
}

// 3. 在Controller中使用分组校验
@RestController
@RequestMapping("/user")
public class UserController {
    
    @PostMapping
    public String addUser(@RequestBody @Validated(AddGroup.class) User user) {
        // 新增用户逻辑
        return "success";
    }
    
    @PutMapping
    public String updateUser(@RequestBody @Validated(UpdateGroup.class) User user) {
        // 更新用户逻辑
        return "success";
    }
}
```

## 嵌套校验

当一个对象包含其他对象作为属性时，可以使用嵌套校验。

### 示例

```java
public class Order {
    
    @NotNull(message = "订单号不能为空")
    private String orderNo;
    
    @NotNull(message = "用户信息不能为空")
    @Valid // 开启嵌套校验
    private User user;
    
    @NotEmpty(message = "订单项不能为空")
    @Valid // 开启嵌套校验
    private List<OrderItem> items;
    
    // getter 和 setter
}

public class User {
    
    @NotBlank(message = "用户名不能为空")
    private String username;
    
    // getter 和 setter
}

public class OrderItem {
    
    @NotNull(message = "商品ID不能为空")
    private Long productId;
    
    @Positive(message = "数量必须为正数")
    private Integer quantity;
    
    // getter 和 setter
}

// 使用示例
@RestController
@RequestMapping("/order")
public class OrderController {
    
    @PostMapping
    public String createOrder(@RequestBody @Validated Order order) {
        // 创建订单逻辑
        return "success";
    }
}
```

## 实际应用

### 全局异常处理

为了优雅地处理校验异常，通常会配合全局异常处理器使用：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Object> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return ResponseEntity.badRequest().body(errors);
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<Object> handleConstraintViolation(ConstraintViolationException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getConstraintViolations().forEach(violation -> {
            String fieldName = violation.getPropertyPath().toString();
            String errorMessage = violation.getMessage();
            errors.put(fieldName, errorMessage);
        });
        return ResponseEntity.badRequest().body(errors);
    }
}
```

### 快速失败模式

默认情况下，校验会检查所有约束并报告所有违反的约束。如果需要在发现第一个违反约束时就停止校验，可以配置快速失败模式：

```java
@Configuration
public class ValidationConfig {
    
    @Bean
    public Validator validator() {
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                .failFast(true)
                .buildValidatorFactory();
        return validatorFactory.getValidator();
    }
}
```