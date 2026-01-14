# 📘 Section 12: Testability & Clean Code

> **Interviewers silently judge this** - Writing testable, maintainable code

---

## 📑 Table of Contents

1. [Writing Testable Code](#1-writing-testable-code)
2. [Dependency Injection](#2-dependency-injection)
3. [Mocking Strategies](#3-mocking-strategies)
4. [Unit vs Integration Tests](#4-unit-vs-integration-tests)
5. [Code Readability & Naming](#5-code-readability--naming)

---

## 1. Writing Testable Code

### Principles of Testable Code

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                       TESTABLE CODE PRINCIPLES                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ✓ Inject dependencies (don't create them)                                         │
│   ✓ Favor composition over inheritance                                              │
│   ✓ Use interfaces for external dependencies                                        │
│   ✓ Avoid static methods for logic                                                  │
│   ✓ Keep methods small and focused                                                  │
│   ✓ Avoid hidden dependencies (global state, singletons)                            │
│   ✓ Make side effects explicit                                                      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Example: Testable vs Non-Testable Code

```java
// ❌ HARD TO TEST
public class OrderService {
    public Order createOrder(OrderRequest request) {
        // Hidden dependency - creates own database connection
        Database db = new MySQLDatabase();
        
        // Static method call - can't mock
        if (!Validator.isValid(request)) {
            throw new ValidationException("Invalid request");
        }
        
        // Direct instantiation - can't mock
        PaymentGateway gateway = new StripeGateway();
        gateway.charge(request.getPayment());
        
        // Static method - hard to control in tests
        String orderId = UUID.randomUUID().toString();
        
        Order order = new Order(orderId, request);
        db.save(order);
        
        // Side effect - sends real emails in tests
        EmailService.sendConfirmation(order);
        
        return order;
    }
}

// ✓ TESTABLE
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentGateway paymentGateway;
    private final OrderValidator validator;
    private final NotificationService notificationService;
    private final IdGenerator idGenerator;
    
    // Dependencies injected via constructor
    public OrderService(
            OrderRepository orderRepository,
            PaymentGateway paymentGateway,
            OrderValidator validator,
            NotificationService notificationService,
            IdGenerator idGenerator) {
        this.orderRepository = orderRepository;
        this.paymentGateway = paymentGateway;
        this.validator = validator;
        this.notificationService = notificationService;
        this.idGenerator = idGenerator;
    }
    
    public Order createOrder(OrderRequest request) {
        // All dependencies are mockable
        ValidationResult validation = validator.validate(request);
        if (!validation.isValid()) {
            throw new ValidationException(validation.getErrors());
        }
        
        PaymentResult payment = paymentGateway.charge(request.getPayment());
        if (!payment.isSuccessful()) {
            throw new PaymentFailedException(payment.getError());
        }
        
        Order order = new Order(idGenerator.generate(), request);
        order.setPaymentId(payment.getTransactionId());
        
        Order savedOrder = orderRepository.save(order);
        
        notificationService.sendOrderConfirmation(savedOrder);
        
        return savedOrder;
    }
}

// Easy to test
@Test
void shouldCreateOrderSuccessfully() {
    // Arrange - create mocks
    OrderRepository mockRepo = mock(OrderRepository.class);
    PaymentGateway mockPayment = mock(PaymentGateway.class);
    OrderValidator mockValidator = mock(OrderValidator.class);
    NotificationService mockNotification = mock(NotificationService.class);
    IdGenerator mockIdGen = mock(IdGenerator.class);
    
    when(mockValidator.validate(any())).thenReturn(ValidationResult.valid());
    when(mockPayment.charge(any())).thenReturn(PaymentResult.success("txn-123"));
    when(mockIdGen.generate()).thenReturn("order-123");
    when(mockRepo.save(any())).thenAnswer(i -> i.getArgument(0));
    
    OrderService service = new OrderService(
        mockRepo, mockPayment, mockValidator, mockNotification, mockIdGen
    );
    
    // Act
    Order result = service.createOrder(new OrderRequest(...));
    
    // Assert
    assertEquals("order-123", result.getId());
    verify(mockPayment).charge(any());
    verify(mockRepo).save(any());
    verify(mockNotification).sendOrderConfirmation(any());
}
```

---

## 2. Dependency Injection

### Types of Dependency Injection

```java
// 1. Constructor Injection (Recommended)
public class OrderService {
    private final OrderRepository repository;
    
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}

// 2. Setter Injection
public class OrderService {
    private OrderRepository repository;
    
    public void setRepository(OrderRepository repository) {
        this.repository = repository;
    }
}

// 3. Field Injection (Avoid - makes testing harder)
public class OrderService {
    @Inject
    private OrderRepository repository;
}
```

### Simple DI Container

```java
public class DIContainer {
    private final Map<Class<?>, Object> instances = new HashMap<>();
    private final Map<Class<?>, Supplier<?>> factories = new HashMap<>();
    
    public <T> void registerSingleton(Class<T> type, T instance) {
        instances.put(type, instance);
    }
    
    public <T> void registerFactory(Class<T> type, Supplier<T> factory) {
        factories.put(type, factory);
    }
    
    @SuppressWarnings("unchecked")
    public <T> T resolve(Class<T> type) {
        // Check singletons first
        if (instances.containsKey(type)) {
            return (T) instances.get(type);
        }
        
        // Check factories
        if (factories.containsKey(type)) {
            return (T) factories.get(type).get();
        }
        
        throw new RuntimeException("No registration for: " + type);
    }
}

// Usage
DIContainer container = new DIContainer();
container.registerSingleton(OrderRepository.class, new InMemoryOrderRepository());
container.registerSingleton(PaymentGateway.class, new MockPaymentGateway());
container.registerFactory(OrderService.class, () -> 
    new OrderService(
        container.resolve(OrderRepository.class),
        container.resolve(PaymentGateway.class)
    )
);

OrderService service = container.resolve(OrderService.class);
```

---

## 3. Mocking Strategies

### Test Doubles

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              TEST DOUBLES                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   DUMMY      - Passed but never used                                                │
│   STUB       - Returns canned answers                                               │
│   SPY        - Records calls for verification                                       │
│   MOCK       - Pre-programmed with expectations                                     │
│   FAKE       - Working implementation (simplified)                                  │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Examples

```java
// STUB - Returns predefined values
public class StubPaymentGateway implements PaymentGateway {
    @Override
    public PaymentResult charge(PaymentRequest request) {
        return PaymentResult.success("stub-txn-123");
    }
}

// FAKE - Simplified working implementation
public class FakeOrderRepository implements OrderRepository {
    private final Map<String, Order> storage = new HashMap<>();
    
    @Override
    public Order save(Order order) {
        storage.put(order.getId(), order);
        return order;
    }
    
    @Override
    public Optional<Order> findById(String id) {
        return Optional.ofNullable(storage.get(id));
    }
}

// SPY - Records interactions for verification
public class SpyNotificationService implements NotificationService {
    private final List<Notification> sentNotifications = new ArrayList<>();
    
    @Override
    public void send(Notification notification) {
        sentNotifications.add(notification);
    }
    
    public List<Notification> getSentNotifications() {
        return sentNotifications;
    }
    
    public boolean wasCalled() {
        return !sentNotifications.isEmpty();
    }
}

// Using Mockito
@Test
void testWithMockito() {
    // Create mock
    OrderRepository mockRepo = mock(OrderRepository.class);
    
    // Stub behavior
    when(mockRepo.findById("123")).thenReturn(Optional.of(testOrder));
    when(mockRepo.save(any())).thenAnswer(invocation -> invocation.getArgument(0));
    
    // Use mock
    OrderService service = new OrderService(mockRepo, ...);
    Order result = service.getOrder("123");
    
    // Verify interactions
    verify(mockRepo).findById("123");
    verify(mockRepo, never()).save(any());
    verify(mockRepo, times(1)).findById(anyString());
}
```

---

## 4. Unit vs Integration Tests

### Test Pyramid

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              TEST PYRAMID                                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                          /\                                                         │
│                         /  \                                                        │
│                        / E2E\           Slow, Expensive, Few                        │
│                       /──────\                                                      │
│                      /        \                                                     │
│                     /Integration\       Medium Speed, Some                          │
│                    /──────────────\                                                 │
│                   /                \                                                │
│                  /    Unit Tests    \   Fast, Cheap, Many                           │
│                 /────────────────────\                                              │
│                                                                                     │
│   Unit Tests:        Test single class/method in isolation                          │
│   Integration Tests: Test multiple components together                              │
│   E2E Tests:         Test entire system from user perspective                       │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Example: Unit Test

```java
class OrderTest {
    
    @Test
    void shouldCalculateTotalCorrectly() {
        // Arrange
        Order order = Order.create(new CustomerId("cust-1"));
        
        // Act
        order.addItem(new ProductId("prod-1"), "Widget", Money.of(10.00, USD), 2);
        order.addItem(new ProductId("prod-2"), "Gadget", Money.of(25.00, USD), 1);
        
        // Assert
        assertEquals(Money.of(45.00, USD), order.getTotal());
    }
    
    @Test
    void shouldNotAllowModificationAfterConfirm() {
        // Arrange
        Order order = createOrderWithItems();
        order.confirm();
        
        // Act & Assert
        assertThrows(InvalidStateException.class, () -> 
            order.addItem(new ProductId("prod-3"), "Another", Money.of(5.00, USD), 1)
        );
    }
}
```

### Example: Integration Test

```java
@SpringBootTest
@Transactional
class OrderServiceIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @MockBean
    private PaymentGateway paymentGateway;  // Mock external service
    
    @Test
    void shouldCreateAndPersistOrder() {
        // Arrange
        when(paymentGateway.charge(any()))
            .thenReturn(PaymentResult.success("txn-123"));
        
        OrderRequest request = new OrderRequest(...)
        
        // Act
        Order result = orderService.createOrder(request);
        
        // Assert - verify persistence
        Optional<Order> persisted = orderRepository.findById(result.getId());
        assertTrue(persisted.isPresent());
        assertEquals(OrderStatus.CONFIRMED, persisted.get().getStatus());
    }
}
```

---

## 5. Code Readability & Naming

### Clean Code Principles

```java
// ❌ BAD - Unclear naming, magic numbers, deep nesting
public double calc(List<Item> l, int t) {
    double r = 0;
    for (int i = 0; i < l.size(); i++) {
        if (l.get(i).getType() == 1) {
            if (l.get(i).getQty() > 10) {
                r += l.get(i).getPrice() * 0.9;
            } else {
                r += l.get(i).getPrice();
            }
        } else if (l.get(i).getType() == 2) {
            r += l.get(i).getPrice() * 1.1;
        }
    }
    if (t == 1) {
        r = r * 0.95;
    }
    return r;
}

// ✓ GOOD - Clear naming, constants, early returns
public class PriceCalculator {
    private static final double BULK_DISCOUNT = 0.10;      // 10% discount
    private static final int BULK_THRESHOLD = 10;
    private static final double PREMIUM_MARKUP = 0.10;     // 10% markup
    private static final double MEMBER_DISCOUNT = 0.05;    // 5% discount
    
    public Money calculateTotal(List<OrderItem> items, CustomerType customerType) {
        Money subtotal = calculateSubtotal(items);
        return applyCustomerDiscount(subtotal, customerType);
    }
    
    private Money calculateSubtotal(List<OrderItem> items) {
        return items.stream()
            .map(this::calculateItemPrice)
            .reduce(Money.ZERO, Money::add);
    }
    
    private Money calculateItemPrice(OrderItem item) {
        Money basePrice = item.getUnitPrice().multiply(item.getQuantity());
        
        if (item.isPremium()) {
            return applyPremiumMarkup(basePrice);
        }
        
        if (item.getQuantity() > BULK_THRESHOLD) {
            return applyBulkDiscount(basePrice);
        }
        
        return basePrice;
    }
    
    private Money applyPremiumMarkup(Money price) {
        return price.multiply(1 + PREMIUM_MARKUP);
    }
    
    private Money applyBulkDiscount(Money price) {
        return price.multiply(1 - BULK_DISCOUNT);
    }
    
    private Money applyCustomerDiscount(Money total, CustomerType type) {
        if (type == CustomerType.MEMBER) {
            return total.multiply(1 - MEMBER_DISCOUNT);
        }
        return total;
    }
}
```

### Naming Conventions Summary

| Type | Convention | Example |
|------|------------|---------|
| **Class** | Noun, PascalCase | `OrderService`, `PaymentGateway` |
| **Method** | Verb, camelCase | `calculateTotal()`, `findById()` |
| **Variable** | Noun, camelCase | `orderCount`, `customerName` |
| **Constant** | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT` |
| **Boolean** | is/has/can/should | `isActive`, `hasPermission` |

---

**Next Section: [Common LLD Interview Problems](../13-interview-problems/README.md)** →
