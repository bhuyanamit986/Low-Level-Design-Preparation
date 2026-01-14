# 📘 Section 1: Foundations of Object-Oriented Design (OOD)

> **Non-negotiable for LLD interviews** - Master these concepts before moving forward.

---

## 📑 Table of Contents

1. [Object-Oriented Programming Basics](#1-object-oriented-programming-basics)
2. [The Four Pillars of OOP](#2-the-four-pillars-of-oop)
3. [Object Relationships](#3-object-relationships)
4. [Cohesion and Coupling](#4-cohesion-and-coupling)
5. [Interfaces vs Abstract Classes](#5-interfaces-vs-abstract-classes)
6. [Practice Exercises](#6-practice-exercises)

---

## 1. Object-Oriented Programming Basics

### What is OOP?

**Object-Oriented Programming (OOP)** is a programming paradigm that organizes software design around **objects** rather than functions and logic. Objects represent real-world entities with attributes (data) and behaviors (methods).

### Classes vs Objects

#### Class
A **class** is a blueprint or template that defines:
- **Attributes** (what the object knows)
- **Methods** (what the object does)

#### Object
An **object** is an instance of a class - a concrete entity created from the blueprint.

```
┌─────────────────────────────────────────────────────────────┐
│                        CLASS: Car                           │
├─────────────────────────────────────────────────────────────┤
│  Blueprint/Template                                         │
│  - Defines structure                                        │
│  - No memory allocated                                      │
│  - Like a cookie cutter                                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ instantiation
                              ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   OBJECT: car1   │  │   OBJECT: car2   │  │   OBJECT: car3   │
├──────────────────┤  ├──────────────────┤  ├──────────────────┤
│ brand: "Toyota"  │  │ brand: "Honda"   │  │ brand: "Tesla"   │
│ color: "Red"     │  │ color: "Blue"    │  │ color: "White"   │
│ speed: 120       │  │ speed: 110       │  │ speed: 150       │
└──────────────────┘  └──────────────────┘  └──────────────────┘
     Memory                Memory                Memory
     Allocated             Allocated             Allocated
```

#### Code Example - Java

```java
// Class Definition - The Blueprint
public class Car {
    // Attributes (Instance Variables)
    private String brand;
    private String color;
    private int speed;
    
    // Constructor
    public Car(String brand, String color) {
        this.brand = brand;
        this.color = color;
        this.speed = 0;
    }
    
    // Methods (Behaviors)
    public void accelerate(int increment) {
        this.speed += increment;
        System.out.println(brand + " accelerating to " + speed + " km/h");
    }
    
    public void brake() {
        this.speed = Math.max(0, speed - 20);
        System.out.println(brand + " slowing down to " + speed + " km/h");
    }
    
    // Getters
    public String getBrand() { return brand; }
    public int getSpeed() { return speed; }
}

// Creating Objects - Instances of the Class
public class Main {
    public static void main(String[] args) {
        // car1 and car2 are objects (instances) of class Car
        Car car1 = new Car("Toyota", "Red");
        Car car2 = new Car("Honda", "Blue");
        
        car1.accelerate(60);  // Toyota accelerating to 60 km/h
        car2.accelerate(80);  // Honda accelerating to 80 km/h
        
        // Each object has its own state
        System.out.println(car1.getSpeed()); // 60
        System.out.println(car2.getSpeed()); // 80
    }
}
```

#### Code Example - Python

```python
# Class Definition - The Blueprint
class Car:
    def __init__(self, brand: str, color: str):
        # Attributes (Instance Variables)
        self.brand = brand
        self.color = color
        self.speed = 0
    
    # Methods (Behaviors)
    def accelerate(self, increment: int) -> None:
        self.speed += increment
        print(f"{self.brand} accelerating to {self.speed} km/h")
    
    def brake(self) -> None:
        self.speed = max(0, self.speed - 20)
        print(f"{self.brand} slowing down to {self.speed} km/h")


# Creating Objects - Instances of the Class
if __name__ == "__main__":
    car1 = Car("Toyota", "Red")
    car2 = Car("Honda", "Blue")
    
    car1.accelerate(60)  # Toyota accelerating to 60 km/h
    car2.accelerate(80)  # Honda accelerating to 80 km/h
    
    print(car1.speed)  # 60
    print(car2.speed)  # 80
```

---

## 2. The Four Pillars of OOP

### 2.1 Abstraction

> **Definition**: Abstraction is the process of hiding complex implementation details and showing only the essential features of an object.

**Real-world analogy**: When you drive a car, you use the steering wheel, pedals, and gear shift. You don't need to know how the engine works internally.

```
┌─────────────────────────────────────────────────────────────┐
│                    ABSTRACTION                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   What user sees:          What's hidden:                   │
│   ┌───────────────┐        ┌───────────────────────────┐   │
│   │ sendEmail()   │   →    │ - Connect to SMTP server  │   │
│   │ subject       │        │ - Authenticate            │   │
│   │ body          │        │ - Format message          │   │
│   │ recipient     │        │ - Handle encoding         │   │
│   └───────────────┘        │ - Retry on failure        │   │
│                            │ - Log transaction         │   │
│                            └───────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Java

```java
// Abstraction using Abstract Class
public abstract class PaymentProcessor {
    // Abstract method - WHAT to do (no implementation)
    public abstract void processPayment(double amount);
    
    // Concrete method - common functionality
    public void validateAmount(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
}

// Concrete implementation - HOW to do it
public class CreditCardProcessor extends PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        validateAmount(amount);
        // Complex implementation hidden
        connectToPaymentGateway();
        authenticateCard();
        chargeCard(amount);
        sendConfirmation();
    }
    
    private void connectToPaymentGateway() { /* ... */ }
    private void authenticateCard() { /* ... */ }
    private void chargeCard(double amount) { /* ... */ }
    private void sendConfirmation() { /* ... */ }
}

// User only sees the simple interface
public class Main {
    public static void main(String[] args) {
        PaymentProcessor processor = new CreditCardProcessor();
        processor.processPayment(100.00); // Simple!
    }
}
```

### 2.2 Encapsulation

> **Definition**: Encapsulation is bundling data (attributes) and methods that operate on that data within a single unit (class), while restricting direct access to some components.

**Key concepts**:
- **Data hiding**: Private attributes
- **Controlled access**: Getters and setters
- **Validation**: Business rules in setters

```
┌─────────────────────────────────────────────────────────────┐
│                    ENCAPSULATION                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  BankAccount                        │   │
│   ├─────────────────────────────────────────────────────┤   │
│   │  PRIVATE (Hidden):                                  │   │
│   │  - balance                                          │   │
│   │  - accountNumber                                    │   │
│   │  - pin                                              │   │
│   ├─────────────────────────────────────────────────────┤   │
│   │  PUBLIC (Accessible):                               │   │
│   │  + getBalance()                                     │   │
│   │  + deposit(amount)                                  │   │
│   │  + withdraw(amount)                                 │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   External code can ONLY interact through public methods    │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Java

```java
public class BankAccount {
    // Private attributes - hidden from outside
    private String accountNumber;
    private double balance;
    private String pin;
    
    public BankAccount(String accountNumber, String pin) {
        this.accountNumber = accountNumber;
        this.pin = pin;
        this.balance = 0.0;
    }
    
    // Public getter - controlled read access
    public double getBalance() {
        return balance;
    }
    
    // Public method with validation - controlled write access
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance += amount;
        System.out.println("Deposited: $" + amount + ". New balance: $" + balance);
    }
    
    // Public method with business logic
    public boolean withdraw(double amount, String enteredPin) {
        // Validation
        if (!this.pin.equals(enteredPin)) {
            System.out.println("Invalid PIN!");
            return false;
        }
        if (amount <= 0) {
            System.out.println("Invalid amount!");
            return false;
        }
        if (amount > balance) {
            System.out.println("Insufficient funds!");
            return false;
        }
        
        this.balance -= amount;
        System.out.println("Withdrawn: $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    // Account number is read-only (no setter)
    public String getAccountNumber() {
        return accountNumber;
    }
    
    // PIN is completely hidden (no getter or setter)
    // Can only be validated internally
}

// Usage
public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount("ACC123", "1234");
        
        account.deposit(1000);              // Works
        account.withdraw(500, "1234");      // Works
        account.withdraw(200, "wrong");     // Fails - invalid PIN
        
        // account.balance = 1000000;       // COMPILE ERROR! balance is private
        // account.pin = "9999";            // COMPILE ERROR! pin is private
    }
}
```

#### Code Example - Python

```python
class BankAccount:
    def __init__(self, account_number: str, pin: str):
        # Private attributes (convention: underscore prefix)
        self._account_number = account_number
        self._balance = 0.0
        self.__pin = pin  # Double underscore for name mangling
    
    @property
    def balance(self) -> float:
        """Read-only property for balance"""
        return self._balance
    
    @property
    def account_number(self) -> str:
        """Read-only property for account number"""
        return self._account_number
    
    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        print(f"Deposited: ${amount}. New balance: ${self._balance}")
    
    def withdraw(self, amount: float, entered_pin: str) -> bool:
        if self.__pin != entered_pin:
            print("Invalid PIN!")
            return False
        if amount <= 0 or amount > self._balance:
            print("Invalid amount or insufficient funds!")
            return False
        
        self._balance -= amount
        print(f"Withdrawn: ${amount}. New balance: ${self._balance}")
        return True


# Usage
if __name__ == "__main__":
    account = BankAccount("ACC123", "1234")
    account.deposit(1000)
    account.withdraw(500, "1234")
    
    print(account.balance)  # 500.0 - using property
    # account.balance = 1000000  # AttributeError! balance is read-only
```

### 2.3 Inheritance

> **Definition**: Inheritance is a mechanism where a new class (child/derived) acquires properties and behaviors from an existing class (parent/base).

**Types of Inheritance**:
- **Single Inheritance**: One parent, one child
- **Multilevel Inheritance**: Grandparent → Parent → Child
- **Hierarchical Inheritance**: One parent, multiple children
- **Multiple Inheritance**: Multiple parents (supported in Python, not Java)

```
┌─────────────────────────────────────────────────────────────┐
│                    INHERITANCE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────────┐                          │
│                    │   Animal    │ ← Parent/Base Class      │
│                    │  - name     │                          │
│                    │  + eat()    │                          │
│                    │  + sleep()  │                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│              ┌────────────┼────────────┐                    │
│              │            │            │                    │
│              ▼            ▼            ▼                    │
│       ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│       │   Dog    │ │   Cat    │ │   Bird   │               │
│       │  + bark()│ │  + meow()│ │  + fly() │               │
│       └──────────┘ └──────────┘ └──────────┘               │
│       Child/Derived Classes                                 │
│       Inherit: name, eat(), sleep()                         │
│       Add: their own specific methods                       │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Java

```java
// Parent class
public class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    public String getInfo() {
        return "Name: " + name + ", Age: " + age;
    }
}

// Child class - inherits from Animal
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    // New method specific to Dog
    public void bark() {
        System.out.println(name + " says: Woof! Woof!");
    }
    
    // Override parent method
    @Override
    public String getInfo() {
        return super.getInfo() + ", Breed: " + breed;
    }
}

// Another child class
public class Cat extends Animal {
    private boolean isIndoor;
    
    public Cat(String name, int age, boolean isIndoor) {
        super(name, age);
        this.isIndoor = isIndoor;
    }
    
    public void meow() {
        System.out.println(name + " says: Meow!");
    }
    
    public void purr() {
        System.out.println(name + " is purring...");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Buddy", 3, "Golden Retriever");
        Cat cat = new Cat("Whiskers", 2, true);
        
        // Inherited methods
        dog.eat();    // Buddy is eating
        dog.sleep();  // Buddy is sleeping
        cat.eat();    // Whiskers is eating
        
        // Own methods
        dog.bark();   // Buddy says: Woof! Woof!
        cat.meow();   // Whiskers says: Meow!
        
        // Overridden method
        System.out.println(dog.getInfo()); 
        // Name: Buddy, Age: 3, Breed: Golden Retriever
    }
}
```

#### IS-A vs HAS-A Relationship

```
IS-A Relationship (Inheritance):
- Dog IS-A Animal ✓
- Car IS-A Vehicle ✓
- Manager IS-A Employee ✓

HAS-A Relationship (Composition):
- Car HAS-A Engine ✓
- Library HAS-A Books ✓
- University HAS-A Departments ✓

⚠️ Common Mistake:
- Rectangle IS-A Square? ✗ (Violates LSP - see SOLID)
- Stack IS-A ArrayList? ✗ (Use composition instead)
```

### 2.4 Polymorphism

> **Definition**: Polymorphism means "many forms" - the ability of objects of different classes to respond to the same method call in different ways.

**Types**:
- **Compile-time (Static)**: Method overloading
- **Runtime (Dynamic)**: Method overriding

```
┌─────────────────────────────────────────────────────────────┐
│                    POLYMORPHISM                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Same method call → Different behaviors                     │
│                                                             │
│      shape.draw()                                           │
│           │                                                 │
│           ├──── Circle object  → draws a circle             │
│           ├──── Square object  → draws a square             │
│           └──── Triangle obj   → draws a triangle           │
│                                                             │
│  The actual method executed depends on the object type      │
│  at RUNTIME, not the reference type.                        │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Java (Method Overriding - Runtime Polymorphism)

```java
// Base class
public abstract class Shape {
    protected String color;
    
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract method - must be implemented by subclasses
    public abstract double calculateArea();
    public abstract void draw();
}

// Concrete implementations
public class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " circle with radius " + radius);
    }
}

public class Rectangle extends Shape {
    private double width;
    private double height;
    
    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " rectangle " + width + "x" + height);
    }
}

public class Triangle extends Shape {
    private double base;
    private double height;
    
    public Triangle(String color, double base, double height) {
        super(color);
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " triangle");
    }
}

// Polymorphism in action
public class Main {
    public static void main(String[] args) {
        // Array of Shape references, holding different shape objects
        Shape[] shapes = {
            new Circle("Red", 5),
            new Rectangle("Blue", 4, 6),
            new Triangle("Green", 3, 4)
        };
        
        // Polymorphic behavior - same method, different results
        for (Shape shape : shapes) {
            shape.draw();
            System.out.println("Area: " + shape.calculateArea());
            System.out.println();
        }
    }
}

/* Output:
Drawing a Red circle with radius 5.0
Area: 78.53981633974483

Drawing a Blue rectangle 4.0x6.0
Area: 24.0

Drawing a Green triangle
Area: 6.0
*/
```

#### Code Example - Java (Method Overloading - Compile-time Polymorphism)

```java
public class Calculator {
    // Same method name, different parameters
    
    public int add(int a, int b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public String add(String a, String b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        
        System.out.println(calc.add(5, 3));           // 8 (int version)
        System.out.println(calc.add(5, 3, 2));        // 10 (3-param version)
        System.out.println(calc.add(5.5, 3.2));       // 8.7 (double version)
        System.out.println(calc.add("Hello", " World")); // Hello World (String version)
    }
}
```

---

## 3. Object Relationships

Understanding how objects relate to each other is crucial for LLD interviews.

### 3.1 Association

> **Definition**: A general relationship where objects are aware of each other. They have their own lifecycle and can exist independently.

```
┌─────────────────────────────────────────────────────────────┐
│                    ASSOCIATION                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Teacher ←────────────────→ Student                        │
│                                                             │
│   - A teacher can teach multiple students                   │
│   - A student can learn from multiple teachers              │
│   - Both exist independently                                │
│   - Neither owns the other                                  │
│                                                             │
│   Multiplicity: many-to-many                                │
└─────────────────────────────────────────────────────────────┘
```

```java
public class Teacher {
    private String name;
    private List<Student> students = new ArrayList<>();
    
    public void addStudent(Student student) {
        students.add(student);
    }
}

public class Student {
    private String name;
    private List<Teacher> teachers = new ArrayList<>();
    
    public void addTeacher(Teacher teacher) {
        teachers.add(teacher);
    }
}
```

### 3.2 Aggregation (Weak "HAS-A")

> **Definition**: A specialized form of association representing a "whole-part" relationship where the parts CAN exist independently of the whole.

```
┌─────────────────────────────────────────────────────────────┐
│                    AGGREGATION (Weak HAS-A)                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Department ◇────────────→ Professor                       │
│                                                             │
│   - Department HAS Professors                               │
│   - Professors CAN exist without Department                 │
│   - If Department is deleted, Professors still exist        │
│   - Professors can belong to multiple departments           │
│                                                             │
│   Symbol: Empty diamond (◇) at the whole side               │
└─────────────────────────────────────────────────────────────┘
```

```java
public class Professor {
    private String name;
    private String specialization;
    
    public Professor(String name, String specialization) {
        this.name = name;
        this.specialization = specialization;
    }
}

public class Department {
    private String name;
    private List<Professor> professors = new ArrayList<>();
    
    // Professors are passed in - created outside
    public void addProfessor(Professor professor) {
        professors.add(professor);
    }
    
    public void removeProfessor(Professor professor) {
        professors.remove(professor);
        // Professor still exists after removal
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Professor prof = new Professor("Dr. Smith", "Physics");
        
        Department physics = new Department("Physics");
        Department math = new Department("Mathematics");
        
        physics.addProfessor(prof);  // Same professor
        math.addProfessor(prof);     // in multiple departments
        
        // If physics department is deleted, prof still exists
    }
}
```

### 3.3 Composition (Strong "HAS-A")

> **Definition**: A strong form of aggregation where the parts CANNOT exist without the whole. The whole controls the lifecycle of its parts.

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPOSITION (Strong HAS-A)               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   House ◆────────────→ Room                                 │
│                                                             │
│   - House HAS Rooms                                         │
│   - Rooms CANNOT exist without House                        │
│   - If House is deleted, Rooms are deleted too              │
│   - House controls Room lifecycle                           │
│                                                             │
│   Symbol: Filled diamond (◆) at the whole side              │
└─────────────────────────────────────────────────────────────┘
```

```java
public class Room {
    private String name;
    private int area;
    
    public Room(String name, int area) {
        this.name = name;
        this.area = area;
    }
}

public class House {
    private String address;
    private List<Room> rooms = new ArrayList<>();
    
    public House(String address) {
        this.address = address;
    }
    
    // Rooms are created INSIDE the house
    public void addRoom(String name, int area) {
        rooms.add(new Room(name, area));  // House creates the Room
    }
    
    // When house is destroyed, rooms go with it
    // No way to get a room and use it outside the house
}

// Usage
public class Main {
    public static void main(String[] args) {
        House house = new House("123 Main St");
        house.addRoom("Living Room", 300);
        house.addRoom("Bedroom", 200);
        
        // If house goes out of scope, rooms are garbage collected too
    }
}
```

### 3.4 Dependency

> **Definition**: A weaker relationship where one class uses another class temporarily. Changes in the used class may affect the dependent class.

```
┌─────────────────────────────────────────────────────────────┐
│                    DEPENDENCY                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   OrderProcessor - - - - -> EmailService                    │
│                                                             │
│   - OrderProcessor USES EmailService                        │
│   - Doesn't own or store EmailService                       │
│   - Temporary usage (method parameter, local variable)      │
│                                                             │
│   Symbol: Dashed arrow (- - ->)                             │
└─────────────────────────────────────────────────────────────┘
```

```java
public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Sending email to " + to);
    }
}

public class OrderProcessor {
    // DEPENDENCY: uses EmailService but doesn't own it
    public void processOrder(Order order, EmailService emailService) {
        // Process the order...
        
        // Temporarily use EmailService
        emailService.sendEmail(
            order.getCustomerEmail(),
            "Order Confirmation",
            "Your order has been processed!"
        );
    }
}
```

### Comparison Chart

| Relationship | Strength | Lifecycle | Example |
|-------------|----------|-----------|---------|
| **Association** | Weakest | Independent | Teacher ↔ Student |
| **Aggregation** | Weak | Part survives | Department ◇→ Professor |
| **Composition** | Strong | Part dies with whole | House ◆→ Room |
| **Dependency** | Temporary | No lifecycle | Processor ⇢ Service |

---

## 4. Cohesion and Coupling

### 4.1 Cohesion

> **Definition**: Cohesion measures how closely related and focused the responsibilities of a single module/class are.

**Goal: HIGH COHESION** - Each class should do one thing well.

```
┌─────────────────────────────────────────────────────────────┐
│                    COHESION                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   LOW COHESION (Bad) ❌         HIGH COHESION (Good) ✓      │
│   ┌──────────────────┐         ┌──────────────────┐         │
│   │    Employee      │         │    Employee      │         │
│   ├──────────────────┤         ├──────────────────┤         │
│   │ - name           │         │ - name           │         │
│   │ - email          │         │ - email          │         │
│   │ + calculatePay() │         │ - department     │         │
│   │ + sendEmail()    │         │ + getDetails()   │         │
│   │ + generateReport()│        └──────────────────┘         │
│   │ + saveToDatabase()│                                     │
│   │ + validateTax()  │         ┌──────────────────┐         │
│   └──────────────────┘         │  PayrollService  │         │
│                                │ + calculatePay() │         │
│   This class does              └──────────────────┘         │
│   too many things!                                          │
│                                ┌──────────────────┐         │
│                                │   EmailService   │         │
│                                │ + sendEmail()    │         │
│                                └──────────────────┘         │
│                                                             │
│                                Each class has ONE focus     │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Low vs High Cohesion

```java
// ❌ LOW COHESION - God class doing everything
public class Employee {
    private String name;
    private double salary;
    private String email;
    
    // Employee data management - OK
    public String getName() { return name; }
    
    // ❌ Payroll calculation - not employee's responsibility
    public double calculateMonthlyPay() {
        return salary / 12;
    }
    
    // ❌ Email sending - not employee's responsibility
    public void sendPayslip() {
        // Email logic here...
    }
    
    // ❌ Database operations - not employee's responsibility
    public void saveToDatabase() {
        // JDBC code here...
    }
    
    // ❌ Tax calculation - not employee's responsibility
    public double calculateTax() {
        // Tax logic here...
    }
    
    // ❌ Report generation - not employee's responsibility
    public String generatePerformanceReport() {
        // Report logic here...
    }
}

// ✓ HIGH COHESION - Single responsibility classes
public class Employee {
    private String id;
    private String name;
    private String email;
    private String department;
    
    public Employee(String id, String name, String email, String department) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.department = department;
    }
    
    // Only employee data related methods
    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getDepartment() { return department; }
}

public class PayrollService {
    public double calculateMonthlyPay(Employee employee, double annualSalary) {
        return annualSalary / 12;
    }
    
    public double calculateTax(Employee employee, double income) {
        // Tax calculation logic
        return income * 0.3;
    }
}

public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        // Email sending logic
    }
    
    public void sendPayslip(Employee employee, double amount) {
        sendEmail(employee.getEmail(), "Your Payslip", "Amount: $" + amount);
    }
}

public class EmployeeRepository {
    public void save(Employee employee) {
        // Database save logic
    }
    
    public Employee findById(String id) {
        // Database read logic
        return null;
    }
}
```

### 4.2 Coupling

> **Definition**: Coupling measures how dependent modules/classes are on each other.

**Goal: LOW COUPLING** - Classes should be independent and interact through well-defined interfaces.

```
┌─────────────────────────────────────────────────────────────┐
│                    COUPLING                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   TIGHT COUPLING (Bad) ❌       LOOSE COUPLING (Good) ✓     │
│                                                             │
│   ┌─────────┐                   ┌─────────┐                 │
│   │ ClassA  │──────────────────▶│ ClassB  │                 │
│   └─────────┘                   └─────────┘                 │
│        │                             ▲                      │
│        │                             │                      │
│        │                    ┌────────┴────────┐             │
│        │                    │   Interface     │             │
│        │                    └────────┬────────┘             │
│        │                             │                      │
│        │                    ┌────────┴────────┐             │
│        └───────────────────▶│ Implementation  │             │
│                             └─────────────────┘             │
│                                                             │
│   Direct dependency          Dependency on abstraction      │
│   Hard to change B           Easy to swap implementations   │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - Tight vs Loose Coupling

```java
// ❌ TIGHT COUPLING - Direct dependency on concrete class
public class OrderService {
    private MySQLDatabase database;  // Direct dependency!
    
    public OrderService() {
        this.database = new MySQLDatabase();  // Creates its own dependency
    }
    
    public void saveOrder(Order order) {
        database.connect();
        database.execute("INSERT INTO orders...");
        database.disconnect();
    }
}

// Problems:
// 1. Cannot use different database without changing OrderService
// 2. Hard to test - cannot mock the database
// 3. OrderService knows too much about MySQLDatabase internals

// ✓ LOOSE COUPLING - Dependency on abstraction
public interface Database {
    void connect();
    void execute(String query);
    void disconnect();
}

public class MySQLDatabase implements Database {
    @Override
    public void connect() { /* MySQL specific */ }
    @Override
    public void execute(String query) { /* MySQL specific */ }
    @Override
    public void disconnect() { /* MySQL specific */ }
}

public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() { /* PostgreSQL specific */ }
    @Override
    public void execute(String query) { /* PostgreSQL specific */ }
    @Override
    public void disconnect() { /* PostgreSQL specific */ }
}

