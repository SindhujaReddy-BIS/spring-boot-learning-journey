# 14 - First Spring Boot REST API

My first real Spring Boot application: a small REST API that says hello, built
step by step from a plain text response to JSON and URL parameters.

## Tech Stack

Java 17 | Spring Boot 3.1.1 | Maven | Spring Web | IntelliJ IDEA

## What I Built, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `GET /hello` returns `Hello World!` | `@SpringBootApplication`, `@RestController`, `@GetMapping` |
| 2 | `server.port=9090`, then back to `8080` | `application.properties` |
| 3 | JSON response and `POST /hello` | `HelloResponse` class, `@PostMapping`, `@RequestBody` |
| 4 | `GET /hello/{name}` | `@PathVariable` |

## API Endpoints

| Method | URL | Response |
|--------|-----|----------|
| GET | `/hello` | `{"message":"Hello, World!"}` |
| GET | `/hello/{name}` | `{"message":"Hello, <name>"}` |
| POST | `/hello` (name in the request body) | `{"message":"Hello, <name>!"}` |

Example: `GET http://localhost:8080/hello/Sindhuja` returns

```json
{"message":"Hello, Sindhuja"}
```

## Project Structure

```
src/main
├── java/com/embarkx/FirstSpring
│   ├── FirstSpringApplication.java   # entry point, @SpringBootApplication
│   ├── HelloController.java          # the three endpoints
│   └── HelloResponse.java            # response object, converted to JSON
└── resources
    └── application.properties        # server.port=8080
src/test/java/com/embarkx/FirstSpring
└── FirstSpringApplicationTests.java  # checks the app context loads
```

## How It Works

**1. The application starts** (`FirstSpringApplication.java`)

```java
@SpringBootApplication
public class FirstSpringApplication {
    public static void main(String[] args) {
        SpringApplication.run(FirstSpringApplication.class, args);
    }
}
```

`@SpringBootApplication` turns on auto-configuration and component scanning,
which is the `@Configuration` + `@ComponentScan` idea from lesson 11, in one
annotation. It also starts an embedded web server.

**2. The controller** (`HelloController.java`)

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public HelloResponse hello() {
        return new HelloResponse("Hello, World!");
    }

    @GetMapping("/hello/{name}")
    public HelloResponse helloParam(@PathVariable String name) {
        return new HelloResponse("Hello, " + name);
    }

    @PostMapping("/hello")
    public HelloResponse helloPost(@RequestBody String name) {
        return new HelloResponse("Hello, " + name + "!");
    }
}
```

- `@RestController`: the class handles web requests and returns data, not pages.
- `@GetMapping` / `@PostMapping`: connect a URL and HTTP method to a method.
- `@PathVariable`: reads `{name}` from the URL.
- `@RequestBody`: reads the request body.

**3. JSON response** (`HelloResponse.java`)

The method returns a `HelloResponse` object. Spring Boot converts it to JSON
automatically using its getters, so `getMessage()` becomes `"message"`.

**4. Configuration** (`application.properties`)

```properties
server.port=8080
```

I changed this to `9090` to see the app move to another port, then set it
back to `8080`.

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Let Maven download the dependencies.
3. Run `FirstSpringApplication.java`.
4. Open `http://localhost:8080/hello` in a browser, or test with Postman.

For the POST endpoint, use Postman: method POST, URL
`http://localhost:8080/hello`, Body set to raw text with a name in it.

## Key Takeaway

With Spring Boot, a working web API is a few annotated classes. There is no
XML, no server setup and no manual wiring. Spring Boot configures it for me.
