---
title: "Methods Common to All Objects: Effective Java"
topic: java
source_pdf: pdf/java/methods-common-to-all-objects-effective-java.pdf
review_needed: true
review_pages_indentation_rebuilt: [2, 3, 4, 5, 6, 7, 8]
review_notes: "No images in this PDF. Code listings rebuilt from PDF text positions. The equals() rules table (p2) and the compareTo() contract table (p7) were rebuilt from the rendered pages. The PDF title is \"Methods Common to All Objects\" Effective Java."
---

# "Methods Common to All Objects" Effective Java.

Ce chapitre traite des **méthodes héritées de `java.lang.Object`** , qu’on doit bien maîtriser et **souvent redéfinir** ( `equals` , `hashCode` , `toString` , etc.).

→ Obey the general contract when overriding `equals`

Goal:

If you override `equals()` , you **must respect its contract**, otherwise your objects behave incorrectly in collections like `Set` , `Map` , etc.

When should you override `equals()` ?

Only if:

- Your class has **logical equality** (two objects are "equal" if their content is the same).
- Examples: `User` , `Point` , `Person` , `Email` .

Don’t override if:

- Your class is meant for **unique instances only** (like singletons or utility classes).
- The class doesn’t need logical equality (e.g. `Thread` , `DatabaseConnection` ).

`equals()` must follow 5 rules (VERY IMPORTANT):

| Rule | Meaning |
|---|---|
| **Reflexive** | `x.equals(x)` must be `true` . |
| **Symmetric** | If `x.equals(y)` is `true` , then `y.equals(x)` must also be `true` . |
| **Transitive** | If `x.equals(y)` and `y.equals(z)` are `true` , then `x.equals(z)` must also be `true` . |
| **Consistent** | Multiple calls should return the same result if nothing changes. |
| **Non-nullity** | `x.equals(null)` must be `false` (never throw `NullPointerException` ). |

Bad `equals()` example:

```java
public class Point {
    public int x, y;

    public boolean equals(Point other) { // ❌ not overriding Object.equals!
        return this.x == other.x && this.y == other.y;
    }
}
```

This doesn't actually override `Object.equals(Object)` because the parameter isn’t `Object` .

Correct `equals()` implementation:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Point)) return false;
    Point p = (Point) o;
    return x == p.x && y == p.y;
}
```

Explanation:

- `this == o` : check if they’re the same object.
- `instanceof` : check if `o` is the right type.
- `(Point) o` : safe cast.
- Then compare the fields.

```java
Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);

System.out.println(p1.equals(p2)); // true
```

But if you don’t override `equals()` , this will print `false` — because `Object.equals()` only checks reference equality (same memory address).

Why does `equals()` return false if not overridden?

Because by default, `Object.equals()` is defined like this (simplified):

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

What does `this == obj` mean?

It means: Are both references pointing to the exact same object in memory?

## Always override `hashCode` when you override `equals`

If you override `equals()` but don’t override `hashCode()` , your objects will behave incorrectly in hash-based collections like:

- `HashMap`
- `HashSet`
- `Hashtable`

Problem example:

```java
Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);

Set<Point> set = new HashSet<>();
set.add(p1);

System.out.println(set.contains(p2)); // ❌ false if hashCode() not overridden
```

Even though `p1.equals(p2)` is `true` , `HashSet` uses `hashCode()` to find the bucket, and if `p1` and `p2` have different hash codes, it won’t find the match.

Contract between `equals()` and `hashCode()` :

- If two objects are equal (via `equals()` ), they **must** have the same hash code.
- If two objects are not equal, they can still have the same hash code (but ideally not too often).

```java
@Override
    public int hashCode() {
        return Objects.hash(x, y); // or custom formula
    }
```

Now works correctly:

```java
Set<Point> set = new HashSet<>();
set.add(new Point(1, 2));

System.out.println(set.contains(new Point(1, 2))); //  true
```

When to override both `equals()` and `hashCode()`

You should override both when:

1. You care about logical equality (objects are equal if their contents are equal).
2. Your objects will be stored in collections like `HashSet` , `HashMap` , `Hashtable` , etc.
3. Your class is a value object (like `User` , `Point` , `Email` , `Product` , etc.)

> Java collections like `HashMap` and `HashSet` use `hashCode()` to find where to store or find your object.
> If you override `equals()` but not `hashCode()` , the collection may not find your object even if it’s “equal” — because it looks in the wrong hash bucket.
> Use `record` in Java 14+ → auto-generates both `equals()` and `hashCode()` .

Override only `equals()` when:

Rare — but possible when:

- You compare logically, but your object is never used in a hash-based collection.
- You’re certain the object won’t be used as key or in `Set` .

Not recommended unless you're 100% sure.

## Why override `toString()` ?

The default `toString()` method from `Object` looks like this:

```
User@3e25a5
```

That’s useless for logging, debugging, or printing your object.

> If you override `toString()` , your object becomes readable, debuggable, and more useful.

Good `toString()` = easy to read

Example:

```java
public class User {
    private String name;
    private String email;

