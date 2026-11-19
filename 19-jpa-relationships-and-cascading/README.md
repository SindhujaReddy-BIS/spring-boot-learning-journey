# 19 - JPA Relationships and Cascading (Social Media)

A small practice project (`media`) to learn how JPA connects tables together.
It models a mini social network with users, profiles, posts and groups, and
shows every relationship type and how **cascading** works.

## Tech Stack

Java 17 | Spring Boot 3.2.5 | Spring Web | Spring Data JPA | Lombok | H2 Database | Maven | IntelliJ IDEA | Postman

## The Data Model

```
SocialProfile  1 --- 1  SocialUser  1 --- *  Post
                           *
                           |
                           *
                      SocialGroup
```

| Relationship | Between | Annotation | Where the foreign key lives |
|--------------|---------|------------|-----------------------------|
| One to One | `SocialUser` and `SocialProfile` | `@OneToOne` | `social_user` column in the profile table |
| One to Many / Many to One | `SocialUser` and `Post` | `@OneToMany` and `@ManyToOne` | `user_id` column in the post table |
| Many to Many | `SocialUser` and `SocialGroup` | `@ManyToMany` | Join table `user_group` |

## Key Concepts

**Owning side and `mappedBy`**: in every relationship one side owns the
foreign key and the other side points to it with `mappedBy`.

| Relationship | Owning side | Inverse side (`mappedBy`) |
|--------------|-------------|---------------------------|
| User - Profile | `SocialProfile.user` (`@JoinColumn`) | `SocialUser.socialProfile` |
| User - Posts | `Post.socialUser` (`@JoinColumn`) | `SocialUser.posts` |
| User - Groups | `SocialUser.groups` (`@JoinTable`) | `SocialGroup.socialUsers` |

**Cascading**: when I save or delete one entity, cascade tells JPA what to do
with the related entities. This is on `SocialUser`:

```java
@OneToOne(mappedBy = "user",
          cascade = {CascadeType.REMOVE, CascadeType.PERSIST, CascadeType.MERGE})
private SocialProfile socialProfile;

@OneToMany(mappedBy = "socialUser",
           cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private List<Post> posts = new ArrayList<>();
```

| Cascade type | Meaning |
|--------------|---------|
| `PERSIST` | Saving the user also saves the profile / posts |
| `MERGE` | Updating the user also updates the profile / posts |
| `REMOVE` | Deleting the user also deletes the related entity |

Here the profile uses all three, so deleting a user deletes their profile.
Posts use only `PERSIST` and `MERGE`, so they are not deleted with the user.

**`@JsonIgnore`**: the relationships point both ways (a user has posts, a post
has a user). Without `@JsonIgnore` on one side, converting to JSON would loop
forever. `@JsonIgnore` on `Post.socialUser`, `SocialProfile.user` and
`SocialGroup.socialUsers` breaks the loop.

## Project Structure

```
src/main/java/com/social/media
├── MediaApplication.java          # entry point
├── DataInitializer.java           # creates sample data on startup
├── controllers
│   └── SocialController.java      # REST endpoints for users
├── services
│   └── SocialService.java
├── repositories                   # one JpaRepository per entity
│   ├── SocialUserRepository.java
│   ├── SocialProfileRepository.java
│   ├── PostRepository.java
│   └── SocialGroupRepository.java
└── models                         # the entities
    ├── SocialUser.java
    ├── SocialProfile.java
    ├── Post.java
    └── SocialGroup.java
```

## Sample Data

`DataInitializer` is a `CommandLineRunner` that runs when the app starts. It
creates 3 users, 2 groups, 3 posts and 3 profiles:

| User | Group(s) |
|------|----------|
| user 1 | group 1 |
| user 2 | group 1 and group 2 |
| user 3 | group 2 |

Each user also gets one post and one profile.

## API Endpoints

| Method | URL | Body | Response |
|--------|-----|------|----------|
| GET | `/social/users` | none | `200 OK`, all users with their profile, posts and groups |
| POST | `/social/users` | see below | `201 Created`, the saved user |
| DELETE | `/social/users/{userId}` | none | `200 OK`, `Deleted Successfully` |

Example `POST /social/users` body. Because of `CascadeType.PERSIST`, the
profile is saved together with the user:

```json
{
  "socialProfile": { "description": "My first profile" }
}
```

## How to Run

1. Open this folder in IntelliJ IDEA (Java 17+, Maven).
2. Install the **Lombok** plugin if IntelliJ asks, and enable annotation processing.
3. Let Maven download the dependencies.
4. Run `MediaApplication.java`.
5. Open `http://localhost:8080/social/users` to see the sample data.

## Viewing the Tables

Open the H2 console at `http://localhost:8080/h2-console` with the JDBC URL
`jdbc:h2:mem:test`. Look at the tables `social_user`, `social_profile`,
`post`, `social_group` and `user_group` to see how each relationship is stored.

## Things to Try

- Delete a user that has a profile and check that the profile row is gone too
  (`CascadeType.REMOVE`).
- Delete a user that has a post. Posts are not cascaded on remove, so the
  database should reject it because the post still points to the user. Add
  `CascadeType.REMOVE` to `posts` and try again.
- Remove `@JsonIgnore` from `Post.socialUser` and see what the GET request does.

## What I Learned

- Mapping one-to-one, one-to-many and many-to-many relationships with JPA
- Owning side vs `mappedBy`, and where the foreign key is stored
- Using join tables for many-to-many relationships
- Controlling saves and deletes with `CascadeType`
- Avoiding infinite loops in JSON with `@JsonIgnore`
- Loading sample data at startup with `CommandLineRunner`
