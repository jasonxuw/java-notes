# Mybatis和Mybatis-Plus常用注解

## 一、Mybatis常用注解

### 1. @Select

**注解说明**：标记查询语句，用于定义查询操作的SQL语句。

**代码示例**：
```java
@Select("SELECT * FROM users WHERE id = #{id}")
User getUserById(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Select注解并创建相应的SqlSource和MappedStatement对象。

### 2. @Insert

**注解说明**：标记插入语句，用于定义插入操作的SQL语句。

**代码示例**：
```java
@Insert("INSERT INTO users(name, age) VALUES(#{name}, #{age})")
int addUser(User user);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Insert注解并创建相应的SqlSource和MappedStatement对象。

### 3. @Update

**注解说明**：标记更新语句，用于定义更新操作的SQL语句。

**代码示例**：
```java
@Update("UPDATE users SET name = #{name}, age = #{age} WHERE id = #{id}")
int updateUser(User user);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Update注解并创建相应的SqlSource和MappedStatement对象。

### 4. @Delete

**注解说明**：标记删除语句，用于定义删除操作的SQL语句。

**代码示例**：
```java
@Delete("DELETE FROM users WHERE id = #{id}")
int deleteUserById(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Delete注解并创建相应的SqlSource和MappedStatement对象。

### 5. @Results

**注解说明**：用于指定多个@Result注解，定义查询结果集的映射关系。

**代码示例**：
```java
@Select("SELECT * FROM users WHERE id = #{id}")
@Results(id = "userResultMap", value = {
    @Result(property = "id", column = "id"),
    @Result(property = "name", column = "name"),
    @Result(property = "age", column = "age")
})
User getUserById(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Results注解并创建ResultMap对象，保存到Configuration中。

### 6. @Result

**注解说明**：用于指定单个属性与结果集中列的映射关系。

**代码示例**：
```java
@Result(property = "userId", column = "id")
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Result注解并创建ResultMapping对象，作为ResultMap的组成部分。

### 7. @ResultMap

**注解说明**：用于引用已定义的结果映射集。

**代码示例**：
```java
@Select("SELECT * FROM users WHERE id = #{id}")
@ResultMap("userResultMap")
User getUserById(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@ResultMap注解并引用已定义的ResultMap对象。

### 8. @Param

**注解说明**：用于为SQL语句中的参数指定名称，便于在SQL中引用。

**代码示例**：
```java
List<User> findByNameAndAge(@Param("name") String name, @Param("age") Integer age);
```

**注解处理类**：由`org.mybatis.spring.mapper.MapperMethod`类负责处理，它会解析@Param注解并创建参数映射，用于SQL语句的参数绑定。

### 9. @Options

**注解说明**：提供了一系列的选项配置，如是否使用自动生成的键、是否使用缓存等。

**代码示例**：
```java
@Insert("INSERT INTO users(name, age) VALUES(#{name}, #{age})")
@Options(useGeneratedKeys = true, keyProperty = "id")
int addUser(User user);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Options注解并配置MappedStatement的相关属性。

### 10. @One

**注解说明**：用于一对一关联查询。

**代码示例**：
```java
@Select("SELECT * FROM orders WHERE id = #{id}")
@Results({
    @Result(property = "id", column = "id"),
    @Result(property = "user", column = "user_id", 
            one = @One(select = "com.example.mapper.UserMapper.getUserById"))
})
Order getOrderById(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@One注解并创建一对一关联的ResultMapping对象。

### 11. @Many

**注解说明**：用于一对多关联查询。

**代码示例**：
```java
@Select("SELECT * FROM users WHERE id = #{id}")
@Results({
    @Result(property = "id", column = "id"),
    @Result(property = "orders", column = "id", 
            many = @Many(select = "com.example.mapper.OrderMapper.getOrdersByUserId"))
})
User getUserWithOrders(@Param("id") Long id);
```

**注解处理类**：由`org.apache.ibatis.builder.annotation.MapperAnnotationBuilder`类负责处理，它会解析@Many注解并创建一对多关联的ResultMapping对象。

## 二、Mybatis-Plus常用注解

### 1. @TableName

**注解说明**：用于标记实体类对应的数据库表名，解决实体类名与表名不一致的问题。

**代码示例**：
```java
@Data
@TableName("t_user")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

**注解处理类**：由`com.baomidou.mybatisplus.core.MybatisConfiguration`和`com.baomidou.mybatisplus.core.metadata.TableInfoHelper`类负责处理，它们会解析@TableName注解并建立实体类与数据库表的映射关系。

### 2. @TableId

**注解说明**：用于标记实体类主键字段，可以指定主键类型和生成策略。

**代码示例**：
```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```

**注解处理类**：由`com.baomidou.mybatisplus.core.metadata.TableInfoHelper`类负责处理，它会解析@TableId注解并确定主键字段及其生成策略。

IdType枚举值说明：
- AUTO：数据库自增
- NONE：不设置主键类型
- INPUT：用户输入ID
- ASSIGN_ID：分配ID (主要是雪花算法)
- ASSIGN_UUID：分配UUID

### 3. @TableField

**注解说明**：用于标记实体类中的非主键字段，解决实体类属性名与表字段名不一致的问题，或者指定字段的一些属性。

**代码示例**：
```java
@TableField(value = "name", insertStrategy = FieldStrategy.NOT_EMPTY)
private String userName;
```

**注解处理类**：由`com.baomidou.mybatisplus.core.metadata.TableInfoHelper`类负责处理，它会解析@TableField注解并建立实体类属性与数据库字段的映射关系。

### 4. @Version

**注解说明**：用于标记乐观锁版本号字段，实现乐观锁机制。

**代码示例**：
```java
@Version
@TableField(value = "version", fill = FieldFill.INSERT)
private Integer version;
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor`类负责处理，它会拦截SQL执行并自动增加版本号条件，实现乐观锁机制。

### 5. @TableLogic

**注解说明**：用于标记逻辑删除字段，实现逻辑删除功能。

**代码示例**：
```java
@TableLogic(value = "0", delval = "1")
@TableField(value = "deleted")
private Integer deleted;
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.plugins.inner.LogicDeleteInnerInterceptor`类负责处理，它会拦截SQL执行并自动替换删除语句为更新逻辑删除字段的语句。

### 6. @EnumValue

**注解说明**：用于标记枚举类中的属性，指定枚举字段存储的值。

**代码示例**：
```java
public enum GenderEnum implements IEnum<Integer> {
    MALE(1, "男"),
    FEMALE(2, "女");
    
    @EnumValue
    private final Integer value;
    
    private final String desc;
    
    GenderEnum(Integer value, String desc) {
        this.value = value;
        this.desc = desc;
    }
    
    @Override
    public Integer getValue() {
        return value;
    }
}
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.handlers.EnumTypeHandler`类负责处理，它会在Java枚举类型与数据库字段值之间进行转换。

### 7. @KeySequence

**注解说明**：用于指定序列生成器的名称，适用于Oracle、PostgreSQL等支持序列的数据库。

**代码示例**：
```java
@KeySequence(value = "seq_user", clazz = Long.class)
@TableName("t_user")
public class User {
    @TableId(value = "id", type = IdType.INPUT)
    private Long id;
    // 其他字段
}
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.incrementer.OracleKeyGenerator`或`com.baomidou.mybatisplus.extension.incrementer.PostgreKeyGenerator`等类负责处理，根据数据库类型选择相应的序列生成器。

### 8. @SqlParser

**注解说明**：用于指定SQL解析器，实现SQL拦截和增强功能。

**代码示例**：
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    @SqlParser(using = MyLogicSqlParser.class)
    List<User> selectAll();
}
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.parser.JsqlParserSupport`类负责处理，它会解析SQL并应用自定义的SQL解析逻辑。

### 9. @InterceptorIgnore

**注解说明**：用于指定忽略特定的拦截器，灵活控制拦截器的使用。

**代码示例**：
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    @InterceptorIgnore(tenantLine = "true")
    List<User> selectAll();
}
```

