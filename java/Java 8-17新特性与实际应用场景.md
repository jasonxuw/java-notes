## 版本演进时间轴

| 版本 | 发布日期 | 代号 | 重大特性 | 主要应用场景 |
|-----|---------|------|---------|------------|
| Java 8 (LTS) | 2014年3月 | - | Lambda表达式、Stream API、新日期时间API | 函数式编程、集合处理优化、时间处理标准化 |
| Java 9 | 2017年9月 | - | 模块系统(Jigsaw)、JShell、集合工厂方法 | 应用模块化、交互式开发、代码简化 |
| Java 10 | 2018年3月 | - | 局部变量类型推断(var)、G1并行Full GC | 代码简化、GC性能提升 |
| Java 11 (LTS) | 2018年9月 | - | HTTP Client API、String新方法、ZGC | 现代HTTP通信、字符串处理增强、低延迟GC |
| Java 12 | 2019年3月 | - | Switch表达式(预览)、Shenandoah GC | 代码简化、低延迟GC |
| Java 13 | 2019年9月 | - | 文本块(预览)、ZGC改进 | 多行字符串处理、GC性能提升 |
| Java 14 | 2020年3月 | - | Switch表达式(正式)、Records(预览)、Pattern Matching(预览) | 代码简洁性、数据类简化、条件处理优化 |
| Java 15 | 2020年9月 | - | 文本块(正式)、Records(第二预览)、Sealed Classes(预览) | 多行字符串处理、数据类简化、类型系统增强 |
| Java 16 | 2021年3月 | - | Records(正式)、Pattern Matching(第二预览)、Vector API(孵化) | 数据类简化、条件处理优化、向量计算 |
| Java 17 (LTS) | 2021年9月 | - | Sealed Classes(正式)、Pattern Matching for switch(预览)、强封装JDK内部API | 类型系统增强、模式匹配、安全性提升 |

## Java 8 核心特性解析

Java 8是一次革命性的版本更新，引入了函数式编程范式，显著改变了Java开发模式。作为长期支持(LTS)版本，它至今仍被广泛使用。

### Lambda表达式与函数式接口

#### 特性概述
Lambda表达式是Java 8引入的最重要特性，它为Java带来了函数式编程的能力，使代码更加简洁、可读性更强。

#### 技术细节
Lambda表达式本质上是一个匿名函数，由参数列表、箭头操作符和函数体组成：
```java
// 基本语法: (参数) -> { 表达式 }
Runnable r1 = () -> System.out.println("Hello Lambda"); // 无参数
Comparator<Integer> c1 = (a, b) -> a.compareTo(b);      // 多参数
Function<String, Integer> f1 = s -> s.length();         // 单参数可省略括号
```

函数式接口是Lambda表达式的基础，它是只包含一个抽象方法的接口，可以使用`@FunctionalInterface`注解标记：
```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // 默认方法不影响函数式接口的定义
    default void printInfo() {
        System.out.println("这是一个计算器接口");
    }
}

// 使用Lambda表达式实现
Calculator addition = (a, b) -> a + b;
Calculator subtraction = (a, b) -> a - b;
```

#### 应用场景
1. **集合迭代与过滤**
```java
// 传统方式
for (String name : names) {
    if (name.startsWith("A")) {
        System.out.println(name);
    }
}

// Lambda方式
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```

2. **事件处理**
```java
// 传统方式
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("按钮被点击");
    }
});

// Lambda方式
button.addActionListener(e -> System.out.println("按钮被点击"));
```

3. **多线程实现**
```java
// 传统方式
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}).start();

// Lambda方式
new Thread(() -> System.out.println("线程执行")).start();
```

### Stream API

#### 特性概述
Stream API提供了一种函数式的集合操作方式，支持串行和并行处理，极大地简化了集合操作。

#### 技术细节
Stream操作分为中间操作和终端操作：
- 中间操作：返回一个新的Stream，如filter、map、sorted等
- 终端操作：返回一个结果或副作用，如collect、forEach、reduce等

```java
// Stream创建
Stream<String> streamFromCollection = list.stream();          // 从集合创建
Stream<String> streamFromArray = Arrays.stream(array);        // 从数组创建
Stream<Integer> streamFromValues = Stream.of(1, 2, 3);        // 从值创建
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n+2); // 无限流创建

// Stream操作链
List<String> result = persons.stream()                        // 获取流
    .filter(p -> p.getAge() > 18)                             // 过滤
    .map(Person::getName)                                     // 映射转换
    .sorted()                                                 // 排序
    .limit(10)                                                // 限制数量
    .collect(Collectors.toList());                            // 收集结果
```

#### 应用场景
1. **数据转换**
```java
// 将人员列表转换为姓名列表
List<String> names = persons.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

2. **数据统计**
```java
// 计算所有人的平均年龄
double averageAge = persons.stream()
    .mapToInt(Person::getAge)
    .average()
    .orElse(0);
```

3. **数据分组**
```java
// 按年龄段分组
Map<AgeGroup, List<Person>> personsByAgeGroup = persons.stream()
    .collect(Collectors.groupingBy(person -> {
        if (person.getAge() < 18) return AgeGroup.UNDER_18;
        else if (person.getAge() < 65) return AgeGroup.ADULT;
        else return AgeGroup.SENIOR;
    }));
```

### 方法引用

#### 特性概述
方法引用是Lambda表达式的一种简化形式，当Lambda表达式的内容仅仅是调用一个已存在的方法时，可以使用方法引用来替代。

#### 技术细节
方法引用有四种形式：
```java
// 1. 静态方法引用：ClassName::staticMethodName
Function<String, Integer> parseInt = Integer::parseInt;

// 2. 实例方法引用：instance::methodName
String str = "Hello";
Supplier<Integer> getLength = str::length;

// 3. 对象方法引用：ClassName::methodName
Function<String, Integer> getStringLength = String::length;

// 4. 构造方法引用：ClassName::new
Supplier<Person> personCreator = Person::new;
```

#### 应用场景
1. **集合操作**
```java
// 打印集合元素
list.forEach(System.out::println);

// 获取所有人的名字
List<String> names = persons.stream()
    .map(Person::getName)  // 使用方法引用替代 p -> p.getName()
    .collect(Collectors.toList());
