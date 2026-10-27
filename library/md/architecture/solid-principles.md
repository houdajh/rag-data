---
title: "SOLID principles"
topic: architecture
source_pdf: pdf/architecture/solid-principles.pdf
review_needed: true
review_pages_diagrams: [1]
review_pages_indentation_rebuilt: [1, 2, 3, 4]
review_notes: "All 4 code blocks rebuilt from PDF text positions; the Liskov example regained its lost 💥 emoji in a comment. The SOLID summary graphic on p1 is a placeholder."
---

# SOLID principles

[diagram: SOLID principles summary]

## S — Single Responsibility Principle

> A class should have only one reason to change — one job, one responsibility.

When a class handles too many things (logic, formatting, persistence), changes to one area break others. Split them apart so each class owns one clear concern.

## O — Open / Closed Principle

> Classes should be open for extension but closed for modification.

Instead of editing existing code every time you add new behaviour (risking bugs), design classes so you extend them via new subclasses or strategy objects.

```java
class DiscountCalculator {
    public double calculate(String type, double price) {
        // Adding a new discount type = editing THIS class ← problem!
        if (type.equals("student"))  return price * 0.8;
        if (type.equals("employee")) return price * 0.7;
        if (type.equals("vip"))      return price * 0.6;
        return price;
    }
}

//After

interface DiscountStrategy {
    double apply(double price);
}

// Each discount is its own class — no touching existing code
class StudentDiscount  implements DiscountStrategy {
    public double apply(double p) { return p * 0.8; }
}
class EmployeeDiscount implements DiscountStrategy {
    public double apply(double p) { return p * 0.7; }
}
class VipDiscount      implements DiscountStrategy {
    public double apply(double p) { return p * 0.6; }
}

class PriceCalculator {
    public double calculate(DiscountStrategy strategy, double price) {
        return strategy.apply(price);
    }
}
```

## L — Liskov Substitution Principle

> Subclasses must be usable wherever their parent class is expected — without breaking the program.

If you swap a parent for a child and something breaks or throws unexpectedly, you've violated LSP. A child class must honour all the contracts (behaviour, exceptions, return types) of its parent.

```java
class Bird {
    public void fly() {
        System.out.println("Flying!");
    }
}

// Penguin IS-A bird, but can't fly ← breaks substitution!
class Penguin extends Bird {
    public void fly() {
        throw new UnsupportedOperationException(
            "Penguins can't fly!"
        );
    }
}

// This call will crash at runtime:
Bird b = new Penguin();
b.fly(); // 💥 UnsupportedOperationException

//after

interface Bird    { String name(); }

interface FlyingBird extends Bird {
    void fly();
}

// Only birds that actually fly implement FlyingBird
class Sparrow implements FlyingBird {
    public String name() { return "Sparrow"; }
    public void fly()    { System.out.println("Flying!"); }
}

// Penguin is a Bird, but NOT a FlyingBird — honest!
class Penguin implements Bird {
    public String name() { return "Penguin"; }
    public void swim()   { System.out.println("Swimming!"); }
}
```

## I — Interface Segregation Principle

> Don't force a class to implement methods it doesn't need.

Fat interfaces that bundle unrelated methods force implementors to write dummy or throwing methods. Split large interfaces into smaller, focused ones so each class only implements what it actually uses.

```java
interface Worker {
    void work();
    void eat();    // ← robots don't eat!
    void sleep();  // ← robots don't sleep!
}

class RobotWorker implements Worker {
    public void work()  { System.out.println("Working"); }

    // Forced to implement meaningless methods:
    public void eat()  { throw new UnsupportedOperationException(); }
    public void sleep(){ throw new UnsupportedOperationException(); }
}

//after

// Split into focused interfaces
interface Workable  { void work(); }
interface Feedable  { void eat(); }
interface Sleepable { void sleep(); }

// Human uses all three
class HumanWorker implements Workable, Feedable, Sleepable {
    public void work()  { System.out.println("Working"); }
    public void eat()   { System.out.println("Eating"); }
    public void sleep() { System.out.println("Sleeping"); }
}

// Robot only implements what it needs
class RobotWorker implements Workable {
    public void work() { System.out.println("Working"); }
}
```

## D — Dependency Inversion Principle

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

When a class creates its own dependencies (`new MySQLDatabase()`), it's tightly coupled and hard to test or swap. Instead, inject abstractions so the class doesn't care about the concrete implementation.

```java
// Low-level: specific database
class MySQLDatabase {
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

// High-level: directly creates the low-level class ← tight coupling!
class UserService {
    private MySQLDatabase db = new MySQLDatabase(); // ← problem

    public void createUser(String name) {
        db.save(name); // Can't switch to MongoDB, PostgreSQL, etc.
    }
}

//after

// Abstraction — neither side knows the other
interface Database {
    void save(String data);
}

// Low-level modules implement the abstraction
class MySQLDatabase   implements Database {
    public void save(String d) { System.out.println("MySQL: "  + d); }
}
class MongoDatabase    implements Database {
    public void save(String d) { System.out.println("MongoDB: " + d); }
}

// High-level depends on the abstraction, injected from outside
class UserService {
    private final Database db; // ← interface, not a concrete class

    public UserService(Database db) { this.db = db; } // ← injected

    public void createUser(String name) { db.save(name); }
}

// Caller decides which database to use — easy to swap or mock
UserService svc = new UserService(new MongoDatabase());
```
