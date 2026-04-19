# Node.js Interview Questions for Experienced Developers (5+ Years)

Welcome to the experienced tier! At this level, interviewers aren't just looking for "how" you write code, but "why" you make certain architectural decisions, how you handle production-grade systems, and your understanding of Node.js under the hood.

Here are some in-depth, scenario-based, and architectural questions that will make you feel great about your AI-assisted preparation! 🚀

---

### 1. Deep Dive into the Event Loop & Microtasks
**Question:** Explain the different phases of the Node.js Event Loop. Specifically, how do `process.nextTick()` and Promise microtasks interact with the event loop phases, and what happens if you recursively call `process.nextTick()`?

**Expert Answer / What to highlight:**
- **Phases:** Outline the 6 main phases: Timers, Pending Callbacks, Idle/Prepare (internal), Poll, Check (`setImmediate`), and Close Callbacks.
- **Microtask Queue:** Emphasize that `process.nextTick` and Promises are *not* technically part of the event loop. They form the microtask queue.
- **Execution Order:** `process.nextTick` has the highest priority and runs immediately after the current operation completes, before the event loop continues to the next phase. Promises run right after `process.nextTick` queue is drained.
- **The Trap:** Recursively calling `process.nextTick()` will starve the event loop (I/O starvation), preventing moving on to the Poll phase. Point out that `setImmediate()` should be preferred for recursive asynchronous callbacks to prevent blocking the event loop.

---

### 2. Diagnosing Production Bottlenecks
**Question:** A critical Node.js microservice is experiencing intermittent 100% CPU spikes, leading to increased latency and dropped connections. Production cannot be brought down. Walk me through your entire process to profile, diagnose, and fix this issue.

**Expert Answer / What to highlight:**
- **Immediate Mitigation:** Scale out horizontally (if not already) or restart unhealthy pods, but ensure to capture diagnostics first.
- **Profiling Tools:** Mention using tools like `node --prof` (though tricky in production), V8 Inspector, or APMs (Datadog/New Relic). Specifically, talk about generating **CPU Profiles (`.cpuprofile`)** or using **Clinic.js** (Doctor, Flame, Bubbleprof) in a staging replication.
- **Root Causes:** Discuss likely causes: Synchronous crypto operations (e.g., heavy `bcrypt` hashing on the main thread), massive JSON parsing (`JSON.parse` is synchronous and blocking), or ReDoS (Regular Expression Denial of Service).
- **Resolution:** Offload CPU-intensive tasks using **Worker Threads** (`worker_threads` module) or externalize to a different service (like a Go or Rust worker) or queue (RabbitMQ/SQS).

---

### 3. Scaling Real-Time Bidirectional Communication
**Question:** You need to design a high-throughput WebSocket server in Node.js that pushes real-time market data to 1 million connected clients simultaneously. How do you architect this to ensure low latency and horizontal scalability?

**Expert Answer / What to highlight:**
- **The Memory Problem:** A single Node.js instance cannot mathematically hold 1M WebSocket connections without exhausting memory limits / file descriptors (ulimit).
- **Horizontal Scaling:** You must deploy multiple Node.js instances behind a Load Balancer. Stickiness might be needed depending on the protocol, but standard WebSockets can be load-balanced with TCP routing.
- **The Pub/Sub Architecture:** If an event happens, how do all instances know to broadcast? Introduce a **Message Broker** like Redis Pub/Sub or Kafka. When the data payload is generated, publish it to Redis. Every Node.js WebSocket node subscribes to Redis, receives the payload, and broadcasts it to its local chunk of connected clients.
- **Optimization:** Use `ws` or `uWebSockets.js` (C++ bindings for extreme performance). Batch messages to reduce TCP overhead when possible.

---

### 4. Memory Leaks and Garbage Collection
**Question:** Explain how V8's Garbage Collector works. Then, describe how you would isolate and fix a memory leak in a Node.js API where memory steadily grows over days until `OOM (Out of Memory)` crashes occur.

**Expert Answer / What to highlight:**
- **V8 GC:** Briefly mention the Generational Hypothesis. New objects go to the "Young Generation" (Scavenger algorithm - fast, stop-the-world but very brief). Surviving objects move to the "Old Generation" (Mark-Sweep algorithm - runs concurrently/incrementally).
- **Finding the Leak:** 
  1. Trigger dynamic **Heap Snapshots** (`v8.getHeapSnapshot()`) or use the `heapdump` module. 
  2. Take three snapshots: Baseline, after load, and after load + GC cooldown.
  3. Load these into Chrome DevTools Memory Inspector and use the "Comparison" view to find the "Delta."
- **Common Culprits:** Global variables, unbounded caches (e.g., creating a Map without a size limit), forgotten event listener cleanups (e.g., `emitter.on` in a loop), and closures capturing massive variables that aren't released.

---

### 5. Advanced Streams Processing
**Question:** You are tasked with downloading a 50GB CSV file from a remote server, transforming the data (e.g., anonymizing PII), and uploading the result directly to an S3 bucket. How do you implement this in Node.js using less than 100MB of RAM?

**Expert Answer / What to highlight:**
- **The Concept:** Never load the whole file into memory. It must be processed chunk-by-chunk using **Streams**.
- **The Pipeline:** `pipeline` or `compose` from `node:stream/promises` is mandatory to handle backpressure and error propagation robustly.
- **Architecture:**
  1. **Readable Stream:** Fetch the file using `axios` with `responseType: 'stream'` or native `https`.
  2. **Transform Stream 1:** Use a CSV parser stream (like `csv-parser`) to chunk raw bytes into manageable Javascript objects.
  3. **Transform Stream 2:** Your custom `Transform` stream that securely hashes/anonymizes the PII data.
  4. **Transform Stream 3:** A stringifier to turn objects back into CSV rows.
  5. **Writable Stream:** Stream straight to S3 using AWS SDK's `Upload` utility which leverages multipart upload behind the scenes, pulling from the stream.
- **Backpressure:** Emphasize that streams automatically regulate the flow of data so a fast reader doesn't overwhelm a slow writer.

---

### 6. Node.js Security Best Practices
**Question:** Beyond standard things like SQL injection, how do you protect a mature Node.js API from Prototype Pollution, Timing Attacks, and ReDoS?

**Expert Answer / What to highlight:**
- **Prototype Pollution:** Prevent external JSON payloads from merging arbitrary properties into `__proto__` or `constructor.prototype`. Use safe merge functions or freeze prototypes (`Object.freeze(Object.prototype)`). Use robust validation tools like Zod or Joi.
- **Timing Attacks:** When comparing sensitive data like passwords or HMAC tokens, standard `===` operators "short circuit" on the first non-matching character, allowing attackers to guess characters. Countermeasure: use `crypto.timingSafeEqual()`.
- **ReDoS (Regular Expression Denial of Service):** Avoid catastrophic backtracking by strictly enforcing regex timeouts (using libraries like `re2` which runs in linear time), or by validating payload sizes *before* regex matching.

---

*These questions aim to test architectural thinking, practical debugging skills, and a deep understanding of the engine that powers Node.js.*
