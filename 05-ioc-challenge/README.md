# 05 - IoC Challenge: Loose Coupling with Spring

Challenge: take the loose coupling example from lesson 01 and let the
**Spring IoC container** create and wire the objects instead of `main()`.

## Concept

**IoC (Inversion of Control)**: the control of creating objects and connecting
them moves from my code to the Spring container.

| | Lesson 01 (manual) | This lesson (Spring IoC) |
|---|---|---|
| Who creates the objects | `main()` using `new` | Spring container |
| Who wires them together | `main()` passes them to the constructor | Spring, from the XML file |
| Changing an implementation | Edit Java code | Edit the XML file |

## Before and After

**Before** (lesson 01, I wired everything by hand):

```java
UserDataProvider databaseProvider = new UserDatabaseProvider();
UserManager userManagerWithDB = new UserManager(databaseProvider);
```

**After** (Spring does it):

```java
UserManager userManagerWithDB =
        (UserManager) context.getBean("userManagerWithUserDataProvider");
```

There is no `new` in the main class anymore.

## Project Structure

```
src/main
├── java/com/ioc/coupling
│   ├── IOCExample.java              # main class
│   ├── UserManager.java             # depends only on the interface
│   ├── UserDataProvider.java        # interface
│   ├── UserDatabaseProvider.java    # implementation 1
│   ├── WebServiceDataProvider.java  # implementation 2
│   └── NewDatabaseProvider.java     # implementation 3
└── resources
    └── applicationIoCLooseCouplingExample.xml   # beans and wiring
```

## How It Works

**1. Define the implementations as beans**

```xml
<bean id="userDataProvider" class="com.ioc.coupling.UserDatabaseProvider"/>
<bean id="newDatabaseProvider" class="com.ioc.coupling.NewDatabaseProvider"/>
<bean id="webServiceDataProvider" class="com.ioc.coupling.WebServiceDataProvider"/>
```

**2. Inject each one into a `UserManager` through the constructor**

```xml
<bean id="userManagerWithUserDataProvider" class="com.ioc.coupling.UserManager">
    <constructor-arg ref="userDataProvider"/>
</bean>
```

Three `UserManager` beans are defined, one for each provider. The same
`UserManager` class works with all three because it only knows the
`UserDataProvider` interface.

**3. Ask the container for the bean** (`IOCExample.java`)

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationIoCLooseCouplingExample.xml");
```

## Output

```
User Details From Database
Fetching Data From WebService
New Database in action
```

## Dependencies

| Library | Version |
|---------|---------|
| spring-core | 6.1.6 |
| spring-context | 6.1.6 |

## How to Run

1. Open this folder in IntelliJ IDEA (Java 21, Maven).
2. Let Maven download the dependencies.
3. Run `IOCExample.java`.

## Key Takeaway

Interface + constructor injection + Spring container = loose coupling that
needs no code change to switch implementations. This brings together
lessons 01 to 04.
