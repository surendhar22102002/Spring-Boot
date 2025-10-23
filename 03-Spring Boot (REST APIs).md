# 🌐 **Spring Boot (REST APIs)**

---

## 🧩 1. `@RestController` and `@RequestMapping`

### 💡 Concept
- `@RestController` → combination of `@Controller + @ResponseBody`
- Used to build **RESTful web services** that return **JSON/XML** instead of JSP/HTML views.
- Handles HTTP requests (GET, POST, PUT, DELETE).

### 🧱 Example
```java
@RestController
@RequestMapping("/api/users")  // Base URL for all user endpoints
public class UserController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello from Spring Boot REST API 👋";
    }
}
```

✅ **Output:**  
`GET http://localhost:8080/api/users/hello` → `"Hello from Spring Boot REST API 👋"`

---

## ⚙️ 2. HTTP Method Mappings

### 📗 `@GetMapping`
Used to **retrieve** data.

```java
@GetMapping("/{id}")
public User getUserById(@PathVariable Long id) {
    return userService.findUserById(id);
}
```

---

### 📘 `@PostMapping`
Used to **create** a new resource.

```java
@PostMapping
public User createUser(@RequestBody User user) {
    return userService.saveUser(user);
}
```

---

### 📙 `@PutMapping`
Used to **update** existing data.

```java
@PutMapping("/{id}")
public User updateUser(@PathVariable Long id, @RequestBody User user) {
    return userService.updateUser(id, user);
}
```

---

### 📕 `@DeleteMapping`
Used to **delete** a resource.

```java
@DeleteMapping("/{id}")
public String deleteUser(@PathVariable Long id) {
    userService.deleteUser(id);
    return "User deleted successfully 🗑️";
}
```

---

## 🧬 3. Request Parameters vs Path Variables

| Type | Annotation | Example URL | Use Case |
|------|-------------|-------------|-----------|
| Path Variable | `@PathVariable` | `/api/users/10` | For specific resource |
| Request Param | `@RequestParam` | `/api/users?id=10` | For filters/search |

### 🧱 Example

### **Path Variable**:

Used to capture dynamic values from URL.

```java
@GetMapping("/users/{id}")
public String getUserById(@PathVariable int id) {
    return "User ID: " + id + " 🧑";
}
```

### **Request Parameter**:

Used to capture query parameters.

```java
@GetMapping("/search")
public String searchUser(@RequestParam String name) {
    return "Searching for: " + name + " 🔍";
}
```

---

## 📦 4. `@RequestBody` and `@ResponseBody`

### 📨 `@RequestBody`
Converts incoming JSON → Java object (using Jackson).
### 📤 `@ResponseBody`
Converts Java object → JSON automatically.

### 🧱 Example
```java
@PostMapping("/register")
public User registerUser(@RequestBody User user) {
    System.out.println("User received: " + user);
    return user; // Spring automatically converts it back to JSON
}
```

📈 No need to manually serialize/deserialize JSON — Spring Boot + Jackson does it for you.

---

## 🧪 5. JSON Serialization & Deserialization (Jackson)

Spring Boot uses **Jackson** under the hood.

### 🔄 Serialization
Java → JSON  
### 🔁 Deserialization
JSON → Java

### 🧱 Example: Custom Field Mapping
```java
public class User {
    @JsonProperty("user_name")
    private String name;

    @JsonIgnore // won't appear in JSON
    private String password;
}
```

🧠 **Notes**
- Use `@JsonIgnoreProperties(ignoreUnknown = true)` to ignore unknown JSON fields.
- Can customize date formats with `@JsonFormat(pattern = "yyyy-MM-dd")`.

---

## 🚦 6. HTTP Status Codes + `ResponseEntity`

### 💡 `ResponseEntity` = Body + Headers + Status Code

### 🧱 Example
```java
// User Controller
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    User user = userService.findUserById(id);
    if (user != null)
        return ResponseEntity.ok(user); // 200 OK
    else
        return ResponseEntity.status(HttpStatus.NOT_FOUND).build(); // 404
}
```

✅ **Common Status Codes**
| Code | Meaning |
|------|----------|
| 200 OK | Success |
| 201 Created | Resource created |
| 400 Bad Request | Invalid input |
| 401 Unauthorized | Authentication failed |
| 403 Forbidden | Access denied |
| 404 Not Found | Resource missing |
| 500 Internal Server Error | Server failure |

---

## 🧪 7. Global Exception Handling (`@ControllerAdvice`, `@ExceptionHandler`)

### 💡 Concept
Handle all exceptions from one central place 💪

### 🧱 Example
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                             .body("❌ " + ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneral(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body("💥 Something went wrong!");
    }
}
```

🧠 **Tip:**  
`@ControllerAdvice` = global level  
`@ExceptionHandler` = method level

---

## 🌍 8. CORS Configuration

### 💡 Why?
CORS (Cross-Origin Resource Sharing) allows front-end apps (e.g., React) from other origins to call backend APIs.

### 🧱 Example 1: Method Level
```java
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/users")
public List<User> getAllUsers() {
    return userService.getAllUsers();
}
```

### 🧱 Example 2: Global Configuration
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET","POST","PUT","DELETE");
    }
}
```

---

## 🟙️ 9. URL Mapping Best Practices

✅ **Do’s**
- Use nouns for resource names → `/users`, `/orders`
- Use plural nouns → `/products`
- Use HTTP verbs properly (GET, POST, PUT, DELETE)
- Use meaningful nesting → `/users/{id}/orders`
- Version your API → `/api/v1/users`

❌ **Don’ts**
- Don’t mix verbs in URL (❌ `/getUserById`)  
- Don’t use unnecessary query params for core resource access.

---

## 🚀 Full Example Controller

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findUserById(id);
        return user != null ? ResponseEntity.ok(user)
                            : ResponseEntity.notFound().build();
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User savedUser = userService.saveUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        User updatedUser = userService.updateUser(id, user);
        return ResponseEntity.ok(updatedUser);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 🧠 Interview Tips
- `@RestController` always returns JSON by default.  
- `ResponseEntity` → best for customizing HTTP status + body.  
- Prefer DTOs (Data Transfer Objects) over direct entity return.  
- Global exception handling improves API cleanliness.  
- Enable CORS carefully — only for trusted domains.  
- Use `@Valid` and validation annotations for request validation.

