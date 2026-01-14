# 📘 Section 3: UML & Design Artifacts

> **You don't need perfect UML, but clarity matters**

UML (Unified Modeling Language) diagrams help communicate your design during interviews. Focus on conveying ideas clearly, not drawing perfect diagrams.

---

## 📑 Table of Contents

1. [Class Diagrams](#1-class-diagrams)
2. [Sequence Diagrams](#2-sequence-diagrams)
3. [State Diagrams](#3-state-diagrams)
4. [Use Case Diagrams](#4-use-case-diagrams)
5. [Interview Tips for UML](#5-interview-tips-for-uml)

---

## 1. Class Diagrams

Class diagrams show the **static structure** of a system - classes, attributes, methods, and relationships.

### Basic Class Notation

```
┌─────────────────────────────────────────────────────────────┐
│                       CLASS NOTATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────────────────────┐                      │
│   │          ClassName               │ ← Class Name         │
│   ├──────────────────────────────────┤                      │
│   │ - privateAttribute: Type         │                      │
│   │ # protectedAttribute: Type       │ ← Attributes         │
│   │ + publicAttribute: Type          │                      │
│   ├──────────────────────────────────┤                      │
│   │ + publicMethod(): ReturnType     │                      │
│   │ - privateMethod(): void          │ ← Methods            │
│   │ # protectedMethod(param: Type)   │                      │
│   └──────────────────────────────────┘                      │
│                                                             │
│   Visibility Symbols:                                       │
│   + public                                                  │
│   - private                                                 │
│   # protected                                               │
│   ~ package/default                                         │
│                                                             │
│   Stereotypes:                                              │
│   «interface»  - Interface                                  │
│   «abstract»   - Abstract class                             │
│   «enum»       - Enumeration                                │
└─────────────────────────────────────────────────────────────┘
```

### Example: E-Commerce Order System

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            E-COMMERCE CLASS DIAGRAM                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│    ┌──────────────────────┐           ┌──────────────────────┐                      │
│    │       «enum»         │           │      «interface»     │                      │
│    │     OrderStatus      │           │    PaymentMethod     │                      │
│    ├──────────────────────┤           ├──────────────────────┤                      │
│    │ PENDING              │           │ + pay(amount): bool  │                      │
│    │ CONFIRMED            │           │ + refund(): bool     │                      │
│    │ SHIPPED              │           └──────────┬───────────┘                      │
│    │ DELIVERED            │                      │                                  │
│    │ CANCELLED            │           ┌──────────┴───────────┐                      │
│    └──────────────────────┘           │                      │                      │
│                                       ▼                      ▼                      │
│                            ┌──────────────────┐  ┌──────────────────┐               │
│                            │   CreditCard     │  │     PayPal       │               │
│                            ├──────────────────┤  ├──────────────────┤               │
│                            │ - cardNumber     │  │ - email          │               │
│                            │ - expiryDate     │  │ - authToken      │               │
│                            │ + pay(): bool    │  │ + pay(): bool    │               │
│                            │ + refund(): bool │  │ + refund(): bool │               │
│                            └──────────────────┘  └──────────────────┘               │
│                                                                                     │
│    ┌────────────────────────┐        ┌────────────────────────┐                     │
│    │        Customer        │        │        Product         │                     │
│    ├────────────────────────┤        ├────────────────────────┤                     │
│    │ - id: String           │        │ - id: String           │                     │
│    │ - name: String         │        │ - name: String         │                     │
│    │ - email: String        │        │ - price: double        │                     │
│    │ - address: Address     │        │ - stock: int           │                     │
│    ├────────────────────────┤        ├────────────────────────┤                     │
│    │ + placeOrder(): Order  │        │ + isAvailable(): bool  │                     │
│    │ + getOrders(): List    │        │ + reduceStock(qty)     │                     │
│    └───────────┬────────────┘        └────────────┬───────────┘                     │
│                │                                  │                                 │
│                │ 1        places        *         │                                 │
│                └──────────────┬───────────────────┘                                 │
│                               │                                                     │
│                               ▼                                                     │
│                  ┌────────────────────────┐                                         │
│                  │         Order          │                                         │
│                  ├────────────────────────┤                                         │
│                  │ - id: String           │                                         │
│                  │ - customer: Customer   │                                         │
│                  │ - items: List<Item>    │                                         │
│                  │ - status: OrderStatus  │                                         │
│                  │ - totalAmount: double  │                                         │
│                  ├────────────────────────┤                                         │
│                  │ + addItem(item)        │                                         │
│                  │ + removeItem(item)     │                                         │
│                  │ + calculateTotal()     │                                         │
│                  │ + checkout(payment)    │                                         │
│                  └────────────┬───────────┘                                         │
│                               │                                                     │
│                               │ 1..*                                                │
│                               ▼                                                     │
│                  ┌────────────────────────┐                                         │
│                  │       OrderItem        │                                         │
│                  ├────────────────────────┤                                         │
│                  │ - product: Product     │                                         │
│                  │ - quantity: int        │                                         │
│                  │ - unitPrice: double    │                                         │
│                  ├────────────────────────┤                                         │
│                  │ + getSubtotal(): double│                                         │
│                  └────────────────────────┘                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Relationship Notations

```
┌─────────────────────────────────────────────────────────────┐
│                    RELATIONSHIP TYPES                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. ASSOCIATION (knows-about)                              │
│      ┌─────┐         ┌─────┐                                │
│      │  A  │─────────│  B  │     A knows B                  │
│      └─────┘         └─────┘                                │
│                                                             │
│   2. DIRECTED ASSOCIATION                                   │
│      ┌─────┐         ┌─────┐                                │
│      │  A  │────────>│  B  │     A knows B (not vice versa) │
│      └─────┘         └─────┘                                │
│                                                             │
│   3. AGGREGATION (has-a, weak)                              │
│      ┌─────┐         ┌─────┐                                │
│      │  A  │◇────────│  B  │     A has B (B can exist alone)│
│      └─────┘         └─────┘                                │
│                                                             │
│   4. COMPOSITION (has-a, strong)                            │
│      ┌─────┐         ┌─────┐                                │
│      │  A  │◆────────│  B  │     A owns B (B dies with A)   │
│      └─────┘         └─────┘                                │
│                                                             │
│   5. INHERITANCE (is-a)                                     │
│      ┌─────┐                                                │
│      │  A  │                     B inherits from A          │
│      └──△──┘                                                │
│         │                                                   │
│      ┌──┴──┐                                                │
│      │  B  │                                                │
│      └─────┘                                                │
│                                                             │
│   6. IMPLEMENTATION (realizes)                              │
│      ┌─────────────┐                                        │
│      │«interface» │                                         │
│      │     A      │             B implements interface A    │
│      └─────△------┘                                         │
│            :                                                │
│      ┌─────┴─────┐                                          │
│      │     B     │                                          │
│      └───────────┘                                          │
│                                                             │
│   7. DEPENDENCY (uses)                                      │
│      ┌─────┐         ┌─────┐                                │
│      │  A  │- - - - >│  B  │     A uses B temporarily       │
│      └─────┘         └─────┘                                │
│                                                             │
│   Multiplicity:                                             │
│   1     - exactly one                                       │
│   0..1  - zero or one                                       │
│   *     - zero or more                                      │
│   1..*  - one or more                                       │
│   n..m  - specific range                                    │
└─────────────────────────────────────────────────────────────┘
```

### Code from Class Diagram

```java
// From the E-Commerce diagram above

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

public interface PaymentMethod {
    boolean pay(double amount);
    boolean refund(double amount);
}

public class CreditCard implements PaymentMethod {
    private String cardNumber;
    private String expiryDate;
    
    @Override
    public boolean pay(double amount) {
        // Process payment
        return true;
    }
    
    @Override
    public boolean refund(double amount) {
        // Process refund
        return true;
    }
}

public class Customer {
    private String id;
    private String name;
    private String email;
    private Address address;
    private List<Order> orders = new ArrayList<>();
    
    public Order placeOrder() {
        Order order = new Order(this);
        orders.add(order);
        return order;
    }
    
    public List<Order> getOrders() {
        return Collections.unmodifiableList(orders);
    }
}

public class Product {
    private String id;
    private String name;
    private double price;
    private int stock;
    
    public boolean isAvailable() {
        return stock > 0;
    }
    
    public void reduceStock(int quantity) {
        if (quantity > stock) {
            throw new IllegalStateException("Not enough stock");
        }
        stock -= quantity;
    }
}

public class Order {
    private String id;
    private Customer customer;
    private List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;
    private double totalAmount;
    
    public Order(Customer customer) {
        this.id = UUID.randomUUID().toString();
        this.customer = customer;
        this.status = OrderStatus.PENDING;
    }
    
    public void addItem(Product product, int quantity) {
        items.add(new OrderItem(product, quantity, product.getPrice()));
        calculateTotal();
    }
    
    public void calculateTotal() {
        this.totalAmount = items.stream()
            .mapToDouble(OrderItem::getSubtotal)
            .sum();
    }
    
    public boolean checkout(PaymentMethod payment) {
        if (payment.pay(totalAmount)) {
            this.status = OrderStatus.CONFIRMED;
            return true;
        }
        return false;
    }
}

public class OrderItem {
    private Product product;
    private int quantity;
    private double unitPrice;
    
    public OrderItem(Product product, int quantity, double unitPrice) {
        this.product = product;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }
    
    public double getSubtotal() {
        return unitPrice * quantity;
    }
}
```

---

## 2. Sequence Diagrams

Sequence diagrams show the **dynamic behavior** - how objects interact over time.

### Basic Notation

```
┌─────────────────────────────────────────────────────────────┐
│                 SEQUENCE DIAGRAM NOTATION                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───────┐          ┌───────┐          ┌───────┐          │
│   │ Actor │          │ObjectA│          │ObjectB│          │
│   └───┬───┘          └───┬───┘          └───┬───┘          │
│       │                  │                  │               │
│       │   1: message()   │                  │               │
│       │─────────────────>│                  │               │
│       │                  │  2: call()       │               │
│       │                  │─────────────────>│               │
│       │                  │                  │               │
│       │                  │  3: response     │               │
│       │                  │<─ ─ ─ ─ ─ ─ ─ ─ ─│               │
│       │   4: result      │                  │               │
│       │<─ ─ ─ ─ ─ ─ ─ ─ ─│                  │               │
│       │                  │                  │               │
│                                                             │
│   Symbols:                                                  │
│   ─────────────> Synchronous message                        │
│   - - - - - - -> Asynchronous message / Return              │
│   █████████████  Activation bar (object is active)          │
│   [condition]    Guard/condition                            │
│   loop           Loop fragment                              │
│   alt            Alternative (if-else)                      │
│   opt            Optional                                   │
└─────────────────────────────────────────────────────────────┘
```

### Example: Order Checkout Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          ORDER CHECKOUT SEQUENCE DIAGRAM                            │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌────────┐   ┌────────────┐   ┌───────┐   ┌─────────┐   ┌───────────┐   ┌───────┐ │
│  │Customer│   │    UI      │   │ Order │   │Inventory│   │  Payment  │   │ Email │ │
│  └────┬───┘   └─────┬──────┘   └───┬───┘   └────┬────┘   └─────┬─────┘   └───┬───┘ │
│       │             │              │            │               │             │     │
│       │ 1: checkout │              │            │               │             │     │
│       │────────────>│              │            │               │             │     │
│       │             │              │            │               │             │     │
│       │             │ 2: validate()│            │               │             │     │
│       │             │─────────────>│            │               │             │     │
│       │             │              │            │               │             │     │
│       │             │              │ 3: checkStock()            │             │     │
│       │             │              │───────────>│               │             │     │
│       │             │              │            │               │             │     │
│       │             │              │  available │               │             │     │
│       │             │              │<───────────│               │             │     │
│       │             │              │            │               │             │     │
│       │             │   valid      │            │               │             │     │
│       │             │<─────────────│            │               │             │     │
│       │             │              │            │               │             │     │
│       │             │───────────────────────────────────────────│             │     │
│       │             │         4: processPayment(amount)         │             │     │
│       │             │──────────────────────────────────────────>│             │     │
│       │             │                                           │             │     │
│       │             │              ┌────────────────────────────┐│             │     │
│       │             │              │ alt [payment successful]   ││             │     │
│       │             │              │                            ││             │     │
│       │             │                  payment success          ││             │     │
│       │             │<──────────────────────────────────────────│             │     │
│       │             │              │                            ││             │     │
│       │             │ 5: confirm() │                            ││             │     │
│       │             │─────────────>│                            ││             │     │
│       │             │              │                            ││             │     │
│       │             │              │ 6: reduceStock()           ││             │     │
│       │             │              │───────────>│               ││             │     │
│       │             │              │            │               ││             │     │
│       │             │              │  done      │               ││             │     │
│       │             │              │<───────────│               ││             │     │
│       │             │              │                            ││             │     │
│       │             │              │─────────────────────────────────────────>││     │
│       │             │              │         7: sendConfirmation()           ││     │
│       │             │              │────────────────────────────────────────>││     │
│       │             │              │                                         ││     │
│       │             │              │                                 sent    ││     │
│       │             │              │<────────────────────────────────────────││     │
│       │             │              │                            ││             │     │
│       │             │   confirmed  │                            ││             │     │
│       │             │<─────────────│                            ││             │     │
│       │             │              └────────────────────────────┘│             │     │
│       │             │              │ [payment failed]           ││             │     │
│       │             │              │                            ││             │     │
│       │             │   failure    │                            ││             │     │
│       │             │<──────────────────────────────────────────│             │     │
│       │             │              └────────────────────────────┘│             │     │
│       │             │              │            │               │             │     │
│       │   result    │              │            │               │             │     │
│       │<────────────│              │            │               │             │     │
│       │             │              │            │               │             │     │
│  ┌────┴───┐   ┌─────┴──────┐   ┌───┴───┐   ┌────┴────┐   ┌─────┴─────┐   ┌───┴───┐ │
│  │Customer│   │    UI      │   │ Order │   │Inventory│   │  Payment  │   │ Email │ │
│  └────────┘   └────────────┘   └───────┘   └─────────┘   └───────────┘   └───────┘ │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. State Diagrams

State diagrams show the **lifecycle** of an object through different states.

### Basic Notation

```
┌─────────────────────────────────────────────────────────────┐
│                  STATE DIAGRAM NOTATION                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ●        Initial state (filled circle)                    │
│                                                             │
│   ┌──────────────┐                                          │
│   │  StateName   │     State                                │
│   │──────────────│                                          │
│   │ entry/ act1  │     Entry action (on entering state)     │
│   │ do/ act2     │     Do action (while in state)           │
│   │ exit/ act3   │     Exit action (on leaving state)       │
│   └──────────────┘                                          │
│                                                             │
│   ────event[guard]/action────>    Transition                │
│                                                             │
│   ◉        Final state (bullseye)                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Example: Order State Machine

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              ORDER STATE DIAGRAM                                    │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                                       ●                                             │
│                                       │                                             │
│                                       │ createOrder()                               │
│                                       ▼                                             │
│                              ┌─────────────────┐                                    │
│                              │     PENDING     │                                    │
│                              │─────────────────│                                    │
│                              │ entry/          │                                    │
│                              │   reserveStock()│                                    │
│                              └────────┬────────┘                                    │
│                                       │                                             │
│                     ┌─────────────────┼─────────────────┐                           │
│                     │                 │                 │                           │
│          cancel()   │    confirm()    │                 │ timeout                   │
│                     ▼                 ▼                 ▼  [after 30min]            │
│              ┌─────────────┐  ┌─────────────────┐  ┌─────────────┐                  │
│              │  CANCELLED  │  │   CONFIRMED     │  │   EXPIRED   │                  │
│              │─────────────│  │─────────────────│  │─────────────│                  │
│              │ entry/      │  │ entry/          │  │ entry/      │                  │
│              │  releaseStock│ │  chargePayment()│  │  releaseStock│                 │
│              │  refund()   │  └────────┬────────┘  │  notify()   │                  │
│              └──────┬──────┘           │           └──────┬──────┘                  │
│                     │                  │ ship()           │                         │
│                     │                  ▼                  │                         │
│                     │         ┌─────────────────┐         │                         │
│                     │         │    SHIPPED      │         │                         │
│                     │         │─────────────────│         │                         │
│                     │         │ entry/          │         │                         │
│                     │         │   sendTracking()│         │                         │
│                     │         └────────┬────────┘         │                         │
│                     │                  │                  │                         │
│                     │                  │ deliver()        │                         │
│                     │                  ▼                  │                         │
│                     │         ┌─────────────────┐         │                         │
│                     │         │   DELIVERED     │         │                         │
│                     │         │─────────────────│         │                         │
│                     │         │ entry/          │         │                         │
│                     │         │  sendConfirm()  │         │                         │
│                     │         └────────┬────────┘         │                         │
│                     │                  │                  │                         │
│                     │                  │                  │                         │
│                     ▼                  ▼                  ▼                         │
│                     ◉                  ◉                  ◉                         │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### State Pattern Implementation

```java
// State interface
public interface OrderState {
    void confirm(Order order);
    void cancel(Order order);
    void ship(Order order);
    void deliver(Order order);
}

// Concrete states
public class PendingState implements OrderState {
    @Override
    public void confirm(Order order) {
        order.chargePayment();
        order.setState(new ConfirmedState());
    }
    
    @Override
    public void cancel(Order order) {
        order.releaseStock();
        order.refund();
        order.setState(new CancelledState());
    }
    
    @Override
    public void ship(Order order) {
        throw new IllegalStateException("Cannot ship pending order");
    }
    
    @Override
    public void deliver(Order order) {
        throw new IllegalStateException("Cannot deliver pending order");
    }
}

public class ConfirmedState implements OrderState {
    @Override
    public void confirm(Order order) {
        throw new IllegalStateException("Order already confirmed");
    }
    
    @Override
    public void cancel(Order order) {
        order.releaseStock();
        order.refund();
        order.setState(new CancelledState());
    }
    
    @Override
    public void ship(Order order) {
        order.sendTrackingInfo();
        order.setState(new ShippedState());
    }
    
    @Override
    public void deliver(Order order) {
        throw new IllegalStateException("Must ship before delivery");
    }
}

// Order class using state pattern
public class Order {
    private String id;
    private OrderState state;
    
    public Order() {
        this.id = UUID.randomUUID().toString();
        this.state = new PendingState();
    }
    
    public void setState(OrderState state) {
        this.state = state;
    }
    
    public void confirm() { state.confirm(this); }
    public void cancel() { state.cancel(this); }
    public void ship() { state.ship(this); }
    public void deliver() { state.deliver(this); }
    
    // Actions
    public void chargePayment() { /* ... */ }
    public void releaseStock() { /* ... */ }
    public void refund() { /* ... */ }
    public void sendTrackingInfo() { /* ... */ }
}
```

---

## 4. Use Case Diagrams

Use case diagrams show **what the system does** from a user's perspective.

### Basic Notation

```
┌─────────────────────────────────────────────────────────────┐
│                 USE CASE DIAGRAM NOTATION                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Actor          Use Case           System Boundary        │
│      O            ┌────────┐         ┌─────────────────┐    │
│     /|\  ───────> │Use Case│         │ System Name     │    │
│     / \           └────────┘         │                 │    │
│                                      │  ○────────○     │    │
│                                      │                 │    │
│   Relationships:                     └─────────────────┘    │
│                                                             │
│   ─────────────>  Association (actor uses case)             │
│                                                             │
│   - - «include»-> Includes another use case (mandatory)     │
│                                                             │
│   - - «extend» -> Extends use case (optional behavior)      │
│                                                             │
│   ─────────────▷  Generalization (inheritance)              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Example: E-Commerce Use Case Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           E-COMMERCE USE CASE DIAGRAM                               │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────────────┐   │
│   │                        E-Commerce System                                    │   │
│   │                                                                             │   │
│   │    ┌──────────────┐         ┌──────────────┐         ┌──────────────┐       │   │
│   │    │ Browse       │         │  Search      │         │  View        │       │   │
│   │    │ Products     │         │  Products    │         │  Product     │       │   │
│   │    └──────┬───────┘         └──────────────┘         └──────────────┘       │   │
│   │           │                        ▲                        ▲               │   │
│   │           │                        │                        │               │   │
│   │    O      │                        │                        │               │   │
│   │   /|\─────┴────────────────────────┴────────────────────────┘               │   │
│   │   / \                                                                       │   │
│   │ Guest                                                                       │   │
│   │                                                                             │   │
│   │                                                                             │   │
│   │    ┌──────────────┐                                                         │   │
│   │    │   Login      │◁────────────────────────────┐                           │   │
│   │    └──────┬───────┘                             │                           │   │
│   │           │                                     │                           │   │
│   │           │«include»                            │                           │   │
│   │           ▼                                     │                           │   │
│   │    ┌──────────────┐         ┌──────────────┐    │                           │   │
│   │    │  Manage      │         │   Place      │────┘                           │   │
│   │    │  Cart        │         │   Order      │                                │   │
│   │    └──────┬───────┘         └──────┬───────┘                                │   │
│   │           │                        │                                        │   │
│   │           │                        │«include»                               │   │
│   │           │                        ▼                                        │   │
│   │    O      │                 ┌──────────────┐                                │   │
│   │   /|\─────┴─────────────────│   Process    │                                │   │
│   │   / \                       │   Payment    │                                │   │
│   │ Customer                    └──────┬───────┘                                │   │
│   │                                    │                                        │   │
│   │                                    │«extend»                                │   │
│   │                                    ▼                                        │   │
│   │                             ┌──────────────┐                                │   │
│   │                             │  Apply       │                                │   │
│   │                             │  Coupon      │                                │   │
│   │                             └──────────────┘                                │   │
│   │                                                                             │   │
│   │    ┌──────────────┐         ┌──────────────┐         ┌──────────────┐       │   │
│   │    │  Manage      │         │  View        │         │  Generate    │       │   │
│   │    │  Products    │         │  Orders      │         │  Reports     │       │   │
│   │    └──────┬───────┘         └──────┬───────┘         └──────┬───────┘       │   │
│   │           │                        │                        │               │   │
│   │    O      │                        │                        │               │   │
│   │   /|\─────┴────────────────────────┴────────────────────────┘               │   │
│   │   / \                                                                       │   │
│   │  Admin                                                                      │   │
│   │                                                                             │   │
│   └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Interview Tips for UML

### What Interviewers Want to See

| Aspect | Good | Bad |
|--------|------|-----|
| **Class Names** | Descriptive nouns (`OrderService`) | Vague names (`Manager`, `Handler`) |
| **Methods** | Verb phrases (`calculateTotal()`) | Unclear (`process()`, `doIt()`) |
| **Relationships** | Correct usage | Everything is association |
| **Abstraction** | Use interfaces appropriately | Concrete everywhere |
| **Multiplicity** | Clear cardinality | Missing or wrong |

### Interview UML Checklist

```
□ Identify all entities/classes
□ Define key attributes
□ Define key methods
□ Show relationships (inheritance, composition, etc.)
□ Add multiplicity (1, *, 0..1, 1..*)
□ Use interfaces where appropriate
□ Show enums for fixed values
□ Keep it simple - don't over-engineer
```

### Quick UML Drawing Tips

1. **Start with nouns** → Classes
2. **Verbs** → Methods or relationships
3. **"has a"** → Aggregation/Composition
4. **"is a"** → Inheritance
5. **"uses"** → Dependency/Association

### Common Interview Patterns in UML

```
Pattern 1: Strategy
┌──────────────┐       ┌────────────────┐
│   Context    │◇─────>│ «interface»    │
│              │       │   Strategy     │
│ +execute()   │       │ +algorithm()   │
└──────────────┘       └───────┬────────┘
                               △
                               │
               ┌───────────────┼───────────────┐
               │               │               │
      ┌────────┴───────┐ ┌─────┴──────┐ ┌──────┴─────┐
      │ ConcreteStratA │ │ConcreteB   │ │ConcreteC   │
      └────────────────┘ └────────────┘ └────────────┘

Pattern 2: Observer
┌──────────────┐       ┌────────────────┐
│  «interface» │       │  «interface»   │
│   Subject    │       │   Observer     │
│ +attach()    │──────>│ +update()      │
│ +detach()    │       └───────┬────────┘
│ +notify()    │               △
└──────────────┘               │
                       ┌───────┴────────┐
                       │ConcreteObserver│
                       └────────────────┘

Pattern 3: Factory
┌──────────────┐       ┌────────────────┐
│   Client     │       │  «interface»   │
│              │──────>│    Product     │
└──────┬───────┘       └───────┬────────┘
       │                       △
       ▼                       │
┌──────────────┐       ┌───────┴────────┐
│   Factory    │.......│ConcreteProduct │
│ +create()    │       └────────────────┘
└──────────────┘
```

---

## 📚 Key Takeaways

1. **Class Diagrams**: Focus on entities, relationships, and responsibilities
2. **Sequence Diagrams**: Show object interactions for key flows
3. **State Diagrams**: Essential for objects with lifecycle
4. **Use Case Diagrams**: Good for requirement clarification
5. **Keep it Simple**: Clarity > Perfection in interviews

---

**Next Section: [Design Patterns](../04-design-patterns/README.md)** →
