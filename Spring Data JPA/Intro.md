What is JPA and Spring Data JPA?

JPA:
  Full form: Java Persistence API
  It is just a set of rules/interfaces that help Java code talk to relational databases like Oracle/MySQL.
  But JPA is only a specification, it does not do the actual work.

Hibernate:
  Hibernate is the real worker (implementation) that follows JPA’s rules.
  It converts your Java objects into SQL queries and talks to the DB.

Spring Data JPA:
  This is a Spring Boot module that wraps Hibernate and JPA.
  It makes your life super easy:
  No need to write SQL for basic tasks
  Just write an interface → Spring will generate SQL internally.


    public interface StudentRepository extends JpaRepository<Student, Long> {
    List<Student> findByName(String name); // Spring will auto-create query!
    }

Configuration (application.properties)
To make JPA work, we must configure Spring Boot to connect to the database and tell it how to behave.

application.properties

      # Database connection (Oracle example)
      spring.datasource.url=jdbc:oracle:thin:@localhost:1521:xe
      spring.datasource.username=system
      spring.datasource.password=oracle
      spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
      
      # JPA behavior
      spring.jpa.hibernate.ddl-auto=update   
      spring.jpa.show-sql=true
      spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle10gDialect

What Each Line Means:

  Property	                                        Meaning
ddl-auto=update	                          If entity class changes, update table structure (safe for dev)
show-sql=true	                            Print SQL queries in console
hibernate.dialect=Oracle10gDialect	      Tell Hibernate to generate Oracle-style SQL


 If you don’t write ddl-auto, Spring won’t create any tables. It expects you to create tables manually. and if table exist already then you can work on that but spring automatically cant create in absence of table.

Entity Class — Mapping Java to DB Table
     This is where your normal Java class becomes a DB table.


StudentInfoModel.java

      import jakarta.persistence.*;

    @Entity
    public class StudentInfoModel {
    
        @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) //GenerationType.IDENTITY does not work with Oracle properly out of the box.
    It works well with MySQL, PostgreSQL, etc., but not Oracle.
    private int id;

    private String name;
    private int age;
    private String city;
    private String country;
    private int maths;
    private int physics;
    private int chemistry;

    // Getters & Setters, Constructors (can be auto-generated)
    }
or other exm

    import jakarta.persistence.*;
    
    @Entity
    @Table(name = "employee")  // matches DB table name
    public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emp_seq_gen")
    @SequenceGenerator(name = "emp_seq_gen", sequenceName = "emp_seq", allocationSize = 1)
    @Column(name = "emp_id")
    private int id;

    @Column(name = "emp_name")
    private String name;

    @Column(name = "emp_sal")
    private double salary;

    // constructors, getters, setters...
    }


Auto Increment with oracle 
    
    @Entity
    public class StudentInfoModel {
    
        @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "student_seq")
    @SequenceGenerator(name = "student_seq", sequenceName = "student_sequence", allocationSize = 1)
    private int id;

    private String name;
    private int age;
    // ... other fields
    }



Real-time example Oracle with auto increment

DB      
     
      CREATE TABLE employee (
          emp_id NUMBER PRIMARY KEY,
          emp_name VARCHAR2(100),
          emp_sal NUMBER
      );
      

Also, you created a sequence for auto-generating IDs:

    CREATE SEQUENCE emp_seq START WITH 1 INCREMENT BY 1;
 full Entity Class in Java (Oracle-Compatible):
   
    import jakarta.persistence.*;
    
    @Entity
    @Table(name = "employee")  // matches your DB table name
    public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emp_seq_gen")
    @SequenceGenerator(name = "emp_seq_gen", sequenceName = "emp_seq", allocationSize = 1)
    @Column(name = "emp_id")
    private int id;

    @Column(name = "emp_name")
    private String name;

    @Column(name = "emp_sal")
    private double salary;

    // 👉 Constructors

    public Employee() {}

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    // 👉 Getters & Setters

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
    }

