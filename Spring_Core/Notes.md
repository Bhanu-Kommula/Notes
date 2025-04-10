                                                            ** Spring Core Module **

Spring: Spring is a lightweight, open-source Java framework designed to simplify the development of web and enterprise applications. It provides good support for building Java applications, mainly focusing on features like dependency injection, aspect-oriented programming, and easy integration with other technologies.

Spring core module is all about IOC (Inversion of Control) and Dependency Injection.

So, to simply define — in Java, normally we create objects and inject the dependencies into them manually. But here, in Spring Core Module, we can ask Spring to take care of this — it will create the object and inject the required dependencies into it.

Spring will have a container (called the Spring Container) which is created when we call the ApplicationContext. So when we call the ApplicationContext, it creates the Spring Container. During this container creation, the objects (beans) and their dependencies will automatically get injected by Spring. So this container will have all the objects along with their injected dependencies.

IOC means object creation is handled by Spring, and DI (Dependency Injection) is all about giving required dependencies to the object.
Here, dependency means – every object has some fields right? So those fields are called dependencies. Giving values or objects to those fields is called injecting.

    
