**Servlets**

Servlets are Java programs, but instead of running them like regular Java apps with a compiler and main method, we run servlets on a web server. That means we need a web server like Apache 
Tomcat, Jetty, etc. Among these, Tomcat is one of the most popular.

Servlets are a server-side technology, which means the web server executes them.

To put it simply, a Servlet is a Java-based web component that runs on the server (inside a container like Tomcat) and is used to generate dynamic content—like HTML, JSON, etc.—based on 
client requests.

Servlets are mainly used for handling HTTP requests on the server side and generating dynamic responses. 
So,
	•	When a user interacts with a webpage (say clicks a button or submits a form), a request goes to the server.
	•	A servlet handles that request on the server.
	•	It processes the data (maybe talks to a database or performs some logic).
	•	Then it generates a dynamic response—usually HTML, JSON, or XML—and sends it back to the browser.

We use servlets to:
	•	Handle web requests (like form submissions)
	•	Talk to databases
	•	Generate dynamic content (HTML/JSON/etc.)
	•	Serve as the backend logic of a web application

If we want to send anything to the backend or get something back from the backend, we need something like Servlets to handle it.


so lets now build a simple html login page, we can add .html files in webapps folder.

	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="UTF-8">
	<title>Login</title>
	</head>
	<body>
	
	<form action="login">
	
	UserName: <input type="text" name="userName">
	Password: <input type="password" name="password">
	
	<br>
	<input type="submit" name="login">
	
	</form>
	</body>
	</html>


Now, lets write a servlet class for this login page ( as we know if we want to send anything to backend and if we want to get anything from backend then
we use this servlets. 

	package com.myproject.servlets;

	import jakarta.servlet.ServletException;
	import jakarta.servlet.http.HttpServlet;
	import jakarta.servlet.http.HttpServletRequest;
	import jakarta.servlet.http.HttpServletResponse;
	import java.io.IOException;
	import java.io.PrintWriter;
	
	public class Login extends HttpServlet {
	private static final long serialVersionUID = 1L;
    
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		PrintWriter out = response.getWriter();
			
		String username = request.getParameter("userName");
		String password = request.getParameter("password");
		
		out.println("welcome to Servlet Page " + "\n Username : " + username + "password : " + password);		
		
	}
	}

Now in web.xml we are defining the servlet, so here we are mapping the servlet so when ever we request for login then it will call the com.myproject.servlets.Login
	
	<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         version="4.0"
	         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
	                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd">
	
	<servlet>
		
			<servlet-name> Login</servlet-name>
			<servlet-class> com.myproject.servlets.Login</servlet-class>
		
	</servlet>
	<servlet-mapping>
	
			<servlet-name>Login</servlet-name>
			<url-pattern>/login</url-pattern>
	
	</servlet-mapping>
	</web-app>


 So when submit is clicked then it triggers the login url so cursor comes toweb.xml and see for login url is there any servlet class associated with
 it then it goes to that class and execute the class. 

Our servlet class prints output to browser by using response obj. this object is given by the tomcat. there is class called PrintWriter so from
that class we will use the getWriter() method using response object.

So as we want to get response from the webserver to brower we are using response object. on the other hand if we want to get the request from 
browser then we will use the request object.

So in the above code we have requested the username and password details from the browser and printed back to the browser.
to get the values from browser we have used the request object with getParameter("name attribute of that tag");. These getParameter return string so
incase of any othertype need to typecast.

	String username = request.getParameter("username");
 	String password = request.getParameter("password");




 