```

2. **排序操作**
```java
// 按姓名排序
persons.sort(Comparator.comparing(Person::getName));
```

### 默认方法与静态方法

#### 特性概述
Java 8允许在接口中定义默认方法和静态方法，这使得接口的设计更加灵活，同时保持了向后兼容性。

#### 技术细节
```java
public interface Vehicle {
    // 抽象方法
    void start();
    
    // 默认方法
    default void honk() {
        System.out.println("嘟嘟");
    }
    
    // 静态方法
    static Vehicle createElectricVehicle() {
        return new ElectricVehicle();
    }
}
```

#### 应用场景
1. **API演进**
```java
// 在Collection接口中添加新方法而不破坏现有实现
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

2. **多重继承行为**
```java
public interface Flyable {
    default void fly() {
        System.out.println("飞行中");
    }
}

public interface Swimmable {
    default void swim() {
        System.out.println("游泳中");
    }
}

// 一个类可以实现多个带有默认方法的接口
public class Duck implements Flyable, Swimmable {
    // 同时获得了fly()和swim()方法
}
```

### 新日期时间API

#### 特性概述
Java 8引入了全新的日期时间API (java.time包)，解决了旧API的设计缺陷，提供了不可变、线程安全的日期时间处理类。

#### 技术细节
核心类包括：
- LocalDate：表示日期 (年-月-日)
- LocalTime：表示时间 (时:分:秒.毫秒)
- LocalDateTime：表示日期和时间
- ZonedDateTime：带时区的日期和时间
- Instant：时间戳
- Duration：两个时间之间的间隔
- Period：两个日期之间的间隔

```java
// 创建日期时间
LocalDate date = LocalDate.of(2023, 3, 25);
LocalTime time = LocalTime.of(13, 45, 20);
LocalDateTime dateTime = LocalDateTime.of(date, time);
ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));

// 日期时间操作
LocalDate tomorrow = date.plusDays(1);
LocalDate lastMonth = date.minusMonths(1);
boolean isLeapYear = date.isLeapYear();

// 日期时间格式化
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formattedDateTime = dateTime.format(formatter);
```

#### 应用场景
1. **日期计算**
```java
// 计算两个日期之间的天数
LocalDate startDate = LocalDate.of(2023, 1, 1);
LocalDate endDate = LocalDate.of(2023, 12, 31);
long daysBetween = ChronoUnit.DAYS.between(startDate, endDate);
```

2. **时区处理**
```java
// 在不同时区之间转换时间
ZonedDateTime tokyoTime = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
ZonedDateTime newYorkTime = tokyoTime.withZoneSameInstant(ZoneId.of("America/New_York"));
```

3. **日期时间解析**
```java
// 解析日期字符串
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate date = LocalDate.parse("2023-03-25", formatter);
``` 

## Java 9 核心特性解析

Java 9在2017年9月发布，带来了模块系统等重大变革，是Java平台现代化的重要一步。

### 模块系统 (Project Jigsaw)

#### 特性概述
模块系统是Java 9最重要的特性，旨在解决Java平台和应用程序的可伸缩性问题，提供更好的封装性和依赖管理。

#### 技术细节
模块是一个自描述的Java代码和数据的集合，通过`module-info.java`文件定义：

```java
// module-info.java
module com.example.myapp {
    // 导出包，使其对其他模块可见
    exports com.example.myapp.api;
    
    // 声明对其他模块的依赖
    requires java.base;  // 隐式依赖，可省略
    requires java.sql;
    
    // 允许反射访问
    opens com.example.myapp.internal to com.example.framework;
    
    // 提供服务实现
    provides com.example.service.MyService with com.example.myapp.impl.MyServiceImpl;
    
    // 使用服务
    uses com.example.service.OtherService;
}
```

模块系统的主要概念：
- 强封装：默认情况下，模块中的包对其他模块不可见
- 显式依赖：模块必须声明其依赖关系
- 可靠配置：编译时和运行时都会验证模块依赖
- 模块化JDK：JDK本身被分解为约100个模块

#### 应用场景
1. **大型应用模块化**
```java
// 应用API模块
module app.api {
    exports com.app.api;
}

// 应用实现模块
module app.impl {
    requires app.api;
    requires database.api;
    
    provides com.app.api.UserService with com.app.impl.UserServiceImpl;
}

// 数据库API模块
module database.api {
    exports com.db.api;
}

// 数据库实现模块
module database.impl {
    requires database.api;
    
    provides com.db.api.Repository with com.db.impl.JdbcRepository;
}
```

2. **创建自定义运行时镜像**
```bash
# 使用jlink创建包含所需模块的自定义运行时
jlink --module-path $JAVA_HOME/jmods:mods --add-modules app.main --output appruntime
```

3. **服务加载**
```java
// 使用ServiceLoader加载服务实现
ServiceLoader<MyService> services = ServiceLoader.load(MyService.class);
for (MyService service : services) {
    service.doSomething();
}
```

### JShell - 交互式Java REPL

#### 特性概述
JShell是Java 9引入的交互式编程环境，允许开发者快速测试Java代码片段，无需编写完整的类或方法。

#### 技术细节
JShell支持以下功能：
- 执行表达式、语句和声明
- 自动导入常用包
- 代码补全和语法高亮
- 访问执行历史
- 保存和加载会话

```bash
# 启动JShell
$ jshell

# 执行简单表达式
jshell> 2 + 2
$1 ==> 4

# 定义变量
jshell> String greeting = "Hello, World!"
greeting ==> "Hello, World!"

# 定义方法
jshell> int sum(int a, int b) { return a + b; }
|  已创建 方法 sum(int,int)

# 使用方法
jshell> sum(10, 20)
$4 ==> 30

# 查看已定义的变量和方法
jshell> /vars
|    String greeting = "Hello, World!"
|    int $4 = 30

jshell> /methods
|    int sum(int,int)
```

#### 应用场景
1. **快速原型开发**
```
// 测试新API
jshell> import java.net.http.*
jshell> var client = HttpClient.newHttpClient()
jshell> var request = HttpRequest.newBuilder().uri(URI.create("https://example.com")).build()
jshell> var response = client.send(request, HttpResponse.BodyHandlers.ofString())
jshell> response.body()
```

2. **教学和学习**
```
// 演示语言特性
jshell> var list = List.of(1, 2, 3, 4, 5)
jshell> list.stream().filter(n -> n % 2 == 0).map(n -> n * n).toList()
$2 ==> [4, 16]
```

