# 🌱 Spring Boot with Databases (Spring Data JPA) 

Spring Boot simplifies working with databases by providing **Spring Data JPA**, which integrates **JPA** (Java Persistence API) and **Hibernate**. It handles **CRUD operations**, **queries**, **transactions**, **fetching strategies**, and **database migrations** with minimal boilerplate.

----------

## 1️⃣ JPA & Hibernate Basics

-   **JPA (Java Persistence API)**:  
    A specification for ORM (Object-Relational Mapping). It defines **how Java objects map to database tables**.
    
-   **Hibernate**:  
    A **popular JPA implementation**. Handles **automatic SQL generation**, **caching**, and **lazy/eager fetching**.
    

**Key Concepts**:

-   **Entity**: Java class mapped to a DB table.
    
-   **Persistence Context**: Keeps track of entity states (`Transient`, `Persistent`, `Detached`, `Removed`).
    
-   **Entity Manager**: Interface to manage entity lifecycle.
    

💡 **Tip**: Use **Spring Data JPA** to avoid directly working with EntityManager; it abstracts boilerplate code.


---------

## 2️⃣ Entities and Annotations

```java
import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(length = 100)
    private String email;

    // constructors, getters/setters
}
```

Example with `@SequenceGenerator` (Postgres-optimized):

-   `@Entity` → Marks a class as a JPA entity.
    
-   `@Table` → Optional, specifies DB table name.
    
-   `@Id` → Primary key.
    
-   `@GeneratedValue` → Auto-generates primary key.
    
-   `@Column` → Column-specific constraints.
    

💡 **Tip**: Always define **equals() and hashCode()** carefully for entities.



### Entities and `@Entity`, `@Id`, `@GeneratedValue`

`@GeneratedValue` strategies:

-   `AUTO` — provider chooses (default).
    
-   `IDENTITY` — DB identity column (auto-increment).
    
-   `SEQUENCE` — use DB sequence (Postgres, Oracle).
    
-   `TABLE` — emulate sequence via a table (rare).
    

Use `IDENTITY` for MySQL autoincrement; `SEQUENCE` for Postgres for better batching.

```java
@Id
@SequenceGenerator(name = "user_seq", sequenceName = "user_seq", allocationSize = 1)
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
private Long id;
```



-----------


## 3️⃣ Relationships

Design example:

-   `User` has one `Address` (`@OneToOne`)
    
-   `User` has many `Post` (`@OneToMany`)
    
-   `Post` can have many `Tag` (and tags many posts) -> `@ManyToMany`

### @OneToOne

```java
@Entity
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String address;

    @OneToOne(mappedBy = "profile", cascade = CascadeType.ALL)
    private User user;
}
```

### @OneToMany / @ManyToOne

```java
@Entity
public class User {

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders;
}

@Entity
public class Order {

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

### @ManyToMany

```java
@Entity
public class Student {

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses;
}
```


💡 **Tip**: Always choose **LAZY fetching** for collections to avoid **N+1 problem**.

### Example 2 

```java
@Entity
public class Address {
  @Id @GeneratedValue
  private Long id;
  private String street;
  private String city;
  // getters/setters
}

@Entity
public class User {
  @Id @GeneratedValue
  private Long id;
  private String name;
  private String email;

  @OneToOne(cascade = CascadeType.ALL)
  @JoinColumn(name = "address_id", referencedColumnName = "id")
  private Address address;

  @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
  private List<Post> posts = new ArrayList<>();
  // ...
}

@Entity
public class Post {
  @Id @GeneratedValue
  private Long id;
  private String title;

  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "user_id")
  private User author;

  @ManyToMany
  @JoinTable(name = "post_tags",
    joinColumns = @JoinColumn(name = "post_id"),
    inverseJoinColumns = @JoinColumn(name = "tag_id"))
  private Set<Tag> tags = new HashSet<>();
  // ...
}

@Entity
public class Tag {
  @Id @GeneratedValue
  private Long id;
  private String name;

  @ManyToMany(mappedBy = "tags")
  private Set<Post> posts = new HashSet<>();
}
```
Notes:

-   `mappedBy` indicates inverse side.
    
-   `cascade` controls propagation of operations (persist/merge/remove).
    
-   `orphanRemoval=true` removes children removed from the collection.
    
-   Prefer `Set` for `@ManyToMany` to avoid duplicates.


--------


## 4️⃣ JpaRepository & CrudRepository

Spring Data provides **repositories** to abstract DB operations.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // derived queries go here
}
```

