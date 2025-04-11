###  Spring JDBC Template — Simplifying Database Access

Spring provides a built-in utility called **JdbcTemplate**, which helps us **interact with the database** without writing the full JDBC boilerplate code ourselves.

 Spring takes care of:
- Establishing connection
- Managing statements
- Closing resources

---

- `JdbcTemplate` is a **class provided by Spring** under the `org.springframework.jdbc.core` package.
- It provides **methods to query and update data** in a relational database.
- We just call its methods with the SQL and let Spring handle the backend logic.

---

###  Commonly Used Methods:

    1. query(String sql, RowMapper<T> rowMapper, Object... args)

Used to fetch data from the database
Requires at least:
SQL query
RowMapper (to map rows to Java objects)
Optionally accepts arguments for the query (Object... varargs)

    2. update(String sql)
Used for insert, update, delete operations
Takes just the SQL string as input

To use it, we must provide a DataSource — which tells Spring:

    Where is the DB? What are the credentials? Which driver to use?
    DataSource is an interface
    DriverManagerDataSource is the implementation we usually use in simple cases

Steps:

Create a DataSource object (DriverManagerDataSource)
Set DB URL, username, password, and driver class
Create a JdbcTemplate object
Inject the datasource into it
Call query() or update() methods to interact with DB


Make sure to have OJDB.jar file in class path of the project. 

So for Update()



Student class

    package com.myproject.java_spring.springJdbc;


    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Lazy;

    public class Student {
    private int id;
    private String name;
    private int age;
    @Autowired
    private Marks marks;

    @Autowired
    private StudentDAO studentDAO;



    public Student(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public StudentDAO getStudentDAO() {
        return studentDAO;
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

    public Marks getMarks() {
        return marks;
    }
    }

Marks class


    package com.myproject.java_spring.springJdbc;


    public class Marks {
    private int mathsMarks;
    private int chemistryMarks;
    private int physicsMarks;

    public int getMathsMarks() {
        return mathsMarks;
    }

    public void setMathsMarks(int mathsMarks) {
        this.mathsMarks = mathsMarks;
    }

    public int getChemistryMarks() {
        return chemistryMarks;
    }

    public void setChemistryMarks(int chemistryMarks) {
        this.chemistryMarks = chemistryMarks;
    }

    public int getPhysicsMarks() {
        return physicsMarks;
    }

    public void setPhysicsMarks(int physicsMarks) {
        this.physicsMarks = physicsMarks;
    }

    public int totalMarks() {
        return mathsMarks + chemistryMarks + physicsMarks;
    }
    }


Student DAO class

    package com.myproject.java_spring.springJdbc;

    import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.stereotype.Repository;

    @Repository
    public class StudentDAO {

    @Autowired
    private Student student;
    @Autowired
    private Marks marks;


    public Student getStudent() {
        return student;
    }

    public Marks getMarks() {
        return marks;
    }

    @Autowired
    private JdbcTemplate template;

    public void createTable(){
        String query = " create table StudentInfo ( StudentID number(4), StudentName varchar2(19), StudentAge number(3), Maths number(3), Physics number(3),Chemistry number(3))";

        template.update(query);
    }

    public void insertTable(){

    String query = " insert into StudentInfo values(?,?,?,?,?,?)";
      template.update(query,student.getId(),student.getName(),student.getAge(),student.getMarks().getMathsMarks(), student.getMarks().getPhysicsMarks(),student.getMarks().getChemistryMarks() );

    }

    public void deleteTable(int id){
        String query = " delete from StudentInfo where StudentID=?";
        template.update(query,student.getId());
    }

    }

Main class

    package com.myproject.java_spring.springJdbc;

    import org.springframework.context.ApplicationContext;
      import org.springframework.context.annotation.AnnotationConfigApplicationContext;

      public class SpringJdbcDemos {
    public static void main(String[] args) {


        ApplicationContext context = new AnnotationConfigApplicationContext(StudentMarksConfig.class);

        Student student = (Student) context.getBean("student");

    student.getStudentDAO().insertTable();
  
    }
    }



query()


