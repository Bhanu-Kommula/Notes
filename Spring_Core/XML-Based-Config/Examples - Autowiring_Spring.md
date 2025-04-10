So here the example code of employee class in regular XML Configuration without any autowire attribute. 

    //Employee class 

    package com.myproject.java_spring;
    
    public class Employee {
   
    private int id;
    private String name;
    private int age;
    private Address address;
    private EmployeeSalary salary;
    private Vehicle vehicle;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

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

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public EmployeeSalary getSalary() {
        return salary;
    }

    public void setSalary(EmployeeSalary salary) {
        this.salary = salary;
    }

    public Vehicle getVehicle() {
        return vehicle;
    }

    public void setVehicle(Vehicle vehicle) {
        this.vehicle = vehicle;
    }

    public void getEmployeeDetails() {
        System.out.println("Employee details - ID : " + getId() + "  name : " + getName() + " age : " + getAge() + " address : " + address.getAddress() + " salary : " + salary.getSalary() + " Vehicle :" + vehicle.getCarDetails());
    }
    }

This class has  6 dependencies ( 3 vlaues and 3 references) 

     private int id;
    private String name;
    private int age;
    private Address address;
    private EmployeeSalary salary;
    private Vehicle vehicle;


Address class

      //Address.java

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
        return "  " + city + "  " + State + "  " + zipCode;
    }
    }

Salary class

      //Salary.java

      package com.myproject.java_spring;

    public class EmployeeSalary {
    private double salary;

    public EmployeeSalary(double salary) {
        this.salary = salary;
    }

    public double getSalary() {
        return salary;
    }

    }

Vehicle class

    //Vehicle.java

    package com.myproject.java_spring;

    public class Vehicle {
    private String make;
    private String model;
    private int year;

    public String getMake() {
        return make;
    }

    public void setMake(String make) {
        this.make = make;
    }

    public String getModel() {
        return model;
    }

    public void setModel(String model) {
        this.model = model;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public String getCarDetails() {
        return " " + make + "  " + model + "  " + year;
    }
    }

Employee XMl file

    //Employee.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="emp" class="com.myproject.java_spring.Employee">
       <property name="id" value="1"/>
        <property name="name" value="BhanuPrasad"/>
        <property name="age" value="24"/>
        <property name="address" ref="address"/>
        <property name="salary" ref="salary"/>
        <property name="vehicle" ref="vehicle"/>

    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class=" com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    </beans>


Main class

    //SpringDemos.java(main class)
      
        package com.myproject.java_spring;
        import org.springframework.context.ApplicationContext;
        import org.springframework.context.support.ClassPathXmlApplicationContext;

    public class SpringDemos {
    public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("Employee.xml");
        Employee employee = (Employee) context.getBean("emp");
        System.out.println(employee.getName());
        System.out.println(employee.getAddress().getCity());
        System.out.println("Car Model is " + employee.getVehicle().getModel());
        System.out.println("    ----------------- \n");

        employee.getEmployeeDetails();
    }
    }

output

            //output 
          
    BhanuPrasad
    Dallas
    Car Model is Hrv - Sport
    ----------------- 

    Employee details - ID : 1  name : BhanuPrasad age : 24 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Honda  Hrv - Sport  2023

    Process finished with exit code 0

    

    
So, The above code is the tradiational XML based configuration. 



### Spring XML Autowiring – `autowire="byName"`

Now let's use the `<bean>` tag’s `autowire` attribute and ask the container to automatically inject the dependencies.

####  `autowire = "byName"`

1 **`byName` autowiring only works with setter methods**, and there must be a **default constructor** in the class.  
   We need this because Spring will first create the object using the default constructor and then inject dependencies using the setter methods.

2 The **field names (variable names)** in the class must match the **bean IDs** in the XML.  
   That’s how Spring knows which bean to inject into which property.

---





so in the above code i made changes in the XML file.

     //Employee.xml  [ OLD File -[Old – Manual Injection] ]

    <bean id="emp" class="com.myproject.java_spring.Employee">
       <property name="id" value="1"/>
        <property name="name" value="BhanuPrasad"/>
        <property name="age" value="24"/>
        <property name="address" ref="address"/>
        <property name="salary" ref="salary"/>
        <property name="vehicle" ref="vehicle"/>

    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class=" com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    </beans>


    // Employee.xml  [ New file  - [New – Autowiring by Name]]
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="emp" class="com.myproject.java_spring.Employee" autowire="byName">
       <property name="id" value="1"/>
        <property name="name" value="BhanuPrasad"/>
        <property name="age" value="24"/>


    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class=" com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>

    </beans>

