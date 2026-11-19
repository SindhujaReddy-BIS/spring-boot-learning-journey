# 20 - eCommerce Project: Products, Search and Image Upload

The next stage of my eCommerce project (`sb-ecom`). In
[18-ecommerce-dto-pagination](../18-ecommerce-dto-pagination) the API managed
categories with DTOs and pagination. Here it gets a full **Product** feature:
products that belong to categories, **pagination and sorting**, **search**, and
**image upload**.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Web | Spring Data JPA | ModelMapper | Lombok | H2 Database | Maven | IntelliJ IDEA | Postman

## What Changed, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `Product` entity linked to `Category` | `@ManyToOne` / `@OneToMany`, `@JoinColumn`, `CascadeType.ALL` |
| 2 | `ProductDTO` and `ProductResponse` | DTO pattern and page details for products |
| 3 | Add a product to a category | `POST`, price after discount (`specialPrice`) |
| 4 | List all products | Pagination and sorting (challenge) |
| 5 | Products by category | Derived query `findByCategoryOrderByPriceAsc` |
| 6 | Search by keyword | `findByProductNameLikeIgnoreCase` |
| 7 | Update and delete a product | `@PutMapping`, `@DeleteMapping` |
| 8 | Upload a product image | `MultipartFile`, `FileService`, `@Value` property |
| 9 | Small cleanup | `totalpages` renamed to `totalPages` in the responses |

## API Endpoints

**Categories** (unchanged from folder 18)

| Method | URL |
|--------|-----|
| GET | `/api/public/categories` |
| POST | `/api/public/categories` |
| PUT | `/api/public/categories/{categoryId}` |
| DELETE | `/api/admin/categories/{categoryId}` |

**Products**

| Method | URL | Body | Response |
|--------|-----|------|----------|
| POST | `/api/admin/categories/{categoryId}/product` | `ProductDTO` JSON | `201 Created`, the new product |
| GET | `/api/public/products` | none | `ProductResponse` (paginated) |
| GET | `/api/public/categories/{categoryId}/products` | none | `ProductResponse` for that category, cheapest first |
| GET | `/api/public/products/keyword/{keyword}` | none | `ProductResponse` of matching products |
| PUT | `/api/admin/products/{productId}` | `ProductDTO` JSON | `200 OK`, the updated product |
| DELETE | `/api/admin/products/{productId}` | none | `200 OK`, the deleted product |
| PUT | `/api/products/{productId}/image` | form-data, key `image` (file) | `200 OK`, the product with the new image name |

The three `GET` list endpoints accept the same optional query parameters:
`pageNumber` (default `0`), `pageSize` (default `50`), `sortBy` (default
`productId`) and `sortOrder` (`asc` or `desc`).

Example `POST /api/admin/categories/1/product` body:

```json
{
  "productName": "iPhone 15",
  "description": "Latest Apple phone",
  "quantity": 10,
  "price": 80000,
  "discount": 10
}
```

The response includes the calculated price after discount and a default image:

```json
{
  "productId": 1,
  "productName": "iPhone 15",
  "image": "default.png",
  "description": "Latest Apple phone",
  "quantity": 10,
  "price": 80000.0,
  "discount": 10.0,
  "specialPrice": 72000.0
}
```

Example `ProductResponse`:

```json
{
  "content": [ { "productId": 1, "productName": "iPhone 15", "...": "..." } ],
  "pageNumber": 0,
  "pageSize": 50,
  "totalElements": 1,
  "totalPages": 1,
  "lastPage": true
}
```

## Error Responses

| Situation | Status | Message |
|-----------|--------|---------|
| Category or product id not found | `404 Not Found` | `Category not found with categoryId: 99` |
| Product name already in that category | `400 Bad Request` | `Product already exist!!` |
| Category has no products | `400 Bad Request` | `<category name> category does not have any products` |
| No product matches the keyword | `400 Bad Request` | `Products not found with keyword: <keyword>` |

