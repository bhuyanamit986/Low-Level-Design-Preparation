# 📘 Section 6: Concurrency & Thread Safety

> **Important for Senior Roles** - Understanding thread safety is crucial for building robust systems

---

## 📑 Table of Contents

1. [Thread Safety Basics](#1-thread-safety-basics)
2. [Synchronization Mechanisms](#2-synchronization-mechanisms)
3. [Immutability](#3-immutability)
4. [Common Concurrency Issues](#4-common-concurrency-issues)
5. [Thread-Safe Design Patterns](#5-thread-safe-design-patterns)
6. [Producer-Consumer Pattern](#6-producer-consumer-pattern)

---

## 1. Thread Safety Basics

### What is Thread Safety?

A piece of code is **thread-safe** if it functions correctly during simultaneous execution by multiple threads, without unintended interactions.

### Thread Safety Problems

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         THREAD SAFETY PROBLEMS                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   1. RACE CONDITION                                                                 │
│   ┌─────────────┐                ┌─────────────┐                                    │
│   │  Thread 1   │                │  Thread 2   │                                    │
│   └─────┬───────┘                └─────┬───────┘                                    │
│         │ read counter (5)             │                                            │
│         │                              │ read counter (5)                           │
│         │ counter = 5 + 1              │                                            │
│         │                              │ counter = 5 + 1                            │
│         │ write counter (6)            │                                            │
│         │                              │ write counter (6)                          │
│         ▼                              ▼                                            │
│   Expected: 7, Actual: 6 (Lost update!)                                             │
│                                                                                     │
│   2. VISIBILITY ISSUE                                                               │
│   Thread 1 updates a variable, Thread 2 doesn't see the update                      │
│   (Each thread may cache variables in CPU registers/cache)                          │
│                                                                                     │
│   3. INSTRUCTION REORDERING                                                         │
│   CPU/Compiler may reorder instructions for optimization                            │
│   Can cause unexpected behavior in multi-threaded code                              │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Simple Example: Counter Problem

```java
// ❌ NOT THREAD-SAFE
public class UnsafeCounter {
    private int count = 0;
    
    public void increment() {
        count++;  // Not atomic! Read-modify-write
    }
    
    public int getCount() {
        return count;
    }
}

// ✓ THREAD-SAFE with synchronized
public class SynchronizedCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// ✓ THREAD-SAFE with AtomicInteger
public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}

// Testing
public class Main {
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter unsafeCounter = new UnsafeCounter();
        AtomicCounter safeCounter = new AtomicCounter();
        
        int numThreads = 100;
        int incrementsPerThread = 1000;
        
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        
        for (int i = 0; i < numThreads; i++) {
            executor.submit(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    unsafeCounter.increment();
                    safeCounter.increment();
                }
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        
        System.out.println("Expected: " + (numThreads * incrementsPerThread));
        System.out.println("Unsafe counter: " + unsafeCounter.getCount());  // Usually less
        System.out.println("Safe counter: " + safeCounter.getCount());      // Always correct
    }
}
```

---

## 2. Synchronization Mechanisms

### synchronized Keyword

```java
public class BankAccount {
    private double balance;
    
    // Method-level synchronization
    public synchronized void deposit(double amount) {
        balance += amount;
    }
    
    public synchronized void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
    
    // Block-level synchronization
    public void transfer(BankAccount to, double amount) {
        synchronized (this) {
            if (this.balance >= amount) {
                this.balance -= amount;
                synchronized (to) {
                    to.balance += amount;
                }
            }
        }
    }
}
```

### ReentrantLock

```java
public class BankAccountWithLock {
    private double balance;
    private final ReentrantLock lock = new ReentrantLock();
    
    public void deposit(double amount) {
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock();  // Always unlock in finally
        }
    }
    
    public boolean tryWithdraw(double amount, long timeout, TimeUnit unit) 
            throws InterruptedException {
        // Try to acquire lock with timeout
        if (lock.tryLock(timeout, unit)) {
            try {
                if (balance >= amount) {
                    balance -= amount;
                    return true;
                }
                return false;
            } finally {
                lock.unlock();
            }
        }
        return false;  // Could not acquire lock
    }
    
    public double getBalance() {
        lock.lock();
        try {
            return balance;
        } finally {
            lock.unlock();
        }
    }
}
```

### ReadWriteLock

```java
// Optimizes for read-heavy workloads
public class Cache<K, V> {
    private final Map<K, V> cache = new HashMap<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock readLock = lock.readLock();
    private final Lock writeLock = lock.writeLock();
    
    public V get(K key) {
        readLock.lock();  // Multiple readers allowed
        try {
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }
    
    public void put(K key, V value) {
        writeLock.lock();  // Exclusive write access
        try {
            cache.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
    
    public V computeIfAbsent(K key, Function<K, V> mappingFunction) {
        // First try with read lock
        readLock.lock();
        try {
            V value = cache.get(key);
            if (value != null) {
                return value;
            }
        } finally {
            readLock.unlock();
        }
        
        // Need to write - acquire write lock
        writeLock.lock();
        try {
            // Double-check after acquiring write lock
            V value = cache.get(key);
            if (value != null) {
                return value;
            }
            value = mappingFunction.apply(key);
            cache.put(key, value);
            return value;
        } finally {
            writeLock.unlock();
        }
    }
}
```

### Comparison

| Mechanism | Pros | Cons |
|-----------|------|------|
| **synchronized** | Simple, built-in | No timeout, no try-lock |
| **ReentrantLock** | Flexible, timeout, try-lock | More verbose, must unlock |
| **ReadWriteLock** | Better for read-heavy | More complex |

---

## 3. Immutability

### Why Immutability?

Immutable objects are inherently thread-safe because:
- No shared mutable state
- No need for synchronization
- Can be safely shared between threads

### Creating Immutable Classes

```java
// Immutable class
public final class ImmutableUser {
    private final String id;
    private final String name;
    private final String email;
    private final List<String> roles;  // Need defensive copy
    
    public ImmutableUser(String id, String name, String email, List<String> roles) {
        this.id = id;
        this.name = name;
        this.email = email;
        // Defensive copy to prevent external modification
        this.roles = new ArrayList<>(roles);
    }
    
    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    
    // Return unmodifiable view
    public List<String> getRoles() {
        return Collections.unmodifiableList(roles);
    }
    
    // "Modification" creates new object
    public ImmutableUser withName(String newName) {
        return new ImmutableUser(id, newName, email, roles);
    }
    
    public ImmutableUser addRole(String role) {
        List<String> newRoles = new ArrayList<>(roles);
        newRoles.add(role);
        return new ImmutableUser(id, name, email, newRoles);
    }
}

// Using Builder for immutable objects
public final class ImmutableOrder {
    private final String orderId;
    private final String customerId;
    private final List<OrderItem> items;
    private final double total;
    private final LocalDateTime createdAt;
    
    private ImmutableOrder(Builder builder) {
        this.orderId = builder.orderId;
        this.customerId = builder.customerId;
        this.items = Collections.unmodifiableList(new ArrayList<>(builder.items));
        this.total = builder.total;
        this.createdAt = builder.createdAt;
    }
    
    // Getters only, no setters
    
    public static class Builder {
        private String orderId;
        private String customerId;
        private List<OrderItem> items = new ArrayList<>();
        private double total;
        private LocalDateTime createdAt;
        
        public Builder orderId(String orderId) {
            this.orderId = orderId;
            return this;
        }
        
        // ... other builder methods
        
        public ImmutableOrder build() {
            return new ImmutableOrder(this);
        }
    }
}
```

---

## 4. Common Concurrency Issues

### Deadlock

```java
// ❌ DEADLOCK EXAMPLE
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock1...");
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            
            synchronized (lock2) {  // Waiting for lock2
                System.out.println("Thread 1: Holding lock1 & lock2...");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: Holding lock2...");
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            
            synchronized (lock1) {  // Waiting for lock1 - DEADLOCK!
                System.out.println("Thread 2: Holding lock2 & lock1...");
            }
        }
    }
}

// ✓ FIXED - Consistent lock ordering
public class FixedDeadlock {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized (lock1) {  // Always acquire lock1 first
            synchronized (lock2) {
                System.out.println("Thread 1: Holding lock1 & lock2...");
            }
        }
    }
    
    public void method2() {
        synchronized (lock1) {  // Same order: lock1 first
            synchronized (lock2) {
                System.out.println("Thread 2: Holding lock1 & lock2...");
            }
        }
    }
}

// ✓ Using tryLock to avoid deadlock
public class TryLockExample {
    private final Lock lock1 = new ReentrantLock();
    private final Lock lock2 = new ReentrantLock();
    
    public boolean transferMoney(Account from, Account to, double amount) {
        while (true) {
            if (lock1.tryLock()) {
                try {
                    if (lock2.tryLock()) {
                        try {
                            // Perform transfer
                            return true;
                        } finally {
                            lock2.unlock();
                        }
                    }
                } finally {
                    lock1.unlock();
                }
            }
            // Back off and retry
            try { Thread.sleep(10); } catch (InterruptedException e) { return false; }
        }
    }
}
```

### Race Condition: Check-Then-Act

```java
// ❌ RACE CONDITION
public class UnsafeCheckThenAct {
    private Map<String, Object> cache = new HashMap<>();
    
    public Object get(String key) {
        if (!cache.containsKey(key)) {  // Check
            cache.put(key, computeValue(key));  // Act - Race condition!
        }
        return cache.get(key);
    }
}

// ✓ FIXED with proper synchronization
public class SafeCheckThenAct {
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    
    public Object get(String key) {
        return cache.computeIfAbsent(key, k -> computeValue(k));
    }
}
```

### Visibility Issue

```java
// ❌ VISIBILITY ISSUE
public class UnsafeFlag {
    private boolean running = true;  // Not visible across threads
    
    public void stop() {
        running = false;
    }
    
    public void run() {
        while (running) {  // May never see the update
            // Do work
        }
    }
}

// ✓ FIXED with volatile
public class SafeFlag {
    private volatile boolean running = true;  // Ensures visibility
    
    public void stop() {
        running = false;
    }
    
    public void run() {
        while (running) {
            // Do work
        }
    }
}
```

---

## 5. Thread-Safe Design Patterns

### Thread-Safe Singleton

```java
// Double-checked locking
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Initialization-on-demand holder (Recommended)
public class SingletonHolder {
    private SingletonHolder() {}
    
    private static class Holder {
        private static final SingletonHolder INSTANCE = new SingletonHolder();
    }
    
    public static SingletonHolder getInstance() {
        return Holder.INSTANCE;
    }
}

// Enum singleton (Most robust)
public enum SingletonEnum {
    INSTANCE;
    
    public void doSomething() {
        // ...
    }
}
```

### Thread-Safe Observer

```java
public class ThreadSafeEventBus {
    private final ConcurrentMap<Class<?>, CopyOnWriteArrayList<EventListener<?>>> listeners 
        = new ConcurrentHashMap<>();
    
    public <T> void subscribe(Class<T> eventType, EventListener<T> listener) {
        listeners.computeIfAbsent(eventType, k -> new CopyOnWriteArrayList<>())
                 .add(listener);
    }
    
    public <T> void unsubscribe(Class<T> eventType, EventListener<T> listener) {
        CopyOnWriteArrayList<EventListener<?>> list = listeners.get(eventType);
        if (list != null) {
            list.remove(listener);
        }
    }
    
    @SuppressWarnings("unchecked")
    public <T> void publish(T event) {
        CopyOnWriteArrayList<EventListener<?>> list = listeners.get(event.getClass());
        if (list != null) {
            for (EventListener<?> listener : list) {
                ((EventListener<T>) listener).onEvent(event);
            }
        }
    }
}
```

---

## 6. Producer-Consumer Pattern

### Using BlockingQueue

```java
public class ProducerConsumerExample {
    private final BlockingQueue<Task> queue;
    private final int numProducers;
    private final int numConsumers;
    private final ExecutorService executor;
    private volatile boolean running = true;
    
    public ProducerConsumerExample(int queueCapacity, int numProducers, int numConsumers) {
        this.queue = new ArrayBlockingQueue<>(queueCapacity);
        this.numProducers = numProducers;
        this.numConsumers = numConsumers;
        this.executor = Executors.newFixedThreadPool(numProducers + numConsumers);
    }
    
    public void start() {
        // Start producers
        for (int i = 0; i < numProducers; i++) {
            final int producerId = i;
            executor.submit(() -> produce(producerId));
        }
        
        // Start consumers
        for (int i = 0; i < numConsumers; i++) {
            final int consumerId = i;
            executor.submit(() -> consume(consumerId));
        }
    }
    
    private void produce(int producerId) {
        while (running) {
            try {
                Task task = generateTask();
                queue.put(task);  // Blocks if queue is full
                System.out.println("Producer " + producerId + " added: " + task);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    private void consume(int consumerId) {
        while (running || !queue.isEmpty()) {
            try {
                Task task = queue.poll(1, TimeUnit.SECONDS);  // Timeout to check running flag
                if (task != null) {
                    processTask(task);
                    System.out.println("Consumer " + consumerId + " processed: " + task);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    public void stop() {
        running = false;
        executor.shutdown();
        try {
            executor.awaitTermination(30, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            executor.shutdownNow();
        }
    }
    
    private Task generateTask() {
        return new Task("Task-" + System.nanoTime());
    }
    
    private void processTask(Task task) {
        // Simulate processing
        try { Thread.sleep(100); } catch (InterruptedException e) {}
    }
}
```

### Custom Bounded Buffer

```java
public class BoundedBuffer<T> {
    private final Queue<T> buffer;
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    
    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
        this.buffer = new LinkedList<>();
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (buffer.size() == capacity) {
                notFull.await();  // Wait until not full
            }
            buffer.add(item);
            notEmpty.signal();  // Signal that buffer is not empty
        } finally {
            lock.unlock();
        }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (buffer.isEmpty()) {
                notEmpty.await();  // Wait until not empty
            }
            T item = buffer.poll();
            notFull.signal();  // Signal that buffer is not full
            return item;
        } finally {
            lock.unlock();
        }
    }
    
    public int size() {
        lock.lock();
        try {
            return buffer.size();
        } finally {
            lock.unlock();
        }
    }
}
```

---

## Best Practices Summary

### Thread Safety Checklist

| Practice | Description |
|----------|-------------|
| **Minimize shared state** | Less sharing = less synchronization needed |
| **Prefer immutability** | Immutable objects are thread-safe by design |
| **Use thread-safe collections** | `ConcurrentHashMap`, `CopyOnWriteArrayList` |
| **Lock ordering** | Always acquire locks in same order |
| **Keep locks short** | Hold locks for minimum time |
| **Use `volatile` for flags** | Ensures visibility of simple flags |
| **Prefer `AtomicXxx`** | For simple counters and references |
| **Document thread safety** | Make it clear if a class is thread-safe |

### Common Thread-Safe Java Classes

```java
// Thread-safe collections
ConcurrentHashMap<K, V>   // Better than Collections.synchronizedMap
CopyOnWriteArrayList<E>   // Good for read-heavy, rarely written lists
ConcurrentLinkedQueue<E>  // Non-blocking queue
BlockingQueue<E>          // Producer-consumer

// Atomic variables
AtomicInteger
AtomicLong
AtomicReference<V>
AtomicBoolean

// Synchronizers
CountDownLatch   // Wait for multiple operations
CyclicBarrier    // Synchronization point for threads
Semaphore        // Control access to limited resources
```

---

**Next Section: [Error Handling & Validation](../07-error-handling/README.md)** →
