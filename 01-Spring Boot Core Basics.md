# 🌱 Spring Boot Core Basics 

---

## 🚀 1. What is Spring Framework?

### 🔹 Definition:
Spring Framework is a **powerful, lightweight, and modular Java framework** used to build enterprise-grade applications.  
It provides **dependency injection**, **aspect-oriented programming**, and a vast ecosystem for web, data, and security layers.

### 🔹 Core Features:
- **IoC (Inversion of Control) / Dependency Injection** — reduces tight coupling.
- **AOP (Aspect-Oriented Programming)** — handles cross-cutting concerns (logging, transactions, etc.).
- **Spring MVC** — builds REST APIs and web apps.
- **Spring Data** — simplifies database operations.
- **Spring Security** — provides authentication and authorization.

### 🔹 Problem with plain Spring:
Developers had to:
- Configure XML files manually 😩  
- Define dependencies themselves 🧩  
- Deploy WAR files on an external server 🧱  

🔗 This was **time-consuming** and **error-prone**.

---

## ⚡ 2. What is Spring Boot?

### 🔹 Definition:
Spring Boot is a **Spring Framework extension** that **eliminates boilerplate configuration**.  
It’s built on top of Spring and provides:
- **Auto-configuration**
- **Embedded servers**
- **Opinionated defaults**
- **Production-ready features** (metrics, health checks, etc.)

### 🔹 Key Idea:
> "Spring Boot makes it easy to create stand-alone, production-grade Spring applications with minimal effort."

### 🔹 Example:
Instead of configuring a web.xml file and DispatcherServlet manually,  
you just add `spring-boot-starter-web`, and Spring Boot auto-configures everything 🎯

---

## 💎 3. Advantages of Spring Boot

| 🔥 Feature | 💡 Description |
|-------------|----------------|
| **Auto Configuration** | Automatically configures Spring beans based on dependencies present in the classpath. |
| **Embedded Servers** | No need for external Tomcat or Jetty; apps can run with `java -jar`. |
| **Opinionated Defaults** | Sensible defaults to get started quickly (e.g., H2 for DB, JSON via Jackson). |
| **Starter Dependencies** | Grouped Maven dependencies simplify project setup. |
| **Spring Boot CLI** | Run and test apps quickly using Groovy scripts. |
| **Actuator** | Built-in monitoring endpoints for health, metrics, and logging. |
| **Microservice Ready** | Perfect base for RESTful and microservice architectures. |

---

## 🏠 4. Spring Boot Architecture

Spring Boot architecture has **4 major layers**:

### 🧬 (a) Application Layer
- Contains the **main class** with `@SpringBootApplication`.
- It’s the entry point that triggers auto-configuration and component scanning.

### ⚙️ (b) Auto-Configuration Layer
- Automatically configures beans using conditions like `@ConditionalOnClass`, `@ConditionalOnMissingBean`.

### 📦 (c) Starter Dependencies Layer
- Provides pre-packaged dependencies for features (like web, JPA, security).

### 🌐 (d) Embedded Server Layer
- Runs application directly via embedded Tomcat/Jetty/Undertow — no WAR needed.

### 🔀 Flow:
```
@SpringBootApplication
     ↓
Component Scan → Finds @Component, @Service, @Repository
     ↓
Auto Configuration → Configures beans based on dependencies
     ↓
Embedded Server → Starts Tomcat automatically
     ↓
Application Ready 🚀
```

---

## ⚙️ 5. Auto-Configuration (Deep Dive)

### 🔹 What it does:
Spring Boot’s **auto-configuration** automatically sets up beans based on the libraries found on the classpath.

For example:
- If `spring-boot-starter-web` is on the classpath → auto-configures **Spring MVC** and an **embedded Tomcat**.
- If `spring-boot-starter-data-jpa` is on the classpath → auto-configures **DataSource**, **EntityManager**, etc.

### 🔹 How it works:
- The annotation `@EnableAutoConfiguration` (part of `@SpringBootApplication`) triggers it.
- It loads configurations from:
  ```
  META-INF/spring.factories
  ```
  which lists auto-config classes like:
  ```
  org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
  ```

