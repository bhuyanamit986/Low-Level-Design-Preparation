# 📘 Section 10: Data Modeling & Persistence

> **Bridge between domain models and storage**

---

## 📑 Table of Contents

1. [In-Memory vs Persistent Models](#1-in-memory-vs-persistent-models)
2. [Repository Pattern](#2-repository-pattern)
3. [DAO vs Repository](#3-dao-vs-repository)
4. [Transaction Boundaries](#4-transaction-boundaries)

---

## 1. In-Memory vs Persistent Models

### Separation of Concerns

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                    IN-MEMORY vs PERSISTENT MODELS                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Domain Layer                    Persistence Layer                                 │
│   (In-Memory)                     (Database)                                        │
│                                                                                     │
│   ┌──────────────────┐            ┌──────────────────┐                              │
│   │  Domain Model    │            │  Database Table  │                              │
│   │  (Rich Object)   │            │  (Normalized)    │                              │
│   ├──────────────────┤            ├──────────────────┤                              │
│   │ - orderId        │            │ - id (PK)        │                              │
│   │ - customer       │◄──────────►│ - customer_id(FK)│                              │
│   │ - items[]        │   Mapper   │ - status         │                              │
│   │ - shippingAddr   │            │ - total          │                              │
│   │ + addItem()      │            │ - created_at     │                              │
│   │ + confirm()      │            │                  │                              │
│   └──────────────────┘            └──────────────────┘                              │
│                                                                                     │
│   Object-Oriented                 Relational                                        │
│   - Behavior + Data               - Data only                                       │
│   - References                    - Foreign keys                                    │
│   - Inheritance                   - Joins                                           │
│   - Collections                   - Separate tables                                 │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Repository Pattern

### What is a Repository?

A **repository** mediates between the domain and data mapping layers, acting like an in-memory collection of domain objects.

```java
// Repository interface (domain layer)
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    List<Order> findByCustomerId(CustomerId customerId);
    List<Order> findByStatus(OrderStatus status);
    Order save(Order order);
    void delete(OrderId id);
}

// Implementation (infrastructure layer)
public class JpaOrderRepository implements OrderRepository {
    private final EntityManager entityManager;
    private final OrderMapper mapper;
    
    public JpaOrderRepository(EntityManager entityManager, OrderMapper mapper) {
        this.entityManager = entityManager;
        this.mapper = mapper;
    }
    
    @Override
    public Optional<Order> findById(OrderId id) {
        OrderEntity entity = entityManager.find(OrderEntity.class, id.getValue());
        return Optional.ofNullable(entity).map(mapper::toDomain);
    }
    
    @Override
    public List<Order> findByCustomerId(CustomerId customerId) {
        String jpql = "SELECT o FROM OrderEntity o WHERE o.customerId = :customerId";
        List<OrderEntity> entities = entityManager
            .createQuery(jpql, OrderEntity.class)
            .setParameter("customerId", customerId.getValue())
            .getResultList();
        return entities.stream().map(mapper::toDomain).collect(toList());
    }
    
    @Override
    public Order save(Order order) {
        OrderEntity entity = mapper.toEntity(order);
        if (order.getId() == null) {
            entityManager.persist(entity);
        } else {
            entity = entityManager.merge(entity);
        }
        return mapper.toDomain(entity);
    }
    
    @Override
    public void delete(OrderId id) {
        OrderEntity entity = entityManager.find(OrderEntity.class, id.getValue());
        if (entity != null) {
            entityManager.remove(entity);
        }
    }
}

// Mapper for converting between domain and persistence
public class OrderMapper {
    public Order toDomain(OrderEntity entity) {
        Order order = Order.reconstitute(
            new OrderId(entity.getId()),
            new CustomerId(entity.getCustomerId()),
            OrderStatus.valueOf(entity.getStatus())
        );
        // Map items, address, etc.
        return order;
    }
    
    public OrderEntity toEntity(Order order) {
        OrderEntity entity = new OrderEntity();
        entity.setId(order.getId().getValue());
        entity.setCustomerId(order.getCustomerId().getValue());
        entity.setStatus(order.getStatus().name());
        // Map other fields
        return entity;
    }
}
```

### In-Memory Repository (for Testing)

```java
public class InMemoryOrderRepository implements OrderRepository {
    private final Map<OrderId, Order> storage = new ConcurrentHashMap<>();
    
    @Override
    public Optional<Order> findById(OrderId id) {
        return Optional.ofNullable(storage.get(id));
    }
    
    @Override
    public List<Order> findByCustomerId(CustomerId customerId) {
        return storage.values().stream()
            .filter(o -> o.getCustomerId().equals(customerId))
            .collect(toList());
    }
    
    @Override
    public Order save(Order order) {
        storage.put(order.getId(), order);
        return order;
    }
    
    @Override
    public void delete(OrderId id) {
        storage.remove(id);
    }
}
```

---

## 3. DAO vs Repository

### Comparison

| Aspect | DAO | Repository |
|--------|-----|------------|
| **Focus** | Data access | Domain collection |
| **Returns** | Data objects | Domain objects |
| **Level** | Table-centric | Aggregate-centric |
| **Operations** | CRUD | Domain queries |
| **Abstraction** | Low (database) | High (domain) |

```java
// DAO - Table-centric, returns data objects
public interface OrderDao {
    OrderEntity findById(Long id);
    List<OrderEntity> findAll();
    Long insert(OrderEntity entity);
    void update(OrderEntity entity);
    void delete(Long id);
}

// Repository - Domain-centric, returns domain objects
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    List<Order> findPendingOrders();
    List<Order> findByCustomerAndDateRange(CustomerId customer, DateRange range);
    Order save(Order order);
}
```

---

## 4. Transaction Boundaries

### Managing Transactions

```java
// Service layer manages transactions
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    
    @Transactional
    public Order placeOrder(CreateOrderRequest request) {
        // All operations in single transaction
        Order order = Order.create(request.getCustomerId());
        
        for (OrderItemRequest item : request.getItems()) {
            inventoryService.reserve(item.getProductId(), item.getQuantity());
            order.addItem(item.getProductId(), item.getQuantity());
        }
        
        paymentService.charge(request.getPaymentMethod(), order.getTotal());
        order.confirm();
        
        return orderRepository.save(order);
        // If any step fails, entire transaction rolls back
    }
}

// Unit of Work pattern (alternative)
public interface UnitOfWork {
    void begin();
    void commit();
    void rollback();
    <T> T getRepository(Class<T> repositoryType);
}

public class OrderService {
    private final UnitOfWork unitOfWork;
    
    public Order placeOrder(CreateOrderRequest request) {
        unitOfWork.begin();
        try {
            OrderRepository orders = unitOfWork.getRepository(OrderRepository.class);
            InventoryRepository inventory = unitOfWork.getRepository(InventoryRepository.class);
            
            // ... business logic
            
            unitOfWork.commit();
            return order;
        } catch (Exception e) {
            unitOfWork.rollback();
            throw e;
        }
    }
}
```

---

**Next Section: [State & Workflow Design](../11-state-workflow/README.md)** →
