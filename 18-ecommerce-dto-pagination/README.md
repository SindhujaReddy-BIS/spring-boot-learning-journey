# 18 - eCommerce Project: DTOs, ModelMapper, Pagination and API Responses

The next stage of my eCommerce project (`sb-ecom`). In
[17-ecommerce-validation-exceptions](../17-ecommerce-validation-exceptions) the
API validated input and handled errors. Here the API stops exposing the
database entity directly. It uses **DTOs**, returns **paginated and sorted**
lists, and sends errors in a consistent **JSON format**.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Web | Spring Data JPA | ModelMapper | Lombok | H2 Database | Maven | IntelliJ IDEA | Postman

## What Changed, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `CategoryDTO` | DTO pattern: the API no longer exposes the `Category` entity |
| 2 | ModelMapper | Converting entity to DTO and back with one line (`AppConfig` bean) |
| 3 | `CategoryResponse` | A response wrapper with the list plus page details |
| 4 | Pagination | `Pageable`, `PageRequest`, `Page`, `@RequestParam` |
| 5 | Sorting | `Sort.by(...)`, ascending and descending |
| 6 | `AppConstants` | Default values for page number, size and sort in one place |
| 7 | `APIResponse` | Consistent JSON for error messages |

## Before and After

| | 17 | 18 |
|---|---|---|
| Request / response body | `Category` entity | `CategoryDTO` |
| `GET` all categories | Plain list | `CategoryResponse` with page details, sortable |
| `POST` / `PUT` / `DELETE` response | Text message | The category as JSON |
| Error response | Plain text | `{"message": "...", "status": false}` |

## API Endpoints

| Method | URL | Body | Success response |
|--------|-----|------|------------------|
| GET | `/api/public/categories` | none | `200 OK`, `CategoryResponse` |
| POST | `/api/public/categories` | `{"categoryName": "Electronics"}` | `201 Created`, the new category |
| PUT | `/api/public/categories/{categoryId}` | `{"categoryName": "Books123"}` | `200 OK`, the updated category |
| DELETE | `/api/admin/categories/{categoryId}` | none | `200 OK`, the deleted category |

**Query parameters for `GET /api/public/categories`** (all optional)

| Parameter | Default | Meaning |
|-----------|---------|---------|
| `pageNumber` | `0` | Which page to return (starts at 0) |
| `pageSize` | `50` | How many categories per page |
| `sortBy` | `categoryId` | Field to sort by |
| `sortOrder` | `asc` | `asc` or `desc` |

Example: `GET /api/public/categories?pageNumber=0&pageSize=5&sortBy=categoryName&sortOrder=desc`

```json
{
  "content": [
    { "categoryId": 2, "categoryName": "Electronics" },
    { "categoryId": 1, "categoryName": "Books123" }
  ],
  "pageNumber": 0,
  "pageSize": 5,
  "totalElements": 2,
  "totalpages": 1,
  "lastPage": true
}
```

**Error example** (`404 Not Found`)

```json
{ "message": "Category not found with categoryId: 99", "status": false }
```

## Project Structure

```
src/main/java/com/ecommerce/project
├── SbEcomApplication.java
├── config
│   ├── AppConfig.java                   # ModelMapper bean
│   └── AppConstants.java                # default page and sort values
├── controller
│   └── CategoryController.java          # works with DTOs, reads page parameters
├── service
│   ├── CategoryService.java
│   └── CategoryServiceImpl.java         # pagination, sorting, entity <-> DTO mapping
├── repositories
│   └── CategoryRepository.java
├── model
│   └── Category.java                    # JPA entity (database only)
├── payload
│   ├── CategoryDTO.java                 # what the client sends and receives
│   ├── CategoryResponse.java            # list + page details
│   └── APIResponse.java                 # message + status for errors
└── exceptions
    ├── MyGlobalExceptionHandler.java    # now returns APIResponse
    ├── ResourceNotFoundException.java
    └── APIException.java
```

## Key Code

**Why a DTO?** The entity maps to a database table. The DTO is what the API
shows to clients, so the database design can change without breaking the API.

```java
@Data @NoArgsConstructor @AllArgsConstructor
public class CategoryDTO {
    private Long categoryId;
    private String categoryName;
}
```

**ModelMapper bean** (`AppConfig.java`)

```java
@Configuration
public class AppConfig {
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

**Pagination and sorting** (`CategoryServiceImpl.java`)

```java
Sort sortByAndOrder = sortOrder.equalsIgnoreCase("asc")
        ? Sort.by(sortBy).ascending()
        : Sort.by(sortBy).descending();

Pageable pageDetails = PageRequest.of(pageNumber, pageSize, sortByAndOrder);
Page<Category> categoryPage = categoryRepository.findAll(pageDetails);
```

**Entity to DTO with ModelMapper**

```java
List<CategoryDTO> categoryDTOS = categories.stream()
        .map(category -> modelMapper.map(category, CategoryDTO.class))
        .toList();
```

**Page parameters with defaults** (`CategoryController.java`)

```java
@GetMapping("/public/categories")
public ResponseEntity<CategoryResponse> getAllCategories(
        @RequestParam(name = "pageNumber", defaultValue = AppConstants.PAGE_NUMBER, required = false) Integer pageNumber,
        @RequestParam(name = "pageSize", defaultValue = AppConstants.PAGE_SIZE, required = false) Integer pageSize,
        @RequestParam(name = "sortBy", defaultValue = AppConstants.SORT_CATEGORIES_BY, required = false) String sortBy,
        @RequestParam(name = "sortOrder", defaultValue = AppConstants.SORT_DIR, required = false) String sortOrder) {
    ...
}
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if IntelliJ asks, and enable annotation processing.
3. Let Maven download the dependencies.
4. Run `SbEcomApplication.java`.
5. Test in Postman at `http://localhost:8080/api/public/categories`. Create a few
   categories, then try `pageSize`, `sortBy` and `sortOrder`.

## Next Steps

- Move the validation annotations (`@NotBlank`, `@Size`) from the `Category`
  entity to `CategoryDTO`, because `@Valid` now checks the DTO.

## What I Learned

- Separating the database entity from the API model with DTOs
- Mapping objects automatically with ModelMapper
- Paginating and sorting results with Spring Data JPA
- Returning a consistent JSON format for responses and errors
- Keeping default values in a constants class
