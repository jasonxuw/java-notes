# SpringBoot常用注解

SpringBoot框架提供了丰富的注解，极大地简化了应用开发。本文将SpringBoot常用注解按功能分组，并提供详细说明和使用示例。

## 一、核心注解

### 1. @SpringBootApplication

这是SpringBoot应用的核心注解，标记在主类上，表明这是一个SpringBoot应用的入口。它是一个组合注解，相当于同时使用了以下三个注解：

- `@Configuration`：允许在Spring上下文中注册额外的bean或导入其他配置类
- `@EnableAutoConfiguration`：启用SpringBoot的自动配置机制
- `@ComponentScan`：扫描被`@Component`及其派生注解（如`@Service`、`@Controller`等）标记的bean

**处理类：** `SpringBootApplicationAnnotationBeanPostProcessor`类处理

**示例代码：**

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

## 二、Spring Bean相关注解

### 1. @Component

通用的注解，标识一个类为Spring组件。如果一个Bean不知道属于哪个层，可以使用`@Component`注解标注。

**处理类：** `ComponentScanAnnotationParser`类处理

**示例代码：**

```java
@Component
public class MyComponent {
    // ...
}
```

### 2. @Repository

对应持久层即Dao层，主要用于数据库相关操作。

**处理类：** `RepositoryComponentProvider`类处理

**示例代码：**

```java
@Repository
public class UserRepository {
    // 数据库操作方法
    public User findById(Long id) {
        // ...
    }
}
```

### 3. @Service

对应服务层，主要涉及一些复杂的业务逻辑，需要用到Dao层。

**处理类：** `ServiceAnnotationBeanPostProcessor`类处理

**示例代码：**

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
}
```

### 4. @Controller

对应Spring MVC控制层，主要用于接收用户请求并调用Service层返回数据给前端页面。

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@Controller
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        model.addAttribute("user", userService.getUserById(id));
        return "userDetail";
    }
}
```

### 5. @RestController

`@RestController`注解是`@Controller`和`@ResponseBody`的组合，表示这是个控制器bean，并且将函数的返回值直接填入HTTP响应体中，是REST风格的控制器。

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@RestController
@RequestMapping("/api/users")
public class UserApiController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}
```

### 6. @Autowired

自动导入对象到类中，被注入进的类同样要被Spring容器管理。

**处理类：** `AutowiredAnnotationBeanPostProcessor`类处理

**示例代码：**

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    // 使用构造函数注入（推荐方式）
    private final RoleService roleService;
    
    @Autowired
    public UserService(RoleService roleService) {
        this.roleService = roleService;
    }
}
```

### 7. @Scope

声明Spring Bean的作用域。

**处理类：** `ScopeMetadataResolver`类处理

**示例代码：**

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // 每次请求都会创建一个新的bean实例
}

@Component
@Scope("singleton")
public class SingletonBean {
    // 默认作用域，整个应用中只创建一个bean实例
}
```

常见的作用域：
- singleton：默认作用域，单例模式，整个应用中只创建一个bean实例
- prototype：每次请求都会创建一个新的bean实例
- request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效
- session：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP session内有效

### 8. @Configuration

标识一个类为配置类，可以使用`@Bean`注解来定义bean。

**处理类：** `ConfigurationClassPostProcessor`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

### 9. 条件注解

#### @Conditional

基础条件注解，根据满足某一特定条件创建一个特定的Bean。

**处理类：** `ConditionEvaluator`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @Conditional(WindowsCondition.class)
    public WindowsService windowsService() {
        return new WindowsServiceImpl();
    }
}

public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        return env.getProperty("os.name").toLowerCase().contains("windows");
    }
}
```

#### @ConditionalOnBean

仅当指定的Bean存在于BeanFactory中时，才会创建这个Bean。

**处理类：** `OnBeanCondition`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnBean(DataSource.class)
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

#### @ConditionalOnMissingBean

仅当指定的Bean不存在于BeanFactory中时，才会创建这个Bean。

