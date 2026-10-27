---
title: "Java evolution"
topic: java
source_pdf: pdf/java/java-evolution.pdf
review_needed: true
review_pages_indentation_rebuilt: [2, 3, 4, 5, 6, 7, 8, 9]
review_notes: "No images in this PDF. Code listings rebuilt from PDF text positions. The Java 8 feature table (p1) and the Java 8 vs 9-11 comparison table (p4) were rebuilt from the rendered pages."
---

# Java evolution

- Module 1 : Java 8 – Fondations modernes.
- Module 3 — Java 9 → 11,
  - Java 9 – principales évolutions
    - A. Collections immuables (List.of, Set.of, Map.of)
    - B. Try-with-resources amélioré
  - C. jshell
  - Java 10 – type inference avec var
  - Java 11 – nouvelles APIs et stabilisation LTS
    - A. Nouveau HttpClient (remplace HttpURLConnection)
    - B. Méthodes utiles sur String
    - C. Améliorations diverses
- Module 4 — Java 12 → 16
  - Java 12 – Switch expressions (aperçu)
  - Java 13–15 – Text Blocks
  - Java 14–16 – Records (finalisés en 16)
- Module 5 – Java 17 (LTS)
  - Sealed Classes – Contrôler l’héritage
  - Variante mixte
  - Pattern Matching pour switch (aperçu en 17)
  - Combinaison Records + Pattern Matching
  - Autres améliorations (Java 17)
- Module 6 : Java 21 Advanced Features (LTS)
  - Virtual Threads — Project Loom
  - Sequenced Collections
  - Record Patterns + Pattern Matching for Switch (finalized)
  - Scoped Values
  - String Templates

## Module 1 : Java 8 – Fondations modernes.

Avant Java 8, Java était purement orienté objet ; à partir de Java 8, il devient **hybride** : orienté objet et fonctionnel.

Nouveautés majeures:

| Domaine | Fonctionnalité | Description rapide |
|---|---|---|
| Fonctions anonymes | **Lambdas** | Permettent d’écrire du code plus concis et expressif |
| Collections | **Stream API** | Chaînage d’opérations (filter, map, reduce…) |
| Null-safety | **Optional** | Conteneur pour éviter les NullPointerException |
| Dates | **java.time** | Nouvelle API de date/heure (remplace `Date` et `Calendar` ) |
| Concurrence | **CompletableFuture** | Exécution asynchrone simplifiée |
| Interfaces | **Méthodes par défaut** | Permettent d’ajouter une méthode dans une interface sans casser les implémentations existantes |

```java
CompletableFuture.supplyAsync(() -> fetchData())
                 .thenApply(data -> process(data))
                 .thenAccept(System.out::println);
```

=> Asynchrone sans Thread explicite.

Méthodes par défaut dans les interface:

```java
interface Vehicle {
    void move();
    default void stop() {
        System.out.println("Vehicle stopped");
    }
}
```

## Module 3 — Java 9 → 11,

### Java 9 – principales évolutions

#### A. Collections immuables ( `List.of` , `Set.of` , `Map.of` )

Avant Java 9 :

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");

```

Depuis Java 9 :

```java
List<String> list = List.of("A", "B");     // immuable
Set<Integer> set = Set.of(1, 2, 3);
Map<Integer,String> map = Map.of(1,"A",2,"B");

```

Tentative de `add` ou `remove` ⇒ `UnsupportedOperationException` .

#### B. Try-with-resources amélioré

Plus besoin de redéclarer la ressource :

```java
BufferedReader br = new BufferedReader(new FileReader("file.txt"));
try (br) {  // <— variable déjà existante
    br.lines().forEach(System.out::println);
}

