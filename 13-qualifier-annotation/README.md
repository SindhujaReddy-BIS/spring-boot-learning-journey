# 13 - @Qualifier Annotation

Telling Spring **which bean** to inject when `@Autowired` could match more
than one.

## Concept

**`@Qualifier("beanName")`**: used together with `@Autowired` to pick a bean by
its name. `@Autowired` matches by type, so if two beans have the same type,
Spring needs this extra hint to choose.

This builds on lesson 12, where `Manager` got an `Employee` through
`@Autowired`.

## Before and After

**Before** (lesson 12):

```java
@Autowired
private Employee employee;
```

**After** (this lesson):

```java
@Autowired
@Qualifier("employee")
private Employee employee;
```

Only `Manager.java` changed.

## Project Structure

```
src/main/java/com/example/autowired/annotation
├── AppConfig.java   # @Configuration + @ComponentScan
├── Employee.java    # @Component("employee") with @Value fields
├── Manager.java     # @Autowired + @Qualifier("employee")
└── App.java         # main class
```

## How It Works

`@Qualifier("employee")` says: inject the bean whose name is `employee`. That
name comes from `@Component("employee")` on the `Employee` class.

In this project there is only one `Employee` bean, so the qualifier changes
nothing in the output. It is here to show the syntax. It becomes necessary
when several beans share a type, for example two `Employee` beans or two
implementations of the same interface (like the data providers in lesson 05).

## Output

```
Employee{employeeId=0, firstName='Hello', lastName='C:\Program Files\Java\jdk-21', salary=16.0}
Manager{employee=Employee{employeeId=0, firstName='Hello', lastName='C:\Program Files\Java\jdk-21', salary=16.0}}
```

`lastName` shows the Java folder of the machine it runs on.

## Dependencies

| Library | Version |
|---------|---------|
| spring-core | 6.1.6 |
| spring-context | 6.1.6 |

## How to Run

1. Open this folder in IntelliJ IDEA (Java 21, Maven).
2. Let Maven download the dependencies.
3. Run `App.java`.

## Key Takeaway

`@Autowired` chooses by **type**; `@Qualifier` chooses by **name**. Use both
when more than one bean of the same type exists.
