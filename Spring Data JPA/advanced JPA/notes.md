You're building an enterprise-level app (e.g., inventory system). You want:

Safe concurrent access ✅

Audit trails (who did what & when) ✅

No hard deletes (for data retention) ✅

Performance at scale (tens of thousands of records) ✅

Complex search filters (dynamic queries) ✅

Regular JPA is not enough here. So we step into Advanced JPA Features.

🔐 1. Optimistic vs Pessimistic Locking
❓ Why?
When multiple users try to update the same record at the same time → data conflict happens.

🎯 Real Case:
In a banking app, 2 admins try to update the same user balance.

✅ Optimistic Locking
📌 Assumption: Conflict is rare

      
      @Entity
      public class Account {
          @Id
          private Long id;
      
          private Double balance;
      
          @Version  // <-- This version is checked before update
          private int version;
      }
What happens under the hood:

Admin A loads the account → version = 1

Admin B loads same account → version = 1

Admin A updates → version becomes 2

Admin B tries to update → JPA sees version mismatch (1 ≠ 2) → throws OptimisticLockException

This saves you from lost updates.


Pessimistic Locking
📌 Assumption: Conflict is likely (e.g., ticket booking system)


      
      @Lock(LockModeType.PESSIMISTIC_WRITE)
      @Query("SELECT a FROM Account a WHERE a.id = :id")
      Account getAccountWithLock(@Param("id") Long id);
This locks the DB row so nobody else can read/write until you're done.




 Soft Delete (Logical Delete)
❓ Problem
You can’t delete rows from DB due to:

Legal reasons

Auditing requirements

✅ Solution: Add a deleted column
java
Copy
Edit
@Column(name = "is_deleted")
private boolean deleted;
Then use:

java
Copy
Edit
@Where(clause = "is_deleted = false")  // Hibernate will auto-filter it
Now:

findAll() won’t return deleted records

But data is still there in DB

🕵️ 3. Auditing
❓ Why?
We need to track:

Who created the record

When it was last updated

✅ Real-Time Steps:
1. Add fields in entity:
java
Copy
Edit
@CreatedDate
private LocalDateTime createdAt;

@LastModifiedDate
private LocalDateTime updatedAt;
2. Enable entity listeners:
java
Copy
Edit
@EntityListeners(AuditingEntityListener.class)
3. Enable auditing globally:
java
Copy
Edit
@SpringBootApplication
@EnableJpaAuditing
public class YourApp { ... }
4. Optionally track user (AuditorAware):
java
Copy
Edit
public class AuditorAwareImpl implements AuditorAware<String> {
    public Optional<String> getCurrentAuditor() {
        return Optional.of("systemUser");
    }
}
Now auditing is fully automated.

🔁 4. Lifecycle Callbacks
Use this to inject behavior during entity events.

When	Annotation	Use case
Before save	@PrePersist	Set timestamps, validations
After delete	@PostRemove	Log event, cleanup files

java
Copy
Edit
@PrePersist
public void preSave() {
    this.createdAt = LocalDateTime.now();
}
This code runs automatically before INSERT.

🧩 5. Custom Repository Implementation
❓ Problem:
You need complex filtering or native SQL that JPA can't generate.

✅ Steps (Live Style):
Interface:

java
Copy
Edit
public interface CustomStoreRepo {
    List<StoreEntity> customSearch();
}
Impl class:

java
Copy
Edit
public class CustomStoreRepoImpl implements CustomStoreRepo {
    @PersistenceContext
    private EntityManager em;

    public List<StoreEntity> customSearch() {
        String jpql = "SELECT s FROM StoreEntity s WHERE s.zipCode > 50000";
        return em.createQuery(jpql, StoreEntity.class).getResultList();
    }
}
Add to main repo:

java
Copy
Edit
public interface StoreRepository extends JpaRepository<StoreEntity, Integer>, CustomStoreRepo {}
✅ Now you can use both: findAll() and customSearch()

⚡ Performance Tuning (Super Important)
1. N+1 Problem
java
Copy
Edit
List<Store> list = repo.findAll();  // Calls SELECT for each child (slow)
✅ Fix:

java
Copy
Edit
@EntityGraph(attributePaths = "products")
List<Store> findAllWithProducts();
Or use:

java
Copy
Edit
@Query("SELECT s FROM Store s JOIN FETCH s.products")
2. Batch Inserts / Updates
Enable batching in properties:

properties
Copy
Edit
spring.jpa.properties.hibernate.jdbc.batch_size=30
3. Paging
java
Copy
Edit
Page<Store> page = repo.findAll(PageRequest.of(0, 10));
4. Streaming Large Data
java
Copy
Edit
@Query("SELECT s FROM StoreEntity s")
Stream<StoreEntity> streamAll();
5. Criteria API
Build dynamic queries programmatically.

java
Copy
Edit
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<StoreEntity> query = cb.createQuery(StoreEntity.class);
Root<StoreEntity> root = query.from(StoreEntity.class);

Predicate zipPredicate = cb.gt(root.get("zipCode"), 50000);
query.select(root).where(zipPredicate);

List<StoreEntity> result = em.createQuery(query).getResultList();
✅ Summary Table
Feature	Annotation / Tool	Use Case
Locking	@Version, @Lock	Prevent concurrent updates
Soft Delete	@Where, is_deleted	Don't lose data
Auditing	@CreatedDate, @AuditorAware	Auto track changes
Callbacks	@PrePersist, etc	Pre/post DB logic
Custom Repo	Interface + Impl	Custom SQL or logic
Performance	@EntityGraph, Paging	Fix N+1, big data, optimize perf

