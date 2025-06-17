What is Global Exception Handling?
🔹 Definition:
Global Exception Handling is a way to catch any error or exception thrown from any controller or service in your Spring Boot app in one centralized place.

You don’t need to write try-catch everywhere. Spring handles it automatically using:

      @ControllerAdvice → Tells Spring: "This class handles exceptions globally"
      
      @ExceptionHandler(Exception.class) → Tells Spring: "Call this method when an exception happens"
      
      
✅ Why Use Global Exception Handling?


| Problem                                | Global Exception Solves        |
| -------------------------------------- | ------------------------------ |
| Repeating try-catch in all controllers | ❌ Avoided                      |
| Sending unstructured error messages    | ✅ Gives consistent error JSON  |
| No control over status codes           | ✅ Lets you send 404, 500, etc. |
| Bad developer experience               | ✅ Cleaner, maintainable code   |


Real-Time Analogy
Think of @ControllerAdvice like a watchman at the gate.
If anything goes wrong inside your service/controller, he will catch the problem and respond politely — instead of letting Spring throw an ugly stack trace.

✅ Key Components
🔹 1. @ControllerAdvice
Marks a class as the global exception handler.

🔹 2. @ExceptionHandler
Tells which method handles which exception.

🔹 3. ErrorResponse (custom class)
Gives consistent error format: timestamp, message, status

🔹 4. Custom Exceptions
Like RecordNotFoundException, ValidationException, etc.



📁 RecordNotFoundException.java

      package com.myprojects.exception;

    public class RecordNotFoundException extends RuntimeException {
        public RecordNotFoundException(String message) {
            super(message);
        }
    }


ErrorResponse.java


      package com.myprojects.exception;
    
    import java.time.LocalDateTime;
    import lombok.*;
    
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class ErrorResponse {
        private LocalDateTime timestamp;
        private int status;
        private String error;
    }


GlobalExceptionHandler

      
      package com.myprojects.exception;
      
      import org.springframework.http.*;
      import org.springframework.web.bind.annotation.*;
      import java.time.LocalDateTime;
      
      @ControllerAdvice
      public class GlobalExceptionHandler {
      
          @ExceptionHandler(RecordNotFoundException.class)
          public ResponseEntity<ErrorResponse> handleNotFound(RecordNotFoundException ex) {
              ErrorResponse error = new ErrorResponse(
                      LocalDateTime.now(),
                      HttpStatus.NOT_FOUND.value(),
                      ex.getMessage()
              );
              return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
          }
      
          @ExceptionHandler(Exception.class)
          public ResponseEntity<ErrorResponse> handleAll(Exception ex) {
              ErrorResponse error = new ErrorResponse(
                      LocalDateTime.now(),
                      500,
                      "Internal Error: " + ex.getMessage()
              );
              return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
          }
      }


StoreService
        
        public StoreEntity getRecordById(int id) {
            return storeRepository.findById(id)
                .orElseThrow(() -> new RecordNotFoundException("No record with ID: " + id));
        }


Notes Summary for Interview / Revision
         
            
            Concept	Description

          @ControllerAdvice	Global exception manager
          @ExceptionHandler	Handles specific exception types
          ErrorResponse	DTO for structured error
          RecordNotFoundException	Custom exception for clean service logic
          ResponseEntity<ErrorResponse>	Sends status + message to client
          Why it's useful?	Centralized, clean, reusable, professional
          
