# Node.js Practical Guidelines & Interview Questions (3 Years Experience)

**Target Profile:** Mid-Level Developer (TCS Wings1 / Digital / Prime / Elevate style)
**Focus Areas:** Practical implementations, clean code, database operations, error handling, and API integration. At the 3-year mark, interviewers expect you to be an independent contributor who writes production-ready code.

---

## 1. Practical API & Middleware Development

### Scenario: The Global Error Handler
**Question:** In an Express.js application, developers often sprinkle `try/catch` everywhere and send inconsistent error messages. How would you design a **Global Error Handling** architecture?

**Practical Guideline Response:**
1. **Custom Error Class:** I would create a custom `AppError` class extending the base `Error` to include an HTTP `statusCode` and an `isOperational` flag.
2. **Async Wrapper:** To avoid writing `try/catch` in every controller, I would implement a `catchAsync` wrapper function that forwards rejected promises directly to the next middleware.
3. **Global Error Middleware:** Provide a central `app.use((err, req, res, next) => { ... })` middleware. If `err.isOperational` is true, send a clean JSON response to the client. If false (a programming bug), log it heavily and send a generic "500 Internal Error" to avoid leaking stack traces.

### Scenario: JWT Authentication & Refresh Tokens
**Question:** Explain theoretically and purely practically how you implement a strict JWT authentication flow with short-lived access tokens and long-lived refresh tokens.

**Practical Guideline Response:**
- **Login:** On successful login, generate a JWT Access Token (expires in 15 mins) and a Refresh Token (expires in 7 days). Store the refresh token securely in an `HttpOnly` cookie or a database table.
- **Auth Middleware:** Create a `protect` middleware that reads the `Authorization: Bearer <token>` header, verifies the JWT using `jsonwebtoken`, and attaches the `req.user`.
- **Token Expiry:** If the access token expires, the frontend catches the `401 Unauthorized`. It then automatically hits the `/api/refresh` endpoint (passing the `HttpOnly` cookie). The server validates the refresh token, checks if it's blacklisted in the DB, and issues a new Access Token.

---

## 2. Database Operations (MongoDB / SQL)

### Scenario: Handling Heavy Aggregations & Pagination
**Question:** Your API endpoint `/api/products` is returning 20,000 records at once and crashing the frontend. How do you implement robust pagination and filtering?

**Practical Guideline Response:**
- **Implementation:** Do not fetch and filter in memory. Use `limit`, `skip`, and `sort` directly at the database level. 
- **Pagination Math:** `const skip = (page - 1) * limit`.
- **Aggregation:** In MongoDB (Mongoose), use the Aggregation Pipeline `$facet` to return both the paginated data array AND the `$count` of total available documents in a single database trip.
- **Search:** Use database indexes on frequently searched fields. For text search, implement `$regex` properly, or better yet, a text index, escaping user input to prevent NoSQL injections.

### Scenario: Transactions (ACID)
**Question:** A user makes a purchase. You need to deduct the balance from the User table and create an Order in the Order table. What happens if the server crashes exactly between those two steps?

**Practical Guideline Response:**
I would implement **Database Transactions**. 
- In SQL (Sequelize/Prisma): Start a transaction. Execute both the UPDATE and INSERT queries passing the transaction object. If both succeed, `commit()`. If any fails, `rollback()`.
- In MongoDB (Mongoose): Use MongoDB replica sets and `session.startTransaction()`. This ensures that either *both* operations succeed or the database state remains completely untouched.

---

## 3. Real-World Refactoring & Clean Code

### Scenario: Callback & Promise Hell
**Question:** Review the following concept: you need to fetch a user, then fetch their orders, then update the order status. Junior developers use nested `.then()`. How do you structure this cleanly?

**Practical Guideline Response:**
I will refactor it using clean `async/await` and `Promise.all` where applicable.
```javascript
// Clean Code Guideline:
exports.updateUserOrders = catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  if (!user) return next(new AppError('User not found', 404));

  // If fetching multiple independent orders, do NOT use a for-loop with await.
  // Instead use map and Promise.all to fetch concurrently.
  const orderPromises = user.orderIds.map(id => Order.findById(id));
  const orders = await Promise.all(orderPromises);
  
  // Logic here...
  
  res.status(200).json({ status: 'success', data: orders });
});
```

### Scenario: File Uploads & AWS S3
**Question:** How would you implement an API endpoint that receives a user profile picture and stores it?

**Practical Guideline Response:**
I would not store images on the server's disk because servers are ephemeral (can be destroyed/recreated).
- I would use `Multer` to parse the `multipart/form-data`.
- I would use `multer-s3` (or memory storage followed by AWS SDK upload) to stream the file directly from the user's request into an AWS S3 bucket.
- The database is only updated with the resulting S3 string URL.

---

## 4. TCS Wings/Digital Special: Core CS Applied

**Q: Explain how Node.js handles multiple requests simultaneously if it is Single-Threaded.**
*Guideline:* Remember to emphasize that Node is single-threaded *for Javascript execution*, but I/O operations (like database interactions, HTTP requests, file system) are delegated to the underlying OS (libuv thread pool). The Event Loop simply registers a callback and moves on to the next request without blocking.

**Q: How do you manage Environment Variables and Secrets in production?**
*Guideline:* Never commit `.env` files. Use standard libraries like `dotenv` for local. In production, rely on environment variables injected by the CI/CD pipeline, Docker, or tools like AWS Secrets Manager/HashiCorp Vault to keep database URIs and JWT secrets secure.
