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


Now, as we have our currency exchange working, let's build our currency conversion  service


