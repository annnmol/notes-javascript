# Chapter 20 — System Design with JavaScript: Part 1 (Node.js Fundamentals & Express Core)

> At senior roles, interviews aren’t just about code. They test whether you can design scalable, efficient systems.  
> Node.js and Express are staples in backend interviews — expect questions on event loop, clustering, streams, scaling, and observability.

---

## 🔥 This file includes
- Node.js core concepts (event loop, libuv, threadpool, worker threads, clustering, streams, EventEmitter, errors, memory leaks, performance tips)  
- Express.js core concepts (middleware, routing, body parsing, error handling, security, scaling)  
- Minimal examples + outputs  
- Practice prompts (your answers preserved) + corrections  
- Rapid-fire snippets  
- Top 10 MNC / FAANG-style questions + crisp answers  
- Quick interview script & takeaways

---

# Part A — Node.js — Core Concepts (theory + minimal examples)

## 1) Single-threaded + Event Loop (high level)

Node runs JS on a single thread but handles many concurrent I/O operations via the **event loop** and **libuv** (the native layer).

- Call stack executes JS. When async I/O occurs, libuv delegates work (OS calls, threadpool) and schedules callbacks when finished.
- Event loop phases (simplified): `timers` → `pending callbacks` → `poll` → `check` → `close`.

**Key interview line:**  
> Node scales by non-blocking I/O, not by adding concurrency per thread.

**Minimal example (non-blocking):**
```js
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
// Output:
// A
// C
// B
```

---

## 2) libuv threadpool vs Worker Threads

- Some Node ops (fs, crypto, DNS lookup) use a **libuv threadpool** (default size ≈ 4). You can change via `UV_THREADPOOL_SIZE`.
- **Worker Threads** (since Node 10+) create actual JS threads for CPU-bound work (safe to run heavy compute off the main event loop).

**When to use what:**  
- Threadpool = built-in native offload for certain Node APIs.  
- Worker threads = run JS CPU tasks without blocking event loop.

---

## 3) Clustering (scale to CPU cores)

Node process is single-core by default. Use `cluster` or process managers (PM2) to fork worker processes — one per core.

```js
const cluster = require("cluster");
const http = require("http");
const os = require("os");

if (cluster.isMaster) {
  os.cpus().forEach(() => cluster.fork());
} else {
  http.createServer((req,res)=> res.end("ok")).listen(3000);
}
```

- Each worker has its own event loop & memory; master distributes connections (or external LB does it).
- Use clustering to scale across CPU cores.

---

## 4) Streams & Backpressure

- Streams handle data in chunks (Readable, Writable, Duplex). Ideal for files, network, large payloads.
- Use `pipe()` to stream from source → destination; Node handles backpressure for you.

```js
const fs = require("fs");
const r = fs.createReadStream("big.mp4");
const w = fs.createWriteStream("copy.mp4");
r.pipe(w);
```

**Why streams:** low memory footprint, faster for large data.

---

## 5) EventEmitter

Core pattern in Node — modules emit events, others listen.

```js
const EventEmitter = require("events");
const bus = new EventEmitter();
bus.on("msg", data => console.log("Got:", data));
bus.emit("msg", "hello");
```

---

## 6) Error handling & process events

- Synchronous: `try/catch`.  
- Async promises: `.catch()` or `try/await` inside async fn.  
- Global process handlers (last resort):
  ```js
  process.on('uncaughtException', handler);
  process.on('unhandledRejection', handler);
  ```
  Use these to **log and exit gracefully**, not to recover normal business flow.

---

## 7) Memory leaks — common causes

- Forgotten intervals/timeouts, big caches, global variables, lingering closures, event listeners not removed.  
- Tools: `process.memoryUsage()`, heap snapshots, Chrome DevTools for Node, `clinic.js`.

---

## 8) Performance best practices (quick bullets)

- Don’t block the event loop (no heavy synchronous compute).  
- Use streams for large I/O.  
- Cache (Redis) for frequently read data.  
- Use gzip/compression, keep-alive, connection pools for DB.  
- Profile and benchmark (autocannon, ab).

---

# Part B — Express.js — Core Concepts (theory + minimal examples)

## 1) Middleware model — core idea

A request flows through a chain of middleware functions: `(req, res, next)`. Middleware can log, auth, parse, or end the response.

**Minimal middleware example:**
```js
const express = require("express");
const app = express();

app.use((req,res,next) => {
  console.log(req.method, req.url);
  next();
});

app.get("/", (req,res) => res.send("Hello"));
app.listen(3000);
```

---

## 2) Routing & Routers

- `app.get/post/put/delete` register routes.
- Use `express.Router()` to modularize routes per resource and attach middleware to routers.

---

## 3) Body parsing & static files

- `express.json()` for JSON body; `express.urlencoded()` for form bodies.
- `express.static(path)` serves static assets efficiently.

---

## 4) Error handling middleware

Signature must be `(err, req, res, next)`.

Centralize error formatting:
```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message || "Internal" });
});
```

---

