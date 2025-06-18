**Pagination & Sorting (Pageable, Page, Sort)**

Why?
 Real-Time Use Case:
Project: Online Store
Problem: Display 1000+ products in frontend — but show 10 at a time with sorting options like price, name, rating.

🧠 Concept:
Spring provides built-in support via:

Pageable: to receive pagination info (page, size, sort)

Page<T>: to return metadata like total pages, size, etc.

Sort: for ascending/descending order


  Repository:


      public interface ProductRepository extends JpaRepository<Product, Long> { }
🔸 Service:

      public Page<Product> getPaginatedProducts(Pageable pageable) {
          return productRepository.findAll(pageable);
      }
🔸 Controller:

    @GetMapping("/products")
    public Page<Product> getPaginatedProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "5") int size,
            @RequestParam(defaultValue = "name,asc") String[] sort) {
    
        Sort.Direction direction = sort[1].equalsIgnoreCase("desc") ? Sort.Direction.DESC : Sort.Direction.ASC;
        Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sort[0]));
        return productService.getPaginatedProducts(pageable);
    }

    
🌐 Frontend Call (React/Postman):


      GET /products?page=0&size=5&sort=price,desc
✅ Output JSON:


      {
        "content": [...products...],
        "totalPages": 20,
        "totalElements": 100,
        "size": 5,
        "number": 0,
        "first": true,
        "last": false
      }



2. Filtering (Static + Dynamic using Specification)



You want to search products by optional filters like:

category

price < 1000

name contains "laptop"

🔹 Static Filtering (Easy):


      List<Product> findByCategory(String category);
      List<Product> findByPriceLessThan(double price);
URL:

      GET /products/filter?category=electronics



Dynamic Filtering using Specification
When filters are optional or combined dynamically.

📦 Step 1: Specification Class

When filters are optional or combined dynamically.

      public class ProductSpecifications {
     
        public static Specification<Product> hasCategory(String category) {
            return (root, query, cb) -> cb.equal(root.get("category"), category);
        }
    
        public static Specification<Product> hasPriceLessThan(Double price) {
            return (root, query, cb) -> cb.lessThan(root.get("price"), price);
        }
    }


Repository

      public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {}


📦 Step 3: Controller

    @GetMapping("/products/filter")
    public List<Product> filterProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double price) {

    Specification<Product> spec = Specification.where(null);

    if (category != null) spec = spec.and(ProductSpecifications.hasCategory(category));
    if (price != null) spec = spec.and(ProductSpecifications.hasPriceLessThan(price));

    return productRepo.findAll(spec);
    }

API Versioning
🎯 Real-Time Use Case:
Your app evolves.

        v1 had: id, name
        
        v2 added: description, rating

We don't want to break old clients.

🔹 Option 1: URL Versioning

      @RestController
      @RequestMapping("/api/v1/products")
      public class ProductV1Controller {
          @GetMapping
          public ProductV1 get() {
              return new ProductV1("TV");
          }
      }
      
      @RestController
      @RequestMapping("/api/v2/products")
      public class ProductV2Controller {
          @GetMapping
          public ProductV2 get() {
              return new ProductV2("TV", "Smart TV", 4.5);
          }
      }


2: Header Versioning

      
      @GetMapping(value = "/products", headers = "X-API-VERSION=1")
      public ProductV1 v1() {
          return new ProductV1("TV");
      }
      
      @GetMapping(value = "/products", headers = "X-API-VERSION=2")
      public ProductV2 v2() {
          return new ProductV2("TV", "Smart TV", 4.5);
      }
Call with Postman:


      Header: X-API-VERSION = 2

HATEOAS (Hypermedia)
🎯 Real-Time Use Case:
When client receives one product, they should also get:

Link to update

Link to all products

Link to related category

📦 Setup:


      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-hateoas</artifactId>
      </dependency>
📦 Code:


        @GetMapping("/products/{id}")
        public EntityModel<Product> getProduct(@PathVariable Long id) {
            Product p = productRepo.findById(id).orElseThrow();

    EntityModel<Product> model = EntityModel.of(p);

    model.add(linkTo(methodOn(ProductController.class).getProduct(id)).withSelfRel());
    model.add(linkTo(methodOn(ProductController.class).getAll()).withRel("all-products"));
    model.add(linkTo(methodOn(ProductController.class).updateProduct(id, null)).withRel("update"));

    return model;
    }
✅ Output JSON:

      {
        "id": 1,
        "name": "Laptop",
        "_links": {
          "self": { "href": "/products/1" },
          "all-products": { "href": "/products" },
          "update": { "href": "/products/1/update" }
        }
      }
