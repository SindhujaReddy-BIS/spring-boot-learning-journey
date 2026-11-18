# 08 - Autowiring by Constructor

Letting Spring connect beans automatically through the **constructor**, so the
XML does not need a `<constructor-arg>` line.

## Concept

**`autowire="constructor"`**: Spring looks at the constructor of the bean
(`Car(Specification specification)`), finds a bean that matches the
parameter type, and passes it in.

It is constructor injection (lesson 03) with the wiring done automatically.

## The Autowiring Modes So Far

| Mode | Lesson | Injects through | Spring matches |
|------|--------|-----------------|----------------|
| `byName` | 06 | Setter | Bean id = property name |
| `byType` | 07 | Setter | Bean class = parameter type |
| `constructor` | 08 | Constructor | Bean class = constructor parameter type |

## Project Structure

```
src/main
├── java/com/example/autowire/constructor
│   ├── Specification.java   # bean with make and model
│   ├── Car.java             # has a constructor that takes a Specification
│   └── App.java             # main class
└── resources
    └── autowireByConstructor.xml   # bean definitions
```

## How It Works

**1. The class has a constructor, not a setter**

```java
public class Car {
    private Specification specification;

    public Car(Specification specification) {
        this.specification = specification;
    }
}
```

**2. The XML only asks for autowiring** (`autowireByConstructor.xml`)

```xml
<bean id="specification1" class="com.example.autowire.constructor.Specification">
    <property name="make" value="Toyota"/>
    <property name="model" value="Corolla"/>
</bean>

<bean id="myCar" class="com.example.autowire.constructor.Car" autowire="constructor"/>
```

No `<constructor-arg>` is needed. Spring sees that `Car` needs a
`Specification` and injects the one bean of that type.

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

- The XML has a second `Specification` bean commented out. Uncomment it and
  run again to see how Spring behaves when two beans of the same type exist.

## Key Takeaway

`autowire="constructor"` gives constructor injection without writing the
wiring in XML. The dependency is required, because the object cannot be
created without it.
