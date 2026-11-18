# 04 - Setter Dependency Injection

Injecting one bean into another through a **setter method**, configured in
Spring's XML file.

## Concept

**Setter injection**: Spring creates the object with its default (no-argument)
constructor, then calls a setter method to give it its dependency.

Here, a `Car` needs a `Specification`. `Car` has a `setSpecification(...)`
method, and Spring calls it with the `Specification` bean.

## Constructor vs Setter Injection

| | Constructor injection (03) | Setter injection (04) |
|---|---|---|
| Dependency passed through | Constructor | Setter method |
| XML tag | `<constructor-arg>` | `<property>` |
| Object without dependency | Cannot be created | Can exist without it |
| Best for | Required dependencies | Optional dependencies |

## Project Structure

```
src/main
├── java/car/example/setter/injection
│   ├── Specification.java   # bean with make and model
│   ├── Car.java             # bean with setSpecification(...)
│   └── App.java             # main class: loads the context and gets the car
└── resources
    └── applicationSetterInjection.xml   # bean definitions and wiring
```

## How It Works

**1. The class**

```java
public class Car {
    private Specification specification;

    public void setSpecification(Specification specification) {  // Spring calls this
        this.specification = specification;
    }
}
```

**2. The wiring** (`applicationSetterInjection.xml`)

```xml
<bean id="carSpecification" class="car.example.setter.injection.Specification">
    <property name="make" value="Toyota"/>
    <property name="model" value="Corolla"/>
</bean>

<bean id="myCar" class="car.example.setter.injection.Car">
    <property name="specification" ref="carSpecification"/>
</bean>
```

`<property name="specification" ref="carSpecification"/>` tells Spring to call
`setSpecification(...)` and pass in the `carSpecification` bean.

**3. Getting the bean** (`App.java`)

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationSetterInjection.xml");
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

The result is the same as constructor injection, but the dependency arrives
through a setter. Setter injection suits optional dependencies; constructor
injection suits required ones.
