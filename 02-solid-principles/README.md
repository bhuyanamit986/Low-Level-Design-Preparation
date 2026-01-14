# 📘 Section 2: SOLID Principles (Very High Priority ⭐⭐⭐)

> **Almost every good LLD answer implicitly uses these principles**

The SOLID principles are five design principles that help create maintainable, flexible, and scalable software. Understanding and applying these principles is crucial for LLD interviews.

---

## 📑 Table of Contents

1. [Single Responsibility Principle (SRP)](#1-single-responsibility-principle-srp)
2. [Open/Closed Principle (OCP)](#2-openclosed-principle-ocp)
3. [Liskov Substitution Principle (LSP)](#3-liskov-substitution-principle-lsp)
4. [Interface Segregation Principle (ISP)](#4-interface-segregation-principle-isp)
5. [Dependency Inversion Principle (DIP)](#5-dependency-inversion-principle-dip)
6. [How Interviewers Detect SOLID Thinking](#6-how-interviewers-detect-solid-thinking)
7. [Practice Exercises](#7-practice-exercises)

---

## 1. Single Responsibility Principle (SRP)

> **"A class should have only one reason to change."** - Robert C. Martin

### What Does This Mean?

A class should have only ONE job or responsibility. If a class has multiple responsibilities, changes to one responsibility might break the other.

```
┌─────────────────────────────────────────────────────────────┐
│              SINGLE RESPONSIBILITY PRINCIPLE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLATION                    ✓ COMPLIANT               │
│   ┌──────────────────┐           ┌──────────────────┐       │
│   │     Invoice      │           │     Invoice      │       │
│   ├──────────────────┤           ├──────────────────┤       │
│   │ - items          │           │ - items          │       │
│   │ - customer       │           │ - customer       │       │
│   │ + calculateTotal()│          │ + calculateTotal()│      │
│   │ + printInvoice() │           │ + getItems()     │       │
│   │ + saveToDatabase()│          └──────────────────┘       │
│   │ + sendByEmail()  │                                      │
│   └──────────────────┘           ┌──────────────────┐       │
│                                  │  InvoicePrinter  │       │
│   4 reasons to change:           │ + print(Invoice) │       │
│   - Business logic changes       └──────────────────┘       │
│   - Print format changes                                    │
│   - Database changes             ┌──────────────────┐       │
│   - Email system changes         │  InvoiceRepository│      │
│                                  │ + save(Invoice)  │       │
│                                  └──────────────────┘       │
│                                                             │
│                                  ┌──────────────────┐       │
│                                  │   EmailService   │       │
│                                  │ + send(Invoice)  │       │
│                                  └──────────────────┘       │
│                                  1 reason to change each    │
└─────────────────────────────────────────────────────────────┘
```

### Code Example - Violation

```java
// ❌ VIOLATION: This class has multiple responsibilities
public class Employee {
    private String name;
    private String email;
    private double salary;
    
    // Responsibility 1: Employee data management
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    // Responsibility 2: Salary calculation (business logic)
    public double calculatePay() {
        // Tax calculations, overtime, benefits...
        return salary * 0.7; // After tax
    }
    
    // Responsibility 3: Persistence
    public void saveToDatabase() {
        // JDBC code to save employee
        Connection conn = DriverManager.getConnection("...");
        PreparedStatement stmt = conn.prepareStatement("INSERT INTO employees...");
        // ...
    }
    
    // Responsibility 4: Reporting
    public String generatePayStub() {
        return "Pay Stub for " + name + "\n" +
               "Gross: $" + salary + "\n" +
               "Net: $" + calculatePay();
    }
    
    // Responsibility 5: Notification
    public void sendPaymentNotification() {
        // Email sending logic
        EmailSender.send(email, "Payment Processed", generatePayStub());
    }
}
```

### Code Example - Compliant

```java
// ✓ COMPLIANT: Each class has a single responsibility

// Responsibility 1: Employee data (entity)
public class Employee {
    private String id;
    private String name;
    private String email;
    private double baseSalary;
    
    public Employee(String id, String name, String email, double baseSalary) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.baseSalary = baseSalary;
    }
    
    // Only getters and setters for employee data
    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public double getBaseSalary() { return baseSalary; }
}

// Responsibility 2: Payroll calculations
public class PayrollCalculator {
    private static final double TAX_RATE = 0.3;
    private static final double OVERTIME_MULTIPLIER = 1.5;
    
    public double calculateNetPay(Employee employee, int hoursWorked) {
        double grossPay = calculateGrossPay(employee, hoursWorked);
        double tax = grossPay * TAX_RATE;
        return grossPay - tax;
    }
    
    public double calculateGrossPay(Employee employee, int hoursWorked) {
        double hourlyRate = employee.getBaseSalary() / 160; // Monthly to hourly
        if (hoursWorked > 160) {
            int overtime = hoursWorked - 160;
            return (160 * hourlyRate) + (overtime * hourlyRate * OVERTIME_MULTIPLIER);
        }
        return hoursWorked * hourlyRate;
    }
}

// Responsibility 3: Persistence
public class EmployeeRepository {
    private DataSource dataSource;
    
    public EmployeeRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    public void save(Employee employee) {
        // Database save logic only
    }
    
    public Employee findById(String id) {
        // Database retrieval logic only
        return null;
    }
    
    public List<Employee> findAll() {
        // Database query logic only
        return new ArrayList<>();
    }
}

// Responsibility 4: Report generation
public class PayStubGenerator {
    private PayrollCalculator calculator;
    
    public PayStubGenerator(PayrollCalculator calculator) {
        this.calculator = calculator;
    }
    
    public String generate(Employee employee, int hoursWorked) {
        double gross = calculator.calculateGrossPay(employee, hoursWorked);
        double net = calculator.calculateNetPay(employee, hoursWorked);
        
        return String.format(
            "=== PAY STUB ===\n" +
            "Employee: %s\n" +
            "Hours: %d\n" +
            "Gross Pay: $%.2f\n" +
            "Net Pay: $%.2f\n",
            employee.getName(), hoursWorked, gross, net
        );
    }
}

// Responsibility 5: Notifications
public class NotificationService {
    private EmailSender emailSender;
    
    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
    
    public void sendPaymentNotification(Employee employee, String payStub) {
        emailSender.send(
            employee.getEmail(),
            "Your Payment Has Been Processed",
            payStub
        );
    }
}

// Usage - Composition of focused classes
public class PayrollProcessor {
    private EmployeeRepository repository;
    private PayrollCalculator calculator;
    private PayStubGenerator stubGenerator;
    private NotificationService notificationService;
    
    public void processPayroll(String employeeId, int hoursWorked) {
        Employee employee = repository.findById(employeeId);
        String payStub = stubGenerator.generate(employee, hoursWorked);
        notificationService.sendPaymentNotification(employee, payStub);
    }
}
```

### Common SRP Violations in Interviews

| Class | Violation | Fix |
|-------|-----------|-----|
| `User` with `saveToDb()` | Entity + Persistence | Separate `UserRepository` |
| `Order` with `sendConfirmationEmail()` | Domain + Notification | Separate `NotificationService` |
| `Report` with `generatePDF()` | Data + Rendering | Separate `PDFRenderer` |
| `Product` with `calculateDiscount()` | Entity + Business Logic | Separate `PricingService` |

---

## 2. Open/Closed Principle (OCP)

> **"Software entities should be open for extension but closed for modification."**

### What Does This Mean?

You should be able to add new functionality without changing existing code. New features should be added by writing new code, not by modifying working code.

```
┌─────────────────────────────────────────────────────────────┐
│                OPEN/CLOSED PRINCIPLE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLATION                    ✓ COMPLIANT               │
│                                                             │
│   Adding new shape requires      Adding new shape requires  │
│   modifying existing code:       only new class:            │
│                                                             │
│   class AreaCalculator {         interface Shape {          │
│     double area(Object shape) {    double calculateArea();  │
│       if (shape instanceof        }                         │
│           Rectangle) {                                      │
│         // rectangle logic        class Rectangle           │
│       }                              implements Shape {     │
│       else if (shape instanceof     double calculateArea()  │
│           Circle) {               }                         │
│         // circle logic                                     │
│       }                           class Circle              │
│       // Must add new else-if       implements Shape {      │
│       // for every new shape!       double calculateArea()  │
│     }                             }                         │
│   }                                                         │
│                                   // Adding Triangle:       │
│   // Adding Triangle:             // Just create new class! │
│   // Must modify this class!      class Triangle            │
│                                      implements Shape {     │
│                                     double calculateArea()  │
│                                   }                         │
└─────────────────────────────────────────────────────────────┘
```

### Code Example - Violation

```java
// ❌ VIOLATION: Must modify this class for every new payment method
public class PaymentProcessor {
    
    public void processPayment(String paymentType, double amount) {
        if (paymentType.equals("CREDIT_CARD")) {
            processCreditCardPayment(amount);
        } 
        else if (paymentType.equals("DEBIT_CARD")) {
            processDebitCardPayment(amount);
        }
        else if (paymentType.equals("PAYPAL")) {
            processPayPalPayment(amount);
        }
        // Every new payment method requires modifying this class!
        // else if (paymentType.equals("CRYPTO")) { ... }
        // else if (paymentType.equals("APPLE_PAY")) { ... }
        else {
            throw new IllegalArgumentException("Unknown payment type");
        }
    }
    
    private void processCreditCardPayment(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
    }
    
    private void processDebitCardPayment(double amount) {
        System.out.println("Processing debit card payment: $" + amount);
    }
    
    private void processPayPalPayment(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}
```

### Code Example - Compliant

```java
// ✓ COMPLIANT: Open for extension, closed for modification

// Define the contract (abstraction)
public interface PaymentMethod {
    void processPayment(double amount);
    boolean validate();
    String getPaymentType();
}

// Concrete implementations
public class CreditCardPayment implements PaymentMethod {
    private String cardNumber;
    private String expiryDate;
    private String cvv;
    
    public CreditCardPayment(String cardNumber, String expiryDate, String cvv) {
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
        this.cvv = cvv;
    }
    
    @Override
    public void processPayment(double amount) {
        if (!validate()) {
            throw new IllegalStateException("Invalid credit card");
        }
        System.out.println("Processing credit card payment: $" + amount);
        // Credit card specific logic
    }
    
    @Override
    public boolean validate() {
        return cardNumber != null && cardNumber.length() == 16;
    }
    
    @Override
    public String getPaymentType() {
        return "CREDIT_CARD";
    }
}

public class PayPalPayment implements PaymentMethod {
    private String email;
    private String authToken;
    
    public PayPalPayment(String email, String authToken) {
        this.email = email;
        this.authToken = authToken;
    }
    
    @Override
    public void processPayment(double amount) {
        if (!validate()) {
            throw new IllegalStateException("Invalid PayPal account");
        }
        System.out.println("Processing PayPal payment: $" + amount);
        // PayPal specific logic
    }
    
    @Override
    public boolean validate() {
        return email != null && authToken != null;
    }
    
    @Override
    public String getPaymentType() {
        return "PAYPAL";
    }
}

// Adding a new payment method doesn't require modifying existing code!
public class CryptoPayment implements PaymentMethod {
    private String walletAddress;
    private String cryptoType;
    
    public CryptoPayment(String walletAddress, String cryptoType) {
        this.walletAddress = walletAddress;
        this.cryptoType = cryptoType;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing " + cryptoType + " payment: $" + amount);
        // Crypto specific logic
    }
    
    @Override
    public boolean validate() {
        return walletAddress != null && walletAddress.length() > 20;
    }
    
    @Override
    public String getPaymentType() {
        return "CRYPTO";
    }
}

// Payment processor is now CLOSED for modification
public class PaymentProcessor {
    
    public void processPayment(PaymentMethod paymentMethod, double amount) {
        // No if-else chains!
        // Works with any payment method without modification
        if (!paymentMethod.validate()) {
            throw new IllegalStateException("Invalid payment method");
        }
        paymentMethod.processPayment(amount);
        logTransaction(paymentMethod.getPaymentType(), amount);
    }
    
    private void logTransaction(String type, double amount) {
        System.out.println("Logged: " + type + " - $" + amount);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        
        // Process different payment types polymorphically
        processor.processPayment(
            new CreditCardPayment("1234567890123456", "12/25", "123"), 
            100.00
        );
        
        processor.processPayment(
            new PayPalPayment("user@example.com", "auth-token"), 
            50.00
        );
        
        processor.processPayment(
            new CryptoPayment("0x1234...abcd", "ETH"), 
            200.00
        );
    }
}
```

### OCP with Strategy Pattern

```java
// Another common OCP implementation using Strategy pattern
public interface DiscountStrategy {
    double calculateDiscount(double price);
    String getDiscountType();
}

public class NoDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(double price) {
        return 0;
    }
    
    @Override
    public String getDiscountType() {
        return "NONE";
    }
}

public class PercentageDiscount implements DiscountStrategy {
    private double percentage;
    
    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }
    
    @Override
    public double calculateDiscount(double price) {
        return price * (percentage / 100);
    }
    
    @Override
    public String getDiscountType() {
        return percentage + "% OFF";
    }
}

public class FlatDiscount implements DiscountStrategy {
    private double amount;
    
    public FlatDiscount(double amount) {
        this.amount = amount;
    }
    
    @Override
    public double calculateDiscount(double price) {
        return Math.min(amount, price);
    }
    
    @Override
    public String getDiscountType() {
        return "$" + amount + " OFF";
    }
}

// Adding new discount types doesn't modify existing code
public class BuyOneGetOneFreeDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(double price) {
        return price / 2; // 50% off effectively
    }
    
    @Override
    public String getDiscountType() {
        return "BOGO";
    }
}

// Price calculator is closed for modification
public class PriceCalculator {
    public double calculateFinalPrice(double originalPrice, DiscountStrategy discount) {
        double discountAmount = discount.calculateDiscount(originalPrice);
        return originalPrice - discountAmount;
    }
}
```

---

## 3. Liskov Substitution Principle (LSP)

> **"Subtypes must be substitutable for their base types."** - Barbara Liskov

### What Does This Mean?

If class B is a subtype of class A, then you should be able to replace A with B without breaking the program. The subclass should extend the capability of the parent, not narrow it down.

```
┌─────────────────────────────────────────────────────────────┐
│              LISKOV SUBSTITUTION PRINCIPLE                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   The Classic Rectangle-Square Problem                      │
│                                                             │
│   Mathematically: Square IS-A Rectangle                     │
│   But in OOP: Square should NOT extend Rectangle!           │
│                                                             │
│   ❌ VIOLATION:                                             │
│   Rectangle r = new Square(5);                              │
│   r.setWidth(10);    // Sets both width AND height to 10    │
│   r.setHeight(20);   // Sets both width AND height to 20    │
│   // Expected area: 10 * 20 = 200                           │
│   // Actual area: 20 * 20 = 400  ← WRONG!                   │
│                                                             │
│   The Square CHANGES the behavior of Rectangle's setters,   │
│   violating the expected contract.                          │
└─────────────────────────────────────────────────────────────┘
```

### Code Example - Violation (Rectangle-Square)

```java
// ❌ VIOLATION: Square violates Rectangle's contract
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getWidth() { return width; }
    public int getHeight() { return height; }
    
    public int calculateArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    // Overriding to maintain square property
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Also changes height!
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height; // Also changes width!
        this.height = height;
    }
}

// This code BREAKS with Square
public class AreaCalculator {
    public void testRectangle(Rectangle rect) {
        rect.setWidth(5);
        rect.setHeight(10);
        
        // Expected: 5 * 10 = 50
        int expectedArea = 50;
        int actualArea = rect.calculateArea();
        
        assert expectedArea == actualArea; // FAILS for Square!
        // Square gives: 10 * 10 = 100
    }
}
```

### Code Example - Compliant

```java
// ✓ COMPLIANT: Use abstraction instead of inheritance

// Option 1: Common interface
public interface Shape {
    int calculateArea();
}

public class Rectangle implements Shape {
    private final int width;
    private final int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public int getWidth() { return width; }
    public int getHeight() { return height; }
    
    @Override
    public int calculateArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private final int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public int getSide() { return side; }
    
    @Override
    public int calculateArea() {
        return side * side;
    }
}

// Both work correctly with Shape interface
public class AreaCalculator {
    public int calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                     .mapToInt(Shape::calculateArea)
                     .sum();
    }
}
```

### Another LSP Violation: Bird Example

```java
// ❌ VIOLATION: Not all birds can fly!
public class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
    
    public void eat() {
        System.out.println("Eating...");
    }
}

public class Sparrow extends Bird {
    // OK - sparrows can fly
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
    // VIOLATION! Penguin breaks the Bird contract
}

// This code breaks with Penguin
public void makeBirdsFly(List<Bird> birds) {
    for (Bird bird : birds) {
        bird.fly(); // Throws exception for Penguin!
    }
}
```

### Compliant Solution - Bird Example

```java
// ✓ COMPLIANT: Separate flying capability
public abstract class Bird {
    public abstract void eat();
    public abstract void move();
}

public interface Flyable {
    void fly();
}

public class Sparrow extends Bird implements Flyable {
    @Override
    public void eat() {
        System.out.println("Sparrow eating seeds");
    }
    
    @Override
    public void move() {
        fly();
    }
    
    @Override
    public void fly() {
        System.out.println("Sparrow flying");
    }
}

public class Penguin extends Bird {
    @Override
    public void eat() {
        System.out.println("Penguin eating fish");
    }
    
    @Override
    public void move() {
        swim();
    }
    
    public void swim() {
        System.out.println("Penguin swimming");
    }
}

// Now this works correctly
public void makeBirdsMove(List<Bird> birds) {
    for (Bird bird : birds) {
        bird.move(); // Works for all birds!
    }
}

public void makeFlyablesFly(List<Flyable> flyables) {
    for (Flyable f : flyables) {
        f.fly(); // Only includes things that can actually fly
    }
}
```

### LSP Checklist

To verify LSP compliance, ask:

| Question | If NO → LSP Violation |
|----------|----------------------|
| Can subclass handle all inputs parent handles? | Subclass narrows input |
| Does subclass produce valid outputs for parent's contract? | Subclass breaks output expectations |
| Does subclass throw only exceptions parent throws? | Subclass adds unexpected exceptions |
| Does subclass maintain parent's invariants? | Subclass breaks internal rules |
| Can you use subclass wherever parent is used? | Substitution breaks behavior |

---

## 4. Interface Segregation Principle (ISP)

> **"Clients should not be forced to depend on interfaces they don't use."**

### What Does This Mean?

Instead of one large, general-purpose interface, create smaller, specific interfaces. Classes should only implement methods they actually need.

```
┌─────────────────────────────────────────────────────────────┐
│              INTERFACE SEGREGATION PRINCIPLE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLATION (Fat Interface)                              │
│   ┌─────────────────────────────────────────────────┐       │
│   │              «interface» Worker                 │       │
│   ├─────────────────────────────────────────────────┤       │
│   │ + work()                                        │       │
│   │ + eat()                                         │       │
│   │ + sleep()                                       │       │
│   │ + attendMeeting()                               │       │
│   │ + writeReport()                                 │       │
│   │ + reviewCode()                                  │       │
│   └─────────────────────────────────────────────────┘       │
│                          │                                  │
│           ┌──────────────┼──────────────┐                   │
│           ▼              ▼              ▼                   │
│     Developer        Manager         Robot                  │
│   (uses all)     (doesn't code)  (doesn't eat/sleep!)       │
│                                                             │
│   ✓ COMPLIANT (Segregated Interfaces)                       │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│   │  Workable    │  │   Feedable   │  │   Codeable   │      │
│   │ + work()     │  │ + eat()      │  │ + writeCode()│      │
│   └──────────────┘  │ + sleep()    │  │ + reviewCode()      │
│                     └──────────────┘  └──────────────┘      │
│                                                             │
│   Developer implements: Workable, Feedable, Codeable        │
│   Manager implements: Workable, Feedable                    │
│   Robot implements: Workable only                           │
└─────────────────────────────────────────────────────────────┘
```

### Code Example - Violation

```java
// ❌ VIOLATION: Fat interface forces unnecessary implementations
public interface Machine {
    void print(Document doc);
    void scan(Document doc);
    void fax(Document doc);
    void photocopy(Document doc);
    void staple(Document doc);
}

// Modern multi-function printer - OK
public class MultiFunctionPrinter implements Machine {
    @Override
    public void print(Document doc) { /* works */ }
    @Override
    public void scan(Document doc) { /* works */ }
    @Override
    public void fax(Document doc) { /* works */ }
    @Override
    public void photocopy(Document doc) { /* works */ }
    @Override
    public void staple(Document doc) { /* works */ }
}

// Simple printer - FORCED to implement unused methods!
public class SimplePrinter implements Machine {
    @Override
    public void print(Document doc) { 
        System.out.println("Printing...");
    }
    
    @Override
    public void scan(Document doc) {
        throw new UnsupportedOperationException("Cannot scan!");
    }
    
    @Override
    public void fax(Document doc) {
        throw new UnsupportedOperationException("Cannot fax!");
    }
    
    @Override
    public void photocopy(Document doc) {
        throw new UnsupportedOperationException("Cannot photocopy!");
    }
    
    @Override
    public void staple(Document doc) {
        throw new UnsupportedOperationException("Cannot staple!");
    }
}
```

### Code Example - Compliant

```java
// ✓ COMPLIANT: Segregated interfaces
public interface Printer {
    void print(Document doc);
}

public interface Scanner {
    void scan(Document doc);
}

public interface Fax {
    void fax(Document doc);
}

public interface Photocopier {
    void photocopy(Document doc);
}

public interface Stapler {
    void staple(Document doc);
}

// Multi-function printer implements all needed interfaces
public class MultiFunctionPrinter implements Printer, Scanner, Fax, Photocopier, Stapler {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document...");
    }
    
    @Override
    public void scan(Document doc) {
        System.out.println("Scanning document...");
    }
    
    @Override
    public void fax(Document doc) {
        System.out.println("Faxing document...");
    }
    
    @Override
    public void photocopy(Document doc) {
        System.out.println("Photocopying document...");
    }
    
    @Override
    public void staple(Document doc) {
        System.out.println("Stapling document...");
    }
}

// Simple printer only implements what it needs
public class SimplePrinter implements Printer {
    @Override
    public void print(Document doc) {
        System.out.println("Simple printing...");
    }
    // No forced empty implementations!
}

// Scanner-only device
public class PortableScanner implements Scanner {
    @Override
    public void scan(Document doc) {
        System.out.println("Portable scanning...");
    }
}

// Client code depends only on what it needs
public class PrintService {
    private Printer printer;
    
    public PrintService(Printer printer) {
        this.printer = printer; // Works with ANY printer
    }
    
    public void printDocument(Document doc) {
        printer.print(doc);
    }
}

public class ScanService {
    private Scanner scanner;
    
    public ScanService(Scanner scanner) {
        this.scanner = scanner; // Works with ANY scanner
    }
    
    public void scanDocument(Document doc) {
        scanner.scan(doc);
    }
}
```

### Real-World ISP Example

```java
// ❌ VIOLATION: Animal interface too broad
public interface Animal {
    void eat();
    void sleep();
    void fly();
    void swim();
    void run();
    void climb();
}

// Every animal must implement ALL methods!

// ✓ COMPLIANT: Capability-based interfaces
public interface Eater {
    void eat();
}

public interface Sleeper {
    void sleep();
}

public interface Flyer {
    void fly();
}

public interface Swimmer {
    void swim();
}

public interface Runner {
    void run();
}

public interface Climber {
    void climb();
}

// Animals compose only the capabilities they have
public class Dog implements Eater, Sleeper, Runner, Swimmer {
    public void eat() { System.out.println("Dog eating"); }
    public void sleep() { System.out.println("Dog sleeping"); }
    public void run() { System.out.println("Dog running"); }
    public void swim() { System.out.println("Dog swimming"); }
}

public class Bird implements Eater, Sleeper, Flyer {
    public void eat() { System.out.println("Bird eating"); }
    public void sleep() { System.out.println("Bird sleeping"); }
    public void fly() { System.out.println("Bird flying"); }
}

public class Fish implements Eater, Swimmer {
    public void eat() { System.out.println("Fish eating"); }
    public void swim() { System.out.println("Fish swimming"); }
    // Fish don't sleep in the traditional sense
}
```

---

## 5. Dependency Inversion Principle (DIP)

> **"High-level modules should not depend on low-level modules. Both should depend on abstractions."**
> 
> **"Abstractions should not depend on details. Details should depend on abstractions."**

### What Does This Mean?

Instead of high-level components depending directly on low-level components, both should depend on interfaces (abstractions). This inverts the traditional dependency direction.

```
┌─────────────────────────────────────────────────────────────┐
│              DEPENDENCY INVERSION PRINCIPLE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLATION                    ✓ COMPLIANT               │
│   (Direct Dependency)            (Inverted Dependency)      │
│                                                             │
│   ┌─────────────────┐            ┌─────────────────┐        │
│   │  OrderService   │            │  OrderService   │        │
│   │  (High-Level)   │            │  (High-Level)   │        │
│   └────────┬────────┘            └────────┬────────┘        │
│            │                              │                 │
│            │ depends on                   │ depends on      │
│            ▼                              ▼                 │
│   ┌─────────────────┐            ┌─────────────────┐        │
│   │  MySQLDatabase  │            │ «interface»     │        │
│   │  (Low-Level)    │            │   Database      │        │
│   └─────────────────┘            └────────┬────────┘        │
│                                           │                 │
│   Hard to change database!                │ implements      │
│   Hard to test!                           ▼                 │
│                                  ┌─────────────────┐        │
│                                  │  MySQLDatabase  │        │
│                                  │  (Low-Level)    │        │
│                                  └─────────────────┘        │
│                                                             │
│                                  Easy to swap database!     │
│                                  Easy to test with mocks!   │
└─────────────────────────────────────────────────────────────┘
```

### Code Example - Violation

```java
// ❌ VIOLATION: High-level depends directly on low-level
public class MySQLDatabase {
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }
    
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
    
    public String query(String sql) {
        return "MySQL result";
    }
}

public class EmailSender {
    public void send(String to, String subject, String body) {
        System.out.println("Sending email via SMTP to " + to);
    }
}

// High-level module directly depends on low-level implementations
public class OrderService {
    private MySQLDatabase database;  // Direct dependency!
    private EmailSender emailSender; // Direct dependency!
    
    public OrderService() {
        this.database = new MySQLDatabase();  // Creates own dependencies
        this.emailSender = new EmailSender();
    }
    
    public void placeOrder(Order order) {
        database.connect();
        database.save(order.toString());
        emailSender.send(order.getCustomerEmail(), "Order Placed", "...");
    }
}

// Problems:
// 1. Cannot use PostgreSQL without changing OrderService
// 2. Cannot send notifications via SMS without changing OrderService
// 3. Cannot test without real MySQL and email server
```

### Code Example - Compliant

```java
// ✓ COMPLIANT: Both depend on abstractions

// Abstractions (interfaces)
public interface Database {
    void connect();
    void save(String data);
    String query(String sql);
    void disconnect();
}

public interface NotificationSender {
    void send(String recipient, String subject, String message);
}

// Low-level implementations
public class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }
    
    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
    
    @Override
    public String query(String sql) {
        return "MySQL result";
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL...");
    }
}

public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL...");
    }
    
    @Override
    public void save(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }
    
    @Override
    public String query(String sql) {
        return "PostgreSQL result";
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL...");
    }
}

public class EmailNotificationSender implements NotificationSender {
    @Override
    public void send(String recipient, String subject, String message) {
        System.out.println("Sending email to " + recipient);
    }
}

public class SMSNotificationSender implements NotificationSender {
    @Override
    public void send(String recipient, String subject, String message) {
        System.out.println("Sending SMS to " + recipient);
    }
}

// High-level module depends on ABSTRACTIONS
public class OrderService {
    private final Database database;
    private final NotificationSender notificationSender;
    
    // Dependencies injected via constructor
    public OrderService(Database database, NotificationSender notificationSender) {
        this.database = database;
        this.notificationSender = notificationSender;
    }
    
    public void placeOrder(Order order) {
        database.connect();
        try {
            database.save(order.toString());
            notificationSender.send(
                order.getCustomerEmail(),
                "Order Confirmation",
                "Your order has been placed!"
            );
        } finally {
            database.disconnect();
        }
    }
}

// Configuration / Composition Root
public class Application {
    public static void main(String[] args) {
        // Production configuration
        Database db = new MySQLDatabase();
        NotificationSender notifier = new EmailNotificationSender();
        OrderService orderService = new OrderService(db, notifier);
        
        // Easy to swap implementations
        // Database postgresDb = new PostgreSQLDatabase();
        // NotificationSender smsNotifier = new SMSNotificationSender();
        // OrderService orderService2 = new OrderService(postgresDb, smsNotifier);
    }
}

// Testing becomes easy with mock implementations
public class MockDatabase implements Database {
    public List<String> savedData = new ArrayList<>();
    
    @Override
    public void connect() { }
    
    @Override
    public void save(String data) {
        savedData.add(data);
    }
    
    @Override
    public String query(String sql) {
        return "mock result";
    }
    
    @Override
    public void disconnect() { }
}

public class OrderServiceTest {
    @Test
    public void testPlaceOrder() {
        MockDatabase mockDb = new MockDatabase();
        MockNotificationSender mockNotifier = new MockNotificationSender();
        
        OrderService service = new OrderService(mockDb, mockNotifier);
        service.placeOrder(new Order("test@example.com", "item1"));
        
        assertEquals(1, mockDb.savedData.size());
        assertTrue(mockNotifier.wasCalled());
    }
}
```

### Dependency Injection Methods

```java
// Method 1: Constructor Injection (Recommended)
public class OrderService {
    private final Database database;
    
    public OrderService(Database database) {
        this.database = database;
    }
}

// Method 2: Setter Injection
public class OrderService {
    private Database database;
    
    public void setDatabase(Database database) {
        this.database = database;
    }
}

// Method 3: Interface Injection
public interface DatabaseInjectable {
    void injectDatabase(Database database);
}

public class OrderService implements DatabaseInjectable {
    private Database database;
    
    @Override
    public void injectDatabase(Database database) {
        this.database = database;
    }
}
```

---

## 6. How Interviewers Detect SOLID Thinking

### What Interviewers Look For

| Principle | Signs You're Using It | Red Flags |
|-----------|----------------------|-----------|
| **SRP** | Small, focused classes | God classes, mixed concerns |
| **OCP** | Interfaces, strategy pattern | Long if-else/switch chains |
| **LSP** | Proper inheritance hierarchies | Override with exceptions |
| **ISP** | Small, specific interfaces | "Not implemented" methods |
| **DIP** | Constructor injection, interfaces | `new` keyword everywhere |

### Interview Questions That Test SOLID

1. **"How would you add a new payment method?"**
   - Good: "I'd create a new class implementing PaymentMethod interface"
   - Bad: "I'd add another if-else in the processPayment method"

2. **"How would you make this testable?"**
   - Good: "I'd inject the dependencies through the constructor"
   - Bad: "I'd use a real database in tests"

3. **"What if requirements change to support multiple notification channels?"**
   - Good: "I'd define a Notifier interface and implement different channels"
   - Bad: "I'd add a boolean flag to the sendNotification method"

---

## 7. Practice Exercises

### Exercise 1: Identify SOLID Violations

```java
public class ReportGenerator {
    public void generateReport(String type, List<Data> data) {
        String report = "";
        
        if (type.equals("PDF")) {
            report = formatAsPDF(data);
            saveToDisk(report, "report.pdf");
        } else if (type.equals("HTML")) {
            report = formatAsHTML(data);
            saveToDisk(report, "report.html");
        } else if (type.equals("CSV")) {
            report = formatAsCSV(data);
            saveToDisk(report, "report.csv");
        }
        
        sendEmail("admin@company.com", report);
    }
    
    private String formatAsPDF(List<Data> data) { /* ... */ }
    private String formatAsHTML(List<Data> data) { /* ... */ }
    private String formatAsCSV(List<Data> data) { /* ... */ }
    private void saveToDisk(String content, String filename) { /* ... */ }
    private void sendEmail(String to, String content) { /* ... */ }
}
```

<details>
<summary>Click to see violations and fix</summary>

**Violations:**
1. **SRP**: Class does formatting, saving, AND emailing
2. **OCP**: Adding new format requires modifying this class
3. **DIP**: Directly depends on file system and email

**Fixed Version:**

```java
// Abstraction for report formatting
public interface ReportFormatter {
    String format(List<Data> data);
    String getFileExtension();
}

public class PDFFormatter implements ReportFormatter {
    @Override
    public String format(List<Data> data) { /* PDF logic */ }
    @Override
    public String getFileExtension() { return "pdf"; }
}

public class HTMLFormatter implements ReportFormatter {
    @Override
    public String format(List<Data> data) { /* HTML logic */ }
    @Override
    public String getFileExtension() { return "html"; }
}

// Abstraction for saving
public interface ReportSaver {
    void save(String content, String filename);
}

// Abstraction for notification
public interface ReportNotifier {
    void notify(String recipient, String content);
}

// Clean, focused report generator
public class ReportGenerator {
    private final ReportSaver saver;
    private final ReportNotifier notifier;
    
    public ReportGenerator(ReportSaver saver, ReportNotifier notifier) {
        this.saver = saver;
        this.notifier = notifier;
    }
    
    public void generateReport(ReportFormatter formatter, List<Data> data, String recipient) {
        String report = formatter.format(data);
        String filename = "report." + formatter.getFileExtension();
        saver.save(report, filename);
        notifier.notify(recipient, report);
    }
}
```

</details>

### Exercise 2: Refactor for LSP

```java
public class FileStorage {
    public void write(String filename, String content) { /* ... */ }
    public String read(String filename) { /* ... */ }
    public void delete(String filename) { /* ... */ }
}

public class ReadOnlyFileStorage extends FileStorage {
    @Override
    public void write(String filename, String content) {
        throw new UnsupportedOperationException("Read-only storage!");
    }
    
    @Override
    public void delete(String filename) {
        throw new UnsupportedOperationException("Read-only storage!");
    }
}
```

<details>
<summary>Click to see the fix</summary>

```java
// Use interface segregation to fix LSP violation
public interface Readable {
    String read(String filename);
}

public interface Writable {
    void write(String filename, String content);
}

public interface Deletable {
    void delete(String filename);
}

public class FileStorage implements Readable, Writable, Deletable {
    @Override
    public void write(String filename, String content) { /* ... */ }
    @Override
    public String read(String filename) { /* ... */ }
    @Override
    public void delete(String filename) { /* ... */ }
}

public class ReadOnlyFileStorage implements Readable {
    @Override
    public String read(String filename) { /* ... */ }
    // No write or delete methods to throw exceptions!
}
```

</details>

---

## 📚 Key Takeaways

| Principle | Remember |
|-----------|----------|
| **SRP** | One class, one responsibility |
| **OCP** | Extend don't modify |
| **LSP** | Subclasses must be substitutable |
| **ISP** | Small, focused interfaces |
| **DIP** | Depend on abstractions |

### SOLID Mnemonic

```
S - "Single job for each class"
O - "Open doors for extension, closed for modification"
L - "Liskov says: kids must behave like parents"
I - "Interface should be slim, not fat"
D - "Depend on contracts, not concrete things"
```

---

**Next Section: [UML & Design Artifacts](../03-uml-design/README.md)** →
