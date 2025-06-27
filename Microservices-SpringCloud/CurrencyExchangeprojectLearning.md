Here, lets implement two things 
      1: currency exchange service which say what is the exchange rate
      2: Currency Conversion service which converts the given amount. 

Lets create a basic currency exchange service 

Dependencies 

    1: Lombok
    2: Spring Dev Tools
    3: Config Client
    4: Spring Web

Application.properties

            spring.application.name=currency-exchange
            server.port=8000
            
            spring.config.import:optional:configserver:http://localhost:8888

 Controller
 
            package com.myprojects.microservices.controller;
            
            import org.springframework.web.bind.annotation.GetMapping;
            import org.springframework.web.bind.annotation.PathVariable;
            import org.springframework.web.bind.annotation.RestController;
            
            import com.myprojects.microservices.model.CurrencyExchange;
            
            @RestController
            public class CurrencyExchangeController {
            
            	
            	
            	@GetMapping("/currency-exchange/from/{from}/to/{to}")
            	public CurrencyExchange getExchnageValue(@PathVariable String from, @PathVariable String to) {
            		 
            		return new CurrencyExchange(1l,from,to, 65.00);	
            	}
            	
            }
model
            
            package com.myprojects.microservices.model;
            
            
            import lombok.AllArgsConstructor;
            import lombok.Data;
            import lombok.NoArgsConstructor;
            
            @Data
            @AllArgsConstructor
            @NoArgsConstructor
            public class CurrencyExchange {
            	
            	private Long id;
            	private String from;
            	private String to;
            	private double conversionMultiple;
            
            }

Output:

            URL : http://localhost:8000/currency-exchange/from/INR/to/USD

            op

                        {
              "id": 1,
              "from": "INR",
              "to": "USD",
              "conversionMultiple": 65
            }
Here we are just hardcoding the values.


To get the port number that an instance or service is running, Spring has a built-in class called Environment ( import org.springframework.core.env.Environment;)


later in the course we will learn about load balancing, so in our case when a currency conversion application is called then it will internally communicate with the currency exchange service to know the conversion rate. So this conversion rate is spread across multiple instance to make sure even distribution of traffic and this is called load balancing. 

so to store this instance name lets have a new variable called environment.  For now let this give only the port number of the service running. 


Model
            package com.myprojects.microservices.model;
            
            
            import lombok.AllArgsConstructor;
            import lombok.Data;
            import lombok.NoArgsConstructor;
            
            @Data
            @AllArgsConstructor
            @NoArgsConstructor
            public class CurrencyExchange {
            	
            	private Long id;
            	private String from;
            	private String to;
            	private double conversionMultiple;
            	private String environment;  // in this variable we store which instance is giving response, so while working on load balancing this will 
            	//which instance is giving response
            	
            	
            	public CurrencyExchange(Long id, String from, String to, double conversionMultiple) {
            		super();
            		this.id = id;
            		this.from = from;
            		this.to = to;
            		this.conversionMultiple = conversionMultiple;
            	}
            
            	
            	
            	
	
                  }


So added a new variable and also added a constructor without env.

            package com.myprojects.microservices.controller;
            
            import org.springframework.beans.factory.annotation.Autowired;
            import org.springframework.core.env.Environment;
            import org.springframework.web.bind.annotation.GetMapping;
            import org.springframework.web.bind.annotation.PathVariable;
            import org.springframework.web.bind.annotation.RestController;
            
            import com.myprojects.microservices.model.CurrencyExchange;
            
            @RestController
            public class CurrencyExchangeController {
            
            	@Autowired
            	private Environment env;
            	
            	
            	
            	@GetMapping("/currency-exchange/from/{from}/to/{to}")
            	public CurrencyExchange getExchnageValue(@PathVariable String from, @PathVariable String to) {
            		 
            		
            		String port = env.getProperty("local.server.port");
            		
            		CurrencyExchange currencyExchange  = new CurrencyExchange(1l,from,to, 65.00);	
            		currencyExchange.setEnvironment(port);
            		return currencyExchange;
            
            		
            		
            	}
            	
            }

