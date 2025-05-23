  <b> Spring Core Module </b>

Spring: Spring is a lightweight, open-source Java framework designed to simplify the development of web and enterprise applications. It provides good support for building Java applications, mainly focusing on features like dependency injection, aspect-oriented programming, and easy integration with other technologies.

Spring core module is all about IOC (Inversion of Control) and Dependency Injection.

So, to simply define — in Java, normally we create objects and inject the dependencies into them manually. But here, in Spring Core Module, we can ask Spring to take care of this — it will create the object and inject the required dependencies into it.

Spring will have a container (called the Spring Container) which is created when we call the ApplicationContext. So when we call the ApplicationContext, it creates the Spring Container. During this container creation, the objects (beans) and their dependencies will automatically get injected by Spring. So this container will have all the objects along with their injected dependencies.

IOC means object creation is handled by Spring, and DI (Dependency Injection) is all about giving required dependencies to the object.
Here, dependency means – every object has some fields right? So those fields are called dependencies. Giving values or objects to those fields is called injecting.

    
Example: 
'''java
        
        package com.myproject.java_spring;
      
      public class Employee {
           private int id;
           private String name;
           private int age;

    public Employee(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public int getId() {
        return id;
    }
    public int getAge() {
     return age;    }
    public String getName() {
        return name;
    }

    public void employeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() );
    }
    }


''' java

     package com.myproject.java_spring;

     public class SpringDemos {
       public static void main(String[] args) {

        Employee emp = new Employee(1,"Bhanu Prasad",5000);
        emp.employeeDetails();

    }
    }
So here in the above Java code, we have 2 classes:

1 - Employee class
This class has the employee details. It contains:

        Employee ID
        Employee Name
        Employee Age

These are defined as fields — basically, they are the dependencies.
We also have a three-argument constructor.

  ** <b>Constructor is nothing but a special method that is used to initialize the values directly at the time of object creation. </b> **

So here we are using the constructor to load the dependencies, and we have getter methods to return those dependency values. We also have employeeDetails() method to print the complete info of the employee.

2 - SpringDemos class (Main class)

In this class, We are creating an object of the Employee class.
Injecting the dependencies via the 3-argument constructor (since the Employee class has a constructor that takes 3 values).
Then calling employeeDetails() to display the employee info.

Here, we are creating the object using new keyword and also manually injecting the dependencies.
So instead of doing this manually, we can give this job to Spring Container — which will take care of object creation and dependency injection automatically.


<b> IoC-DI-Configuration-Types </b>

So now we know that we can Spring core module can handle the inversion of control and dependency injection.  

So we can achieve this in 3 primary was

<b> 
1 — XML-Based Configuration
 We define the beans and their dependencies inside an XML file (usually applicationContext.xml or similar).

2 — Annotation-Based Configuration
We define the beans and their dependencies using annotations in a configuration class (like with @Configuration, @Bean, etc.).

3 — Java-Based Configuration
 We define the beans and their dependencies using annotations like @Component, @Service, @Repository, and @Autowired directly on Java classes.

</b>



Next Concept - https://github.com/Bhanu-Kommula/Notes/blob/dbe4956915f825fc6ad40c755712d5d364bd58c2/Spring_Core/XML-Based-Config.md