Explanation of that line:


      @SequenceGenerator(
          name = "emp_seq_gen",              // Just a Java-side name
          sequenceName = "emp_seq",          // This MUST MATCH your Oracle DB sequence name
          allocationSize = 1
      )


What Annotations Do:
Annotation	                  Purpose
@Entity	              Tells Spring to treat this class as a table
@Id	                      Primary key
@GeneratedValue	          Auto-increment the ID
@Column(name="...")	          Optional — use to set custom column name



Repository Layer 

A Repository is like a Java interface that connects your app to the database — without writing SQL.
It extends built-in interfaces like JpaRepository, which gives you ready-made DB methods.


      @Repository
      public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
          // auto-generated methods are available
      }

Built-in Methods You Get for Free:
Method	                  Description
save(emp)	            Insert or update record
  findById(id)        	Fetch by ID
findAll()	            Get all records
deleteById(id)	        Delete by ID
count()	              Total count
existsById(id)	      Check if record exists


Most Common Naming Patterns:
Method	                                          Description
findByName(String name)	                              Exact match
findBySalaryGreaterThan(double sal)	                  Greater than >
findBySalaryLessThan(double sal)	                       Less than <
findBySalaryBetween(double min, double max)	            Range check
findByNameAndSalary(String name, double sal)	          Multiple conditions
findByNameOrSalary(String name, double sal)	            OR condition
findByNameContaining(String keyword)	                  LIKE '%keyword%'
findByNameStartsWith(String prefix)	                  LIKE 'prefix%'
findByNameEndsWith(String suffix)	            L IKE '%suffix'
findBySalaryOrderByNameAsc()	                Sort by name ascending
findBySalaryOrderByNameDesc()	                Sort by name descending
findByNameIgnoreCase(String name)	                Case-insensitive match



      List<Employee> findBySalaryBetweenAndNameContaining(double min, double max, String keyword);
This will generate:

      SELECT * FROM employee 
      WHERE salary BETWEEN ? AND ? 
      AND name LIKE %keyword%;


What does .get() do in Optional?
Let's say we write:


      Optional<Employee> emp = employeeRepository.findById(5);
      Employee e = emp.get(); // 

What is Optional.get()?
.get() is used to extract the actual object inside the Optional.

Think of Optional like a wrapper box around your object.
.get() opens the box and gives you the value.



BUT — Important Warning:
If the Optional is empty (i.e., no employee with that ID), and you call .get(), it will throw:

      NoSuchElementException: No value present
That’s why we usually do this instead:

    
    Employee e = emp.orElse(null); // safer
Or:


    if(emp.isPresent()) {
       Employee e = emp.get();
    }


What is @Transactional?
It tells Spring:
“Run everything inside this method as one transaction. If anything fails, roll everything back.”

💡 Real-Life Analogy:
You send money to a friend (₹500)

Bank deducts money from your account

Bank credits friend’s account

If step 2 fails → step 3 should not happen

Same logic in DB.

      
      @Transactional
      public void hireEmployee(Employee emp) {
          employeeRepository.save(emp);   // insert employee
          auditRepository.save(null);     // crash here ❌
      
          // Spring will rollback employee save automatically
      }

Key Points:
Rule	                                    Explanation
@Transactional                         on method or class	Makes all DB actions atomic
Rollback happens on RuntimeExceptions	          Like NullPointerException
Use rollbackFor = Exception.class                     for checked exceptions	Optional
Best practice:                               use in Service Layer	Not Controller layer


      @Transactional
      public void doWork() {
         // all DB operations are committed together
      }
If any operation throws an error — the entire method is rolled back.

Using @Query (for full control)
JPQL Query (uses class & field names)

      @Query("SELECT e FROM Employee e WHERE e.name = :name")
      List<Employee> getByName(@Param("name") String name);

Employee is your class
e.name is the Java field


2. Native SQL Query (uses actual DB table & column names)


        @Query(value = "SELECT * FROM employee WHERE emp_name = :name", nativeQuery = true)
        List<Employee> getByNameNative(@Param("name") String name);
        
