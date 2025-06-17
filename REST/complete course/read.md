 REST (Representational State Transfer) is an architectural style for designing networked applications using stateless communication over HTTP.

  6 Core REST Constraints (Very Important)
Constraint	                Meaning
🔹 Client-Server	              Clear separation of client and backend
🔹 Stateless	                No session stored on server
🔹 Cacheable	                    Responses should declare if cacheable
🔹 Uniform Interface	              Consistent URL + HTTP methods
🔹 Layered System                  	Multiple intermediaries allowed
🔹 Code on Demand (Optional)	    Client can download code/scripts from server

What is a Resource?
Anything exposed over HTTP (user, order, product).

Represented using a URI like /api/users/1.


HTTP                   Methods & Their Meaning
HTTP                   Method	Action	Use For
GET	                  Read	Fetch data
POST	                  Create	Create new resource
PUT	                      Replace	Update existing resource (full)
PATCH	                      Modify	Partial update
DELETE	                Remove	Delete resource

      
      Example:
      GET /users/1 → Get user with ID 1
      POST /users → Create new user
      DELETE /users/1 → Delete user with ID 1

 HTTP Status Codes
 
        | Code | Category     | Meaning                |
        | ---- | ------------ | ---------------------- |
        | 200  | OK           | Success                |
        | 201  | Created      | Resource Created       |
        | 204  | No Content   | Success with no data   |
        | 400  | Bad Request  | Invalid input          |
        | 401  | Unauthorized | No valid credentials   |
        | 403  | Forbidden    | Access denied          |
        | 404  | Not Found    | Resource doesn't exist |
        | 500  | Server Error | Backend crash or bug   |


 REST vs SOAP

    | Feature        | REST             | SOAP                          |
    | -------------- | ---------------- | ----------------------------- |
    | Format         | JSON/XML         | XML only                      |
    | Lightweight?   | Yes              | No                            |
    | Learning curve | Low              | High                          |
    | Used in        | Web, Mobile APIs | Enterprise-only (banks, etc.) |




What is Validation in REST APIs?

    Validation in REST APIs is all about making sure the incoming request data (usually JSON) follows the rules or constraints you define.
    
    So yes — it's not about checking business logic like correct password, but rather checking that the structure and content of incoming data is valid before processing it.


| Field     | Validation Rule          | Annotation       |
| --------- | ------------------------ | ---------------- |
| `name`    | Cannot be blank          | `@NotBlank`      |
| `age`     | Must be at least 18      | `@Min(18)`       |
| `email`   | Must be in proper format | `@Email`         |
| `country` | Cannot be null           | `@NotNull`       |
| `city`    | Must have min 3 chars    | `@Size(min = 3)` |


You typically use validation annotations in a DTO class, like this:

        public class StudentRequestDTO {

    @NotBlank(message = "Name is required")
    private String name;

    @Min(value = 18, message = "Age must be at least 18")
    private int age;

    @NotBlank(message = "City is required")
    private String city;

    @NotBlank(message = "Country is required")
    private String country;
    }

Then in your controller:


      @PostMapping
      public ResponseEntity<String> createStudent(@Valid @RequestBody StudentRequestDTO studentDTO) {
          // If validation fails, it won’t even come here.
          return ResponseEntity.ok("Student created!");
      }


What is Exception Handling in REST?

In Spring Boot REST, exception handling means:

Catching runtime errors (like invalid input, DB failures, missing data) and converting them into clear, structured, and meaningful HTTP responses.

Instead of crashing the app or showing ugly error stack traces to clients, we send clean responses like:
    
    {
      "error": "Student Not Found",
      "status": 404,
      "timestamp": "2025-06-17T10:15:00"
    }

✅ 2. Why Is It Important?


| 🔍 Problem                      | 🛠️ Exception Handling Helps                 |
| ------------------------------- | -------------------------------------------- |
| App crashes on null or DB error | Gracefully catch and respond                 |
| Clients see stacktrace          | Show clean error message                     |
| No clue what went wrong         | Give helpful error details                   |
| All errors return 500           | Return correct status codes (400, 404, etc.) |


✅ 3. Common Exceptions in REST APIs

| Exception                         | When it occurs                             |
| --------------------------------- | ------------------------------------------ |
| `MethodArgumentNotValidException` | Validation errors                          |
| `HttpMessageNotReadableException` | Bad JSON (invalid format)                  |
| `EntityNotFoundException`         | No record found in DB                      |
| `DataIntegrityViolationException` | DB constraint broken (e.g., duplicate key) |
| `NullPointerException`            | Something was `null` unexpectedly          |
| `CustomBusinessException`         | You define your own exception              |


✅ 4. Exception Handling Approaches in Spring Boot

There are 3 main levels of exception handling:


| Level                                         | What it does                            |
| --------------------------------------------- | --------------------------------------- |
| **Local (try-catch)**                         | Handle specific logic inside a method   |
| **Controller Level (`@ExceptionHandler`)**    | Handle exceptions in a controller       |
| **Global Level (`@ControllerAdvice`)** ✅ Best | Handle all REST exceptions in one place |


We use Global Exception Handling for real-time apps.


✅ 5. Global Exception Handling (Real-Time Style)

Step 1: Create Custom Exception

    public class StudentNotFoundException extends RuntimeException {
        public StudentNotFoundException(String message) {
            super(message);
        }
    }


