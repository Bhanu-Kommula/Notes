CORS is a security mechanism that browsers enforce to restrict web pages from making requests to a different domain (or port) than the one that served the web page.

For example:

    Frontend (React): http://localhost:3000
    
    Backend (Spring Boot): http://localhost:8080
    
    This will trigger a CORS policy error unless allowed explicitly.

So,
React runs on port 3000, and Spring Boot (Tomcat) runs on port 8080.
Since these are different origins, the browser blocks such cross-origin HTTP calls by default due to CORS policy.

To allow the frontend to access backend resources, we need to explicitly enable CORS in Spring Boot using the @CrossOrigin annotation or a global CORS config.


Without CORS – Error

    @RestController
    @RequestMapping("/api")
    public class ProductController {
    
        @GetMapping("/products")
        public List<String> getProducts() {
            return List.of("TV", "Laptop", "Mobile");
        }
    }
When you call http://localhost:8080/api/products from http://localhost:3000, browser blocks the request with:

      
      Access to fetch at 'http://localhost:8080/api/products'
      from origin 'http://localhost:3000' has been blocked by CORS policy


  Enable CORS using @CrossOrigin

      @RestController
    @RequestMapping("/api")
    @CrossOrigin(origins = "http://localhost:3000") // 👈 Allow frontend
    public class ProductController {
    
        @GetMapping("/products")
        public List<String> getProducts() {
            return List.of("TV", "Laptop", "Mobile");
        }
    }

🟢 Now, your frontend at http://localhost:3000 can call this API without any issue.
This means:

      Allow all HTTP methods (GET, POST, PUT, DELETE, etc.)
      
      Only from http://localhost:3000
      
      No restriction on headers

But what if you want to allow only GET and POST?

🧠 Optional: Allow Multiple Origins or Methods


    @CrossOrigin(
        origins = {"http://localhost:3000", "http://example.com"},
        methods = {RequestMethod.GET, RequestMethod.POST},
        allowedHeaders = {"Content-Type", "Authorization"}
    )


This means:

Only GET and POST requests are allowed

Only from localhost:3000 or example.com

Only if the request contains Content-Type or Authorization headers

👉 Any other HTTP method or unknown header → ❌ blocked by CORS



🌐 3. Global CORS Config (if you don’t want to annotate each controller)

    
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")  // Allow all endpoints
                    .allowedOrigins("http://localhost:3000") // Allow frontend
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*");
        }
    }

This will automatically allow all controllers to accept requests from the frontend.


| Feature        | Method                     | Use When?                |
| -------------- | -------------------------- | ------------------------ |
| `@CrossOrigin` | On controller or method    | Quick fix per controller |
| WebConfig      | Global `addCorsMappings()` | Project-wide config      |
