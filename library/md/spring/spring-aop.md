---
title: "Spring AOP"
topic: spring
source_pdf: pdf/spring/spring-aop.pdf
review_needed: true
review_pages_indentation_rebuilt: [1, 2, 3, 4, 5]
review_notes: "No images in this PDF. Code rebuilt from PDF text positions (exact indentation). The two tables on page 2 were rebuilt from the text order. In the LoggingAspect listing (p3) one line was wrapped in the PDF in the middle of a string; it was joined without a space."
---

# Spring AOP

- Pourquoi la POA ?
  - Pourquoi des proxies en AOP ?
- Concepts de base de la POA
- Implémentation avec Spring AOP
  - 1. Dépendance Maven (si aspectj)
  - 2. Exemple de classe métier
  - 3. Aspect de logging
- Spring AOP fonctionne-t-il sur les méthodes privées ?
- Spring AOP fonctionne-t-il si j’appelle une méthode depuis la même classe ?
- Pourquoi Spring AOP ne marche pas quand tu appelles une méthode dans la même classe ?

## Pourquoi la POA ?

> En POO, certaines **logiques transversales** (comme la sécurité, les logs, les transactions) sont répétées partout ➝ code dupliqué, difficile à maintenir.

Exemples de préoccupations transversales :

- Logs
- Transactions
- Sécurité
- Gestion des exceptions
- Temps d’exécution

AOP permet de **séparer ces logiques dans des "aspects"** dédiés, sans polluer le code métier.

### Pourquoi des proxies en AOP ?

Spring AOP fonctionne en **interceptant l’appel d’une méthode**, sans modifier le code source.

Pour faire ça, Spring crée un **proxy** autour de ton bean :

- Le proxy intercepte les appels.
- Il exécute ton aspect (log, sécurité, etc).
- Puis il appelle la vraie méthode du bean.

```
Client ➝ Proxy ➝ Bean Réel
          ↑
     (Aspects : logs, sécurité, etc)
```

## Concepts de base de la POA

| Concept | Définition simple |
|---|---|
| **Aspect** | Classe qui regroupe une préoccupation transversale (ex: `LoggingAspect` ). |
| **Join Point** | Point d’exécution possible (ex: méthode appelée). |
| **Pointcut** | une règle (expression) qui dit **où** l'aspect doit s'appliquer. |
| **Advice** | Code à exécuter à ce Join Point (avant, après, autour...). |
| **Weaving** | Processus qui injecte l’Advice au Join Point. |

Types de Advice:

| Advice type | Quand le code est exécuté |
|---|---|
| `@Before` | Avant l'exécution de la méthode |
| `@After` | Après l'exécution (succès ou erreur) |
| `@AfterReturning` | Après exécution réussie |
| `@AfterThrowing` | Si exception levée |
| `@Around` | Entoure toute l’exécution de la méthode (permet de la bloquer, modifier args...) |

## Implémentation avec Spring AOP

### 1. Dépendance Maven (si aspectj)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 2. Exemple de classe métier

```java
@Service
public class UserService {
    public void createUser(String name) {
        System.out.println("User created: " + name);
    }
}
```

### 3. Aspect de logging

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.UserService.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature().getName());
    }

    @AfterReturning("execution(* com.example.service.UserService.*(..))")
    public void logAfter() {
        System.out.println("Méthode exécutée avec succès");
    }
}
```

Ici, le **JoinPoint** est l’appel d’une méthode dans `UserService` (comme `getUser()` , `createUser()` …).

Le paramètre `JoinPoint joinPoint` te donne des infos sur ce point d’exécution : `joinPoint.getSignature().getName()` te donne le nom de la méthode appelée (ex: `"createUser"` ).

```java
@Aspect
@Component
public class PerformanceAspect {

    @Around("execution(* com.example.service.UserService.*(..))")
    public Object measureExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();

        // Exécution de la méthode réelle
        Object result = pjp.proceed();

        long end = System.currentTimeMillis();
        System.out.println("Temps d'exécution de " + pjp.getSignature().getName() + ": " + (end - start) + " ms");

        return result; // important : retourner le résultat
    }
}
```

## Spring AOP fonctionne-t-il sur les méthodes privées ?

Non, car Spring AOP passe par un proxy → il intercepte uniquement les appels publics via Spring.

## Spring AOP fonctionne-t-il si j’appelle une méthode depuis la même classe ?

Non ! Car l’appel se fait en interne, sans passer par le proxy.

Solution : extraire la méthode dans un autre bean.

## Pourquoi Spring AOP ne marche pas quand tu appelles une méthode dans la même classe ?

Parce que Spring AOP utilise un **proxy** (JDK proxy ou CGLIB).

Le proxy intercepte les appels **de l’extérieur vers l’objet** :

```
[Caller] → [Proxy] → [TonBean]

```

Mais si une méthode A appelle une méthode B dans la même classe, l’appel devient :

```
TonBean.methodA() → TonBean.methodB()

```

Le proxy n’est jamais appelé.

Donc pas d’aspect, pas de `@Transactional` , pas de `@Around` , etc.

C’est un appel direct, interne, naturel → le proxy est contourné.

Exemple concret

```java
@Service
public class DemoService {

    @Transactional
    public void methodA() {
        methodB(); //  NO AOP, NO TRANSACTION !!!
    }

    @Transactional
    public void methodB() {
        // ...
    }
}

```

`methodB()` ne sera **jamais transactionnelle** si elle est appelée depuis `methodA()` .

Le proxy ne peut intercepter que ceci :

```java
demoService.methodB();  // appelé de l'extérieur → OK, aspect déclenché

```

Pas ceci :

```java
this.methodB();  // appel interne → pas d’aspect

```

Mais attention : il existe des exceptions

Voici les trois cas où ça marche quand même :

Si tu forces l’appel à passer par le proxy

Exemple :

```java
((DemoService) AopContext.currentProxy()).methodB();

```

Il faut activer :

```java
@EnableAspectJAutoProxy(exposeProxy = true)

```

→ Dans ce cas l’appel interne devient :

```
methodA → proxy → methodB

```

aspect déclenché

Si tu utilises AspectJ compile-time ou load-time weaving

Ce n’est plus du proxy Spring.

C’est du vrai weaving → AOP bytecode-level.

Dans ce cas :

```
methodA → methodB
```

→ Les deux sont interceptés, même en interne.

Si tu mets la méthode dans un autre bean (la solution recommandée)

```
ServiceA.methodA() → ServiceB.methodB()

```

Dans ce cas, Spring AOP fonctionne toujours.