### 🔹 Conditional Loading:
Auto-config classes use annotations like:
- `@ConditionalOnClass` → loads config only if class exists.
- `@ConditionalOnMissingBean` → avoids overriding user-defined beans.

🛪️ This allows **customization without losing control**.

---

## 📦 6. Starter Dependencies

### 🔹 What are they?
Spring Boot provides **starter POMs** — collections of commonly used dependencies bundled together.

### 🔹 Example:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Automatically includes:
- Spring MVC
- Embedded Tomcat
- Jackson for JSON
- Logging (SLF4J + Logback)

### 🔹 Common Starters:
| Starter | Purpose |
|----------|----------|
| `spring-boot-starter-web` | Web + REST API |
| `spring-boot-starter-data-jpa` | Database (Hibernate + JPA) |
| `spring-boot-starter-security` | Security + Authentication |
| `spring-boot-starter-test` | Unit + Integration testing |
| `spring-boot-starter-actuator` | Monitoring and health checks |

---

## 🧮 7. Spring Boot Initializr

### 🔹 What it is:
An online project generator that helps you quickly bootstrap new Spring Boot projects.

🔗 **Website:** [https://start.spring.io](https://start.spring.io)

### 🔹 You can select:
- Project type (Maven/Gradle)
- Language (Java/Kotlin/Groovy)
- Spring Boot version
- Dependencies (web, JPA, etc.)

💥 It auto-generates the structure and dependencies for you — ready to code.

---

## ⚙️ 8. Application Properties / YAML

### 🔹 Purpose:
Used to **configure application settings** such as:
- Server port
- Database connection
- Logging levels
- Profiles (dev, prod)

### 🔹 Example (application.properties):
```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/testdb
spring.datasource.username=root
spring.datasource.password=admin
logging.level.org.springframework=INFO
```

### 🔹 Example (application.yml):
```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb
    username: root
    password: admin
logging:
  level:
    org.springframework: INFO
```

🧠 YAML is more readable and hierarchical — preferred in modern projects.

---

## 🧠 9. Main Class & `@SpringBootApplication` Annotation

### 🔹 Example:
```java
@SpringBootApplication
public class MyApp {
  public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
  }
}
```

### 🔹 What it does:
`@SpringBootApplication` = Combination of:
1. `@Configuration` → Marks class as a source of bean definitions.  
2. `@EnableAutoConfiguration` → Enables auto-configuration.  
3. `@ComponentScan` → Scans for components in the package.

🔗 It’s the entry point of the Spring Boot app.

---

## ⚙️ 10. How Spring Boot Works Internally (Auto-Config + Embedded Server)

### 🧬 Step-by-Step:

1. **Run Main Class**
   ```bash
   java -jar myapp.jar
   ```
2. **SpringApplication.run()**
   - Creates an `ApplicationContext`.
   - Performs **component scanning**.
3. **Auto-Configuration Kicks In**
   - Scans `META-INF/spring.factories` for config classes.
   - Loads matching configurations (e.g., Tomcat, DataSource).
4. **Embedded Server Initialization**
   - Auto-configures and starts **Tomcat/Jetty/Undertow**.
   - Deploys the Spring context inside this server.
5. **App Ready**
   - Your REST endpoints are now active 🎉
   - Visit `http://localhost:8080`

---

## 🧭 Summary Table

| Concept | Description |
|----------|-------------|
| **Spring Framework** | Core dependency injection + modular structure. |
| **Spring Boot** | Simplifies setup with auto-config + embedded server. |
| **Auto-Configuration** | Automatically configures beans based on classpath. |
| **Starter Dependencies** | Grouped dependencies for quick setup. |
| **Initializr** | GUI to generate project scaffolding. |
| **Properties/YAML** | External configuration of app behavior. |
| **@SpringBootApplication** | Entry point + triggers auto-config + scanning. |
| **Internal Flow** | Run main → Auto-configure → Start embedded server → Serve app. |

