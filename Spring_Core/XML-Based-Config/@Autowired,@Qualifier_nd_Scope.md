### `@Autowired` Annotation-Based Dependency Injection

So far, we were using the `<bean>` tag with the `autowire` attribute in the XML file to inject dependencies.  
But Spring also provides an easier and cleaner way — using the **`@Autowired` annotation**.

To use `@Autowired`, we need to declare one additional tag in the XML config:

<context:annotation-config/>

      
XML file

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <context:annotation-config/>
    <bean id="emp" class="com.myproject.java_spring.Employee">
      <constructor-arg value="1"/>
        <constructor-arg value="Bhanu"/>
        <constructor-arg value="25"/>


    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class="com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    </beans>  

Employee class

      package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Autowired;

    public class Employee {
    private int id;
    private String name;
    private int age;
    @Autowired
    private Address address;
    @Autowired
    private EmployeeSalary salary;
    @Autowired
    private Vehicle vehicle;

    public Employee(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;

    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Address getAddress() {
        return address;
    }

    public EmployeeSalary getSalary() {
        return salary;
    }

    public Vehicle getVehicle() {
        return vehicle;
    }

    public void getEmployeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() + " address : "
        + address.getAddress() + " salary : " + salary.getSalary() + " Vehicle :" + vehicle.getCarDetails());
    }
    }


Just by placing @Autowired above the fields, Spring will automatically inject the dependencies.
By default, @Autowired works by type. So Spring:

      Checks the type of the field (like Address)
      Looks for a matching bean of the same type in the XML
      Injects that bean automatically


@Autowired Can Be Placed On:

    Fields (most common)
    Constructor (recommended in modern Spring)
    Setter methods (if you prefer setter-based injection)


1️ - Field Injection (default, quick, clean)

    @Autowired
    private Address address;


No constructor or setter needed
Spring injects directly into the field
Easy to write, but harder to unit test/mock


mock

2️ - Constructor Injection (recommended)

    private Address address;

    @Autowired
    public Employee(Address address) {
    this.address = address;
    }

Injected when object is constructed
Works great with final fields
Better for immutability and testing

Also used automatically in Spring Boot without @Autowired if only one constructor exists

3️ - Setter Injection

    private Address address;

    @Autowired
    public void setAddress(Address address) {
    this.address = address;
    }
    
Injects after object creation via setter method
Good if the dependency is optional
Not as commonly used in modern Spring



Now, so till now we have only one bean of each class( address, vehicle, salary) lets say what if employee got 2 vehicles so there will be 2 beans
for those 2 vehicles right then spring will run into ambiguity / confuse to pick which bean so it will throw an error.


XML File

      <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <context:annotation-config/>
    <bean id="emp" class="com.myproject.java_spring.Employee">
      <constructor-arg value="1"/>
        <constructor-arg value="Bhanu"/>
        <constructor-arg value="25"/>


    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class="com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    <bean id="vehicle1" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Nissan"/>
        <property name="model" value="Rouge - Platinum"/>
        <property name="year"   value="2023"/>
    </bean> 
    </beans>


Employee class

    package com.myproject.java_spring;
    
    import org.springframework.beans.factory.annotation.Autowired;

    public class Employee {
    private int id;
    private String name;
    private int age;
    @Autowired
    private Address address;
    @Autowired
    private EmployeeSalary salary;
    @Autowired
    private Vehicle vehicle;

    public Employee(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;

    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Address getAddress() {
        return address;
    }

    public EmployeeSalary getSalary() {
        return salary;
    }

    public Vehicle getVehicle() {
        return vehicle;
    }

    public void getEmployeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() + " address : " + address.getAddress() + " salary : " + salary.getSalary() + " Vehicle :" + vehicle.getCarDetails());
    }
    }


### Handling Multiple Beans of Same Type with `@Autowired` and `@Qualifier`

In this example, we are defining **two beans** of the same class (`Vehicle`) in the XML file:

output 
  
      Bhanu
      Dallas
      Car Model is Hrv - Sport
    ----------------- 

      Employee details - ID : 1  name : Bhanu age : 25 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Honda  Hrv - Sport  2023

      Process finished with exit code 0


Normally, if Spring finds multiple beans of the same type, it gets confused and throws an error like:


NoUniqueBeanDefinitionException

Because in XML-based config, when @Autowired is used:

    Spring first checks the type
    If multiple beans are found, it checks if any bean ID matches the field name

In our case:


      <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
          
      @Autowired
      private Vehicle vehicle; 

Field name = vehicle
Bean ID = vehicle 

So Spring injected that bean!



Now lets change the bean id ans see how spring reacts.

XML file after changing the  vehicle bean ids

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <context:annotation-config/>
    <bean id="emp" class="com.myproject.java_spring.Employee">
      <constructor-arg value="1"/>
        <constructor-arg value="Bhanu"/>
        <constructor-arg value="25"/>


    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class="com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="v" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    <bean id="v1" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Nissan"/>
        <property name="model" value="Rouge - Platinum"/>
        <property name="year"   value="2023"/>
    </bean>

    </beans>


It thored this error 

    Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.myproject.java_spring.Vehicle' available: expected single matching bean but found 2: v,v1

so it is confused, now we can say spring which bean to pick. we can do this by using @Qualifier.


Employee class file after using Qualifier annotation.

    package com.myproject.java_spring;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;

    public class Employee {
    private int id;
    private String name;
    private int age;
    @Autowired
    private Address address;
    @Autowired
    private EmployeeSalary salary;
    @Autowired
    @Qualifier("v1")
    private Vehicle vehicle;

    public Employee(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;

    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Address getAddress() {
        return address;
    }

    public EmployeeSalary getSalary() {
        return salary;
    }

    public Vehicle getVehicle() {
        return vehicle;
    }

    public void getEmployeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() + " address : " + address.getAddress() + " salary : " + salary.getSalary() + " Vehicle :" + vehicle.getCarDetails());
    }
    }

output 

    Bhanu
    Dallas
    Car Model is Rouge - Platinum
    ----------------- 

    Employee details - ID : 1  name : Bhanu age : 25 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Nissan  Rouge - Platinum  2023

    Process finished with exit code 0

So we can see v1 dependencies are injected.





###  `@Scope` in Spring Framework

Spring allows us to define the **lifecycle and visibility of a bean** using the `@Scope` annotation.


###  `@Scope("singleton")` (Default Scope)

So far, we have been using the **default scope**, which is **Singleton**.

    In this scope, Spring creates **only one instance** of the bean.
    The bean is created **at the time of Spring container initialization** (when `ApplicationContext` is created).
    That single instance is then **reused** wherever the bean is injected.

In short:
Only **one object** is created and **shared** throughout the application.

      <bean id="myService" class="com.myproject.MyService" scope="singleton"/>
Even if we don’t specify the scope, Spring treats it as singleton.


scope="prototype"

In this scope, Spring does not create the bean when the container starts.

    Instead, it creates a new bean every time you call getBean() or inject it somewhere.
    Great when you need fresh objects each time.


    <bean id="myService" class="com.myproject.MyService" scope="prototype"/>