Here, you're writing full Oracle SQL
Good when using joins or complex Oracle-specific syntax


Summary of Entity + Repository Layer:
Layer                    	            Role
Entity (@Entity)	              Maps Java class to DB table
@Id, @GeneratedValue, @Column	        Maps fields to columns
Repository (JpaRepository)	            Gives built-in DB operations
Custom methods                  	Created using method naming or @Query
Native SQL	                Use nativeQuery = true for Oracle syntax


REST CONTROLLER LAYER — Exposing APIs

 What Is the Controller Layer?

    Up to now, you wrote:
    Entity to map with DB
    Repository to talk to DB
    Service to hold logic

But that’s all backend-internal.

Now it’s time to expose endpoints like:

    GET /employees
    POST /employees
    DELETE /employees/{id}

So that Postman, frontend, or mobile app can call your backend and get/send data.

That’s the job of the Controller.

Real-Life Analogy:
Layer          	Role
🧠 Service	    Thinks & calculates
📦 Repository    	Stores & fetches
🧍 Controller    	Talks to the outside world — takes requests, returns responses

How to Write a REST Controller?
Create a new class with:

      @RestController annotation (Spring knows it’s a controller)
      @RequestMapping("/employees") for base path
      Inject your EmployeeService
      Create methods with @GetMapping, @PostMapping, 

    
    
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.*;
    import jakarta.validation.Valid;
    import jakarta.validation.constraints.Min;
    import java.util.List;
    
    @RestController // This class exposes REST APIs
    @RequestMapping("/employees") // Base path for all endpoints
    @Validated // Enables validation for @RequestParam
    public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    // 1️⃣ CREATE EMPLOYEE (POST)
    @PostMapping
    public ResponseEntity<String> saveEmployee(@Valid @RequestBody Employee emp) { 
        // @Valid checks constraints like @NotNull, @Min inside Employee
        String message = employeeService.create(emp);
        return ResponseEntity.ok(message); // returns 200 OK + message
    }

    // 2️⃣ GET EMPLOYEE BY ID
    @GetMapping("/{id}")
    public ResponseEntity<Employee> getById(@PathVariable int id) {
        Employee emp = employeeService.getById(id);
        if (emp == null) {
            throw new ResourceNotFoundException("Employee not found with ID: " + id); // custom exception
        }
        return ResponseEntity.ok(emp);
    }

    // 3️⃣ GET ALL EMPLOYEES
    @GetMapping
    public ResponseEntity<List<Employee>> getAll() {
        return ResponseEntity.ok(employeeService.getAll());
    }

    // 4️⃣ DELETE EMPLOYEE
    @DeleteMapping("/{id}")
    public ResponseEntity<String> delete(@PathVariable int id) {
        return ResponseEntity.ok(employeeService.deleteById(id));
    }

    // 5️⃣ UPDATE EMPLOYEE
    @PutMapping("/{id}")
    public ResponseEntity<String> update(@PathVariable int id, @Valid @RequestBody Employee updatedEmp) {
        Employee existing = employeeService.getById(id);
        if (existing == null) {
            throw new ResourceNotFoundException("Employee not found with ID: " + id);
        }
        updatedEmp.setId(id); // keep same ID
        employeeService.create(updatedEmp); // reuse create() method
        return ResponseEntity.ok("Employee updated successfully");
    }

    // 6️⃣ GET EMPLOYEES BY NAME (USING QUERY PARAM)
    @GetMapping("/search")
    public ResponseEntity<List<Employee>> searchByName(@RequestParam String name) {
        return ResponseEntity.ok(employeeService.searchByName(name));
    }

    // 7️⃣ GET TOTAL SALARY (AGGREGATION LOGIC)
    @GetMapping("/total-salary")
    public ResponseEntity<Double> getTotalSalary() {
        return ResponseEntity.ok(employeeService.getTotalSalary());
    }

    // 8️⃣ GET TOP PAID EMPLOYEE
    @GetMapping("/top-paid")
    public ResponseEntity<Employee> getTopPaid() {
        return ResponseEntity.ok(employeeService.getTopPaid());
    }

    // 9️⃣ PAGINATED EMPLOYEES (Pageable)
    @GetMapping("/page")
    public ResponseEntity<List<Employee>> getPaginated(@RequestParam(defaultValue = "0") int page,
                                                       @RequestParam(defaultValue = "3") int size) {
        return ResponseEntity.ok(employeeService.getPaginated(page, size));
    }

    // 🔟 CHECK MINIMUM SALARY (Validate @RequestParam)
    @GetMapping("/filter-by-salary")
    public ResponseEntity<List<Employee>> filterBySalary(@RequestParam @Min(1000) double salary) {
        return ResponseEntity.ok(employeeService.findByMinSalary(salary));
    }
    }


