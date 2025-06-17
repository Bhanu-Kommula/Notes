Builder is useful when:

You want cleaner code for object creation

You want to avoid large constructors with too many parameters

You want to set only required fields and skip the rest



🧱 Step 1: Modify StoreEntity to Support Builder

      
      
      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      @Builder // 🔥 Add this Lombok annotation
      @Table(name = "store")
      public class StoreEntity {
          @Id
          private int id;
          private String name;
          private String city;
          private String state;
          private String email;
          private int zipCode;
      }


🧱 Step 2: Modify StoreDto to Support Builder


      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      @Builder // 🔥 Add this too
      public class StoreDto {
          private int id;
          private String name;
          private String city;
          private String state;
          private String email;
          private int zipCode;
      }


 Step 3: Use Builder in StoreService.java
🔁 Saving Data — DTO ➝ Entity Using Builder

    
    
    
    public String saveStore(StoreDto dto) {
        StoreEntity entity = StoreEntity.builder()
                .id(dto.getId())
                .name(dto.getName())
                .city(dto.getCity())
                .state(dto.getState())
                .email(dto.getEmail())
                .zipCode(dto.getZipCode())
                .build(); // 🔨 Builder creates the object
    
        storeRepository.save(entity);
        return "Saved successfully";
    }


Summary Notes on Builder Pattern
Annotate class with @Builder (Lombok)

Use .builder().field().field().build() syntax

Makes code more readable, maintainable, and flexible

Recommended when working with DTOs and Entities in real-time projects