    @Override
    public String toString() {
        return "User{name='" + name + "', email='" + email + "'}";
    }
}
```

Output:

```
User{name='Houda', email='houda@gmail.com'}
```

Much better than: `User@1a2b3c4`

When is `toString()` used automatically?

- In `System.out.println(object)`
- In logs
- In debugger views
- In string concatenation: `"User is " + user`

→ Behind the scenes, Java does this:

```java
System.out.println(user.toString());
```

## Override `clone()` judiciously

- `clone()` is a method from `java.lang.Object`
- It creates a **copy of an object** in memory
- It’s part of the `Cloneable` interface

But there’s a problem…

> The default `clone()` is **dangerous and confusing**:

- It makes a **shallow copy** (not deep)
- Throws `CloneNotSupportedException`
- Requires extra work to be done correctly
- Breaks easily in complex object graphs

That’s why **Joshua Bloch** recommends **avoiding `clone()`** in most cases and using **copy constructors or static factory methods** instead.

OK, but when should I override `clone()` ?

Only if:

- You absolutely need a performance-optimized copy
- You have a simple object, or
- You’re working with a legacy system that expects `clone()`

How to correctly override `clone()` :

1. Implement `Cloneable`
2. Override `clone()` and call `super.clone()`
3. Cast the result and copy mutable fields carefully

```java
public class User implements Cloneable {
    private String name;
    private List<String> roles;

    @Override
    public User clone() {
        try {
            User cloned = (User) super.clone();
            // Deep copy the mutable list
            cloned.roles = new ArrayList<>(this.roles);
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // should never happen
        }
    }
}
```

Bad clone (shallow copy):

```java
cloned.roles = this.roles; // ❌ same list in both objects
```

→ Any change to `cloned.roles` will affect the original.

Better Alternative: Use a copy constructor or static factory

Copy constructor:

```java
public User(User other) {
    this.name = other.name;
    this.roles = new ArrayList<>(other.roles);
}
```

Static factory:

```java
public static User copyOf(User other) {
    return new User(other);
}
```

Easier to maintain, works well with `final` fields, avoids `Cloneable` mess.

## Consider implementing `Comparable`

`Comparable` is an interface that lets you define the **default (natural) sorting order** for your class.

sort User by name

```java
public class User implements Comparable<User> {
    private String name;

    @Override
    public int compareTo(User other) {
        return this.name.compareTo(other.name);
    }
}
```

How it works:

- `compareTo()` returns:
  - `< 0` if `this` is less than `other`
  - `0` if they’re equal
  - `> 0` if `this` is greater than `other`

### `compareTo()` contract (very important!)

Just like `equals()` and `hashCode()` , `compareTo()` must respect some rules:

| Rule | Example |
|---|---|
| **Consistent with equals** | If `a.equals(b)` , then `a.compareTo(b) == 0` |
| **Transitive** | If `a > b` and `b > c` , then `a > c` |
| **Anti-symmetric** | If `a < b` , then `b > a` |

Reason: `compareTo()` is a method on objects, not primitives

If `age` is declared as:

```java
private int age; // primitive type
```

Then `this.age.compareTo(other.age)` will **not compile**, because:

- `int` is not an object
- It has no methods, including `compareTo`

So why `Integer.compare(...)` works?

Because `Integer.compare(int x, int y)` is a **static method** introduced in Java 7+, and works for primitives:

```java
return Integer.compare(this.age, other.age); // safe for ints
```

## Comparator

`Comparable` defines a single “natural” order (e.g., by name).
But sometimes, you want to sort your objects in different ways.

That’s where `Comparator` comes in it allows **custom sorting logic** without modifying the class itself.

`Comparator` is an interface that lets you define **custom sorting rules** — even multiple orders.

It contains:

```java
int compare(T o1, T o2);
```

You don’t modify the original class. You create the sort logic outside it.

How to use `Comparator`

Option 1: Use `Comparator.comparing(...)`

```java
List<User> users = ...

// Sort by name
users.sort(Comparator.comparing(User::getName));

// Sort by age
users.sort(Comparator.comparingInt(User::getAge));

```

Option 2: Chain Comparators

```java
// Sort by age, then by name
users.sort(
    Comparator.comparingInt(User::getAge)
              .thenComparing(User::getName)
);
```
