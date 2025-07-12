[ so as entry point in our case is when user places order then the action should be taken so we start building form the order]



Folder Structure (Post-Creation)
Configure pom.xml 
Configure application.properties
Create Order Entity & OrderRepository
 Create REST API to Place Order    ->  Create DTO for API Request  -> Create Service Layer --> Create Controller Layer--> Test the API (Postman)
  Create a New Spring Boot Project: inventory-service --> application.properties --> Create OrderPlacedEvent.java --> Create Kafka Consumer --> Test  --> to have DB Product Entity & Repository -->  service logic
  
  Create a New Project: notification-service --> application.properties --> Create DTO to Receive Kafka Event --> Create Kafka Consumer Service  --> Test 

   Setup Eureka Server & Register Services--> application.properties -->  Enable Eureka Server ( DiscoveryServerApplication.java (mainclass)) --> Register Other Services --> eureka client Add this dependency to each service (order-service, inventory-service, etc.) --> Update application.properties of each service: 

NOTe:

    ✅ Exactly. You're mostly right — but let’s break it down properly.
     
     💡 When Do We Use Eureka?
     Eureka is used when you want to dynamically discover service instances by name, instead of hardcoding their IP/port.
     
     So...
     
     ✅ Eureka is definitely useful in:
     ✅ REST-based communication
     One microservice calls another using Feign, RestTemplate, or WebClient.
     
     You write:
     http://inventory-service/api/inventory/123
     instead of
     http://localhost:8082/api/inventory/123
     
     Eureka resolves the real address behind inventory-service.
     
     ❌ Eureka is not needed for:
     ❌ Pure Kafka-based communication
     Services don't call each other.     
     They publish/subscribe to Kafka topics.     
     Kafka brokers act as the middleman.     
     Eureka doesn't help here, because:     
     Services don’t need to know where the other service is     
     Kafka handles the message delivery     
     🔄 When Both Kafka & REST Are Used:
     Many real-world systems use both:    
     Use Kafka for async, decoupled communication     
     Use REST when you need instant responses or data sync
     ➡️ In this hybrid case, Eureka becomes very helpful for REST-based calls.


API GATEWAY 

         What is the use of API Gateway (Spring Cloud Gateway)?
     An API Gateway is the single entry point into your entire microservices system.
     Think of it as a front desk at a hotel:
     You don’t go directly to the rooms (services),
     You talk to the receptionist (API Gateway),
     They guide you to the right place (routes the request to the right service).
     
     🧠 Why Use Spring Cloud Gateway?
     Spring Cloud Gateway helps you:
     
     ✅ Route Requests to Microservices
     🔐 Secure your system centrally (JWT auth, roles)
     🪄 Apply common logic like logging, rate limiting, headers
     ⚙️ Load balance using Eureka + lb://service-name
     🔁 Rewrite paths, forward headers, manipulate payloads
     
     📊 Monitor traffic easilySo if we’re using Kafka, do we still need or use API Gateway?
     Answer:
     Yes — but for different reasons.
     
     ✅ Kafka is for internal service-to-service communication (async)
     Kafka lets your services talk indirectly using topics:
     
     order-service → produces to Kafka
     inventory-service → consumes from Kafka
     ➡️ No API Gateway involved in that communication
     ➡️ It’s internal, event-driven using Kafka topics
     
     ✅ API Gateway is for external client → microservice access (sync)
     If your frontend app (like React/Angular/mobile) or external client wants to:
     Place an order: POST /api/orders
     Fetch order details: GET /api/orders/{id}
     Register a user: POST /api/users
     📍 These are HTTP requests — they go through the API Gateway.
     
     API Gateway acts like a gatekeeper:
     🔐 Checks JWT or roles
     📦 Routes the request to the correct service (via Eureka or static routing)
     🧾 Returns response to client
