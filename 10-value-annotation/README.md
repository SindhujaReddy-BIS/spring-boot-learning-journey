# 10 - @Value Annotation

Giving a bean its values directly in the class with **`@Value`**, instead of
`<property>` tags in the XML.

## Concept

**`@Value`**: sets the value of a field when Spring creates the bean. It can
take three kinds of values.

| Kind | Example | Meaning |
|------|---------|---------|
| Literal | `@Value("Hello")` | A fixed value |
| Property placeholder `${...}` | `@Value("${java.home}")` | A value read from a property or system property |
| SpEL expression `#{...}` | `@Value("#{4*4}")` | A Spring Expression Language calculation |

This builds on lesson 09: `Employee` was a bean with empty default values.
Now Spring fills the fields.

## Project Structure

```
src/main
├── java/com/example/componentscan
│   ├── Employee.java    # @Component with @Value fields
│   └── App.java         # main class
└── resources
    └── componentScanDemo.xml   # turns on component scanning
```

## How It Works

```java
@Component("employee")
public class Employee {
    private int employeeId;

    @Value("Hello")
    private String firstName;       // literal value

    @Value("${java.home}")
    private String lastName;        // system property: the Java install folder

    @Value("#{4*4}")
    private double salary;          // SpEL expression: 16
}
```

- `firstName` becomes `Hello`.
- `lastName` becomes the path of the Java installation on my computer, read
  from the `java.home` system property.
- `salary` becomes `16.0`, because Spring calculates `4*4` and converts it to
  a `double`.
- `employeeId` has no `@Value`, so it keeps the default `0`.

## Output

```
Employee{employeeId=0, firstName='Hello', lastName='<path to my Java installation>', salary=16.0}
```

`lastName` shows the Java folder of the machine it runs on, for example
`C:\Program Files\Java\jdk-21`.

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

- Change `#{4*4}` to `#{2*50000.5}` and check the salary.
- Use another system property, such as `${user.name}`.

## Key Takeaway

`@Value` injects simple values (text, numbers, properties, calculated values)
straight into fields. Later, the same `${...}` syntax reads values from
`application.properties` in Spring Boot.
