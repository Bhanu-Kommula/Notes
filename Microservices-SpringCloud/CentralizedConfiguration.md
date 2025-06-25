Spring cloud: Key Microservice solutions

      Centralized Configuration
      Distributed Tracing
      Load Balancing
      Service Discovery
      Edge Server
      Fault Tolerance


Spring cloud Config Serve : This will helps us to implement Centralised Configurations here we will have the configuration relaterd to all the microservices and env in one git repo. 


Lets Build **Centralized Configuration**

Dependencies --> 

        Spring Web for building REST and web applications
        Spring Actuator for monitoring and managing the application
        Config client - this helps the microservice to talk to the Spring Cloud config server

<img width="1236" alt="image" src="https://github.com/user-attachments/assets/0634cbd6-09d2-4e58-9829-cda020238de0" />

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


Application.properties -- change the port number,  as our limit service is also running on port 8080, we are changing the port number and the standardized port number for spring cloud config server is 8888

            spring.application.name=spring-cloud-config-server
            server.port=8888


So now we have to use the @EnableConfigServer to make to make it work like a real config server 

            
            package com.myprojects.microservices;
            
            import org.springframework.boot.SpringApplication;
            import org.springframework.boot.autoconfigure.SpringBootApplication;
            import org.springframework.cloud.config.server.EnableConfigServer;
            
            @EnableConfigServer
            @SpringBootApplication
            public class SpringCloudConfigServerApplication {
            
            	public static void main(String[] args) {
            		SpringApplication.run(SpringCloudConfigServerApplication.class, args);
            	}
            
            }



Now that we have limits service and also a config server up running, let's create the Git Repo


Git - used as a version control system, so go to git-scm.com and download the git client

 so, after downloading the git go to the dir where you want to have the git repo and then create a new dir using mkdir git-localconfig-repo


git commands used

      git --version - to check the version 

      pwd - present working directory to know where you are ( which directory you are in currently)
      cd -- change directory - used to go the directory you want like ex - cd Documents/BhanuProject/ this will take me to bhanuproject dir 

      mkdir - to create a new directory 
      
       /Users/bhanuprasadkommula/Documents/BhanuProject/git-localconfig-repo/

            git init - this will initialize an empty repository so that we can make use of this dir to store all our configuration files in this dir ( git-localconfig-repo)

Now open a text editor (vscode) and navigate to that folder and create a new file name it as limits-service.properties,  so let's store limit-service properties in this repository. Now copy the limitservice properties from application.properties of limitservice. 


our current limit service - application.properties has 

                              spring.application.name=limits-service
            spring.config.import=optional:configserver:http://localhost:8888
            
            limitsservice.minimum=3
            limitsservice.maximum=997

            
Move the properties of limitservice to limits-services.properties
limits-services.properties

                        
            limitsservice.minimum=3
            limitsservice.maximum=997


So now we have the limits-service.properties ready to commit to the repository.

                  ls  - gives the list of all the files in the directory
                  git add limits-services.properties   or git add * - to add all the files 
                  git commit -m "adding the limit-service properties" 


So now we have limit service, a config server, and also a repo and now lets connect them together. 


first lets connect spring cloud config server to git repo

To do that, go to the application.properties of spring-cloud-config-server  and configure the folder 

            
            spring.application.name=spring-cloud-config-server
            server.port=8888
            
            spring.cloud.config.server.git.uri= file:///Users/bhanuprasadkommula/Documents/BhanuProject/git-localconfig-repo

            
This connects the config server with repo to check open 

            http://localhost:8888/limits-service/default    will give the limits values from repo 



             
Now let's connect our limit service to the Spring Cloud Config server 

So to connect the limit service to the Spring Cloud Config server, we need two main configurations, which we already made 

            1. spring-cloud-started-config ( starter.io --  config client) This dependency will help to talk with config server

      2. In application.properties of limit service 

       spring.config.import=optional:configserver:http://localhost:8888   or        spring.config.import=configserver:http://localhost:8888   - we are saying to import the config server from http://localhost:8888. 

       and also we need to say what configuration in the config server should this application make use of. so in our case we want the application to make use of limits-service.propeties file

       
       spring. application.name=limits-service

            
so file looks like 
            
            spring.application.name=limits-service
            spring.config.import=optional:configserver:http://localhost:8888
            


Now let's see how we store separate configuration for limit service ( like qa, dev,) and how do we make use of it Spring Cloud Config Server. 


so create multiple files in git repo (git-localconfig-repo) 

            create multiple files like 
                  limits-service-dev.properties
                  limits-service-qa.properties

                  and add the diff limits accordingly. 

so to view the limits of diff profiles/environments, use 

      http://localhost:8888/limits-service/qa
      http://localhost:8888/limits-service/dev


so now we have values for multiple env picked up from git repo but how to make the application to pick these from cloud config server 

to do configure in the application.properties of limit service

                        spring.application.name=limits-service
            spring.config.import=optional:configserver:http://localhost:8888
            
            spring.profiles.active=dev
            spring.cloud.config.profile=dev

| Property                          | Used by             | Purpose                              |
| --------------------------------- | ------------------- | ------------------------------------ |
| `spring.profiles.active=dev`      | Spring Boot         | Activates the `dev` profile in app   |
| `spring.cloud.config.profile=dev` | Spring Cloud Client | Fetches the `dev` config from server |


spring.cloud.config.profile=dev → tells the Config Client what profile to fetch from the Config Server

spring.profiles.active=dev → activates that profile inside the application after it's fetched