public class OrderService {
    private Database database;  // Depends on abstraction!
    
    // Dependency injection
    public OrderService(Database database) {
        this.database = database;
    }
    
    public void saveOrder(Order order) {
        database.connect();
        database.execute("INSERT INTO orders...");
        database.disconnect();
    }
}

// Usage - Easy to swap implementations
public class Main {
    public static void main(String[] args) {
        // Production
        Database db = new MySQLDatabase();
        OrderService orderService = new OrderService(db);
        
        // Testing
        Database mockDb = new MockDatabase();  // Test double
        OrderService testService = new OrderService(mockDb);
    }
}
```

### Why Cohesion and Coupling Matter

| Aspect | High Cohesion | Low Coupling |
|--------|---------------|--------------|
| **Maintainability** | Easy to understand one class | Changes don't ripple |
| **Testability** | Easy to test in isolation | Easy to mock dependencies |
| **Reusability** | Class can be reused | Components are independent |
| **Flexibility** | Clear responsibilities | Easy to swap implementations |

---

## 5. Interfaces vs Abstract Classes

### Interface

> **Definition**: A contract that defines WHAT a class must do, without specifying HOW.

**Characteristics**:
- All methods are abstract (until Java 8 default methods)
- Cannot have instance variables (only constants)
- A class can implement multiple interfaces
- Defines a capability or behavior

### Abstract Class

> **Definition**: A partially implemented class that provides a common base for subclasses.

**Characteristics**:
- Can have both abstract and concrete methods
- Can have instance variables with any access modifier
- A class can extend only one abstract class
- Defines a common type/identity

```
┌─────────────────────────────────────────────────────────────┐
│              INTERFACE vs ABSTRACT CLASS                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   INTERFACE                    ABSTRACT CLASS               │
│   «interface»                  {abstract}                   │
│   ┌─────────────┐              ┌─────────────────┐          │
│   │  Flyable    │              │     Animal      │          │
│   ├─────────────┤              ├─────────────────┤          │
│   │ + fly()     │              │ # name: String  │          │
│   └─────────────┘              │ + eat()         │          │
│                                │ + sleep() {abs} │          │
│   WHAT it can do               └─────────────────┘          │
│   (capability)                 WHAT it IS                   │
│                                (identity)                   │
│                                                             │
│   Bird implements Flyable      Dog extends Animal           │
│   Plane implements Flyable     Cat extends Animal           │
│   Superman implements Flyable                               │
│                                                             │
│   Multiple inheritance: YES    Multiple inheritance: NO     │
│   State: NO (only constants)   State: YES                   │
│   Constructors: NO             Constructors: YES            │
└─────────────────────────────────────────────────────────────┘
```

#### Code Example - When to Use Interface

```java
// Interface for CAPABILITY - can be implemented by unrelated classes
public interface Drawable {
    void draw();
}

