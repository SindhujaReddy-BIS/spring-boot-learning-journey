# 07 - Autowiring by Type

Letting Spring connect beans automatically by matching the **type** (class)
of the dependency, so the bean ids no longer have to match.

## Concept

**`autowire="byType"`**: Spring looks at the setter parameter type
(`setSpecification(Specification ...)` needs a `Specification`) and injects the
bean of that type, whatever its id is.

## byName vs byType

| | byName (06) | byType (07) |
|---|---|---|
| Spring matches | Bean **id** to property name | Bean **class** to parameter type |
| Bean id matters | Yes | No |
| Two beans of the same type | Works (name picks one) | **Error** (Spring cannot choose) |

## Project Structure

```
src/main
├── java/com/example/autowire/type
│   ├── Specification.java   # bean with make and model
│   ├── Car.java             # has setSpecification(Specification ...)
│   └── App.java             # main class
└── resources
    └── autowireByType.xml   # bean definitions
```

## How It Works

```xml
<bean id="specification1" class="com.example.autowire.type.Specification">
    <property name="make" value="Toyota"/>
    <property name="model" value="Corolla"/>
</bean>

<bean id="myCar" class="com.example.autowire.type.Car" autowire="byType"/>
```

`Car` needs a `Specification`. Spring finds exactly one bean of that type
(`specification1`) and injects it. The id `specification1` does not match the
property name, but that is fine for `byType`.

## Why Only One Specification Bean?

The XML has a second `Specification` bean commented out on purpose. With
`byType`, two beans of the same class make Spring unable to choose, and it
fails with:

```
NoUniqueBeanDefinitionException: expected single matching bean but found 2
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

## Things to Try

- Uncomment the second `Specification` bean and run again to see the
  "expected single matching bean" error.

## Key Takeaway

`byType` works only when there is exactly one bean of the required type.
If there are several, use `byName` or a more specific setup.