3. **API探索**
```
// 探索类方法
jshell> /imports
|    import java.io.*
|    import java.math.*
|    import java.net.*
// ...

jshell> String.class.getMethods()
// 显示String类的所有方法
```

### 集合工厂方法

#### 特性概述
Java 9引入了创建不可变集合的便捷工厂方法，使创建小型集合实例更加简洁。

#### 技术细节
新增的工厂方法包括：
- `List.of()`
- `Set.of()`
- `Map.of()`和`Map.ofEntries()`

```java
// 创建不可变List
List<String> list = List.of("Java", "Python", "JavaScript");

// 创建不可变Set
Set<Integer> set = Set.of(1, 2, 3, 4, 5);

// 创建不可变Map (最多10个键值对)
Map<String, Integer> map = Map.of(
    "Java", 1995,
    "Python", 1991,
    "JavaScript", 1995
);

// 创建不可变Map (超过10个键值对)
Map<String, Integer> largeMap = Map.ofEntries(
    Map.entry("Java", 1995),
    Map.entry("Python", 1991),
    Map.entry("JavaScript", 1995),
    // ...更多条目
);
```

这些集合有以下特点：
- 不可变（不支持添加、删除或替换元素）
- 不允许null元素
- 结构紧凑，内存效率高
- 对于Map.of()，最多支持10个键值对

#### 应用场景
1. **常量集合定义**
```java
// 定义常量列表
private static final List<String> SUPPORTED_LANGUAGES = 
    List.of("Java", "Kotlin", "Scala", "Groovy");
```

2. **方法返回值**
```java
// 返回不可变集合
public List<User> getDefaultUsers() {
    return List.of(
        new User("admin", Role.ADMIN),
        new User("guest", Role.GUEST)
    );
}
```

3. **参数传递**
```java
// 传递不可变集合作为参数
processItems(Set.of("item1", "item2", "item3"));
```

### 接口私有方法

#### 特性概述
Java 9允许在接口中定义私有方法，进一步增强了接口的封装能力，使默认方法的代码复用更加便捷。

#### 技术细节
接口可以定义两种私有方法：
- 私有实例方法：供默认方法调用
- 私有静态方法：供静态方法和默认方法调用

```java
public interface Logger {
    // 公共抽象方法
    void log(String message);
    
    // 默认方法
    default void logInfo(String message) {
        log(addSeverity("INFO", message));
    }
    
    default void logWarning(String message) {
        log(addSeverity("WARNING", message));
    }
    
    default void logError(String message) {
        log(addSeverity("ERROR", message));
    }
    
    // 私有方法 - 供默认方法复用
    private String addSeverity(String severity, String message) {
        return "[" + severity + "] " + message;
    }
    
    // 私有静态方法
    private static String getCurrentTimestamp() {
        return LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
    }
}
```

#### 应用场景
1. **代码复用**
```java
public interface PaymentProcessor {
    void processPayment(Payment payment);
    
    default void processDebitPayment(DebitPayment payment) {
        validatePayment(payment);
        processPayment(payment);
    }
    
    default void processCreditPayment(CreditPayment payment) {
        validatePayment(payment);
        processPayment(payment);
    }
    
    // 私有辅助方法
    private void validatePayment(Payment payment) {
        if (payment.getAmount() <= 0) {
            throw new IllegalArgumentException("Payment amount must be positive");
        }
        // 其他验证逻辑
    }
}
```

2. **接口实现辅助**
```java
public interface DataProcessor {
    void process(List<String> data);
    
    default void processFile(Path filePath) throws IOException {
        process(readLines(filePath));
    }
    
    // 私有辅助方法
    private List<String> readLines(Path filePath) throws IOException {
        return Files.readAllLines(filePath);
    }
}
```

### 改进的Stream API

#### 特性概述
Java 9对Stream API进行了增强，添加了几个新的方法，使流处理更加灵活和强大。

#### 技术细节
新增的Stream方法包括：
- `takeWhile()`: 依次获取满足条件的元素，直到遇到第一个不满足条件的元素
- `dropWhile()`: 依次丢弃满足条件的元素，直到遇到第一个不满足条件的元素
- `ofNullable()`: 创建一个包含单个元素的流，如果元素为null则创建空流
- `iterate()`: 增强版的迭代方法，支持终止条件

```java
// takeWhile 示例
Stream.of(1, 2, 3, 4, 5, 1, 2)
      .takeWhile(n -> n < 4)  // 获取元素直到遇到 >= 4 的元素
      .forEach(System.out::println);  // 输出: 1, 2, 3

// dropWhile 示例
Stream.of(1, 2, 3, 4, 5, 1, 2)
      .dropWhile(n -> n < 4)  // 丢弃元素直到遇到 >= 4 的元素
      .forEach(System.out::println);  // 输出: 4, 5, 1, 2

// ofNullable 示例
Stream<String> stream = Stream.ofNullable(getNullableValue());
// 如果getNullableValue()返回null，则stream是空流

// 带终止条件的iterate
Stream.iterate(1, n -> n <= 100, n -> n * 2)  // 从1开始，每次乘2，直到超过100
      .forEach(System.out::println);  // 输出: 1, 2, 4, 8, 16, 32, 64
```

#### 应用场景
1. **处理有序数据**
```java
// 处理有序日志，直到遇到错误日志
logs.stream()
    .takeWhile(log -> log.getLevel() != LogLevel.ERROR)
    .forEach(System.out::println);
```

2. **数据分段处理**
```java
// 跳过所有小于阈值的数据，处理其余数据
measurements.stream()
    .dropWhile(m -> m.getValue() < threshold)
    .forEach(processor::process);
```

3. **可能为空的数据处理**
```java
// 处理可能为null的用户数据
Stream.ofNullable(getUserData())
      .flatMap(Collection::stream)
      .forEach(this::processUserData);
```

### 其他重要特性

#### HTTP/2客户端 (孵化)
Java 9引入了新的HTTP客户端API (incubator)，支持HTTP/2和WebSocket：

```java
// 创建HTTP客户端
HttpClient client = HttpClient.newHttpClient();

// 构建请求
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .GET()
    .build();

// 发送同步请求
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println("状态码: " + response.statusCode());
System.out.println("响应体: " + response.body());

// 发送异步请求
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join(); // 等待完成

// POST请求示例
HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com/users"))
    .timeout(Duration.ofSeconds(30))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"张三\",\"age\":30}"))
    .build();
```

#### 多版本JAR文件
Java 9支持创建包含针对不同Java版本的类文件的JAR包：

