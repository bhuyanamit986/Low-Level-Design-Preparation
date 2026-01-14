# 🏛️ Structural Design Patterns

> **Compose objects into larger structures**

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

---

## 📑 Table of Contents

1. [Adapter Pattern](#1-adapter-pattern)
2. [Decorator Pattern](#2-decorator-pattern)
3. [Facade Pattern](#3-facade-pattern)
4. [Composite Pattern](#4-composite-pattern)
5. [Proxy Pattern](#5-proxy-pattern)

---

## 1. Adapter Pattern

### Intent
Convert the interface of a class into another interface clients expect. Allows classes with incompatible interfaces to work together.

### When to Use
- Integrate legacy code with new systems
- Use third-party libraries with different interfaces
- Create a reusable class that cooperates with unrelated classes

### UML Diagram

```
┌────────────────┐     ┌────────────────────┐     ┌─────────────────┐
│    Client      │────>│  «interface»       │     │    Adaptee      │
│                │     │     Target         │     ├─────────────────┤
└────────────────┘     ├────────────────────┤     │ + specificReq() │
                       │ + request()        │     └────────┬────────┘
                       └─────────△──────────┘              │
                                 │                         │
                       ┌─────────┴──────────┐              │
                       │     Adapter        │──────────────┘
                       ├────────────────────┤    delegates to
                       │ - adaptee: Adaptee │
                       │ + request()        │
                       └────────────────────┘
```

### Real-World Example: Payment Gateway Integration

```java
// Your application's expected interface
public interface PaymentProcessor {
    boolean processPayment(String customerId, double amount);
    boolean refund(String transactionId, double amount);
}

// Third-party payment gateway (Stripe-like)
public class StripeAPI {
    public StripePaymentResult charge(String stripeCustomerId, int amountInCents) {
        System.out.println("Stripe: Charging " + amountInCents + " cents");
        return new StripePaymentResult("txn_123", true);
    }
    
    public StripeRefundResult createRefund(String chargeId, int amountInCents) {
        System.out.println("Stripe: Refunding " + amountInCents + " cents");
        return new StripeRefundResult(true);
    }
}

// Another third-party (PayPal-like)
public class PayPalSDK {
    public PayPalResponse executePayment(String payerId, double amount, String currency) {
        System.out.println("PayPal: Executing payment of $" + amount);
        return new PayPalResponse("PAYPAL_TXN_456", "SUCCESS");
    }
    
    public PayPalResponse refundPayment(String transactionId) {
        System.out.println("PayPal: Refunding transaction " + transactionId);
        return new PayPalResponse("REFUND_789", "SUCCESS");
    }
}

// Adapter for Stripe
public class StripeAdapter implements PaymentProcessor {
    private StripeAPI stripeAPI;
    
    public StripeAdapter(StripeAPI stripeAPI) {
        this.stripeAPI = stripeAPI;
    }
    
    @Override
    public boolean processPayment(String customerId, double amount) {
        // Convert dollars to cents
        int amountInCents = (int) (amount * 100);
        StripePaymentResult result = stripeAPI.charge(customerId, amountInCents);
        return result.isSuccess();
    }
    
    @Override
    public boolean refund(String transactionId, double amount) {
        int amountInCents = (int) (amount * 100);
        StripeRefundResult result = stripeAPI.createRefund(transactionId, amountInCents);
        return result.isSuccess();
    }
}

// Adapter for PayPal
public class PayPalAdapter implements PaymentProcessor {
    private PayPalSDK payPalSDK;
    
    public PayPalAdapter(PayPalSDK payPalSDK) {
        this.payPalSDK = payPalSDK;
    }
    
    @Override
    public boolean processPayment(String customerId, double amount) {
        PayPalResponse response = payPalSDK.executePayment(customerId, amount, "USD");
        return "SUCCESS".equals(response.getStatus());
    }
    
    @Override
    public boolean refund(String transactionId, double amount) {
        PayPalResponse response = payPalSDK.refundPayment(transactionId);
        return "SUCCESS".equals(response.getStatus());
    }
}

// Client code - works with any payment processor
public class OrderService {
    private PaymentProcessor paymentProcessor;
    
    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    
    public boolean checkout(String customerId, double amount) {
        return paymentProcessor.processPayment(customerId, amount);
    }
}

// Usage
OrderService orderService1 = new OrderService(new StripeAdapter(new StripeAPI()));
OrderService orderService2 = new OrderService(new PayPalAdapter(new PayPalSDK()));
```

---

## 2. Decorator Pattern

### Intent
Attach additional responsibilities to an object **dynamically**. Provides a flexible alternative to subclassing for extending functionality.

### When to Use
- Add responsibilities to objects without affecting other objects
- Responsibilities can be withdrawn
- Extension by subclassing is impractical
- Need to combine multiple behaviors

### UML Diagram

```
┌──────────────────┐
│  «interface»     │
│   Component      │
├──────────────────┤
│ + operation()    │
└───────△──────────┘
        │
   ┌────┴────────────────────────┐
   │                             │
┌──┴───────────┐     ┌───────────┴────────┐
│  Concrete    │     │    Decorator       │
│  Component   │     ├────────────────────┤
├──────────────┤     │ - component        │
│ + operation()│     │ + operation()      │
└──────────────┘     └─────────△──────────┘
                               │
                  ┌────────────┴────────────┐
                  │                         │
          ┌───────┴───────┐        ┌────────┴──────┐
          │ ConcreteDecorA│        │ConcreteDecorB │
          │ + operation() │        │+ operation()  │
          │ + addedBehav()│        │+ addedBehav() │
          └───────────────┘        └───────────────┘
```

### Classic Example: Coffee Shop

```java
// Component interface
public interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.00;
    }
}

public class Espresso implements Coffee {
    @Override
    public String getDescription() {
        return "Espresso";
    }
    
    @Override
    public double getCost() {
        return 3.00;
    }
}

// Abstract decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.50;
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.25;
    }
}

public class WhippedCreamDecorator extends CoffeeDecorator {
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Whipped Cream";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.75;
    }
}

public class CaramelDecorator extends CoffeeDecorator {
    public CaramelDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Caramel";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.60;
    }
}

// Usage - Decorators can be stacked!
public class Main {
    public static void main(String[] args) {
        // Simple coffee
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee $2.0
        
        // Coffee with milk
        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee, Milk $2.5
        
        // Coffee with milk and sugar
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee, Milk, Sugar $2.75
        
        // Fancy coffee all at once
        Coffee fancyCoffee = new WhippedCreamDecorator(
            new CaramelDecorator(
                new MilkDecorator(
                    new Espresso()
                )
            )
        );
        System.out.println(fancyCoffee.getDescription() + " $" + fancyCoffee.getCost());
        // Espresso, Milk, Caramel, Whipped Cream $4.85
    }
}
```

### Real-World Example: Input Stream Decorators

```java
// Java I/O uses decorator pattern extensively
public class Main {
    public static void main(String[] args) throws IOException {
        // BufferedInputStream decorates FileInputStream
        // DataInputStream decorates BufferedInputStream
        InputStream is = new DataInputStream(
            new BufferedInputStream(
                new FileInputStream("data.txt")
            )
        );
        
        // Custom example: Logging decorator
        InputStream loggingStream = new LoggingInputStream(
            new BufferedInputStream(
                new FileInputStream("data.txt")
            )
        );
    }
}

// Custom logging decorator
public class LoggingInputStream extends FilterInputStream {
    private static Logger logger = Logger.getLogger(LoggingInputStream.class.getName());
    
    public LoggingInputStream(InputStream in) {
        super(in);
    }
    
    @Override
    public int read() throws IOException {
        int data = super.read();
        logger.info("Read byte: " + data);
        return data;
    }
    
    @Override
    public int read(byte[] b, int off, int len) throws IOException {
        int bytesRead = super.read(b, off, len);
        logger.info("Read " + bytesRead + " bytes");
        return bytesRead;
    }
}
```

---

## 3. Facade Pattern

### Intent
Provide a **unified interface** to a set of interfaces in a subsystem. Defines a higher-level interface that makes the subsystem easier to use.

### When to Use
- Simplify complex subsystem
- Decouple clients from subsystem components
- Layer your subsystems
- Provide simple interface to complex library

### UML Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      Facade                                 │
├─────────────────────────────────────────────────────────────┤
│  + simpleOperation()                                        │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  Subsystem Classes                    │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │  │
│  │  │ ClassA  │  │ ClassB  │  │ ClassC  │  │ ClassD  │  │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
         ▲
         │ simple interface
         │
    ┌────┴────┐
    │ Client  │
    └─────────┘
```

### Real-World Example: Home Theater Facade

```java
// Complex subsystem classes
public class DVDPlayer {
    public void on() { System.out.println("DVD Player on"); }
    public void off() { System.out.println("DVD Player off"); }
    public void play(String movie) { System.out.println("Playing: " + movie); }
    public void stop() { System.out.println("DVD stopped"); }
    public void eject() { System.out.println("DVD ejected"); }
}

public class Projector {
    public void on() { System.out.println("Projector on"); }
    public void off() { System.out.println("Projector off"); }
    public void setInput(String input) { System.out.println("Projector input: " + input); }
    public void wideScreenMode() { System.out.println("Widescreen mode enabled"); }
}

public class SurroundSoundSystem {
    public void on() { System.out.println("Surround sound on"); }
    public void off() { System.out.println("Surround sound off"); }
    public void setVolume(int level) { System.out.println("Volume set to: " + level); }
    public void setSurroundMode() { System.out.println("Surround mode enabled"); }
}

public class TheaterLights {
    public void on() { System.out.println("Lights on"); }
    public void off() { System.out.println("Lights off"); }
    public void dim(int level) { System.out.println("Lights dimmed to: " + level + "%"); }
}

public class PopcornMachine {
    public void on() { System.out.println("Popcorn machine on"); }
    public void off() { System.out.println("Popcorn machine off"); }
    public void pop() { System.out.println("Popping popcorn!"); }
}

// Facade - Simplifies the complex subsystem
public class HomeTheaterFacade {
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private SurroundSoundSystem soundSystem;
    private TheaterLights lights;
    private PopcornMachine popcorn;
    
    public HomeTheaterFacade(
            DVDPlayer dvdPlayer, 
            Projector projector,
            SurroundSoundSystem soundSystem, 
            TheaterLights lights,
            PopcornMachine popcorn) {
        this.dvdPlayer = dvdPlayer;
        this.projector = projector;
        this.soundSystem = soundSystem;
        this.lights = lights;
        this.popcorn = popcorn;
    }
    
    // Simple method that orchestrates complex operations
    public void watchMovie(String movie) {
        System.out.println("\n=== Getting ready to watch: " + movie + " ===\n");
        
        popcorn.on();
        popcorn.pop();
        
        lights.dim(10);
        
        projector.on();
        projector.setInput("DVD");
        projector.wideScreenMode();
        
        soundSystem.on();
        soundSystem.setVolume(50);
        soundSystem.setSurroundMode();
        
        dvdPlayer.on();
        dvdPlayer.play(movie);
    }
    
    public void endMovie() {
        System.out.println("\n=== Shutting down theater ===\n");
        
        dvdPlayer.stop();
        dvdPlayer.eject();
        dvdPlayer.off();
        
        soundSystem.off();
        projector.off();
        lights.on();
        popcorn.off();
    }
}

// Client - Uses the simple facade interface
public class Main {
    public static void main(String[] args) {
        // Setup subsystem components
        DVDPlayer dvd = new DVDPlayer();
        Projector projector = new Projector();
        SurroundSoundSystem sound = new SurroundSoundSystem();
        TheaterLights lights = new TheaterLights();
        PopcornMachine popcorn = new PopcornMachine();
        
        // Create facade
        HomeTheaterFacade theater = new HomeTheaterFacade(
            dvd, projector, sound, lights, popcorn
        );
        
        // Simple interface - one method call instead of many
        theater.watchMovie("Inception");
        
        // ... watch movie ...
        
        theater.endMovie();
    }
}
```

### Real-World Example: Order Processing Facade

```java
// Complex subsystem
public class InventoryService {
    public boolean checkStock(String productId, int quantity) { /* ... */ }
    public void reserveStock(String productId, int quantity) { /* ... */ }
    public void releaseStock(String productId, int quantity) { /* ... */ }
}

public class PaymentService {
    public String processPayment(String customerId, double amount) { /* ... */ }
    public void refundPayment(String transactionId) { /* ... */ }
}

public class ShippingService {
    public String createShipment(String orderId, String address) { /* ... */ }
    public void cancelShipment(String shipmentId) { /* ... */ }
}

public class NotificationService {
    public void sendOrderConfirmation(String email, String orderId) { /* ... */ }
    public void sendShippingNotification(String email, String trackingId) { /* ... */ }
}

// Facade
public class OrderFacade {
    private InventoryService inventory;
    private PaymentService payment;
    private ShippingService shipping;
    private NotificationService notification;
    
    public OrderFacade() {
        this.inventory = new InventoryService();
        this.payment = new PaymentService();
        this.shipping = new ShippingService();
        this.notification = new NotificationService();
    }
    
    // Simple interface for complex order processing
    public OrderResult placeOrder(OrderRequest request) {
        try {
            // Check inventory
            if (!inventory.checkStock(request.getProductId(), request.getQuantity())) {
                return OrderResult.failure("Out of stock");
            }
            
            // Reserve stock
            inventory.reserveStock(request.getProductId(), request.getQuantity());
            
            // Process payment
            String transactionId = payment.processPayment(
                request.getCustomerId(), 
                request.getTotalAmount()
            );
            
            if (transactionId == null) {
                inventory.releaseStock(request.getProductId(), request.getQuantity());
                return OrderResult.failure("Payment failed");
            }
            
            // Create shipment
            String shipmentId = shipping.createShipment(
                request.getOrderId(), 
                request.getShippingAddress()
            );
            
            // Send notifications
            notification.sendOrderConfirmation(
                request.getCustomerEmail(), 
                request.getOrderId()
            );
            
            return OrderResult.success(request.getOrderId(), transactionId, shipmentId);
            
        } catch (Exception e) {
            return OrderResult.failure("Order processing failed: " + e.getMessage());
        }
    }
}

// Client - Simple usage
OrderFacade orderFacade = new OrderFacade();
OrderResult result = orderFacade.placeOrder(orderRequest);
```

---

## 4. Composite Pattern

### Intent
Compose objects into **tree structures** to represent part-whole hierarchies. Lets clients treat individual objects and compositions uniformly.

### When to Use
- Represent part-whole hierarchies
- Clients should ignore difference between compositions and individual objects
- Tree structures (file systems, UI components, organizations)

### UML Diagram

```
                    ┌────────────────────┐
                    │    «interface»     │
                    │     Component      │
                    ├────────────────────┤
                    │ + operation()      │
                    │ + add(Component)   │
                    │ + remove(Component)│
                    │ + getChild(int)    │
                    └─────────△──────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
     ┌────────┴────────┐            ┌─────────┴────────┐
     │      Leaf       │            │    Composite     │
     ├─────────────────┤            ├──────────────────┤
     │ + operation()   │            │ - children: List │
     └─────────────────┘            │ + operation()    │
                                    │ + add(Component) │
                                    │ + remove()       │
                                    └──────────────────┘
```

### Real-World Example: File System

```java
// Component interface
public interface FileSystemItem {
    String getName();
    long getSize();
    void display(String indent);
}

// Leaf - File
public class File implements FileSystemItem {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        return size;
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + " bytes)");
    }
}

// Composite - Directory
public class Directory implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();
    
    public Directory(String name) {
        this.name = name;
    }
    
    public void add(FileSystemItem item) {
        children.add(item);
    }
    
    public void remove(FileSystemItem item) {
        children.remove(item);
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        // Sum of all children's sizes
        return children.stream()
            .mapToLong(FileSystemItem::getSize)
            .sum();
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📁 " + name + " (" + getSize() + " bytes)");
        for (FileSystemItem child : children) {
            child.display(indent + "  ");
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Create file structure
        Directory root = new Directory("root");
        
        Directory documents = new Directory("documents");
        documents.add(new File("resume.pdf", 1024));
        documents.add(new File("cover_letter.docx", 512));
        
        Directory photos = new Directory("photos");
        Directory vacation = new Directory("vacation");
        vacation.add(new File("beach.jpg", 2048));
        vacation.add(new File("sunset.jpg", 1536));
        photos.add(vacation);
        photos.add(new File("profile.png", 768));
        
        root.add(documents);
        root.add(photos);
        root.add(new File("readme.txt", 256));
        
        // Display tree - treats files and directories uniformly
        root.display("");
        
        System.out.println("\nTotal size: " + root.getSize() + " bytes");
    }
}

/* Output:
📁 root (6144 bytes)
  📁 documents (1536 bytes)
    📄 resume.pdf (1024 bytes)
    📄 cover_letter.docx (512 bytes)
  📁 photos (4352 bytes)
    📁 vacation (3584 bytes)
      📄 beach.jpg (2048 bytes)
      📄 sunset.jpg (1536 bytes)
    📄 profile.png (768 bytes)
  📄 readme.txt (256 bytes)

Total size: 6144 bytes
*/
```

### Real-World Example: Organization Hierarchy

```java
public interface Employee {
    String getName();
    double getSalary();
    void showDetails(String indent);
}

// Leaf - Individual contributor
public class Developer implements Employee {
    private String name;
    private double salary;
    
    public Developer(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public double getSalary() { return salary; }
    
    @Override
    public void showDetails(String indent) {
        System.out.println(indent + "👤 " + name + " (Developer) - $" + salary);
    }
}

// Composite - Manager with team
public class Manager implements Employee {
    private String name;
    private double salary;
    private List<Employee> team = new ArrayList<>();
    
    public Manager(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public void addTeamMember(Employee employee) {
        team.add(employee);
    }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public double getSalary() {
        // Manager's salary + team salaries
        return salary + team.stream()
            .mapToDouble(Employee::getSalary)
            .sum();
    }
    
    @Override
    public void showDetails(String indent) {
        System.out.println(indent + "👔 " + name + " (Manager) - $" + salary);
        for (Employee member : team) {
            member.showDetails(indent + "  ");
        }
    }
}
```

---

## 5. Proxy Pattern

### Intent
Provide a **surrogate or placeholder** for another object to control access to it.

### Types of Proxies
- **Virtual Proxy**: Lazy initialization, load expensive objects on demand
- **Protection Proxy**: Access control
- **Remote Proxy**: Local representative for remote object
- **Caching Proxy**: Cache results of expensive operations

### UML Diagram

```
┌────────────────┐       ┌────────────────────┐
│    Client      │──────>│   «interface»      │
└────────────────┘       │     Subject        │
                         ├────────────────────┤
                         │ + request()        │
                         └─────────△──────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
          ┌─────────┴──────────┐       ┌──────────┴────────┐
          │    RealSubject     │       │       Proxy       │
          ├────────────────────┤       ├───────────────────┤
          │ + request()        │<──────│ - realSubject     │
          └────────────────────┘       │ + request()       │
                                       └───────────────────┘
```

### Virtual Proxy Example: Image Loading

```java
// Subject interface
public interface Image {
    void display();
    int getWidth();
    int getHeight();
}

// Real subject - expensive to create
public class RealImage implements Image {
    private String filename;
    private byte[] imageData;
    private int width;
    private int height;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();  // Expensive operation
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
        // Simulate expensive loading
        try {
            Thread.sleep(2000);  // 2 seconds
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        this.width = 1920;
        this.height = 1080;
        this.imageData = new byte[width * height * 4];
        System.out.println("Image loaded: " + filename);
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
    
    @Override
    public int getWidth() { return width; }
    
    @Override
    public int getHeight() { return height; }
}

// Virtual Proxy - lazy loading
public class ImageProxy implements Image {
    private String filename;
    private RealImage realImage;
    private int width;
    private int height;
    
    public ImageProxy(String filename, int width, int height) {
        this.filename = filename;
        // Store dimensions without loading image
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void display() {
        // Load only when needed
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
    
    @Override
    public int getWidth() {
        return width;  // Return cached value
    }
    
    @Override
    public int getHeight() {
        return height;  // Return cached value
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Create proxy - fast, no loading
        Image image1 = new ImageProxy("photo1.jpg", 1920, 1080);
        Image image2 = new ImageProxy("photo2.jpg", 1280, 720);
        Image image3 = new ImageProxy("photo3.jpg", 3840, 2160);
        
        // Dimensions available immediately
        System.out.println("Image 1 size: " + image1.getWidth() + "x" + image1.getHeight());
        System.out.println("Image 2 size: " + image2.getWidth() + "x" + image2.getHeight());
        
        // Image loaded only when displayed
        System.out.println("\nDisplaying image 1:");
        image1.display();
        
        // image2 and image3 never loaded if not displayed
    }
}
```

### Protection Proxy Example: Access Control

```java
public interface Document {
    void read();
    void write(String content);
    void delete();
}

public class RealDocument implements Document {
    private String name;
    private String content;
    
    public RealDocument(String name) {
        this.name = name;
        this.content = "";
    }
    
    @Override
    public void read() {
        System.out.println("Reading document: " + name);
        System.out.println("Content: " + content);
    }
    
    @Override
    public void write(String content) {
        this.content = content;
        System.out.println("Writing to document: " + name);
    }
    
    @Override
    public void delete() {
        System.out.println("Deleting document: " + name);
    }
}

// Protection proxy with role-based access
public class ProtectedDocument implements Document {
    private RealDocument document;
    private User user;
    
    public ProtectedDocument(String name, User user) {
        this.document = new RealDocument(name);
        this.user = user;
    }
    
    @Override
    public void read() {
        if (user.hasPermission("READ")) {
            document.read();
        } else {
            System.out.println("Access denied: No read permission");
        }
    }
    
    @Override
    public void write(String content) {
        if (user.hasPermission("WRITE")) {
            document.write(content);
        } else {
            System.out.println("Access denied: No write permission");
        }
    }
    
    @Override
    public void delete() {
        if (user.hasPermission("DELETE")) {
            document.delete();
        } else {
            System.out.println("Access denied: No delete permission");
        }
    }
}

// Usage
User admin = new User("admin", Set.of("READ", "WRITE", "DELETE"));
User viewer = new User("viewer", Set.of("READ"));

Document doc1 = new ProtectedDocument("secret.txt", admin);
doc1.write("Secret content");  // Works
doc1.read();  // Works

Document doc2 = new ProtectedDocument("secret.txt", viewer);
doc2.read();   // Works
doc2.write("Hacked!");  // Access denied
```

### Caching Proxy Example

```java
public interface DataService {
    Data fetchData(String id);
}

public class RemoteDataService implements DataService {
    @Override
    public Data fetchData(String id) {
        System.out.println("Fetching data from remote server: " + id);
        // Simulate network call
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return new Data(id, "Data for " + id);
    }
}

public class CachingProxy implements DataService {
    private DataService realService;
    private Map<String, Data> cache = new ConcurrentHashMap<>();
    private Map<String, Long> cacheTime = new ConcurrentHashMap<>();
    private long cacheDuration = 60000; // 1 minute
    
    public CachingProxy(DataService realService) {
        this.realService = realService;
    }
    
    @Override
    public Data fetchData(String id) {
        // Check cache
        if (cache.containsKey(id) && !isExpired(id)) {
            System.out.println("Cache hit for: " + id);
            return cache.get(id);
        }
        
        // Cache miss - fetch and cache
        System.out.println("Cache miss for: " + id);
        Data data = realService.fetchData(id);
        cache.put(id, data);
        cacheTime.put(id, System.currentTimeMillis());
        return data;
    }
    
    private boolean isExpired(String id) {
        Long time = cacheTime.get(id);
        return time == null || System.currentTimeMillis() - time > cacheDuration;
    }
}
```

---

## Summary

| Pattern | Intent | Key Use Case |
|---------|--------|--------------|
| **Adapter** | Convert interface | Legacy integration |
| **Decorator** | Add behavior | Coffee toppings, I/O streams |
| **Facade** | Simplify interface | Home theater, order processing |
| **Composite** | Tree structure | File systems, organizations |
| **Proxy** | Control access | Lazy loading, caching, security |

---

**Next: [Behavioral Patterns →](../behavioral/README.md)**