In the regular way, we used <property> tags to manually inject the Address, EmployeeSalary, and Vehicle beans using ref.

Now with autowire="byName", Spring will handle this for us automatically by checking:

    Which dependencies are already injected (id, name, age) via <property>
    Which ones are leftover (address, salary, vehicle)
    Then it looks at the bean IDs in the XML — since we named them the same as the field names, Spring injects them directly using setters.

No need to manually declare the <property name="..." ref="..." /> lines for those!
           
        //output 
          
    BhanuPrasad
    Dallas
    Car Model is Hrv - Sport
    ----------------- 

    Employee details - ID : 1  name : BhanuPrasad age : 24 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Honda  Hrv - Sport  2023

    Process finished with exit code 0


### Spring XML Autowiring – `autowire="byType"`

When you use **`autowire="byType"`**, Spring will:

Look at the **type of the property** in your class (like `Address`, `EmployeeSalary`, `Vehicle`)  
Then inject the matching bean by **class type**, not by bean ID

---

### Key Points:

- It **still requires** a **default constructor**
- It needs **setter methods** in your class (since it's setter-based injection)
- There must be **only one matching bean per type** — otherwise Spring will throw an error

---

so,
When we use `autowire="byType"`, Spring checks:
- What are the **leftover fields** that haven’t been injected manually
- Then it looks into the XML config to see if there are any beans where the **class type matches the property type** in your class

If it finds **exactly one matching bean**, it will automatically inject it  
If multiple beans of the same type exist, Spring won’t know which one to pick and will throw an error


    //EMployee.XML file

    <bean id="emp" class="com.myproject.java_spring.Employee" autowire="byType">
       <property name="id" value="1"/>
        <property name="name" value="BhanuPrasad"/>
        <property name="age" value="24"/>


    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class=" com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>


            //output 
          
    BhanuPrasad
    Dallas
    Car Model is Hrv - Sport
    ----------------- 

    Employee details - ID : 1  name : BhanuPrasad age : 24 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Honda  Hrv - Sport  2023

    Process finished with exit code 0

### `autowire="constructor"` in Spring XML

To use **constructor-based autowiring**, there should be a **constructor** in the class that accepts all the dependencies as parameters.

Now, in the XML file, we use the `autowire="constructor"` attribute inside the `<bean>` tag.

When using this, **Spring will try to inject all constructor parameters automatically by type**.  
So you **don't need to specify `<constructor-arg>` manually** — but only **if all parameters are object types and their beans are defined**.


###  Important Note:
If your constructor includes **primitive types** like `int`, `String`, etc., then **Spring cannot autowire them** by type — because those are not beans.

So in that case, even with `autowire="constructor"`, you must manually provide those values using `<constructor-arg>` for primitive values, and Spring will autowire the rest (object types like `Address`, `Vehicle`, etc.)


### Best Practice:

If you want to use a constructor with both primitive and object dependencies, the **recommended way** is:


Employee class

    //employee.java
    
        package com.myproject.java_spring;

    public class Employee {
    private int id;
    private String name;
    private int age;
    private Address address;
    private EmployeeSalary salary;
    private Vehicle vehicle;

    public Employee(int id, String name, int age, Address address, EmployeeSalary salary, Vehicle vehicle) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.address = address;
        this.salary = salary;
        this.vehicle = vehicle;
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

Employee.xml 

        //Employee.xml 


    <bean id="emp" class="com.myproject.java_spring.Employee" autowire="constructor">
      <constructor-arg value="1"/>
        <constructor-arg value="Bhanu"/>
        <constructor-arg value="25"/>
        <constructor-arg ref="address"/>
        <constructor-arg ref="salary"/>
        <constructor-arg ref="vehicle"/>

    </bean>

    <bean id="address" class="com.myproject.java_spring.Address">
        <property name="state" value="Tx"/>
        <property name="city" value="Dallas"/>
        <property name="zipCode" value="22331"/>
    </bean>

    <bean id="salary" class=" com.myproject.java_spring.EmployeeSalary">
        <constructor-arg value="10000"/>
    </bean>

    <bean id="vehicle" class="com.myproject.java_spring.Vehicle">
        <property name="make" value="Honda"/>
        <property name="model" value="Hrv - Sport"/>
        <property name="year"   value="2023"/>
    </bean>
  
output

      //output 
          
    BhanuPrasad
    Dallas
    Car Model is Hrv - Sport
    ----------------- 

    Employee details - ID : 1  name : BhanuPrasad age : 24 address :   Dallas  Tx  22331 salary : 10000.0 Vehicle : Honda  Hrv - Sport  2023

    Process finished with exit code 0



