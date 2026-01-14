# 📘 Section 11: State & Workflow Design

> **Managing complex state transitions and workflows**

---

## 📑 Table of Contents

1. [State Machines](#1-state-machines)
2. [State Transitions](#2-state-transitions)
3. [Idempotent Operations](#3-idempotent-operations)
4. [Handling Retries and Partial Failures](#4-handling-retries-and-partial-failures)

---

## 1. State Machines

### Finite State Machine (FSM)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           ORDER STATE MACHINE                                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                              ┌─────────┐                                            │
│                    ┌────────>│ PENDING │<────────┐                                  │
│                    │         └────┬────┘         │                                  │
│               timeout│             │confirm      │                                  │
│                    │             ▼               │                                  │
│              ┌─────┴─────┐  ┌─────────┐          │                                  │
│              │  EXPIRED  │  │CONFIRMED│──────────┘ cancel                           │
│              └───────────┘  └────┬────┘                                             │
│                                  │ship                                              │
│                                  ▼                                                  │
│                            ┌─────────┐                                              │
│              cancel        │ SHIPPED │                                              │
│                ┌──────────>└────┬────┘                                              │
│                │                │deliver                                            │
│                │                ▼                                                   │
│         ┌──────┴─────┐    ┌──────────┐                                              │
│         │ CANCELLED  │    │DELIVERED │                                              │
│         └────────────┘    └──────────┘                                              │
│                                                                                     │
│   States: PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED, EXPIRED                │
│   Events: confirm, ship, deliver, cancel, timeout                                   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementation with State Pattern

```java
// State interface
public interface OrderState {
    void confirm(OrderContext context);
    void ship(OrderContext context);
    void deliver(OrderContext context);
    void cancel(OrderContext context);
    String getStateName();
}

// Context that holds current state
public class OrderContext {
    private OrderState currentState;
    private Order order;
    
    public OrderContext(Order order) {
        this.order = order;
        this.currentState = createStateFromOrder(order);
    }
    
    public void setState(OrderState state) {
        this.currentState = state;
        order.setStatus(OrderStatus.valueOf(state.getStateName()));
    }
    
    public void confirm() { currentState.confirm(this); }
    public void ship() { currentState.ship(this); }
    public void deliver() { currentState.deliver(this); }
    public void cancel() { currentState.cancel(this); }
}

// Concrete states
public class PendingState implements OrderState {
    @Override
    public void confirm(OrderContext context) {
        context.getOrder().validateCanConfirm();
        context.setState(new ConfirmedState());
    }
    
    @Override
    public void ship(OrderContext context) {
        throw new InvalidStateTransitionException("Cannot ship pending order");
    }
    
    @Override
    public void deliver(OrderContext context) {
        throw new InvalidStateTransitionException("Cannot deliver pending order");
    }
    
    @Override
    public void cancel(OrderContext context) {
        context.getOrder().releaseInventory();
        context.setState(new CancelledState());
    }
    
    @Override
    public String getStateName() { return "PENDING"; }
}

public class ConfirmedState implements OrderState {
    @Override
    public void confirm(OrderContext context) {
        throw new InvalidStateTransitionException("Order already confirmed");
    }
    
    @Override
    public void ship(OrderContext context) {
        context.getOrder().createShipment();
        context.setState(new ShippedState());
    }
    
    @Override
    public void deliver(OrderContext context) {
        throw new InvalidStateTransitionException("Must ship before delivery");
    }
    
    @Override
    public void cancel(OrderContext context) {
        context.getOrder().refundPayment();
        context.getOrder().releaseInventory();
        context.setState(new CancelledState());
    }
    
    @Override
    public String getStateName() { return "CONFIRMED"; }
}
```

### Implementation with Enum and Transition Table

```java
public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED,
    EXPIRED;
    
    // Define valid transitions
    private static final Map<OrderStatus, Set<OrderStatus>> TRANSITIONS = Map.of(
        PENDING, Set.of(CONFIRMED, CANCELLED, EXPIRED),
        CONFIRMED, Set.of(SHIPPED, CANCELLED),
        SHIPPED, Set.of(DELIVERED),
        DELIVERED, Set.of(),
        CANCELLED, Set.of(),
        EXPIRED, Set.of()
    );
    
    public boolean canTransitionTo(OrderStatus target) {
        return TRANSITIONS.getOrDefault(this, Set.of()).contains(target);
    }
    
    public OrderStatus transitionTo(OrderStatus target) {
        if (!canTransitionTo(target)) {
            throw new InvalidStateTransitionException(
                "Cannot transition from " + this + " to " + target
            );
        }
        return target;
    }
}

// Usage in Order entity
public class Order {
    private OrderStatus status = OrderStatus.PENDING;
    
    public void confirm() {
        status = status.transitionTo(OrderStatus.CONFIRMED);
    }
    
    public void ship() {
        status = status.transitionTo(OrderStatus.SHIPPED);
    }
}
```

---

## 2. State Transitions

### Guarded Transitions

```java
public class OrderStateMachine {
    private final Map<Transition, Guard> guards = new HashMap<>();
    private final Map<Transition, Action> actions = new HashMap<>();
    
    public OrderStateMachine() {
        // Define guards (conditions that must be true)
        guards.put(new Transition(PENDING, CONFIRMED), 
            order -> !order.getItems().isEmpty() && order.getShippingAddress() != null);
        
        guards.put(new Transition(CONFIRMED, SHIPPED),
            order -> order.hasPayment() && order.getInventoryReserved());
        
        // Define actions (side effects on transition)
        actions.put(new Transition(PENDING, CONFIRMED),
            order -> {
                order.reserveInventory();
                order.chargePayment();
            });
        
        actions.put(new Transition(CONFIRMED, CANCELLED),
            order -> {
                order.releaseInventory();
                order.refundPayment();
            });
    }
    
    public void transition(Order order, OrderStatus targetStatus) {
        Transition transition = new Transition(order.getStatus(), targetStatus);
        
        // Check guard
        Guard guard = guards.get(transition);
        if (guard != null && !guard.isSatisfied(order)) {
            throw new TransitionGuardFailedException(
                "Guard condition not met for " + transition
            );
        }
        
        // Execute action
        Action action = actions.get(transition);
        if (action != null) {
            action.execute(order);
        }
        
        // Update state
        order.setStatus(targetStatus);
    }
}

@FunctionalInterface
interface Guard {
    boolean isSatisfied(Order order);
}

@FunctionalInterface
interface Action {
    void execute(Order order);
}

class Transition {
    private final OrderStatus from;
    private final OrderStatus to;
    // equals, hashCode
}
```

---

## 3. Idempotent Operations

### Making State Transitions Idempotent

```java
public class IdempotentOrderService {
    private final OrderRepository orderRepository;
    private final IdempotencyKeyStore idempotencyStore;
    
    @Transactional
    public Order confirmOrder(String orderId, String idempotencyKey) {
        // Check if already processed
        if (idempotencyStore.exists(idempotencyKey)) {
            return orderRepository.findById(orderId)
                .orElseThrow(() -> new OrderNotFoundException(orderId));
        }
        
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        
        // Idempotent transition - already in target state is OK
        if (order.getStatus() == OrderStatus.CONFIRMED) {
            return order;  // Already confirmed, return success
        }
        
        if (order.getStatus() != OrderStatus.PENDING) {
            throw new InvalidStateException(
                "Cannot confirm order in status: " + order.getStatus()
            );
        }
        
        order.confirm();
        orderRepository.save(order);
        idempotencyStore.store(idempotencyKey, order.getId());
        
        return order;
    }
}
```

---

## 4. Handling Retries and Partial Failures

### Saga Pattern for Distributed Transactions

```java
// Saga orchestrator for order processing
public class OrderSaga {
    private final List<SagaStep> steps;
    private final List<CompensatingAction> compensations = new ArrayList<>();
    
    public OrderSaga() {
        steps = List.of(
            new ReserveInventoryStep(),
            new ProcessPaymentStep(),
            new CreateShipmentStep(),
            new SendNotificationStep()
        );
    }
    
    public SagaResult execute(Order order) {
        for (SagaStep step : steps) {
            try {
                step.execute(order);
                compensations.add(step.getCompensation(order));
            } catch (Exception e) {
                // Rollback all completed steps
                rollback();
                return SagaResult.failed(e);
            }
        }
        return SagaResult.success();
    }
    
    private void rollback() {
        // Execute compensations in reverse order
        Collections.reverse(compensations);
        for (CompensatingAction compensation : compensations) {
            try {
                compensation.execute();
            } catch (Exception e) {
                // Log and continue - compensation failures need manual intervention
                log.error("Compensation failed", e);
            }
        }
    }
}

// Saga step interface
public interface SagaStep {
    void execute(Order order);
    CompensatingAction getCompensation(Order order);
}

public class ReserveInventoryStep implements SagaStep {
    @Override
    public void execute(Order order) {
        inventoryService.reserve(order.getItems());
    }
    
    @Override
    public CompensatingAction getCompensation(Order order) {
        return () -> inventoryService.release(order.getItems());
    }
}

public class ProcessPaymentStep implements SagaStep {
    @Override
    public void execute(Order order) {
        paymentService.charge(order.getPaymentMethod(), order.getTotal());
    }
    
    @Override
    public CompensatingAction getCompensation(Order order) {
        return () -> paymentService.refund(order.getPaymentId());
    }
}
```

---

**Next Section: [Testability & Clean Code](../12-testability/README.md)** →
