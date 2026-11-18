# 01 - Tight Coupling vs Loose Coupling

Introduction to the idea that sits behind Spring's Dependency Injection: how
tightly or loosely one class depends on another.

## Concept

**Tight coupling**: a class creates the objects it depends on by itself
(`new UserDatabase()`). Changing the data source means editing the class.

**Loose coupling**: a class depends on an interface and receives the
implementation from outside (constructor injection). Changing the data
source needs no change in the class.

## Project Structure

```
src/main/java/com
├── tight/coupling
│   ├── TightCouplingExample.java   # main class
│   ├── UserManager.java            # creates UserDatabase with `new`
│   └── UserDatabase.java
└── loose/coupling
    ├── LooseCouplingExample.java   # main class
    ├── UserManager.java            # depends only on the interface
    ├── UserDataProvider.java       # interface
    ├── UserDatabaseProvider.java   # implementation 1
    ├── WebServiceDataProvider.java # implementation 2
    └── NewDatabaseProvider.java    # implementation 3
```

## Tight Coupling

```java
public class UserManager {
    private UserDatabase userDatabase = new UserDatabase(); // hard-wired
}
```

`UserManager` is stuck with `UserDatabase`. To use a web service or another
database, the class itself must be changed.

## Loose Coupling

```java
public class UserManager {
    private UserDataProvider userDataProvider;

    public UserManager(UserDataProvider userDataProvider) { // injected
        this.userDataProvider = userDataProvider;
    }
}
```

`UserManager` only knows the `UserDataProvider` interface. The same class
works with a database, a web service or a new database without any change.

## Output

Tight coupling:
```
User Details From Database
```

Loose coupling:
```
User Details From Database
Fetching Data From WebService
New Database in action
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 21, Maven).
2. Run `TightCouplingExample` or `LooseCouplingExample`.

## Key Takeaway

Program to an interface, not an implementation. Spring's IoC container
automates this wiring, which is what the next lessons cover.
