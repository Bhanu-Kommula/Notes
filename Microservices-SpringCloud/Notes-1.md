Spring cloud: Key Microservice solutions

      Centralized Configuration
      Distributed Tracing
      Load Balancing
      Service Discovery
      Edge Server
      Fault Tolerance


Spring cloud Config Serve : This will helps us to implement Centralised Configurations here we will have the configuration relaterd to all the microservices and env in one git repo. 


Dependencies --> 

        Spring web for building rest and web applications
        Spring Actuator for monitoring and managing the application
        Config client - this helps the microservice to talk to Spring cloud config server


Now lets create a limits microservices and this will talk to the config server. 

In application.properties

        Spring.config.import = optional:configserver:http://localhost:8888    - > this is the default of cloud config server 

 
