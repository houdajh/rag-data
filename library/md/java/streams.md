---
title: "Streams"
topic: java
source_pdf: pdf/java/streams.pdf
review_needed: true
review_pages_indentation_rebuilt: [1, 2, 3, 4, 5]
review_notes: "No images in this PDF. Code listings rebuilt from PDF text positions (exact indentation). The tables on pages 1-2 and 4 were rebuilt from the rendered pages. The coloured-circle emoji (blue / red) before 'Opérations Intermédiaires / Terminales' were read from the rendered page."
---

# Streams

"Explique-moi la différence entre un `Stream` et une `Collection` en Java. Pourquoi on utiliserait l'un plutôt que l'autre ?

**Collection** → stocke des données en mémoire, tu peux les parcourir plusieurs fois, ajouter/supprimer des éléments.

**Stream** → ne stocke rien. C'est un pipeline de traitement qui décrit *comment* transformer des données. Deux points clés que l'interviewer attend :

1. **Lazy Evaluation (évaluation paresseuse)** Les opérations intermédiaires ( `filter` , `map` , `sorted` ...) ne s'exécutent pas immédiatement. Elles attendent qu'une opération terminale ( `collect` , `forEach` , `count` ...) soit appelée.

```java
Stream<String> stream = List.of("Ali", "Bob", "Clara")
    .stream()
    .filter(s -> {
        System.out.println("filter: " + s); // pas encore exécuté !
        return s.length() > 2;
    });
// Rien ne s'affiche ici

stream.collect(Collectors.toList()); // là ça s'exécute
```

2. **Un Stream s'utilise une seule fois**

```java
Stream<String> s = list.stream();
s.collect(Collectors.toList()); // OK
s.collect(Collectors.toList()); // IllegalStateException !
```

3. **Stream peut exister sans Collection**

```java
Stream.of("a", "b", "c");       // depuis des valeurs
Stream.iterate(0, n -> n + 1);  // stream infini !
Stream.generate(Math::random);  // stream infini !
```

"Quelles sont les opérations intermédiaires et terminales ? Cite-moi des exemples et dis-moi pourquoi cette distinction est importante."

🔵 Opérations Intermédiaires

- Retournent un `Stream` → donc chaînables
- **Lazy** → ne s'exécutent pas tout de suite
- Exemples :

| Méthode | Ce qu'elle fait |
|---|---|
| `filter(predicate)` | garde les éléments qui matchent |
| `map(function)` | transforme chaque élément |
| `sorted()` | trie |
| `distinct()` | supprime les doublons |
| `limit(n)` | garde les n premiers |
| `skip(n)` | saute les n premiers |

🔴 Opérations Terminales

- **Déclenchent** l'exécution du pipeline
- **Retournent autre chose** qu'un Stream (liste, int, void...)
- Le Stream est **consommé** après, inutilisable

| Méthode | Ce qu'elle retourne |
|---|---|
| `collect(...)` | une Collection (List, Set...) |
| `forEach(...)` | void |
| `count()` | long |
| `findFirst()` | Optional |
| `anyMatch(...)` | boolean |
| `reduce(...)` | une valeur agrégée |

## Exercice pratique — Tu codes

Voici une liste d'entiers :

```java
List<Integer> nombres = List.of(5, 3, 8, 1, 9, 2, 7, 4, 6);
```

**Écris un Stream qui :**

1. Garde uniquement les nombres **pairs**
2. Les **multiplie par 2**
3. Trie dans l'ordre **décroissant**
4. Retourne une **List**

```java
List<Integer> nombres = List.of(5, 3, 8, 1, 9, 2, 7, 4, 6);

List<Integer> result = nombres.stream()
    .filter(e -> e % 2 == 0)        // 8, 2, 4, 6
    .map(e -> e * 2)                 // 16, 4, 8, 12
    .sorted(Comparator.reverseOrder()) // 16, 12, 8, 4
    .toList();

System.out.println(result); // [16, 12, 8, 4]
```

Bonus à savoir pour l'entretien