**处理类：** `OnBeanCondition`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnMissingBean
    public DataSource defaultDataSource() {
        // 只有在没有其他DataSource Bean时才创建默认数据源
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
}
```

#### @ConditionalOnClass

仅当指定的类存在于类路径上时，才会创建这个Bean。

**处理类：** `OnClassCondition`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnClass(name = "org.springframework.data.redis.core.RedisTemplate")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        // 只有当Redis相关类在类路径上时才创建RedisTemplate
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        return template;
    }
}
```

#### @ConditionalOnMissingClass

仅当指定的类不存在于类路径上时，才会创建这个Bean。

**处理类：** `OnClassCondition`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnMissingClass("com.mysql.jdbc.Driver")
    public DataSource h2DataSource() {
        // 只有当MySQL驱动不在类路径上时才创建H2数据源
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
}
```

#### @ConditionalOnProperty

仅当指定的属性具有指定的值时，才会创建这个Bean。

**处理类：** `OnPropertyCondition`类处理

**示例代码：**

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        // 只有当app.cache.enabled=true时才创建缓存管理器
        return new ConcurrentMapCacheManager();
    }
}
```

#### @ConditionalOnWebApplication

仅当当前应用是Web应用时，才会创建这个Bean。

**处理类：** `OnWebApplicationCondition`类处理

**示例代码：**

```java
@Configuration
public class WebConfig {
    @Bean
    @ConditionalOnWebApplication
    public FilterRegistrationBean<CorsFilter> corsFilter() {
        // 只有在Web应用中才创建CORS过滤器
        FilterRegistrationBean<CorsFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new CorsFilter());
        bean.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return bean;
    }
}
```

#### @ConditionalOnNotWebApplication

仅当当前应用不是Web应用时，才会创建这个Bean。

**处理类：** `OnWebApplicationCondition`类处理

**示例代码：**

```java
@Configuration
public class BatchConfig {
    @Bean
    @ConditionalOnNotWebApplication
    public JobLauncher batchJobLauncher() {
        // 只有在非Web应用中才创建批处理作业启动器
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
        // 配置作业启动器
        return jobLauncher;
    }
}
```

## 三、HTTP请求处理相关注解

### 1. @RequestMapping

用于映射Web请求，包括访问路径和参数。

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@RestController
public class UserController {
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    public List<User> getAllUsers() {
        // 返回所有用户
    }
}
```

### 2. HTTP方法特定注解

Spring提供了一系列针对HTTP方法的简化注解：

#### @GetMapping

等价于`@RequestMapping(method = RequestMethod.GET)`

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@GetMapping("/users")
public List<User> getAllUsers() {
    // 处理GET请求
}
```

#### @PostMapping

等价于`@RequestMapping(method = RequestMethod.POST)`

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    // 处理POST请求，创建用户
}
```

#### @PutMapping

等价于`@RequestMapping(method = RequestMethod.PUT)`

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@PutMapping("/users/{id}")
public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
    // 处理PUT请求，更新用户
}
```

#### @DeleteMapping

等价于`@RequestMapping(method = RequestMethod.DELETE)`

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    // 处理DELETE请求，删除用户
}
```

#### @PatchMapping

等价于`@RequestMapping(method = RequestMethod.PATCH)`

**处理类：** `RequestMappingHandlerMapping`类处理

**示例代码：**

```java
@PatchMapping("/users/{id}")
public ResponseEntity<User> partialUpdateUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    // 处理PATCH请求，部分更新用户
}
```

## 四、参数绑定注解

### 1. @PathVariable

用于获取URL路径中的参数。

**处理类：** `PathVariableMethodArgumentResolver`类处理

**示例代码：**

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable("id") Long userId) {
    // 获取路径中的id参数
}
```

### 2. @RequestParam

用于获取查询参数。

**处理类：** `RequestParamMethodArgumentResolver`类处理

**示例代码：**

```java
@GetMapping("/users")
public List<User> getUsersByRole(@RequestParam(value = "role", required = false) String role) {
    // 获取查询参数role的值
}
```

### 3. @RequestBody

