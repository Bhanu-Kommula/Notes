**Vallidations** 

| Annotation         | Description                                                                  |
| ------------------ | ---------------------------------------------------------------------------- |
| `@NotNull`         | Field must not be `null`                                                     |
| `@NotBlank`        | Field must not be `null` and must have at least one non-whitespace character |
| `@NotEmpty`        | Field must not be `null` or empty                                            |
| `@Size(min, max)`  | Checks string/collection length                                              |
| `@Min(value)`      | Must be a number ≥ given value                                               |
| `@Max(value)`      | Must be a number ≤ given value                                               |
| `@Positive`        | Must be a positive number (> 0)                                              |
| `@PositiveOrZero`  | Must be ≥ 0                                                                  |
| `@Negative`        | Must be a negative number (< 0)                                              |
| `@NegativeOrZero`  | Must be ≤ 0                                                                  |
| `@Email`           | Must be a valid email address                                                |
| `@Pattern(regex)`  | Must match the specified regex                                               |
| `@Past`            | Must be a past date                                                          |
| `@PastOrPresent`   | Must be past or today                                                        |
| `@Future`          | Must be a future date                                                        |
| `@FutureOrPresent` | Must be future or today                                                      |
| `@Digits(i,f)`     | Must be numeric with `i` integer and `f` fraction digits                     |
| `@DecimalMin`      | Must be ≥ specified decimal                                                  |
| `@DecimalMax`      | Must be ≤ specified decimal                                                  |
| `@AssertTrue`      | Must be `true`                                                               |
| `@AssertFalse`     | Must be `false`                                                              |



**JpaRepository methods**

| Method                          | Description                                 |
| ------------------------------- | ------------------------------------------- |
| `save(S entity)`                | Save or update an entity                    |
| `saveAll(Iterable<S> entities)` | Save or update multiple entities            |
| `findById(ID id)`               | Find by primary key                         |
| `findAll()`                     | Fetch all records                           |
| `findAllById(Iterable<ID> ids)` | Fetch multiple records by IDs               |
| `deleteById(ID id)`             | Delete by ID                                |
| `delete(Entity)`                | Delete specific entity                      |
| `deleteAll()`                   | Delete all records                          |
| `deleteAllById(Iterable<ID>)`   | Delete records by IDs                       |
| `count()`                       | Count total records                         |
| `existsById(ID id)`             | Check if record exists by ID                |
| `flush()`                       | Flush changes to DB                         |
| `getById(ID id)`                | Get reference without fetching (deprecated) |
| `getOne(ID id)`                 | Lazy fetch proxy (deprecated)               |
| `findAll(Sort sort)`            | Find all with sorting                       |
| `findAll(Pageable pageable)`    | Find all with pagination                    |
| `saveAndFlush(S entity)`        | Save and flush in one call                  |

✅ Custom Method Naming in Spring Data JPA
| Method Name                 | Meaning                            |
| --------------------------- | ---------------------------------- |
| `findById(int id)`          | SELECT \* WHERE id = ?             |
| `findByName(String name)`   | SELECT \* WHERE name = ?           |
| `deleteById(int id)`        | DELETE WHERE id = ?                |
| `deleteByName(String name)` | DELETE WHERE name = ?              |
| `existsByName(String name)` | SELECT EXISTS (check name present) |
| `countByCity(String city)`  | COUNT(\*) WHERE city = ?           |
✅ For complex logic → use @Query

        @Query("SELECT s FROM StoreEntity s WHERE s.city = :city AND s.state = :state")
    List<StoreEntity> findByCityAndState(@Param("city") String city, @Param("state") String state);



**JPA annotations related to @Id and primary key generation**
 | Annotation                                            | Description                                             |
| ----------------------------------------------------- | ------------------------------------------------------- |
| `@Id`                                                 | Marks the primary key field                             |
| `@GeneratedValue`                                     | Specifies how the primary key is generated              |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | Auto-increment (DB identity column)                     |
| `@GeneratedValue(strategy = GenerationType.SEQUENCE)` | Uses database sequence                                  |
| `@GeneratedValue(strategy = GenerationType.TABLE)`    | Uses a table to simulate sequence                       |
| `@GeneratedValue(strategy = GenerationType.AUTO)`     | Let JPA choose strategy based on DB                     |
| `@SequenceGenerator`                                  | Defines a DB sequence generator (used with `SEQUENCE`)  |
| `@TableGenerator`                                     | Defines a table-based key generator (used with `TABLE`) |
| `@Column`                                             | Maps the field to a specific DB column                  |
| `@EmbeddedId`                                         | Composite primary key with embedded class               |
| `@IdClass`                                            | Composite primary key with external class               |


**JPA annotations for table and column mapping:**

| Annotation                    | Description                        |
| ----------------------------- | ---------------------------------- |
| `@Entity`                     | Marks the class as a JPA entity    |
| `@Table(name = "table_name")` | Maps entity to a specific DB table |


| Annotation                          | Description                                      |
| ----------------------------------- | ------------------------------------------------ |
| `@Column(name = "col_name")`        | Maps field to a specific DB column               |
| `@Column(length = N)`               | Sets max length for string columns               |
| `@Column(nullable = false)`         | Makes column NOT NULL                            |
| `@Column(unique = true)`            | Adds UNIQUE constraint                           |
| `@Column(updatable = false)`        | Column cannot be updated after insert            |
| `@Column(insertable = false)`       | Column excluded from INSERT                      |
| `@Column(precision = X, scale = Y)` | For decimal values (X total digits, Y after dot) |



