Derived Queries (Method Naming Convention)
Spring Data JPA can automatically generate queries by parsing method names in the Repository interface.

📌 Examples:
      
      // Simple equality
      List<StoreEntity> findByCity(String city);
      
      // Multiple conditions
      List<StoreEntity> findByCityAndState(String city, String state);
      
      // Comparison
      List<StoreEntity> findByZipCodeGreaterThan(int zip);
      
      // Like / Contains
      List<StoreEntity> findByNameContaining(String name);

📌 These translate into SQL automatically without writing queries.


Custom Queries using @Query

Use when:

You need custom logic

Or want more control than derived queries


A. JPQL Query (uses Entity/field names)

    @Query("SELECT s FROM StoreEntity s WHERE s.city = :city")
    List<StoreEntity> findStoresByCity(@Param("city") String city);
✅ JPQL is object-oriented (not table/column names), works with Entity fields.

Native SQL Query

          
        @Query(value = "SELECT * FROM store WHERE state = :state", nativeQuery = true)
        List<StoreEntity> findStoresByState(@Param("state") String state);
✅ Use nativeQuery = true to write actual SQL using DB column/table names.



Named vs Positional Parameters



Named Parameters (Best Practice ✅)

      
      @Query("SELECT s FROM StoreEntity s WHERE s.state = :state")
      List<StoreEntity> getByState(@Param("state") String state);
Use :state inside query

Use @Param("state") to bind

B. Positional Parameters (Old style ❌ not recommended)

      
      @Query("SELECT s FROM StoreEntity s WHERE s.city = ?1 AND s.state = ?2")
      List<StoreEntity> getByCityAndState(String city, String state);
?1, ?2 based on method arg position




4. DTO Projections (Return custom data, not entire entity)
Use when:

You don’t need all entity fields

You want a lightweight response

🔹 A. Interface-based Projection (Super clean ✅)
📁 Create Interface


      public interface StoreView {
          String getName();
          String getCity();
      }
📁 Repository

      @Query("SELECT s.name AS name, s.city AS city FROM StoreEntity s WHERE s.state = :state")
      List<StoreView> findByStateReturnPartial(@Param("state") String state);


✅ Spring will auto-populate the interface using column names.

🔹 B. Class-based Projection (DTO)
📁 Create DTO

        @AllArgsConstructor
        @Data
        public class StoreSummaryDto {
            private String name;
            private int zipCode;
        }
📁 Repository


        @Query("SELECT new com.myprojects.dto.StoreSummaryDto(s.name, s.zipCode) FROM StoreEntity s WHERE s.city = :city")
        List<StoreSummaryDto> findSummaryByCity(@Param("city") String city);
🔸 new ...() must match constructor in DTO exactly.

✅ Summary Table
| Feature              | Use When                 | Syntax                                      |
| -------------------- | ------------------------ | ------------------------------------------- |
| `findBy...`          | Simple filters           | Method name                                 |
| `@Query` JPQL        | Custom logic with Entity | `@Query("FROM Entity WHERE...")`            |
| `@Query` Native      | Raw SQL                  | `@Query(value = "...", nativeQuery = true)` |
| Named Param          | Preferred                | `:param` + `@Param`                         |
| Positional Param     | Avoid                    | `?1`, `?2`                                  |
| Interface Projection | Lightweight, fast        | `interface getX()`                          |
| Class Projection     | Structured DTO           | `SELECT new Dto(x,y)`                       |


✅ Real-Life Example Use Case
Scenario: You want to show only store name and city on homepage (not full entity).

🧠 You should:
Create a projection interface StoreView

Use a query with SELECT s.name, s.city

Return List<StoreView> — efficient & clean
