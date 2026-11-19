# 12 - @Autowired Annotation

Connecting one bean to another with **`@Autowired`**, with no XML and no
wiring code.

## Concept

**`@Autowired`**: tells Spring "find a bean of this type and inject it here".

This is the annotation version of the autowiring modes from lessons 06 to 08,
combined with the Java configuration from lesson 11.

Here, a `Manager` needs an `Employee`. `Manager` never creates the
`Employee`. Spring injects it.

## Project Structure

```
src/main/java/com/example/autowired/annotation
├── AppConfig.java   # @Configuration + @ComponentScan
├── Employee.java    # @Component with @Value fields
├── Manager.java     # @Component with an @Autowired Employee
└── App.java         # main class
```

## How It Works

**1. Both classes are beans** (`@Component`)

```java
@Component("employee")
public class Employee { ... }

@Component
public class Manager { ... }
```

`Manager` has no name given, so its bean name is `manager` (class name with a
lowercase first letter).

**2. Spring injects the Employee into the Manager** (`Manager.java`)

```java
@Component
public class Manager {
    @Autowired
    private Employee employee;
}
```

**3. Get both beans** (`App.java`)

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
Employee employee = context.getBean("employee", Employee.class);
Manager manager = context.getBean("manager", Manager.class);
```

## Where @Autowired Can Go

| Style | Example | Note |
|-------|---------|------|
| Field injection | `@Autowired private Employee employee;` | Used in this lesson. Shortest. |
| Constructor injection | `@Autowired public Manager(Employee employee)` | Commented out in `Manager.java`. Recommended in real projects. |
| Setter injection | `@Autowired public void setEmployee(...)` | For optional dependencies. |

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

## Things to Try

- Comment out the field injection and enable the constructor version
  that is commented out in `Manager.java`. The output stays the same.
- Remove `@Component` from `Employee` and run again to see Spring fail
  because it has no `Employee` bean to inject.

## Key Takeaway

`@Autowired` lets Spring wire beans together without any XML. This is the
same dependency injection used in every Spring Boot application.
