---
title: "Collection"
topic: java
source_pdf: pdf/java/collection.pdf
review_needed: true
review_pages_transcribed_from_images: [3, 6]
review_pages_diagrams: [1, 2, 3, 9]
review_notes: "Pages 1-3 and 9 are diagram / cheat-sheet screenshots, replaced by one-line placeholders (their box contents are images in the PDF, not text). The code on page 3 (ArrayList / LinkedList) and page 6 (Queue / Deque) was transcribed from images. The tables on pages 4, 5 and 7 were rebuilt from the rendered pages because the text extraction scrambled them."
---

# Collection

[diagram: Java collections hierarchy — Iterable, Collection (List, Set, Queue/Deque) and Map]

[diagram: List interface — ArrayList, LinkedList, Vector, with cache-locality note]

[diagram: Set — HashSet, LinkedHashSet, TreeSet]

[diagram: Queue / Deque — ArrayDeque, PriorityQueue, BlockingQueue]

[diagram: Map — HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap]

Stack is in the diagram — but hidden because it's **legacy and avoided**. Let me show you exactly where it fits and why :

```
java.util.Stack
    extends Vector
        extends AbstractList
            implements List
```

**Stack is technically a List**

---

## List — ordered, index-based, duplicates OK

ArrayList — backed by array:

```java
// Fast: get(i) → O(1) ✓
// Slow: add/remove middle → O(n)
// Memory: compact

List<String> list =
  new ArrayList<>();
list.add("Ali");
list.add("Sara");
list.get(0); // O(1)
```

LinkedList — doubly linked nodes:

```java
// Fast: add/remove head/tail O(1)
// Slow: get(i) → O(n) ✗
// Memory: node overhead

List<String> list =
  new LinkedList<>();
list.addFirst("Ali"); // O(1)
list.get(5);          // O(n)
```

**Other List facts**

```java
// List.of() — immutable, Java 9+
List<String> immutable = List.of("a", "b");
immutable.add("c"); // UnsupportedOperationException!

// Collections.unmodifiableList() — unmodifiable view
List<String> view = Collections.unmodifiableList(list);

// Sorting
Collections.sort(list);                               // natural order
list.sort(Comparator.comparing(String::length)); // custom
```

> Interview answer: "Use ArrayList by default. Use LinkedList only if you frequently add/remove from both ends — but even then, ArrayDeque is usually better."

**What is cache locality ?**

```
ArrayList — elements stored CONTIGUOUSLY in memory :
RAM: [ "Ali" | "Sara" | "Zak" | "Joe" ]
              addr 100 101 102 103
              ↑
              CPU loads a "cache line" = chunk of nearby memory at once
              → "Sara", "Zak", "Joe" already in cache → FAST

LinkedList — elements scattered RANDOMLY in memory :
RAM: "Ali" at addr 100 → pointer → "Sara" at addr 847
                                           → pointer → "Zak" at addr 203
             ↑
             CPU must follow each pointer to a random address
             → cache miss every time → much SLOWER
```

So even though LinkedList has O(1) add at head theoretically — in practice ArrayList is often faster because of cache locality. The CPU doesn't have to jump around in memory.

## Map — key → value, keys unique

| Implementation | Order | get/put | Key requirement |
|---|---|---|---|
| HashMap | ❌ none | O(1) | equals + hashCode |
| LinkedHashMap | ✓ insertion order | O(1) | equals + hashCode |
| TreeMap | ✓ sorted (natural) | O(log n) | Comparable or Comparator |

```java
// HashMap — most common
Map<String, Integer> map = new HashMap<>();
map.put("Ali", 30);
map.get("Ali");                 // 30
map.getOrDefault("X", 0); // 0 — no NPE
map.putIfAbsent("Ali", 99); // ignored — already exists
map.computeIfAbsent("B", k -> k.length()); // adds "B"→1

// Iterate entries
for (Map.Entry<String,Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " → " + e.getValue());
}
// or with streams
map.forEach((k, v) -> System.out.println(k + " → " + v));
```

> HashMap allows ONE null key and multiple null values. TreeMap does NOT allow null keys.

**How HashMap works internally**

```java
// When you do: map.put("Ali", 30)

// Step 1 — compute hash of key
int hash = "Ali".hashCode(); // e.g. 65328

// Step 2 — find bucket index
int index = hash % capacity; // e.g. index = 5

// Step 3 — store in bucket (linked list)
// If two keys → same bucket → collision!
// Java chains them in a linked list

// Step 4 — Java 8+ optimization
// If bucket size > 8 → convert to red-black tree
// get/put: O(1) avg, O(log n) worst case


// "Ali" → bucket 5
// "Mia" → also bucket 5 ← collision
// Java chains them:
// bucket[5] → ["Ali",30] → ["Mia",25]
// get("Mia") → go to bucket 5
//           → check equals("Mia") on each node
```

> If you override `equals()` without overriding `hashCode()` → two equal objects may land in different buckets → `get()` returns null even though the key exists!
>
> HashMap uses an array of buckets. Key is hashed to find the bucket. Collisions are handled by a linked list, converted to a red-black tree after 8 entries. Load factor 0.75 triggers resize.

## HashSet vs LinkedHashSet vs TreeSet

| Implementation | Order | add/contains | Null |
|---|---|---|---|
| HashSet | ❌ none | O(1) | ✓ one null |
| LinkedHashSet | ✓ insertion order | O(1) | ✓ one null |
| TreeSet | ✓ sorted | O(log n) | ❌ no null |

