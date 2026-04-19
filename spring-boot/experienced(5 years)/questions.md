# Spring Boot Architectural Guidelines & Questions (5+ Years)

**Target Profile:** Senior Developer / Node Architect
**Focus Areas:** Reactive Spring (WebFlux), Custom Starters, Advanced Testing, Asynchronous Processing, and Connection Pool Tuning.

---

### 1. Spring WebFlux vs Spring WebMvc
**Question:** Explain the architectural difference between Spring MVC and Spring WebFlux. In what exact scenario would you transition an app from MVC to WebFlux?
**Expert Answer:**
- **Spring MVC** uses a **Thread-per-Request** model (usually wrapped in Tomcat). Every incoming HTTP request consumes an OS thread. If an external API call or DB query is slow, that thread is blocked. At high concurrency, you exhaust your thread pool, crashing the app.
- **Spring WebFlux** (running on Netty) is **Event-Loop driven** and Fully Reactive (Project Reactor implementing Mono/Flux). A small number of threads handles all requests. I/O operations are non-blocking. 
- **Transition Scenario:** I would NOT migrate a CPU-heavy monolith. I would ONLY utilize WebFlux in a massive API Gateway or a microservice that is extremely I/O bound (streaming data, proxying hundreds of thousands of network requests concurrently to other microservices) where thread exhaustion is the primary bottleneck.

### 2. Creating Custom Spring Boot Starters
**Question:** In an enterprise company with 20 microservices, every team is copy-pasting the same JWT Auth logic and Logging configurations. How do you solve this architecturally using Spring Boot?
**Expert Answer:**
I would abstract the logic into a **Custom Spring Boot Starter** (e.g., `company-security-starter`).
1. Create an independent Maven/Gradle module.
2. Abstract the JWT `@Configuration` and Filters.
3. Use `@ConditionalOnProperty` to allow teams to toggle it on/off.
4. Add the configuration class to the `org.springframework.boot.autoconfigure.AutoConfiguration.imports` file.
5. Publish it to the company's private repository (Artifactory/Nexus).
Now, the 20 teams just add a single maven dependency, and Spring Boot magically auto-configures the security for them.

### 3. Proper Integration Testing using Testcontainers
**Question:** Many developers use H2 (In-Memory Database) for testing a PostgreSQL application. Why is this considered an anti-pattern for senior developers, and what is the alternative?
**Expert Answer:**
**The Anti-pattern:** H2 is a completely different dialect of SQL than PostgreSQL. If you use advanced features like JSONB types, Window functions, or custom PostgreSQL extensions, your H2 tests will fail or give false positives. You are testing against a database you don't actually release to production.
**The Alternative:** I use **Testcontainers**. Testcontainers hooks into Docker and spins up an actual ephemeral PostgreSQL container before the `@SpringBootTest` context loads, runs the tests against a real Postgres instance, and tears the container down afterward.

### 4. Tuning HikariCP for High Throughput
**Question:** During load testing, your application starts throwing "Connection is not available, request timed out after X ms." How do you analyze and tune the database connection pool (HikariCP)?
**Expert Answer:**
HikariCP's default maximum pool size is 10. When concurrency spikes, threads wait for a database connection to release and eventually time out.
1. **Analysis:** I would enable Hikari metrics using Micrometer (`spring.datasource.hikari.pool-name`) and monitor active vs. idle connections in Prometheus/Grafana.
2. **Tuning:** I'd increase `spring.datasource.hikari.maximum-pool-size`. 
3. **The Catch:** I wouldn't just blindly set it to 1000. PostgreSQL connections are heavy OS processes. Based on PostgreSQL formula: `Connections = ((core_count * 2) + effective_spindle_count)`, I'd tune it generally between 20 to 50 for a typical decoupled microservice to avoid context-switching overhead on the database server.

### 5. Managing `@Async` Thread Pools
**Question:** You annotated a heavy email-sending method with `@Async` so it doesn't block the API response. On Black Friday, the app ran out of memory (OOM Crash). Why did `@Async` crash the app, and how do you fix it?
**Expert Answer:**
**The Root Cause:** By default, Spring's `@Async` relies on a `SimpleAsyncTaskExecutor`. This executor does *not* reuse threads. It spawns a brand new thread for every single task! If 10,000 emails are triggered instantly, it spawns 10,000 threads, blowing up memory and crashing the JVM.
**The Fix:** I must override the default executor by providing a custom `ThreadPoolTaskExecutor` bean. I would explicitly set the `CorePoolSize`, a `MaxPoolSize` (e.g., 50 threads), and crucially, define a `QueueCapacity`. If the queue overflows, I'd define a `RejectedExecutionHandler` to either drop the email, execute it synchronously, or throw an exception rather than crashing the system.
