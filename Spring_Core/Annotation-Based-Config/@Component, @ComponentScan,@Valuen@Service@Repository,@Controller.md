###  Using `@Component` and `@ComponentScan` in Spring

In annotation-based configuration, we were using `@Bean` inside a `@Configuration` class to declare beans manually.

But Spring also gives us an easier way — we can simply use `@Component` directly on the class.  
This makes the class a **Spring-managed bean automatically**, without needing to define it in any method.

---

### What does `@ComponentScan` do?

To tell Spring **where to look** for these `@Component` classes, we use:


        @ComponentScan(basePackages = { "com.myproject.java_spring" })

This annotation tells Spring to scan the given package for any classes marked with @Component, @Service, @Repository, or @Controller.
The basePackages attribute accepts an array of strings, so you can include multiple packages if needed.
It will also scan subpackages of the given package.

Injecting Values using @Value
When you're using @Component, and want to inject values (like strings, ints, etc.), you can use:


        @Value("value here")
This works on:
      
       Field level
       Constructor arguments
       Setter methods

So you can choose whichever injection style you prefer.


Student class

    package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Autowired;

    public class Student {
    private String name;
    private int age;
    private String gender;
    private StudentAddress address;
    
     @Autowired
    private StudentMarks  marks;

    public String getName() {
        return name;
    }

    public StudentMarks getMarks() {
        return marks;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public StudentAddress getAddress() {
        return address;
    }

    @Autowired
    public void setAddress(StudentAddress address) {
        this.address = address;
    }

    public void getStudentDetails() {
        System.out.println("Name: " + name + ", Age: " + age + ", Gender: " + gender + ", Address: " + address.getStudentAddress() + ", Marks: " + marks.TotalScore());
    }
    }

StudentAddress class

    package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Component;

    @Component
    public class StudentAddress {

    private String city;
    private String state;

    public StudentAddress(  @Value("NYC") String city,    @Value("NY")  String state) {
        this.city = city;
       this.state = state;
    }
    public String getCity() {
        return city;
    }
    public String getState() {
        return state;
    }

    public String getStudentAddress(){
        return ("Student Address " + city + " " + state  );
    }
      }
      
StudentMarks class

    package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Component;

    @Component
    @Scope("prototype")
    public class StudentMarks {
     @Value("98")
    private int Science;
    @Value("89")
    private int Math;
    @Value("70")
    private int English;



    public int getScience() {
        return Science;
    }

    public int getMath() {
        return Math;
    }

    public int getEnglish() {
        return English;
    }

    public int TotalScore() {
        return Science + Math + English;
    }
    }


Student config class


    package com.myproject.java_spring;

    import org.springframework.context.annotation.*;

    @Configuration
      @ComponentScan(basePackages = {"com.myproject.java_spring"})
    public class StudentConfig {
    @Bean(name = "studentInfo")
    public Student student() {
        Student student = new Student();
        student.setName("BhanuPrasad");
        student.setAge(22);
        student.setGender("Male");
    return student;
    }

    }


Main Class

    package com.myproject.java_spring;


    import org.springframework.context.ApplicationContext;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  
    public class SpringDemos {
    public static void main(String[] args) {

        ApplicationContext context = new AnnotationConfigApplicationContext(StudentConfig.class);
    Student student = (Student) context.getBean("studentInfo");
    System.out.println(student.getName());
        System.out.println(student.getAddress().getCity());

        System.out.println( "   ----------------- \n");

      student.getStudentDetails();



    }
    }

###  Spring Application Layer Structure and Bean Annotations

In Spring, the application development is logically divided into **layers**, and each layer interacts with the next one:

###  Layered Architecture (Top to Bottom):

1. **UI Layer (Web / MVC)**  
   - Deals with user input, controllers, web requests  
   - Uses: `@Controller`, `@RestController`

2. **Service Layer (Business Logic)**  
   - Contains the core logic of the application  
   - Uses: `@Service`

3. **DAO Layer (Persistence / Data Access)**  
   - Responsible for interacting with the database  
   - Uses: `@Repository`

---

###  What Do These Annotations Really Do?

All of the following annotations:
- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`

➡ Internally, they all mean the **same thing**:  
They tell Spring:  
      
       "Hey Spring, please register this class as a bean in the ApplicationContext."

 So yes — **they all create Spring beans**.

But they are named differently to **clearly indicate which layer** they belong to.  
This improves readability and **layer separation** in the code.

---

###  Terminology Explained Simply:

| Layer        | Annotation        | Purpose                                                  |
|--------------|-------------------|----------------------------------------------------------|
| **Any Layer**| `@Component`      | Generic Spring-managed bean                              |
| **Service**  | `@Service`        | Marks business logic layer                               |
| **DAO**      | `@Repository`     | Marks data access layer, adds persistence exception handling |
| **Web/MVC**  | `@Controller`     | Handles web requests (UI layer)                          |
| **REST API** | `@RestController` | Shortcut for `@Controller + @ResponseBody`               |


###  When Is It Called a "Bean"?

An object is called a **Bean** when:
> It is created and managed by the Spring IOC container.

---

###  Scanning Beans Automatically

To scan for all these annotations, we use:

```java
@ComponentScan(basePackages = {"com.myproject"})


It searches for all classes annotated with the above annotations
Includes sub-packages too
Registers those classes as beans in the Spring context


