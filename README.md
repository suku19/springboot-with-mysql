# springboot-with-mysql

Spring Boot - Accessing data with MySQL :: Learn how to set up and manage user details on MySQL and how to configure Spring Boot to connect to it at run time.

## **Setup Application:**

### Create a new database

>- mysql> create database user_db; -- Create the new database
>- mysql> create user 'root'@'localhost' identified by 'root'; -- Ignore if the user is already created
>- mysql> grant all on user_db.* to 'root'@'root'; -- Gives all the privileges to the new user on the newly created database

> **Note:** Find the mySql connection details in "/src/main/resources/application.properties"

### Create the application.properties file

>- In the sources folder, you create a resource file src/main/resources/application.properties

```
  spring.jpa.hibernate.ddl-auto=update
  spring.datasource.url=jdbc:mysql://localhost:3306/user_db
  spring.datasource.username=root
  spring.datasource.password=root
```
> **Note:** Here, spring.jpa.hibernate.ddl-auto can be none, update, create, create-drop, refer to the Hibernate documentation for details.
> [Database initialization](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html)

### Create the @Entity model

/src/main/java/com/user/app/entity/User.java

```
package com.user.app.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity // This tells Hibernate to make a table out of this class
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private String name;

    private String email;
    
    private String profession;

	public String getProfession() {
		return profession;
	}

	public void setProfession(String profession) {
		this.profession = profession;
	}

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}
}
```
 
This is the entity class which Hibernate will automatically translate into a table.

### Create the repository

/src/main/java/com/user/app/repository/UserRepository.java

```
package com.user.app.repository;

import org.springframework.data.repository.CrudRepository;

import com.user.app.entity.User;

// This will be AUTO IMPLEMENTED by Spring into a Bean called userRepository
// CRUD refers Create, Read, Update, Delete

public interface UserRepository extends CrudRepository<User, Long> {

}
```

This is the repository interface, this will be automatically implemented by Spring in a bean with the same name with changing case The bean name will be userRepository

### Create a new controller for your Spring application

src/main/java/hello/MainController.java

```
package com.user.app.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import com.user.app.entity.User;
import com.user.app.model.UserVO;
import com.user.app.repository.UserRepository;

@Controller    // This means that this class is a Controller
@RequestMapping(path="/users") // This means URL's start with /users (after Application path)
public class UserController {
	@Autowired // This means to get the bean called userRepository
	           // Which is auto-generated by Spring, we will use it to handle the data
	private UserRepository userRepository;
	
	@RequestMapping(method=RequestMethod.GET) 
	public @ResponseBody Iterable<User> getAllUsers() {
		// This returns a JSON or XML with the users
		return userRepository.findAll();
	}
	
	@RequestMapping(path="/{id}", method=RequestMethod.GET) 
	public @ResponseBody User getUser(@PathVariable long id) {
		// This returns a JSON or XML with the users
		
		return userRepository.findOne(id);
	}
	
	@RequestMapping(method=RequestMethod.PUT) 
	public @ResponseBody String addNewUser (@RequestBody UserVO userVo) {
		// @ResponseBody means the returned String is the response, not a view name
		// @RequestParam means it is a parameter from the GET or POST request
		
		User n = new User();
		n.setName(userVo.getName());
		n.setEmail(userVo.getEmail());
		n.setProfession(userVo.getProfession());
		userRepository.save(n);
		return "Saved";
	}
	
	@RequestMapping(method=RequestMethod.POST) 
	public @ResponseBody String updateUser (@RequestBody UserVO userVo) {
		// @ResponseBody means the returned String is the response, not a view name
		// @RequestParam means it is a parameter from the GET or POST request
		
		User n = new User();
		n.setId(userVo.getId());
		n.setName(userVo.getName());
		n.setEmail(userVo.getEmail());
		n.setProfession(userVo.getProfession());
		userRepository.save(n);
		return "Updated";
	}
	
	@RequestMapping(path="/{id}", method=RequestMethod.DELETE) 
	public @ResponseBody String deleteUser (@PathVariable Long id) {
		// @ResponseBody means the returned String is the response, not a view name
		// @RequestParam means it is a parameter from the GET or POST request
		
		User n = new User();
		n.setId(id);
		userRepository.delete(n);
		return "Deleted";
	}
}

```

The above example does specify GET, PUT, POST, and DELETE. @RequestMapping maps all HTTP operations according to the http method type. If you need same method multiple time within controller then you need to define unique path. Example:  ``` @RequestMapping(path="/testPath") ```

### Request and Response Value Object

/src/main/java/com/user/app/model/ResponseVO.java
/src/main/java/com/user/app/model/UserVO.java

```
package com.user.app.model;

public class ResponseVO {
	
	private boolean success;
	private String message;
	
	public ResponseVO(){}

	public boolean isSuccess() {
		return success;
	}

	public void setSuccess(boolean success) {
		this.success = success;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}
	
}
```
```
package com.user.app.model;

public class UserVO {
    private Long id;

    private String name;

    private String email;
    
    private String profession;

	public String getProfession() {
		return profession;
	}

	public void setProfession(String profession) {
		this.profession = profession;
	}

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}
    
    
}

```
Above model is use to create/map the request and response.

### Make the application executable

Although it is possible to package this service as a traditional WAR file for deployment to an external application server, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java main() method. Along the way, you use Spring’s support for embedding the Tomcat servlet container as the HTTP runtime, instead of deploying to an external instance.

src/main/java/com/user/app/UserDataManagementApplication.java

```
package com.user.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserDataManagementApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserDataManagementApplication.class, args);
	}
}

```
### Build an executable JAR

You can run the application from the command line with Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Maven, you can run the application using ./mvnw spring-boot:run. Or you can build the JAR file with ./mvnw clean package. Then you can run the JAR file:

java -jar target/usermanagement-0.0.1-SNAPSHOT.jar
The procedure above will create a runnable JAR. You can also opt to build a classic WAR file instead.
Logging output is displayed. The service should be up and running within a few seconds.

###Test the application

Now that the application is running, you can test it.

Install Postman. [Installing the Postman Chrome App](https://www.getpostman.com/docs/introduction)

> **Note:** Install Postman. Postman is available as a native app (recommended) for Mac / Windows / Linux, and as a Chrome App. The Postman Chrome app can only run on the Chrome browser. To use the Postman Chrome app, you will first need to install Google Chrome. 


 Import the collection file "SpringBootUserDataManagement.postman_collection.json". Click on the ‘Import’ button on the top bar, and paste a URL to the collection, or the collection JSON itself, and click ‘Import’. Find the more details in [Getting started with postman Collections](https://www.getpostman.com/docs/collections)

