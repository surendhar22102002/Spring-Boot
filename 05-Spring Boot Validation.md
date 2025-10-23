# 🌟 Spring Boot Validation 

## 🧩 What is Validation?
Validation ensures the data coming into your application (via API request, form input, etc.) meets the required rules before processing it. Spring Boot integrates **Bean Validation (JSR-380)** using **Hibernate Validator** under the hood.

---

## ⚙️ 1️⃣ Enabling Validation

To enable validation, add this dependency in your `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## 🧠 2️⃣ `@Valid` vs `@Validated`

| Feature | `@Valid` | `@Validated` |
|----------|-----------|--------------|
| Provided by | JSR-303 (Jakarta Validation API) | Spring Framework |
| Used on | Method parameters or beans | Method parameters, classes |
| Groups supported | ❌ No | ✅ Yes |
| Common use | Validate request bodies (DTOs) | Validate with custom groups or services |

### 💡 Example:

```java
@PostMapping("/register")
public ResponseEntity<String> registerUser(@Valid @RequestBody UserDTO user) {
    return ResponseEntity.ok("User registered successfully!");
}
```

If using `@Validated` for custom validation groups:
```java
@PostMapping("/save")
public ResponseEntity<String> saveData(@Validated(OnCreate.class) @RequestBody ProductDTO dto) {
    return ResponseEntity.ok("Saved");
}
```

---

## 🧾 3️⃣ Common Validation Annotations ✨

| Annotation | Description | Example |
|-------------|--------------|----------|
| `@NotNull` | Field must not be null | `@NotNull(message="Name can't be null")` |
| `@NotEmpty` | Field must not be null **and** not empty | `@NotEmpty(message="List can't be empty")` |
| `@NotBlank` | Field must not be null **and** not blank (for Strings) | `@NotBlank(message="Username required")` |
| `@Email` | Must be a valid email format | `@Email(message="Invalid email")` |
| `@Size(min, max)` | Field length constraint | `@Size(min=3, max=10)` |
| `@Min` / `@Max` | Numeric constraints | `@Min(18) @Max(60)` |
| `@Positive` / `@Negative` | Must be >0 or <0 | `@Positive(message="Age must be positive")` |
| `@Past` / `@Future` | For date/time validation | `@Past(message="DOB must be in past")` |
| `@Pattern(regexp=...)` | Regex matching | `@Pattern(regexp="^[A-Z].*$")` |
| `@AssertTrue` / `@AssertFalse` | Must evaluate to true/false | `@AssertTrue(message="Must accept terms")` |

---

## 🧰 4️⃣ Example DTO using Validations 💻

```java
public class UserDTO {

    @NotBlank(message = "Name cannot be blank")
    private String name;

    @Email(message = "Enter valid email")
    private String email;

    @Size(min = 6, message = "Password must be at least 6 chars")
    private String password;

    @Min(value = 18, message = "Age must be >= 18")
    private int age;

    @AssertTrue(message = "Terms must be accepted")
    private boolean termsAccepted;

    // getters and setters
}
```

---

## ⚡ 5️⃣ BindingResult — Handling Validation Errors Gracefully 🎯

```java
@PostMapping("/register")
public ResponseEntity<?> register(@Valid @RequestBody UserDTO user, BindingResult result) {
    if (result.hasErrors()) {
        List<String> errors = result.getFieldErrors()
                                    .stream()
                                    .map(FieldError::getDefaultMessage)
                                    .toList();
        return ResponseEntity.badRequest().body(errors);
    }

    return ResponseEntity.ok("User registered successfully!");
}
```

✅ **Tip:** `BindingResult` **must immediately follow** the validated parameter!

---

## 💥 6️⃣ Global Exception Handling using `@ControllerAdvice` 🧱

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
                                .getFieldErrors()
                                .stream()
                                .map(FieldError::getDefaultMessage)
                                .toList();

        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

## 🧑‍🔧 7️⃣ Custom Validation Annotation 🧩

### 🪄 Step 1: Create Custom Annotation

```java
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordValidator.class)
public @interface StrongPassword {
    String message() default "Password must contain uppercase, number, and special character";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### 🪄 Step 2: Create Validator Logic

```java
public class PasswordValidator implements ConstraintValidator<StrongPassword, String> {

    private static final String PASSWORD_PATTERN =
        "^(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$";

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return value != null && value.matches(PASSWORD_PATTERN);
    }
}
```

### 🪄 Step 3: Use in DTO

```java
public class RegisterDTO {

    @StrongPassword
    private String password;

    // other fields
}
```

---

## 🎯 8️⃣ Validation in Service Layer using `@Validated`

```java
@Service
@Validated
public class PaymentService {

    public void processPayment(@NotNull @Min(1) Double amount) {
        System.out.println("Processing payment: " + amount);
    }
}
```

If invalid data is passed, Spring will throw: `ConstraintViolationException`

---

## 🧩 9️⃣ Custom Validation Groups

```java
public interface OnCreate {}
public interface OnUpdate {}

public class ProductDTO {

    @NotNull(groups = OnUpdate.class)
    private Long id;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String name;
}
```

Controller:
```java
@PostMapping("/create")
public void create(@Validated(OnCreate.class) @RequestBody ProductDTO dto) { }

@PutMapping("/update")
public void update(@Validated(OnUpdate.class) @RequestBody ProductDTO dto) { }
```

---

## 🧠 10️⃣ Validation for PathVariable & RequestParam 🎯

```java
@RestController
@Validated
public class ProductController {

    @GetMapping("/products/{id}")
    public String getProduct(@PathVariable @Min(1) Long id) {
        return "Product " + id;
    }
}
```

If invalid (e.g., `/products/0`), throws `ConstraintViolationException`.

---

## 🧵 11️⃣ Summary Table 🧭

| Layer | Validation Annotation | Exception |
|--------|-----------------------|------------|
| DTO | `@Valid`, `@NotBlank`, `@Email`, etc. | `MethodArgumentNotValidException` |
| Controller | `@Validated` | `ConstraintViolationException` |
| Service | `@Validated` + Method param constraints | `ConstraintViolationException` |
| Custom | `@Constraint` | Custom exception or default |

---

## 💬 Interview Tips 💡

- 🔹 `@Valid` → Bean-level validation (DTOs)  
- 🔹 `@Validated` → Supports **groups** and **method-level** validation  
- 🔹 `BindingResult` → Collects field errors immediately after DTO  
- 🔹 `MethodArgumentNotValidException` → Triggered automatically if BindingResult is absent  
- 🔹 Always centralize validation logic using `@ControllerAdvice`  
- 🔹 `Hibernate Validator` is default provider for JSR-303