`toList()` (Java 16+) retourne une liste **immutable**. Si t'as besoin de modifier la liste après :

```java
.collect(Collectors.toList()) // mutable
```

" `map()` vs `flatMap()` — c'est quoi la différence ? Donne-moi un cas concret où tu dois utiliser `flatMap()` plutôt que `map()` ."

Règle simple à retenir :

`map()` → 1 élément donne 1 élément `flatMap()` → 1 élément donne plusieurs éléments (aplatit le résultat)

`map()` — transformation simple

```java
List<String> noms = List.of("Ali", "Bob", "Clara");

List<Integer> longueurs = noms.stream()
    .map(n -> n.length())  // chaque String → un Integer
    .toList();

// [3, 3, 5]
```

1 élément entrant → 1 élément sortant. Simple.

Le problème que `flatMap()` résout

Imagine que tu as une liste de listes :

```java
List<List<Integer>> groupes = List.of(
    List.of(1, 2, 3),
    List.of(4, 5),
    List.of(6, 7, 8, 9)
);
```

Avec `map()` tu obtiens... une liste de listes :

```java
// Stream<List<Integer>> → pas ce qu'on veut
groupes.stream()
    .map(g -> g.stream()) // Stream<Stream<Integer>>
```

Avec `flatMap()` tu aplatis en un seul Stream :

```java
List<Integer> tousLesNombres = groupes.stream()
    .flatMap(g -> g.stream())  // aplatit tout
    .toList();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

"Explique-moi ce que fait `reduce()` et à quoi ça sert. C'est quoi la différence avec `collect()` ?"

L'idée centrale

> `reduce()` combine tous les éléments du Stream en **une seule valeur**.

C'est ça la différence avec `map()` qui transforme chaque élément individuellement.

La signature

```java
reduce(valeurInitiale, (accumulateur, elementCourant) -> résultat)
```

| | `reduce()` | `collect()` |
|---|---|---|
| Retourne | une valeur (int, String...) | une Collection (List, Map...) |
| Usage | calculs, agrégations | regrouper des éléments |
| Exemple | somme, max, concaténation | toList(), toMap(), groupingBy() |

"Explique-moi ce que fait `collect()` et cite-moi les `Collectors` que tu connais avec leurs cas d'usage."

L'idée centrale

> `collect()` est une opération **terminale** qui **regroupe** les éléments du Stream dans un conteneur (List, Set, Map, String...)

C'est le plus utilisé en pratique !

Les Collectors les plus importants

1️⃣ `toList()` / `toSet()`

```java
List<String> liste = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
```

2️⃣ `joining()` — concaténer des Strings

```java
List<String> mots = List.of("Java", "est", "cool");

String result = mots.stream()
    .collect(Collectors.joining(", "));
// "Java, est, cool"

// Avec préfixe et suffixe :
String result2 = mots.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// "[Java, est, cool]"
```

3️⃣ `counting()` — compter

```java
long count = stream.collect(Collectors.counting());
```

4️⃣ `groupingBy()` — grouper par critère très demandé

```java
List<String> noms = List.of("Ali", "Adam", "Bob", "Clara", "Chloe");

Map<Character, List<String>> parLettre = noms.stream()
    .collect(Collectors.groupingBy(n -> n.charAt(0)));

// {A=[Ali, Adam], B=[Bob], C=[Clara, Chloe]}
```

5️⃣ `partitioningBy()` — diviser en 2 groupes

```java
List<Integer> nombres = List.of(1, 2, 3, 4, 5, 6);

Map<Boolean, List<Integer>> partition = nombres.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// {false=[1, 3, 5], true=[2, 4, 6]}
```

6️⃣ `toMap()` — convertir en Map

```java
List<String> noms = List.of("Ali", "Bob", "Clara");

Map<String, Integer> nomLongueur = noms.stream()
    .collect(Collectors.toMap(
        n -> n,           // clé
        n -> n.length()   // valeur
    ));
// {Ali=3, Bob=3, Clara=5}
```
