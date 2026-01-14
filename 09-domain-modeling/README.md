# 📘 Section 9: Domain Modeling (Critical Skill ⭐⭐⭐)

> **This is where strong candidates stand out** - Understanding the problem domain deeply

---

## 📑 Table of Contents

1. [Identifying Entities](#1-identifying-entities)
2. [Value Objects](#2-value-objects)
3. [Aggregates and Aggregate Roots](#3-aggregates-and-aggregate-roots)
4. [Domain Boundaries](#4-domain-boundaries)
5. [Avoiding Anemic Models](#5-avoiding-anemic-models)

---

## 1. Identifying Entities

### What is an Entity?

An **entity** is a domain object defined by its **identity**, not its attributes. Two entities with the same attributes but different IDs are different entities.

### Entity Characteristics

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                               ENTITY CHARACTERISTICS                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌──────────────────────────────────────────────────────────────────────────────┐  │
│   │                              ENTITY                                          │  │
│   ├──────────────────────────────────────────────────────────────────────────────┤  │
│   │  ✓ Has a unique identifier (ID)                                              │  │
│   │  ✓ Identity remains constant over time                                       │  │
│   │  ✓ Attributes can change, but it's still the same entity                     │  │
│   │  ✓ Equality is based on ID, not attributes                                   │  │
│   │  ✓ Has a lifecycle (created, modified, deleted)                              │  │
│   └──────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                     │
│   Examples:                                                                         │
│   - User (identified by userId)                                                     │
│   - Order (identified by orderId)                                                   │
│   - Product (identified by SKU or productId)                                        │
│   - Account (identified by accountNumber)                                           │
│                                                                                     │
│   Counter-examples (NOT entities):                                                  │
│   - Address (typically a value object)                                              │
│   - Money (value object - $100 is $100, no identity)                                │
│   - DateRange (value object)                                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementing Entities

```java
// Entity base class
public abstract class Entity<ID> {
    protected ID id;
    
    protected Entity() {}
    
    protected Entity(ID id) {
        this.id = id;
    }
    
    public ID getId() {
        return id;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Entity<?> entity = (Entity<?>) o;
        return id != null && id.equals(entity.id);
    }
    
    @Override
    public int hashCode() {
        return id != null ? id.hashCode() : 0;
    }
}

// Concrete entity
public class User extends Entity<UserId> {
    private String email;
    private String name;
    private UserStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime lastModifiedAt;
    
    // Private constructor - use factory method
    private User(UserId id, String email, String name) {
        super(id);
        this.email = email;
        this.name = name;
        this.status = UserStatus.ACTIVE;
        this.createdAt = LocalDateTime.now();
        this.lastModifiedAt = this.createdAt;
    }
    
    // Factory method
    public static User create(String email, String name) {
        validateEmail(email);
        validateName(name);
        return new User(UserId.generate(), email, name);
    }
    
    // Business methods
    public void updateProfile(String newName, String newEmail) {
        validateEmail(newEmail);
        validateName(newName);
        this.name = newName;
        this.email = newEmail;
        this.lastModifiedAt = LocalDateTime.now();
    }
    
    public void deactivate() {
        if (this.status == UserStatus.DEACTIVATED) {
            throw new BusinessException("User already deactivated");
        }
        this.status = UserStatus.DEACTIVATED;
        this.lastModifiedAt = LocalDateTime.now();
    }
    
    public void activate() {
        if (this.status == UserStatus.ACTIVE) {
            throw new BusinessException("User already active");
        }
        this.status = UserStatus.ACTIVE;
        this.lastModifiedAt = LocalDateTime.now();
    }
    
    // Validation methods
    private static void validateEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new ValidationException("Invalid email");
        }
    }
    
    private static void validateName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new ValidationException("Name cannot be empty");
        }
    }
    
    // Getters (no setters - use business methods)
    public String getEmail() { return email; }
    public String getName() { return name; }
    public UserStatus getStatus() { return status; }
    public boolean isActive() { return status == UserStatus.ACTIVE; }
}
```

---

## 2. Value Objects

### What is a Value Object?

A **value object** is defined by its **attributes**, not identity. Two value objects with the same attributes are equal and interchangeable.

### Value Object Characteristics

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           VALUE OBJECT CHARACTERISTICS                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌──────────────────────────────────────────────────────────────────────────────┐  │
│   │                           VALUE OBJECT                                       │  │
│   ├──────────────────────────────────────────────────────────────────────────────┤  │
│   │  ✓ No unique identifier                                                      │  │
│   │  ✓ Immutable (cannot change after creation)                                  │  │
│   │  ✓ Equality based on all attributes                                          │  │
│   │  ✓ Side-effect free behavior                                                 │  │
│   │  ✓ Replaceable (can swap one for another with same values)                   │  │
│   └──────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                     │
│   Examples:                                                                         │
│   - Money (amount + currency)                                                       │
│   - Address (street, city, country)                                                 │
│   - DateRange (start, end)                                                          │
│   - Email (validated email string)                                                  │
│   - PhoneNumber                                                                     │
│   - Coordinates (latitude, longitude)                                               │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementing Value Objects

```java
// Money value object
public final class Money {
    public static final Money ZERO = new Money(BigDecimal.ZERO, Currency.USD);
    
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, Currency currency) {
        if (amount == null || currency == null) {
            throw new IllegalArgumentException("Amount and currency are required");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency;
    }
    
    public static Money of(double amount, Currency currency) {
        return new Money(BigDecimal.valueOf(amount), currency);
    }
    
    // Operations return new instances (immutable)
    public Money add(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money subtract(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    public Money multiply(int multiplier) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(multiplier)), this.currency);
    }
    
    public boolean isGreaterThan(Money other) {
        validateSameCurrency(other);
        return this.amount.compareTo(other.amount) > 0;
    }
    
    public boolean isNegative() {
        return this.amount.compareTo(BigDecimal.ZERO) < 0;
    }
    
    private void validateSameCurrency(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot operate on different currencies");
        }
    }
    
    // Getters
    public BigDecimal getAmount() { return amount; }
    public Currency getCurrency() { return currency; }
    
    // Equality based on ALL attributes
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return amount.equals(money.amount) && currency.equals(money.currency);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
    
    @Override
    public String toString() {
        return currency.getSymbol() + amount;
    }
}

// Address value object
public final class Address {
    private final String street;
    private final String city;
    private final String state;
    private final String postalCode;
    private final String country;
    
    public Address(String street, String city, String state, 
                   String postalCode, String country) {
        this.street = Objects.requireNonNull(street);
        this.city = Objects.requireNonNull(city);
        this.state = state;
        this.postalCode = Objects.requireNonNull(postalCode);
        this.country = Objects.requireNonNull(country);
    }
    
    // "Modification" returns new instance
    public Address withStreet(String newStreet) {
        return new Address(newStreet, city, state, postalCode, country);
    }
    
    public String getFullAddress() {
        return String.format("%s, %s, %s %s, %s", 
            street, city, state, postalCode, country);
    }
    
    // Getters, equals(), hashCode()...
}

// Email value object with validation
public final class Email {
    private static final Pattern EMAIL_PATTERN = 
        Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$");
    
    private final String value;
    
    public Email(String value) {
        if (value == null || !EMAIL_PATTERN.matcher(value).matches()) {
            throw new InvalidEmailException(value);
        }
        this.value = value.toLowerCase();
    }
    
    public String getValue() { return value; }
    
    public String getDomain() {
        return value.substring(value.indexOf('@') + 1);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Email email = (Email) o;
        return value.equals(email.value);
    }
    
    @Override
    public int hashCode() { return value.hashCode(); }
    
    @Override
    public String toString() { return value; }
}
```

---

## 3. Aggregates and Aggregate Roots

### What is an Aggregate?

An **aggregate** is a cluster of domain objects that are treated as a single unit for data changes. The **aggregate root** is the entry point to the aggregate.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              AGGREGATE PATTERN                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐           │
│   │                    ORDER AGGREGATE                                  │           │
│   │  ┌─────────────────────────────────────────────────────────────┐   │           │
│   │  │                  Order (Aggregate Root)                     │   │           │
│   │  │  - orderId (identity)                                       │   │           │
│   │  │  - customerId (reference to another aggregate)              │   │           │
│   │  │  - status                                                   │   │           │
│   │  │  - totalAmount                                              │   │           │
│   │  └─────────────────────────────┬───────────────────────────────┘   │           │
│   │                                │                                   │           │
│   │                    ┌───────────┴───────────┐                       │           │
│   │                    │                       │                       │           │
│   │  ┌─────────────────┴──────┐  ┌─────────────┴──────────┐            │           │
│   │  │      OrderItem         │  │     ShippingAddress    │            │           │
│   │  │  - product info        │  │  (Value Object)        │            │           │
│   │  │  - quantity            │  └────────────────────────┘            │           │
│   │  │  - price               │                                        │           │
│   │  └────────────────────────┘                                        │           │
│   └─────────────────────────────────────────────────────────────────────┘           │
│                                                                                     │
│   RULES:                                                                            │
│   1. External objects reference aggregate by root's ID only                         │
│   2. All changes go through the aggregate root                                      │
│   3. Aggregate root ensures invariants                                              │
│   4. Delete root → delete all children                                              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementing Aggregates

```java
// Aggregate Root
public class Order extends Entity<OrderId> {
    private CustomerId customerId;  // Reference to another aggregate
    private List<OrderItem> items;
    private OrderStatus status;
    private Money totalAmount;
    private Address shippingAddress;
    private LocalDateTime createdAt;
    
    // Private constructor
    private Order(OrderId id, CustomerId customerId) {
        super(id);
        this.customerId = customerId;
        this.items = new ArrayList<>();
        this.status = OrderStatus.DRAFT;
        this.totalAmount = Money.ZERO;
        this.createdAt = LocalDateTime.now();
    }
    
    // Factory method
    public static Order create(CustomerId customerId) {
        return new Order(OrderId.generate(), customerId);
    }
    
    // All modifications go through aggregate root
    public void addItem(ProductId productId, String productName, 
                        Money unitPrice, int quantity) {
        validateModifiable();
        
        // Check if item already exists
        Optional<OrderItem> existing = findItem(productId);
        if (existing.isPresent()) {
            existing.get().increaseQuantity(quantity);
        } else {
            items.add(new OrderItem(productId, productName, unitPrice, quantity));
        }
        
        recalculateTotal();
    }
    
    public void removeItem(ProductId productId) {
        validateModifiable();
        items.removeIf(item -> item.getProductId().equals(productId));
        recalculateTotal();
    }
    
    public void updateItemQuantity(ProductId productId, int newQuantity) {
        validateModifiable();
        OrderItem item = findItem(productId)
            .orElseThrow(() -> new ItemNotFoundException(productId));
        
        if (newQuantity <= 0) {
            removeItem(productId);
        } else {
            item.setQuantity(newQuantity);
            recalculateTotal();
        }
    }
    
    public void setShippingAddress(Address address) {
        validateModifiable();
        this.shippingAddress = Objects.requireNonNull(address);
    }
    
    // State transitions
    public void confirm() {
        validateCanConfirm();
        this.status = OrderStatus.CONFIRMED;
        // Could raise domain event here
    }
    
    public void ship() {
        if (status != OrderStatus.CONFIRMED) {
            throw new InvalidStateException("Can only ship confirmed orders");
        }
        this.status = OrderStatus.SHIPPED;
    }
    
    public void deliver() {
        if (status != OrderStatus.SHIPPED) {
            throw new InvalidStateException("Can only deliver shipped orders");
        }
        this.status = OrderStatus.DELIVERED;
    }
    
    public void cancel() {
        if (status == OrderStatus.DELIVERED) {
            throw new InvalidStateException("Cannot cancel delivered order");
        }
        this.status = OrderStatus.CANCELLED;
    }
    
    // Invariant validation
    private void validateModifiable() {
        if (status != OrderStatus.DRAFT) {
            throw new InvalidStateException("Cannot modify non-draft order");
        }
    }
    
    private void validateCanConfirm() {
        if (items.isEmpty()) {
            throw new BusinessException("Cannot confirm empty order");
        }
        if (shippingAddress == null) {
            throw new BusinessException("Shipping address required");
        }
        if (status != OrderStatus.DRAFT) {
            throw new InvalidStateException("Can only confirm draft orders");
        }
    }
    
    private void recalculateTotal() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    private Optional<OrderItem> findItem(ProductId productId) {
        return items.stream()
            .filter(item -> item.getProductId().equals(productId))
            .findFirst();
    }
    
    // Read-only access to items
    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }
    
    // Other getters...
}

// Entity within aggregate (not accessible outside)
public class OrderItem {
    private ProductId productId;
    private String productName;
    private Money unitPrice;
    private int quantity;
    
    OrderItem(ProductId productId, String productName, Money unitPrice, int quantity) {
        this.productId = productId;
        this.productName = productName;
        this.unitPrice = unitPrice;
        this.quantity = quantity;
    }
    
    void increaseQuantity(int amount) {
        this.quantity += amount;
    }
    
    void setQuantity(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        this.quantity = quantity;
    }
    
    public Money getSubtotal() {
        return unitPrice.multiply(quantity);
    }
    
    // Getters...
}
```

---

## 4. Domain Boundaries

### Bounded Contexts

Different parts of a system may have different models for the same concept.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            BOUNDED CONTEXTS                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   E-Commerce System:                                                                │
│                                                                                     │
│   ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐      │
│   │   Sales Context      │  │  Shipping Context    │  │  Billing Context     │      │
│   ├──────────────────────┤  ├──────────────────────┤  ├──────────────────────┤      │
│   │                      │  │                      │  │                      │      │
│   │  Product:            │  │  Product:            │  │  Product:            │      │
│   │  - name              │  │  - weight            │  │  - price             │      │
│   │  - price             │  │  - dimensions        │  │  - taxCategory       │      │
│   │  - description       │  │  - isFragile         │  │  - discount          │      │
│   │  - images            │  │  - shippingClass     │  │                      │      │
│   │                      │  │                      │  │                      │      │
│   │  Customer:           │  │  Customer:           │  │  Customer:           │      │
│   │  - name              │  │  - address           │  │  - billingAddress    │      │
│   │  - preferences       │  │  - phone             │  │  - paymentMethods    │      │
│   │                      │  │  - deliveryNotes     │  │  - creditLimit       │      │
│   │                      │  │                      │  │                      │      │
│   └──────────────────────┘  └──────────────────────┘  └──────────────────────┘      │
│                                                                                     │
│   Same concept (Product, Customer) has different meaning in each context            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Avoiding Anemic Models

### Anemic vs Rich Domain Model

```java
// ❌ ANEMIC MODEL - Just data, no behavior
public class Order {
    private String id;
    private String customerId;
    private List<OrderItem> items;
    private String status;
    private double total;
    
    // Only getters and setters - NO business logic
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    // ... more getters/setters
}

// Business logic in service (procedural style)
public class OrderService {
    public void addItem(Order order, OrderItem item) {
        if (!"DRAFT".equals(order.getStatus())) {
            throw new Exception("Cannot modify");
        }
        order.getItems().add(item);
        recalculateTotal(order);
    }
    
    public void confirm(Order order) {
        if (order.getItems().isEmpty()) {
            throw new Exception("Empty order");
        }
        order.setStatus("CONFIRMED");
    }
    
    private void recalculateTotal(Order order) {
        double total = 0;
        for (OrderItem item : order.getItems()) {
            total += item.getPrice() * item.getQuantity();
        }
        order.setTotal(total);
    }
}


// ✓ RICH DOMAIN MODEL - Data AND behavior together
public class Order extends Entity<OrderId> {
    private CustomerId customerId;
    private List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;
    private Money total;
    
    // Business logic IN the entity
    public void addItem(Product product, int quantity) {
        validateModifiable();
        items.add(new OrderItem(product, quantity));
        recalculateTotal();
    }
    
    public void confirm() {
        if (items.isEmpty()) {
            throw new EmptyOrderException();
        }
        status = OrderStatus.CONFIRMED;
    }
    
    private void validateModifiable() {
        if (status != OrderStatus.DRAFT) {
            throw new OrderNotModifiableException();
        }
    }
    
    private void recalculateTotal() {
        total = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    // Getters only, no setters
}
```

### Benefits of Rich Domain Model

| Aspect | Anemic | Rich |
|--------|--------|------|
| **Encapsulation** | Poor - data exposed | Good - controlled access |
| **Invariants** | Hard to enforce | Enforced by entity |
| **Testability** | Test service + entity | Test entity directly |
| **Discoverability** | Logic scattered | Logic in one place |
| **Maintainability** | Harder | Easier |

---

**Next Section: [Data Modeling & Persistence](../10-data-modeling/README.md)** →