Important interfaces:

-   `CrudRepository<T, ID>` — CRUD only.
    
-   `PagingAndSortingRepository<T, ID>` — adds paging/sorting.
    
-   `JpaRepository<T, ID>` — extends above + JPA-specific methods (flush, deleteInBatch).
    

Use `JpaRepository` most of the time.


----- 


## 5️⃣ Derived Query Methods

Spring Data JPA can automatically create queries based on **method names**.

```java
List<User> findByName(String name);
Optional<User> findByEmail(String email);
List<User> findByNameContainingIgnoreCase(String part);
List<User> findByPostsTitle(String title); // traverse relation: posts.title
```

Common keywords: `And`, `Or`, `Between`, `LessThan`, `GreaterThan`, `Like`, `Containing`, `StartsWith`, `OrderBy`.

Pitfall: very long method names become unreadable — switch to `@Query` for complex logic.

---------

## 6️⃣ Custom JPQL Queries (`@Query`)

`@Query` supports JPQL (entity-centric) and native SQL (`nativeQuery = true`).

JPQL example:

```java
public interface PostRepository extends JpaRepository<Post, Long> {

  @Query("select p from Post p join p.tags t where t.name = :tagName")
  List<Post> findByTagName(@Param("tagName") String tagName);

  @Query("select p from Post p where lower(p.title) like lower(concat('%',:term,'%'))")
  Page<Post> searchByTitle(@Param("term") String term, Pageable pageable);

  @Query(value = "select * from posts p where p.title ilike %:term%", nativeQuery = true)
  List<Post> nativeSearch(@Param("term") String term);
}
```

-   JPQL uses **entities** instead of DB tables.
    
-   Supports **parameters**, **joins**, **aggregates**.
    

💡 **Tip**: Use **nativeQuery = true** for raw SQL if necessary.


----------

## 7️⃣ Pagination and Sorting

Use `Pageable` and `Page`:

```java
Pageable pageable = PageRequest.of(0, 10, Sort.by("createdAt").descending());
Page<Post> page = postRepository.findAll(pageable);

List<Post> pageContent = page.getContent();
long totalElements = page.getTotalElements();
int totalPages = page.getTotalPages();
```

Repository example:

```java
Page<Post> findByAuthorId(Long authorId, Pageable pageable);
```

Best practices:

-   Request one page at a time from DB (avoid retrieving huge lists).
    
-   Prefer `Cursor`-style (seek) pagination for large datasets and stability.


-------------- 

## 8️⃣ Transactions (@Transactional)

`@Transactional` demarcates a transaction. Apply to service layer methods (not controllers/repositories directly typically).

Example service:

```java
@Service
public class UserService {
  private final UserRepository userRepo;
  private final PostRepository postRepo;

  @Autowired
  public UserService(UserRepository userRepo, PostRepository postRepo) {
    this.userRepo = userRepo; this.postRepo = postRepo;
  }

  @Transactional
  public User createUserWithPost(User user, Post post) {
    User saved = userRepo.save(user); // persisted but still in tx
    post.setAuthor(saved);
    postRepo.save(post);
    // if exception thrown, both saves roll back
    return saved;
  }

  @Transactional(readOnly = true)
  public List<Post> getPostsForUser(Long userId) {
    return postRepo.findByAuthorId(userId);
  }
}
```

Notes:

-   `@Transactional` on a private method or same-class call won't start a transaction (Spring proxies). Put it on public service methods.
    
-   `readOnly=true` can hint to DB/ORM for optimizations.
    
-   Set proper propagation (`REQUIRED` default) and isolation if needed.

-----------


##  Lazy vs Eager Fetching

- `EAGER`: load immediately.
- `LAZY`: load on access.

Fix LazyInitializationException: fetch joins, DTOs, `EntityGraph`.

```java
@Query("select u from User u left join fetch u.posts where u.id = :id")
Optional<User> findByIdWithPosts(@Param("id") Long id);
```

---

##  Database Initialization (`schema.sql`, `data.sql`)

```sql
-- schema.sql
CREATE TABLE users (id BIGINT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), email VARCHAR(255) UNIQUE);
-- data.sql
INSERT INTO users (name, email) VALUES ('Raja', 'raja@example.com');
```

---

## Flyway / Liquibase

Flyway example:
```sql
-- V1__init.sql
CREATE TABLE users (id BIGSERIAL PRIMARY KEY, name VARCHAR(255), email VARCHAR(255) UNIQUE);
```