```
multi-release-jar/
├── META-INF/
│   └── versions/
│       ├── 9/
│       │   └── com/example/
│       │       └── MyClass.class  # Java 9特定实现
│       └── 10/
│           └── com/example/
│               └── MyClass.class  # Java 10特定实现
└── com/
    └── example/
        └── MyClass.class          # 基础实现
```

#### 改进的Javadoc
Java 9的Javadoc支持HTML5和搜索功能，并添加了新的标签：

```java
/**
 * 这是一个示例类
 * 
 * @apiNote 这个API的使用注意事项
 * @implSpec 实现规范说明
 * @implNote 实现细节说明
 */
public class Example {
    // ...
}
```

## Java 10 核心特性解析

Java 10于2018年3月发布，是Java采用新的六个月发布周期后的第一个版本，虽然特性较少，但引入了一些实用的改进。

### 局部变量类型推断 (var)

#### 特性概述
Java 10引入了局部变量类型推断，允许使用`var`关键字声明局部变量，编译器会根据变量的初始化表达式自动推断其类型。

#### 技术细节
`var`关键字的使用有以下限制：
- 只能用于局部变量声明，不能用于方法参数、字段、返回类型等
- 声明时必须初始化变量
- 不能将值设为null
- 不能用于lambda表达式的参数

```java
// 基本用法
var text = "Hello, World!";              // 推断为String
var numbers = List.of(1, 2, 3, 4, 5);    // 推断为List<Integer>
var map = new HashMap<String, Integer>(); // 推断为HashMap<String, Integer>

// 循环中使用
for (var i = 0; i < 10; i++) {
    System.out.println(i);
}

// 增强for循环
for (var item : collection) {
    System.out.println(item);
}

// try-with-resources
try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    // 使用reader
}
```

这些集合有以下特点：
- 不可变（不支持添加、删除或替换元素）
- 不允许null元素
- 结构紧凑，内存效率高
- 对于Map.of()，最多支持10个键值对

#### 应用场景
1. **减少冗长的类型声明**
```java
// 传统方式
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

// 使用var
var reader = new BufferedReader(new InputStreamReader(System.in));
```

2. **使用匿名类**
```java
// 传统方式
Comparator<String> comparator = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};

// 使用var
var comparator = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};
```

3. **复杂泛型类型**
```java
// 传统方式
Map<String, List<Map<Integer, String>>> complexMap = new HashMap<>();

// 使用var
var complexMap = new HashMap<String, List<Map<Integer, String>>>();
```

### G1垃圾收集器的并行Full GC

#### 特性概述
Java 10将G1垃圾收集器的Full GC实现改为并行，显著提高了G1收集器在最坏情况下的性能。

#### 技术细节
G1垃圾收集器在Java 9成为默认垃圾收集器，但其Full GC过程仍然是单线程的，这在大型堆上可能导致长时间停顿。Java 10改进了这一点：

- 使用多线程并行标记-清除-压缩算法
- 与之前的单线程实现相比，大幅减少了Full GC的停顿时间
- 通过`-XX:ParallelGCThreads`参数控制并行线程数

```bash
# 启用G1收集器（Java 9+默认启用）
java -XX:+UseG1GC -XX:ParallelGCThreads=4 MyApplication
```

#### 应用场景
1. **大内存服务器应用**
```
# 为大型服务器应用配置G1
java -XX:+UseG1GC -Xms4g -Xmx4g -XX:ParallelGCThreads=8 -XX:ConcGCThreads=2 ServerApplication
```

2. **低延迟要求的应用**
```
# 优化低延迟应用的GC配置
java -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:InitiatingHeapOccupancyPercent=45 LowLatencyApp
```

### 应用类数据共享 (Application Class-Data Sharing)

#### 特性概述
Java 10扩展了类数据共享(CDS)功能，允许将应用类放入共享存档中，减少启动时间和内存占用。

#### 技术细节
应用类数据共享(AppCDS)的工作流程：
1. 创建类列表
2. 创建共享存档
3. 使用共享存档启动应用

```bash
# 步骤1：生成类列表
java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.lst -cp app.jar MyApp

# 步骤2：创建共享存档
java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=classes.lst -XX:SharedArchiveFile=app.jsa -cp app.jar

# 步骤3：使用共享存档启动应用
java -Xshare:on -XX:+UseAppCDS -XX:SharedArchiveFile=app.jsa -cp app.jar MyApp
```

#### 应用场景
1. **微服务和容器化应用**
```bash
# 为Docker容器中的微服务创建共享存档
java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=microservice.lst -XX:SharedArchiveFile=microservice.jsa -cp microservice.jar
```

2. **频繁启动的应用**
```bash
# 优化命令行工具的启动时间
java -Xshare:on -XX:+UseAppCDS -XX:SharedArchiveFile=tool.jsa -cp tool.jar CommandLineTool
```

### 其他重要特性

#### 基于时间的发布版本控制
Java 10开始采用基于时间的版本号格式：`$FEATURE.$INTERIM.$UPDATE.$PATCH`
- FEATURE：每六个月发布一次的功能版本
- INTERIM：中间版本（预留）
- UPDATE：兼容性更新版本
- PATCH：紧急修复版本

例如：Java 10.0.1表示第10个功能版本的第1个更新版本。

#### 统一的垃圾收集器接口
Java 10引入了一个统一的垃圾收集器接口，使得实现新的垃圾收集器更加容易，并简化了现有收集器的代码。

#### 线程局部握手
Java 10引入了线程局部握手机制，允许JVM在不停止全部线程的情况下，停止单个线程，提高了GC等操作的效率。

#### 移除了JavaEE和CORBA模块
Java 10移除了Java EE和CORBA模块，这些模块在Java 9中已被标记为废弃：
- javax.activation
- javax.xml.bind
- javax.xml.ws
- javax.xml.ws.annotation
- javax.jws
- javax.jws.soap
- javax.transaction
- javax.xml.soap
- org.omg.CORBA
- 等等

#### 额外的Unicode语言标签扩展
Java 10增强了`java.util.Locale`类和相关的API，支持更多的BCP 47语言标签。

#### 根证书
Java 10在JDK中添加了一组默认的根证书，提高了开箱即用的安全性.
## Java 11 核心特性解析

Java 11于2018年9月发布，是继Java 8之后的第二个长期支持(LTS)版本，提供了多项重要的新特性和改进。

