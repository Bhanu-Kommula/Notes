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


    //SpringDemos.java(main class)
    
