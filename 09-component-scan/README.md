# 09 - @Component and Component Scan

Creating beans with an **annotation** instead of writing a `<bean>` tag for
each class in the XML file.

## Concept

**`@Component`**: a marker on a class that tells Spring "make a bean from this
class".

**Component scan**: Spring searches a package for classes marked with
`@Component` and registers them as beans automatically.

Together they replace the `<bean ... />` lines I wrote in lessons 02 to 08.

## Before and After

**Before** (XML bean definition for every class):

```xml
<bean id="employee" class="com.example.componentscan.Employee"/>
```

**After** (annotation on the class + one scan line in XML):

```java
@Component("employee")
public class Employee { ... }
```

```xml
<context:component-scan base-package="com.example.componentscan"/>
```

## Project Structure

```
src/main
├── java/com/example/componentscan
│   ├── Employee.java    # @Component("employee")
│   └── App.java         # main class
└── resources
    └── componentScanDemo.xml   # turns on component scanning
```

## How It Works

**1. Mark the class** (`Employee.java`)

```java
@Component("employee")
public class Employee { ... }
```

`"employee"` is the bean name. If left out, Spring uses the class name with a
lowercase first letter (`employee`).

**2. Turn on scanning** (`componentScanDemo.xml`)

```xml
<context:component-scan base-package="com.example.componentscan"/>
```

Spring scans `com.example.componentscan` and its sub-packages. The `context`
namespace must be declared at the top of the XML (`xmlns:context=...`).

**3. Get the bean** (`App.java`)

```java
Employee employee = context.getBean("employee", Employee.class);
```

Passing `Employee.class` returns the right type, so no cast is needed.

## Output

```
Employee{employeeId=0, firstName='null', lastName='null', salary=0.0}
```

The values are the Java defaults (`0`, `null`, `0.0`) because nothing sets
them yet. The point of this lesson is that the bean exists, even though there
is no `<bean>` tag for it.

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

- Remove `@Component` from `Employee` and run again. Spring will fail
  with a `NoSuchBeanDefinitionException` because no bean is registered.
- Change `base-package` to a different package and see the bean disappear.

## Key Takeaway

`@Component` + `<context:component-scan>` lets Spring discover beans by itself,
so the XML stays short even with many classes. This is the step toward
annotation-based Spring configuration.
