What is a Transaction?
A transaction is a logical unit of work that must be completed entirely or not at all (ACID). In Java/Spring, this is often a set of DB operations that we want to treat as a single unit.


. @Transactional Annotation
🔸 Basic Usage

    
    @Transactional
    public void processOrder() {
        // DB operations: Save order, update stock, generate invoice
    }
✅ If any line fails, all DB changes are rolled back.



Where to Use

| Location         | Use Case                                   |
| ---------------- | ------------------------------------------ |
| **Method Level** | Apply to specific business method          |
| **Class Level**  | Applies to all public methods in the class |


    @Service
    public class PaymentService {
    
        @Transactional
        public void completePayment(int orderId) {
            orderRepo.updateOrderStatus(orderId, "PAID");
            stockRepo.decreaseStock(orderId);
            // If exception → entire operation rolls back
        }
    }


✅ 2. Read-Only vs Write Transactions


Read-Only Transaction

        
        @Transactional(readOnly = true)
        public List<Order> fetchOrders() {
            return orderRepo.findAll();
        }
💡 readOnly = true tells Spring:

Optimize for SELECTs

Don’t allow INSERT/UPDATE/DELETE

Improves performance

Useful for read-only services like reporting, analytics

✅ 3. Rollback Scenarios
🔸 Default Behavior
By default, Spring rolls back only on RuntimeException or Error

      
      @Transactional
      public void saveData() {
          repository.save(entity);
          throw new RuntimeException("fail"); // ✅ Rolls back
      }

      
      @Transactional
      public void saveData() throws IOException {
          throw new IOException();  // ❌ Won’t roll back
      }
🔸 Custom Rollback Rules
You can control rollback behavior:


      
      @Transactional(rollbackFor = IOException.class)
      public void saveData() throws IOException {
          throw new IOException(); // ✅ Now it rolls back
      }
✅ 4. Commit & Rollback Flow
Step	What Happens
Start	Transaction starts
Try Block	All DB operations
If Success	Commits transaction
If Exception	Rolls back transaction

🔁 Nested Transactions (Advanced)
If using nested methods, only outermost @Transactional applies unless using REQUIRES_NEW propagation.

💡 Real-Time Tips
Tip	Why
Use @Transactional on Service layer	Controller should not manage transactions
Avoid using on private methods	Spring AOP can’t proxy them
Use readOnly = true for query-only methods	Improves speed
Handle rollback scenarios explicitly	Especially for checked exceptions

✅ Summary
Feature	Annotation
Start transaction	@Transactional
Read-only operation	@Transactional(readOnly = true)
Rollback on checked	rollbackFor = Exception.class
Scope	Method or Class level