用于读取请求体中的JSON数据并自动绑定到Java对象。

**处理类：** `RequestResponseBodyMethodProcessor`类处理

**示例代码：**

```java
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    // user对象会自动从请求体的JSON数据映射而来
    return userService.save(user);
}
```

### 4. @RequestHeader

用于获取请求头信息。

**处理类：** `RequestHeaderMethodArgumentResolver`类处理

**示例代码：**

```java
@GetMapping("/greeting")
public String greeting(@RequestHeader("User-Agent") String userAgent) {
    // 获取请求头中的User-Agent信息
    return "Your User-Agent is: " + userAgent;
}
```

## 五、配置属性注解

### 1. @Value

用于获取配置文件中的属性值。

**处理类：** `AutowiredAnnotationBeanPostProcessor`类处理

**示例代码：**

```java
@Component
public class AppProperties {
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.description}")
    private String appDescription;
}
```

### 2. @ConfigurationProperties

用于将配置文件中的属性批量注入到Java对象中。

**处理类：** `ConfigurationPropertiesBindingPostProcessor`类处理

**示例代码：**

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private String description;
    private List<String> servers;
    
    // getters and setters
}
```

对应的配置文件：

```yaml
app:
  name: MyApp
  description: My awesome application
  servers:
    - dev.example.com
    - prod.example.com
```

### 3. @PropertySource

用于指定加载特定的属性文件。

**处理类：** `PropertySourcesPlaceholderConfigurer`类处理

**示例代码：**

```java
@Component
@PropertySource("classpath:custom.properties")
public class CustomProperties {
    @Value("${custom.property}")
    private String customProperty;
}
```

## 六、异常处理注解

### 1. @ControllerAdvice

全局异常处理和应用到所有@RequestMapping方法。

**处理类：** `ControllerAdviceBean`类处理

**示例代码：**

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

### 2. @ExceptionHandler

用于处理特定异常。

**处理类：** `ExceptionHandlerExceptionResolver`类处理

**示例代码：**

```java
@RestController
public class UserController {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {
        // 处理UserNotFoundException异常
        return new ResponseEntity<>(new ErrorResponse(ex.getMessage()), HttpStatus.NOT_FOUND);
    }
}
```

## 七、事务注解

### @Transactional

用于声明事务。

**处理类：** `TransactionInterceptor`类处理

**示例代码：**

```java
@Service
public class UserService {
    
    @Transactional
    public void createUserWithRoles(User user, List<Role> roles) {
        userRepository.save(user);
        roleRepository.saveAll(roles);
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void updateUser(User user) {
        // 发生任何异常都回滚事务
    }
}
```

## 八、缓存注解

### 1. @Cacheable

标记方法的结果需要被缓存。

**处理类：** `CacheInterceptor`类处理

**示例代码：**

```java
@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User getUserById(Long id) {
        // 如果缓存中有数据，则不会执行此方法
        return userRepository.findById(id).orElse(null);
    }
}
```

### 2. @CacheEvict

标记要清除缓存的方法。

**处理类：** `CacheInterceptor`类处理

**示例代码：**

```java
@Service
public class UserService {
    
    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(User user) {
        userRepository.save(user);
    }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllUsersCache() {
        // 清除users缓存中的所有条目
    }
}
```

### 3. @CachePut

更新缓存而不干扰方法执行。

**处理类：** `CacheInterceptor`类处理

**示例代码：**

```java
@Service
public class UserService {
    
    @CachePut(value = "users", key = "#user.id")
    public User updateUserDetails(User user) {
        // 方法始终会被执行，然后结果被放入缓存
        return userRepository.save(user);
    }
}
```

## 九、测试相关注解

### 1. @SpringBootTest

用于SpringBoot集成测试。

**处理类：** `SpringBootTestContextBootstrapper`类处理

**示例代码：**

```java
@SpringBootTest
public class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    public void testGetUserById() {
        User user = userService.getUserById(1L);
        assertNotNull(user);
        assertEquals("admin", user.getUsername());
    }
}
```

### 2. @MockBean

创建并注入一个Mockito mock。

**处理类：** `MockitoPostProcessor`类处理

**示例代码：**

```java
@SpringBootTest
public class UserServiceTest {
    
