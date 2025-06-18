Caching with @Cacheable

What is it?
Caching stores frequently used data (e.g., product list) in memory to avoid hitting the DB again and again.

🔹 Why use it?
Speeds up performance

Reduces database load

Improves user experience

🔹 Example Code:


// Step 1: Enable caching in your main class
   

      @SpringBootApplication
      @EnableCaching // 🔥 Turns on Spring Cache
      public class MyApp {
          public static void main(String[] args) {
              SpringApplication.run(MyApp.class, args);
          }
        }

// Step 2: Service with @Cacheable
     
      @Service
      public class ProductService {
      
          @Autowired
          private ProductRepository productRepo;
      
          @Cacheable("products") // 🔥 This caches the result based on the method name + args
          public List<Product> getAllProducts() {
              System.out.println("Fetching from DB..."); // ✅ Won't print if cache is hit
              return productRepo.findAll();
          }
      }

// Step 3: Add simple controller
      
      
      @RestController
      public class ProductController {
          @Autowired
          private ProductService productService;
      
          @GetMapping("/products")
          public List<Product> fetch() {
              return productService.getAllProducts();
          }
      }


Content Negotiation (produces, consumes)
🔹 What is it?
The client can request different formats like JSON or XML using Accept or Content-Type headers.

🔹 Why use it?
Supports multiple frontends or 3rd party systems needing different formats.

🔹 Example:

      @RestController
      @RequestMapping("/api")
      public class ProductController {
      
          // 🔥 This sends JSON
          @GetMapping(value = "/products", produces = "application/json")
          public List<Product> getProductsJson() {
              return List.of(new Product(1, "Laptop", 1200));
          }
      
          // 🔥 This sends XML
          @GetMapping(value = "/products-xml", produces = "application/xml")
          public List<Product> getProductsXml() {
              return List.of(new Product(1, "Laptop", 1200));
          }
      }
      
      // For XML to work, add dependency in pom.xml
      <dependency>
          <groupId>com.fasterxml.jackson.dataformat</groupId>
          <artifactId>jackson-dataformat-xml</artifactId>
      </dependency>
In Postman, use:

Header: Accept: application/json or Accept: application/xml



Swagger / OpenAPI (Documentation)
🔹 What is it?
Swagger auto-generates interactive documentation for your REST APIs.

🔹 Why use it?
Share API contract with frontend / 3rd party

Test endpoints from browser

Auto-docs and endpoint visibility

🔹 How to add it?

      <!-- Add Swagger dependency -->
      <dependency>
          <groupId>org.springdoc</groupId>
          <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
          <version>2.1.0</version>
      </dependency>


Once added:
Start your app

Go to 👉 http://localhost:8080/swagger-ui.html ✅

You'll see a complete UI where you can test every API with input/output shown.


| Feature             | What it does                    | Why it's useful                     |
| ------------------- | ------------------------------- | ----------------------------------- |
| `@Cacheable`        | Saves result in memory          | Improves speed, reduces DB calls    |
| Content Negotiation | Switch between JSON/XML formats | Multi-client support                |
| Swagger             | API Docs + Testing UI           | Easy integration with frontend/devs |