### HTTP客户端标准化

#### 特性概述
Java 11将Java 9中引入的HTTP客户端API从孵化模块升级为标准模块，提供了现代化的HTTP客户端，支持HTTP/1.1和HTTP/2协议。

#### 技术细节
新的HTTP客户端API位于`java.net.http`包中，主要包含以下类：
- `HttpClient`：客户端主类，用于发送请求和接收响应
- `HttpRequest`：表示HTTP请求
- `HttpResponse`：表示HTTP响应
- `WebSocket`：支持WebSocket通信

```java
// 创建HTTP客户端
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)
    .build();

// 构建请求
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .timeout(Duration.ofSeconds(30))
    .header("Content-Type", "application/json")
    .GET()
    .build();

// 同步发送请求
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println("状态码: " + response.statusCode());
System.out.println("响应体: " + response.body());

// 异步发送请求
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join(); // 等待完成

// POST请求示例
HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .timeout(Duration.ofSeconds(30))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"张三\",\"age\":30}"))
    .build();
```

#### 应用场景
1. **RESTful API调用**
```java
// 调用RESTful API
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/products"))
    .header("Accept", "application/json")
    .build();

client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenApply(body -> {
          // 解析JSON响应
          return parseJson(body);
      })
      .thenAccept(products -> {
          // 处理产品数据
          processProducts(products);
      });
```

2. **并行API请求**
```java
// 并行发送多个请求
List<URI> uris = List.of(
    URI.create("https://api.example.com/users"),
    URI.create("https://api.example.com/products"),
    URI.create("https://api.example.com/orders")
);

HttpClient client = HttpClient.newHttpClient();

List<CompletableFuture<String>> futures = uris.stream()
    .map(uri -> HttpRequest.newBuilder(uri).build())
    .map(request -> client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                         .thenApply(HttpResponse::body))
    .collect(Collectors.toList());

CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
                .thenAccept(v -> {
                    // 所有请求完成后处理结果
                    List<String> results = futures.stream()
                        .map(CompletableFuture::join)
                        .collect(Collectors.toList());
                    processResults(results);
                });
```

3. **文件上传**
```java
// 上传文件
Path path = Path.of("large-file.zip");
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/upload"))
    .header("Content-Type", "application/octet-stream")
    .POST(HttpRequest.BodyPublishers.ofFile(path))
    .build();

HttpClient client = HttpClient.newHttpClient();
client.send(request, HttpResponse.BodyHandlers.ofString());
```

### String新方法

#### 特性概述
Java 11为`String`类添加了多个实用的新方法，简化了字符串处理。

#### 技术细节
新增的String方法包括：
- `isBlank()`：判断字符串是否为空或仅包含空白字符
- `lines()`：将字符串分割为行，返回Stream
- `strip()`、`stripLeading()`、`stripTrailing()`：去除首尾空白字符（支持Unicode空白）
- `repeat(int)`：重复字符串指定次数

```java
// isBlank() - 检查字符串是否为空或仅包含空白字符
String emptyString = "";
String blankString = "   \t\n";
System.out.println(emptyString.isBlank());  // true
System.out.println(blankString.isBlank());  // true
System.out.println("Hello".isBlank());      // false

// lines() - 将字符串分割为行
String multiline = "Line 1\nLine 2\nLine 3";
multiline.lines()
         .forEach(System.out::println);

// strip() - 去除首尾空白字符
String text = "  Hello World!  ";
System.out.println("'" + text.strip() + "'");       // 'Hello World!'
System.out.println("'" + text.stripLeading() + "'"); // 'Hello World!  '
System.out.println("'" + text.stripTrailing() + "'"); // '  Hello World!'

// repeat() - 重复字符串
String star = "*";
System.out.println(star.repeat(5));  // *****
System.out.println("abc".repeat(3)); // abcabcabc
```

#### 应用场景
1. **表单输入验证**
```java
// 验证用户输入不为空
public boolean isValidInput(String input) {
    return input != null && !input.isBlank();
}
```

2. **文本处理**
```java
// 处理多行文本
String log = getLogContent();
log.lines()
   .filter(line -> line.contains("ERROR"))
   .map(line -> line.strip())
   .forEach(errorLine -> processError(errorLine));
```

3. **格式化输出**
```java
// 创建格式化的表格
String header = "| Name | Age | City |";
String separator = "|" + "-".repeat(6) + "|" + "-".repeat(5) + "|" + "-".repeat(6) + "|";
System.out.println(header);
System.out.println(separator);
```

### Lambda参数的局部变量语法

#### 特性概述
Java 11允许在Lambda表达式的参数中使用`var`关键字，与Java 10引入的局部变量类型推断相呼应，同时可以添加注解。

#### 技术细节
在Lambda表达式中使用var的语法：

```java
// 不使用var
(String s1, String s2) -> s1 + s2

// 使用var
(var s1, var s2) -> s1 + s2

// 使用var并添加注解
(@NotNull var s1, @Nullable var s2) -> s1 + s2
```

需要注意的是，如果使用var，必须为所有参数都使用var，不能混用。

#### 应用场景
1. **添加参数注解**
```java
// 添加参数校验注解
list.stream()
    .filter((@NotNull var item) -> item.length() > 5)
    .collect(Collectors.toList());
```

2. **提高代码可读性**
```java
// 使用var可以让注解更加突出
Function<String, String> func = (@Deprecated var x) -> x.toLowerCase();
```

### ZGC: 可扩展低延迟垃圾收集器 (实验性)

#### 特性概述
Java 11引入了ZGC (Z Garbage Collector)，这是一个可扩展的低延迟垃圾收集器，旨在将GC停顿时间控制在10毫秒以内，无论堆大小如何。

#### 技术细节
ZGC的主要特点：
- 停顿时间不超过10毫秒
- 停顿时间不会随着堆大小或活动对象数量增加而增加
- 可处理从几百MB到TB级别的堆
- 支持NUMA感知的内存分配
- 使用标记-整理算法，不会产生内存碎片
- 并发执行所有昂贵的工作

```bash
# 启用ZGC
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -Xmx16g MyApplication
```

#### 应用场景
1. **低延迟交易系统**
```bash
# 为金融交易系统配置ZGC
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -Xms8g -Xmx8g -XX:+AlwaysPreTouch -XX:+DisableExplicitGC TradingSystem
```