## 5) Security (practical checklist)

- Use `helmet()` to set secure headers.  
- Enable CORS with `cors()` selectively.  
- Rate limiting (`express-rate-limit`).  
- Input validation (Joi, express-validator).  
- Don’t store secrets in code — use env vars / secret stores.

---

## 6) Performance & scaling patterns (Express)

- Keep handlers stateless (store sessions in Redis) so workers can be scaled.  
- Use `compression()` and caching headers.  
- Reverse proxy (NGINX) in front for TLS termination and static offloading.  
- Offload heavy work to worker queues (Bull, RabbitMQ).

---

## 7) WebSockets & real-time

- `socket.io` (or `ws`) for real-time.  
- Scaling websockets requires adapters (Redis adapter for `socket.io`) and sticky sessions or external pub/sub so events reach every server.

---

## 8) Testing + Observability

- Unit test controllers/services; use `supertest` for route tests.  
- Logging (pino/winston), metrics (Prometheus), tracing (OpenTelemetry/Jaeger).

---

# Minimal examples that show important behaviors

## Non-blocking fs (async)
```js
const fs = require("fs");
fs.readFile("small.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log("file read");
});
console.log("after readFile call");
// Output: after readFile call
// then later: file read
```

## Express route with async/await guard
```js
app.get("/user/:id", async (req,res,next) => {
  try {
    const user = await db.findUser(req.params.id);
    res.json(user);
  } catch (err) {
    next(err); // forward to error middleware
  }
});
```

---

# Practice — your turn (you already answered — I preserved them below)

Please answer the following (you already provided answers; they are preserved exactly as you wrote them):

**P1.** Why is Node better for a chat/streaming API than a CPU-bound image-processing service? (short answer)

**Your answer (preserved):**  
> Node.js is a single-threaded event-driven which follows the event loop to handle the thousands of thousands of requests at the same time. It is best for the input-heavy events. It can also handle the async events. It is best for a chat streaming API chat streaming API because it is best for the input-heavy task. Why? Because because in like the response per request may be slower than java or go but it can handle the thousands of requests at the same time efficiently. That's why it is better for chat streaming APIs than a CPU-bound image processing service.

**P2.** What does `process.on('unhandledRejection', handler)` do and why should you still avoid relying on it for control flow?

**Your answer (preserved):**  
> It is a error handler and it handles the error we encounter during the execution of our code. Since it is a global error handler and when this error is encountered, it can stop our service. So, it is best to use try-catch blocks or async try/catch or promise .catch blocks in our code itself rather than using the global error handlers.

**P3.** In Express, where should you remove an event listener you added on a per-request basis to prevent leaks?

**Your answer (preserved):**  
> In ExpressJS, we can remove the event listener added per request basis on the finally block, finally block code like when we use try, catch, and finally block. In the finally block since it will execute in the both the cases will execute so we can add that but I'm not sure about this

**P4.** How would you scale socket.io to many servers (brief architecture note — mention sticky sessions or Redis adapter)?

**Your answer (preserved):**  
> One of the common problems ... use the sticky sessions and use the Redis adapters so that they can use Pub/sub methods so the communication can happen in the multiple node clusters .This way we can build a really scalable socket IO system which can scale up to many thousands of concurrent socket sessions at a single time.

**P5.** Give 3 concrete changes you would make to an Express REST API to improve throughput and latency.

**Your answer (preserved):**  
> we can use compressions second we can use redis for the caching JWT for session management reverse proxy or nginx for TLS termination ,pagination, query optimization, aggregation queries and if the data is frequently accessed we can store the data in redis and every time return the data cached from the redis and avoid use of heavy jobs we can offload them to worker queues or background jobs using rabbit mq or bull

---

## ✅ Polished corrections & interview-ready answers (do not memorize the preserved raw text — use these polished sound-bytes in interviews)

### P1 (polished)
**Q:** Why Node better for chat/streaming vs CPU-bound image processing?  
**Answer:**  
> “Node is event-driven and non-blocking, so it excels at I/O-heavy workloads like chat apps or streaming where many concurrent connections need fast responsiveness. CPU-heavy tasks (like image processing) block the event loop and should be offloaded to worker threads or separate services.”

---

### P2 (polished)
**Q:** What does `process.on('unhandledRejection')` do and why not rely on it for control flow?  
**Answer:**  
> “It catches unhandled promise rejections globally — useful for logging and graceful shutdown — but you should handle errors close to where they happen (try/catch or `.catch()`), not rely on global handlers for application logic.”

---

### P3 (polished)
**Q:** Where to remove per-request listeners in Express?  
**Answer & example:**  
> “Remove listeners in the request lifecycle handlers — e.g., `req.on('close', () => eventBus.off('x', handler))` — or in cleanup code after response. `finally` is OK in synchronous flows, but `req.on('close')` is the robust approach for streaming/long-polling routes.”

Example:
```js
app.get("/stream", (req, res) => {
  const handler = data => res.write(data);
  eventBus.on("data", handler);
  req.on("close", () => {
    eventBus.off("data", handler);
  });
});
```

