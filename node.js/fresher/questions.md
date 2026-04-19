# Node.js Interview Questions for Freshers

**Target Profile:** Entry-Level / Fresher
**Focus Areas:** Core JavaScript concepts in Node, basic Express.js, understanding of the file system, REST API basics, and the package ecosystem.

For fresher interviews, the focus is generally on your foundational understanding. Interviewers want to know if you understand what Node.js actually is, rather than just how to blindly copy-paste code.

---

### 1. What is Node.js and why do we use it?
**Question:** Explain what Node.js is in simple terms. Is it a framework, a library, or a programming language?

**Answer:**
Node.js is **not** a framework, library, or a programming language. It is a **Runtime Environment** built on Chrome's V8 JavaScript engine. 
Historically, JavaScript could only run inside a web browser. Node.js takes JavaScript and allows it to run on the server (the backend). We use it because it is fast, highly scalable, uses an asynchronous non-blocking I/O model, and allows developers to write both frontend and backend code in the exact same language (JavaScript).

---

### 2. Synchronous vs. Asynchronous File Operations
**Question:** What is the difference between `fs.readFile` and `fs.readFileSync` in Node.js? Which one is better to use in a web server?

**Answer:**
- `fs.readFileSync` is **synchronous**. It means the code will pause and wait until the entire file is read before moving to the next line of code.
- `fs.readFile` is **asynchronous**. It starts reading the file in the background and allows the rest of the code to keep running. Once the file is fully read, a callback function is triggered with the data.
- **Which is better:** In a web server, you should almost *always* use the asynchronous version (`fs.readFile`). If you use the synchronous version, you will block the main thread, and no other users will be able to access your website until that file finishes loading.

---

### 3. NPM and Dependencies
**Question:** What is NPM? What is the core difference between `package.json` and `package-lock.json`?

**Answer:**
- **NPM (Node Package Manager):** It is a tool used to install, share, and manage third-party libraries (packages) in a Node.js project.
- **package.json:** This file holds the metadata about the project (name, version) and lists the *approximate* versions of dependencies your project needs (like `"express": "^4.18.2"`).
- **package-lock.json:** This file is generated automatically. It locks down the *exact* versions of the packages and their sub-dependencies. This ensures that if another developer clones the project, they get the exact same environment, preventing "it works on my machine" bugs.

---

### 4. Basic REST API & HTTP Methods
**Question:** What are the most common HTTP methods used in RESTful APIs, and what do they generally do?

**Answer:**
- **GET:** Used to retrieve or fetch data from the server (e.g., getting a list of users). It should never modify data.
- **POST:** Used to send new data to the server to create a new record (e.g., submitting a signup form).
- **PUT:** Used to securely update or replace an existing record completely.
- **PATCH:** Used to partially update an existing record.
- **DELETE:** Used to remove a record from the database.

---

### 5. Middleware in Express.js
**Question:** You are using Express.js. What is "Middleware"? Provide a simple example.

**Answer:**
Middleware functions are functions that sit in the middle of a request and a response. When a user sends a request, it goes through middleware before reaching the final route handler. Middleware can modify the request, log details, or even stop the request entirely (like checking if a user is logged in).

**Example:**
```javascript
const express = require('express');
const app = express();

// This is a middleware function
const loggerMiddleware = (req, res, next) => {
  console.log(`New request made to: ${req.url}`);
  next(); // You MUST call next() to pass control to the next function
};

// Applying the middleware globally
app.use(loggerMiddleware);

app.get('/', (req, res) => {
  res.send('Hello World!');
});
```

---

### 6. The Single-Threaded Nature of Node.js
**Question:** If Node.js only has a single thread, how can it handle thousands of concurrent users downloading files from a server without crashing?

**Answer:**
Node.js handles this using the **Event Loop** and its **Non-Blocking I/O** architecture. 
When a request comes in that requires heavy work (like reading a file or querying a database), Node.js does not wait for it to finish. It delegates that heavy task to the background (the OS or internal thread pool) and immediately moves on to serve the next user request. Once the background task finishes, it sends the data back to the Event Loop to return the response.

---

### 7. CommonJS vs ES Modules
**Question:** What is the difference between `require('express')` and `import express from 'express'`?

**Answer:**
- `require()` belongs to the **CommonJS (CJS)** module system. It is the older, original way Node.js handles modules. It is synchronous.
- `import` belongs to **ES Modules (ESM)**. It is the modern JavaScript standard (used in React and modern Node). It is asynchronous and allows for features like Tree Shaking (removing unused code). 
*(Note: To use `import` in Node, you often need to set `"type": "module"` in your package.json).*