Using the Environment spring class to get the port by using env.getProperty("local.server.port");


output 


					{
		  "id": 1,
		  "from": "INR",
		  "to": "USD",
		  "conversionMultiple": 65,
		  "environment": "8000"
		}


Now lets say I want to run 2 instance of this application running to do that 

	right click and select run configuration ---> add 8000 to name of current application to show differnece (currency-exchange-service - CurrencyExchangeServiceApplication8000)
 	now duplicate the application ( left handside right click on the currency exchange and select duplicate 

  	now rename this duplicate as 8001 (currency-exchange-service - CurrencyExchangeServiceApplication8001) and to make this to run in 8001 port click on arguments and write this command -Dserver.port=8001

		run this --- http://localhost:8001/currency-exchange/from/INR/to/USD  will get the same output with diff port no but from differnet port so for the same application we have 2 instances now 

		{
		  "id": 1,
		  "from": "INR",
		  "to": "USD",
		  "conversionMultiple": 65,
		  "environment": "8001"
		}



  

So now, as we have hardcoded the restapi now let's make it fetch from DB and let's use in in-memory database called H2, and lets use JPA to talk to DB

So add dependencies

	Spring-boot-starter-data-jpa
 	h2
After adding these dependencies, creates a db at a random location, so lets not use a random db url, so let's configure one 

Application.properties 

   	spring.jpa.show-sql=true
	spring.datasource.url= jdbc:h2:mem:testdb
	spring.h2.console.enabled=true

	http://localhost:8000/h2-console  url to access h2 console

now in login page change the url to the one mentioned in application.properties

		url = jdbc:h2:mem:testdb
  click connect to h2 console.

  Now as we have the db ready lets make the model class as @Entity to create the db 

  if run this it will throw an error

	Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "create table currency_exchange (conversion_multiple float(53) not null, id bigint not null, environment varchar(255), [*]from varchar(255), to varchar(255), primary key (id))"; expected "identifier"; SQL statement:
	create table currency_exchange (conversion_multiple float(53) not null, id bigint not null, environment varchar(255), from varchar(255), to varchar(255), primary key (id)) [42001-232]

Because we are using from and to as variables in our model class but these are used as keywords in DB so keywords cant be as column names so lets keep a different column names in our DB, to do that we use @Column(name=currency_from) 

			package com.myprojects.microservices.model;
		
		
		import jakarta.persistence.Column;
		import jakarta.persistence.Entity;
		import jakarta.persistence.Id;
		import lombok.AllArgsConstructor;
		import lombok.Data;
		import lombok.NoArgsConstructor;
		
		@Data
		@AllArgsConstructor
		@NoArgsConstructor
		@Entity
		public class CurrencyExchange {
			
			@Id
			private Long id;
			@Column(name="currency_from")
			private String from;
			@Column(name="currency_to")
			private String to;
			private double conversionMultiple;
			private String environment;  //In this variable we store which instance is giving response, so while working on load balancing this will 
			//which instance is giving the response
			
			
			public CurrencyExchange(Long id, String from, String to, double conversionMultiple) {
				super();
				this.id = id;
				this.from = from;
				this.to = to;
				this.conversionMultiple = conversionMultiple;
			}	
		}


Now as we have table ready lets insert some values, lets create a text file and write insert querry

data.sql

  		insert into currency_exchange
		(id,currency_from,currency_to,conversionMultiple,environment)
		values(101,'USD','INR',80,'');
		
		insert into currency_exchange
		(id,currency_from,currency_to,conversionMultiple,environment)
		values(102,'AUD','INR',30,'');
		
		insert into currency_exchange
		(id,currency_from,currency_to,conversionMultiple,environment)
		values(103,'EUR','INR',90,'');
		
		// Dont give values like this values will inserting randomly into diff columns and throwing error 
		insert into currency_exchange
		values(104,'USD','INR',80,'');
		

So save this file in src/main/resource folder and name should be data.sql so whenever we start the application, the data from this file will be loaded  into the db.

so as we now have data.sql, we want this file to be loaded after tables are created but
In the latest version of Spring Boot (2.4+), the load of data sql is done before tables creation to avoid that we have to make a configuration to deffer the execution of data.sql

application.properties

		spring.jpa.defer-datasource-initialization=true

  
Now as we have data in DB lets use the rest api to fetch the data


Let's create a repository interface and create a cusotm method to get the record based on from and to , as we know jpa writes the querry for us if follow the convention correctly, findByFromAndTo 

		
		package com.myprojects.microservices.repository;
		
		import org.springframework.data.jpa.repository.JpaRepository;
		
		import com.myprojects.microservices.model.CurrencyExchange;
		
		public interface CurrencyExchangeRepository extends JpaRepository<CurrencyExchange, Long> {
			CurrencyExchange findByFromAndTo(String from, String to);

		}


Now that we have the method ready, let's call the api
		
		package com.myprojects.microservices.controller;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.core.env.Environment;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RestController;
	
	import com.myprojects.microservices.model.CurrencyExchange;
	import com.myprojects.microservices.repository.CurrencyExchangeRepository;
	
	@RestController
	public class CurrencyExchangeController  {
	
		@Autowired
		private Environment env;
		
		@Autowired
		private CurrencyExchangeRepository repo;
		
		@GetMapping("/currency-exchange/from/{from}/to/{to}")
		public CurrencyExchange getExchnageValue(@PathVariable String from, @PathVariable String to) {
			 
			
			String port = env.getProperty("local.server.port");
			
			CurrencyExchange currencyExchange  = repo.findByFromAndTo(from, to);	
			
			if(currencyExchange == null) {
				throw new RuntimeException ("No Records Found");
			}
			
			currencyExchange.setEnvironment(port);
	
			
			return currencyExchange;
	
			
			
		}
		
	}


Now, as we have our currency exchange working, let's build our currency conversion  service and let the conversion service talk to the exchange service and get the conversion rate.

Dependencies - 

		lombok
  		spring deve tools
    		Sprinf web
    		config client
      		acturator 
		

application 
		
		spring.application.name=currency-conversion
		server.port=8100
		
		spring.config.import=optional:configserver:http://localhost:8888
		

model


			package com.myprojects.microservices.model;
		
		import lombok.AllArgsConstructor;
		import lombok.Data;
		import lombok.NoArgsConstructor;
		
		@Data
		@AllArgsConstructor
		@NoArgsConstructor
		public class CurrencyConversion {
			
			
			
			private Long id;
			//@Column(name="currency_from")
			private String from;
			//@Column(name="currency_to")
			private String to;
			private double quantity;
			private double conversionMultiple;
			private double totalCalculateAmount;
			private String environment; 
			
			
			
		
		}


controller


			package com.myprojects.microservices.controller;

		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.core.env.Environment;
		import org.springframework.web.bind.annotation.GetMapping;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RestController;
		
		import com.myprojects.microservices.model.CurrencyConversion;
		
		@RestController
		public class CurrencyConversionController {
			
			@Autowired
			private Environment env;
			
			@GetMapping("currency-conversion/from/{from}/to/{to}/of/{of}")
			public CurrencyConversion convertedvalue(@PathVariable String from, @PathVariable String to, @PathVariable double of) {
				
				
				String port = env.getProperty("local.server.port");
				
				
				return new CurrencyConversion (101L,from,to,of, 65, of*65, port );
				
			}
		
		}


output of these hardcoded values 

	http://localhost:8100/currency-conversion/from/USD/to/INR/of/20

 output

	 	{
	  "id": 101,
	  "from": "USD",
	  "to": "INR",
	  "quantity": 20,
	  "conversionMultiple": 65,
	  "totalCalculateAmount": 1300,
	  "environment": "8100"
	}
		



****Feign Client****

Now let's talk to currency exchange from currency conversion, and then get the values 

So, for the communication over the rest btw different services, to avoid boilerplate code and resttemplate or webclient, we can use Feign Client for the communication.

FEIGN Client is a declarative HTTP client developed by Netflix and used in Spring cloud.

Add the OpenFeign dependency 


Now to establish a proxy, so lets create a new interface 

	package com.myprojects.microservices.proxy;
	
	import org.springframework.cloud.openfeign.FeignClient;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	
	import com.myprojects.microservices.model.CurrencyConversion;
	
	@FeignClient(name = "currency-exchange", url = "localhost:8000") // in name we will keep the application name that we want to call and in url
	//we say the location of that application running
	
	public interface CurrencyExchnageProxy {
	
		@GetMapping("/currency-exchange/from/{from}/to/{to}")
		public CurrencyConversion getExchnageValue(@PathVariable String from, @PathVariable String to) ;
			 
			
			
	}

We have to enable the feign client in main app


		package com.myprojects.microservices;
		
		import org.springframework.boot.SpringApplication;
		import org.springframework.boot.autoconfigure.SpringBootApplication;
		import org.springframework.cloud.openfeign.EnableFeignClients;
		
		@SpringBootApplication
		@EnableFeignClients
		public class CurrencyConversionServiceApplication {
		
			public static void main(String[] args) {
				SpringApplication.run(CurrencyConversionServiceApplication.class, args);
			}
		
		}
		

Controller

		package com.myprojects.microservices.controller;
		
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.web.bind.annotation.GetMapping;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RestController;
		
		import com.myprojects.microservices.model.CurrencyConversion;
		import com.myprojects.microservices.proxy.CurrencyExchnageProxy;
		
		@RestController
		public class CurrencyConversionController {
			
			
			@Autowired
			private CurrencyExchnageProxy proxy;
			
			
			@GetMapping("currency-conversion/from/{from}/to/{to}/of/{of}")
			public CurrencyConversion convertedvalue(@PathVariable String from, @PathVariable String to, @PathVariable double of) {
				
				
				
				
				return new CurrencyConversion (101L,from,to,of, 65, of*65, "" );
				
			}
			
			
			@GetMapping("currency-conversion-feign/from/{from}/to/{to}/of/{of}")
			public CurrencyConversion convertedvalueFeign(@PathVariable String from, @PathVariable String to, @PathVariable double of) {
				
		 CurrencyConversion currencyConversion = proxy.getExchnageValue(from, to);
				
				
				return new CurrencyConversion(currencyConversion.getId(),
						from, to, of,
						currencyConversion.getConversionMultiple(),
						of*(currencyConversion.getConversionMultiple()),
						currencyConversion.getEnvironment());
			
			}	
		}



So now as we have the chain working correctly,  in currency exchange proxy we are hard coding the url of currency exchange service.  lets say if we want to have multiple instances of this application, and how to see which is down and how to make sure load is shared among them equally ( load balancing) that's where go for service registry or naming server

****Service Registry****

So in microservices architecture, all the instances of all the microservices would register with a service registry. So the currency conversion microswervice , currency exchange microservice, and other microservices would register with this naming server called the service registry.

So let's say the Currency conversion service would like to talk to the currency exchange service, then it would ask the service registry what are the addresses of the currency exchange then the service registry would return those back to the currency conversion then they can establish the communication. 


So simply all the instances would register with the naming server/ service registry, and whenever ever currency conversion microservice want to find out the active instances then it asks the naming server to get the instances and load balances btw them.



Now lets start creating with Naming server and Spring provides is Eureka

New starter project 

dependencies 

 	Spring web tools
  	Eureka server
   	Spring Actuator 


So we have to enable the @EnableEurekaServer in the main app

		package com.myprojects.microservices;
		
		import org.springframework.boot.SpringApplication;
		import org.springframework.boot.autoconfigure.SpringBootApplication;
		import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
		
		@SpringBootApplication
		@EnableEurekaServer
		public class NamingServerApplication {
		
			public static void main(String[] args) {
				SpringApplication.run(NamingServerApplication.class, args);
			}
		
		}

		
Now we have to make the configuration in application.properties 

port to 8761
And there are also a couple of other configurations which are recommended by Eureka, We are creating the Eureka server, so we don't want to register with itself To do that, we have to add a couple of properties 
		
		
		spring.application.name=naming-server
		server.port=8761
		
		eureka.client.register-with-eureka=false
		eureka.client.fetch-registry=false


So now we have our naming server, so let's connect our microservices to the naming server. So let's connect currency conversion and exchange service to connect with the naming server, and then conversion talks to exchange via the naming server 



To do that, add a dependency, Eureka Discovery Client, in both the services, and then refresh the Eureka console 

		http://localhost:8761/

then you will see both the services registered with the Eureka server, but to be really safe, lets configure the url in the application.proerties 
 
Currency conversion application.properties  just add eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka


		spring.application.name=currency-exchange
		server.port=8000
		
		spring.config.import:optional:configserver:http://localhost:8888
		
		spring.jpa.show-sql=true
		spring.datasource.url= jdbc:h2:mem:testdb
		spring.h2.console.enabled=true
		
		spring.jpa.defer-datasource-initialization=true
		
		eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka		


Now let's work on load balancing btw multiple instances of currency exchange and currency conversion 


just remove the url from the proxy url and add 	@FeignClient(name = "currency-exchange")


	package com.myprojects.microservices.proxy;
	
	import org.springframework.cloud.openfeign.FeignClient;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	
	import com.myprojects.microservices.model.CurrencyConversion;
	
	//@FeignClient(name = "currency-exchange", url = "localhost:8000") // in name we will keep the application name that we want to call and in url
	//we say the location of that application running
	@FeignClient(name = "currency-exchange")
	public interface CurrencyExchnageProxy {
	
		@GetMapping("/currency-exchange/from/{from}/to/{to}")
		public CurrencyConversion getExchnageValue(@PathVariable String from, @PathVariable String to) ;
			 
			
			
	}



So inside the currency conversion service there is a load balancer talking to naming server finding the instances and doing automatic load balancing btw them and this is called ****Client side Load Balancing**** this is happening throught **Feign**

In the earlier versions of spring cloud, the load balance which was used was Ribbon, in the recent version spring cloud shifted to using spring cloud load balancer as the load balancer. if you are using **Eureka** and **Feign** then loadbalancing comes for free this is client side load balanacing. 



 Now One of the most imp challenges when it comes to microservices is that lots of these microservices will have common features, lets say i want to implment rate limiting or authentication and authorisation which is same fro all the services then why to write for each service instead we can write at one palce and use it for the services, then the concept of **API gateway** comes into picture. So before calling any service the req route through the API Gateway. lets say there is a request coming to conversion it would route throguht the api gateway. lets say if we want to call exchange from conversion even that has to route throught the APi Gateway

 lets setup API gateway project

Dependencies

	lombok
 	spring dev tools
  	Actuator
   	Eureka Discovery Client // So to registry API gateway with eureka
    	Reactive gateway 


**Spring Cloud Gateway** earlier the API gateway recommended for spring cloud was **Zuul**, but Zuul is no longer avaiable, Spring cloud gateway is the recommended one right now.

it is simple, yet effective way to route to APIs
So all the common features are implemented here, so it will be applicable to all the microservices as they are the common features.

it provides cross cutting concerns: concerns which applicable to multiple microservices like 
	Security
 	Monitoring/metrics etc

  It takes reactive approach, as it is built on top of Spring WebFlux

  Features:
  
  	Match routes on any req attribute
   	Define predicates (predicates are basically how you can match again a req) and filters ( add functionality or features for ex  authentication can be a filter authorization can be a filter logging can be a filter.
	it can integrate with Spring cloud Discovery client( load balancing) and it can discover the active instances of your service and load balance btw them. 
 	path rewriting ( what ever path coming we can change it before sending to the proxied services ( proxied services are microservices in our case conversion and exchange services)


Now lets make the configuration in application.properties of api-gateway like port and connect to eureka

		spring.application.name=api-gateway
		server.port=8765
		
		eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka

 	
 
