# 📘 Section 5: Core System Components (Interview Favorites)

> **These components are asked repeatedly across LLD interviews**

Understanding how to design these fundamental components demonstrates strong system design skills.

---

## 📑 Table of Contents

1. [LRU Cache](#1-lru-cache)
2. [LFU Cache](#2-lfu-cache)
3. [Rate Limiter](#3-rate-limiter)
4. [Logging Framework](#4-logging-framework)
5. [Configuration Management](#5-configuration-management)
6. [Retry Mechanism & Circuit Breaker](#6-retry-mechanism--circuit-breaker)

---

## 1. LRU Cache

### What is LRU Cache?

**Least Recently Used (LRU) Cache** evicts the least recently accessed item when the cache is full. It's based on the principle that recently used items are more likely to be used again.

### Design Requirements

- **get(key)**: Return value if exists, mark as recently used, O(1)
- **put(key, value)**: Add or update value, evict LRU if full, O(1)
- **Capacity**: Fixed maximum size

### Data Structure Choice

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              LRU CACHE STRUCTURE                                    │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   HashMap (O(1) lookup)           Doubly Linked List (O(1) remove/add)              │
│   ┌───────────────────┐           ┌──────┐     ┌──────┐     ┌──────┐                │
│   │ key1 → Node1      │           │ HEAD │ ←→  │Node1 │ ←→  │ TAIL │               │
│   │ key2 → Node2      │──────────>│(dummy)│    │      │     │(dummy)│               │
│   │ key3 → Node3      │           └──────┘     └──────┘     └──────┘                │
│   └───────────────────┘                                                             │
│                                                                                     │
│   Most Recently Used (MRU) ← ─ ─ ─ ─ ─ ─ ─ → Least Recently Used (LRU)              │
│                                                                                     │
│   On access: Move node to front (head)                                              │
│   On eviction: Remove from back (tail)                                              │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementation

```java
public class LRUCache<K, V> {
    
    // Doubly linked list node
    private class Node {
        K key;
        V value;
        Node prev;
        Node next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private final int capacity;
    private final Map<K, Node> cache;
    private final Node head;  // Dummy head (MRU side)
    private final Node tail;  // Dummy tail (LRU side)
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        
        // Initialize dummy head and tail
        head = new Node(null, null);
        tail = new Node(null, null);
        head.next = tail;
        tail.prev = head;
    }
    
    public V get(K key) {
        Node node = cache.get(key);
        if (node == null) {
            return null;
        }
        
        // Move to front (mark as recently used)
        moveToFront(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        Node existingNode = cache.get(key);
        
        if (existingNode != null) {
            // Update existing node
            existingNode.value = value;
            moveToFront(existingNode);
        } else {
            // Add new node
            Node newNode = new Node(key, value);
            
            // Evict if at capacity
            if (cache.size() >= capacity) {
                evictLRU();
            }
            
            cache.put(key, newNode);
            addToFront(newNode);
        }
    }
    
    public boolean containsKey(K key) {
        return cache.containsKey(key);
    }
    
    public int size() {
        return cache.size();
    }
    
    // Remove node from current position and add to front
    private void moveToFront(Node node) {
        removeNode(node);
        addToFront(node);
    }
    
    // Remove node from linked list
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    // Add node right after head
    private void addToFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
    
    // Remove the least recently used item (from tail)
    private void evictLRU() {
        Node lruNode = tail.prev;
        if (lruNode != head) {
            removeNode(lruNode);
            cache.remove(lruNode.key);
        }
    }
    
    // Debug: Print cache state
    public void printCache() {
        System.out.print("Cache [MRU -> LRU]: ");
        Node current = head.next;
        while (current != tail) {
            System.out.print(current.key + "=" + current.value);
            current = current.next;
            if (current != tail) System.out.print(" -> ");
        }
        System.out.println();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        LRUCache<String, Integer> cache = new LRUCache<>(3);
        
        cache.put("a", 1);
        cache.put("b", 2);
        cache.put("c", 3);
        cache.printCache();  // c=3 -> b=2 -> a=1
        
        cache.get("a");      // Access 'a', moves to front
        cache.printCache();  // a=1 -> c=3 -> b=2
        
        cache.put("d", 4);   // 'b' gets evicted (LRU)
        cache.printCache();  // d=4 -> a=1 -> c=3
        
        System.out.println(cache.get("b"));  // null (evicted)
    }
}
```

### Python Implementation

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()
    
    def get(self, key: str):
        if key not in self.cache:
            return None
        # Move to end (most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key: str, value) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # Remove first (LRU)


# Usage
cache = LRUCache(3)
cache.put("a", 1)
cache.put("b", 2)
cache.put("c", 3)
cache.get("a")       # Access 'a'
cache.put("d", 4)    # 'b' gets evicted
print(cache.get("b"))  # None
```

---

## 2. LFU Cache

### What is LFU Cache?

**Least Frequently Used (LFU) Cache** evicts the item with the lowest access frequency. If there's a tie, evict the least recently used among them.

### Implementation

```java
public class LFUCache<K, V> {
    
    private class Node {
        K key;
        V value;
        int frequency;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
            this.frequency = 1;
        }
    }
    
    private final int capacity;
    private int minFrequency;
    private final Map<K, Node> cache;
    private final Map<Integer, LinkedHashSet<K>> frequencyMap;
    
    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.minFrequency = 0;
        this.cache = new HashMap<>();
        this.frequencyMap = new HashMap<>();
    }
    
    public V get(K key) {
        Node node = cache.get(key);
        if (node == null) {
            return null;
        }
        
        updateFrequency(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        if (capacity <= 0) return;
        
        Node existingNode = cache.get(key);
        
        if (existingNode != null) {
            existingNode.value = value;
            updateFrequency(existingNode);
        } else {
            if (cache.size() >= capacity) {
                evictLFU();
            }
            
            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            
            frequencyMap.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
            minFrequency = 1;
        }
    }
    
    private void updateFrequency(Node node) {
        int oldFreq = node.frequency;
        int newFreq = oldFreq + 1;
        
        // Remove from old frequency list
        LinkedHashSet<K> oldFreqSet = frequencyMap.get(oldFreq);
        oldFreqSet.remove(node.key);
        
        // Update min frequency if needed
        if (oldFreq == minFrequency && oldFreqSet.isEmpty()) {
            minFrequency = newFreq;
        }
        
        // Add to new frequency list
        node.frequency = newFreq;
        frequencyMap.computeIfAbsent(newFreq, k -> new LinkedHashSet<>()).add(node.key);
    }
    
    private void evictLFU() {
        LinkedHashSet<K> minFreqSet = frequencyMap.get(minFrequency);
        if (minFreqSet != null && !minFreqSet.isEmpty()) {
            // Get first element (LRU among same frequency)
            K evictKey = minFreqSet.iterator().next();
            minFreqSet.remove(evictKey);
            cache.remove(evictKey);
        }
    }
}
```

---

## 3. Rate Limiter

### What is a Rate Limiter?

A rate limiter controls the rate of requests a client can make to a service. It's essential for:
- Preventing abuse
- Ensuring fair usage
- Protecting backend services

### Common Algorithms

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           RATE LIMITING ALGORITHMS                                  │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   1. TOKEN BUCKET                      2. LEAKY BUCKET                              │
│   ┌─────────────────────┐              ┌─────────────────────┐                      │
│   │ 🪙 🪙 🪙 🪙 🪙        │              │ 💧 💧 💧 💧 💧        │                      │
│   │    Token Bucket     │              │    Queue            │                      │
│   │    (capacity: 5)    │              │    (bucket)         │                      │
│   └──────────┬──────────┘              └──────────┬──────────┘                      │
│              │ consume                            │ constant                        │
│              ▼                                    ▼ rate leak                       │
│   Tokens refill at rate R              Requests processed at fixed rate             │
│   Bursty traffic OK                    Smooth output                                │
│                                                                                     │
│   3. FIXED WINDOW                      4. SLIDING WINDOW LOG                        │
│   ┌─────────────────────┐              ┌─────────────────────┐                      │
│   │ [0:00-1:00] → 100   │              │ Timestamps:         │                      │
│   │ [1:00-2:00] → 100   │              │ 12:00:15, 12:00:30  │                      │
│   └─────────────────────┘              │ 12:00:45, 12:01:10  │                      │
│   Simple, boundary issues              └─────────────────────┘                      │
│                                        Accurate, memory intensive                   │
│                                                                                     │
│   5. SLIDING WINDOW COUNTER                                                         │
│   Weighted combination of current and previous window                               │
│   Best balance of accuracy and efficiency                                           │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Token Bucket Implementation

```java
public class TokenBucketRateLimiter {
    private final int capacity;           // Maximum tokens
    private final int refillRate;         // Tokens per second
    private double currentTokens;
    private long lastRefillTime;
    
    public TokenBucketRateLimiter(int capacity, int refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.currentTokens = capacity;
        this.lastRefillTime = System.nanoTime();
    }
    
    public synchronized boolean tryAcquire() {
        return tryAcquire(1);
    }
    
    public synchronized boolean tryAcquire(int tokens) {
        refillTokens();
        
        if (currentTokens >= tokens) {
            currentTokens -= tokens;
            return true;
        }
        
        return false;
    }
    
    private void refillTokens() {
        long now = System.nanoTime();
        double elapsedSeconds = (now - lastRefillTime) / 1_000_000_000.0;
        
        double tokensToAdd = elapsedSeconds * refillRate;
        currentTokens = Math.min(capacity, currentTokens + tokensToAdd);
        lastRefillTime = now;
    }
    
    public double getAvailableTokens() {
        refillTokens();
        return currentTokens;
    }
}

// Usage
public class APIController {
    // 10 requests per second, burst of 20
    private TokenBucketRateLimiter rateLimiter = new TokenBucketRateLimiter(20, 10);
    
    public Response handleRequest(Request request) {
        if (!rateLimiter.tryAcquire()) {
            return new Response(429, "Too Many Requests");
        }
        
        // Process request
        return processRequest(request);
    }
}
```

### Sliding Window Rate Limiter

```java
public class SlidingWindowRateLimiter {
    private final int maxRequests;
    private final long windowSizeMs;
    private final Map<String, Deque<Long>> clientRequests;
    
    public SlidingWindowRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
        this.clientRequests = new ConcurrentHashMap<>();
    }
    
    public boolean tryAcquire(String clientId) {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;
        
        Deque<Long> requests = clientRequests.computeIfAbsent(
            clientId, k -> new LinkedList<>()
        );
        
        synchronized (requests) {
            // Remove expired timestamps
            while (!requests.isEmpty() && requests.peekFirst() <= windowStart) {
                requests.pollFirst();
            }
            
            // Check if within limit
            if (requests.size() < maxRequests) {
                requests.addLast(now);
                return true;
            }
            
            return false;
        }
    }
    
    public int getRemainingRequests(String clientId) {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;
        
        Deque<Long> requests = clientRequests.get(clientId);
        if (requests == null) return maxRequests;
        
        synchronized (requests) {
            while (!requests.isEmpty() && requests.peekFirst() <= windowStart) {
                requests.pollFirst();
            }
            return maxRequests - requests.size();
        }
    }
}
```

### Rate Limiter with Multiple Strategies

```java
public interface RateLimiter {
    boolean tryAcquire(String clientId);
    boolean tryAcquire(String clientId, int permits);
}

public class CompositeRateLimiter implements RateLimiter {
    private final List<RateLimiter> limiters;
    
    public CompositeRateLimiter(RateLimiter... limiters) {
        this.limiters = Arrays.asList(limiters);
    }
    
    @Override
    public boolean tryAcquire(String clientId) {
        // All limiters must allow
        return limiters.stream().allMatch(l -> l.tryAcquire(clientId));
    }
    
    @Override
    public boolean tryAcquire(String clientId, int permits) {
        return limiters.stream().allMatch(l -> l.tryAcquire(clientId, permits));
    }
}

// Usage: Combine per-second and per-minute limits
RateLimiter rateLimiter = new CompositeRateLimiter(
    new SlidingWindowRateLimiter(10, 1000),     // 10 per second
    new SlidingWindowRateLimiter(100, 60000)    // 100 per minute
);
```

---

## 4. Logging Framework

### Design a Logging Framework

```java
// Log levels
public enum LogLevel {
    TRACE(0), DEBUG(1), INFO(2), WARN(3), ERROR(4), FATAL(5);
    
    private final int severity;
    
    LogLevel(int severity) {
        this.severity = severity;
    }
    
    public int getSeverity() {
        return severity;
    }
}

// Log message
public class LogMessage {
    private final LogLevel level;
    private final String message;
    private final LocalDateTime timestamp;
    private final String loggerName;
    private final String threadName;
    private final Throwable throwable;
    
    public LogMessage(LogLevel level, String loggerName, String message, Throwable throwable) {
        this.level = level;
        this.message = message;
        this.timestamp = LocalDateTime.now();
        this.loggerName = loggerName;
        this.threadName = Thread.currentThread().getName();
        this.throwable = throwable;
    }
    
    // Getters...
}

// Appender interface (Strategy Pattern)
public interface LogAppender {
    void append(LogMessage message);
    void close();
}

// Console appender
public class ConsoleAppender implements LogAppender {
    private final LogFormatter formatter;
    
    public ConsoleAppender(LogFormatter formatter) {
        this.formatter = formatter;
    }
    
    @Override
    public void append(LogMessage message) {
        System.out.println(formatter.format(message));
    }
    
    @Override
    public void close() {}
}

// File appender
public class FileAppender implements LogAppender {
    private final String filePath;
    private final LogFormatter formatter;
    private BufferedWriter writer;
    
    public FileAppender(String filePath, LogFormatter formatter) throws IOException {
        this.filePath = filePath;
        this.formatter = formatter;
        this.writer = new BufferedWriter(new FileWriter(filePath, true));
    }
    
    @Override
    public synchronized void append(LogMessage message) {
        try {
            writer.write(formatter.format(message));
            writer.newLine();
            writer.flush();
        } catch (IOException e) {
            System.err.println("Failed to write to log file: " + e.getMessage());
        }
    }
    
    @Override
    public void close() {
        try {
            writer.close();
        } catch (IOException e) {
            System.err.println("Failed to close log file: " + e.getMessage());
        }
    }
}

// Formatter interface
public interface LogFormatter {
    String format(LogMessage message);
}

// Simple formatter
public class SimpleLogFormatter implements LogFormatter {
    private static final DateTimeFormatter TIME_FORMAT = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS");
    
    @Override
    public String format(LogMessage message) {
        StringBuilder sb = new StringBuilder();
        sb.append(message.getTimestamp().format(TIME_FORMAT))
          .append(" [").append(message.getLevel()).append("] ")
          .append("[").append(message.getThreadName()).append("] ")
          .append(message.getLoggerName()).append(" - ")
          .append(message.getMessage());
        
        if (message.getThrowable() != null) {
            sb.append("\n").append(getStackTrace(message.getThrowable()));
        }
        
        return sb.toString();
    }
    
    private String getStackTrace(Throwable throwable) {
        StringWriter sw = new StringWriter();
        throwable.printStackTrace(new PrintWriter(sw));
        return sw.toString();
    }
}

// Logger class
public class Logger {
    private final String name;
    private LogLevel level;
    private final List<LogAppender> appenders;
    
    Logger(String name, LogLevel level) {
        this.name = name;
        this.level = level;
        this.appenders = new ArrayList<>();
    }
    
    public void addAppender(LogAppender appender) {
        appenders.add(appender);
    }
    
    public void setLevel(LogLevel level) {
        this.level = level;
    }
    
    public void log(LogLevel level, String message, Throwable throwable) {
        if (level.getSeverity() < this.level.getSeverity()) {
            return;
        }
        
        LogMessage logMessage = new LogMessage(level, name, message, throwable);
        for (LogAppender appender : appenders) {
            appender.append(logMessage);
        }
    }
    
    // Convenience methods
    public void trace(String message) { log(LogLevel.TRACE, message, null); }
    public void debug(String message) { log(LogLevel.DEBUG, message, null); }
    public void info(String message) { log(LogLevel.INFO, message, null); }
    public void warn(String message) { log(LogLevel.WARN, message, null); }
    public void error(String message) { log(LogLevel.ERROR, message, null); }
    public void error(String message, Throwable t) { log(LogLevel.ERROR, message, t); }
    public void fatal(String message) { log(LogLevel.FATAL, message, null); }
}

// Logger factory (Singleton + Factory pattern)
public class LoggerFactory {
    private static final Map<String, Logger> loggers = new ConcurrentHashMap<>();
    private static LogLevel defaultLevel = LogLevel.INFO;
    private static final List<LogAppender> defaultAppenders = new ArrayList<>();
    
    static {
        // Default console appender
        defaultAppenders.add(new ConsoleAppender(new SimpleLogFormatter()));
    }
    
    public static Logger getLogger(Class<?> clazz) {
        return getLogger(clazz.getName());
    }
    
    public static Logger getLogger(String name) {
        return loggers.computeIfAbsent(name, n -> {
            Logger logger = new Logger(n, defaultLevel);
            for (LogAppender appender : defaultAppenders) {
                logger.addAppender(appender);
            }
            return logger;
        });
    }
    
    public static void setDefaultLevel(LogLevel level) {
        defaultLevel = level;
    }
    
    public static void addDefaultAppender(LogAppender appender) {
        defaultAppenders.add(appender);
    }
}

// Usage
public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    public void createUser(String username) {
        logger.info("Creating user: " + username);
        try {
            // Create user logic
            logger.debug("User created successfully");
        } catch (Exception e) {
            logger.error("Failed to create user: " + username, e);
        }
    }
}
```

---

## 5. Configuration Management

### Design a Configuration System

```java
// Configuration source interface
public interface ConfigSource {
    Map<String, String> loadProperties();
    int getPriority();  // Higher priority overrides lower
}

// File-based config source
public class FileConfigSource implements ConfigSource {
    private final String filePath;
    private final int priority;
    
    public FileConfigSource(String filePath, int priority) {
        this.filePath = filePath;
        this.priority = priority;
    }
    
    @Override
    public Map<String, String> loadProperties() {
        Map<String, String> props = new HashMap<>();
        try (InputStream is = new FileInputStream(filePath)) {
            Properties properties = new Properties();
            properties.load(is);
            for (String key : properties.stringPropertyNames()) {
                props.put(key, properties.getProperty(key));
            }
        } catch (IOException e) {
            System.err.println("Failed to load config from: " + filePath);
        }
        return props;
    }
    
    @Override
    public int getPriority() {
        return priority;
    }
}

// Environment variable config source
public class EnvironmentConfigSource implements ConfigSource {
    private final int priority;
    
    public EnvironmentConfigSource(int priority) {
        this.priority = priority;
    }
    
    @Override
    public Map<String, String> loadProperties() {
        return new HashMap<>(System.getenv());
    }
    
    @Override
    public int getPriority() {
        return priority;
    }
}

// Configuration manager
public class ConfigurationManager {
    private static volatile ConfigurationManager instance;
    private final Map<String, String> config;
    private final List<ConfigSource> sources;
    private final List<ConfigChangeListener> listeners;
    
    private ConfigurationManager() {
        this.config = new ConcurrentHashMap<>();
        this.sources = new ArrayList<>();
        this.listeners = new ArrayList<>();
    }
    
    public static ConfigurationManager getInstance() {
        if (instance == null) {
            synchronized (ConfigurationManager.class) {
                if (instance == null) {
                    instance = new ConfigurationManager();
                }
            }
        }
        return instance;
    }
    
    public void addSource(ConfigSource source) {
        sources.add(source);
        // Sort by priority (ascending, so higher priority is loaded last and overwrites)
        sources.sort(Comparator.comparingInt(ConfigSource::getPriority));
        reload();
    }
    
    public void reload() {
        Map<String, String> newConfig = new HashMap<>();
        for (ConfigSource source : sources) {
            newConfig.putAll(source.loadProperties());
        }
        
        // Detect changes
        Set<String> changedKeys = new HashSet<>();
        for (String key : newConfig.keySet()) {
            if (!newConfig.get(key).equals(config.get(key))) {
                changedKeys.add(key);
            }
        }
        
        config.clear();
        config.putAll(newConfig);
        
        // Notify listeners
        for (String key : changedKeys) {
            notifyListeners(key, config.get(key));
        }
    }
    
    public String getString(String key) {
        return config.get(key);
    }
    
    public String getString(String key, String defaultValue) {
        return config.getOrDefault(key, defaultValue);
    }
    
    public int getInt(String key, int defaultValue) {
        String value = config.get(key);
        if (value == null) return defaultValue;
        try {
            return Integer.parseInt(value);
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }
    
    public boolean getBoolean(String key, boolean defaultValue) {
        String value = config.get(key);
        if (value == null) return defaultValue;
        return Boolean.parseBoolean(value);
    }
    
    public void addChangeListener(ConfigChangeListener listener) {
        listeners.add(listener);
    }
    
    private void notifyListeners(String key, String value) {
        for (ConfigChangeListener listener : listeners) {
            listener.onConfigChange(key, value);
        }
    }
}

// Change listener interface
public interface ConfigChangeListener {
    void onConfigChange(String key, String newValue);
}

// Usage
ConfigurationManager config = ConfigurationManager.getInstance();
config.addSource(new FileConfigSource("application.properties", 1));
config.addSource(new EnvironmentConfigSource(2));  // Environment overrides file

String dbHost = config.getString("database.host", "localhost");
int dbPort = config.getInt("database.port", 5432);
```

---

## 6. Retry Mechanism & Circuit Breaker

### Retry with Exponential Backoff

```java
public class RetryTemplate {
    private final int maxRetries;
    private final long initialDelay;
    private final double multiplier;
    private final long maxDelay;
    
    public RetryTemplate(int maxRetries, long initialDelay, double multiplier, long maxDelay) {
        this.maxRetries = maxRetries;
        this.initialDelay = initialDelay;
        this.multiplier = multiplier;
        this.maxDelay = maxDelay;
    }
    
    public <T> T execute(Callable<T> operation) throws Exception {
        int attempt = 0;
        long delay = initialDelay;
        
        while (true) {
            try {
                return operation.call();
            } catch (Exception e) {
                attempt++;
                if (attempt >= maxRetries) {
                    throw e;
                }
                
                System.out.println("Attempt " + attempt + " failed, retrying in " + delay + "ms");
                Thread.sleep(delay);
                
                delay = Math.min((long) (delay * multiplier), maxDelay);
            }
        }
    }
    
    // Builder pattern
    public static class Builder {
        private int maxRetries = 3;
        private long initialDelay = 1000;
        private double multiplier = 2.0;
        private long maxDelay = 30000;
        
        public Builder maxRetries(int maxRetries) {
            this.maxRetries = maxRetries;
            return this;
        }
        
        public Builder initialDelay(long initialDelay) {
            this.initialDelay = initialDelay;
            return this;
        }
        
        public Builder multiplier(double multiplier) {
            this.multiplier = multiplier;
            return this;
        }
        
        public Builder maxDelay(long maxDelay) {
            this.maxDelay = maxDelay;
            return this;
        }
        
        public RetryTemplate build() {
            return new RetryTemplate(maxRetries, initialDelay, multiplier, maxDelay);
        }
    }
}

// Usage
RetryTemplate retry = new RetryTemplate.Builder()
    .maxRetries(3)
    .initialDelay(1000)
    .multiplier(2.0)
    .maxDelay(10000)
    .build();

String result = retry.execute(() -> externalService.call());
```

### Circuit Breaker

```java
public enum CircuitState {
    CLOSED,      // Normal operation
    OPEN,        // Failing, reject requests
    HALF_OPEN    // Testing if service recovered
}

public class CircuitBreaker {
    private final String name;
    private final int failureThreshold;
    private final long resetTimeout;
    private final int halfOpenMaxCalls;
    
    private CircuitState state = CircuitState.CLOSED;
    private int failureCount = 0;
    private int successCount = 0;
    private int halfOpenCalls = 0;
    private long lastFailureTime = 0;
    
    public CircuitBreaker(String name, int failureThreshold, long resetTimeout, int halfOpenMaxCalls) {
        this.name = name;
        this.failureThreshold = failureThreshold;
        this.resetTimeout = resetTimeout;
        this.halfOpenMaxCalls = halfOpenMaxCalls;
    }
    
    public <T> T execute(Callable<T> operation) throws Exception {
        if (!allowRequest()) {
            throw new CircuitBreakerOpenException("Circuit breaker " + name + " is OPEN");
        }
        
        try {
            T result = operation.call();
            recordSuccess();
            return result;
        } catch (Exception e) {
            recordFailure();
            throw e;
        }
    }
    
    private synchronized boolean allowRequest() {
        switch (state) {
            case CLOSED:
                return true;
                
            case OPEN:
                if (System.currentTimeMillis() - lastFailureTime >= resetTimeout) {
                    transitionTo(CircuitState.HALF_OPEN);
                    return true;
                }
                return false;
                
            case HALF_OPEN:
                if (halfOpenCalls < halfOpenMaxCalls) {
                    halfOpenCalls++;
                    return true;
                }
                return false;
                
            default:
                return false;
        }
    }
    
    private synchronized void recordSuccess() {
        switch (state) {
            case CLOSED:
                failureCount = 0;
                break;
                
            case HALF_OPEN:
                successCount++;
                if (successCount >= halfOpenMaxCalls) {
                    transitionTo(CircuitState.CLOSED);
                }
                break;
        }
    }
    
    private synchronized void recordFailure() {
        lastFailureTime = System.currentTimeMillis();
        
        switch (state) {
            case CLOSED:
                failureCount++;
                if (failureCount >= failureThreshold) {
                    transitionTo(CircuitState.OPEN);
                }
                break;
                
            case HALF_OPEN:
                transitionTo(CircuitState.OPEN);
                break;
        }
    }
    
    private void transitionTo(CircuitState newState) {
        System.out.println("Circuit breaker " + name + ": " + state + " -> " + newState);
        state = newState;
        failureCount = 0;
        successCount = 0;
        halfOpenCalls = 0;
    }
    
    public CircuitState getState() {
        return state;
    }
}

// Custom exception
public class CircuitBreakerOpenException extends RuntimeException {
    public CircuitBreakerOpenException(String message) {
        super(message);
    }
}

// Usage
CircuitBreaker circuitBreaker = new CircuitBreaker(
    "payment-service",
    5,      // Open after 5 failures
    30000,  // Try again after 30 seconds
    3       // Allow 3 test calls in half-open state
);

try {
    String result = circuitBreaker.execute(() -> paymentService.process(payment));
} catch (CircuitBreakerOpenException e) {
    // Handle gracefully - maybe use fallback
    return getFallbackResponse();
}
```

---

## Summary

| Component | Key Concept | Data Structures |
|-----------|-------------|-----------------|
| **LRU Cache** | Evict least recently used | HashMap + Doubly Linked List |
| **LFU Cache** | Evict least frequently used | HashMap + Frequency Map |
| **Rate Limiter** | Control request rate | Token Bucket, Sliding Window |
| **Logger** | Configurable logging | Strategy Pattern |
| **Config Manager** | Centralized configuration | Observer Pattern |
| **Circuit Breaker** | Prevent cascade failures | State Machine |

---

**Next Section: [Concurrency & Thread Safety](../06-concurrency/README.md)** →