---

##  Full small example: Controller-Service-Repository

`UserRepository`:
```java
public interface UserRepository extends JpaRepository<User, Long> {
  Optional<User> findByEmail(String email);
  Page<User> findByNameContainingIgnoreCase(String name, Pageable pageable);

  @EntityGraph(attributePaths = {"posts"})
  @Query("select u from User u where u.id = :id")
  Optional<User> findByIdWithPosts(@Param("id") Long id);
}

```

`UserService`:
```java
@Service
@RequiredArgsConstructor
public class UserService {
  private final UserRepository userRepo;
  private final PostRepository postRepo;

  @Transactional
  public User create(User user) { return userRepo.save(user); }

  @Transactional(readOnly=true)
  public User getUserWithPosts(Long id) {
    return userRepo.findByIdWithPosts(id)
                   .orElseThrow(() -> new EntityNotFoundException("User not found"));
  }
}
```

`UserController`:
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
  private final UserService userService;

  @GetMapping("/{id}")
  public ResponseEntity<UserDto> get(@PathVariable Long id) {
    User user = userService.getUserWithPosts(id);
    UserDto dto = UserMapper.toDto(user); // map to DTO to avoid exposing entities
    return ResponseEntity.ok(dto);
  }
}
```
Important: map entities to DTOs for API responses to avoid lazy loading outside tx and to protect internal structure.


---


## Performance & best practices (quick list)

-   Avoid returning entities directly from controllers — use DTOs.
    
-   Avoid `EAGER` everywhere — use fetch joins or `EntityGraph` to fetch what you need.
    
-   Use pagination for collection endpoints.
    
-   Beware N+1 problem — use `join fetch` or `@EntityGraph`.
    
-   Prefer JPQL or projections for big selects instead of fetching whole entity graphs.
    
-   Use migrations (Flyway/Liquibase) in CI/CD.
    
-   Set `ddl-auto` to `validate` in prod, not `update` or `create`.
    
-   Use proper indexing at DB level for frequently queried columns.
    
-   Profile queries (`spring.jpa.show-sql=true`) only in dev; use logs to find slow queries.


-----

##  Best Practices

- DTOs for API responses.
- Avoid `EAGER` everywhere.
- Pagination for collections.
- Fetch joins to avoid N+1.
- Use migrations in CI/CD.
- Profile queries in dev.

---

##  Common Pitfalls


-   **LazyInitializationException** — fix by fetching in service layer, using DTOs, or using fetch joins.
    
-   **Transaction boundaries** — `@Transactional` needs to be on public methods called from outside (proxy).
    
-   **Cascade misuse** — don’t cascade `REMOVE` accidentally and delete unrelated data.
    
-   **Bidirectional relationships** — always manage both sides of relation in code (e.g., when adding a post to user, set `post.setAuthor(user)` and `user.getPosts().add(post)`).
    
-   **Bulk updates** — use `@Modifying @Query` for bulk updates and call `entityManager.clear()` if needed; bulk JPQL/SQL bypasses the persistence context.
    

Example bulk update:

```java
@Modifying
@Query("update Post p set p.published = true where p.createdAt < :date")
int publishOldPosts(@Param("date") LocalDate date);
```


Remember to annotate service method with `@Transactional` when calling modifying queries.


---

## Quick Snippets

Fetch join to avoid lazy exception:
```java
@Query("select u from User u left join fetch u.posts where u.id = :id")
User findWithPosts(@Param("id") Long id);
```
DTO projection:
```java
public interface UserSummary {
  Long getId();
  String getName();
  String getEmail();
}
List<UserSummary> findByNameContaining(String name);
```
EntityGraph:
```java
@EntityGraph(attributePaths = {"address", "posts"})
Optional<User> findById(Long id);
```
Transactional bulk update:
```java
@Modifying
@Query("update Post p set p.published = true where p.createdAt < :date")
int publishOldPosts(@Param("date") LocalDate date);
```

---

## Small checklist to deliver production-ready DB integration

1.  Use Flyway/Liquibase migrations.
    
2.  Configure connection pool & timeouts.
    
3.  Set `hibernate.ddl-auto=validate`.
    
4.  Add indexes and constraints in migrations.
    
5.  Use DTOs and map in services.
    
6.  Add pagination on API endpoints returning lists.
    
7.  Monitor slow queries and set proper logging.
    
8.  Add integration tests with Testcontainers (Postgres/MySQL) instead of H2 for parity.
  


-------
