# 02 - Creating Your First Bean

My first Spring bean, defined in an XML file and fetched from the Spring
container (IoC container).

## Concept

A **bean** is an object created and managed by the Spring container instead of
by `new` in my own code. The container reads the configuration, creates the
object, sets its properties (setter injection) and hands it over when asked.

This builds on the previous lesson: loose coupling means my code no longer
creates its own dependencies. Spring now does that job.

## Project Structure

```
src/main
├── java/car/example/bean
│   ├── MyBean.java      # plain Java class (the bean)
│   └── App.java         # main class: loads the context and gets the bean
└── resources
    └── applicationBeanContext.xml   # bean definition
```

## How It Works

**1. The bean definition** (`applicationBeanContext.xml`)

```xml
<bean id="myBean" class="car.example.bean.MyBean">
    <property name="message" value="I am a first bean"/>
</bean>
```

`id` is the name of the bean, `class` is the class to create, and `<property>`
calls the `setMessage(...)` setter to inject the value.

**2. Loading the container and getting the bean** (`App.java`)

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationBeanContext.xml");

MyBean myBean = (MyBean) context.getBean("myBean");
System.out.println(myBean);
```

Spring creates `MyBean` itself. There is no `new MyBean()` in my code.

## Output

```
MyBean{message='I am a first bean'}
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

Spring creates and manages objects (beans) for me. I describe them in
configuration and ask the container for them.