public interface Resizable {
    void resize(double factor);
}

public interface Clickable {
    void onClick();
}

// A shape that can be drawn, resized, and clicked
public class Circle implements Drawable, Resizable, Clickable {
    private double radius;
    
    @Override
    public void draw() {
        System.out.println("Drawing circle with radius " + radius);
    }
    
    @Override
    public void resize(double factor) {
        this.radius *= factor;
    }
    
    @Override
    public void onClick() {
        System.out.println("Circle clicked!");
    }
}

// A button that can be drawn and clicked (not resizable)
public class Button implements Drawable, Clickable {
    private String label;
    
    @Override
    public void draw() {
        System.out.println("Drawing button: " + label);
    }
    
    @Override
    public void onClick() {
        System.out.println("Button clicked: " + label);
    }
}
```

#### Code Example - When to Use Abstract Class

```java
// Abstract class for IDENTITY - defines what something IS
public abstract class Vehicle {
    // State that all vehicles share
    protected String brand;
    protected String model;
    protected int year;
    protected double fuelLevel;
    
    // Common constructor
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.fuelLevel = 100.0;
    }
    
    // Concrete method - common implementation
    public void refuel(double amount) {
        this.fuelLevel = Math.min(100.0, fuelLevel + amount);
        System.out.println(brand + " refueled to " + fuelLevel + "%");
    }
    
    // Concrete method
    public String getInfo() {
        return year + " " + brand + " " + model;
    }
    
    // Abstract methods - must be implemented by subclasses
    public abstract void start();
    public abstract void stop();
    public abstract double calculateFuelConsumption(double distance);
}

