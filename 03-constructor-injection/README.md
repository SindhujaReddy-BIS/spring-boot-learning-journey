# 03 - Constructor Dependency Injection

Injecting one bean into another through the **constructor**, configured in
Spring's XML file.

## Concept

**Dependency Injection (DI)**: a class does not create the objects it needs.
The Spring container creates them and gives them to the class.

**Constructor injection**: the dependency arrives as a constructor argument.
The object cannot be created without it, so the dependency is always present.

Here, a `Car` needs a `Specification`. `Car` never writes
`new Specification()`. Spring creates the `Specification` bean first and passes
it into the `Car` constructor.

## Project Structure

```
src/main
├── java/car/example/constructor/injection
│   ├── Specification.java   # bean with make and model (setter injection)
│   ├── Car.java             # bean that needs a Specification (constructor injection)
│   └── App.java             # main class: loads the context and gets the car
└── resources
    └── applicationConstructorInjection.xml   # bean definitions and wiring
```

## How It Works

**1. The classes**

```java
public class Car {
    private Specification specification;

    public Car(Specification specification) {   // dependency comes in here
        this.specification = specification;
    }
}
```

**2. The wiring** (`applicationConstructorInjection.xml`)

```xml
<bean id="carSpecification" class="car.example.constructor.injection.Specification">
    <property name="make" value="Toyota"/>
    <property name="model" value="Corolla"/>
</bean>

<bean id="myCar" class="car.example.constructor.injection.Car">
    <constructor-arg ref="carSpecification"/>
</bean>
```

- `<property>` sets values through setters (setter injection).
- `<constructor-arg ref="...">` passes another bean into the constructor
  (constructor injection).

**3. Getting the bean** (`App.java`)

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationConstructorInjection.xml");
Car myCar = (Car) context.getBean("myCar");
myCar.displayDetails();
```

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

## Key Takeaway

`Car` depends on `Specification` but does not create it. Spring builds the
objects and connects them. This is the loose coupling from lesson 01, now done
by the Spring container.
