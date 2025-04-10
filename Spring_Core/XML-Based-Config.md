**Spring** has an interface called **`ApplicationContext`**, which contains several abstract methods.  
As we know, this **`ApplicationContext`** is used to **load the Spring container**.

There’s a built-in class called **`ClassPathXmlApplicationContext`**, which takes a **string input** (usually the XML file ).  
This class contains built-in logic to **create the beans** and **inject their dependencies** — it was developed by Spring developers to make our work easier.

So, we create an `.xml` file (like **`Employee.xml`**) that contains the **bean definitions** and their **dependencies**.

When we create an object of **`ClassPathXmlApplicationContext`**, it's like we are **creating (or loading) the Spring container**.  
Since we pass the `.xml` file to this class, by the time the object is created, **all the beans (objects) and their dependencies are already created and injected by Spring automatically**.


Example:

    // Employee.java
    
    package com.myproject.java_spring;

    public class Employee {
    
    private int id;
    private String name;
    private int age;
    private Address address;

    public Address getAddress() {
        return address;
    }


    public Employee(int id, String name, int age, Address address) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public int getId() {
        return id;
    }
    public int getAge() {
     return age;    }
    public String getName() {
        return name;
    }

    public void getEmployeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() + " address : " + address.getAddress());
    }
    }

This is the employee class, it has 4 dependencies ( which include 3 varibales and one object(address class)).
         
     private int id;
     private String name;
     private int age;
     private Address address;

and also has the 4-argument constructor. and getter methods along with getEmployeeDetails()  which is printing out complete employee details. 


    // Address. Java
    
    package com.myproject.java_spring;

    public class Address {
    
    private String city;
    private String State;
    private int zipCode;

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getState() {
        return State;
    }

    public void setState(String state) {
        State = state;
    }

    public int getZipCode() {
        return zipCode;
    }

    public void setZipCode(int zipCode) {
        this.zipCode = zipCode;
    }

    public String getAddress() {
        return " City " + city + " State " + State + " Zip Code " + zipCode;
    }
    }

 In this address class we have 3 dependencies 
     
    private String city;
    private String State;
    private int zipCode;

and setter and getter methods to set and get the values, and also one getAddress method which returns all the values (dependencies together)


    //Employee.XML 
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="emp" class="com.myproject.java_spring.Employee">
        <constructor-arg value="1"/>
        <constructor-arg value="Bhanu"/>
        <constructor-arg value="22"/>
        <constructor-arg ref="add"/>
    </bean>

    <bean id="add" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>
    </beans>

This is the XML file. here we declare all the beans and dependencies. 

Initially, we are declaring the <?xml> to let the java complier know this is the xml file .


     <Beans>  ( this is the opening tag of Beans)
     </Beans> ( this is the closing tag of Beans)

So we declare all the beans inside the <Beans> tag. and the <Beans> tag contains the definition of spring beans mean.  Don't worry about we can copy it directly from the Google, or if you are using Maven project, then Maven will handle it for you(it will come directly when creating the XML file in a Maven project).

Now to declare a Bean, we use open and close <Bean> tags

Usually the Bean tag has two things 
  
    1 - ID = ""  Id can be anything its basically like give you bean a name to call or use that bean whenever we need it.
    2 - class = "" in the class, we give the complete class path for which we want to create the bean.
    
So simply, in a Bean tag we will say like 
  hey spring create a Bean (object) for the class at ( com.myproject.java_spring.Address) [ in this package - for this class] and name that bean as (id ) .


  So this <Bean> creates the bean. [ So object creation is done] 

  now DI 

  So, in  XML-based configuration the dependency injection can be done in two ways

      1 - setter injection ( we inject dependencies usign the setter methods)
      2 - Constructor injection ( we inject dependencies usign the Constructor )


  So in the above code, 
      for the Employee class, the dependencies were injected using constructor injection.
      For the Address class, the dependencies were injected using setter injection.


  <b> Constructor Injection </b>

  so we know <bean id="" class=""> is used to create the Bean (object )

  Now to do constructor injection, we will use the <constructor-arg> tag

    <bean id="emp" class="com.myproject.java_spring.Employee">
        
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="Bhanu"/>
        <constructor-arg value="22"/>
        <constructor-arg index="3" ref="add"/>
        
    </bean>

So the <constructor-arg> tag is used to inject the dependecies (values) so we specify under the values attribute (values ="") and as we are doing based on constructor, the index attribute is used to say the index of the parameters. as we know the id is the first parameter so declared the index as 0 and as we know the address obj is last (4) parameter so we declared the index value as 3.

So to inject values we use the value attribute and to inject the objects we use the reference attribute (ref = "" ). As in our case, the address variable is used to hold the Address class object so we used the ref = "" instead of value = "".  The ref = " reference class bean id " so we pass the bean id of other class asa  reference so in our case the bean id of the Address class is add so we passed ( ref = "add")


  <b> Setter Injection </b>

 The <property> tag is used to inject the dependencies (values) in the setter injection. 

    <bean id="add" class="com.myproject.java_spring.Address">
    
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
        
    </bean>

this <property> tag has 2 main properties 
    1 - name = ""  [ here we declare the feild name so into which variable(field) we want spring to inject the value(dependency).
    2 - value = "" ( here we declare the values )
    3 - ref = "" ( we use this as to reference to other object, and we pass the bean id of the reffered class).



Finally the main class.

       //SpringDemos.java

       package com.myproject.java_spring;

     import org.springframework.context.ApplicationContext;
      import org.springframework.context.support.ClassPathXmlApplicationContext;

      public class SpringDemos {
      public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("Employee.xml");
     
        Employee employee = (Employee) context.getBean("emp");
        System.out.println(employee.getName());
        System.out.println(employee.getAddress().getCity());
        System.out.println("-----------------");
      
      employee.getEmployeeDetails();
    }
    }

 
 ApplicationContext context = new ClassPathXmlApplicationContext("Employee.xml");   is used to load the container so at the time of this class creation it will go to the Employee.xml file create all the beans and inject the dependencies. 

 Now using the getBean(" bean id ")  we are getting the object. 
 so the getBean() returns an object. In our case, we are requesting the container to give the bean of employee. 
                     
                     Employee employee = (Employee) context.getBean("emp");

So the getBean() returns us the Employee class object so we are typcasting it and storing in Employee type employee variable. 

and using this variable to access the data. 



Next Concept - https://github.com/Bhanu-Kommula/Notes/blob/2433fabe5a3cf289f479c57533aeeeb2509892d6/Spring_Core/Autowiring_Spring-(XML_Based).md
Previous Concpet - https://github.com/Bhanu-Kommula/Notes/blob/2433fabe5a3cf289f479c57533aeeeb2509892d6/Spring_Core/Spring_Intro.md
