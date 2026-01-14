# 🏗️ Creational Design Patterns

> **Control object creation mechanisms**

Creational patterns deal with object creation, trying to create objects in a manner suitable to the situation.

---

## 📑 Table of Contents

1. [Singleton Pattern](#1-singleton-pattern)
2. [Factory Pattern](#2-factory-pattern)
3. [Abstract Factory Pattern](#3-abstract-factory-pattern)
4. [Builder Pattern](#4-builder-pattern)
5. [Prototype Pattern](#5-prototype-pattern)

---

## 1. Singleton Pattern

### Intent
Ensure a class has only **one instance** and provide a **global point of access** to it.

### When to Use
- Database connections
- Logger instances
- Configuration managers
- Thread pools
- Caches

### UML Diagram

```
┌─────────────────────────────────┐
│          Singleton              │
├─────────────────────────────────┤
│ - instance: Singleton           │
│ - data: SomeType                │
├─────────────────────────────────┤
│ - Singleton()                   │  ← Private constructor
│ + getInstance(): Singleton      │  ← Static method
│ + doSomething(): void           │
└─────────────────────────────────┘
```

### Implementation - Basic (Not Thread-Safe)

```java
// ❌ NOT THREAD-SAFE - Don't use in production
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // Private constructor
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton(); // Race condition possible!
        }
        return instance;
    }
}
```

### Implementation - Thread-Safe with Synchronized

```java
// ✓ Thread-safe but has performance overhead
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    // Synchronized ensures only one thread creates instance
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### Implementation - Double-Checked Locking (Recommended)

```java
// ✓ RECOMMENDED - Thread-safe with good performance
public class Singleton {
    // volatile ensures visibility across threads
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        // First check (no locking)
        if (instance == null) {
            synchronized (Singleton.class) {
                // Second check (with locking)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### Implementation - Bill Pugh (Best for Java)

```java
// ✓ BEST APPROACH - Uses class loading mechanism
public class Singleton {
    
    private Singleton() {}
    
    // Inner static class - loaded only when referenced
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

### Implementation - Enum (Most Robust)

```java
// ✓ MOST ROBUST - Handles serialization and reflection
public enum Singleton {
    INSTANCE;
    
    private String data;
    
    public void doSomething() {
        System.out.println("Doing something...");
    }
    
    public void setData(String data) {
        this.data = data;
    }
    
    public String getData() {
        return data;
    }
}

// Usage
Singleton.INSTANCE.doSomething();
```

### Real-World Example: Logger

```java
public class Logger {
    private static volatile Logger instance;
    private List<String> logs;
    
    private Logger() {
        logs = new ArrayList<>();
    }
    
    public static Logger getInstance() {
        if (instance == null) {
            synchronized (Logger.class) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }
    
    public void log(String message) {
        String timestamp = LocalDateTime.now().toString();
        String logEntry = timestamp + " - " + message;
        logs.add(logEntry);
        System.out.println(logEntry);
    }
    
    public List<String> getLogs() {
        return Collections.unmodifiableList(logs);
    }
}

// Usage
Logger.getInstance().log("Application started");
Logger.getInstance().log("User logged in");
```

### Python Implementation

```python
class Singleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # Initialize only once
        if not hasattr(self, 'initialized'):
            self.data = None
            self.initialized = True


# Usage
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

### ⚠️ When NOT to Use Singleton

1. **Unit Testing** - Hard to mock, creates tight coupling
2. **Parallel Processing** - Can become bottleneck
3. **Stateless Services** - Use dependency injection instead
4. **When multiple instances might be needed later**

---

## 2. Factory Pattern

### Intent
Define an interface for creating objects, but let **subclasses decide** which class to instantiate.

### When to Use
- Object creation logic is complex
- Object type determined at runtime
- Decouple object creation from usage
- Multiple objects share a common interface

### UML Diagram

```
┌────────────────────┐           ┌────────────────────┐
│    «interface»     │           │   ProductFactory   │
│      Product       │           ├────────────────────┤
├────────────────────┤           │ + create(type):    │
│ + operation()      │           │     Product        │
└─────────△──────────┘           └────────────────────┘
          │                                │
          │                                │ creates
    ┌─────┴─────┐                          │
    │           │                          ▼
┌───┴────┐ ┌────┴───┐              ┌───────────────┐
│ProductA│ │ProductB│              │    Client     │
└────────┘ └────────┘              └───────────────┘
```

### Implementation

```java
// Product interface
public interface Document {
    void open();
    void save();
    String getType();
}

// Concrete products
public class PDFDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening PDF document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving PDF document");
    }
    
    @Override
    public String getType() {
        return "PDF";
    }
}

public class WordDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening Word document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving Word document");
    }
    
    @Override
    public String getType() {
        return "WORD";
    }
}

public class ExcelDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening Excel document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving Excel document");
    }
    
    @Override
    public String getType() {
        return "EXCEL";
    }
}

// Factory
public class DocumentFactory {
    
    public static Document createDocument(String type) {
        switch (type.toUpperCase()) {
            case "PDF":
                return new PDFDocument();
            case "WORD":
                return new WordDocument();
            case "EXCEL":
                return new ExcelDocument();
            default:
                throw new IllegalArgumentException("Unknown document type: " + type);
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Document doc1 = DocumentFactory.createDocument("PDF");
        doc1.open();  // Opening PDF document
        
        Document doc2 = DocumentFactory.createDocument("WORD");
        doc2.open();  // Opening Word document
    }
}
```

### Factory with Registration (Extensible)

```java
// More flexible factory using registration
public class DocumentFactory {
    private static Map<String, Supplier<Document>> registry = new HashMap<>();
    
    static {
        // Register default types
        registry.put("PDF", PDFDocument::new);
        registry.put("WORD", WordDocument::new);
        registry.put("EXCEL", ExcelDocument::new);
    }
    
    // Allow registering new types without modifying factory
    public static void register(String type, Supplier<Document> supplier) {
        registry.put(type.toUpperCase(), supplier);
    }
    
    public static Document createDocument(String type) {
        Supplier<Document> supplier = registry.get(type.toUpperCase());
        if (supplier == null) {
            throw new IllegalArgumentException("Unknown document type: " + type);
        }
        return supplier.get();
    }
}

// Usage - Adding new type without changing factory
DocumentFactory.register("MARKDOWN", MarkdownDocument::new);
Document md = DocumentFactory.createDocument("MARKDOWN");
```

### Real-World Example: Notification Factory

```java
public interface Notification {
    void send(String message, String recipient);
}

public class EmailNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending EMAIL to " + recipient + ": " + message);
    }
}

public class SMSNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
    }
}

public class PushNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending PUSH to " + recipient + ": " + message);
    }
}

public class NotificationFactory {
    public enum NotificationType {
        EMAIL, SMS, PUSH
    }
    
    public static Notification createNotification(NotificationType type) {
        switch (type) {
            case EMAIL:
                return new EmailNotification();
            case SMS:
                return new SMSNotification();
            case PUSH:
                return new PushNotification();
            default:
                throw new IllegalArgumentException("Unknown type");
        }
    }
}

// Usage
Notification notification = NotificationFactory.createNotification(
    NotificationFactory.NotificationType.EMAIL
);
notification.send("Hello!", "user@example.com");
```

---

## 3. Abstract Factory Pattern

### Intent
Provide an interface for creating **families of related objects** without specifying concrete classes.

### When to Use
- Create families of related objects
- System should be independent of how products are created
- Need to enforce constraints between related products
- UI themes, cross-platform code

### UML Diagram

```
┌──────────────────────┐
│ «interface»          │
│  AbstractFactory     │
├──────────────────────┤
│ + createButton()     │
│ + createCheckbox()   │
│ + createTextField()  │
└──────────△───────────┘
           │
     ┌─────┴─────┐
     │           │
┌────┴─────┐ ┌───┴──────┐
│WinFactory│ │MacFactory│
└──────────┘ └──────────┘
     │             │
     │ creates     │ creates
     ▼             ▼
┌──────────┐  ┌──────────┐
│WinButton │  │MacButton │
│WinCheckbox│ │MacCheckbox│
└──────────┘  └──────────┘
```

### Implementation

```java
// Abstract Products
public interface Button {
    void render();
    void onClick();
}

public interface Checkbox {
    void render();
    boolean isChecked();
}

public interface TextField {
    void render();
    String getValue();
}

// Windows Concrete Products
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows-style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Windows button clicked");
    }
}

public class WindowsCheckbox implements Checkbox {
    private boolean checked = false;
    
    @Override
    public void render() {
        System.out.println("Rendering Windows-style checkbox");
    }
    
    @Override
    public boolean isChecked() {
        return checked;
    }
}

// Mac Concrete Products
public class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac-style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Mac button clicked");
    }
}

public class MacCheckbox implements Checkbox {
    private boolean checked = false;
    
    @Override
    public void render() {
        System.out.println("Rendering Mac-style checkbox");
    }
    
    @Override
    public boolean isChecked() {
        return checked;
    }
}

// Abstract Factory
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
    TextField createTextField();
}

// Concrete Factories
public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
    
    @Override
    public TextField createTextField() {
        return new WindowsTextField();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
    
    @Override
    public TextField createTextField() {
        return new MacTextField();
    }
}

// Client code
public class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void render() {
        button.render();
        checkbox.render();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        GUIFactory factory;
        String osName = System.getProperty("os.name").toLowerCase();
        
        if (osName.contains("windows")) {
            factory = new WindowsFactory();
        } else {
            factory = new MacFactory();
        }
        
        Application app = new Application(factory);
        app.render();
    }
}
```

### Factory vs Abstract Factory

| Aspect | Factory | Abstract Factory |
|--------|---------|------------------|
| Purpose | Create ONE product | Create FAMILY of products |
| Complexity | Simple | More complex |
| Products | Single type | Multiple related types |
| Example | DocumentFactory | GUIFactory (Button + Checkbox + ...) |

---

## 4. Builder Pattern

### Intent
Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

### When to Use
- Object has many optional parameters
- Object construction has many steps
- Want immutable objects with many fields
- Avoid telescoping constructors

### UML Diagram

```
┌──────────────────────┐      ┌───────────────────────┐
│       Director       │      │        Builder        │
├──────────────────────┤      ├───────────────────────┤
│ - builder: Builder   │─────>│ + buildPartA()        │
├──────────────────────┤      │ + buildPartB()        │
│ + construct()        │      │ + getResult(): Product│
└──────────────────────┘      └───────────△───────────┘
                                          │
                              ┌───────────┴───────────┐
                              │   ConcreteBuilder     │
                              ├───────────────────────┤
                              │ - product: Product    │
                              │ + buildPartA()        │
                              │ + buildPartB()        │
                              │ + getResult()         │
                              └───────────────────────┘
```

### Implementation - Fluent Builder (Most Common)

```java
public class User {
    // Required parameters
    private final String firstName;
    private final String lastName;
    
    // Optional parameters
    private final int age;
    private final String phone;
    private final String email;
    private final String address;
    
    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.email = builder.email;
        this.address = builder.address;
    }
    
    // Only getters - object is immutable
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
    public String getPhone() { return phone; }
    public String getEmail() { return email; }
    public String getAddress() { return address; }
    
    // Static inner Builder class
    public static class UserBuilder {
        // Required parameters
        private final String firstName;
        private final String lastName;
        
        // Optional parameters with defaults
        private int age = 0;
        private String phone = "";
        private String email = "";
        private String address = "";
        
        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public UserBuilder email(String email) {
            this.email = email;
            return this;
        }
        
        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            // Validation can go here
            return new User(this);
        }
    }
}

// Usage - Fluent interface
User user = new User.UserBuilder("John", "Doe")
    .age(30)
    .email("john@example.com")
    .phone("123-456-7890")
    .build();
```

### Real-World Example: SQL Query Builder

```java
public class SQLQueryBuilder {
    private StringBuilder query = new StringBuilder();
    private List<String> columns = new ArrayList<>();
    private String table;
    private List<String> conditions = new ArrayList<>();
    private String orderBy;
    private Integer limit;
    
    public SQLQueryBuilder select(String... columns) {
        this.columns.addAll(Arrays.asList(columns));
        return this;
    }
    
    public SQLQueryBuilder from(String table) {
        this.table = table;
        return this;
    }
    
    public SQLQueryBuilder where(String condition) {
        conditions.add(condition);
        return this;
    }
    
    public SQLQueryBuilder orderBy(String column) {
        this.orderBy = column;
        return this;
    }
    
    public SQLQueryBuilder limit(int limit) {
        this.limit = limit;
        return this;
    }
    
    public String build() {
        query.append("SELECT ");
        if (columns.isEmpty()) {
            query.append("*");
        } else {
            query.append(String.join(", ", columns));
        }
        
        query.append(" FROM ").append(table);
        
        if (!conditions.isEmpty()) {
            query.append(" WHERE ");
            query.append(String.join(" AND ", conditions));
        }
        
        if (orderBy != null) {
            query.append(" ORDER BY ").append(orderBy);
        }
        
        if (limit != null) {
            query.append(" LIMIT ").append(limit);
        }
        
        return query.toString();
    }
}

// Usage
String query = new SQLQueryBuilder()
    .select("id", "name", "email")
    .from("users")
    .where("age > 18")
    .where("status = 'active'")
    .orderBy("name")
    .limit(10)
    .build();

// Result: SELECT id, name, email FROM users WHERE age > 18 AND status = 'active' ORDER BY name LIMIT 10
```

### Real-World Example: HTTP Request Builder

```java
public class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final String body;
    private final int timeout;
    
    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = Collections.unmodifiableMap(builder.headers);
        this.body = builder.body;
        this.timeout = builder.timeout;
    }
    
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeout = 30000;
        
        public Builder(String url) {
            this.url = url;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            headers.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public Builder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }
        
        public HttpRequest build() {
            if (url == null || url.isEmpty()) {
                throw new IllegalStateException("URL is required");
            }
            return new HttpRequest(this);
        }
    }
}

// Usage
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body("{\"name\": \"John\"}")
    .timeout(5000)
    .build();
```

---

## 5. Prototype Pattern

### Intent
Create new objects by **cloning existing objects**, avoiding the cost of creating objects from scratch.

### When to Use
- Object creation is expensive
- Need copies of existing objects
- Runtime configuration
- Avoid subclasses of factory

### Implementation

```java
// Prototype interface
public interface Prototype<T> {
    T clone();
}

// Concrete prototype
public class Document implements Prototype<Document> {
    private String title;
    private String content;
    private List<String> images;
    private Map<String, String> metadata;
    
    public Document() {
        this.images = new ArrayList<>();
        this.metadata = new HashMap<>();
    }
    
    // Copy constructor for deep cloning
    public Document(Document source) {
        this.title = source.title;
        this.content = source.content;
        this.images = new ArrayList<>(source.images);
        this.metadata = new HashMap<>(source.metadata);
    }
    
    @Override
    public Document clone() {
        return new Document(this);
    }
    
    // Getters and setters
    public void setTitle(String title) { this.title = title; }
    public void setContent(String content) { this.content = content; }
    public void addImage(String image) { images.add(image); }
    public void addMetadata(String key, String value) { metadata.put(key, value); }
}

// Prototype registry
public class DocumentRegistry {
    private Map<String, Document> templates = new HashMap<>();
    
    public void registerTemplate(String name, Document doc) {
        templates.put(name, doc);
    }
    
    public Document createFromTemplate(String name) {
        Document template = templates.get(name);
        if (template == null) {
            throw new IllegalArgumentException("Template not found: " + name);
        }
        return template.clone();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Create and register templates
        Document invoiceTemplate = new Document();
        invoiceTemplate.setTitle("Invoice");
        invoiceTemplate.addMetadata("type", "invoice");
        invoiceTemplate.addImage("company-logo.png");
        
        DocumentRegistry registry = new DocumentRegistry();
        registry.registerTemplate("invoice", invoiceTemplate);
        
        // Create new documents from template
        Document invoice1 = registry.createFromTemplate("invoice");
        invoice1.setContent("Invoice #001 content");
        
        Document invoice2 = registry.createFromTemplate("invoice");
        invoice2.setContent("Invoice #002 content");
    }
}
```

---

## Summary

| Pattern | When to Use | Key Benefit |
|---------|-------------|-------------|
| **Singleton** | One instance needed | Global access point |
| **Factory** | Hide creation logic | Decouples creation from usage |
| **Abstract Factory** | Related object families | Ensures compatibility |
| **Builder** | Complex construction | Readable, flexible creation |
| **Prototype** | Clone existing objects | Avoid expensive creation |

---

**Next: [Structural Patterns →](../structural/README.md)**