// Concrete subclass
public class Car extends Vehicle {
    private int numberOfDoors;
    
    public Car(String brand, String model, int year, int numberOfDoors) {
        super(brand, model, year);  // Call parent constructor
        this.numberOfDoors = numberOfDoors;
    }
    
    @Override
    public void start() {
        System.out.println(brand + " car engine started");
    }
    
    @Override
    public void stop() {
        System.out.println(brand + " car engine stopped");
    }
    
    @Override
    public double calculateFuelConsumption(double distance) {
        return distance * 0.08;  // 8 liters per 100km
    }
}

public class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    
    public Motorcycle(String brand, String model, int year, boolean hasSidecar) {
        super(brand, model, year);
        this.hasSidecar = hasSidecar;
    }
    
    @Override
    public void start() {
        System.out.println(brand + " motorcycle engine started with a roar!");
    }
    
    @Override
    public void stop() {
        System.out.println(brand + " motorcycle engine stopped");
    }
    
    @Override
    public double calculateFuelConsumption(double distance) {
        return distance * 0.04;  // 4 liters per 100km
    }
}
```

### Decision Guide: Interface vs Abstract Class

| Question | If YES → | If NO → |
|----------|----------|---------|
| Do unrelated classes need this behavior? | Interface | Abstract Class |
| Do you need multiple inheritance? | Interface | Either |
| Do you need to provide common state? | Abstract Class | Interface |
| Do you need constructors? | Abstract Class | Interface |
| Is it a "CAN-DO" relationship? | Interface | Abstract Class |
| Is it an "IS-A" relationship? | Abstract Class | Interface |

#### Interview Tip 💡

```
Use INTERFACE when:
- Defining a contract/capability
- Multiple classes need the same behavior
- You want to achieve multiple inheritance
- Example: Comparable, Serializable, Runnable

