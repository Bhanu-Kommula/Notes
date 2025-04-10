Spring Core - XML-Based Configuration

Welcome to the XML-Based Configuration section of the Spring Core notes! This section contains beginner-friendly, hands-on explanations and examples for using Spring's XML configuration style — perfect for building a solid foundation before jumping into annotations or Java-based config.

 Topics Covered

 1. Basic XML Bean Configuration

Creating and using <bean> tags

Constructor-based injection with <constructor-arg>

Setter-based injection with <property>

Injecting primitive and object dependencies into beans

 2. Autowiring with XML

autowire="byName"

Injects beans by matching field name with bean ID

Requires setter methods and default constructor

autowire="byType"

Injects beans by class type

Only works when one bean of that type exists

autowire="constructor"

Spring matches constructor parameters by type and order

Manual values required for primitives, rest autowired by type

 3. Annotation-Based Injection in XML Projects

Using @Autowired with <context:annotation-config/>

How Spring injects by type

Why field name matters when multiple beans of the same type exist

Using @Qualifier with @Autowired to resolve conflicts

 4. Bean Scopes

scope="singleton" (default)

Bean is created once and shared

scope="prototype"

Bean is created every time it is requested

Practical demo using getBean() to compare both scopes

 5. Bonus Concepts

Do you need a default constructor? (depends on injection type)

Field vs Setter vs Constructor injection explained

Spring’s behavior with duplicate bean types and no @Qualifier
