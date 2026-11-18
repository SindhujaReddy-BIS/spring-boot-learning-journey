# 11 - From XML to Annotations (Java Configuration)

Removing the XML file completely. The Spring configuration now lives in a Java
class marked with `@Configuration`.

## Concept

**`@Configuration`**: marks a class as a source of Spring configuration. It
replaces the XML file.

**`@ComponentScan`**: the annotation version of `<context:component-scan>`.
It tells Spring which package to scan for `@Component` classes.

**`AnnotationConfigApplicationContext`**: the container that starts from a
Java config class, replacing `ClassPathXmlApplicationContext`.

## XML vs Java Configuration

| | Lesson 10 (XML) | Lesson 11 (Java config) |
|---|---|---|
| Configuration lives in | `componentScanDemo.xml` | `AppConfig.java` |
| Scan instruction | `<context:component-scan base-package="..."/>` | `@ComponentScan(basePackages = "...")` |
| Container class | `ClassPathXmlApplicationContext("...xml")` | `AnnotationConfigApplicationContext(AppConfig.class)` |
| XML files needed | 1 | **0** |

## Project Structure

```
src/main/java/com/example/componentscan/annotation
├── AppConfig.java   # @Configuration + @ComponentScan (replaces the XML)
├── Employee.java    # @Component with @Value fields
└── App.java         # main class
```

There is no `resources` XML file in this project.

## How It Works

**1. The config class** (`AppConfig.java`)

```java
@Configuration
@ComponentScan(basePackages = "com.example.componentscan.annotation")
public class AppConfig {
}
```

**2. Start the container from the config class** (`App.java`)

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
Employee employee = context.getBean("employee", Employee.class);
```

**3. The bean** (`Employee.java`) is the same as lesson 10: `@Component` with
`@Value` on its fields.

## Output

```
Employee{employeeId=0, firstName='Hello', lastName='C:\Program Files\Java\jdk-21', salary=16.0}
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

Spring can be configured with no XML at all: `@Configuration` +
`@ComponentScan` + `@Component`. This is the style Spring Boot uses, so
it is the bridge from classic Spring to Spring Boot.
