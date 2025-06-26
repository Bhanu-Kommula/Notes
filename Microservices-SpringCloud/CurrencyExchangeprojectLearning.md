Here lets implement two things 
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



  



