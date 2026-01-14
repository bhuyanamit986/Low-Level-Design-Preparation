# 📘 Section 7: Error Handling & Validation

> **Robust error handling separates good code from great code**

---

## 📑 Table of Contents

1. [Exception Types](#1-exception-types)
2. [Custom Exceptions](#2-custom-exceptions)
3. [Error Handling Strategies](#3-error-handling-strategies)
4. [Input Validation](#4-input-validation)
5. [Fail-Fast vs Fail-Safe](#5-fail-fast-vs-fail-safe)

---

## 1. Exception Types

### Checked vs Unchecked Exceptions

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           EXCEPTION HIERARCHY                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                              Throwable                                              │
│                                  │                                                  │
│                    ┌─────────────┴─────────────┐                                    │
│                    │                           │                                    │
│                  Error                     Exception                                │
│           (Don't catch!)                       │                                    │
│           - OutOfMemoryError     ┌─────────────┴─────────────┐                      │
│           - StackOverflowError   │                           │                      │
│                            RuntimeException            Checked Exceptions           │
│                            (Unchecked)                 (Must handle)                │
│                            - NullPointerException      - IOException                │
│                            - IllegalArgumentException  - SQLException               │
│                            - IndexOutOfBoundsException - FileNotFoundException      │
│                                                                                     │
│   CHECKED: Compiler forces you to handle or declare                                 │
│   UNCHECKED: Programming errors, don't need to declare                              │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### When to Use Which

| Type | When to Use | Example |
|------|-------------|---------|
| **Checked** | Recoverable conditions, external failures | File not found, network error |
| **Unchecked** | Programming errors, bugs | Null pointer, invalid argument |

---

## 2. Custom Exceptions

### Creating a Custom Exception Hierarchy

```java
// Base exception for the application
public class ApplicationException extends RuntimeException {
    private final ErrorCode errorCode;
    private final Map<String, Object> context;
    
    public ApplicationException(ErrorCode errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }
    
    public ApplicationException(ErrorCode errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }
    
    public ApplicationException addContext(String key, Object value) {
        context.put(key, value);
        return this;
    }
    
    public ErrorCode getErrorCode() { return errorCode; }
    public Map<String, Object> getContext() { return context; }
}

// Error codes enum
public enum ErrorCode {
    // Validation errors (400)
    INVALID_INPUT("E001", "Invalid input"),
    MISSING_REQUIRED_FIELD("E002", "Missing required field"),
    VALIDATION_FAILED("E003", "Validation failed"),
    
    // Not found errors (404)
    RESOURCE_NOT_FOUND("E101", "Resource not found"),
    USER_NOT_FOUND("E102", "User not found"),
    ORDER_NOT_FOUND("E103", "Order not found"),
    
    // Business logic errors (409)
    INSUFFICIENT_BALANCE("E201", "Insufficient balance"),
    DUPLICATE_ENTRY("E202", "Duplicate entry"),
    OPERATION_NOT_ALLOWED("E203", "Operation not allowed"),
    
    // External service errors (502, 503)
    EXTERNAL_SERVICE_ERROR("E301", "External service error"),
    DATABASE_ERROR("E302", "Database error"),
    
    // Internal errors (500)
    INTERNAL_ERROR("E999", "Internal server error");
    
    private final String code;
    private final String defaultMessage;
    
    ErrorCode(String code, String defaultMessage) {
        this.code = code;
        this.defaultMessage = defaultMessage;
    }
    
    public String getCode() { return code; }
    public String getDefaultMessage() { return defaultMessage; }
}

// Specific exception classes
public class ValidationException extends ApplicationException {
    private final List<ValidationError> errors;
    
    public ValidationException(List<ValidationError> errors) {
        super(ErrorCode.VALIDATION_FAILED, "Validation failed");
        this.errors = errors;
    }
    
    public List<ValidationError> getErrors() { return errors; }
}

public class ResourceNotFoundException extends ApplicationException {
    public ResourceNotFoundException(String resourceType, String resourceId) {
        super(ErrorCode.RESOURCE_NOT_FOUND, 
              resourceType + " not found with id: " + resourceId);
        addContext("resourceType", resourceType);
        addContext("resourceId", resourceId);
    }
}

public class BusinessException extends ApplicationException {
    public BusinessException(ErrorCode errorCode, String message) {
        super(errorCode, message);
    }
}

// Usage
public class UserService {
    public User getUser(String userId) {
        User user = userRepository.findById(userId);
        if (user == null) {
            throw new ResourceNotFoundException("User", userId);
        }
        return user;
    }
    
    public void transferMoney(String fromId, String toId, double amount) {
        Account from = getAccount(fromId);
        if (from.getBalance() < amount) {
            throw new BusinessException(ErrorCode.INSUFFICIENT_BALANCE,
                "Cannot transfer " + amount + ", available: " + from.getBalance())
                .addContext("requestedAmount", amount)
                .addContext("availableBalance", from.getBalance());
        }
        // Perform transfer
    }
}
```

---

## 3. Error Handling Strategies

### Global Exception Handler

```java
// Spring-style global exception handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        ErrorResponse response = new ErrorResponse(
            ex.getErrorCode().getCode(),
            ex.getMessage(),
            ex.getErrors()
        );
        return ResponseEntity.badRequest().body(response);
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse response = new ErrorResponse(
            ex.getErrorCode().getCode(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusiness(BusinessException ex) {
        ErrorResponse response = new ErrorResponse(
            ex.getErrorCode().getCode(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        logger.error("Unexpected error", ex);
        ErrorResponse response = new ErrorResponse(
            ErrorCode.INTERNAL_ERROR.getCode(),
            "An unexpected error occurred"
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}

// Error response DTO
public class ErrorResponse {
    private String errorCode;
    private String message;
    private LocalDateTime timestamp;
    private List<ValidationError> details;
    
    public ErrorResponse(String errorCode, String message) {
        this.errorCode = errorCode;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }
    
    // Constructor with details, getters, etc.
}
```

### Result Pattern (Functional Error Handling)

```java
// Result class for operations that can fail
public class Result<T> {
    private final T value;
    private final Error error;
    private final boolean success;
    
    private Result(T value, Error error, boolean success) {
        this.value = value;
        this.error = error;
        this.success = success;
    }
    
    public static <T> Result<T> success(T value) {
        return new Result<>(value, null, true);
    }
    
    public static <T> Result<T> failure(Error error) {
        return new Result<>(null, error, false);
    }
    
    public static <T> Result<T> failure(String message) {
        return new Result<>(null, new Error(message), false);
    }
    
    public boolean isSuccess() { return success; }
    public boolean isFailure() { return !success; }
    
    public T getValue() {
        if (!success) {
            throw new IllegalStateException("Cannot get value from failed result");
        }
        return value;
    }
    
    public Error getError() { return error; }
    
    public T getOrElse(T defaultValue) {
        return success ? value : defaultValue;
    }
    
    public <U> Result<U> map(Function<T, U> mapper) {
        if (success) {
            return Result.success(mapper.apply(value));
        }
        return Result.failure(error);
    }
    
    public <U> Result<U> flatMap(Function<T, Result<U>> mapper) {
        if (success) {
            return mapper.apply(value);
        }
        return Result.failure(error);
    }
}

// Usage
public class OrderService {
    public Result<Order> createOrder(OrderRequest request) {
        // Validate
        Result<OrderRequest> validationResult = validator.validate(request);
        if (validationResult.isFailure()) {
            return Result.failure(validationResult.getError());
        }
        
        // Check inventory
        Result<Boolean> inventoryResult = inventoryService.checkAvailability(request.getItems());
        if (inventoryResult.isFailure()) {
            return Result.failure(inventoryResult.getError());
        }
        
        // Process payment
        Result<PaymentConfirmation> paymentResult = paymentService.process(request.getPayment());
        if (paymentResult.isFailure()) {
            return Result.failure(paymentResult.getError());
        }
        
        // Create order
        Order order = new Order(request, paymentResult.getValue());
        return Result.success(orderRepository.save(order));
    }
}
```

---

## 4. Input Validation

### Validation Framework

```java
// Validation rule interface
public interface ValidationRule<T> {
    ValidationResult validate(T value);
}

// Validation result
public class ValidationResult {
    private final boolean valid;
    private final String message;
    
    private ValidationResult(boolean valid, String message) {
        this.valid = valid;
        this.message = message;
    }
    
    public static ValidationResult valid() {
        return new ValidationResult(true, null);
    }
    
    public static ValidationResult invalid(String message) {
        return new ValidationResult(false, message);
    }
    
    public boolean isValid() { return valid; }
    public String getMessage() { return message; }
}

// Common validation rules
public class ValidationRules {
    
    public static ValidationRule<String> notEmpty() {
        return value -> {
            if (value == null || value.trim().isEmpty()) {
                return ValidationResult.invalid("Value cannot be empty");
            }
            return ValidationResult.valid();
        };
    }
    
    public static ValidationRule<String> minLength(int min) {
        return value -> {
            if (value == null || value.length() < min) {
                return ValidationResult.invalid("Minimum length is " + min);
            }
            return ValidationResult.valid();
        };
    }
    
    public static ValidationRule<String> maxLength(int max) {
        return value -> {
            if (value != null && value.length() > max) {
                return ValidationResult.invalid("Maximum length is " + max);
            }
            return ValidationResult.valid();
        };
    }
    
    public static ValidationRule<String> email() {
        return value -> {
            if (value == null || !value.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
                return ValidationResult.invalid("Invalid email format");
            }
            return ValidationResult.valid();
        };
    }
    
    public static <T extends Comparable<T>> ValidationRule<T> min(T min) {
        return value -> {
            if (value == null || value.compareTo(min) < 0) {
                return ValidationResult.invalid("Value must be at least " + min);
            }
            return ValidationResult.valid();
        };
    }
}

// Validator builder
public class Validator<T> {
    private final List<FieldValidation> validations = new ArrayList<>();
    
    public <V> Validator<T> field(String fieldName, Function<T, V> extractor, 
                                   ValidationRule<V>... rules) {
        validations.add(new FieldValidation(fieldName, extractor, Arrays.asList(rules)));
        return this;
    }
    
    public List<ValidationError> validate(T object) {
        List<ValidationError> errors = new ArrayList<>();
        
        for (FieldValidation validation : validations) {
            Object value = validation.extractor.apply(object);
            for (ValidationRule rule : validation.rules) {
                ValidationResult result = rule.validate(value);
                if (!result.isValid()) {
                    errors.add(new ValidationError(validation.fieldName, result.getMessage()));
                }
            }
        }
        
        return errors;
    }
    
    public void validateAndThrow(T object) {
        List<ValidationError> errors = validate(object);
        if (!errors.isEmpty()) {
            throw new ValidationException(errors);
        }
    }
}

// Usage
Validator<UserRequest> userValidator = new Validator<UserRequest>()
    .field("email", UserRequest::getEmail, 
           ValidationRules.notEmpty(), 
           ValidationRules.email())
    .field("name", UserRequest::getName, 
           ValidationRules.notEmpty(), 
           ValidationRules.minLength(2),
           ValidationRules.maxLength(100))
    .field("age", UserRequest::getAge, 
           ValidationRules.min(18));

userValidator.validateAndThrow(request);
```

---

## 5. Fail-Fast vs Fail-Safe

### Fail-Fast

**Fail-Fast**: Immediately report any failure condition. Better for development and catching bugs early.

```java
public class FailFastExample {
    
    // Fail-fast: Check preconditions immediately
    public void processOrder(Order order) {
        Objects.requireNonNull(order, "Order cannot be null");
        
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
        
        if (order.getTotal() <= 0) {
            throw new IllegalArgumentException("Order total must be positive");
        }
        
        // Process only if all preconditions pass
        doProcessOrder(order);
    }
    
    // Fail-fast iterator (ConcurrentModificationException)
    public void iterateList() {
        List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
        
        for (String item : list) {
            if (item.equals("b")) {
                list.remove(item);  // Throws ConcurrentModificationException
            }
        }
    }
}
```

### Fail-Safe

**Fail-Safe**: Continue operation even if failures occur. Better for production stability.

```java
public class FailSafeExample {
    
    // Fail-safe: Graceful degradation
    public OrderSummary getOrderSummary(String orderId) {
        Order order = orderRepository.findById(orderId);
        
        OrderSummary summary = new OrderSummary(order);
        
        // Try to enrich with additional data, but don't fail if unavailable
        try {
            Customer customer = customerService.getCustomer(order.getCustomerId());
            summary.setCustomerName(customer.getName());
        } catch (Exception e) {
            logger.warn("Could not fetch customer details", e);
            summary.setCustomerName("Unknown");
        }
        
        try {
            ShippingInfo shipping = shippingService.getShippingInfo(orderId);
            summary.setShippingStatus(shipping.getStatus());
        } catch (Exception e) {
            logger.warn("Could not fetch shipping info", e);
            summary.setShippingStatus("Unknown");
        }
        
        return summary;
    }
    
    // Fail-safe iterator (CopyOnWriteArrayList)
    public void iterateListSafe() {
        List<String> list = new CopyOnWriteArrayList<>(Arrays.asList("a", "b", "c"));
        
        for (String item : list) {
            if (item.equals("b")) {
                list.remove(item);  // Safe - works on a copy
            }
        }
    }
}
```

### When to Use Which

| Scenario | Approach | Reason |
|----------|----------|--------|
| Development/Testing | Fail-Fast | Catch bugs early |
| Critical operations | Fail-Fast | Data integrity |
| Optional features | Fail-Safe | User experience |
| External dependencies | Fail-Safe | Resilience |
| Input validation | Fail-Fast | Security |

---

## Summary

| Concept | Best Practice |
|---------|---------------|
| **Custom Exceptions** | Create meaningful hierarchy |
| **Error Codes** | Use consistent, documented codes |
| **Global Handler** | Centralize exception handling |
| **Validation** | Validate early, fail fast |
| **Error Response** | Provide actionable error messages |
| **Logging** | Log errors with context |

---

**Next Section: [API & Interface Design](../08-api-design/README.md)** →