2. **大内存服务器应用**
```bash
# 为大内存应用配置ZGC
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -Xms32g -Xmx32g -XX:ZCollectionInterval=120 BigDataApplication
```

### Epsilon: 无操作垃圾收集器 (实验性)

#### 特性概述
Java 11引入了Epsilon GC，这是一个"无操作"垃圾收集器，它分配内存但不回收内存。当堆内存耗尽时，JVM会退出。

#### 技术细节
Epsilon GC适用于短生命周期的应用或测试场景：
- 不执行任何垃圾回收
- 只处理内存分配
- 当堆内存耗尽时，JVM会抛出`OutOfMemoryError`并退出

```bash
# 启用Epsilon GC
java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC -Xms2g -Xmx2g MyApplication
```

#### 应用场景
1. **性能测试和基准测试**
```bash
# 测量GC对性能的影响
java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC -Xmx1g BenchmarkApplication
```

2. **短生命周期应用**
```bash
# 运行短生命周期的命令行工具
java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC -Xmx100m CommandLineTool
```

### 飞行记录器 (Flight Recorder)

#### 特性概述
Java 11开源了之前商业版JDK中的飞行记录器(JFR)功能，它是一个低开销的数据收集框架，用于分析Java应用的运行情况。

#### 技术细节
JFR可以收集各种运行时信息：
- JVM信息：类加载、编译、GC等
- 操作系统信息：CPU、内存、网络等
- 应用信息：线程、锁等
- 自定义事件

```bash
# 启动应用并开启JFR记录
java -XX:StartFlightRecording=duration=120s,filename=recording.jfr MyApplication

# 使用jcmd动态开启JFR
jcmd <pid> JFR.start duration=120s filename=recording.jfr

# 使用Java代码开启JFR
Configuration config = Configuration.getConfiguration("default");
Recording recording = new Recording(config);
recording.start();
// ... 应用运行
recording.stop();
recording.dump(Path.of("recording.jfr"));
```

#### 应用场景
1. **性能问题诊断**
```bash
# 诊断生产环境性能问题
java -XX:StartFlightRecording=name=leak,settings=profile,disk=true,dumponexit=true,duration=0,filename=leak.jfr ProductionApp
```

2. **内存泄漏分析**
```java
// 分析内存泄漏
java -XX:StartFlightRecording=name=leak,settings=profile,disk=true,dumponexit=true,duration=0,filename=leak.jfr MemoryLeakApp
```

### 其他重要特性

#### 标准化嵌套访问控制
Java 11标准化了嵌套类访问私有成员的规则，使得编译器和JVM处理嵌套类的方式更加一致。

#### 动态类文件常量
Java 11增强了类文件格式，支持新的常量池形式`CONSTANT_Dynamic`，简化了创建某些类型的可变类的实现。

#### 低开销堆分析
Java 11引入了一种新的低开销堆分析方法，可以在不暂停应用的情况下获取Java对象的堆信息。

```java
// 使用JVMTI获取堆信息
// 需要通过JVM参数启用
// -XX:+UnlockCommercialFeatures -XX:+FlightRecorder
```

#### 移除Java EE和CORBA模块
Java 11继续移除了更多的Java EE和CORBA模块：
- java.xml.ws (JAX-WS)
- java.xml.bind (JAXB)
- java.activation (JAF)
- java.xml.ws.annotation (Common Annotations)
- java.corba (CORBA)
- java.transaction (JTA)
- java.se.ee (Aggregator module)

#### 废弃Nashorn JavaScript引擎
Java 11废弃了Nashorn JavaScript引擎，计划在未来的版本中移除。

#### 支持TLS 1.3
Java 11支持传输层安全协议(TLS) 1.3版本，提供更好的安全性和性能。

```java
// 启用TLS 1.3
SSLContext context = SSLContext.getInstance("TLSv1.3");
context.init(null, null, null);

SSLServerSocketFactory ssf = context.getServerSocketFactory();
SSLServerSocket serverSocket = (SSLServerSocket) ssf.createServerSocket(9999);
```

## Java 12 核心特性解析

Java 12于2019年3月发布，是一个非LTS版本，引入了一些实验性特性和性能改进。

### Switch表达式（预览）

#### 特性概述
Java 12首次引入了Switch表达式作为预览特性，这是对传统switch语句的增强，使其可以作为表达式使用，并提供了更简洁的语法。

#### 技术细节
Switch表达式的主要改进：
- 可以作为表达式返回值
- 使用`->`箭头语法代替传统的`case:`和`break;`
- 不会出现传统switch语句中的fall-through问题
- 编译器会检查是否覆盖了所有可能的情况

```java
// 传统switch语句
String dayType;
switch (day) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        dayType = "工作日";
        break;
    case SATURDAY:
    case SUNDAY:
        dayType = "周末";
        break;
    default:
        dayType = "未知";
}

// Java 12 switch表达式（使用箭头语法）
String dayType = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "工作日";
    case SATURDAY, SUNDAY -> "周末";
    default -> "未知";
};
```

需要注意的是，在Java 12中，这是一个预览特性，需要使用`--enable-preview`标志启用。

#### 应用场景
1. **枚举类型处理**
```java
// 处理枚举类型
enum Status { ACTIVE, INACTIVE, PENDING, DELETED }

String message = switch (status) {
    case ACTIVE -> "账户已激活";
    case INACTIVE -> "账户未激活";
    case PENDING -> "账户待审核";
    case DELETED -> "账户已删除";
};
```

2. **复杂条件判断**
```java
// 替代复杂的if-else链
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    case THURSDAY, SATURDAY -> 8;
    case WEDNESDAY -> 9;
    default -> throw new IllegalStateException("无效的日期: " + day);
};
```

### Shenandoah GC（实验性）

#### 特性概述
Java 12引入了Shenandoah垃圾收集器作为实验性特性，它是一种低停顿时间的垃圾收集器，旨在减少GC暂停时间。

#### 技术细节
Shenandoah GC的主要特点：
- 与应用程序并发执行，包括并发压缩
- 停顿时间与堆大小无关，无论堆是200MB还是200GB
- 使用Brooks转发指针和读屏障技术
- 适用于需要低延迟的应用程序

```bash
# 启用Shenandoah GC
java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC MyApplication
```

#### 应用场景
1. **交互式应用**
```bash
# 为交互式应用配置Shenandoah
java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCHeuristics=adaptive InteractiveApp
```

