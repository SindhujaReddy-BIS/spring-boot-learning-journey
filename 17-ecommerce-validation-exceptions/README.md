# 17 - eCommerce Project: Validation and Exception Handling

The next stage of my eCommerce project (`sb-ecom`). In
[16-ecommerce-jpa-h2](../16-ecommerce-jpa-h2) the API saved categories in a
database. Here it gets **input validation**, **custom exceptions** and one
**global exception handler**, so every error returns a clear message and the
right HTTP status.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Web | Spring Data JPA | Bean Validation | Lombok | H2 Database | Maven | IntelliJ IDEA | Postman

## What Changed, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | Validate the category name | `spring-boot-starter-validation`, `@NotBlank`, `@Size`, `@Valid` |
| 2 | Global handler for validation errors | `@RestControllerAdvice`, `@ExceptionHandler`, `MethodArgumentNotValidException` |
| 3 | Less boilerplate in the model | Lombok: `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor` |
| 4 | `ResourceNotFoundException` | Custom exception with a clear message, 404 response |
| 5 | `APIException` | Custom exception for business rules, 400 response |
| 6 | Duplicate name check | Derived query `findByCategoryName` |
| 7 | Cleaner controllers | Removed `try/catch`; the global handler deals with errors |

## Before and After

**Before** (lesson 16): the controller caught errors itself.

```java
try {
    String status = categoryService.deleteCategory(categoryId);
    return ResponseEntity.status(HttpStatus.OK).body(status);
} catch (ResponseStatusException e) {
    return new ResponseEntity<>(e.getReason(), e.getStatusCode());
}
```

**After** (this lesson): the service throws, the global handler responds.

```java
String status = categoryService.deleteCategory(categoryId);
return new ResponseEntity<>(status, HttpStatus.OK);
```

## API Endpoints

| Method | URL | Body | Success response |
|--------|-----|------|------------------|
| GET | `/api/public/categories` | none | `200 OK` and the list of categories |
| POST | `/api/public/categories` | `{"categoryName": "Electronics"}` | `201 Created` |
| PUT | `/api/public/categories/{categoryId}` | `{"categoryName": "Books123"}` | `200 OK` |
| DELETE | `/api/admin/categories/{categoryId}` | none | `200 OK` |

## Error Responses

| Situation | Status | Response |
|-----------|--------|----------|
| Name shorter than 5 characters | `400 Bad Request` | `{"categoryName": "Category name must contain atleast 5 characters"}` |
| Name is blank | `400 Bad Request` | JSON with the field and its message |
| Category name already exists | `400 Bad Request` | `Category with the name Books already exists !!!` |
| No categories yet (GET all) | `400 Bad Request` | `No category created till now.` |
| Id not found (PUT / DELETE) | `404 Not Found` | `Category not found with categoryId: 99` |

## Project Structure

```
src/main/java/com/ecommerce/project
├── SbEcomApplication.java
├── controller
│   └── CategoryController.java          # @Valid on request bodies, no try/catch
├── service
│   ├── CategoryService.java
│   └── CategoryServiceImpl.java         # throws the custom exceptions
├── repositories
│   └── CategoryRepository.java          # adds findByCategoryName
├── model
│   └── Category.java                    # validation annotations + Lombok
└── exceptions
    ├── MyGlobalExceptionHandler.java    # one place for all error responses
    ├── ResourceNotFoundException.java   # "not found" errors
    └── APIException.java                # business rule errors
```

## Key Code

**Validation on the model** (`Category.java`)

```java
@Entity(name = "categories")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long categoryId;

    @NotBlank
    @Size(min = 5, message = "Category name must contain atleast 5 characters")
    private String categoryName;
}
```

Lombok writes the getters, setters and constructors, so the class stays short.

**Turning validation on** (`CategoryController.java`)

```java
@PostMapping("/public/categories")
public ResponseEntity<String> createCategory(@Valid @RequestBody Category category) {
    categoryService.createCategory(category);
    return new ResponseEntity<>("Category added successfully", HttpStatus.CREATED);
}
```

**The global exception handler** (`MyGlobalExceptionHandler.java`)

```java
@RestControllerAdvice
public class MyGlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> myMethodArgumentNotValidException(
            MethodArgumentNotValidException e) {
        Map<String, String> response = new HashMap<>();
        e.getBindingResult().getAllErrors().forEach(err -> {
            String fieldName = ((FieldError) err).getField();
            response.put(fieldName, err.getDefaultMessage());
        });
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> myResourceNotFoundException(ResourceNotFoundException e) {
        return new ResponseEntity<>(e.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(APIException.class)
    public ResponseEntity<String> myAPIException(APIException e) {
        return new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }
}
```

**Throwing custom exceptions** (`CategoryServiceImpl.java`)

```java
Category category = categoryRepository.findById(categoryId)
        .orElseThrow(() -> new ResourceNotFoundException("Category", "categoryId", categoryId));
```

```java
Category savedCategory = categoryRepository.findByCategoryName(category.getCategoryName());
if (savedCategory != null)
    throw new APIException("Category with the name " + category.getCategoryName() + " already exists !!!");
```

**Derived query** (`CategoryRepository.java`)

```java
Category findByCategoryName(String categoryName);
```

Spring Data JPA builds the query from the method name. No SQL needed.

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if IntelliJ asks, and enable annotation processing.
3. Let Maven download the dependencies.
4. Run `SbEcomApplication.java`.
5. Test in Postman at `http://localhost:8080/api/public/categories`. Try a short
   name like `abc`, a duplicate name, and an id that does not exist.

## What I Learned

- Validating request bodies with Bean Validation annotations and `@Valid`
- Handling every error in one place with `@RestControllerAdvice`
- Writing custom exceptions and mapping them to HTTP status codes
- Using derived queries in Spring Data JPA
- Reducing boilerplate with Lombok
- Keeping controllers clean by moving error handling out of them
