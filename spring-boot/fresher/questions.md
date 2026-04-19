# Spring Boot Interview Questions for Freshers

**Target Profile:** Entry-Level / Fresher
**Focus Areas:** Core Spring Boot Concepts, Annotations, Configuration, and Basic Request Handling.

---

### 1. Spring vs Spring Boot
**Question:** What is the fundamental difference between the traditional Spring Framework and Spring Boot?
**Answer:**
- **Spring Framework** is a massive ecosystem for enterprise Java apps, but configuring it to work requires heavy boilerplate, complex XML files, and manual dependency management.
- **Spring Boot** is an extension of Spring. It comes with "Auto-Configuration" and "Starter Dependencies" that automatically set up the boilerplate. It embeds a web server (like Tomcat) directly into the app, meaning you just write your code, click run, and it works out of the box (`Just run it` philosophy).

### 2. The Magic of `@SpringBootApplication`
**Question:** When you generated your project, the main class had `@SpringBootApplication`. What does this annotation actually do?
**Answer:**
It is a wrapper annotation that combines three very important annotations:
1. `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
2. `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the current package and its sub-packages, allowing it to find your controllers and services automatically.
3. `@Configuration`: Tags the class as a source of bean definitions for the application context.

### 3. Application Properties
**Question:** How do you pass hardcoded values from your configuration file into your Java code?
**Answer:**
We use the `application.properties` or `application.yml` file to store values (like database URLs or custom greeting messages).
In the Java class, I can inject these values directly using the `@Value("${property.key.name}")` annotation.

### 4. `@RestController` vs `@Controller`
**Question:** What is the difference between `@Controller` and `@RestController` in Spring Web?
**Answer:**
- `@Controller` is used for traditional web applications that return an HTML page (like Thymeleaf/JSP).
- `@RestController` is used for building REST APIs. It is actually a combination of `@Controller` and `@ResponseBody`. It ensures that the returned object is automatically serialized into JSON and written directly to the HTTP response body.

### 5. Dependency Injection Basics
**Question:** If you have an `EmployeeService` class and an `EmployeeController` class, how do you link them together without using the `new` keyword?
**Answer:**
I would annotate the `EmployeeService` with `@Service`. Then, in the `EmployeeController`, I use **Constructor Injection** or the `@Autowired` field annotation. Spring Boot’s Inversion of Control (IoC) container will automatically create the `EmployeeService` object and pass it into the controller when the app starts.