2. **大内存服务器应用**
```bash
# 为大内存应用配置Shenandoah
java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCHeuristics=compact -Xms16g -Xmx16g BigDataApp
```

### 微基准测试套件

#### 特性概述
Java 12添加了一套基于JMH（Java Microbenchmark Harness）的微基准测试套件，用于测试和比较JDK的性能。

#### 技术细节
微基准测试套件位于`jdk/test/micro`目录下，可以用来测试JDK各个组件的性能：
- 字符串操作
- 集合框架
- 正则表达式
- 流处理
- 等等

```bash
# 运行微基准测试
cd jdk/test/micro
make run-micro-benchmarks
```

#### 应用场景
1. **性能调优**
```java
// 使用JMH进行基准测试
@Benchmark
public void testStringConcatenation() {
    String result = "Hello" + ", " + "World!";
}
```

2. **比较算法实现**
```java
// 比较不同排序算法
@Benchmark
public void testQuickSort(Blackhole bh) {
    int[] array = generateRandomArray(1000);
    Arrays.sort(array);
    bh.consume(array);
}

@Benchmark
public void testCustomSort(Blackhole bh) {
    int[] array = generateRandomArray(1000);
    customSort(array);
    bh.consume(array);
}
```

### 紧凑的数字格式

#### 特性概述
Java 12增强了`NumberFormat`类，添加了紧凑的数字格式化支持，可以将大数字以更易读的形式显示。

#### 技术细节
紧凑数字格式有两种样式：
- 短样式：如"1K"、"1M"、"1B"
- 长样式：如"1千"、"1百万"、"1十亿"

```java
// 短样式
NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.SHORT);
System.out.println(fmt.format(1000));       // 1千
System.out.println(fmt.format(1000000));    // 1百万
System.out.println(fmt.format(1000000000)); // 10亿

// 长样式
NumberFormat fmtLong = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.LONG);
System.out.println(fmtLong.format(1000));       // 1千
System.out.println(fmtLong.format(1000000));    // 1百万
System.out.println(fmtLong.format(1000000000)); // 10亿

// 自定义格式
NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
fmt.setMaximumFractionDigits(1);
System.out.println(fmt.format(1234));    // 1.2K
System.out.println(fmt.format(1234567)); // 1.2M
```

#### 应用场景
1. **用户界面显示**
```java
// 显示社交媒体数据
long viewCount = 1234567;
NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.SHORT);
System.out.println("视频播放量: " + fmt.format(viewCount)); // 视频播放量: 123万
```

2. **数据可视化**
```java
// 图表数据标签
long[] values = {1200, 45000, 1250000, 98000000};
NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.SHORT);
for (long value : values) {
    System.out.println(fmt.format(value));
}
// 输出: 1.2千, 4.5万, 125万, 9800万
```

### 文件比较API

#### 特性概述
Java 12引入了`Files.mismatch()`方法，用于比较两个文件的内容，并返回第一个不匹配的位置。

#### 技术细节
`mismatch()`方法返回两个文件内容第一个不匹配字节的位置，如果文件内容相同则返回-1。

```java
// 比较两个文件
Path path1 = Path.of("file1.txt");
Path path2 = Path.of("file2.txt");

long mismatchPosition = Files.mismatch(path1, path2);
if (mismatchPosition == -1) {
    System.out.println("文件内容完全相同");
} else {
    System.out.println("文件在位置 " + mismatchPosition + " 处开始不同");
}
```

#### 应用场景
1. **文件完整性检查**
```java
// 检查文件是否被修改
Path original = Path.of("original.dat");
Path backup = Path.of("backup.dat");

if (Files.mismatch(original, backup) != -1) {
    System.out.println("警告：文件已被修改！");
    backupFile(original);
}
```

2. **二进制文件比较**
```java
// 比较二进制文件
Path binary1 = Path.of("program1.bin");
Path binary2 = Path.of("program2.bin");

long diff = Files.mismatch(binary1, binary2);
if (diff != -1) {
    byte[] content1 = Files.readAllBytes(binary1);
    byte[] content2 = Files.readAllBytes(binary2);
    System.out.printf("位置 %d: %02X vs %02X%n", diff, content1[(int)diff], content2[(int)diff]);
}
```

### 其他重要特性

#### 默认CDS归档文件
Java 12在安装过程中生成默认的类数据共享(CDS)归档文件，提高了应用启动性能。

#### 改进G1垃圾收集器
Java 12对G1垃圾收集器进行了多项改进：
- 可中止的混合收集集合
- 立即返回未使用的已提交内存

```bash
# 使用改进的G1收集器
java -XX:+UseG1GC -XX:G1HeapRegionSize=16m MyApplication
```

#### 改进Aarch64实现
Java 12改进了对ARM 64位架构(Aarch64)的支持，包括：
- 实现了Aarch64上的默认CDS
- 改进了Aarch64上的字符串和数组内在函数

#### JVM常量API
Java 12增强了JVM常量API，为Java语言和JVM上的其他语言提供了更丰富的符号引用信息。

```java
// 使用常量API
MethodHandles.Lookup lookup = MethodHandles.lookup();
VarHandle vh = lookup.findVarHandle(MyClass.class, "myField", int.class);
```

#### 移除了的功能
Java 12移除了以下功能：
- 移除了`com.sun.awt.SecurityWarning`类
- 移除了`javac`的`-source`和`-target`选项对6和1.6的支持
- 移除了`GTE CyberTrust Global Root`证书
## Java 13 核心特性解析

Java 13于2019年9月发布，是一个非LTS版本，继续改进了一些预览特性并引入了新的功能。

### 文本块（预览）

#### 特性概述
Java 13引入了文本块作为预览特性，使得多行字符串字面量的表示更加简洁和直观，解决了传统字符串表示多行文本时的可读性问题。

#### 技术细节
文本块使用三个双引号(`"""`)作为定界符，可以包含多行文本，保留换行和缩进，无需转义大多数特殊字符。

```java
// 传统多行字符串
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, World!</p>\n" +
              "    </body>\n" +
              "</html>";

// 使用文本块
String html = """
              <html>
                  <body>
                      <p>Hello, World!</p>
                  </body>
              </html>
              """;
```

文本块还提供了两个新的转义序列：
- `\s`：表示一个空格，用于保留行尾空格
- `\<line-terminator>`：用于抑制换行符

```java
// 使用\s保留行尾空格
String text = """
              行尾有三个空格   \s
              下一行
              """;

// 使用\抑制换行
String text = """
              这是一行很长的文本，\
              但在代码中分成两行，\
              实际输出是一行
              """;
```