Exception Handler (Global)
Create this to handle all exceptions nicely.

    import org.springframework.http.*;
    import org.springframework.web.bind.annotation.*;
    
    @ControllerAdvice
    public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneric(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Something went wrong: " + ex.getMessage());
    }
    }

And a custom exception:

    public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String msg) {
        super(msg);
        }
    }
Employee.java (Validation Example)

    import jakarta.persistence.*;
    import jakarta.validation.constraints.*;
    
    @Entity
    public class Employee {
    
        @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emp_seq_gen")
    @SequenceGenerator(name = "emp_seq_gen", sequenceName = "emp_seq", allocationSize = 1)
    private int id;

    @NotBlank(message = "Name must not be blank")
    private String name;

    @Min(value = 1000, message = "Salary must be at least 1000")
    private double salary;

    // getters/setters
    }


Integrating Swagger UI in Spring Boot (Live API Docs + Testing)


      What is Swagger?
      Swagger is a tool that helps you:
      
      📃 Automatically generate API documentation
      
      🎯 Interact with your REST endpoints (test GET, POST, DELETE, etc.)
      
      🚀 Share UI with frontend/mobile team to understand and try APIs
      
 Example:
Without Swagger:

You need Postman or curl to test every API manually

With Swagger:

You open a link like http://localhost:8080/swagger-ui/index.html
All your APIs appear in a beautiful UI — with input boxes, dropdowns, sample JSON, etc.

✅ Step-by-Step: Add Swagger to Spring Boot Project
🧱 Step 1: Add Maven Dependency (OpenAPI Swagger)
In your pom.xml, add:
    
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.1.0</version>
    </dependency>
This gives you Swagger UI + OpenAPI 3 support for Spring Boot 3.x or above.

Step 2: Run Your Project
When you start your Spring Boot app, Swagger will auto-detect all your REST APIs.

Now open this in browser:


      http://localhost:8080/swagger-ui/index.html


Optional Configuration (if needed)
You can customize Swagger info using application.properties:

      springdoc.api-docs.enabled=true
      springdoc.swagger-ui.path=/swagger-ui.html


And if you want to group APIs, add:

    import io.swagger.v3.oas.annotations.OpenAPIDefinition;
      import io.swagger.v3.oas.annotations.info.Info;
      import org.springframework.context.annotation.Configuration;
      
      @OpenAPIDefinition(
          info = @Info(title = "Employee API", version = "1.0", description = "API for managing employees")
      )
      @Configuration
      public class SwaggerConfig {
          // no need to write anything else — auto-detection will work
      }

 Testing APIs in Swagger
Once UI is open:

Click on your endpoint → e.g., /employees

Click “Try it out”

Fill in inputs (like employee JSON)

Click “Execute”

See response body + status + curl command

✅ Summary: Why Swagger is 🔥
Feature	Why It’s Useful
📖 Auto-generated docs	No need to write anything manually
🚀 Real-time testing	Works like Postman, but built-in
🤝 Frontend-friendly	Frontend/mobile teams can self-test
⚙️ Easy to integrate	Just 1 dependency
✅ Supports validation, enums, types	Shows everything clearly
