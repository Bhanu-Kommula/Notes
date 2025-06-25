Spring cloud: Key Microservice solutions

      Centralized Configuration
      Distributed Tracing
      Load Balancing
      Service Discovery
      Edge Server
      Fault Tolerance


Spring cloud Config Serve : This will helps us to implement Centralised Configurations here we will have the configuration relaterd to all the microservices and env in one git repo. 


Dependencies --> 

        Spring Web for building REST and web applications
        Spring Actuator for monitoring and managing the application
        Config client - this helps the microservice to talk to the Spring Cloud config server


Now lets create a limits microservice and this will talk to the config server. 

In application.properties

        Spring.config.import = optional:configserver:http://localhost:8888    - > this is the default of cloud config server 

 

Now let's write a limits class 
                  
                  package com.myprojects.microservices.model;
                  
                  import lombok.AllArgsConstructor;
                  import lombok.Data;
                  import lombok.NoArgsConstructor;
                  
                  @Data
                  @AllArgsConstructor
                  @NoArgsConstructor
                  public class Limits {
                  	
                  	private int minimum;
                  	private int maximum;
                  
                                    }

Limits controller 
                  
            package com.myprojects.microservices.controller;
                  
                  import org.springframework.web.bind.annotation.GetMapping;
                  import org.springframework.web.bind.annotation.RestController;
                  
                  import com.myprojects.microservices.model.Limits;
                  
                  @RestController
                  public class LimitsController {
                  
                  	
                  	@GetMapping("/limits")
                  	public Limits getLimits() {
                  		
                  		return new Limits(1,1000);
                  		
                  
                  		
                  	}
                  }
                  


So here we need two values for the  minimum and maximum so we are hardcoding the values. but if we want this values to come from the configuration file then 

in Application.properties 


            spring.application.name=limits-service
            spring.config.import=optional:configserver:http://localhost:8888

            limitsservice.minimum=3
            limitsservice.maximum=997

Here we are setting the values in the application.properties file. and limitsservice ( this can be any name of our choice) .minimum( the field names in the limit class), so for example it can be abc.minimum =3 and abc.maximum=997 or bhanu.minimum=4. 


After declaring the values, to make the spring pick up values from application properties and return to the controller. spring makes it easy to pick up the values this can be done by creating a new class called Configuration  class with @ConfigurationProperties("limitsserive"), here the name as to be the same as in the application.properties file so that I can pick the min and max values from the application.properties. 


                  package com.myprojects.microservices.configuration;
            
            import org.springframework.boot.context.properties.ConfigurationProperties;
            import org.springframework.stereotype.Component;
            
            import lombok.Data;
            
            @Data
            @Component
            @ConfigurationProperties("limitsservice")  
            public class Configuration {
            	
            	private int maximum;
            	private int minimum;
            
            }

limits controller 

            package com.myprojects.microservices.controller;
            
            import org.springframework.beans.factory.annotation.Autowired;
            import org.springframework.web.bind.annotation.GetMapping;
            import org.springframework.web.bind.annotation.RestController;
            
            import com.myprojects.microservices.configuration.Configuration;
            import com.myprojects.microservices.model.Limits;
            
            @RestController
            public class LimitsController {
            
            	@Autowired
            	private Configuration config;
            	
            	@GetMapping("/limits")
            	public Limits getLimits() {
            		
            		
            		return new Limits(config.getMinimum(), config.getMaximum());
            
            		
            	}
            }

**Standardization of Ports**


1. Limits Microservice
Ports: 8080, 8081, etc.

2. Spring Cloud Config Server
Port: 8888

3. Currency Exchange Microservice
Ports: 8000, 8001, 8002, etc.

4. Currency Conversion Microservice
Ports: 8100, 8101, 8102, etc.

5. Netflix Eureka Naming Server
Port: 8761

6. API Gateway
Port: 8765

7. Zipkin Distributed Tracing Server
Port: 9411

Adhering to these standards ensures smooth interaction and avoids conflicts in a distributed system.





So now we have the basic limits service ready, let's build the Spring Cloud Config server 

Create a new project using spring boot starter 
dependencies 

           Spring Dev Tools
           Config Server


Application.properties -- change the port num, the standardized port number for spring cloud config server is 8888

            spring.application.name=spring-cloud-config-server
            server.port=8888


So now we have use the @ to make this to make it work like a real config server 





Now as we have limits service and also a config server up running let's now create the Git Repo