Errors are returned as `{"message": "...", "status": false}`.

## Project Structure

```
src/main/java/com/ecommerce/project
├── SbEcomApplication.java
├── config
│   ├── AppConfig.java                   # ModelMapper bean
│   └── AppConstants.java                # default page and sort values
├── controller
│   ├── CategoryController.java
│   └── ProductController.java           # NEW
├── service
│   ├── CategoryService.java / CategoryServiceImpl.java
│   ├── ProductService.java / ProductServiceImpl.java    # NEW
│   └── FileService.java / FileServiceImpl.java          # NEW: saves uploaded images
├── repositories
│   ├── CategoryRepository.java
│   └── ProductRepository.java           # NEW
├── model
│   ├── Category.java                    # now has a list of products
│   └── Product.java                     # NEW
├── payload
│   ├── CategoryDTO.java / CategoryResponse.java
│   ├── ProductDTO.java / ProductResponse.java           # NEW
│   └── APIResponse.java
└── exceptions
    ├── MyGlobalExceptionHandler.java
    ├── ResourceNotFoundException.java
    └── APIException.java
src/main/resources
└── application.properties               # adds project.image=images/
```

## Key Code

**Category and Product relationship** (`Category.java` and `Product.java`)

```java
// Category.java
@OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
private List<Product> products;

// Product.java
@ManyToOne
@JoinColumn(name = "category_id")
private Category category;
```

One category has many products. The `category_id` column lives in the product
table. This is the same relationship idea as in
[19-jpa-relationships-and-cascading](../19-jpa-relationships-and-cascading).

**Price after discount** (`ProductServiceImpl.java`)

```java
double specialPrice = product.getPrice() -
        ((product.getDiscount() * 0.01) * product.getPrice());
product.setSpecialPrice(specialPrice);
```

**Derived queries** (`ProductRepository.java`)

```java
Page<Product> findByCategoryOrderByPriceAsc(Category category, Pageable pageDetails);

Page<Product> findByProductNameLikeIgnoreCase(String keyword, Pageable pageDetails);
```

Spring Data JPA builds the SQL from the method names. The keyword search wraps
the word in `%...%`, so `phone` finds `iPhone 15`, ignoring upper and lower case.

**Image upload** (`FileServiceImpl.java`)

```java
String randomId = UUID.randomUUID().toString();
String fileName = randomId.concat(originalFileName.substring(originalFileName.lastIndexOf('.')));
String filePath = path + File.separator + fileName;
Files.copy(file.getInputStream(), Paths.get(filePath));
```

The file gets a random name so two uploads never overwrite each other. The
folder comes from `application.properties`:

```properties
project.image=images/
```

and is read in the service with `@Value("${project.image}")`.

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if IntelliJ asks, and enable annotation processing.
3. Let Maven download the dependencies.
4. Run `SbEcomApplication.java`.
5. In Postman: create a category, add a product to it, then try the list, search,
   update and delete requests.
6. To test the image upload: `PUT http://localhost:8080/api/products/1/image`,
   Body > form-data, key `image`, type File, and choose a picture.

## Notes

- Uploaded pictures are saved in the `images/` folder next to the project. The
  folder is listed in `.gitignore`, so it is not uploaded to GitHub.
- Spring Boot limits uploads to 1 MB by default.
- H2 is an in-memory database, so the data is cleared when the app restarts.

## Next Steps

- Move the validation annotations (`@NotBlank`, `@Size`) from the `Category` and
  `Product` entities to `CategoryDTO` and `ProductDTO`, because `@Valid` checks
  the DTOs.

## What I Learned

- Modeling a one-to-many relationship (category and products) in a real project
- Writing derived queries for filtering, ordering and searching
- Paginating and sorting product lists
- Uploading files with `MultipartFile` and reading settings with `@Value`
- Keeping the same DTO, service and exception structure across features
