# 🎭 Behavioral Design Patterns

> **Define how objects interact and communicate**

Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects.

---

## 📑 Table of Contents

1. [Strategy Pattern](#1-strategy-pattern)
2. [Observer Pattern](#2-observer-pattern)
3. [Command Pattern](#3-command-pattern)
4. [State Pattern](#4-state-pattern)
5. [Template Method Pattern](#5-template-method-pattern)
6. [Chain of Responsibility Pattern](#6-chain-of-responsibility-pattern)
7. [Iterator Pattern](#7-iterator-pattern)

---

## 1. Strategy Pattern

### Intent
Define a family of algorithms, encapsulate each one, and make them **interchangeable**. Lets the algorithm vary independently from clients that use it.

### When to Use
- Multiple algorithms for a specific task
- Algorithm selection at runtime
- Avoid multiple conditionals
- Different variants of an algorithm

### UML Diagram

```
┌──────────────────────┐
│       Context        │
├──────────────────────┤          ┌────────────────────┐
│ - strategy: Strategy │─────────>│   «interface»      │
├──────────────────────┤          │     Strategy       │
│ + setStrategy()      │          ├────────────────────┤
│ + executeStrategy()  │          │ + execute()        │
└──────────────────────┘          └─────────△──────────┘
                                            │
                          ┌─────────────────┼─────────────────┐
                          │                 │                 │
                 ┌────────┴──────┐ ┌────────┴──────┐ ┌────────┴──────┐
                 │ ConcreteStratA│ │ ConcreteStratB│ │ ConcreteStratC│
                 │ + execute()   │ │ + execute()   │ │ + execute()   │
                 └───────────────┘ └───────────────┘ └───────────────┘
```

### Real-World Example: Payment Processing

```java
// Strategy interface
public interface PaymentStrategy {
    boolean pay(double amount);
    String getPaymentMethod();
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String cvv;
    private String expiryDate;
    
    public CreditCardPayment(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Paying $" + amount + " using Credit Card ending in " + 
            cardNumber.substring(cardNumber.length() - 4));
        // Process payment...
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "CREDIT_CARD";
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Paying $" + amount + " using PayPal account: " + email);
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "PAYPAL";
    }
}

public class CryptoPayment implements PaymentStrategy {
    private String walletAddress;
    private String cryptoType;
    
    public CryptoPayment(String walletAddress, String cryptoType) {
        this.walletAddress = walletAddress;
        this.cryptoType = cryptoType;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Paying $" + amount + " using " + cryptoType + 
            " to wallet: " + walletAddress.substring(0, 10) + "...");
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "CRYPTO";
    }
}

// Context
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
    }
    
    public boolean checkout() {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }
        double total = calculateTotal();
        return paymentStrategy.pay(total);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(new Item("Laptop", 999.99));
        cart.addItem(new Item("Mouse", 29.99));
        
        // Pay with credit card
        cart.setPaymentStrategy(new CreditCardPayment("1234567890123456", "123", "12/25"));
        cart.checkout();
        
        // Or pay with PayPal
        cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
        cart.checkout();
        
        // Or pay with crypto
        cart.setPaymentStrategy(new CryptoPayment("0x1234...abcd", "ETH"));
        cart.checkout();
    }
}
```

### Real-World Example: Compression Strategy

```java
public interface CompressionStrategy {
    byte[] compress(byte[] data);
    byte[] decompress(byte[] data);
    String getAlgorithm();
}

public class ZipCompression implements CompressionStrategy {
    @Override
    public byte[] compress(byte[] data) {
        System.out.println("Compressing using ZIP...");
        // ZIP compression logic
        return data;
    }
    
    @Override
    public byte[] decompress(byte[] data) {
        // ZIP decompression logic
        return data;
    }
    
    @Override
    public String getAlgorithm() { return "ZIP"; }
}

public class GzipCompression implements CompressionStrategy {
    @Override
    public byte[] compress(byte[] data) {
        System.out.println("Compressing using GZIP...");
        // GZIP compression logic
        return data;
    }
    
    @Override
    public byte[] decompress(byte[] data) {
        return data;
    }
    
    @Override
    public String getAlgorithm() { return "GZIP"; }
}

public class FileCompressor {
    private CompressionStrategy strategy;
    
    public void setStrategy(CompressionStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void compressFile(String filePath) {
        byte[] data = readFile(filePath);
        byte[] compressed = strategy.compress(data);
        writeFile(filePath + "." + strategy.getAlgorithm().toLowerCase(), compressed);
    }
}
```

---

## 2. Observer Pattern

### Intent
Define a **one-to-many dependency** between objects so that when one object changes state, all its dependents are notified automatically.

### When to Use
- Event handling systems
- Model-View separation
- Distributed event handling
- Implementing publish-subscribe

### UML Diagram

```
┌────────────────────────────┐           ┌─────────────────────────┐
│       «interface»          │           │      «interface»        │
│         Subject            │           │        Observer         │
├────────────────────────────┤           ├─────────────────────────┤
│ + attach(Observer)         │           │ + update(Subject)       │
│ + detach(Observer)         │──────────>│                         │
│ + notify()                 │           └───────────△─────────────┘
└────────────────────────────┘                       │
              △                            ┌─────────┴─────────┐
              │                            │                   │
┌─────────────┴────────────┐     ┌─────────┴───────┐ ┌─────────┴───────┐
│     ConcreteSubject      │     │ ConcreteObserverA│ │ConcreteObserverB│
├──────────────────────────┤     │   + update()     │ │  + update()     │
│ - state                  │     └─────────────────-┘ └─────────────────┘
│ - observers: List        │
│ + getState()             │
│ + setState()             │
└──────────────────────────┘
```

### Real-World Example: Stock Price Notification

```java
// Observer interface
public interface StockObserver {
    void update(String stockSymbol, double price);
}

// Subject interface
public interface StockSubject {
    void attach(StockObserver observer);
    void detach(StockObserver observer);
    void notifyObservers();
}

// Concrete Subject
public class Stock implements StockSubject {
    private String symbol;
    private double price;
    private List<StockObserver> observers = new ArrayList<>();
    
    public Stock(String symbol, double initialPrice) {
        this.symbol = symbol;
        this.price = initialPrice;
    }
    
    @Override
    public void attach(StockObserver observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(StockObserver observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (StockObserver observer : observers) {
            observer.update(symbol, price);
        }
    }
    
    public void setPrice(double newPrice) {
        double oldPrice = this.price;
        this.price = newPrice;
        
        if (oldPrice != newPrice) {
            System.out.println("\n" + symbol + " price changed: $" + oldPrice + " -> $" + newPrice);
            notifyObservers();
        }
    }
    
    public double getPrice() {
        return price;
    }
    
    public String getSymbol() {
        return symbol;
    }
}

// Concrete Observers
public class StockTrader implements StockObserver {
    private String name;
    private Map<String, Double> portfolio = new HashMap<>();
    
    public StockTrader(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String stockSymbol, double price) {
        System.out.println("Trader " + name + " notified: " + stockSymbol + " is now $" + price);
        
        // Trading logic
        if (portfolio.containsKey(stockSymbol)) {
            double buyPrice = portfolio.get(stockSymbol);
            if (price > buyPrice * 1.1) {
                System.out.println("  → " + name + " considering selling (10% profit)");
            } else if (price < buyPrice * 0.95) {
                System.out.println("  → " + name + " considering stop-loss (5% loss)");
            }
        }
    }
    
    public void buy(String symbol, double price) {
        portfolio.put(symbol, price);
        System.out.println(name + " bought " + symbol + " at $" + price);
    }
}

public class StockAlert implements StockObserver {
    private String stockSymbol;
    private double targetPrice;
    private boolean alertAbove;
    
    public StockAlert(String symbol, double target, boolean alertAbove) {
        this.stockSymbol = symbol;
        this.targetPrice = target;
        this.alertAbove = alertAbove;
    }
    
    @Override
    public void update(String symbol, double price) {
        if (!symbol.equals(stockSymbol)) return;
        
        if (alertAbove && price >= targetPrice) {
            System.out.println("🔔 ALERT: " + symbol + " reached $" + price + " (target: $" + targetPrice + ")");
        } else if (!alertAbove && price <= targetPrice) {
            System.out.println("🔔 ALERT: " + symbol + " dropped to $" + price + " (target: $" + targetPrice + ")");
        }
    }
}

public class StockLogger implements StockObserver {
    @Override
    public void update(String stockSymbol, double price) {
        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_TIME);
        System.out.println("[LOG " + timestamp + "] " + stockSymbol + ": $" + price);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Create stocks
        Stock appleStock = new Stock("AAPL", 150.00);
        Stock googleStock = new Stock("GOOGL", 2800.00);
        
        // Create observers
        StockTrader trader1 = new StockTrader("Alice");
        StockTrader trader2 = new StockTrader("Bob");
        StockAlert highAlert = new StockAlert("AAPL", 160.00, true);
        StockAlert lowAlert = new StockAlert("AAPL", 140.00, false);
        StockLogger logger = new StockLogger();
        
        // Register observers
        appleStock.attach(trader1);
        appleStock.attach(trader2);
        appleStock.attach(highAlert);
        appleStock.attach(lowAlert);
        appleStock.attach(logger);
        
        googleStock.attach(trader1);
        googleStock.attach(logger);
        
        // Traders buy stocks
        trader1.buy("AAPL", 150.00);
        trader2.buy("AAPL", 148.00);
        
        // Price changes - all observers notified
        appleStock.setPrice(155.00);
        appleStock.setPrice(162.00);  // Triggers high alert
        appleStock.setPrice(138.00);  // Triggers low alert
    }
}
```

### Real-World Example: Event System

```java
// Generic event system using Observer pattern
public interface EventListener<T> {
    void onEvent(T event);
}

public class EventBus {
    private Map<Class<?>, List<EventListener<?>>> listeners = new HashMap<>();
    
    public <T> void subscribe(Class<T> eventType, EventListener<T> listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>())
                 .add(listener);
    }
    
    public <T> void unsubscribe(Class<T> eventType, EventListener<T> listener) {
        List<EventListener<?>> eventListeners = listeners.get(eventType);
        if (eventListeners != null) {
            eventListeners.remove(listener);
        }
    }
    
    @SuppressWarnings("unchecked")
    public <T> void publish(T event) {
        List<EventListener<?>> eventListeners = listeners.get(event.getClass());
        if (eventListeners != null) {
            for (EventListener<?> listener : eventListeners) {
                ((EventListener<T>) listener).onEvent(event);
            }
        }
    }
}

// Events
public class UserRegisteredEvent {
    private String userId;
    private String email;
    // ...
}

public class OrderPlacedEvent {
    private String orderId;
    private double amount;
    // ...
}

// Listeners
public class WelcomeEmailListener implements EventListener<UserRegisteredEvent> {
    @Override
    public void onEvent(UserRegisteredEvent event) {
        System.out.println("Sending welcome email to: " + event.getEmail());
    }
}

public class AnalyticsListener implements EventListener<OrderPlacedEvent> {
    @Override
    public void onEvent(OrderPlacedEvent event) {
        System.out.println("Recording order analytics: " + event.getOrderId());
    }
}

// Usage
EventBus eventBus = new EventBus();
eventBus.subscribe(UserRegisteredEvent.class, new WelcomeEmailListener());
eventBus.subscribe(OrderPlacedEvent.class, new AnalyticsListener());

eventBus.publish(new UserRegisteredEvent("user1", "user@example.com"));
eventBus.publish(new OrderPlacedEvent("order1", 99.99));
```

---

## 3. Command Pattern

### Intent
Encapsulate a request as an **object**, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

### When to Use
- Implement undo/redo functionality
- Queue operations
- Log operations
- Support transactions
- Decouple sender from receiver

### UML Diagram

```
┌──────────────┐        ┌───────────────────┐
│   Invoker    │───────>│   «interface»     │
├──────────────┤        │     Command       │
│ - command    │        ├───────────────────┤
│ + execute()  │        │ + execute()       │
└──────────────┘        │ + undo()          │
                        └─────────△─────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
           ┌────────┴────────┐        ┌─────────┴───────┐
           │ConcreteCommandA │        │ ConcreteCommandB│
           ├─────────────────┤        ├─────────────────┤
           │ - receiver      │        │ - receiver      │
           │ + execute()     │        │ + execute()     │
           │ + undo()        │        │ + undo()        │
           └─────────────────┘        └─────────────────┘
                    │                          │
                    ▼                          ▼
            ┌───────────────┐          ┌───────────────┐
            │   Receiver    │          │   Receiver    │
            └───────────────┘          └───────────────┘
```

### Real-World Example: Text Editor with Undo/Redo

```java
// Command interface
public interface Command {
    void execute();
    void undo();
    String getDescription();
}

// Receiver - the actual text document
public class TextDocument {
    private StringBuilder content = new StringBuilder();
    
    public void insertText(int position, String text) {
        content.insert(position, text);
    }
    
    public void deleteText(int position, int length) {
        content.delete(position, position + length);
    }
    
    public String getContent() {
        return content.toString();
    }
    
    public int getLength() {
        return content.length();
    }
}

// Concrete commands
public class InsertTextCommand implements Command {
    private TextDocument document;
    private int position;
    private String text;
    
    public InsertTextCommand(TextDocument document, int position, String text) {
        this.document = document;
        this.position = position;
        this.text = text;
    }
    
    @Override
    public void execute() {
        document.insertText(position, text);
    }
    
    @Override
    public void undo() {
        document.deleteText(position, text.length());
    }
    
    @Override
    public String getDescription() {
        return "Insert '" + text + "' at position " + position;
    }
}

public class DeleteTextCommand implements Command {
    private TextDocument document;
    private int position;
    private int length;
    private String deletedText;  // Store for undo
    
    public DeleteTextCommand(TextDocument document, int position, int length) {
        this.document = document;
        this.position = position;
        this.length = length;
    }
    
    @Override
    public void execute() {
        // Save text before deleting for undo
        deletedText = document.getContent().substring(position, position + length);
        document.deleteText(position, length);
    }
    
    @Override
    public void undo() {
        document.insertText(position, deletedText);
    }
    
    @Override
    public String getDescription() {
        return "Delete " + length + " characters at position " + position;
    }
}

// Invoker with undo/redo support
public class TextEditor {
    private TextDocument document;
    private Stack<Command> undoStack = new Stack<>();
    private Stack<Command> redoStack = new Stack<>();
    
    public TextEditor() {
        this.document = new TextDocument();
    }
    
    public void executeCommand(Command command) {
        command.execute();
        undoStack.push(command);
        redoStack.clear();  // Clear redo stack on new command
        System.out.println("Executed: " + command.getDescription());
    }
    
    public void undo() {
        if (undoStack.isEmpty()) {
            System.out.println("Nothing to undo");
            return;
        }
        Command command = undoStack.pop();
        command.undo();
        redoStack.push(command);
        System.out.println("Undone: " + command.getDescription());
    }
    
    public void redo() {
        if (redoStack.isEmpty()) {
            System.out.println("Nothing to redo");
            return;
        }
        Command command = redoStack.pop();
        command.execute();
        undoStack.push(command);
        System.out.println("Redone: " + command.getDescription());
    }
    
    public void type(String text) {
        int position = document.getLength();
        executeCommand(new InsertTextCommand(document, position, text));
    }
    
    public void delete(int position, int length) {
        executeCommand(new DeleteTextCommand(document, position, length));
    }
    
    public String getContent() {
        return document.getContent();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        
        editor.type("Hello");
        System.out.println("Content: " + editor.getContent());  // Hello
        
        editor.type(" World");
        System.out.println("Content: " + editor.getContent());  // Hello World
        
        editor.delete(5, 6);  // Delete " World"
        System.out.println("Content: " + editor.getContent());  // Hello
        
        editor.undo();
        System.out.println("Content: " + editor.getContent());  // Hello World
        
        editor.undo();
        System.out.println("Content: " + editor.getContent());  // Hello
        
        editor.redo();
        System.out.println("Content: " + editor.getContent());  // Hello World
    }
}
```

### Real-World Example: Order Processing with Commands

```java
public interface OrderCommand {
    void execute();
    void undo();
}

public class Order {
    private String id;
    private OrderStatus status;
    private List<OrderItem> items = new ArrayList<>();
    
    // ... getters, setters
}

public class AddItemCommand implements OrderCommand {
    private Order order;
    private OrderItem item;
    
    @Override
    public void execute() {
        order.addItem(item);
    }
    
    @Override
    public void undo() {
        order.removeItem(item);
    }
}

public class ApplyDiscountCommand implements OrderCommand {
    private Order order;
    private double discount;
    private double originalTotal;
    
    @Override
    public void execute() {
        originalTotal = order.getTotal();
        order.applyDiscount(discount);
    }
    
    @Override
    public void undo() {
        order.setTotal(originalTotal);
    }
}

// Command queue for batch processing
public class OrderProcessor {
    private Queue<OrderCommand> commandQueue = new LinkedList<>();
    
    public void addCommand(OrderCommand command) {
        commandQueue.add(command);
    }
    
    public void processAll() {
        while (!commandQueue.isEmpty()) {
            OrderCommand command = commandQueue.poll();
            command.execute();
        }
    }
}
```

---

## 4. State Pattern

### Intent
Allow an object to **alter its behavior** when its internal state changes. The object will appear to change its class.

### When to Use
- Object behavior depends on state
- Operations have large conditional statements based on state
- State transitions are complex
- Implement state machines

### UML Diagram

```
┌─────────────────────────┐
│        Context          │         ┌───────────────────────┐
├─────────────────────────┤         │     «interface»       │
│ - state: State          │────────>│        State          │
├─────────────────────────┤         ├───────────────────────┤
│ + request()             │         │ + handle(Context)     │
│ + setState(State)       │         └───────────△───────────┘
└─────────────────────────┘                     │
                               ┌────────────────┼────────────────┐
                               │                │                │
                     ┌─────────┴───────┐ ┌──────┴──────┐ ┌───────┴──────┐
                     │  ConcreteStateA │ │ConcreteStateB│ │ConcreteStateC│
                     │  + handle()     │ │ + handle()   │ │ + handle()   │
                     └─────────────────┘ └──────────────┘ └──────────────┘
```

### Real-World Example: Vending Machine

```java
// State interface
public interface VendingMachineState {
    void insertCoin(VendingMachine machine);
    void selectProduct(VendingMachine machine, String product);
    void dispense(VendingMachine machine);
    void cancel(VendingMachine machine);
}

// Concrete States
public class IdleState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin inserted");
        machine.addBalance(1.00);
        machine.setState(new HasMoneyState());
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("Please insert coin first");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Please insert coin and select product");
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("Nothing to cancel");
    }
}

public class HasMoneyState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Additional coin inserted");
        machine.addBalance(1.00);
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        Product p = machine.getProduct(product);
        if (p == null) {
            System.out.println("Product not found");
            return;
        }
        if (p.getPrice() > machine.getBalance()) {
            System.out.println("Not enough money. Need $" + 
                (p.getPrice() - machine.getBalance()) + " more");
            return;
        }
        if (p.getQuantity() <= 0) {
            System.out.println("Product out of stock");
            return;
        }
        
        machine.setSelectedProduct(p);
        machine.setState(new DispensingState());
        machine.dispense();
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Please select a product first");
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("Returning $" + machine.getBalance());
        machine.refund();
        machine.setState(new IdleState());
    }
}

public class DispensingState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Please wait, dispensing product");
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("Please wait, dispensing product");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        Product product = machine.getSelectedProduct();
        double price = product.getPrice();
        
        System.out.println("Dispensing: " + product.getName());
        product.decrementQuantity();
        
        double change = machine.getBalance() - price;
        if (change > 0) {
            System.out.println("Returning change: $" + change);
        }
        
        machine.setBalance(0);
        machine.setSelectedProduct(null);
        machine.setState(new IdleState());
    }
    
    @Override
    public void cancel(VendingMachine machine) {
        System.out.println("Cannot cancel during dispensing");
    }
}

// Context
public class VendingMachine {
    private VendingMachineState state;
    private Map<String, Product> products = new HashMap<>();
    private double balance;
    private Product selectedProduct;
    
    public VendingMachine() {
        this.state = new IdleState();
        initializeProducts();
    }
    
    private void initializeProducts() {
        products.put("A1", new Product("Coca-Cola", 1.50, 10));
        products.put("A2", new Product("Pepsi", 1.50, 10));
        products.put("B1", new Product("Chips", 1.00, 5));
        products.put("B2", new Product("Chocolate", 1.25, 8));
    }
    
    // Delegate to current state
    public void insertCoin() { state.insertCoin(this); }
    public void selectProduct(String code) { state.selectProduct(this, code); }
    public void dispense() { state.dispense(this); }
    public void cancel() { state.cancel(this); }
    
    // State management
    public void setState(VendingMachineState state) {
        this.state = state;
    }
    
    // Balance management
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }
    public void addBalance(double amount) { this.balance += amount; }
    public void refund() { this.balance = 0; }
    
    // Product management
    public Product getProduct(String code) { return products.get(code); }
    public Product getSelectedProduct() { return selectedProduct; }
    public void setSelectedProduct(Product product) { this.selectedProduct = product; }
}

// Usage
public class Main {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine();
        
        machine.selectProduct("A1");  // Please insert coin first
        
        machine.insertCoin();         // Coin inserted
        machine.insertCoin();         // Additional coin inserted
        
        machine.selectProduct("A1");  // Dispensing: Coca-Cola, Returning change: $0.50
        
        machine.insertCoin();
        machine.cancel();             // Returning $1.00
    }
}
```

---

## 5. Template Method Pattern

### Intent
Define the **skeleton of an algorithm** in a method, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

### When to Use
- Implement invariant parts of algorithm once
- Common behavior among subclasses should be factored
- Control subclass extensions
- Hook operations

### Implementation

```java
// Abstract class with template method
public abstract class DataProcessor {
    
    // Template method - defines the algorithm structure
    public final void process() {
        readData();
        processData();
        if (shouldValidate()) {  // Hook method
            validateData();
        }
        writeData();
        cleanup();  // Hook method with default implementation
    }
    
    // Abstract methods - must be implemented by subclasses
    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();
    
    // Hook method - can be overridden
    protected boolean shouldValidate() {
        return true;
    }
    
    // Hook method with default implementation
    protected void validateData() {
        System.out.println("Default validation...");
    }
    
    // Hook method with default implementation
    protected void cleanup() {
        System.out.println("Default cleanup...");
    }
}

// Concrete implementation for CSV
public class CSVDataProcessor extends DataProcessor {
    private List<String[]> data;
    
    @Override
    protected void readData() {
        System.out.println("Reading CSV file...");
        // CSV reading logic
        data = new ArrayList<>();
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data...");
        // CSV processing logic
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing processed CSV...");
        // CSV writing logic
    }
}

// Concrete implementation for JSON
public class JSONDataProcessor extends DataProcessor {
    private JsonObject data;
    
    @Override
    protected void readData() {
        System.out.println("Reading JSON file...");
        // JSON reading logic
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing JSON data...");
        // JSON processing logic
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing processed JSON...");
        // JSON writing logic
    }
    
    @Override
    protected boolean shouldValidate() {
        return false;  // Skip validation for JSON
    }
    
    @Override
    protected void cleanup() {
        System.out.println("JSON-specific cleanup...");
    }
}

// Usage
DataProcessor csvProcessor = new CSVDataProcessor();
csvProcessor.process();

DataProcessor jsonProcessor = new JSONDataProcessor();
jsonProcessor.process();
```

---

## 6. Chain of Responsibility Pattern

### Intent
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along until an object handles it.

### When to Use
- Multiple objects may handle a request
- Handler isn't known a priori
- Set of handlers specified dynamically
- Request processing pipelines

### Implementation: Request Handler Chain

```java
// Handler interface
public abstract class RequestHandler {
    protected RequestHandler nextHandler;
    
    public void setNext(RequestHandler handler) {
        this.nextHandler = handler;
    }
    
    public abstract void handle(Request request);
    
    protected void passToNext(Request request) {
        if (nextHandler != null) {
            nextHandler.handle(request);
        } else {
            System.out.println("End of chain - request not handled");
        }
    }
}

// Concrete handlers
public class AuthenticationHandler extends RequestHandler {
    @Override
    public void handle(Request request) {
        if (request.getHeader("Authorization") == null) {
            System.out.println("Authentication failed - no token");
            request.setStatus(401);
            return;
        }
        System.out.println("Authentication passed");
        passToNext(request);
    }
}

public class AuthorizationHandler extends RequestHandler {
    @Override
    public void handle(Request request) {
        String role = request.getHeader("Role");
        if (!"ADMIN".equals(role) && request.getPath().startsWith("/admin")) {
            System.out.println("Authorization failed - insufficient permissions");
            request.setStatus(403);
            return;
        }
        System.out.println("Authorization passed");
        passToNext(request);
    }
}

public class RateLimitHandler extends RequestHandler {
    private Map<String, Integer> requestCounts = new HashMap<>();
    private static final int LIMIT = 100;
    
    @Override
    public void handle(Request request) {
        String clientId = request.getHeader("Client-Id");
        int count = requestCounts.getOrDefault(clientId, 0);
        
        if (count >= LIMIT) {
            System.out.println("Rate limit exceeded");
            request.setStatus(429);
            return;
        }
        
        requestCounts.put(clientId, count + 1);
        System.out.println("Rate limit check passed");
        passToNext(request);
    }
}

public class LoggingHandler extends RequestHandler {
    @Override
    public void handle(Request request) {
        System.out.println("[LOG] " + request.getMethod() + " " + request.getPath());
        passToNext(request);
    }
}

public class BusinessLogicHandler extends RequestHandler {
    @Override
    public void handle(Request request) {
        System.out.println("Executing business logic for: " + request.getPath());
        request.setStatus(200);
        request.setBody("Success");
    }
}

// Build and use chain
public class Main {
    public static void main(String[] args) {
        // Build chain
        RequestHandler chain = new LoggingHandler();
        RequestHandler auth = new AuthenticationHandler();
        RequestHandler authz = new AuthorizationHandler();
        RequestHandler rateLimit = new RateLimitHandler();
        RequestHandler business = new BusinessLogicHandler();
        
        chain.setNext(auth);
        auth.setNext(authz);
        authz.setNext(rateLimit);
        rateLimit.setNext(business);
        
        // Process request
        Request request = new Request("GET", "/api/users");
        request.addHeader("Authorization", "Bearer token123");
        request.addHeader("Role", "USER");
        request.addHeader("Client-Id", "client1");
        
        chain.handle(request);
        System.out.println("Response status: " + request.getStatus());
    }
}
```

---

## 7. Iterator Pattern

### Intent
Provide a way to access elements of a collection **sequentially** without exposing its underlying representation.

### When to Use
- Access collection elements without exposing internal structure
- Support multiple traversals of collection
- Provide uniform interface for traversing different collections

### Implementation

```java
// Iterator interface
public interface Iterator<T> {
    boolean hasNext();
    T next();
    void reset();
}

// Aggregate interface
public interface IterableCollection<T> {
    Iterator<T> createIterator();
}

// Concrete collection
public class BookCollection implements IterableCollection<Book> {
    private List<Book> books = new ArrayList<>();
    
    public void addBook(Book book) {
        books.add(book);
    }
    
    public int size() {
        return books.size();
    }
    
    public Book get(int index) {
        return books.get(index);
    }
    
    @Override
    public Iterator<Book> createIterator() {
        return new BookIterator(this);
    }
    
    // Can have multiple iterator types
    public Iterator<Book> createReverseIterator() {
        return new ReverseBookIterator(this);
    }
}

// Concrete iterator
public class BookIterator implements Iterator<Book> {
    private BookCollection collection;
    private int currentIndex = 0;
    
    public BookIterator(BookCollection collection) {
        this.collection = collection;
    }
    
    @Override
    public boolean hasNext() {
        return currentIndex < collection.size();
    }
    
    @Override
    public Book next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return collection.get(currentIndex++);
    }
    
    @Override
    public void reset() {
        currentIndex = 0;
    }
}

// Usage
BookCollection library = new BookCollection();
library.addBook(new Book("Design Patterns", "GoF"));
library.addBook(new Book("Clean Code", "Robert Martin"));
library.addBook(new Book("Refactoring", "Martin Fowler"));

Iterator<Book> iterator = library.createIterator();
while (iterator.hasNext()) {
    Book book = iterator.next();
    System.out.println(book.getTitle() + " by " + book.getAuthor());
}
```

---

## Summary

| Pattern | Intent | Key Use Case |
|---------|--------|--------------|
| **Strategy** | Interchangeable algorithms | Payment methods |
| **Observer** | Notify on changes | Event systems |
| **Command** | Encapsulate request | Undo/redo |
| **State** | Behavior by state | Vending machine |
| **Template Method** | Algorithm skeleton | Data processing |
| **Chain of Responsibility** | Pass request | Request handlers |
| **Iterator** | Traverse collection | Collection access |

---

**Back to: [Design Patterns Overview](../README.md)**
