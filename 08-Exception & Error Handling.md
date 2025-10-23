# 🌟 Spring Boot Exception & Error Handling 

Spring Boot provides a robust way to handle exceptions and errors in a standardized way so that your API responds consistently and developers can debug easily. This guide covers all topics including missed concepts, with examples and emojis.

---

## 1️⃣ Basics: Exception vs Error

| Type        | Description |
|------------|-------------|
| **Exception** ✅ | Represents recoverable issues. Example: `FileNotFoundException`, `IllegalArgumentException`. |
| **Error** ❌ | Represents serious issues that your app usually cannot handle. Example: `OutOfMemoryError`, `StackOverflowError`. |

> In Spring Boot, we mainly handle **Exceptions**, not Errors.

---

## 2️⃣ Handling Exceptions in Spring Boot

### a) Using try-catch 🟡
```java
@GetMapping("/divide")
public int divide(@RequestParam int a, @RequestParam int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Cannot divide by zero");
    }
}
```

### b) Using `@ExceptionHandler` (Controller-level) 🔹
```java
@RestController
@RequestMapping("/api")
public class MyController {

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable int id) {
        if (id <= 0) throw new UserNotFoundException("Invalid user ID");
        return new User(id, "D");
    }

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.NOT_FOUND.value(),
                ex.getMessage(),
                "User not found in database"
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

### c) Global Exception Handling using `@ControllerAdvice` 🌍
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.NOT_FOUND.value(),
                ex.getMessage(),
                "User not found in database"
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                ex.getMessage(),
                "Internal server error"
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

## 3️⃣ Creating Custom Exception Classes 🛠️
```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

## 4️⃣ Error Response Models 📦
```java
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String message;
    private String details;
    // Constructor, getters, setters
}
```

---

## 5️⃣ Handling Validation Errors 📝
```java
@PostMapping("/user")
public User createUser(@Valid @RequestBody User user) {
    return userService.save(user);
}

@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {
    List<String> errors = ex.getBindingResult()
                            .getFieldErrors()
                            .stream()
                            .map(err -> err.getField() + ": " + err.getDefaultMessage())
                            .toList();
    ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            errors.toString()
    );
    return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
}
```

---

## 6️⃣ Using `ResponseStatusException` ⚡
```java
@GetMapping("/product/{id}")
public Product getProduct(@PathVariable int id) {
    Product p = productService.findById(id);
    if (p == null) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Product not found");
    }
    return p;
}
```

---

## 7️⃣ Using `@ResponseStatus` 🎯
```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "User Not Found")
public class UserNotFoundException extends RuntimeException { }
```

---

## 8️⃣ ErrorController for Custom Error Pages / API Errors 🖥️
```java
@RestController
public class CustomErrorController implements ErrorController {

    @RequestMapping("/error")
    public ErrorResponse handleError(HttpServletRequest request) {
        Object status = request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ErrorResponse(
                LocalDateTime.now(),
                status != null ? Integer.parseInt(status.toString()) : 500,
                "Something went wrong",
                request.getRequestURI()
        );
    }
}
```

---

## 9️⃣ Common Best Practices ✅
1. Global exception handling with `@ControllerAdvice`.
2. Custom exception classes for meaningful error reporting.
3. Structured error response (timestamp, status, message, details).
4. Handle validation errors explicitly.
5. Use `ResponseStatusException` for quick inline errors.
6. Avoid catching `Error` classes; only catch `Exception` or custom exceptions.
7. Log all exceptions properly for debugging and monitoring. 📝

---

## 🔟 Advanced / Missed Topics 🔥
- Nested exceptions: `ex.getCause()`
- ConstraintViolationException for query params / path variables
- Logging exceptions: SLF4J / Logback
- Internationalization (i18n) of error messages
- Spring Boot actuator: `/actuator/health`
- Custom API error codes (useful in microservices)

---

## 1️⃣1️⃣ Full Example: Global Exception Handling 💎
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(LocalDateTime.now(),
                HttpStatus.NOT_FOUND.value(), ex.getMessage(), "User API error");
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
                                .stream().map(err -> err.getField() + ": " + err.getDefaultMessage())
                                .toList();
        ErrorResponse error = new ErrorResponse(LocalDateTime.now(), HttpStatus.BAD_REQUEST.value(),
                                                "Validation failed", errors.toString());
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobal(Exception ex) {
        ErrorResponse error = new ErrorResponse(LocalDateTime.now(), HttpStatus.INTERNAL_SERVER_ERROR.value(),
                                                ex.getMessage(), "Internal server error");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