```

### C. jshell

Parfait pour tester du code rapide.

```
jshell> int x = 5;
jshell> x + 2
```

#### D. Module System (JPMS)

Nécessaire pour très gros projets, mais à connaître :

- `module-info.java`
- contrôle explicite des dépendances / export de packages
- Pour l’instant, retiens : c’est optionnel, mais utile pour des applis cloisonnées.

### Java 10 – type inference avec `var`

Permet d’alléger le code sans perdre le typage statique :

```java
var list = List.of("A", "B", "C");   // List<String>
for (var item : list) {
    System.out.println(item);
}
```

`var` ne peut être utilisé que pour des variables locales initialisées.
Pas pour : champs, paramètres, ou valeurs non-initialisées.

### Java 11 – nouvelles APIs et stabilisation LTS

#### A. Nouveau HttpClient (remplace `HttpURLConnection` )

Exemple synchrone :

```java
import java.net.http.*;
import java.net.URI;

HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.github.com"))
        .build();

HttpResponse<String> response =
        client.send(request, HttpResponse.BodyHandlers.ofString());

System.out.println(response.statusCode());
System.out.println(response.body());

```

Asynchrone :

```java
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);

```

#### B. Méthodes utiles sur `String`

```java
"   ".isBlank();                // true
"abc\n".lines().count();        // 1
"Hello".repeat(3);              // HelloHelloHello
```

#### C. Améliorations diverses

- Fichiers en ligne de commande : `java Hello.java`
- Garbage Collector G1 par défaut
- Support complet TLS 1.3

| Catégorie | Java 8 | Java 9–11 ajout |
|---|---|---|
| Lambdas | ✅ | — |
| Streams | ✅ | — |
| Optional | ✅ | — |
| API Date | ✅ | — |
| Collections immuables | ❌ | ✅ |
| `var` | ❌ | ✅ |
| HttpClient moderne | ❌ | ✅ |
| String utils ( `isBlank` , `repeat` ) | ❌ | ✅ |
| JShell | ❌ | ✅ |
| Module System | ❌ | ✅ |

## Module 4 — Java 12 → 16

### Java 12 – Switch expressions (aperçu)

Avant :

```java
int num = 2;
String result;
switch (num) {
    case 1: result = "un"; break;
    case 2: result = "deux"; break;
    default: result = "autre";
}

```

Depuis Java 14 (finalisé en Java 14) :

```java
int num = 2;
String result = switch (num) {
    case 1 -> "un";
    case 2 -> "deux";
    default -> "autre";
};

```

Plus de `break` , plus clair, retourne directement une valeur.

### Java 13–15 – Text Blocks

Les text blocks simplifient les chaînes multilignes :

```java
String json = """
    {
        "name": "Mouad",
        "city": "Casablanca"
    }
    """;

System.out.println(json);

```

Fini les `"\n"` et les concaténations.
Java 15 a finalisé cette feature.

### Java 14–16 – Records (finalisés en 16)

Un record est une classe immuable, idéale pour DTO / valeurs simples :

Avant :

```java
class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() { return name; }
    public int age() { return age; }
}
```

Depuis Java 16 :

```java
public record Person(String name, int age) {}
```

Automatiquement :

- champs `final`
- constructeur
- getters ( `name()` , `age()` )
- `equals` , `hashCode` , `toString`

## Module 5 – Java 17 (LTS)

Découvrir ce que Java 17 apporte de stable :

- Sealed classes : hiérarchies contrôlées
- Pattern matching pour switch (aperçu)
- Records + Pattern matching combinés
- Améliorations API & performance

### Sealed Classes – Contrôler l’héritage

Tu définis quelles classes ont le droit d’hériter d’une super-classe.
Cela rend les hiérarchies plus sûres et plus explicites.

```java
public abstract sealed class Shape
    permits Circle, Rectangle, Triangle { }

public final class Circle extends Shape { }
public final class Rectangle extends Shape { }
public final class Triangle extends Shape { }

```

Seule une classe listée dans `permits` peut hériter de `Shape` .
Chaque sous-classe doit être `final` , `sealed` , ou `non-sealed` .

### Variante mixte

```java
public abstract sealed class Vehicle permits Car, Truck, Drone { }

public final class Car extends Vehicle { }
public sealed class Truck extends Vehicle permits ElectricTruck { }
public non-sealed class Drone extends Vehicle { } // libre héritage

