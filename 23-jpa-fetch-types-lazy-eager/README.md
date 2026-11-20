# 23 - JPA Fetch Types: Lazy vs Eager (Social Media)

A follow-up to [19-jpa-relationships-and-cascading](../19-jpa-relationships-and-cascading).
It uses the same mini social network (users, profiles, posts and groups) to
learn **when JPA loads related data**: right away (**eager**) or only when it is
first used (**lazy**).

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Web | Spring Data JPA | Lombok | H2 Database | Maven | IntelliJ IDEA | Postman

## What Changed From Folder 19

| Change | Where | Why |
|--------|-------|-----|
| `fetch = FetchType.LAZY` written explicitly | `SocialUser.groups` (`@ManyToMany`) | Shows the fetch type in the code |
| `spring.jpa.show-sql=true` and `format_sql=true` | `application.properties` | Prints the SQL so I can see what Hibernate loads |
| `userRepository.findById(1L)` after the sample data | `DataInitializer` | A single read whose SQL I can inspect |

## Key Concepts

**Fetch type** decides when related entities are loaded from the database.

| Fetch type | Meaning |
|------------|---------|
| `EAGER` | Related data is loaded together with the main entity |
| `LAZY` | Related data is loaded only when I first access it |

**Defaults in JPA**

| Annotation | Default fetch type |
|------------|--------------------|
| `@OneToOne` | `EAGER` |
| `@ManyToOne` | `EAGER` |
| `@OneToMany` | `LAZY` |
| `@ManyToMany` | `LAZY` |

So in this project, loading a `SocialUser` also loads its `SocialProfile`
(`@OneToOne`). Its `posts` and `groups` are not loaded until they are used.
Writing `FetchType.LAZY` on `groups` does not change the behaviour because it is
already the default, but it makes the intention clear.

**Why it matters**
- Too much `EAGER` loading makes every query heavier than needed.
- `LAZY` loading is lighter, but reading the data after the session or
  transaction has closed causes a `LazyInitializationException`.
- Reading lazy data one entity at a time in a loop causes the **N+1 query
  problem**. This is a common interview topic.

## The Experiment

`DataInitializer` creates 3 users, 2 groups, 3 posts and 3 profiles. It then
runs:

```java
System.out.println("FETCHING SOCIAL USER");
userRepository.findById(1L);
```

With `show-sql` on, look at the console after the line `FETCHING SOCIAL USER`.
It shows the `select` that Hibernate runs for the user. Compare it with what
happens when you change `groups` to `FetchType.EAGER`, or add
`fetch = FetchType.LAZY` to the `@OneToOne` profile, and run again.

## The Data Model

```
SocialProfile  1 --- 1  SocialUser  1 --- *  Post
                           *
                           |
                           *
                      SocialGroup
```

The relationships and cascade types are explained in folder 19.

## API Endpoints

| Method | URL | Response |
|--------|-----|----------|
| GET | `/social/users` | All users with their profile, posts and groups |
| POST | `/social/users` | The saved user |
| DELETE | `/social/users/{userId}` | Deletes the user (and the profile, by cascade) |

## Project Structure

```
src/main/java/com/social/media
├── MediaApplication.java
├── DataInitializer.java        # sample data + the fetch experiment
├── controllers/SocialController.java
├── services/SocialService.java
├── repositories                # one JpaRepository per entity
└── models                      # SocialUser, SocialProfile, Post, SocialGroup
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if asked, and enable annotation processing.
3. Run `MediaApplication.java` and read the SQL printed in the console.
4. Open `GET http://localhost:8080/social/users` in Postman or the browser.

## Next Steps
- Try `@EntityGraph` or `JOIN FETCH` to load related data in one query.
- Reproduce a `LazyInitializationException` on purpose and fix it.
