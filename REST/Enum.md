What is “Enum as PathVariable or RequestParam” in REST?
👉 Sometimes, instead of accepting free-form text like "PENDING" or "APPROVED" as strings in your URL or query params, we want to use Enums in Java so that:

✅ Only valid values are allowed
✅ Code is type-safe and avoids typos
✅ We get autocomplete and avoid if-else string comparisons

🎯 Scenario: Let’s say we have a system that handles Order Status
Step 1: Create Enum
      
      public enum Status {
          PENDING,
          APPROVED,
          REJECTED
      }


👉 This is a simple enum that represents possible values for an order’s status.

Using Enum as @PathVariable (i.e., inside the URL path)
Controller

    @RestController
    @RequestMapping("/orders")
    public class OrderController {
    
        @GetMapping("/status/{status}")
        public ResponseEntity<String> getOrdersByStatus(@PathVariable Status status) {
            // this "status" is auto-converted from URL to enum
            return ResponseEntity.ok("You requested orders with status: " + status);
        }
    }
URL: /orders/status/APPROVED

Spring will convert "APPROVED" (string) to Status.APPROVED enum object automatically

It works only when the string matches exactly (case-sensitive)


 Using Enum as @RequestParam (i.e., in query string)

     @GetMapping("/filter")
    public ResponseEntity<String> filterOrders(@RequestParam Status status) {
        return ResponseEntity.ok("Filtering orders by status: " + status);
    }

Call like:


    GET /orders/filter?status=PENDING


Spring will convert PENDING to Status.PENDING.

What if user sends lowercase? (e.g., /orders/filter?status=approved)
❌ You'll get:

      400 Bad Request – Cannot convert 'approved' to enum


Fix: Make it Case-Insensitive (Optional but Professional)

        @InitBinder
        public void initBinder(WebDataBinder binder) {
            binder.registerCustomEditor(Status.class, new PropertyEditorSupport() {
                @Override
                public void setAsText(String text) {
                    setValue(Status.valueOf(text.toUpperCase()));
                }
            });
        }
Add this in the controller — now it will convert any case like approved, APPROVED, Approved to Status.APPROVED.





 When Do We Use Enum in REST?
🔹 1. Admin/Backoffice Panels
Example: Your internal admin wants to filter orders by status.


        GET /orders/filter?status=REJECTED
➡ Admin sees all rejected orders in the dashboard.

✅ So this is a backend/internal use case — not for normal app users.

🔹 2. For Internal System-to-System Calls (Microservices)
One microservice calls another and sends enum status:


        POST /approval/update-status?status=APPROVED
➡ Here, another system sends that value, not an end-user.

🔹 3. For Frontend Dropdowns with Preloaded Values
If frontend wants to let user pick status (e.g., to filter orders), it can first call:


        GET /statuses
Backend will return:


      ["PENDING", "APPROVED", "REJECTED"]
➡ Then user selects from a dropdown — and frontend sends status=APPROVED to backend.
Still, it’s frontend choosing from controlled values, not user typing it manually.

✅ Real-World Architecture Flow

User (UI)
   ↓
Frontend (React/Angular)
   ↓        → Shows dropdown with values from enum (from API)
Backend (Spring Boot REST)
   ↓
Database (Status column stored as Enum or String)




ex


Status.java (Enum)

    package com.example.demo.enums;
    
    public enum Status {
        PENDING,
        APPROVED,
        REJECTED
    }


This defines allowed values for order status. We use this enum everywhere to ensure only valid values go in/out.


OrderResponseDTO.java

      package com.example.demo.dto;
      
      import com.example.demo.enums.Status;
      
      public class OrderResponseDTO {
      
          private int orderId;
          private String itemName;
          private Status status;
      
          // Constructor
          public OrderResponseDTO(int orderId, String itemName, Status status) {
              this.orderId = orderId;
              this.itemName = itemName;
              this.status = status;
          }
      
          // Getters
          public int getOrderId() {
              return orderId;
          }
      
          public String getItemName() {
              return itemName;
          }
      
          public Status getStatus() {
              return status;
          }
      }


🧠 This is just a dummy response DTO we’ll return from the API, simulating order details.

✅ Step 3: OrderController.java

      
      package com.example.demo.controller;
      
      import com.example.demo.dto.OrderResponseDTO;
      import com.example.demo.enums.Status;
      import org.springframework.web.bind.WebDataBinder;
      import org.springframework.web.bind.annotation.*;
      
      import java.beans.PropertyEditorSupport;
      import java.util.List;
      
      @RestController
      @RequestMapping("/orders")
      public class OrderController {
      
          // This allows case-insensitive enum values like "approved", "APPROVED", etc.
          @InitBinder
          public void initBinder(WebDataBinder binder) {
              binder.registerCustomEditor(Status.class, new PropertyEditorSupport() {
                  @Override
                  public void setAsText(String text) {
                      setValue(Status.valueOf(text.toUpperCase()));
                  }
              });
          }
      
          // GET by status as PathVariable
          @GetMapping("/status/{status}")
          public OrderResponseDTO getOrderByStatus(@PathVariable Status status) {
              // Simulate one order for the given status
              return new OrderResponseDTO(101, "iPhone", status);
          }
      
          // GET by status as RequestParam
          @GetMapping("/filter")
          public List<OrderResponseDTO> filterOrders(@RequestParam Status status) {
              // Simulate a list of filtered orders
              return List.of(
                      new OrderResponseDTO(201, "MacBook", status),
                      new OrderResponseDTO(202, "AirPods", status)
              );
          }
      }
🧠 This controller shows both types:

/orders/status/APPROVED (PathVariable)

/orders/filter?status=pending (RequestParam)

✅ Step 4: application.properties (optional)
  
       server.port=8081
✅ Run Example Requests:
1. Test Enum as PathVariable (Case-Insensitive):

        GET http://localhost:8081/orders/status/approved
Response:


    {
      "orderId": 101,
      "itemName": "iPhone",
      "status": "APPROVED"
    }
2. Test Enum as RequestParam (Case-Insensitive):


    GET http://localhost:8081/orders/filter?status=Rejected
Response:

        [
          {
            "orderId": 201,
            "itemName": "MacBook",
            "status": "REJECTED"
          },
          {
            "orderId": 202,
            "itemName": "AirPods",
            "status": "REJECTED"
          }
        ]