```

Lisibilité + sécurité : impossible d’ajouter un type inattendu ailleurs.

### Pattern Matching pour switch (aperçu en 17)

On peut combiner `instanceof` + `switch` de façon élégante :

```java
static String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + (i * 2);
        case String s  -> "String: " + s.toUpperCase();
        case null      -> "Null value";
        default        -> "Unknown type";
    };
}

```

Ce pattern matching pour switch était en aperçu dans Java 17, finalisé en 21.

### Combinaison Records + Pattern Matching

Tu peux déstructurer un record dans un switch :

```java
public record Point(int x, int y) {}

static String position(Point p) {
    return switch (p) {
        case Point(int x, int y) when x == 0 && y == 0 -> "Origine";
        case Point(int x, int y) when x == 0 -> "Sur axe Y";
        case Point(int x, int y) when y == 0 -> "Sur axe X";
        default -> "Autre point";
    };
}
```

### Autres améliorations (Java 17)

- RandomGenerator API moderne : `RandomGeneratorFactory.of("L64X128MixRandom")`
- Suppressions : Applet API, RMI Activation, expérimental Nashorn JS engine
- Nouvel algorithme de GC : ZGC et G1 stabilisés
- Sécurité accrue : interdiction de strong encapsulation par défaut ( `--illegal-access` supprimé)

## Module 6 : Java 21 Advanced Features (LTS)

Master the big innovations introduced or finalized in Java 21 LTS:

- Virtual Threads (JEP 444)
- Sequenced Collections (JEP 431)
- Record Patterns & Pattern Matching for Switch (final in 21)
- Scoped Values (JEP 446 preview)
- String Templates (JEP 430 preview)

### Virtual Threads — Project Loom

A Virtual Thread is a lightweight, OS-independent thread managed by the JVM.
You can create thousands of them cheaply for blocking I/O tasks.

```java
public class VirtualThreadsDemo {
    public static void main(String[] args) throws InterruptedException {
        try (var executor = java . util . concurrent . Executors . newVirtualThreadPerTaskExecutor ( ) ) {
            for (int i = 0; i < 10; i++) {
                int id = i;
                executor.submit(() -> {
                    System.out.println("Task " + id + " → " + Thread . currentThread ( ) ) ;
                    Thread.sleep(200);
                    return null;
                });
            }
        }
    }
}

```

Each task runs in its own virtual thread — minimal memory and context-switch cost.
Use them whenever you do blocking I/O (HTTP, DB, file) instead of classic thread-pools.

### Sequenced Collections

Finally, `List` , `Set` , and `Map` now share common order-aware methods:

```java
List<String> list = List.of("A", "B", "C");
System.out.println(list.getFirst());  // A
System.out.println(list.getLast());   // C
list.reversed().forEach(System.out::println);  // C, B, A

Map<Integer, String> map = Map.of(1,"one",2,"two");
System.out.println(map.firstEntry());  // 1=one
System.out.println(map.lastEntry());   // 2=two

```

Consistent iteration order APIs across all collections.

### Record Patterns + Pattern Matching for Switch (finalized)

You already used these in the last module — Java 21 finalized them (no `--enable-preview` ).

```java
record Point(int x, int y) {}

static String describe(Object o) {
    return switch (o) {
        case Point(int x, int y) when x == y -> "Diagonal";
        case Point(int x, int y) -> "Point (" + x + "," + y + ")";
        default -> "Unknown";
    };
}

```

### Scoped Values

A safer, immutable replacement for `ThreadLocal` — lets you share read-only context across threads (even virtual threads).

```java
import jdk.incubator.concurrent.ScopedValue;

public class ScopedValuesDemo {
    static final ScopedValue<String> USER = ScopedValue.newInstance();

    public static void main(String[] args) {
        ScopedValue.where(USER, "Mouad")
                   .run(() -> {
                       System.out.println("Current user: " + USER . get ( ) ) ;
                   });
    }
}

```

Perfect for per-request context (auth, trace IDs) without leaking memory between threads.

### String Templates

Simpler, safer string interpolation (still preview):

```java
String name = "Mouad";
int age = 26;

String message = STR."Hello \{name}, you are \{age} years old!";
System.out.println(message);

```

→ No `String.format` , no concatenation, and injection-safe.