    @MockBean
    private UserRepository userRepository;
    
    @Autowired
    private UserService userService;
    
    @Test
    public void testGetUserById() {
        User mockUser = new User(1L, "admin");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));
        
        User user = userService.getUserById(1L);
        assertEquals("admin", user.getUsername());
    }
}
```

## 十、安全相关注解

### 1. @EnableWebSecurity

启用Spring Security的Web安全支持。

**处理类：** `WebSecurityConfiguration`类处理

**示例代码：**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/public/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin();
    }
}
```

### 2. @PreAuthorize

方法执行前进行权限检查。

**处理类：** `AuthorizationAttributeSourceAdvisor`类处理

**示例代码：**

```java
@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        // 只有ADMIN角色的用户才能访问
        return userRepository.findAll();
    }
}
```

## 十一、JPA相关注解

### 1. @Entity

标记类为JPA实体。

**处理类：** `EntityManagerFactoryBuilderImpl`类处理

**示例代码：**

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    
    // getters and setters
}
```

### 2. @Repository

标记接口为JPA Repository。

**处理类：** `JpaRepositoriesRegistrar`类处理

**示例代码：**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByUsername(String username);
    
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

## 十二、JSON处理注解

### 1. @JsonIgnore

在序列化过程中忽略某个属性。

**处理类：** `JacksonAnnotationIntrospector`类处理

**示例代码：**

```java
public class User {
    private Long id;
    private String username;
    
    @JsonIgnore
    private String password; // 在JSON响应中不会包含此字段
}
```

### 2. @JsonFormat

格式化日期或时间字段。

**处理类：** `JacksonAnnotationIntrospector`类处理

**示例代码：**

```java
public class User {
    private Long id;
    private String username;
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createdAt;
}
```

## 十三、AOP相关注解

### 1. @Aspect

标记一个类为切面，包含通知（Advice）和切点（Pointcut）。

**处理类：** `AnnotationAwareAspectJAutoProxyCreator`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    // 切面实现
}
```

### 2. @Pointcut

定义切点表达式，指定在哪些方法上应用通知。

**处理类：** `AspectJExpressionPointcut`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    @Before("serviceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        // 在服务方法执行前记录日志
    }
}
```

### 3. @Before

前置通知，在目标方法执行前执行。

**处理类：** `AspectJMethodBeforeAdvice`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("Before method: " + methodName);
    }
}
```

### 4. @After

后置通知，在目标方法执行后执行（无论是否抛出异常）。

**处理类：** `AspectJAfterAdvice`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    
    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("After method: " + methodName);
    }
}
```

### 5. @AfterReturning

返回通知，在目标方法成功执行并返回结果后执行。

**处理类：** `AspectJAfterReturningAdvice`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("Method " + methodName + " returned: " + result);
    }
}
```

### 6. @AfterThrowing

异常通知，在目标方法抛出异常后执行。

**处理类：** `AspectJAfterThrowingAdvice`类处理

**示例代码：**

```java
@Aspect
@Component
public class LoggingAspect {
    
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("Method " + methodName + " threw exception: " + ex.getMessage());
    }
}
```

### 7. @Around

环绕通知，可以在目标方法执行前后自定义行为，甚至可以决定是否执行目标方法。

**处理类：** `AspectJAroundAdvice`类处理

**示例代码：**

```java
@Aspect
@Component
public class PerformanceMonitoringAspect {
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        Object result = joinPoint.proceed(); // 执行目标方法
        
        long endTime = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().getName();
        System.out.println("Method " + methodName + " executed in " + (endTime - startTime) + "ms");
        
        return result;
    }
}
```

### 8. @EnableAspectJAutoProxy

启用AspectJ自动代理，通常在配置类上使用。

**处理类：** `AspectJAutoProxyRegistrar`类处理

**示例代码：**

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    // 配置
}
```