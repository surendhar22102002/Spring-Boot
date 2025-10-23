# 🌱 Spring Boot Configuration & Profiles – Deep Guide

Spring Boot provides a **flexible and powerful configuration system** to manage application settings, secrets, and environment-specific behavior.

---

## 1️⃣ Configuration Files: `application.properties` vs `application.yml`

### a) `application.properties`
- Key-value pairs  
- Example:

```properties
# Server configuration
server.port=8080
spring.application.name=MyApp

# Database configuration
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=pass123
```

### b) `application.yml` (YAML)
- Supports hierarchical structures
- Example:

```yaml
server:
  port: 8080
spring:
  application:
    name: MyApp
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: pass123
```

### ⚖️ Comparison

| Feature | application.properties | application.yml |
|---|---|---|
| Hierarchical structure | ❌ Harder | ✅ Easy |
| Multiple profiles | Limited | ✅ Excellent |
| Readability | Moderate | ✅ Good |
| Support for lists/maps | ❌ Complicated | ✅ Easy |

---

## 2️⃣ Profiles: `application-dev.yml`, `application-prod.yml`

### a) Active Profile
```properties
spring.profiles.active=dev
```
```yaml
spring:
  profiles:
    active: dev
```
- Command-line override:
```bash
java -jar myapp.jar --spring.profiles.active=prod
```

### b) Profile-specific files
```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/devdb
    username: devuser
    password: devpass
```
```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-db-server:3306/proddb
    username: produser
    password: prodpass
```

### c) Profile-specific beans
```java
@Configuration
@Profile("dev")
public class DevDatabaseConfig { }

@Configuration
@Profile("prod")
public class ProdDatabaseConfig { }
```

---

## 3️⃣ Accessing Config Values

### a) `@Value`
```java
@Value("${spring.datasource.url}")
private String datasourceUrl;
```

### b) `@ConfigurationProperties`
```java
@Component
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {
    private String url;
    private String username;
    private String password;
    // getters & setters
}
```

### c) `Environment` object
```java
@Autowired
private Environment env;
System.out.println(env.getProperty("spring.datasource.url"));
```

---

## 4️⃣ Environment Variables
```properties
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```
- Command-line:
```bash
DB_USER=root DB_PASS=secret java -jar myapp.jar
```

---

## 5️⃣ Externalized Configurations
- Command-line > Environment vars > profile-specific yml > default application.yml
- Custom config file:
```java
@Configuration
@PropertySource("classpath:custom-config.properties")
public class CustomConfig { }
```

---

## 6️⃣ Secrets Management

### a) Spring Cloud Vault
```yaml
spring:
  cloud:
    vault:
      uri: http://localhost:8200
      token: s.xxxxxx
      kv:
        enabled: true
```
```java
@Value("${secret.datasource.password}")
private String dbPassword;
```

### b) AWS Secrets Manager
- Spring Cloud AWS integration
- Load secrets dynamically via `@Value` or `@ConfigurationProperties`

---

## 7️⃣ Advanced / Missing Concepts

1. Relaxed Binding – `my.property-name` == `MY_PROPERTY_NAME` == `myPropertyName` ✅  
2. Combine profiles: `spring.profiles: "dev,debug"`  
3. Encrypted properties with Spring Cloud Config or Vault 🔒  
4. Dynamic reload: `@RefreshScope`  
5. Lists / Maps in YAML:
```yaml
my:
  servers:
    - host: localhost
      port: 8080
    - host: 10.0.0.1
      port: 9090
```
6. Programmatic profile activation:
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApp.class);
        app.setAdditionalProfiles("dev");
        app.run(args);
    }
}
```

---

## 8️⃣ Interview Tips 💡

- `@Value` vs `@ConfigurationProperties` – know when to use
- Profile precedence: command-line > env vars > profile-specific > application.yml
- Secrets best practices – Vault/AWS, never commit passwords
- Externalized config – 12-factor apps
- Dynamic config refresh – `@RefreshScope`

---

## ✅ Summary Table

| Concept | Usage / Notes |
|---|---|
| application.properties | Simple key-value, small apps |
| application.yml | Hierarchical, readable, supports profiles |
| @Value | Single property injection, simple |
| @ConfigurationProperties | Grouped, type-safe, scalable |
| Environment variables | OS-level overrides, 12-factor apps |
| Profiles | Dev/prod/test configs, conditional beans |
| Secrets Management | Vault, AWS Secrets, encrypted configs |
| Externalized Config | Command-line, env, config files, custom paths |

