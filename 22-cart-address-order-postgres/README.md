# 22 - Cart, Addresses and Orders with PostgreSQL

The next stage of my eCommerce project (`sb-ecom`). In
[21-spring-security-jwt-auth](../21-spring-security-jwt-auth) users could sign
in. Now a signed-in user can **manage addresses, fill a shopping cart and place
an order with a payment record**. The database also moves from in-memory H2 to
**PostgreSQL**.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Security 6 + JWT | Spring Data JPA | PostgreSQL | ModelMapper | Lombok | Maven | Docker Compose | Postman

## What Changed, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `Address` entity and `AddressController` | CRUD for a user's addresses, validation annotations |
| 2 | `Cart` and `CartItem` entities | `@OneToOne` user to cart, `@OneToMany` cart to items, `orphanRemoval` |
| 3 | Add a product to the cart | Stock and duplicate checks, running `totalPrice` |
| 4 | Increase or decrease quantity, remove an item | `PUT` with an operation, `DELETE` |
| 5 | `AuthUtil` | Gets the logged-in user's id and email from the `SecurityContext` |
| 6 | `Order`, `OrderItem` and `Payment` entities | `@OneToOne` order to payment, `@ManyToOne` order to address |
| 7 | `placeOrder` in `OrderServiceImpl` | `@Transactional`: cart to order, stock reduced, cart emptied |
| 8 | PostgreSQL | `ddl-auto=update`, credentials from environment variables |

## API Endpoints

All endpoints below need a signed-in user (sign in with `/api/auth/signin`
first).

**Addresses**

| Method | URL | Body | Response |
|--------|-----|------|----------|
| POST | `/api/addresses` | `AddressDTO` JSON | `201 Created`, the new address |
| GET | `/api/addresses` | none | All addresses |
| GET | `/api/addresses/{addressId}` | none | One address |
| GET | `/api/users/addresses` | none | Addresses of the logged-in user |
| PUT | `/api/addresses/{addressId}` | `AddressDTO` JSON | The updated address |
| DELETE | `/api/addresses/{addressId}` | none | Confirmation message |

**Cart**

| Method | URL | Response |
|--------|-----|----------|
| POST | `/api/carts/products/{productId}/quantity/{quantity}` | `201 Created`, the cart |
| GET | `/api/carts` | All carts |
| GET | `/api/carts/users/cart` | The logged-in user's cart |
| PUT | `/api/cart/products/{productId}/quantity/{operation}` | `operation` = `add` (+1) or `delete` (-1) |
| DELETE | `/api/carts/{cartId}/product/{productId}` | Item removed from the cart |

**Orders**

| Method | URL | Body | Response |
|--------|-----|------|----------|
| POST | `/api/order/users/payments/{paymentMethod}` | `OrderRequestDTO` JSON | `OrderDTO` with items, payment and status |

Example address body:

```json
{
  "street": "12 MG Road",
  "buildingName": "Sunrise Apartments",
  "city": "Hyderabad",
  "state": "Telangana",
  "country": "India",
  "pincode": "500001"
}
```

Example order body (the `pg*` fields describe a payment gateway result):

```json
{
  "addressId": 1,
  "paymentMethod": "Online",
  "pgName": "Stripe",
  "pgPaymentId": "pay_123",
  "pgStatus": "succeeded",
  "pgResponseMessage": "Payment successful"
}
```

## How an Order Is Placed

1. Find the cart of the logged-in user (by email) and the chosen address.
2. Create an `Order` with today's date, the cart total and the status
   `Order Accepted !`.
3. Save a `Payment` linked to the order.
4. Copy each `CartItem` into an `OrderItem`, keeping the price and discount at
   the time of purchase.
5. Reduce each product's stock and remove the item from the cart.

All of this runs in one `@Transactional` method, so it is saved fully or not at
all.

## Error Responses

| Situation | Message |
|-----------|---------|
| Product already in the cart | `Product <name> already exists in the cart` |
| Product out of stock | `<name> is not available` |
| Requested quantity is more than the stock | `Please, make an order of the <name> less than or equal to the quantity <n>` |
| Ordering with an empty cart | `Cart is empty` |
| Unknown cart, product or address | `404`, `<Resource> not found with <field>: <value>` |

## Database Design

```
users 1---1 carts 1---* cart_items *---1 products
users 1---* addresses
orders *---1 addresses
orders 1---1 payments
orders 1---* order_items *---1 products
```

## Project Structure (new in this folder)

```
src/main/java/com/ecommerce/project
├── controller   AddressController, CartController, OrderController
├── service      Address/Cart/Order service + implementation
├── repositories Address, Cart, CartItem, Order, OrderItem, Payment
├── model        Address, Cart, CartItem, Order, OrderItem, Payment
├── payload      AddressDTO, CartDTO, CartItemDTO, OrderDTO, OrderItemDTO,
│                OrderRequestDTO, PaymentDTO
└── util         AuthUtil: logged-in user's id, email and entity
```

## How to Run

1. Start PostgreSQL (needs Docker), using your own password:
   ```bash
   DB_PASSWORD=yourpassword docker compose up -d
   ```
   On Windows Command Prompt:
   ```bat
   set DB_PASSWORD=yourpassword
   docker compose up -d
   ```
2. Set the same `DB_PASSWORD` environment variable for the app. In IntelliJ:
   Run, Edit Configurations, Environment variables.
3. Run `SbEcomApplication.java`. Tables are created automatically.
4. In Postman: sign in (`admin` / `adminPass`), create a category and product,
   add the product to the cart, create an address, then place the order.

## Notes and Next Steps

- Role-based URL rules are still to do. Right now any signed-in user can call
  admin-style endpoints and `GET /api/carts` (all carts).
- The payment is only recorded. The `pg*` values come from the request, with no
  real gateway call yet. A payment gateway such as Stripe is a possible next step.
- Add unit and integration tests for the cart and order services.