需要注意的是，在Java 13中，这是一个预览特性，需要使用`--enable-preview`标志启用。

#### 应用场景
1. **HTML或XML模板**
```java
// HTML模板
String htmlTemplate = """
                      <!DOCTYPE html>
                      <html>
                        <head>
                          <title>%s</title>
                        </head>
                        <body>
                          <h1>%s</h1>
                          <p>%s</p>
                        </body>
                      </html>
                      """;
String html = String.format(htmlTemplate, "标题", "欢迎", "这是一个使用文本块的示例。");
```

2. **SQL查询**
```java
// SQL查询
String query = """
               SELECT u.id, u.name, u.email, a.city, a.street
               FROM users u
               JOIN addresses a ON u.id = a.user_id
               WHERE u.status = 'ACTIVE'
               ORDER BY u.name
               LIMIT 10
               """;
```

3. **JSON数据**
```java
// JSON数据
String json = """
              {
                "name": "张三",
                "age": 30,
                "address": {
                  "city": "北京",
                  "street": "朝阳区"
                },
                "phoneNumbers": [
                  "13800138000",
                  "13900139000"
                ]
              }
              """;
```

### Switch表达式更新（第二轮预览）

#### 特性概述
Java 13对Java 12中引入的Switch表达式进行了更新，添加了`yield`关键字用于返回值，替代了原来的`break`关键字返回值的语法。

#### 技术细节
Java 13中的Switch表达式有两种形式：
- 使用箭头语法(`->`)的简洁形式
- 使用传统语法和`yield`关键字的详细形式

```java
// 箭头语法（简洁形式）
String dayType = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "工作日";
    case SATURDAY, SUNDAY -> "周末";
    default -> "未知";
};

// 传统语法与yield（详细形式）
String dayType = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY: 
        yield "工作日";
    case SATURDAY, SUNDAY: 
        yield "周末";
    default: 
        yield "未知";
};
```

`yield`关键字用于从Switch表达式返回值，类似于方法中的`return`。

需要注意的是，在Java 13中，这仍然是一个预览特性，需要使用`--enable-preview`标志启用。

#### 应用场景
1. **复杂逻辑处理**
```java
// 使用yield处理复杂逻辑
int result = switch (status) {
    case PENDING: 
        System.out.println("处理中...");
        yield 0;
    case APPROVED: 
        if (isVIP) {
            System.out.println("VIP已批准");
            yield 100;
        } else {
            System.out.println("普通用户已批准");
            yield 50;
        }
    case REJECTED: 
        logRejection(reason);
        yield -1;
    default: 
        throw new IllegalStateException("未知状态");
};
```

2. **根据条件返回不同类型的对象**
```java
// 返回不同类型的对象
Response response = switch (requestType) {
    case GET: 
        var data = fetchData(id);
        yield new SuccessResponse(data);
    case POST: 
        var result = saveData(payload);
        yield new CreatedResponse(result.getId());
    case DELETE: 
        deleteData(id);
        yield new NoContentResponse();
    default: 
        yield new MethodNotAllowedResponse();
};
```

### 重新实现Socket API

#### 特性概述
Java 13对传统的Socket API进行了重新实现，使用了更简单、更现代的实现方式，同时保持了与旧API的兼容性。

#### 技术细节
新的Socket实现：
- 替换了遗留的`java.net.Socket`和`java.net.ServerSocket`实现
- 使用`NioSocketImpl`作为底层实现，替代了`PlainSocketImpl`
- 更容易维护和调试
- 与旧API完全兼容，对用户代码没有影响

```java
// Socket API的使用方式保持不变
ServerSocket serverSocket = new ServerSocket(8080);
Socket clientSocket = serverSocket.accept();

InputStream in = clientSocket.getInputStream();
OutputStream out = clientSocket.getOutputStream();

// 读写数据
byte[] buffer = new byte[1024];
int bytesRead = in.read(buffer);
out.write(response);
```

如果遇到问题，可以通过系统属性回退到旧实现：
```
java -Djdk.net.usePlainSocketImpl=true MyApplication
```

#### 应用场景
1. **网络服务器**
```java
// 简单的回显服务器
try (ServerSocket serverSocket = new ServerSocket(8080)) {
    System.out.println("服务器启动，监听端口 8080");
    
    while (true) {
        try (Socket clientSocket = serverSocket.accept()) {
            System.out.println("客户端连接：" + clientSocket.getInetAddress());
            
            InputStream in = clientSocket.getInputStream();
            OutputStream out = clientSocket.getOutputStream();
            
            byte[] buffer = new byte[1024];
            int bytesRead;
            
            while ((bytesRead = in.read(buffer)) != -1) {
                out.write(buffer, 0, bytesRead);
            }
        }
    }
}
```

2. **客户端应用**
```java
// 简单的HTTP客户端
try (Socket socket = new Socket("api.example.com", 80)) {
    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
    BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    
    // 发送HTTP请求
    out.println("GET / HTTP/1.1");
    out.println("Host: api.example.com");
    out.println("Connection: close");
    out.println();
    
    // 读取响应
    String line;
    while ((line = in.readLine()) != null) {
        System.out.println(line);
    }
}
```

### ZGC：取消提交未使用内存

#### 特性概述
Java 13增强了Z垃圾收集器(ZGC)，添加了将未使用的堆内存返回给操作系统的功能，提高了内存使用效率。

#### 技术细节
ZGC现在可以将长时间未使用的堆内存返回给操作系统，这对于以下场景特别有用：
- 容器化环境中的应用
- 内存使用波动较大的应用
- 与其他应用共享资源的环境

可以通过以下参数控制此功能：
- `-XX:ZUncommitDelay=<seconds>`：控制内存空闲多长时间后取消提交（默认300秒）
- `-XX:-ZUncommit`：完全禁用此功能

```bash
# 启用ZGC并配置取消提交延迟为60秒
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:ZUncommitDelay=60 MyApplication
```

#### 应用场景
1. **容器化微服务**
```bash
# 为Docker容器中的微服务配置ZGC
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:ZUncommitDelay=30 -Xms256m -Xmx1g MicroserviceApplication
```

2. **批处理应用**
```bash
# 为批处理应用配置ZGC
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:ZUncommitDelay=120 -Xms1g -Xmx8g BatchProcessingApp
```