# 15 - eCommerce Project (sb-ecom)

The main project of my Spring Boot course: an eCommerce backend built step by
step with Spring Boot. This first stage is a complete **Category REST API**
(create, read, update, delete) with a controller and a service layer.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Maven | Spring Web | IntelliJ IDEA | Postman

## What I Built, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `GET` all categories | `Category` model, `@RestController`, `@GetMapping` |
| 2 | `POST` a new category | `@PostMapping`, `@RequestBody` |
| 3 | Category service | Service layer, interface + implementation, `@Service`, `@Autowired` |
| 4 | Automatic category ids | Managing ids in the service |
| 5 | `DELETE` a category | `@DeleteMapping`, `@PathVariable`, `ResponseStatusException` |
| 6 | `ResponseEntity` for delete | Returning status codes and bodies |
| 7 | `ResponseEntity` for all endpoints | `HttpStatus.OK`, `HttpStatus.CREATED` |
| 8 | `PUT` update a category | `@PutMapping`, find-and-update logic |
| 9 | Shared `/api` prefix | `@RequestMapping` on the class |

## API Endpoints

| Method | URL | Body | Success response |
|--------|-----|------|------------------|
| GET | `/api/public/categories` | none | `200 OK` and the list of categories (JSON) |
| POST | `/api/public/categories` | `{"categoryName": "Electronics"}` | `201 Created` |
| PUT | `/api/public/categories/{categoryId}` | `{"categoryName": "Books"}` | `200 OK` |
| DELETE | `/api/admin/categories/{categoryId}` | none | `200 OK` |

If the category id does not exist, PUT and DELETE return `404 Not Found`
with a message.

Example `GET /api/public/categories`:

```json
[
  { "categoryId": 1, "categoryName": "Electronics" },
  { "categoryId": 2, "categoryName": "Books" }
]
```

## Project Structure

```
src/main/java/com/ecommerce/project
├── SbEcomApplication.java            # entry point, @SpringBootApplication
├── controller
│   └── CategoryController.java       # handles HTTP requests
├── service
│   ├── CategoryService.java          # interface: what the service can do
│   └── CategoryServiceImpl.java      # implementation: the business logic
└── model
    └── Category.java                 # categoryId, categoryName
```

## Architecture

```
Client (Postman)  ->  CategoryController  ->  CategoryService  ->  in-memory list
                      (HTTP, status codes)     (business logic)     (ArrayList)
```

- **Controller**: receives the request and builds the HTTP response.
- **Service**: holds the logic (create, find, update, delete). The controller
  depends on the `CategoryService` **interface**, which is the loose coupling
  idea from the first lessons, now used in a real project.
- **Model**: the `Category` object, converted to and from JSON automatically.

## Key Code

**Shared URL prefix** (`CategoryController.java`)

```java
@RestController
@RequestMapping("/api")
public class CategoryController {

    @Autowired
    private CategoryService categoryService;

    @GetMapping("/public/categories")
    public ResponseEntity<List<Category>> getAllCategories() {
        List<Category> categories = categoryService.getAllCategories();
        return new ResponseEntity<>(categories, HttpStatus.OK);
    }
}
```

**Not found handling** (`CategoryServiceImpl.java`)

```java
Category category = categories.stream()
        .filter(c -> c.getCategoryId().equals(categoryId))
        .findFirst()
        .orElseThrow(() -> new ResponseStatusException(
                HttpStatus.NOT_FOUND, "Resource not found"));
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Let Maven download the dependencies.
3. Run `SbEcomApplication.java`.
4. Test the endpoints in Postman at `http://localhost:8080/api/public/categories`.

## Notes

- Categories are stored in memory (an `ArrayList`), so the data is lost
  when the app restarts. A database is the next step for a real application.
- The `public` and `admin` parts of the URLs prepare for adding security later.

## What I Learned

- Building a REST API with the controller / service / model layers
- Using `ResponseEntity` to control status codes and response bodies
- Handling "not found" errors with `ResponseStatusException`
- Grouping endpoints with `@RequestMapping`
- Writing against an interface instead of a class
