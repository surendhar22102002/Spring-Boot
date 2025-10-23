# 🌱 **Dependency Injection (IoC Container)**

---

## 🧩 1. **Inversion of Control (IoC) Concept**

### 🧠 What is IoC?
Inversion of Control (IoC) means — instead of your **code creating and managing dependencies**, **Spring Framework takes control** of creating, managing, and injecting objects (beans) wherever needed.

> 👥 Normally, *you create objects*.  
> In IoC, *Spring creates and injects objects for you.*

### ⚙️ Example (Without IoC)
```java
public class UserService {
    private UserRepository userRepository = new UserRepository(); // ❌ Tight coupling
}
```

### ✅ With IoC (Using Spring)
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // ✅ Spring injects dependency
}
```

### 💡 Key Point:
IoC = **"Don't call us, we'll call you."**  
Spring controls object creation, lifecycle, and injection.

---

## 🦯 2. **Dependency Injection (DI)**

### 💬 What is DI?
Dependency Injection is a **design pattern** where dependencies (objects that a class needs) are **provided from outside**, not created inside.

### 🔧 Types of DI:
1. **Constructor Injection**
2. **Setter Injection**
3. **Field Injection**

---

## 🧱 3. **Beans and Bean Lifecycle**

### ☕ What is a Bean?
A **Bean** = Object managed by the **Spring IoC Container**.

Beans are created, initialized, used, and destroyed by the container.

### 🔄 Bean Lifecycle Steps:
1. **Instantiation** → Bean object created  
2. **Dependency Injection** → Dependencies injected  
3. **Initialization** → Custom init methods called (`@PostConstruct` or `initMethod`)  
4. **Ready to Use** → Bean is available for use  
5. **Destruction** → Before container shuts down (`@PreDestroy` or `destroyMethod`)

### 🌿 Example:
```java
@Component
public class DemoBean {
    @PostConstruct
    public void init() {
        System.out.println("✅ Bean Initialized");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("💠 Bean Destroyed");
    }
}
```

---

## 🥉 4. **Stereotype Annotations**

These annotations mark classes as **Spring-managed beans** and define their roles.

| Annotation | Description | Typical Layer |
|-------------|--------------|----------------|
| `@Component` | Generic bean (base stereotype) | Utility / Helper classes |
| `@Service` | Business logic component | Service Layer |
| `@Repository` | DAO component (handles DB) | Persistence Layer |
| `@Controller` | Handles web requests | Web Layer |

### 💡 Example:
```java
@Repository
public class UserRepository {}

@Service
public class UserService {}

@Controller
public class UserController {}
```

> 🧠 **Note:** All these are detected during **Component Scanning** (`@ComponentScan`).

---

## �� 5. **@Autowired, @Qualifier, @Primary**

### ⚙️ `@Autowired`
Automatically injects a dependency into a class field, constructor, or setter.

Example (Field Injection):
```java
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

### ⚙️ `@Qualifier`
Used when there are **multiple beans of same type** — tells Spring which one to inject.

Example:
```java
@Service("paypalPayment")
public class PaypalPaymentService implements PaymentService {}

@Service("creditCardPayment")
public class CreditCardPaymentService implements PaymentService {}

@Autowired
@Qualifier("paypalPayment")
private PaymentService paymentService;  // 👉 Chooses PayPal bean
```

### ⚙️ `@Primary`
Marks one bean as default if multiple candidates exist.

```java
@Primary
@Service
public class PaypalPaymentService implements PaymentService {}
```

If no `@Qualifier` is given, Spring uses the `@Primary` one.

---

## 💇‍♂️ 6. **Constructor vs Setter Injection**

| Type | Description | Advantages | When to Use |
|------|--------------|-------------|--------------|
| **Constructor Injection** | Dependencies provided via constructor | Immutable, easy for testing, recommended | Most common |
| **Setter Injection** | Dependencies provided via setter methods | Optional dependencies, reconfigurable | When dependencies are optional |

### 💡 Example

**Constructor Injection**
```java
@Service
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {  // 👉 injected here
        this.userRepository = userRepository;
    }
}
```

**Setter Injection**
```java
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

> 🧠 **Best Practice:** Prefer **Constructor Injection** for required dependencies.

---

## 💤 7. **Lazy Initialization**

By default, Spring creates all singleton beans **eagerly** at startup.

If a bean is not needed immediately, you can make it **lazy**, so it’s created **only when first used**.

### ⚙️ Example
```java
@Component
@Lazy
public class HeavyService {
    public HeavyService() {
        System.out.println("🚀 HeavyService Bean Created!");
    }
}
```

Spring will create this bean **only when** it’s first requested.

### 💡 Global Lazy Init:
```java
@SpringBootApplication
@Lazy
public class MyApplication {}
```

---

## ♾️ 8. **Prototype vs Singleton Scope**

### 💬 What is Bean Scope?
Scope defines **how many instances** of a bean Spring creates and **how long they live**.

| Scope | Description |
|--------|-------------|
| **Singleton (default)** | Only one instance per Spring Container |
| **Prototype** | New instance every time bean is requested |
| **Request** | One instance per HTTP request (web only) |
| **Session** | One per HTTP session (web only) |

### ⚙️ Example:
```java
@Component
@Scope("singleton")
public class SingletonBean {}

@Component
@Scope("prototype")
public class PrototypeBean {}
```

### 🧠 Key Notes:
- Singleton beans are created once and shared.
- Prototype beans are created **on demand**.
- Singleton can depend on Prototype via `ObjectProvider`.

---

## ⚙️ 9. **@Configuration and @Bean**

### 🤩 `@Configuration`
Marks a class that defines bean methods manually.

### 🤩 `@Bean`
Used inside a `@Configuration` class to define beans programmatically.

### 🥱 Example:
```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com");
    }
}
```

This bean is now managed by the IoC container and can be `@Autowired` anywhere.

### 💡 Why use @Bean?
Useful when:
- You want **third-party classes** (not annotated) as beans.
- You need **custom initialization logic**.

---

## 🔄 Summary Table

| Concept | Description | Example |
|----------|--------------|----------|
| IoC | Spring controls object creation | @Autowired beans |
| DI | Injecting dependencies | Constructor Injection |
| Bean | Object managed by Spring | @Component |
| @Component | Generic Spring Bean | @Component class |
| @Service | Business logic class | Service Layer |
| @Repository | DB access class | Repository Layer |
| @Controller | Web controller | MVC Controller |
| @Autowired | Inject dependency | Field/Constructor |
| @Qualifier | Choose specific bean | @Qualifier("beanName") |
| @Primary | Default bean | @Primary on a class |
| Lazy Init | Bean created on demand | @Lazy annotation |
| Scope | Defines bean lifetime | singleton/prototype |
| @Configuration + @Bean | Manual bean creation | External class beans |

---

## 💬 Interview Tips 💡

✅ **Q:** What is IoC Container?  
**A:** It's a Spring mechanism that creates, manages, and injects beans.

✅ **Q:** Difference between `@Component`, `@Service`, `@Repository`?  
**A:** All are stereotypes, but semantically represent different layers.

✅ **Q:** What’s the default bean scope?  
**A:** Singleton.

✅ **Q:** Constructor vs Setter injection — which is preferred?  
**A:** Constructor Injection — better for immutability & testing.

✅ **Q:** What’s `@Lazy`?  
**A:** Creates bean only when needed, not at startup.

✅ **Q:** When do we use `@Configuration` and `@Bean`?  
**A:** When defining third-party or complex beans programmatically.

---