Step 2: Create Global Exception Handler

      
      @RestControllerAdvice
      public class GlobalExceptionHandler {

    // Handle Custom Exception
    @ExceptionHandler(StudentNotFoundException.class)
    public ResponseEntity<?> handleStudentNotFound(StudentNotFoundException ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("error", ex.getMessage());
        error.put("status", HttpStatus.NOT_FOUND.value());
        error.put("timestamp", LocalDateTime.now());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    // Handle Validation Errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> fieldErrors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            fieldErrors.put(error.getField(), error.getDefaultMessage()));
        return new ResponseEntity<>(fieldErrors, HttpStatus.BAD_REQUEST);
    }

    // Handle All Other Exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handleAll(Exception ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("error", ex.getMessage());
        error.put("status", HttpStatus.INTERNAL_SERVER_ERROR.value());
        error.put("timestamp", LocalDateTime.now());
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
      }

Step 3: Throw Exception in Service


      @Override
    public StudentEntity getStudentById(Long id) {
        return studentRepo.findById(id)
            .orElseThrow(() -> new StudentNotFoundException("Student ID " + id + " not found"));
    }
7. Best Practices for Exception Handling
✅ Do	❌ Avoid
Create custom exceptions for business rules	Swallowing exceptions silently
Use @ControllerAdvice to centralize error logic	Writing duplicate try-catch everywhere
Use correct status codes (400, 404, 409, 500)	Always returning 500
Log exceptions for debugging	Exposing stacktrace in response






🔹 1. Custom Response DTOs


Right now, you’re returning entire entity objects from your controller like:
  
    List<StudentEntity> getAllStudents()

That’s bad because:

Exposes DB structure (tight coupling)

Can leak sensitive fields (like password, status)

Hard to change later (e.g., if you rename a column)



✅ Solution: Use DTOs (Data Transfer Objects)

👉 You create a new class to represent only what the API should return.


✅ a) Create a DTO Class

      
      
      // StudentResponseDTO.java
      public class StudentResponseDTO {
          private Long id;
          private String name;
          private String city;
      
          // Constructor
          public StudentResponseDTO(Long id, String name, String city) {
              this.id = id;
              this.name = name;
              this.city = city;
          }
      
          // Getters and Setters
      }

✅ b) Modify Service Layer


    @Override
    public List<StudentResponseDTO> getAllStudents() {
        List<StudentEntity> students = studentRepo.findAll();
    
        return students.stream()
            .map(s -> new StudentResponseDTO(s.getId(), s.getName(), s.getCity()))
            .collect(Collectors.toList());
    }
    
    
✅ c) Controller Remains Clean


      @GetMapping
      public List<StudentResponseDTO> getStudents() {
          return studentService.getAllStudents();
      }


✅ 2. Pagination & Filtering


Problem:
Your API returns all records, which:

Is slow if DB has 1M+ rows

Not efficient for mobile apps

No way to search or slice


 Use Pagination + Filtering with Spring Data JPA


✅ a) Update Repository to accept Pageable


    package com.myprojects.studentportal.repository;
    
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.jpa.repository.JpaRepository;
    
    import com.myprojects.studentportal.entity.StudentEntity;
    
    public interface StudentRepository extends JpaRepository<StudentEntity, Long> {
    
        // 🔍 Custom method: filter by city (case-insensitive) with pagination
        Page<StudentEntity> findByCityContainingIgnoreCase(String city, Pageable pageable);
    }

✅ Response DTO


        xpackage com.myprojects.studentportal.dto;

    public class StudentResponseDTO {
    
        private Long id;
        private String name;
        private String city;
    
        public StudentResponseDTO(Long id, String name, String city) {
            this.id = id;
            this.name = name;
            this.city = city;
        }
    
        // ✅ Add Getters (you can use Lombok if preferred)
        public Long getId() { return id; }
        public String getName() { return name; }
        public String getCity() { return city; }
    }


✅ Service Layer

      
      package com.myprojects.studentportal.service;
      
      import com.myprojects.studentportal.dto.StudentResponseDTO;
      import com.myprojects.studentportal.entity.StudentEntity;
      import com.myprojects.studentportal.repository.StudentRepository;
      
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.data.domain.*;
      import org.springframework.stereotype.Service;
      
      @Service
      public class StudentService {
      
          @Autowired
          private StudentRepository studentRepo;
      
          // 🧠 Method to filter by city with pagination
          public Page<StudentResponseDTO> getStudentsByCity(String city, int page, int size) {
      
              // ⏳ Create pagination config (page number, size)
              Pageable pageable = PageRequest.of(page, size);
      
              // 🔍 Query the database: find students by city with pagination
              Page<StudentEntity> pageResult = studentRepo.findByCityContainingIgnoreCase(city, pageable);
      
              // 🔁 Convert each StudentEntity into StudentResponseDTO
              return pageResult.map(s -> new StudentResponseDTO(s.getId(), s.getName(), s.getCity()));
          }
      }
      
      
      

✅ Controller Layer

    
    package com.myprojects.studentportal.controller;
    
    import com.myprojects.studentportal.dto.StudentResponseDTO;
    import com.myprojects.studentportal.service.StudentService;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.domain.Page;
    import org.springframework.web.bind.annotation.*;
    
    @RestController
    @RequestMapping("/api/students")
    public class StudentController {
    
        @Autowired
        private StudentService studentService;
    
        // 📦 API: GET /api/students/filter?city=hyd&page=0&size=5
        @GetMapping("/filter")
        public Page<StudentResponseDTO> filterStudentsByCity(
            @RequestParam String city,                      // ✅ filter param
            @RequestParam(defaultValue = "0") int page,     // ✅ pagination: page number (0-based)
            @RequestParam(defaultValue = "5") int size      // ✅ pagination: size per page
        ) {
            // 🎯 Delegate to service layer which returns paginated result
            return studentService.getStudentsByCity(city, page, size);
        }
    }



