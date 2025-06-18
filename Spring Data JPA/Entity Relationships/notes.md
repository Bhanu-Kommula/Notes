Why Entity Relationships Matter?
     
      In the real world, one table is always connected to other tables:
      
      A Student has one Address (OneToOne)
      
      A Store has many Products (OneToMany)
      
      A Product belongs to one Store (ManyToOne)
      
      A Student has many Courses, and vice versa (ManyToMany)

=====


      A Person has One Passport → OneToOne
      
      A Department has Many Employees → OneToMany
      
      Many Students belong to One School → ManyToOne
      
      A Student can join many Courses, and each Course has many Students → ManyToMany
      


1. Types of Entity Relationships
🔹 A. @OneToOne
💡 One entity has exactly one related entity.

Example: One User has One Address

✅ Step 1: Address.java

      
      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class Address {
      
          @Id
          private int id;
          private String city;
          private String state;
      }

✅ Step 2: User.java

      
      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class User {
      
          @Id
          private int id;
          private String name;
      
          @OneToOne  // 💡 Defines one-to-one mapping
          @JoinColumn(name = "address_id") // 🧠 FK in User table pointing to Address(id)
          private Address address;
      }
      
This will create:

      
      User table:
      id | name | address_id (FK to Address.id)




  OneToMany & ManyToOne (Bidirectional)
Example: One Department has many Employees

✅ Step 1: Department.java

      
      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class Department {
      
          @Id
          private int id;
          private String name;
      
          @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
          private List<Employee> employees;
      }
✅ Step 2: Employee.java


      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class Employee {
      
          @Id
          private int id;
          private String name;
      
          @ManyToOne  // 💡 Many employees belong to one department
          @JoinColumn(name = "dept_id") // 🧠 FK column in Employee table
          private Department department;
      }
💡 This creates:

      
      Employee table:
      id | name | dept_id (FK to Department.id)


ManyToMany Relationship
Example: Student enrolls in many Courses, and Courses have many Students.

✅ Step 1: Student.java

      
      @Entity
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class Student {
      
          @Id
          private int id;
          private String name;
      
          @ManyToMany
          @JoinTable(
              name = "student_course", // 🔁 Join table
              joinColumns = @JoinColumn(name = "student_id"),
              inverseJoinColumns = @JoinColumn(name = "course_id")
          )
          private List<Course> courses;
      }
✅ Step 2: Course.java

    
    @Entity
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class Course {
    
        @Id
        private int id;
        private String title;
    
        @ManyToMany(mappedBy = "courses")  // 👈 This side mapped by Student
        private List<Student> students;
    }
💡 This creates 3 tables:
      
      student
      
      course
      
      student_course (join table with student_id & course_id)

 Fetch Types: EAGER vs LAZY
FetchType        	Meaning
  EAGER	        Loads the associated entity immediately with main entity
LAZY	          Loads only when accessed in code (default for collections)







real-time scenarios explaining cascade, fetch, and orphanRemoval clearly:

🔸 CascadeType.ALL – Real-Time Example
Use Case: Blogging platform
Scenario: You want to save a blog post with its comments in one go.

java
Copy
Edit
@Entity
public class BlogPost {
    @Id private Long id;
    private String title;

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL)
    private List<Comment> comments;
}

@Entity
public class Comment {
    @Id private Long id;
    private String content;

    @ManyToOne
    private BlogPost post;
}
🟢 Explanation:
If you create a new BlogPost with 3 Comment objects and just call blogPostRepo.save(post), it will automatically save the comments too.
Without cascade = CascadeType.ALL, you’d need to manually save each comment.

🔸 FetchType.EAGER vs LAZY
Use Case: E-Commerce website
Scenario:

java
Copy
Edit
@Entity
public class Category {
    @Id private Long id;
    private String name;

    @OneToMany(mappedBy = "category", fetch = FetchType.LAZY)
    private List<Product> products;
}
🟢 LAZY (default for OneToMany)
You load the Category → products will not be loaded immediately.
They load only when accessed, saving performance (especially useful when products are many).

🟠 EAGER (default for ManyToOne)
If Product has a @ManyToOne Category, it will be fetched immediately when you fetch Product.

Use LAZY when:

You don’t always need the child data.
Use EAGER when:

You always show both (e.g., Product with Category name).

🔸 orphanRemoval = true
Use Case: University course management
Scenario:

java
Copy
Edit
@Entity
public class Course {
    @Id private Long id;
    private String name;

    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Student> students;
}
🟢 Explanation:
If you remove a Student from the students list of a course and save the course, JPA will also delete the removed student from the DB — no need for manual delete.

Without orphanRemoval, it will still stay in the DB, causing junk records.














Example:


        @OneToMany(fetch = FetchType.LAZY) // loads later
        🔥 Cascade Types
        They define how actions (save, delete) cascade from parent to child:
        
....


      cascade = CascadeType.ALL
      Common types:
      
      PERSIST – Save children when parent saved
      
      REMOVE – Delete children when parent deleted
      
      MERGE – Merge child entities when parent merged
      
      ALL – All above
      
      DETACH, REFRESH – Less common




orphanRemoval

@OneToMany(orphanRemoval = true)
If you remove a child from list, it deletes it from DB too.

🔁 Uni vs Bi-directional Mapping
        | Mapping Type   | Description                                |
| -------------- | ------------------------------------------ |
| Unidirectional | Only one entity knows the relation         |
| Bidirectional  | Both entities know and manage the relation |



Summary

| Concept       | Keyword                | Notes                        |
| ------------- | ---------------------- | ---------------------------- |
| OneToOne      | `@OneToOne`            | FK column with `@JoinColumn` |
| OneToMany     | `@OneToMany`           | Use `mappedBy`               |
| ManyToOne     | `@ManyToOne`           | FK in child entity           |
| ManyToMany    | `@ManyToMany`          | Needs `@JoinTable`           |
| Fetch Type    | `EAGER`, `LAZY`        | EAGER = fast but heavy       |
| Cascade       | `CascadeType.ALL`      | Auto-save/delete child       |
| orphanRemoval | `orphanRemoval = true` | Deletes removed children     |
| Bidirectional | `mappedBy` used        | Both sides talk              |