Use ABSTRACT CLASS when:
- You have a common base with shared code
- Subclasses share state (fields)
- You're defining a class hierarchy
- Example: Animal, Vehicle, Shape
```

---

## 6. Practice Exercises

### Exercise 1: Design a Media Player

Design classes for a media player that can play different types of media.

**Requirements**:
- Support for Audio, Video, and Podcast
- Each media type has: play(), pause(), stop()
- Audio has volume, Video has resolution
- All media has title, duration, and artist

<details>
<summary>Click to see solution</summary>

```java
// Interface for playable behavior
public interface Playable {
    void play();
    void pause();
    void stop();
}

// Abstract base class for common media properties
public abstract class Media implements Playable {
    protected String title;
    protected int duration; // in seconds
    protected String artist;
    protected boolean isPlaying;
    
    public Media(String title, int duration, String artist) {
        this.title = title;
        this.duration = duration;
        this.artist = artist;
        this.isPlaying = false;
    }
    
    @Override
    public void play() {
        isPlaying = true;
        System.out.println("Playing: " + title);
    }
    
    @Override
    public void pause() {
        isPlaying = false;
        System.out.println("Paused: " + title);
    }
    
    @Override
    public void stop() {
        isPlaying = false;
        System.out.println("Stopped: " + title);
    }
    