---

### P4 (polished)
**Q:** How to scale Socket.IO across many servers?  
**Answer:**  
> “Use sticky sessions (so a socket remains connected to the same worker) and a pub/sub adapter (Redis adapter) to broadcast events across processes/servers. Or use a managed socket gateway (e.g., Socket.IO + Redis or NATS) and a load balancer in front.”

---

### P5 (polished)
**Q:** 3 concrete changes to improve Express throughput & latency  
**Answer (pick 3–5):**
- Enable gzip compression + caching headers.  
- Add Redis caching for hot responses with TTLs.  
- Offload heavy work to background queues (Bull/RabbitMQ).  
- Add DB indexes + pagination to reduce query time.  
- Use NGINX as reverse proxy for TLS termination and connection pooling.

---

# Rapid-fire snippets (quick mental checks)

- **R1**
  ```js
  console.log("A");
  setTimeout(() => console.log("B"), 0);
  console.log("C");
  ```
  **Output:** `A` `C` `B`

- **R2** (stream piping)
  ```js
  r.pipe(w);
  ```
  **Effect:** Streams data from read to write with backpressure handling.

- **R3** (EventEmitter)
  ```js
  emitter.once("greet", msg => console.log("Hi", msg));
  emitter.emit("greet", "Anmol");
  emitter.emit("greet", "Again");
  ```
  **Output:** `Hi Anmol` (only once)

- **R4** (cluster)
  ```js
  console.log(cluster.isMaster ? "Master" : "Worker");
  ```
  **Output:** `Master` in master process, `Worker` in each forked worker.

- **R5** (memory check)
  ```js
  console.log(process.memoryUsage());
  ```
  **Output:** object with heapUsed/heapTotal etc.

---

# Top 10 MNC / FAANG-style Questions (Node + Express) — with crisp interview lines

**Q1.** How does Node handle 10k concurrent requests with one thread?  
- **Answer:** Non-blocking I/O + event loop. Network/file calls are offloaded and callbacks scheduled — so the single thread remains available to handle many connections.

**Q2.** When would you use clustering vs worker threads?  
- **Answer:** Use **cluster** to utilize multiple CPU cores (process-level scaling). Use **worker threads** for CPU-bound JS tasks to avoid blocking the event loop.

**Q3.** How do streams help with large file processing?  
- **Answer:** Streams operate on chunks and manage backpressure; they avoid loading entire files into memory and allow processing to start before all data is present.

**Q4.** How to prevent memory leaks in Node apps?  
- **Answer:** Remove event listeners; clear timers; avoid global caches with unlimited growth; use heap snapshots and profiling to find leaks.

**Q5.** How to scale real-time websockets like Socket.IO?  
- **Answer:** Use sticky sessions + Redis adapter (pub/sub) or a message broker so events are broadcast across all instances; use LB + horizontal scaling.

**Q6.** How to handle backpressure on writable streams?  
- **Answer:** Check `stream.write()` return value; if `false`, wait for `'drain'` before resuming writes.

**Q7.** Where should you centralize error handling in Express?  
- **Answer:** Use an error-handling middleware `(err, req, res, next)` to format and log errors, while handlers forward errors using `next(err)`.

**Q8.** What are good cache strategies for APIs?  
- **Answer:** Use Redis with TTLs for frequently-read data; cache at CDN for static assets; invalidate/expire caches on writes.

**Q9.** How do you monitor a Node production app for performance?  
- **Answer:** Structured logs (pino/winston), metrics (Prometheus/Grafana), APM/tracing (OpenTelemetry, Jaeger), and heap/cpu profiling (clinic/Chrome DevTools).

**Q10.** If a route is slow in Express, how would you debug?  
- **Answer:** Profile with autocannon to reproduce; add per-route timing logs; inspect DB queries (explain plans), check external API latency, and run CPU/heap profiler.

---

# Answer Key (Practice recap)

- **P1:** Node best for I/O (chat/streaming) — event-driven, non-blocking; CPU-bound tasks block the event loop.  
- **P2:** `process.on('unhandledRejection')` is a global hook for unhandled Promise rejections — use for logging/graceful shutdown, not normal control flow.  
- **P3:** Remove per-request listeners in `req.on('close')` or cleanup after response (or in final lifecycle handlers).  
- **P4:** Scale Socket.IO with sticky sessions + Redis adapter (pub/sub) or use dedicated socket gateway.  
- **P5:** Use compression, Redis caching, reverse proxy (NGINX), pagination & indexes, offload heavy jobs to background workers.

---

## ✍️ Quick Interview script (2–3 lines to memorize)

- “Node scales via the event loop and non-blocking I/O; it’s ideal for many concurrent I/O connections but not for CPU-bound work.”  
- “For scale: use clustering/PM2 for multi-core, worker threads for CPU jobs, streams for large I/O, and Redis or CDN for caching.”  
- “In Express, centralize error handling, keep handlers small and async, and offload heavy tasks to background queues.”

---
