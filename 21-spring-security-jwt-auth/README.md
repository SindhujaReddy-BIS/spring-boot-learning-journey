# 21 - Spring Security with JWT: Sign Up, Sign In and Sign Out

The next stage of my eCommerce project (`sb-ecom`). In
[20-ecommerce-products-and-images](../20-ecommerce-products-and-images) the API
was open to everyone. Here it gets **authentication**: users register, sign in
and receive a **JWT stored in a cookie**. Every other request is checked by a
filter that validates that token.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Security 6 | JWT (jjwt 0.12.6) | BCrypt | Spring Data JPA | ModelMapper | Lombok | H2 Database | Maven | Postman

## What Changed, Step by Step

| Step | What I added | Concept |
|------|--------------|---------|
| 1 | `User`, `Role` and `AppRole` entities | `@ManyToMany` users and roles, `@Enumerated` |
| 2 | `UserRepository` and `RoleRepository` | Derived queries such as `findByUserName` |
| 3 | `UserDetailsImpl` and `UserDetailsServiceImpl` | Connecting my `User` to Spring Security |
| 4 | `JwtUtils` | Generate, read and validate a JWT, and put it in a cookie |
| 5 | `AuthTokenFilter` | A `OncePerRequestFilter` that authenticates each request from the cookie |
| 6 | `AuthEntryPointJwt` | Returns a JSON `401` for unauthenticated requests |
| 7 | `WebSecurityConfig` | Stateless `SecurityFilterChain`, BCrypt, `AuthenticationManager` |
| 8 | `AuthController` | `signin`, `signup`, `signout` and user info endpoints |
| 9 | `Product` linked to a seller (`User`) | `@ManyToOne` from product to user |

## How the Login Flow Works

1. `POST /api/auth/signin` with a username and password.
2. `AuthenticationManager` checks the password against the BCrypt hash.
3. `JwtUtils` creates a JWT and returns it in a `Set-Cookie` header.
4. On later requests the browser (or Postman) sends the cookie back.
5. `AuthTokenFilter` validates the token, loads the user and sets the
   `SecurityContext`, so the request is treated as authenticated.
6. `POST /api/auth/signout` replaces the cookie with an empty one.

## API Endpoints

**Authentication** (public)

| Method | URL | Body | Response |
|--------|-----|------|----------|
| POST | `/api/auth/signup` | `username`, `email`, `password`, optional `role` | `200 OK`, `User registered successfully!` |
| POST | `/api/auth/signin` | `username`, `password` | `200 OK`, user id, roles and the `Set-Cookie` header |
| POST | `/api/auth/signout` | none | `200 OK`, `You've been signed out!` |
| GET | `/api/auth/username` | none | Current username, or empty text |
| GET | `/api/auth/user` | none | Current user's id, username and roles |

**Categories and Products**: the same endpoints as folder 20. They now **require
a signed-in user**.

Example `POST /api/auth/signup` body:

```json
{
  "username": "john",
  "email": "john@example.com",
  "password": "secret123",
  "role": ["seller"]
}
```

`role` accepts `admin`, `seller` or `user`. If it is left out, the user gets
`ROLE_USER`.

## Demo Users

On start-up a `CommandLineRunner` creates the three roles and these users
(in-memory H2 only, for testing):

| Username | Password | Roles |
|----------|----------|-------|
| `user1` | `password1` | `ROLE_USER` |
| `seller1` | `password2` | `ROLE_SELLER` |
| `admin` | `adminPass` | `ROLE_USER`, `ROLE_SELLER`, `ROLE_ADMIN` |

## Error Responses

| Situation | Status | Body |
|-----------|--------|------|
| Wrong username or password | `404 Not Found` | `{"message": "Bad credentials", "status": false}` |
| Username or email already used | `400 Bad Request` | `Error: Username is already taken!` / `Error: Email is already in use!` |
| No valid token on a protected URL | `401 Unauthorized` | JSON error from `AuthEntryPointJwt` |

## Project Structure

```
src/main/java/com/ecommerce/project
├── SbEcomApplication.java
├── config                    # ModelMapper bean, page defaults
├── controller
│   ├── AuthController.java   # NEW: signin, signup, signout
│   ├── CategoryController.java
│   └── ProductController.java
├── model
│   ├── User.java             # NEW
│   ├── Role.java / AppRole.java   # NEW
│   ├── Address.java          # NEW
│   ├── Category.java
│   └── Product.java          # now has a seller (User)
├── repositories              # UserRepository, RoleRepository are NEW
├── security                  # NEW package
│   ├── WebSecurityConfig.java        # filter chain, BCrypt, demo data
│   ├── jwt
│   │   ├── JwtUtils.java
│   │   ├── AuthTokenFilter.java
│   │   └── AuthEntryPointJwt.java
│   ├── request               # LoginRequest, SignupRequest
│   ├── response              # MessageResponse, UserInfoResponse
│   └── services              # UserDetailsImpl, UserDetailsServiceImpl
├── service, payload, exceptions   # as in folder 20
```

## Key Code

**Stateless filter chain** (`WebSecurityConfig.java`)

```java
http.csrf(csrf -> csrf.disable())
    .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/auth/**").permitAll()
        .anyRequest().authenticated());

http.addFilterBefore(authenticationJwtTokenFilter(),
        UsernamePasswordAuthenticationFilter.class);
```

No server session is kept. Each request proves who it is with the token.

**Creating the token** (`JwtUtils.java`)

```java
return Jwts.builder()
        .subject(username)
        .issuedAt(new Date())
        .expiration(new Date(new Date().getTime() + jwtExpirationMs))
        .signWith(key())
        .compact();
```

## Configuration

The JWT secret is read from the `JWT_SECRET` environment variable. The value in
`application.properties` is only a fallback for local learning.

```bash
export JWT_SECRET=<a long random Base64 string>
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if asked, and enable annotation processing.
3. Run `SbEcomApplication.java`.
4. In Postman: call `POST /api/auth/signin` with a demo user. Postman stores
   the cookie.
5. Call any category or product endpoint. Then call `signout` and try again to
   see the `401`.

H2 console: `http://localhost:8080/h2-console` (JDBC URL `jdbc:h2:mem:test`).

## Notes and Next Steps

- Roles are stored and returned, but the URLs are not yet restricted by role
  (`/api/admin/**` needs only a signed-in user). Role-based rules come next.
- The cookie is currently readable by JavaScript (`httpOnly(false)`) because
  this stage is tested with Postman. Real use should set `httpOnly(true)` and
  `secure(true)`.
- Use a real database (PostgreSQL) and a strong secret from the environment in
  production.