    public abstract String getMediaInfo();
}

// Concrete classes
public class Audio extends Media {
    private int volume;
    
    public Audio(String title, int duration, String artist) {
        super(title, duration, artist);
        this.volume = 50;
    }
    
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
    }
    
    @Override
    public String getMediaInfo() {
        return "Audio: " + title + " by " + artist + " [" + duration + "s]";
    }
}

public class Video extends Media {
    private String resolution;
    
    public Video(String title, int duration, String artist, String resolution) {
        super(title, duration, artist);
        this.resolution = resolution;
    }
    
    @Override
    public void play() {
        super.play();
        System.out.println("Resolution: " + resolution);
    }
    
    @Override
    public String getMediaInfo() {
        return "Video: " + title + " [" + resolution + "]";
    }
}
```
</details>

### Exercise 2: Identify Relationships

For each pair, identify the relationship type:

1. Library and Book
2. Order and Customer
3. Car and Engine
4. Employee and Department
5. Logger and FileWriter

<details>
<summary>Click to see answers</summary>

1. **Library and Book**: Aggregation (Books exist independently, can be in multiple libraries)
2. **Order and Customer**: Association (Both exist independently, Order references Customer)
3. **Car and Engine**: Composition (Engine is created with and destroyed with Car)
4. **Employee and Department**: Aggregation (Employee can exist without Department)
5. **Logger and FileWriter**: Dependency (Logger uses FileWriter temporarily)

</details>

### Exercise 3: Refactor for High Cohesion

Refactor this low-cohesion class:

```java
public class User {
    private String name;
    private String email;
    