```java
// HashSet — backed by HashMap internally!
Set<String> set = new HashSet<>();
set.add("Ali");
set.add("Ali"); // ignored — duplicate
set.size();      // 1

// TreeSet — sorted automatically
Set<Integer> sorted = new TreeSet<>();
sorted.add(5); sorted.add(1); sorted.add(3);
System.out.println(sorted); // [1, 3, 5]

// Set operations — interview classic
Set<String> a = new HashSet<>(Set.of("a","b","c"));
Set<String> b = new HashSet<>(Set.of("b","c","d"));
a.retainAll(b); // intersection → [b, c]
a.addAll(b); // union {a, b, c, d}
a.removeAll(b); // difference
```

> HashMap stocke des paires clé → valeur.
>
> HashSet ne stocke que des clés — pas de valeurs.
>
> Donc comment Java implémente HashSet en utilisant HashMap ? Il invente une fausse valeur — un dummy

```java
// HashSet utilise les mêmes règles que HashMap pour les clés
// → equals() détermine si deux éléments sont "égaux"
// → hashCode() détermine dans quel bucket ils vont

Set<Point> set = new HashSet<>();
set.add(new Point(1, 2));
set.add(new Point(1, 2)); // doublon ? seulement si equals() + hashCode() corrects!

// Si tu n'override pas equals/hashCode → deux objets différents en mémoire
// → HashSet les considère DIFFÉRENTS → set.size() = 2 ← bug!
```

## Queue & Deque — Ordered processing

Queue — FIFO (first in, first out) and Deque — double-ended queue

```java
// Like a supermarket line
Queue<String> q =
    new LinkedList<>();
q.offer("Ali");   // add to tail
q.offer("Sara");
q.peek();  // "Ali" — see head
q.poll();  // "Ali" — remove head
// offer/peek/poll → no exception
// add/element/remove → throw exception
```

```java
// Stack AND Queue in one
Deque<String> dq =
    new ArrayDeque<>();
dq.addFirst("Ali");  // push front
dq.addLast("Sara");  // push back
dq.peekFirst(); // "Ali"
dq.pollLast();  // "Sara"

// Use as Stack:
dq.push("X");  // = addFirst
dq.pop();      // = removeFirst
```

**PriorityQueue — sorted by priority**

```java
// Natural order — smallest first
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5); pq.offer(1); pq.offer(3);
pq.poll(); // 1 — always the smallest!
pq.poll(); // 3

// Max heap — largest first
PriorityQueue<Integer> maxPQ =
 new PriorityQueue<>(Comparator.reverseOrder());
maxPQ.offer(5); maxPQ.offer(1); maxPQ.offer(3);
maxPQ.poll(); // 5
```

## Complexity

| Collection | get/search | add | remove | Notes |
|---|---|---|---|---|
| ArrayList | O(1) | O(1)* | O(n) | *amortized, worst O(n) resize |
| LinkedList | O(n) | O(1) ends | O(1) ends | O(n) middle |
| HashMap | O(1) | O(1) | O(1) | avg, O(n) worst collision |
| TreeMap | O(log n) | O(log n) | O(log n) | always sorted |
| HashSet | O(1) | O(1) | O(1) | backed by HashMap |
| TreeSet | O(log n) | O(log n) | O(log n) | backed by TreeMap |
| ArrayDeque | O(n) | O(1) ends | O(1) ends | best Stack/Queue |
| PriorityQueue | O(log n) | O(log n) | O(log n) | peek O(1) |

> ArrayList vs LinkedList?" → ArrayList for random access O(1). LinkedList for frequent head/tail operations. In practice, ArrayList wins almost always because of cache locality.

## Stack

**Why never use Stack class ?**

Stack is technically a List — which makes no sense for a stack data structure. That's the problem :

```java
Stack<String> s = new Stack<>();
s.push("Ali");
s.push("Sara");

// Nonsense operations that Stack inherits from Vector/List :
s.get(0);            // random access on a stack?
s.add(0, "middle"); // insert at index?
s.remove(0);         // remove by index?
s.set(0, "hacked"); // replace by index?
```

The correct replacement :

```java
// Use Deque as a Stack — clean, no legacy baggage
Deque<String> stack = new ArrayDeque<>();

stack.push("Ali"); // = addFirst
stack.push("Sara");

stack.peek(); // "Sara" — see top O(1)
stack.pop(); // "Sara" — remove top O(1)
stack.pop(); // "Ali"
```

Where it sits in the schema :

```
Collection
└── List
    └── Vector (legacy, synchronized)
            └── Stack (legacy, extends Vector — AVOID)

vs

Collection
└── Queue
   └── Deque
           └── ArrayDeque ← use this as Stack ✓
```

> "Stack extends Vector which is synchronized and exposes List methods that break the stack contract. Always use ArrayDeque — it implements Deque which has push/pop/peek and nothing else."

## Collections & thread safety

> ArrayList, HashMap, HashSet are NOT thread-safe. Using them from multiple threads without synchronization → data corruption.

```java
// Option 1 — synchronized wrappers (legacy)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String,Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
// locks the WHOLE collection — slow

// Option 2 — concurrent collections (modern, preferred)
import java.util.concurrent.*;

ConcurrentHashMap<String,Integer> cmap = new ConcurrentHashMap<>();
// locks only the BUCKET — much faster!

CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
// copy whole array on write — great for read-heavy

BlockingQueue<String> bq = new LinkedBlockingQueue<>();
// blocks thread until element available — producer/consumer
```

> "ConcurrentHashMap vs Hashtable?" → Hashtable locks the whole map (legacy).
>
> ConcurrentHashMap locks per bucket → much faster. Always use ConcurrentHashMap.

[diagram: cheat sheet — "Choose the right one" and "Classic interview questions"]
