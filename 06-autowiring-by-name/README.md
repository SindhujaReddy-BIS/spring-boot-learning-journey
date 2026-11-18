# 06 - Autowiring by Name

Letting Spring connect beans automatically by matching **names**, so the XML
no longer needs a `<property ref="...">` line.

## Concept

**Autowiring**: Spring finds and injects dependencies by itself.

**`autowire="byName"`**: Spring looks at the setter property names of a bean
(`setSpecification` means the property is `specification`) and injects the bean
whose **id** has the same name.

## Before and After

**Before** (lesson 04, wired by hand):

```xml
<bean id="myCar" class="...Car">
    <property name="specification" ref="carSpecification"/>
</bean>
```

**After** (autowired):

```xml
<bean id="myCar" class="com.example.autowire.name.Car" autowire="byName"/>
```

## Project Structure

```
src/main
├── java/com/example/autowire/name
│   ├── Specification.java   # bean with make and model
│   ├── Car.java             # has setSpecification(...)
│   └── App.java             # main class
└── resources
    └── autowireByName.xml   # bean definitions
```

## How It Works

```xml
<bean id="specification"  class="...Specification">  <!-- Toyota / Corolla -->
<bean id="specification1" class="...Specification">  <!-- Toyota1 / Corolla1 -->

<bean id="myCar" class="...Car" autowire="byName"/>
```

1. `Car` has the setter `setSpecification(...)`, so Spring looks for a bean
   with the id **`specification`**.
2. It finds it and injects it. The bean `specification1` is ignored because
   its name does not match.

Two `Specification` beans exist on purpose, to show that the **name** decides
which one is used.

## Output

```
Car Details: Specification{make='Toyota', model='Corolla'}
```

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

- Rename the bean id `specification` to `spec`. The match is lost and
  `Car` gets no `Specification`, so `displayDetails()` throws a
  `NullPointerException`.
- Rename `specification1` to `specification` and `specification` to
  `specification1` to see the other bean get picked.

## Key Takeaway

With `byName`, the bean id must match the property name in the class.
Less XML to write, but a renamed bean can silently break the wiring.
