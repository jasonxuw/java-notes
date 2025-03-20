# Lombok常用注解

Lombok是一个Java库，通过注解的方式帮助开发者减少样板代码的编写，提高开发效率。本文将Lombok常用注解分类整理，并提供详细说明和使用示例。

## 目录
- [构造器相关注解](#构造器相关注解)
- [字段相关注解](#字段相关注解)
- [方法相关注解](#方法相关注解)
- [代码简化注解](#代码简化注解)
- [异常处理注解](#异常处理注解)
- [日志相关注解](#日志相关注解)
- [实用工具注解](#实用工具注解)
- [高级用法注解](#高级用法注解)
- [配置与扩展](#配置与扩展)

## 构造器相关注解

### @NoArgsConstructor
**作用**：生成一个无参构造器。

**示例**：
```java
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
}
```

### @AllArgsConstructor
**作用**：生成一个包含所有字段的构造器。

**示例**：
```java
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}
```

### @RequiredArgsConstructor
**作用**：为所有带有`final`修饰或被`@NonNull`注解的字段生成构造器。

**示例**：
```java
@RequiredArgsConstructor
public class User {
    private final Long id;
    @NonNull
    private String name;
    private Integer age; // 不会包含在生成的构造器中
}
```

## 字段相关注解

### @Getter
**作用**：为类的所有字段生成getter方法，也可以单独应用于某个字段。

**示例**：
```java
@Getter
public class User {
    private Long id;
    private String name;
}

// 或者单独应用于字段
public class User {
    @Getter
    private Long id;
    private String name;
}
```

### @Setter
**作用**：为类的所有非final字段生成setter方法，也可以单独应用于某个字段。

**示例**：
```java
@Setter
public class User {
    private Long id;
    private String name;
}

// 或者单独应用于字段
public class User {
    @Setter
    private Long id;
    private String name;
}
```

### @NonNull
**作用**：在生成的方法中添加非空检查，如果参数为null，则抛出NullPointerException。

**示例**：
```java
public class User {
    @NonNull
    private String name;
    
    public void setName(@NonNull String name) {
        this.name = name;
    }
}
```

### @ToString.Exclude
**作用**：排除特定字段不包含在生成的toString方法中。

**示例**：
```java
@ToString
public class User {
    private Long id;
    private String name;
    
    @ToString.Exclude
    private String password; // 不会包含在toString方法中
}
```

## 方法相关注解

### @ToString
**作用**：生成toString方法，可以通过参数控制包含哪些字段。

**示例**：
```java
@ToString(exclude = "password")
public class User {
    private Long id;
    private String name;
    private String password;
}

// 或者只包含指定字段
@ToString(of = {"id", "name"})
public class User {
    private Long id;
    private String name;
    private String password;
}
```

### @EqualsAndHashCode
**作用**：生成equals和hashCode方法。

**示例**：
```java
@EqualsAndHashCode
public class User {
    private Long id;
    private String name;
}

// 可以指定使用哪些字段
@EqualsAndHashCode(of = {"id"})
public class User {
    private Long id;
    private String name;
}
```

## 代码简化注解

### @Data
**作用**：组合注解，相当于同时使用了@Getter、@Setter、@ToString、@EqualsAndHashCode和@RequiredArgsConstructor。

**示例**：
```java
@Data
public class User {
    private Long id;
    private String name;
}
```

### @Value
**作用**：创建不可变类，所有字段都被标记为private final，并生成getter方法（没有setter）。

**示例**：
```java
@Value
public class ImmutableUser {
    Long id;
    String name;
}
```

### @Builder
**作用**：实现建造者模式，为类提供一个流式的构建器API。

**示例**：
```java
@Builder
public class User {
    private Long id;
    private String name;
    private Integer age;
}

// 使用方式
User user = User.builder()
    .id(1L)
    .name("张三")
    .age(25)
    .build();
```

### @Accessors
**作用**：自定义访问器（getter和setter）的行为。

**示例**：
```java
@Data
@Accessors(chain = true) // 启用链式调用
public class User {
    private Long id;
    private String name;
}

// 使用方式
User user = new User()
    .setId(1L)
    .setName("张三");
```

## 异常处理注解

### @SneakyThrows
**作用**：允许在不声明throws的情况下抛出受检异常。

**示例**：
```java
public class FileReader {
    @SneakyThrows
    public String readFile(String path) {
        return Files.readString(Path.of(path)); // 无需显式处理IOException
    }
}
```

## 日志相关注解

### @Slf4j
**作用**：自动为类添加SLF4J日志对象。

**示例**：
```java
@Slf4j
public class UserService {
    public void saveUser(User user) {
        log.info("保存用户: {}", user);
        // 业务逻辑
    }
}
```

### @Log
**作用**：自动为类添加java.util.logging.Logger对象。

**示例**：
```java
@Log
public class UserService {
    public void saveUser(User user) {
        log.info("保存用户: " + user);
        // 业务逻辑
    }
}
```

### @Log4j / @Log4j2
**作用**：自动为类添加Log4j/Log4j2日志对象。

**示例**：
```java
@Log4j2
public class UserService {
    public void saveUser(User user) {
        log.info("保存用户: {}", user);
        // 业务逻辑
    }
}
```

## 实用工具注解

### @Cleanup
**作用**：自动管理资源，确保在作用域结束时调用close()方法。

**示例**：
```java
public void readFile(String path) {
    @Cleanup InputStream in = new FileInputStream(path);
    // 使用in读取文件
    // 方法结束时会自动调用in.close()
}
```

### @Synchronized
**作用**：方法同步的更安全的变体，使用私有锁对象而不是this。

**示例**：
```java
public class Counter {
    private int count = 0;
    
    @Synchronized
    public void increment() {
        count++;
    }
}
```

### @With
**作用**：为不可变对象生成"withX"方法，返回字段值被修改的对象副本。

**示例**：
```java
@Value
@With
public class ImmutableUser {
    Long id;
    String name;
}

// 使用方式
ImmutableUser user = new ImmutableUser(1L, "张三");
ImmutableUser newUser = user.withName("李四"); // 创建一个新对象，只有name被修改
```

### @Singular
**作用**：与@Builder一起使用，为集合类型的字段提供单数形式的添加方法。

**示例**：
```java
@Builder
public class User {
    private Long id;
    private String name;
    
    @Singular
    private List<String> roles;
}

// 使用方式
User user = User.builder()
    .id(1L)
    .name("张三")
    .role("ADMIN")  // 单数形式添加
    .role("USER")   // 可以多次调用
    .build();
```

## 高级用法注解

### @Delegate
**作用**：实现委托模式，将方法调用委托给指定字段。

**示例**：
```java
public class UserManager {
    @Delegate
    private final UserRepository repository = new UserRepositoryImpl();
    
    // 无需实现UserRepository的方法，自动委托给repository字段
}
```

### @FieldDefaults
**作用**：批量设置字段的访问级别和修饰符。

**示例**：
```java
@FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
public class User {
    Long id;        // 等同于 private final Long id;
    String name;    // 等同于 private final String name;
    
    @FieldDefaults(makeFinal = false)
    Integer age;    // 等同于 private Integer age;
}
```

### @Wither
**作用**：@With的旧版本，功能相同，为不可变对象生成"withX"方法。

**示例**：
```java
@Value
@Wither
public class ImmutableUser {
    Long id;
    String name;
}
```

### @UtilityClass
**作用**：创建工具类，将构造器设为私有，并将类标记为final。

**示例**：
```java
@UtilityClass
public class StringUtils {
    public String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return Character.toUpperCase(str.charAt(0)) + str.substring(1);
    }
    
    // 无需手动添加private构造器和final修饰符
}
```

### @Helper
**作用**：将内部类中的静态方法"提升"到外部类，使其可以直接通过外部类调用。

**示例**：
```java
public class StringUtils {
    @Helper
    static class Helpers {
        static String capitalize(String str) {
            if (str == null || str.isEmpty()) {
                return str;
            }
            return Character.toUpperCase(str.charAt(0)) + str.substring(1);
        }
    }
    
    // 可以直接通过StringUtils.capitalize()调用
}
```

### @ExtensionMethod
**作用**：为已有类添加扩展方法，类似于C#的扩展方法。

**示例**：
```java
@ExtensionMethod({StringExtensions.class})
public class StringProcessor {
    public void process(String input) {
        // 直接在String对象上调用扩展方法
        String result = input.capitalize();
    }
}

class StringExtensions {
    public static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return Character.toUpperCase(str.charAt(0)) + str.substring(1);
    }
}
```

### @SuperBuilder
**作用**：增强版的@Builder，支持继承关系中的构建器模式。

**示例**：
```java
@SuperBuilder
public class Person {
    private String name;
    private int age;
}

@SuperBuilder
public class Employee extends Person {
    private String company;
    private double salary;
}

// 使用方式
Employee employee = Employee.builder()
    .name("张三")       // Person类的属性
    .age(30)           // Person类的属性
    .company("ABC公司") // Employee类的属性
    .salary(10000.0)   // Employee类的属性
    .build();
```

### @Jacksonized
**作用**：与@Builder或@SuperBuilder一起使用，使生成的构建器与Jackson序列化/反序列化兼容。

**示例**：
```java
@Jacksonized
@Builder
public class User {
    private Long id;
    private String name;
    private int age;
}

// 可以直接通过Jackson反序列化为User对象
```

### @FieldNameConstants
**作用**：为类的所有字段生成常量名称，便于反射操作和动态查询。

**示例**：
```java
@FieldNameConstants
public class User {
    private String name;
    
    @FieldDefaults(makeFinal = true)
    private Long id;
}

// 生成的常量可以这样使用
String idFieldName = User.Fields.id;    // "id"
String nameFieldName = User.Fields.name; // "name"
```

### @UtilityClass
**作用**：创建工具类，将构造器设为私有，并将类标记为final。

**示例**：
```java
@UtilityClass
public class StringUtils {
    public String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return Character.toUpperCase(str.charAt(0)) + str.substring(1);
    }
    
    // 无需手动添加private构造器和final修饰符
}
```

### @Helper
**作用**：将内部类中的静态方法"提升"到外部类，使其可以直接通过外部类调用。

**示例**：
```java
public class StringUtils {
    @Helper
    static class Helpers {
        static String capitalize(String str) {
            if (str == null || str.isEmpty()) {
                return str;
            }
            return Character.toUpperCase(str.charAt(0)) + str.substring(1);
        }
    }
    
    // 可以直接通过StringUtils.capitalize()调用
}
```

### @ExtensionMethod
**作用**：为已有类添加扩展方法，类似于C#的扩展方法。

**示例**：
```java
@ExtensionMethod({StringExtensions.class})
public class StringProcessor {
    public void process(String input) {
        // 直接在String对象上调用扩展方法
        String result = input.capitalize();
    }
}

class StringExtensions {
    public static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return Character.toUpperCase(str.charAt(0)) + str.substring(1);
    }
}
```

### @SuperBuilder
**作用**：增强版的@Builder，支持继承关系中的构建器模式。

**示例**：
```java
@SuperBuilder
public class Person {
    private String name;
    private int age;
}

@SuperBuilder
public class Employee extends Person {
    private String company;
    private double salary;
}

// 使用方式
Employee employee = Employee.builder()
    .name("张三")       // Person类的属性
    .age(30)           // Person类的属性
    .company("ABC公司") // Employee类的属性
    .salary(10000.0)   // Employee类的属性
    .build();
```

### @Jacksonized
**作用**：与@Builder或@SuperBuilder一起使用，使生成的构建器与Jackson序列化/反序列化兼容。

**示例**：
```java
@Jacksonized
@Builder
public class User {
    private Long id;
    private String name;
    private int age;
}

// 可以直接通过Jackson反序列化为User对象
```

### @FieldNameConstants
**作用**：为类的所有字段生成常量名称，便于反射操作和动态查询。

**示例**：
```java
@FieldNameConstants
public class User {
    private String name;
    
    @FieldDefaults(makeFinal = true)
    private Long id;
}

// 生成的常量可以这样使用
String idFieldName = User.Fields.id;    // "id"
String nameFieldName = User.Fields.name; // "name"
```

## 配置与扩展

### @lombok.experimental.Accessors
**作用**：提供更多自定义选项来控制生成的访问器方法。

**参数说明**：
- `chain`：启用链式调用，setter方法返回this
- `fluent`：生成不带get/set前缀的访问器
- `prefix`：指定要去除的字段前缀

**示例**：
```java
@Data
@Accessors(fluent = true, chain = true)
public class User {
    private Long id;
    private String name;
}

// 使用方式
User user = new User()
    .id(1L)      // 而不是setId
    .name("张三"); // 而不是setName

Long userId = user.id(); // 而不是getId
```

### @lombok.experimental.NonFinal
**作用**：与@Value一起使用，允许某些字段不是final的。

**示例**：
```java
@Value
public class ImmutableUser {
    Long id;
    String name;
    
    @NonFinal
    int loginCount; // 不是final，可以修改
}
```

### @lombok.experimental.PackagePrivate
**作用**：将字段或方法的访问级别设置为包私有。

**示例**：
```java
public class User {
    @PackagePrivate
    String name; // 包私有访问级别
    
    @PackagePrivate
    void updateName(String newName) {
        this.name = newName;
    }
}
```

### lombok.config 配置文件
**作用**：在项目级别自定义Lombok的行为。

**常用配置**：
```properties
# 停用特定注解
lombok.data.flagUsage=error
lombok.value.flagUsage=error

# 自定义toString方法的格式
lombok.toString.includeFieldNames=false
lombok.toString.doNotUseGetters=true

# 自定义equalsAndHashCode方法
lombok.equalsAndHashCode.callSuper=call

# 自定义访问器
lombok.accessors.chain=true
lombok.accessors.fluent=true

# 自定义日志注解
lombok.log.fieldName=LOGGER
lombok.log.fieldIsStatic=true
```

### @lombok.experimental.Builder.Default
**作用**：指定Builder模式中字段的默认值。

**示例**：
```java
@Builder
public class User {
    private Long id;
    private String name;
    
    @Builder.Default
    private boolean active = true;
    
    @Builder.Default
    private List<String> roles = new ArrayList<>(Arrays.asList("USER"));
}

// 使用方式
User user = User.builder()
    .id(1L)
    .name("张三")
    // 不设置active和roles，将使用默认值
    .build();
```

### @lombok.experimental.Tolerate
**作用**：为自动生成的方法提供替代实现。

**示例**：
```java
@Data
public class User {
    private String name;
    
    @Tolerate
    public void setName(String name) {
        // 自定义实现，覆盖Lombok生成的setName方法
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("名称不能为空");
        }
        this.name = name;
    }
}
```

### @lombok.experimental.Delegate.Exclude
**作用**：排除某些方法不被@Delegate委托。

**示例**：
```java
public interface UserRepository {
    User findById(Long id);
    List<User> findAll();
    void save(User user);
    void delete(User user);
}

public class UserManager {
    @Delegate(excludes = DangerousOperations.class)
    private final UserRepository repository = new UserRepositoryImpl();
    
    public interface DangerousOperations {
        void delete(User user); // 这个方法不会被委托
    }
    
    // 需要自定义实现delete方法
    public void delete(User user) {
        // 添加额外的安全检查
        if (hasPermission()) {
            repository.delete(user);
        } else {
            throw new SecurityException("没有权限删除用户");
        }
    }
    
    private boolean hasPermission() {
        // 权限检查逻辑
        return true;
    }
}
```

## 最佳实践与注意事项

### 组合使用注解

Lombok注解可以组合使用，但需要注意避免冗余和冲突：

```java
// 推荐的组合
@Value
@Builder
public class ImmutableUser {
    Long id;
    String name;
}

// 不推荐的组合（冗余）
@Getter
@Setter
@ToString
@EqualsAndHashCode
@NoArgsConstructor // 这些注解可以用@Data替代
public class User {
    private Long id;
    private String name;
}
```

### 与其他框架的集成

Lombok与其他框架集成时的注意事项：

1. **与JPA/Hibernate集成**：
   ```java
   @Entity
   @Data
   @NoArgsConstructor // JPA需要无参构造器
   public class User {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       
       private String name;
       
       @ToString.Exclude // 避免循环引用导致的StackOverflowError
       @ManyToMany
       private List<Role> roles;
   }
   ```

2. **与Jackson集成**：
   ```java
   @Data
   @Builder
   @NoArgsConstructor // Jackson需要无参构造器
   @AllArgsConstructor // 配合@Builder使用
   public class User {
       private Long id;
       private String name;
       
       @JsonIgnore // Jackson注解与Lombok兼容
       private String password;
   }
   ```

3. **与Spring框架集成**：
   ```java
   @Service
   @RequiredArgsConstructor // 自动注入final字段
   @Slf4j
   public class UserService {
       private final UserRepository userRepository;
       
       public User findById(Long id) {
           log.info("查找用户: {}", id);
           return userRepository.findById(id)
               .orElseThrow(() -> new UserNotFoundException(id));
       }
   }
   ```

### 调试与反编译

当遇到Lombok相关问题时，可以通过反编译查看生成的代码：

1. 在IDE中使用反编译功能
2. 使用`javap -c`命令
3. 使用`delombok`工具：
   ```bash
   java -jar lombok.jar delombok src -d delombok-src
   ```

## 总结

Lombok通过注解大大简化了Java开发中的样板代码，提高了代码的可读性和开发效率。在使用时，需要注意以下几点：

1. 需要在IDE中安装Lombok插件以获得代码提示和编译支持
2. 合理组合使用注解，避免冗余
3. 某些注解可能会影响代码的调试和阅读，需要团队成员都熟悉Lombok
4. 在与其他框架集成时，需要注意可能的冲突
5. 对于高级用法，建议先了解其原理，再谨慎使用