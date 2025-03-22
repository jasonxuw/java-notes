# Spring Boot 自定义 Starter 开发教程

在 Spring Boot 生态系统中，Starter 是一种特殊的依赖项，它能够自动配置和启用特定功能，大大简化了应用开发过程。本教程将带您一步步创建自己的自定义 Starter，让您深入理解 Spring Boot 自动配置的核心机制。

## 什么是 Spring Boot Starter？

Starter 是 Spring Boot 的一个核心特性，它包含了一组依赖描述，用来简化应用程序的依赖配置。使用 Starter 可以避免手动管理依赖项的版本冲突问题，同时通过自动配置机制，无需编写大量配置代码就能快速集成某项功能。

## 为什么需要自定义 Starter？

当我们开发了一个通用功能模块，希望在多个项目中复用时，将其封装为 Starter 是一个很好的选择。这样其他项目只需要引入你的 Starter 依赖，就能自动获得该功能，而无需关心内部实现细节。

## 自定义 Starter 的命名规范

- 官方 Starter 命名：`spring-boot-starter-{name}`
- 非官方 Starter 命名：`{name}-spring-boot-starter`

按照惯例，我们的自定义 Starter 应该使用非官方命名方式。

## 案例：创建一个问候语 Starter

下面我们将创建一个简单的问候语 Starter，它能够根据配置生成不同的问候语。

### 第一步：创建父工程

首先创建一个父工程，用于管理我们的 Starter 模块。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>greeting-spring-boot</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>greeting-spring-boot-autoconfigure</module>
        <module>greeting-spring-boot-starter</module>
    </modules>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring-boot.version>2.7.0</spring-boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 第二步：创建自动配置模块

创建 `greeting-spring-boot-autoconfigure` 模块，负责实现自动配置逻辑。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>greeting-spring-boot</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>greeting-spring-boot-autoconfigure</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

### 第三步：创建配置属性类

在自动配置模块中，创建一个用于映射配置属性的类：

```java
package com.example.greeting.autoconfigure;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "greeting")
public class GreetingProperties {
    
    /**
     * 问候语的前缀
     */
    private String prefix = "Hello";
    
    /**
     * 问候语的后缀
     */
    private String suffix = "!";

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

### 第四步：创建核心服务类

接下来，创建一个提供核心功能的服务类：

```java
package com.example.greeting.autoconfigure;

public class GreetingService {
    
    private final String prefix;
    private final String suffix;
    
    public GreetingService(String prefix, String suffix) {
        this.prefix = prefix;
        this.suffix = suffix;
    }
    
    public String greet(String name) {
        return prefix + " " + name + suffix;
    }
}
```

### 第五步：创建自动配置类

创建自动配置类，根据配置属性自动创建并注册服务：

```java
package com.example.greeting.autoconfigure;

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(GreetingProperties.class)
public class GreetingAutoConfiguration {
    
    private final GreetingProperties properties;
    
    public GreetingAutoConfiguration(GreetingProperties properties) {
        this.properties = properties;
    }
    
    @Bean
    @ConditionalOnMissingBean
    public GreetingService greetingService() {
        return new GreetingService(properties.getPrefix(), properties.getSuffix());
    }
}
```

### 第六步：注册自动配置类

在自动配置模块的 `resources` 目录下创建 `META-INF/spring.factories` 文件：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.greeting.autoconfigure.GreetingAutoConfiguration
```

这一步是关键，它告诉 Spring Boot 在启动时自动加载我们的配置类。

### 第七步：创建 Starter 模块

创建 `greeting-spring-boot-starter` 模块，作为最终的依赖包：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>greeting-spring-boot</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>greeting-spring-boot-starter</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>greeting-spring-boot-autoconfigure</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

Starter 模块通常只包含必要的依赖，不包含实际代码。

### 第八步：打包安装

运行 Maven 命令来打包并安装到本地仓库：

```bash
mvn clean install
```

## 使用我们的自定义 Starter

在其他 Spring Boot 项目中使用我们刚刚创建的 Starter：

1. 添加依赖

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>greeting-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

2. 在配置文件中自定义属性（可选）

```yaml
greeting:
  prefix: "你好,"
  suffix: "，欢迎来到Spring Boot世界!"
```

3. 在代码中注入并使用 GreetingService

```java
@RestController
public class GreetingController {
    
    @Autowired
    private GreetingService greetingService;
    
    @GetMapping("/greet/{name}")
    public String greet(@PathVariable String name) {
        return greetingService.greet(name);
    }
}
```

## 条件化配置

Spring Boot 的条件化配置是自动配置的关键机制，以下是一些常用的条件注解：

- `@ConditionalOnClass`: 当类路径下有指定的类时，配置生效
- `@ConditionalOnMissingBean`: 当容器中没有指定的 Bean 时，配置生效
- `@ConditionalOnProperty`: 当配置中有指定的属性时，配置生效
- `@ConditionalOnMissingClass`: 当类路径下没有指定的类时，配置生效
- `@ConditionalOnWebApplication`: 当应用是 Web 应用时，配置生效

示例：

```java
@Configuration
@ConditionalOnClass(DataSource.class)
public class DatabaseAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "database", name = "enabled", havingValue = "true")
    public DataSource dataSource() {
        // 创建数据源...
    }
}
```

## 最佳实践

1. **模块分离**：将自动配置逻辑和 Starter 分离为两个模块
2. **合理默认值**：提供合理的默认配置，让用户"开箱即用"
3. **灵活配置**：提供足够的配置项，让用户可以根据需要自定义行为
4. **条件化配置**：使用条件注解避免不必要的配置加载
5. **良好文档**：为你的 Starter 编写清晰的文档和使用示例

## 总结

自定义 Starter 是复用 Spring Boot 项目功能的绝佳方式。通过本教程，我们学习了如何：

1. 创建配置属性类来映射外部配置
2. 实现核心服务类提供功能
3. 创建自动配置类来注册 Bean
4. 注册自动配置类到 Spring Boot
5. 封装为可复用的 Starter

通过这些步骤，我们不仅能够开发自己的 Starter，还能更深入地理解 Spring Boot 的自动配置机制，对使用第三方 Starter 也会有更清晰的认识。
