Spring has an interface called ApplicationContext which contains different abstract methods. As we know, this ApplicationContext is used to load the Spring container.

Now, there is a built-in class called ClassPathXmlApplicationContext. This class takes a string value as an input parameter (which is the XML file ).
 Basically, this class has built-in logic to create the beans and inject their dependencies.
 This class was developed by Spring developers to make our work easier.

So, we usually have an .xml file (example: spring.xml) that contains the bean definitions and their dependencies.

When we create an object of ClassPathXmlApplicationContext, it's like we are creating (or loading) the Spring container.
Since we are passing the .xml file to this class, by the time the object of ClassPathXmlApplicationContext is created, all the beans (objects) and their dependencies are already created and injected by Spring automatically.

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