    public void save() { /* DB logic */ }
    public void sendEmail(String message) { /* Email logic */ }
    public boolean validate() { /* Validation logic */ }
    public String generateReport() { /* Report logic */ }
    public void notify(String message) { /* Push notification */ }
}
```

<details>
<summary>Click to see solution</summary>

```java
// High cohesion - each class has single responsibility
public class User {
    private String name;
    private String email;
    
    public String getName() { return name; }
    public String getEmail() { return email; }
}

public class UserRepository {
    public void save(User user) { /* DB logic */ }
    public User findByEmail(String email) { /* DB logic */ }
}

public class UserValidator {
    public boolean validate(User user) { /* Validation logic */ }
}

public class EmailService {
    public void sendEmail(String to, String message) { /* Email logic */ }
}

public class NotificationService {
    public void notify(User user, String message) { /* Push notification */ }
}

public class UserReportGenerator {
    public String generateReport(User user) { /* Report logic */ }
}
```
</details>

---

## 📚 Key Takeaways

1. **OOP Pillars**: Abstraction, Encapsulation, Inheritance, Polymorphism
2. **Relationships**: Know the difference between Association, Aggregation, Composition, and Dependency
3. **High Cohesion**: Each class should have a single, well-defined responsibility
4. **Low Coupling**: Depend on abstractions, not concrete implementations
5. **Interface vs Abstract Class**: Interface for capabilities, Abstract Class for identity

---

**Next Section: [SOLID Principles](../02-solid-principles/README.md)** →
