---
title: "Optional"
topic: java
source_pdf: pdf/java/optional.pdf
review_needed: true
review_pages_diagrams: [4]
review_pages_indentation_rebuilt: [1, 2, 3, 4]
review_notes: "Code listings rebuilt from PDF text positions (exact indentation). The cheat-sheet image at the end of page 4 is replaced by a placeholder."
---

# Optional

> "Optional is a container type for return values that forces callers to handle the absent case — it replaces null with an explicit API contract."

## Without Optional — NPE waiting to happen

```java
// findUser returns null if not found
User u = db.findUser(id);
u.getName(); // NullPointerException!

// You must remember to check every time
if (u != null) {
  u.getName();
}
```

## With Optional — null is part of the type

```java
// return type SAYS it might be absent
Optional<User> u = db.findUser(id);

// compiler forces you to handle absence
u.ifPresent(user ->
  System.out.println(user.getName())
);
```

> Optional = boîte qui force à gérer l'absence.

## 3 ways to create an Optional

```java
// 1. of() — value is GUARANTEED non-null
//    throws NullPointerException if you pass null!
Optional<String> a = Optional.of("hello");
Optional<String> x = Optional.of(null);    // NPE!

// 2. ofNullable() — value MAY be null
//    wraps it if present, returns empty if null
Optional<String> b = Optional.ofNullable(maybeNull);

// 3. empty() — explicitly absent
Optional<String> c = Optional.empty();
```

> use of when you KNOW it's not null (e.g. a constant). Use ofNullable when wrapping values from legacy code or external APIs.

## Getting the value out — safe vs unsafe

```java
Optional<String> opt = Optional.of("Java");
Optional<String> empty = Optional.empty();

// ✓ get() — UNSAFE, throws if empty
opt.get();      // "Java"
empty.get();    // NoSuchElementException!

// ✓ orElse() — default value (always evaluated)
empty.orElse("default");    // "default"

// ✓ orElseGet() — lazy supplier (only called if empty)
empty.orElseGet(() -> compute()); // cheaper!

// ✓ orElseThrow() — throw custom exception
empty.orElseThrow(() ->
    new NotFoundException("not found"));

// ✓ ifPresent() — run action only if value exists
opt.ifPresent(v -> System.out.println(v));

// ✓ ifPresentOrElse() — Java 9+
opt.ifPresentOrElse(
    v  -> System.out.println(v),
    () -> System.out.println("absent")
);
```

> Never use `get()` without first checking `isPresent()` — defeats the purpose of Optional entirely.

## Transforming values — the functional core

```java
// map() — transform the value if present
//         wraps result back in Optional
Optional<String> name = Optional.of("java");

Optional<String> upper = name
    .map(String::toUpperCase);     // Optional["JAVA"]

Optional<Integer> len = name
    .map(String::length);          // Optional[4]

Optional<String> empty = Optional.<String>empty();
empty.map(String::toUpperCase);    // Optional.empty() — no NPE

─────────────────────────────────────────
// flatMap() — use when the mapper itself
//             returns an Optional
//             avoids Optional<Optional<T>>

class User {
    Optional<Address> getAddress() { ... }
}

// map gives Optional<Optional<Address>> — wrong!
user.map(User::getAddress);

// flatMap gives Optional<Address> — correct!
user.flatMap(User::getAddress)
    .map(Address::getCity)
    .orElse("unknown");
```

> Rule: if your mapper returns `Optional<X>` → use `flatMap` . If it returns `X` → use `map` .

## Filtering and chaining Optionals

```java
// filter() — keep value only if predicate passes
//            returns empty if predicate fails
Optional<Integer> age = Optional.of(17);

age.filter(a -> a >= 18);   // Optional.empty()
age.filter(a -> a >= 10);   // Optional[17]

─────────────────────────────────────────
// or() — Java 9+ fallback to another Optional
//        like orElseGet() but returns Optional
Optional<String> primary   = Optional.empty();
Optional<String> secondary = Optional.of("backup");

primary.or(() -> secondary);   // Optional["backup"]

─────────────────────────────────────────
// stream() — Java 9+ bridge to Stream API
//            0 or 1 element stream
List<Optional<String>> list = ...;

list.stream()
    .flatMap(Optional::stream)  // flatten all presents
    .collect(toList());
```

## Real-world chaining — the power of Optional

```java
// Goal: get city of user's address, or "unknown"

// Old Java 8 style (nested null checks)
String city = "unknown";
if (user != null) {
    Address addr = user.getAddress();
    if (addr != null) {
        String c = addr.getCity();
        if (c != null) city = c;
    }
}

─────────────────────────────────────────
// With Optional — clean pipeline
String city = Optional.ofNullable(user)
    .flatMap(u  -> u.getAddress())   // Optional<Address>
    .map(addr   -> addr.getCity())   // Optional<String>
    .filter(c   -> !c.isBlank())     // guard
    .orElse("unknown");             // default

─────────────────────────────────────────
// With Java 21 + record pattern (bonus)
record Address(String street, String city) {}
record User(String name, Address address)  {}

String city = findUser(id)
    .map(User::address)
    .map(Address::city)
    .orElseThrow(UserNotFoundException::new);
```

## What NOT to do

Do NOT use Optional as a field in a class — it is not Serializable

```java
class User {
    Optional<String> email; // wrong! not serializable
}
class User {
    String email; // nullable field is fine
    Optional<String> getEmail() { return Optional.ofNullable(email); }
}
```

Do NOT use Optional as a method parameter

```java
void send(Optional<String> email) { ... } // wrong
void send(String email) { ... }            // nullable param is cleaner
```

Do NOT use `isPresent()` + `get()` — use map/ifPresent instead

```java
if (opt.isPresent()) { return opt.get(); } // verbose
return opt.orElse(defaultValue);           // clean

// instead of this
if (opt.isPresent()) { send(opt.get()); }
// use this
opt.ifPresent(v -> send(v));

// instead of this
if (opt.isPresent()) { return opt.get().toUpperCase(); }
// use this
return opt.map(String::toUpperCase).orElse("default");
```

Optional is designed for **return types only** — to signal "this method might not find a result".

[diagram: Optional cheat sheet — create, transform, unwrap, Java 9+ methods]
