---
title: "Immutabilité"
topic: java
source_pdf: pdf/java/immutabilite.pdf
review_needed: true
review_pages_transcribed_from_images: [1]
review_pages_diagrams: [3, 4, 5]
review_pages_indentation_rebuilt: [1, 2, 3, 4]
review_notes: "Code listings rebuilt from PDF text positions (exact indentation, wrapped comments re-joined). Page 1 Mutable/Immutable Point code is a screenshot; the last two lines of the right-hand box (Point p = new Point(5,3); p.x = 99;) were read at low resolution."
---

# Immutabilité

## What is immutability?

Mutable — state can change after creation:

```java
class Point {
  int x, y;
}
Point p = new Point();
p.x = 5;   // anyone can change
p.x = 99;  // again, anytime
```

Immutable — state locked after construction:

```java
final class Point {
  final int x, y;
  Point(int x,int y){
    this.x=x;this.y=y;
  }
}
Point p = new Point(5,3);
p.x = 99; // compile error!
```

- ✓ Thread-safe by default — no synchronization needed
- ✓ Safe to share freely — no defensive copying for caller
- ✓ Perfect HashMap key — hashCode never changes
- ✓ Easier to reason about — state is always predictable
- ✓ Caching friendly — String pool, Integer cache

> "An immutable object's state cannot change after construction — it is inherently thread-safe and freely shareable."

## The 5 rules to make a class truly immutable

```java
public final class Money {       // RULE 1: final class — no subclassing

    private final double amount;  // RULE 2: all fields private + final
    private final String currency;

    public Money(double a, String c) {
        this.amount   = a;          // RULE 3: assign only in constructor
        this.currency = c;
    }
                                   // RULE 4: no setters — ever
    public double getAmount()   { return amount; }
    public String getCurrency() { return currency; }

    // RULE 5: return NEW object instead of mutating
    public Money add(double extra) {
        return new Money(amount + extra, currency);
    }
}

Money m1 = new Money(100, "EUR");
Money m2 = m1.add(50);
// m1 → still 100 EUR — untouched
// m2 → new object 150 EUR
```

FC · PF · CO · NS · RN
Final Class · Private Final · Constructor Only · No Setters · Return New

## Most common interview trap — mutable object inside

```java
public final class Team {
    private final String       name;
    private final List<String> members; // final ref, mutable content!

    public Team(String name, List<String> members) {
        this.name    = name;
        this.members = members;      // stores the original list — WRONG
    }

    public List<String> getMembers() {
        return members;              // exposes mutable list — WRONG
    }
}

List<String> list = new ArrayList<>(List.of("Ali", "Sara"));
Team team = new Team("Dev", list);
list.add("Hacker");                // pollutes team.members!
team.getMembers().add("Hacker2"); // also works — object broken!
```

## Fix — defensive copy in AND out

```java
public final class Team {
    private final String       name;
    private final List<String> members;

    public Team(String name, List<String> members) {
        this.name    = name;
        this.members = List.copyOf(members); // copy on input ✓
    }

    public List<String> getMembers() {
        return List.copyOf(members);        // copy on output ✓
    }
}

list.add("Hacker");               // ignored — team has its own copy
team.getMembers().add("Hacker2"); // UnsupportedOperationException!
```

"final on a reference prevents reassigning the pointer — it does NOT prevent mutating the object it points to. That's why defensive copying is required."

## Famous immutable classes

```java
// String — immutable since Java 1.0
String s = "hello";
s.toUpperCase();     // returns NEW String — s unchanged
s.replace("h","j");  // returns NEW String
System.out.println(s); // still "hello"

// String pool — safe because String is immutable
String a = "Java";
String b = "Java";
a == b; // true — same object in pool

// Integer cache — -128 to 127
Integer x = 100; Integer y = 100;
x == y; // true — cached
Integer p = 200; Integer q = 200;
p == q; // false — outside cache range!
```

[diagram: immutable in JDK vs mutable — be careful]

> Integer est immutable — sa valeur ne peut pas changer après création. Le cache -128/127 est une optimisation mémoire : Java réutilise le même objet pour éviter d'en créer des milliers. C'est pourquoi `==` retourne `true` dans ce range — même objet — mais c'est un détail d'implémentation, pas une règle d'immutabilité.Integer est TOUJOURS immutable — peu importe la valeur
>
> Ce que le cache -128/127 change, c'est l'identité de l'objet — pas l'immutabilité

```java
// Dans le cache — MÊME objet en mémoire
Integer x = 100;
Integer y = 100;
x == y;      // true  → même référence (objet réutilisé)
x.equals(y); // true  → même valeur

// Hors cache — deux objets DIFFÉRENTS en mémoire
Integer p = 200;
Integer q = 200;
p == q;      // false → deux objets distincts
p.equals(q); // true  → même valeur
```

## Java 16 Records — immutability built-in

```java
// Record = immutable class, zero boilerplate
record Point(int x, int y) {}

// Compiler generates automatically:
public final class Point {           // final class ✓
    private final int x, y;          // private final ✓
    public Point(int x, int y) { ... } // constructor ✓
    public int x() { return x; }     // accessors ✓
    public int y() { return y; }
    // + equals, hashCode, toString ✓
}
```

⚠️ Record trap — same List problem applies!

```java
record Team(String name, List<String> members) {}

Team t = new Team("Dev", new ArrayList<>(List.of("Ali")));
t.members().add("Hacker"); // works! record NOT deeply immutable

// Fix — compact constructor
record Team(String name, List<String> members) {
    Team {                              // compact constructor
        members = List.copyOf(members); // seal the list ✓
    }
}
t.members().add("Hacker"); // UnsupportedOperationException — safe!
```

> Records handle shallow immutability automatically. Deep immutability (mutable fields) is still YOUR responsibility.

## Why immutability = automatic thread safety

```java
// Mutable shared state — race condition
class Counter {
    int count = 0;
    void increment() { count++; } // NOT thread-safe
}
// Thread A reads 5, Thread B reads 5
// Thread A writes 6, Thread B writes 6
// Expected 7 — Got 6 — silent data loss!
```

```java
// Immutable — no shared mutable state = no race condition
final class Counter {
    private final int count;
    Counter(int c) { this.count = c; }

    Counter increment() {
        return new Counter(count + 1); // new object each time
    }
    int value() { return count; }
}
// Thread A and B only READ — nothing to synchronize!
```

> This is exactly why `String` is shared safely across threads — nothing can ever change it after creation.
>
> Immutable objects are thread-safe because threads can only read them — there is no write to synchronize.

[diagram: shallow vs deep immutability]

[diagram: summary of the 5 rules, JDK examples, pitfalls and benefits]
