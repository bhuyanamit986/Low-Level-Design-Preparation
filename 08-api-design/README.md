# 📘 Section 8: API & Interface Design

> **Well-designed APIs are intuitive, consistent, and hard to misuse**

---

## 📑 Table of Contents

1. [Designing Clean APIs](#1-designing-clean-apis)
2. [Method Naming Conventions](#2-method-naming-conventions)
3. [DTOs vs Entities](#3-dtos-vs-entities)
4. [API Versioning](#4-api-versioning)
5. [Idempotency](#5-idempotency)

---

## 1. Designing Clean APIs

### Principles of Good API Design

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         API DESIGN PRINCIPLES                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   1. EASY TO USE CORRECTLY                                                          │
│      - Intuitive naming                                                             │
│      - Sensible defaults                                                            │
│      - Clear documentation                                                          │
│                                                                                     │
│   2. HARD TO USE INCORRECTLY                                                        │
│      - Compile-time checks over runtime                                             │
│      - Validate inputs                                                              │
│      - Clear error messages                                                         │
│                                                                                     │
│   3. CONSISTENT                                                                     │
│      - Follow naming conventions                                                    │
│      - Similar operations look similar                                              │
│      - Predictable behavior                                                         │
│                                                                                     │
│   4. MINIMAL                                                                        │
│      - Don't expose implementation details                                          │
│      - Only what's necessary                                                        │
│      - Easy to add, hard to remove                                                  │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Example: Designing a Payment Service API

```java
// ❌ BAD API DESIGN
public class PaymentService {
    // Too many parameters, easy to mix up
    public String pay(String from, String to, double amount, String currency,
                      String type, boolean save, String note) { ... }
    
    // Unclear return type
    public Object getPayment(String id) { ... }
    
    // Inconsistent naming
    public void cancelPayment(String id) { ... }
    public void remove_payment(String id) { ... }
    public boolean checkIfPaymentExists(String id) { ... }
}

// ✓ GOOD API DESIGN
public class PaymentService {
    
    // Use builder pattern for complex operations
    public PaymentResult processPayment(PaymentRequest request) {
        validateRequest(request);
        // ...
    }
    
    // Clear, typed return values
    public Optional<Payment> findById(String paymentId) {
        return paymentRepository.findById(paymentId);
    }
    
    // Consistent naming
    public List<Payment> findByCustomerId(String customerId) { ... }
    public List<Payment> findByDateRange(LocalDate start, LocalDate end) { ... }
    
    // Clear status check
    public boolean exists(String paymentId) {
        return paymentRepository.existsById(paymentId);
    }
    
    // Operations return result objects
    public CancellationResult cancel(String paymentId, CancellationReason reason) { ... }
    public RefundResult refund(RefundRequest request) { ... }
}

// Request object with builder
public class PaymentRequest {
    private final String fromAccountId;
    private final String toAccountId;
    private final Money amount;
    private final PaymentMethod method;
    private final String description;
    
    private PaymentRequest(Builder builder) {
        this.fromAccountId = Objects.requireNonNull(builder.fromAccountId);
        this.toAccountId = Objects.requireNonNull(builder.toAccountId);
        this.amount = Objects.requireNonNull(builder.amount);
        this.method = builder.method != null ? builder.method : PaymentMethod.BANK_TRANSFER;
        this.description = builder.description;
    }
    
    public static class Builder {
        private String fromAccountId;
        private String toAccountId;
        private Money amount;
        private PaymentMethod method;
        private String description;
        
        public Builder from(String accountId) {
            this.fromAccountId = accountId;
            return this;
        }
        
        public Builder to(String accountId) {
            this.toAccountId = accountId;
            return this;
        }
        
        public Builder amount(Money amount) {
            this.amount = amount;
            return this;
        }
        
        public Builder method(PaymentMethod method) {
            this.method = method;
            return this;
        }
        
        public Builder description(String description) {
            this.description = description;
            return this;
        }
        
        public PaymentRequest build() {
            return new PaymentRequest(this);
        }
    }
}

// Usage - Clear and hard to misuse
PaymentRequest request = new PaymentRequest.Builder()
    .from("account-123")
    .to("account-456")
    .amount(Money.of(100.00, Currency.USD))
    .method(PaymentMethod.CREDIT_CARD)
    .description("Monthly subscription")
    .build();

PaymentResult result = paymentService.processPayment(request);
```

---

## 2. Method Naming Conventions

### Naming Guidelines

| Purpose | Convention | Examples |
|---------|------------|----------|
| **Get single item** | `get`, `find`, `fetch` | `getUser()`, `findById()` |
| **Get collection** | `list`, `findAll`, `getAll` | `listUsers()`, `findAllByStatus()` |
| **Create** | `create`, `add`, `register` | `createUser()`, `addItem()` |
| **Update** | `update`, `modify`, `change` | `updateUser()`, `changeStatus()` |
| **Delete** | `delete`, `remove` | `deleteUser()`, `removeItem()` |
| **Check existence** | `exists`, `has`, `is` | `exists()`, `hasPermission()`, `isActive()` |
| **Check state** | `is`, `can`, `should` | `isValid()`, `canProcess()` |
| **Convert** | `to`, `as`, `from` | `toString()`, `asList()`, `fromJson()` |
| **Compute** | `calculate`, `compute` | `calculateTotal()`, `computeHash()` |

### Boolean Method Naming

```java
// ✓ GOOD - Clear boolean names
public boolean isActive() { ... }
public boolean hasPermission(String permission) { ... }
public boolean canProcess() { ... }
public boolean exists(String id) { ... }
public boolean contains(String item) { ... }
public boolean isEmpty() { ... }

// ❌ BAD - Unclear boolean names
public boolean active() { ... }         // Noun, not clear it returns boolean
public boolean checkPermission() { ... } // Sounds like void method
public boolean process() { ... }         // Sounds like action
```

### Method Overloading Best Practices

```java
public class UserService {
    // Clear overloading with different use cases
    public User findById(String id) { ... }
    
    public User findByEmail(String email) { ... }
    
    public List<User> findByRole(Role role) { ... }
    
    public List<User> findByRole(Role role, int limit) { ... }
    
    // ❌ AVOID - Too similar, easy to confuse
    public User find(String id) { ... }
    public User find(String email) { ... }  // Same signature!
}
```

---

## 3. DTOs vs Entities

### When to Use Each

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           DTOs vs ENTITIES                                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ENTITY (Domain Object)              DTO (Data Transfer Object)                    │
│   ┌─────────────────────┐             ┌─────────────────────┐                       │
│   │ - Has identity      │             │ - No identity       │                       │
│   │ - Has behavior      │             │ - Only data         │                       │
│   │ - Business rules    │             │ - No business logic │                       │
│   │ - Internal to domain│             │ - External interface│                       │
│   └─────────────────────┘             └─────────────────────┘                       │
│                                                                                     │
│   WHEN TO USE:                        WHEN TO USE:                                  │
│   - Business logic                    - API requests/responses                      │
│   - Database operations               - Service boundaries                          │
│   - Domain layer                      - External communication                      │
│                                       - Hiding internal details                     │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementation

```java
// Entity - Rich domain model with behavior
@Entity
public class Order {
    @Id
    private String id;
    private String customerId;
    private List<OrderItem> items;
    private OrderStatus status;
    private Money totalAmount;
    private LocalDateTime createdAt;
    
    // Business behavior
    public void addItem(Product product, int quantity) {
        validateCanModify();
        OrderItem item = new OrderItem(product, quantity);
        items.add(item);
        recalculateTotal();
    }
    
    public void removeItem(String itemId) {
        validateCanModify();
        items.removeIf(i -> i.getId().equals(itemId));
        recalculateTotal();
    }
    
    public void confirm() {
        if (items.isEmpty()) {
            throw new BusinessException("Cannot confirm empty order");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    private void validateCanModify() {
        if (status != OrderStatus.DRAFT) {
            throw new BusinessException("Cannot modify confirmed order");
        }
    }
    
    private void recalculateTotal() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
}

// Request DTO - For creating orders
public class CreateOrderRequest {
    @NotNull
    private String customerId;
    
    @NotEmpty
    private List<OrderItemRequest> items;
    
    private String notes;
    
    // Only getters and setters, no business logic
    public String getCustomerId() { return customerId; }
    public void setCustomerId(String customerId) { this.customerId = customerId; }
    // ...
}

// Response DTO - For API responses
public class OrderResponse {
    private String id;
    private String customerId;
    private String customerName;  // Flattened from Customer entity
    private List<OrderItemResponse> items;
    private String status;
    private BigDecimal totalAmount;
    private String currency;
    private String createdAt;
    
    // Static factory method for conversion
    public static OrderResponse from(Order order, Customer customer) {
        OrderResponse response = new OrderResponse();
        response.id = order.getId();
        response.customerId = order.getCustomerId();
        response.customerName = customer.getName();
        response.items = order.getItems().stream()
            .map(OrderItemResponse::from)
            .collect(toList());
        response.status = order.getStatus().name();
        response.totalAmount = order.getTotalAmount().getAmount();
        response.currency = order.getTotalAmount().getCurrency().getCode();
        response.createdAt = order.getCreatedAt().toString();
        return response;
    }
}

// Controller using DTOs
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        Customer customer = customerService.findById(order.getCustomerId());
        return ResponseEntity.ok(OrderResponse.from(order, customer));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String id) {
        Order order = orderService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Order", id));
        Customer customer = customerService.findById(order.getCustomerId());
        return ResponseEntity.ok(OrderResponse.from(order, customer));
    }
}
```

---

## 4. API Versioning

### Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/api/v1/users` | Clear, easy to route | URL changes |
| **Query Param** | `/api/users?version=1` | Optional, backward compatible | Can be overlooked |
| **Header** | `Accept: application/vnd.api.v1+json` | Clean URLs | Less visible |
| **Content-Type** | `Content-Type: application/vnd.api.v1+json` | Standard HTTP | Complex |

### Implementation: URL Path Versioning

```java
// Version 1 Controller
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseV1> getUser(@PathVariable String id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(UserResponseV1.from(user));
    }
}

// Version 2 Controller with breaking changes
@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseV2> getUser(@PathVariable String id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(UserResponseV2.from(user));
    }
}

// V1 Response - Original structure
public class UserResponseV1 {
    private String id;
    private String name;  // Full name as single field
    private String email;
}

// V2 Response - Breaking change: split name
public class UserResponseV2 {
    private String id;
    private String firstName;  // Name split into parts
    private String lastName;
    private String email;
    private AddressResponse address;  // New field
}
```

### Backward Compatible Changes

```java
// These changes are backward compatible:
// 1. Adding new optional fields
// 2. Adding new endpoints
// 3. Adding new optional query parameters
// 4. Making required fields optional

// ✓ V1 response - original
{
    "id": "123",
    "name": "John Doe"
}

// ✓ V1.1 response - added optional field (backward compatible)
{
    "id": "123",
    "name": "John Doe",
    "createdAt": "2024-01-01T10:00:00Z"  // New optional field
}

// ❌ Breaking change - removed field
{
    "id": "123"
    // "name" removed - breaks existing clients!
}

// ❌ Breaking change - renamed field
{
    "id": "123",
    "fullName": "John Doe"  // Renamed from "name" - breaks clients!
}
```

---

## 5. Idempotency

### What is Idempotency?

An operation is **idempotent** if performing it multiple times has the same effect as performing it once.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              IDEMPOTENCY                                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   HTTP Methods:                                                                     │
│   ┌──────────────┬─────────────┬───────────────────────────────────────────────┐    │
│   │    Method    │ Idempotent? │                   Example                     │    │
│   ├──────────────┼─────────────┼───────────────────────────────────────────────┤    │
│   │ GET          │     ✓       │ GET /users/123 always returns same user       │    │
│   │ PUT          │     ✓       │ PUT /users/123 {name: "John"} → same result   │    │
│   │ DELETE       │     ✓       │ DELETE /users/123 → user deleted (or 404)     │    │
│   │ POST         │     ✗       │ POST /orders → creates new order each time    │    │
│   │ PATCH        │     ✗*      │ Depends on implementation                     │    │
│   └──────────────┴─────────────┴───────────────────────────────────────────────┘    │
│                                                                                     │
│   Making POST idempotent:                                                           │
│   - Use idempotency keys                                                            │
│   - Client generates unique key for each logical request                            │
│   - Server stores and checks key                                                    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementing Idempotency

```java
// Idempotency key storage
public interface IdempotencyStore {
    boolean exists(String key);
    void store(String key, Object response, Duration ttl);
    Optional<Object> get(String key);
}

@Service
public class IdempotencyService {
    private final IdempotencyStore store;
    private static final Duration DEFAULT_TTL = Duration.ofHours(24);
    
    public IdempotencyService(IdempotencyStore store) {
        this.store = store;
    }
    
    public <T> T executeIdempotent(String idempotencyKey, Supplier<T> operation) {
        // Check if already processed
        Optional<Object> cachedResult = store.get(idempotencyKey);
        if (cachedResult.isPresent()) {
            return (T) cachedResult.get();
        }
        
        // Execute operation
        T result = operation.get();
        
        // Store result
        store.store(idempotencyKey, result, DEFAULT_TTL);
        
        return result;
    }
}

// Controller with idempotency
@RestController
@RequestMapping("/api/payments")
public class PaymentController {
    
    private final PaymentService paymentService;
    private final IdempotencyService idempotencyService;
    
    @PostMapping
    public ResponseEntity<PaymentResponse> createPayment(
            @RequestHeader("Idempotency-Key") String idempotencyKey,
            @RequestBody PaymentRequest request) {
        
        PaymentResponse response = idempotencyService.executeIdempotent(
            idempotencyKey,
            () -> paymentService.processPayment(request)
        );
        
        return ResponseEntity.ok(response);
    }
}

// Client usage
// First request
POST /api/payments
Idempotency-Key: unique-key-123
{...}
→ Payment processed, returns PaymentResponse

// Retry with same key (e.g., network timeout)
POST /api/payments
Idempotency-Key: unique-key-123
{...}
→ Returns cached PaymentResponse (no duplicate payment!)
```

---

## API Design Checklist

| Aspect | Check |
|--------|-------|
| **Naming** | Consistent, clear, follows conventions |
| **Parameters** | Validated, documented, sensible defaults |
| **Responses** | Typed, consistent structure, proper status codes |
| **Errors** | Meaningful messages, error codes, documentation |
| **Versioning** | Strategy chosen, backward compatibility |
| **Idempotency** | POST operations have idempotency keys |
| **Documentation** | OpenAPI/Swagger, examples, changelog |

---

**Next Section: [Domain Modeling](../09-domain-modeling/README.md)** →
