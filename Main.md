This document outlines the steps to set up a Spring Boot application that interacts with a MySQL database for user management.

## Table of Contents
1. [Set Up Your MySQL Database](#set-up-your-mysql-database)
2. [Create a Spring Boot Project](#create-a-spring-boot-project)
3. [Application Properties](#application-properties)
4. [User Entity](#user-entity)
5. [User Controller](#user-controller)
6. [Testing with Postman](#testing-with-postman)

## Set Up Your MySQL Database

First, create a MySQL database and a `users` table:

```sql
CREATE DATABASE test_db;
USE test_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
```

## Create a Spring Boot Project

Create a new Spring Boot project using Spring Initializr or your preferred method. Select the following dependencies:
- Spring Web - Provides essential components for building RESTful web services and web applications. It includes features for handling HTTP requests, creating controllers, and setting up routes.
- Spring Data JPA - Offers easy integration with Java Persistence API (JPA) to interact with relational databases. It simplifies database operations such as CRUD (Create, Read, Update, Delete) by using repository interfaces without the need to write complex SQL.
- MySQL Driver - This is the JDBC driver required to connect a Spring Boot application to a MySQL database. It allows the application to communicate with the MySQL server and perform database operations.


## Application Properties

Configure your application properties to connect to the MySQL database by adding the following lines to `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test_db
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

## User Entity

Create a `Users` entity class to represent the `users` table:

```java
package com.example.demo;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Users {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;
    private String email;

    // Default constructor
    public Users() {}

    // Parameterized constructor
    public Users(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
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

## User Controller

Create a `UserController` class to handle user-related endpoints:

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
public class UserController {

    @Autowired
    private UserRepository userrepository; // Corrected spelling and case of UserRepository

    // Get all Users
    @GetMapping("/users") // endpoint path
    public List<Users> getAllUsers() { // Corrected the method name and return type
        List<Users> users = userrepository.findAll(); // Fixed the method call to use the repository
        
        return users; // Corrected spelling
    }
    
    // Get User by ID
    @GetMapping("/user/{id}") // endpoint path
    public Users getUser(@PathVariable long id) { // PathVariable annotation case and class name
        Users user = userrepository.findById(id).get(); // method call
        return user;
    }
    
    // Create User
    @PostMapping("/add") // endpoint path
    @ResponseStatus(code = HttpStatus.CREATED) // Corrected response status
    public void createUser(@RequestBody Users user) { // Corrected RequestBody annotation and class name
    	userrepository.save(user); // Fixed the method call and added a semicolon
    }
    
    @PutMapping("/user/update/{id}")
    public Users updateUser(@PathVariable Long id) { // Updated method
        Users user = userrepository.findById(id).get(); // Find existing user
     
            user.setName(user.getName()); // Update name
            user.setEmail(user.getEmail()); // Update email
            userrepository.save(user); // Save updated user
   
        return user; // Return the updated user
    }
    
    @DeleteMapping("/user/delete/{id}")
    public void removeUser(@PathVariable Long id) {
    	Users user = userrepository.findById(id).get();
    	 userrepository.delete(user);
    }
}

```

## Testing with Postman

1. **Get All Users**
   - Method: `GET`
   - URL: `http://localhost:8080/users`
   - Expected Response: A list of users.

2. **Get User by ID**
   - Method: `GET`
   - URL: `http://localhost:8080/users/{id}`
   - Replace `{id}` with the user ID.
   - Expected Response: User details or a 404 status if not found.

3. **Create User**
   - Method: `POST`
   - URL: `http://localhost:8080/users/add`
   - Body (JSON):
     ```json
     {
       "name": "John Doe",
       "email": "john.doe@example.com"
     }
     ```
   - Expected Response: Status `201 Created`.

4. **Update User**
   - Method: `PUT`
   - URL: `http://localhost:8080/users/update/{id}`
   - Replace `{id}` with the user ID.
   - Body (JSON):
     ```json
     {
       "name": "Jane Doe",
       "email": "jane.doe@example.com"
     }
     ```
   - Expected Response: Updated user details or a 404 status if not found.

5. **Delete User**
   - Method: `DELETE`
   - URL: `http://localhost:8080/users/delete/{id}`
   - Replace `{id}` with the user ID.
   - Expected Response: Status `204 No Content` or a 404 status if not found.
