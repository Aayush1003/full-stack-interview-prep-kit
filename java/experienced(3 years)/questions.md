# Java & Spring Boot Practical Guidelines (3 Years)

**Target Profile:** Mid-Level Developer
**Focus Areas:** Spring Data JPA, REST API Design, Java 8 Streams, Exception Handling, and IoC.

---

### 1. Spring IoC and Dependency Injection
**Question:** How does Spring's Inversion of Control (IoC) and Dependency Injection (DI) make applications more flexible? Explain `@Autowired`.
**Practical Guideline Response:**
IoC means handing over control of object creation to the Spring Framework. Instead of manually using the `new` keyword to create dependent objects, Spring creates them in the "Application Context".
`@Autowired` is used to inject these managed beans into other classes. I prefer to use **Constructor Injection** instead of completely relying on `@Autowired` on fields, as it makes unit testing without Spring much easier and guarantees the instance won't be null.

### 2. Handling REST API Exceptions
**Question:** A user requests a record by ID that doesn't exist. How do you return a clean Error JSON instead of a massive HTML stack trace?
**Practical Guideline Response:**
I utilize Spring's `@ControllerAdvice` and `@ExceptionHandler`.
When my service layer throws a custom `ResourceNotFoundException`, the `@ControllerAdvice` global class catches it, logs it appropriately, and constructs a clean generic JSON template containing an HTTP 404 status, a timestamp, and a user-friendly error message, passing it back via `ResponseEntity`.

### 3. Hibernate / JPA n+1 Problem
**Question:** What is the N+1 select problem in Hibernate, and how do you resolve it practically?
**Practical Guideline Response:**
**Problem:** If I fetch a list of 10 Authors, and each Author has a `@OneToMany` list of Books (which defaults to Lazy fetch), looping through the authors and printing their books will trigger 1 query for the authors, and exactly 10 additional queries for their books. Hence 1 + N queries, severely lagging the database.
**Solution:** I write JPQL queries utilizing `JOIN FETCH`. Alternatively, I use `EntityGraphs` in my Spring Data Repository to explicitly configure that when retrieving an Author, the Books should be eagerly fetched in one combined SQL join.

### 4. Java Core: Streams API
**Question:** Given a list of employee objects, construct a clean Java 8 stream pipeline to get a comma-separated String containing the upper-cased names of all developers who earn more than $5000.
**Practical Guideline Response:**
```java
String devNames = employees.stream()
    .filter(e -> "Developer".equals(e.getDepartment()))
    .filter(e -> e.getSalary() > 5000)
    .map(e -> e.getName().toUpperCase())
    .collect(Collectors.joining(", "));
```
This is drastically safer and cleaner than using conventional for-loops and if-conditions.

### 5. Transactions
**Question:** You have a service method annotated with `@Transactional`. It calls a database insert, and then immediately throws a runtime exception. Are the inserted rows saved in the DB?
**Practical Guideline Response:**
No, they are explicitly **Rollbacked**. Spring's Transaction Interceptor catches the runtime exception and automatically rolls back everything executed inside the method to maintain database integrity (ACID properties). *(Note: By default, Spring only rolls back on Unchecked Runtime Exceptions, not Checked Exceptions unless specifically told via `@Transactional(rollbackFor = Exception.class)`).*
