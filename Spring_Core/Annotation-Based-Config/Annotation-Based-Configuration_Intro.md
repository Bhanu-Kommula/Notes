###  Annotation-Based Configuration in Spring

Now let’s see how we can create beans and inject dependencies using **Annotation-Based Configuration** (without using any XML).

### Step 1: Create a Configuration Class

We create a **separate Java class** where we define our beans.  
To do this, we use:

- `@Configuration` → To tell Spring this is a config class
- `@Bean` → To define methods that return objects (beans)


Student class

    package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;

    public class Student {
    private String name;
    private int age;
    private String gender;
    private StudentAddress address;

    public String getName() {
        return name;
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
    @Qualifier("studentAddress1")
    public void setAddress(StudentAddress address) {
        this.address = address;
    }

    public void getStudentDetails() {
        System.out.println("Name: " + name + ", Age: " + age + ", Gender: " + gender + ", Address: " + address.getStudentAddress());
    }
    }


StudentAddress class

    package com.myproject.java_spring;

    public class StudentAddress {
    private String city;
    private String state;

    public StudentAddress(String city, String state) {
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



StudentConfig class


     package com.myproject.java_spring;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Scope;

    @Configuration
    public class StudentConfig {
    @Bean(name = "studentInfo")
    @Scope(value = "prototype")
    public Student student() {
        Student student = new Student();
        student.setName("BhanuPrasad");
        student.setAge(22);
        student.setGender("Male");
    return student;
    }

    @Bean
    public StudentAddress studentAddress() {
        return new StudentAddress("Dallas", "TX");

    }


    @Bean
    public StudentAddress studentAddress1() {
        return new StudentAddress("Palno", "TX");

    }
    }


- `@Configuration` → To tell Spring this is a config class
- `@Bean` → To define methods that return objects (beans)

Step 2: Create the Spring Container

SpringDemos class - Main Class

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

    Student student2 = (Student) context.getBean("studentInfo");
        System.out.println( "   ----------------- \n");


        System.out.println("First bean" + student);
        System.out.println("Second bean" + student2);

    }
    }


Instead of using ClassPathXmlApplicationContext, we now use:

    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfiguration.class);
This line will:

    Create the Spring container
    Look for the @Configuration class
    Create and manage all the beans defined using @Bean




 **Note:**  
In this example, we used:
- **Setter injection** (`setAddress(...)`)
- **`@Autowired` + `@Qualifier`** to inject the right `StudentAddress` bean
- Also showed **`@Scope("prototype")`** to create a new object on every `getBean()` call

We already discussed these concepts in **XML-based configuration** earlier.  
So if you want to deeply understand how setter injection, autowiring, and qualifiers work step by step — check the **XML-Based** section:

 `Notes/Spring_Core/XML-Based/@Autowired,@Qualifier_nd_Scope.md`

The concepts are **exactly the same** — only the syntax changes between XML and annotation-based config.

---
