# Comparable vs Comparator

## 1. Comparable

## Package: java.lang

## Interface: Comparable<T>

## Method: int compareTo(T o)

```
Purpose: Defines the natural ordering of objects.
When to use: If you want a class to have a default sorting logic.
```

## Limitation: A class can only implement one compareTo method, so it can have only one natural order.

Example

Usage:

## 2. Comparator

## Package: java.util

## Interface: Comparator<T>

## Method: int compare(T o1, T o2)

```
Purpose: Defines a custom ordering (can have multiple strategies).
When to use: If you want different sorting logics for the same class.
Flexibility: You can create as many comparators as you like.
```

Example

Usage:

```
1 class User implements Comparable<User> {
2 private int id;
3 private String name;
4
5 @Override
6 public int compareTo(User other) {
7 return Integer.compare(this.id, other.id); // sort by id
8 }
9 }
10
```

```
1 List<User> users = ...
2 Collections.sort(users); // uses compareTo()
3
```

```
1 class UserNameComparator implements Comparator<User> {
2 @Override
3 public int compare(User u1, User u2) {
4 return u1.getName().compareTo(u2.getName()); // sort by name
5 }
6 }
7
```

```
1 List<User> users = ...
2 Collections.sort(users, new UserNameComparator()); // uses compare()
```

### 3. Key Differences Table

✅ Rule of thumb:

```
Use Comparable when there is one natural order (e.g., numbers, dates, alphabetical).
Use Comparator when you need multiple sorting strategies (e.g., sort Users by age, then by name).
```

```
3
```

## Package java.lang java.util

## Method compareTo(T

## o)

## compare(T o1,

## T o2)

```
Ordering Natural (default) Custom (multiple
possible)
Implementation Implemented by the
class itself
```

```
Implemented as
separate class/lambda
Number allowed Only one per class Multiple comparators
per class
```

```
Aspect Comparable Comparator
```