**注解处理类**：由`com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor`类负责处理，它会根据注解配置决定是否跳过特定的拦截器。

### 10. @MapperScan

**注解说明**：用于扫描Mapper接口所在的包，将其注册为Spring Bean。

**代码示例**：
```java
@SpringBootApplication
@MapperScan("com.example.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**注解处理类**：由`org.mybatis.spring.mapper.MapperScannerConfigurer`和`org.mybatis.spring.annotation.MapperScannerRegistrar`类负责处理，它们会扫描指定包下的Mapper接口并注册为Spring Bean。

### 11. @Mapper

**注解说明**：用于标记接口为Mybatis的Mapper接口，将其注册为Spring Bean。

**代码示例**：
```java
@Mapper
@Repository
public interface UserMapper extends BaseMapper<User> {
    // 自定义方法
}
```

**注解处理类**：由`org.apache.ibatis.annotations.Mapper`类负责处理，它会识别@Mapper注解并将接口注册为Mybatis的Mapper。

### 12. @DS

**注解说明**：用于多数据源切换，指定使用的数据源。

**代码示例**：
```java
@DS("slave")
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 方法将使用slave数据源
}
```

**注解处理类**：由`com.baomidou.dynamic.datasource.annotation.DS`类负责处理，它会在方法执行前后切换数据源。