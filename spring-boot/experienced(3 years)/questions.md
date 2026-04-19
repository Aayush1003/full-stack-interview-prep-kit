# Spring Boot Practical Guidelines & Interview Questions (3 Years)

**Target Profile:** Mid-Level Developer
**Focus Areas:** Auto-Configuration, Spring Data JPA, Exception Handling, Security, and Environments.

---

### 1. How Auto-Configuration Works
**Question:** We say Spring Boot has "Auto-Configuration". How does it actually figure out what beans to configure under the hood?
**Practical Guideline Response:**
Spring Boot looks at the `META-INF/spring.factories` file (or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in newer versions) bundled inside its jar. This file lists hundreds of configuration classes.
Spring Boot uses `@Conditional` annotations like `@ConditionalOnClass` or `@ConditionalOnMissingBean`. If a class (like `DataSource.class` or `Hibernate.class`) is on the classpath, the auto-configuration runs and injects those default beans automatically. 

### 2. Spring Data JPA: Custom Modifications
**Question:** Spring Data JPA allows "derived query methods" like `findByFirstName(String name)`. But how do you write a bulk update query, and why does it sometimes crash?
**Practical Guideline Response:**
Derived queries are great for `SELECT`, but for `UPDATE/DELETE`, I have to use the `@Query` annotation and write actual JPQL or Native SQL.
For it to not crash, I must combine it with two annotations:
1. `@Modifying`: Tells Spring Data this query modifies the database context, not fetches data.
2. `@Transactional`: Placed on the service level (or respiratory level), ensures the bulk operation is wrapped in a transactional boundary so it can commit properly.

### 3. Managing Environments with Profiles
**Question:** Your database URL is `localhost` locally, but `aws-rds-url` in production. How do you practically manage this without changing code before deployment?
**Practical Guideline Response:**
I utilize **Spring Profiles**.
I create separate properties files: `application-dev.yml` and `application-prod.yml`.
When I run the app locally, I set my active profile to `dev` using IntelliJ or `-Dspring.profiles.active=dev` in the CLI. In AWS/Production, the environment variable `SPRING_PROFILES_ACTIVE` is set to `prod`. Spring Boot dynamically overrides the base properties with the profile-specific definitions.

### 4. JWT Security Flow
**Question:** Walk me through how you integrate Stateless JWT Authentication in Spring Boot.
**Practical Guideline Response:**
1. I add `spring-boot-starter-security`.
2. I disable CSFR and session creation (setting it to `STATELESS`).
3. I implement a custom `OncePerRequestFilter`. For every API call, this filter extracts the `Authorization: Bearer <token>` header.
4. I validate the token signature and expiration date.
5. If valid, I extract the user's username/roles from the claims, construct a `UsernamePasswordAuthenticationToken`, and set it in the `SecurityContextHolder`.

### 5. Actuator and Security Risks
**Question:** What is Spring Boot Actuator? Why can it be incredibly dangerous in Production?
**Practical Guideline Response:**
Actuator (`spring-boot-starter-actuator`) exposes built-in operational endpoints for monitoring application health, metrics, trace logs, and environment variables.
**The Danger:** If endpoints like `/actuator/env` or `/actuator/heapdump` are exposed to the public internet, hackers can download your environment variables (giving them DB passwords) or download memory dumps to steal active user tokens. 
**Fix:** In production, I strictly set `management.endpoints.web.exposure.include=health,info` and ensure all other actuator endpoints are secured behind admin credentials.
